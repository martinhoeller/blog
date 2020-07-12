---
title:  "Objects and Actions"
date:   2020-07-12 10:00:00 +0200
categories: Boing
tags: lua love2d game-development creative-coding boing
images: ../assets/images/boing_actions.png
---

This is post number 4 in a series of articles covering how I am building my pet project **Boing**, a scripting playground to create [Amiga style cracktros](https://www.youtube.com/watch?v=50WWFEBsgfk&).

Last time we have successfully set up Löve2D and tested loading a script. Now it is time to set up Boing's basic architecture and get something drawn on the screen.

<!--more-->

## Inspiration
When developing games I always liked the nodes and actions mechanism that I first encountered in [Cocos2D](http://cocos2d.org/) and which was also adopted in Apple's [SpriteKit](https://developer.apple.com/spritekit/). It makes it very intuitive to define actions like animations and assign them to an on-screen object. It is possible to assign multiple actions simultaneously, group them, play them in sequence or reverse them. It is a perfect match for Boing. In fact, I built the Paranoimia demo presented in the first article of this series using SpriteKit actions.

Löve2D does not provide a mechanism like that and also some research did not yield any 3rd party libraries that would provide similar functionality. So I decided to implement my own little nodes and actions framework, heavily inspired by and borrowing a lot from [Cocos2D-Objc](https://github.com/cocos2d/cocos2d-objc).

## Objection!
First I needed to implement the concept of nodes. In Cocos2D nodes can have sub-nodes and make up a whole hierarchy called the scene graph. For now I do not need this complexity in Boing and will only create a flat list of nodes, eventually implementing a tree-like structure only if really necessary.

Nodes are able to run actions, i.e. you can add actions to a node which will modify certain properties like its position, scale or rotation over time.

Actions have a certain inheritance hierarchy:

[![Boing Actions]({{ site.url }}{{ site.baseurl }}/assets/images/boing_actions.png)]({{ site.url }}{{ site.baseurl }}/assets/images/boing_actions.png)

The Lua language is not inherently object oriented like other languages, i.e. it does not support classes or inheritance out of the box. But it is easily possible to get a simple OOP setup working. I used a small library called [Classic](https://github.com/rxi/classic).

## Nodes
The `Node` base class defines the basic properties common for every node:

{% highlight lua %}
-- node.lua

Node = Object:extend()

function Node:new(position)
    self.position = position
    self.rotation = 0
    self.anchor = Point(0.5, 0.5)
    self.width = 0
    self.height = 0
    self.scaleX = 1
    self.scaleY = 1
end
{% endhighlight %}

The `node` class also has two functions that are meant to be implemented by its subclasses: `draw()` and `update()`.

The `update()` function will be called from `main.lua`'s `love.update(dt)` function and gets the time passed since the program started as the `dt` parameter. This will later be used to update the nodes actions and do any other time-dependent operations.

{% highlight lua %}
function Node:update(dt)
    -- TODO
end
{% endhighlight %}

The `draw()` function will be implemented by subclasses to do their specific drawing. Like `update()` this is called from `main.lua` in the `love.draw()` function for each node in the scene.

{% highlight lua %}
function Node:draw()
    -- to be implemented by subclasses
end
{% endhighlight %}

## main.lua
Back in the main file there are some additional things we need. First we need a place to store all the nodes and a way to add them:

{% highlight lua %}
-- main.lua

nodes = {}

function addNode(node)
    table.insert(nodes, node)
end
{% endhighlight %}

Then there are the implementations for `love.update()` and `love.draw()` which both for now just iterate all nodes in the scene and call their respective `update()` and `draw()` functions. This will get more complicated once we start supporting animatable properties.

{% highlight lua %}
function love.update(dt)
    for i, node in ipairs(nodes) do
        node:update(dt)
    end
end
{% endhighlight %}

{% highlight lua %}
function love.draw()
    for i, node in ipairs(nodes) do
        node:draw()
    end
end
{% endhighlight %}


## The First Node is a Box
To test out the setup let's create a simple node that just draws a box on the screen.

{% highlight lua %}
-- box.lua

Box = Node:extend()

function Box:new(position, width, height, color)
    Box.super.new(self, position)
    self.width = width
    self.height = height
    self.color = color
end

function Box:draw()
    love.graphics.setColor(self.color)
    love.graphics.rectangle("fill", self.position.x, self.position.y, self.width, self.height)
end
{% endhighlight %}

And let's also create a test scene:

{% highlight lua %}
-- box.lua

box = Box(Point(screenWidth / 2, screenHeight / 2), 50, 50, {1,1,1})
addNode(box)

-- screenWidth and screenHeight are globals defined in main.lua
{% endhighlight %}

And by running `$ love . box.lua` ... we have a box!

[![Boing Actions]({{ site.url }}{{ site.baseurl }}/assets/images/boing_box.png)]({{ site.url }}{{ site.baseurl }}/assets/images/boing_box.png)
