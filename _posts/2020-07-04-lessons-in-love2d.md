---
title:  "Lessons in Love(2D)"
date:   2020-07-04 09:00:00 +0200
categories: Boing
tags: lua love2d game-development creative-coding boing
image: ../assets/images/talesfromthescript.png
---

This is the third post in a series of articles covering how I am building my pet project **Boing**, a scripting playground to create [Amiga style cracktros](https://www.youtube.com/watch?v=50WWFEBsgfk&).

In my [previous post](https://blog.martinhoeller.net/development/2020/07/02/boing.html) I described how the name **Boing** came to be and why I chose [Löve2D](https://love2d.org/) to be the 2D engine I want to work with. This post covers my vision for Boing, my first contact with the Lua programming language and my first experiments with the engine.

<!--more-->

## What I want Boing to be

As I mentioned before Boing should be a scripting playground. I want to be able to write a simple script that completely describes a demo scene and feed it into Boing. Boing will then interpret and run the scene.

A scene script could look something like this:

{% highlight lua %}
-- myscene.lua

-- play background music
playMusic("music.mp3")

-- create a white square with side length 50px at position 100/100
box = Rect(Point(100, 100), 50, 50, Color(1, 1, 1))
addNode(box)

-- move the box to 200/100 with a an animation duration of 1 sec
move = MoveTo(1, Point(200, 100))
box:addAction(move)

{% endhighlight %}

To run it, simply pass it into Boing from the command line: `$ ./boing myscene.lua`

## First steps in Löve2D
Getting started with Löve2D is incredibly easy. It is literally just downloading the binary, creating a `main.lua` file and writing a simple `love.draw()` function that prints "Hello World" on the screen. Although it is geared towards total programming beginners the [How to Löve Tutorial](https://sheepolution.com/learn/book/contents) by Sheepolution helped me a lot to get the basics of both Löve2D and Lua.

## Can this relationship work?
One major concern I had was packaging, distribution and loading scripts. To test this I created a simple `main.lua` file that parses the command line arguments for another file that it tries to execute.

{% highlight lua %}
-- main.lua

function love.load(args)
    local filename = args[1]
    if not filename then
        print("please provide a filename")
        love.event.push("quit")
    else
        dofile(filename) -- this is the magic line
    end
end
{% endhighlight %}

I also created a `test.lua` file that just had a `print` command.

{% highlight lua %}
-- testscript.lua

print("Tales from the script")
{% endhighlight %}

To my delight running it with `love . testscript.lua` actually worked and printed the line in the terminal window. A quick test packing up everything in a macOS app bundle yielded the same result. Nice!

![Tales from the script]({{ site.url }}{{ site.baseurl }}/assets/images/talesfromthescript.png)

Next up will be implementing the basic architecture of Boing and coming up with a way to interpret a scene script.