---
layout: post
title: How to integrate Spotify and Genius API to easily crawl song lyrics with Python
author: Will Soares
categories: tech
tags: python
description: Get the lyrics for the song currently playing on Spotify
image: /images/posts/spotify/head_image.png
---

Hello, everyone!

For this post, I decided to talk about a Python script I wrote last year, which I am still using it very often nowadays.

This was one of those moments that you have an idea and you get really excited about it but have no clue on whether that would work as you expect or not. I started basically with some knowledge of Python and knowing which API I was going to use to accomplish that idea.

The goal was to implement a script with which I could fetch the lyrics for the song currently playing on Spotify. So, at first, I knew I was going to need something that would provide me with an interface I could use to connect that script with the Spotify service running on my computer. Also, as I was a loyal user of [Genius.com](https://genius.com) I knew that they have a large public API I could get some help from.

For the long-term Spotify users, you probably remember when Spotify had the functionality to show the song lyrics within the desktop app, right? Good old times. I still don't understand why that was removed but all we got now is the `Behind the Lyrics` feature, which shows some ~~useless~~ information and small snippets of the lyrics for the song we are listening (it seems it is available only on mobile devices).

<div class="center-align">
  <img class="responsive-img" src="/images/posts/spotify/behind_the_lyrics.png">
</div>

From here I will start to tear apart the script following the order it was implemented. If you want to check the full script before continuing, you can go to [this repository](https://github.com/willamesoares/lyrics-crawler) on Github. There, you will also have instructions on how to run it, but keep in mind that I will guide you through that in this post as well :)

### Connecting to Spotify

This was one of the things I had no clue how to do it so I had to do some googling to accomplish that. Luckily, this [answer](https://stackoverflow.com/questions/33883360/get-spotify-currently-playing-track/33923095#33923095) by [jooon](https://stackoverflow.com/users/1014870/jooon) on Stackoverflow saved me a lot of time, so _kudos_ to you sir.

Basically, as I am a Linux user, the Spotify client I use automatically creates a D-Bus interface called MPRIS - Media Player Remote Interfacing Specification.

From the [D-Bus](https://dbus.freedesktop.org/doc/dbus-tutorial.html#whatis) specifications page, the D-Bus API is commonly used to implement a service which will be consumed by multiple client programs. This kind of message system is designed for two specific cases: communication between desktop applications in the same desktop session and communication between the desktop session and the operating system, which includes any system processes (this is our case).

A D-Bus system has several layers and one of them is formed by wrapper libraries, which is the kind of tool we are using here. In order for the Python script to communicate with the Spotify desktop application, we will use the [dbus-python](https://pypi.python.org/pypi/dbus-python/1.2.4) wrapper library or technically called a `binding` for the `libdbus` library. After having that library installed in your system you can use the `dbus` module to create a `SessionBus` which we will use to communicate with Spotify.

If you are on Ubuntu and, for any reason, you do not have the `dbus-python` installed, you can install it by running: `apt-get install python-dbus`  


```python
import dbus

def get_current_song_info():
    # kudos to jooon from this stackoverflow question http://stackoverflow.com/a/33923095
    session_bus = dbus.SessionBus()
    spotify_bus = session_bus.get_object('org.mpris.MediaPlayer2.spotify',
                                         '/org/mpris/MediaPlayer2')
    spotify_properties = dbus.Interface(spotify_bus,
                                        'org.freedesktop.DBus.Properties')
    metadata = spotify_properties.Get('org.mpris.MediaPlayer2.Player', 'Metadata')

    return {'artist': metadata['xesam:artist'][0], 'title': metadata['xesam:title']}
```

In the code above we a simply creating a `SessionBus` instance and using that to create an `Interface` with the Spotify application.

From that connection, we can extract the metadata for that session, which will provide us with lots of info about the current Spotify session, including the name of the artist and song currently playing, which is what we need. Notice that this method returns a dictionary with two keys, `artist` and `title`.

### Connecting to Genius API

Since we already have the name of artist and song we want to search, we can go ahead and make a request to the Genius API.

For that, we need an access token, which we can get by creating a Genius API client. First, we have to [sign up](https://genius.com/signup_or_login) for an account (or sign in if you already have an account on Genius). After that, you can go to the Genius API documentation page in order to [manage your clients](https://genius.com/api-clients). The process to create a new client is pretty straightforward and after that, you will finally get your access token.

Once you have your access token we can implement the next method.

```python
import requests

def request_song_info(song_title, artist_name):
    base_url = 'https://api.genius.com'
    headers = {'Authorization': 'Bearer ' + 'INSERT YOUR TOKEN HERE'}
    search_url = base_url + '/search'
    data = {'q': song_title + ' ' + artist_name}
    response = requests.get(search_url, data=data, headers=headers)

    return response

```

This method receives the song and artist name we extracted from the Spotify session and sends a request to the Genius API. Notice we are using the [requests](http://docs.python-requests.org/en/master/) HTTP library to send a GET request.  


### Extracting the song lyrics

If everything works fine, you will have now a response object that contains tons of information about all the matches that were found in the API.

Due to the object shape we got from the request, we will have to iterate over the `hits` key in that object and look for an exact match using the `artist_name` variable.

```python
# Search for matches in the request response
    response = request_song_info(song_title, artist_name)
    json = response.json()
    remote_song_info = None

    for hit in json['response']['hits']:
        if artist_name.lower() in hit['result']['primary_artist']['name'].lower():
            remote_song_info = hit
            break
```

If we successfully have a match in that object, it means that the song we look for is available in the API and is now available in the `remote_song_info` variable.

Even though we have all of that information about a specific song, guess what, we do not have the LYRICS!! Exactly, the Genius API (as for the date this script was implemented) does not directly offer you the lyrics for a song. There is no endpoint to search for lyrics, as well.

In order to bypass that situation, we would have to, in fact, crawl the web page in which that song is available. In the `remote_song_info`, you will have the URL for that song.

```python
# Extract lyrics from URL if the song was found
if remote_song_info:
    song_url = remote_song_info['result']['url']
```

From here we basically have to make our script access that page and crawl the lyrics for us.

```python
from bs4 import BeautifulSoup

def scrap_song_url(url):
    page = requests.get(url)
    html = BeautifulSoup(page.text, 'html.parser')
    lyrics = html.find('div', class_='lyrics').get_text()

    return lyrics
```

This method receives the song URL and uses the [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) library to pull data from HTML files in an easier way.

During the implementation of this method, I had to analyze what was the structure for the HTML returned in order to figure out what I could use to extract only the text that represents the lyrics of the song. I ended up using the `div` that contains the class `lyrics`, which holds all the song lyrics. I hope they will never change the markup for that :)

We finally have the lyrics to print out to the console. Tha cool thing is that during the crawling process each line in the lyrics automatically receives a new line escape sequence (`\n`). With that, we get a good looking structure in the terminal.

<div class="center-align">
  <img class="responsive-img" src="/images/posts/spotify/lyric_example.png">
</div>

### How to use the Github repository

All the methods mentioned in this post are available in this [repository](https://github.com/willamesoares/lyrics-crawler).

In order to use it locally, you can simply clone the repository, add your access token to the `token.txt` file and from within the repo folder you can run the command below to get the lyrics for a song. Remember to open Spotify and play something :)

```shell
python get-lyric.py
```

The script should work with both Python 2.7 and Python 3. However, you should check which version you are using and which one you used to install the dependencies.

From the repository, you get some extra functionalities, such as:

 - Automatically have the song lyrics available in the `lyric-view.txt` file.
 - Search for any song by passing the artist and song names (this mode does not interact with Spotify so there is no need to open it). Check the [README.md](https://github.com/willamesoares/lyrics-crawler/blob/master/README.md).

One last thing I would like to notice is that you don't always have to be inside the folder to get the lyrics, if you create an alias for that command in your `.bashrc` file.

For me, I had to add this line to my `.bashrc` file.

```shell
alias lyric="python ~/repos/lyrics-crawler/get-lyric.py"
```
And also, I had to use an absolute path for the `token.txt` and `lyric-view.txt` files in the script code.

With that, I can simply run the command `lyric` from any directory and get the lyrics for the song playing via Spotify.

I hope this was a good reading for you. If you have any suggestions or features for the repository, let me know or open a Pull Request, I'll appreciate it :)

Thanks!
