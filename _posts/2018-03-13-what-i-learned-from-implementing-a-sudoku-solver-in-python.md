---
layout: post
title: What I learned from implementing a Sudoku solver in Python
author: Will Soares
categories: tech
tags: python
description: The pros and cons of knowing a language before using it to solve your problem
image: /images/posts/railway.jpg
---

Hey there!

For those who like storytelling, this post will probably be a good reading. However, if you are not into storytelling or just here for the technical part, jump to the `Show me the code` section.

For this post, I decided to write about the journey I took when I decided to implement a sudoku solver in Python (I hope I remember all the details and also what my legacy code is doing).

The language I chose to solve this problem is, in fact, one of the main reasons why I'm writing this post. Without it, maybe this whole story wouldn't be a journey worth telling.

Firstly, I'll explain in which context I decided to implement a solution for the [Sudoku](http://www.sudoku.com/) game. Of course, there are many things I usually think that I can solve with code, but those things rarely end up being implemented. So why was this case different from the others?

Well, last year I graduated in Computer Science and _luckily_ that was the same year that the Department of Education in Brazil was evaluating all the undergraduate courses in public and private universities. It was a test called [Enade](http://portal.inep.gov.br/enade), which is applied every three* years for each field of study.

As I was already a senior student, I had to take the test in order to complete my degree ([that is a law now in Brazil](http://portal.inep.gov.br/perguntas-frequentes4)). I was pretty comfortable in doing it (I just did not agree with it being a duty) and I thought that with a quick review on some CS related concepts, I was going to be fine. Well...

It is well known that this test gathers topics from the entire course (4 years, in general) and try to formulate questions, in which you are going to put things together in order to solve them. I was aware that I was going to face both pretty basic problems and really hard ones (those that you usually skip and come back at the end of the test just to be sure that you really have no idea how to solve it). It turns out that there was this one question that I thought it would be straightforward to answer but then it became the one I was going to skip, for sure.

The fact is that when I came back to the question I thought I was solving it but then when I finished writing a pseudolanguage solution I realized that there was a bug in my solution and I had no time to fix it :(

I was intrigued by that question, for real! So I decided I was going to implement it at home.

Why don't we move to the technical details right away?

### Show me the code! (almost there)

In the mentioned question, you would have to write an algorithm that applied the concepts of recursive programming and backtracking to solve a sudoku game. For that, you could assume that some of the required functions were already implemented, for instance, the function responsible for telling you if a specific number is allowed in a specific column and row (of course it had to be the easiest one, not helping that much here, right?).

Basically, you would have as an input an array of pre-populated positions and a common number (or any other allowed character) to represent the positions your algorithm will have to fill out.

And that was the situation the question was providing you with. The way you were going to reach a solution would be up to you, regarding programming language and general logic (you could even use a pseudolanguage). The requirements were that you implement a recursive solution with a backtracking behavior.

If you don't remember or have never heard about those two concepts, here is a quick review.

#### Recursive programming

When you apply a recursive approach to solve a problem you are basically tearing apart the functionality of your program into smaller programs that solve the same problem. In the end, what you have to do is to reuse (that's why it's called recursive) the smaller function you created to solve your problem in its entirety.

I know, the previous paragraph was not enough to make the concept clear (I myself didn't get the idea at the first or second, maybe the third attempt?).

Let me try again, putting it in a simpler way, a recursive program is the one that tries to solve a problem by calling itself. The thing is that in that second call, the input will be a portion of the original input, so it can solve smaller problems and finally put everything together to give you the final solution.

Well, can you imagine how confusing it is to keep track of what a function is doing if it keeps calling itself again and again? That's not fun to do, trust me.

The cool thing here is that you don't really have to do that in order to understand the concept. It is enough if you are sure that the recursive function you implemented to solve a small portion of your problem is giving you the correct output. If it helps you, just accept that this is some sort of magic :)

Maybe, by now, you are wondering how is this thing going to stop if it keeps calling itself?

Well, here comes an important concept in recursive programming: the *base case* - this is the situation in which your program arrived at, let's say, the smallest portion of your program and is able to return a value instead of a call to the same function. That is when your program starts returning values so it can reach a final solution.

Let's go to a real example.

```python
def sum (n):                           def sumR (n):
    res = 0                                if n == 1:
    for i in range(1, n+1):                    return 1
        res = res + i                      else:
                                               return n + sumR(n-1)
    return res
```

Both of the functions above are responsible for summing up to a given number and returning the total value. The difference is that on the left side, you have an iterative function (the "traditional" way of solving things) and on the right side it is a recursive version of it.

Notice that in the recursive method the function `sumR` is called within itself, but with an input of `n-1`. What is happening here? This is the _magic_ part I mentioned.

Let's imagine that the input for the `sumR` function is `3`, so `n = 3`.

In the first call, since 3 is not equal to 1, the program will execute the `else` block:  

  `return 3 + sumR(3 - 1)`

From here the program is going to enter in a new iteration and a new level in the stack of calls, but at this time the input will be `3 - 1`, which is 2.

Again, 2 is not equal to 1 so the else block will be executed again and now the input for `sumR` will be `2 - 1`, which is 1.

Oh hey, now the `if` block will be executed since the input is equal to 1. So what happens here is that the same function is not going to be executed again, instead, the value 1 will be returned.

With that value returned, it will be used in the previous call which was:  

  `return 2 + sumR(2 -1)`, equivalent to `return 2 + 1`, which is 3

Now we have one more level solved and the value obtained will be used in the first recursive call to `sumR`, which was:

  `return 3 + sumR(3 - 1)`, equivalent to `return 3 + 3`, which is 6.

And that is the final summation of the numbers from 1 to 3.

Did you notice how exhausting it is to keep track of that even for a small input? If not, try to do the same thing for an input of 5 or 10 :)

After understanding how recursion works, if you wanna keep practicing it, this [quiz](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-189-a-gentle-introduction-to-programming-using-python-january-iap-2011/lectures/MIT6_189IAP11_rec_problems.pdf) is a good starting point. It might not be that easy at the beginning but once you get it, it's like magic!

#### Backtracking

This concept is also important to have in mind before reading the solution presented in this post. The idea here is simply to look for a solution in a range of possible candidates by trying each one of them and then discarding it at the first sign of an incorrect path.

It looks like a dumb way to solve things, but that's it. It recursively tries all the possible solutions until it finds the one. Imagine that the program is going through all the branches one at a time and then coming back, in regret, because that was the wrong path and then it goes to the next one.

But it can't be that bad, right? Some solutions implement heuristics for the backtracking approach, in which the algorithm automatically refuses candidates that are obviously bad. Here is a good [article](http://jeffe.cs.illinois.edu/teaching/algorithms/notes/03-backtracking.pdf) on backtracking.

### Show me the code! (for real) - The Naive Solution

The very first thing I thought about before diving into this adventure was to choose a language with a syntax I would be comfortable with. That was the main reason why I chose Python. I didn't want to spend time checking semicolons or curly brackets at the beginning. I was actually trying to focus only on the algorithm logic.

So here we go. Starting from scratch.

As a developer I always try to break problems in tiny pieces, implementing a solution for those pieces and then putting everything together. For this problem, I started with the most basic functions, such as the one previously mentioned: validating an input in a specific position in the array.

Just a quick recap: the input for this algorithm should be an array with some prepopulated positions, those will be the fixed numbers that cannot be changed, as in the real sudoku game.

For the input array I set this example (with 0's representing the positions to fill out):

```python
V = [
    0, 0, 0, 2, 6, 0, 7, 0, 1,
    6, 8, 0, 0, 7, 0, 0, 9, 0,
    1, 9, 0, 0, 0, 4, 5, 0, 0,
    8, 2, 0, 1, 0, 0, 0, 4, 0,
    0, 0, 4, 6, 0, 2, 9, 0, 0,
    0, 5, 0, 0, 0, 3, 0, 2, 8,
    0, 0, 9, 3, 0, 0, 0, 7, 4,
    0, 4, 0, 0, 5, 0, 0, 3, 6,
    7, 0, 3, 0, 1, 8, 0, 0, 0
]
```
Here you would have lots of options as to how the shape of this array should be. I decided to use a 81 length array, due to simplicity, in my opinion. Notice I have not thought about performance issues so far :)

You could also have a `(9,9)` array shape, in which each item would be another array of 9 items. The nested arrays could represent either columns or rows, it is up to you.

##### `has_violation`

```python
def has_violation(V, value, index):
    pos_to_check = get_positions_to_check(V, index)

    if pos_to_check:
        for i in pos_to_check:
            if V[i] == value:
                return True
        return False
    else :
        return "No index available."
```

This is the function responsible for checking for violations when inserting a value in a given position. It receives the array 'V' with the current state of filled positions, the value to be inserted, and the index that will be filled.

Here, I am basically using the `get_positions_to_check` to get all the positions that intersect vertically and horizontally with the given index. With that, the function checks if at least one of these positions has the same value as the value passed as a parameter to the `has_violation` function. It returns `False` if no violations occur, `True` otherwise.

##### `get_positions_to_check`

Be aware that this one could possibly be unpleasant for your eyes :)

```python
def get_positions_to_check(V, i):
    positions_to_check = [i]
    before_up = i - 9
    after_down = i + 9
    before_left = i
    after_right = i + 1

    if i >= 0 and i <= 80:
        while before_up >= 0:
            positions_to_check.append(before_up)
            before_up -= 9

        while after_down <= 80:
            positions_to_check.append(after_down)
            after_down += 9

        while before_left % 9 != 0:
            before_left -= 1
            positions_to_check.append(before_left)

        while after_right % 9 != 0:
            positions_to_check.append(after_right)
            after_right += 1

        return positions_to_check
```

This is the implementation of the function previously mentioned. It basically checks for all the positions that intersect with a given index.

It does that by checking all the positions that are both up, down, right, and left of the specific index. Those positions are determined by a factor of 9 starting from the index received as a parameter. For instance, if the index received is 30 (showed in green in the image below), the positions in the up direction will all be located in indexes that differ from exactly 9 positions from each one.

<div class="center-align">
  <img class="responsive-img" src="/images/posts/sudoku/board1.png">
</div>
<small>Remember that 0's are used to represent blank positions.</small>

In this image, the values that are above the position in green are respectively in the positions 3, 12, and 21. Since we received 30 as the index to check, we have:

`3  == 30 - (9 * 3)`
`12 == 30 - (9 * 2)`
`21 == 30 - (9 * 1)`

With that is easy to get the positions that are vertically intersecting the received index.

For the positions in the same row, we have to work differently. Here I'm using the fact that each row starts with a multiple of 9 as its position number. For the index received, the first position in the same row has 27 as its index (you can count in the image above). So, is easy to use that as a stopping point for the positions on the left. We can use the same logic for the positions on the right, the difference is that the multiple of 9 will be the first number in the next row, so we don't want to include that in the `positions_to_check` array.

You should notice that the shape of the array ended up making this step a little bit cumbersome. Probably if I have used an array with a shape of `(9, 9)`, this step, specifically, would have been easier.

##### `fill_matrix`
```python
def fill_matrix(V, ignore_pos, i = 0, direction = 'fw'):
    if i > 80:
        return True

    if i in ignore_pos:
        if direction == 'fw':
            fill_matrix(V, ignore_pos, i + 1)
        else :
            fill_matrix(V, ignore_pos, i - 1, 'bw')
    elif V[i] < 9:
        if has_violation(V, V[i] + 1, i):
            V[i] += 1
            fill_matrix(V, ignore_pos, i)
        else :
            V[i] += 1
            fill_matrix(V, ignore_pos, i + 1)
    else :
        V[i] = 0
        fill_matrix(V, ignore_pos, i - 1, 'bw')
```

And finally, we get to the recursive function where all the _magic_ happens.

Here I'm using a pretty straightforward approach, which didn't have to be that simple, but it actually works pretty well for the purpose of this post.

In order to control the paths the algorithm should follow, I'm using some _flags_ (`fw` and `bw`) here as a sign that would tell the function to keep going `forward` or `backward`, in case it finds a wrong candidate. This is exactly where the backtracking requirement comes into play.

You can easily see that for a given index `i`, if it is in the list of positions to ignore or if the value in that position is already the highest value possible (9), then the algorithm should backtrack since that path is not the correct one. That is what the last line of the function is doing:

```python
fill_matrix(V, ignore_pos, i - 1, 'bw')
```

Probably, you have already noticed the first `if` statement in this function and that is exactly what you are thinking: the *base case*. This is the stopping point of the algorithm. It assumes that if the index has arrived at the last position without an error, it means that the sudoku was solved. Yay!

The other `if` statements in this function are pretty simple, you can understand it just by looking at it, of course, if you have grasped how recursive functions work.

To take a look at the entire script, you can check this [gist](https://gist.github.com/willamesoares/77c598269f409f3d50d09bf11246dfe8).


#### So what? What's the real point in all of this?

Well, so far it felt like I was really doing a great job and was finally coming up with a Sudoku solver until I finally tested it. Guess what happened?

IT DID NOT WORK!

Yeah, all of that hard work and fancy concepts did not give me the output I was expecting and instead, it threw an ugly and large error log, like the one in the image below.

<p class="center-align">
  <img class="responsive-img" src="/images/posts/sudoku/log1.png">
</p>

Unfortunately, as I was extremely frustrated at the moment, I didn't really see the message that says `RecursionError: maximum recursion depth exceeded in comparison` and ended up trying to debug the whole error log. Yes, I tried to follow what the algorithm was doing step-by-step (it was probably about 30 min doing that) until I realized that the problem was not related to the logic I was following, but with the language itself.

When I read that the problem was that the limit of recursion level was exceeded, I was like: why is that? I'm just trying to solve a simple Sudoku board!

In fact, the stack of recursive calls that the algorithm was building started to grow really big and after some research, I realized an interesting thing about Python.

```python
    Python does not offer you Tail Recursion Elimination.
```

But wait, what does `Tail Recursion Elimination` even mean? I knew about recursive programming and I thought that was the only type of recursion I would have to write. However, I realized that what I was doing was actually a kind of recursion called tail recursion.

Tail recursion is when the last line of your recursive function is a call to itself, and there is nothing more after that. That is exactly what the `fill_matrix` function looks like, right?

The thing here is that the tail recursion approach is much better for saving memory space since the computer does not have to do anything else after calling the recursive function, so it can forget about all the variables in that function and even reuse that space.

The real problem with the approach used was, in fact, the language I chose to solve it with. As the requirements of the question were that I should implement a recursive algorithm, Python is not really the best option to do that.

Even though I was doing tail recursion (without knowing), Python was not really doing anything with that, because it simply does not support it (or don't know what to do with it). Due to that, the stack of calls was growing too large.

What are the solutions here? Maybe using a programming language that offers Tail Recursion Elimination (functional programming languages often offer that) or doing some trick to make it work with Python.

After all, I don't blame Python for that, it wasn't built around the idea of recursion.

Just to let you know, I got the output I wanted by simply increasing the recursion limit supported in Python. That was a really dirty and dangerous way to solve it, but I just wanted to see if my algorithm was working. Actually, there are some modules with which you can add this feature to the language. See useful links below.

This was the line I used to increase the recursion limit.
```python
    sys.setrecursionlimit(10000)
```

As this post is growing as large as the stack of recursive calls of my algorithm, I want to summarise the idea I wanted to communicate. I learned the importance of knowing your language before using it to solve a specific problem. It is important to know what a language offers to you before even getting to think about applying it to a real-world problem. Imagine if that was a large project and I have spent lots of months implementing a solution just to realize later that I have a tragic limitation with a language and I would spend more time trying to work around it.

As regarding the learning experience, it was actually awesome that I used Python for this. Those are the kind of challenge that makes you learn the most.

After all, I could appreciate the Sudoku board solved and it felt like heaven.

This is the solution if you wanna make sure the algorithm works.

```python
 4, 3, 5, 2, 6, 9, 7, 8, 1,
 6, 8, 2, 5, 7, 1, 4, 9, 3,
 1, 9, 7, 8, 3, 4, 5, 6, 2,
 8, 2, 6, 1, 9, 5, 3, 4, 7,
 3, 7, 4, 6, 8, 2, 9, 1, 5,
 9, 5, 1, 7, 4, 3, 6, 2, 8,
 5, 1, 9, 3, 2, 6, 8, 7, 4,
 2, 4, 8, 9, 5, 7, 1, 3, 6,
 7, 6, 3, 4, 1, 8, 2, 5, 9
```

Ps.: sorry about some references being available only in Portuguese :(

Thanks for reading, I hope you liked it. If so, let me know so I can keep writing posts like this :)

Some useful links:
[StackOverflow - Does Python optimize tail recursion?](https://stackoverflow.com/questions/13591970/does-python-optimize-tail-recursion)
[Python Tail Recursion](http://chrispenner.ca/posts/python-tail-recursion)
[brauchel - tco module](https://github.com/baruchel/tco)
