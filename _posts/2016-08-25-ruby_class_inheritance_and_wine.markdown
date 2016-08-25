---
layout: post
title:  "Ruby class inheritance (and wine!)"
date:   2016-08-24 22:26:27 -0400
---


The long-awaited technical blog post has arrived! 

I was having a beer the other night when suddenly it came to me: the topic for my belated blog post, without which I had been unable to blog--drinks! I had been wanting to write about Classes and Instances, but was lacking an interesting example I could use. As an occasional home brewer, it occurred to me that beer is a great candidate--or any fermented beverage, really--for discussing inheritance (we'll be discussing wine here).

The reason beer and its cousins are perfect for illustrating inheritance is that all fermented beverages have certain characteristics in common, yet each type and subtype and style have unique characteristics as well. I'm going to start off simple for this post, with plans to revisit this concept in future posts and refactor and expand the code. Maybe eventually it'll even turn into something useful, like a way to look up brewing recipes or something similar. But let's not get ahead of ourselves.

We'll start with an overall ```Beverage``` class, which should initialize with a few attributes. Every beverage needs a name and the ingredients that go into fermenting. For those who don't already know, every fermented beverage is produced by the action of yeast on a starch of some sort, such as grapes for wine, honey for mead, apples for cider and malted barley (most often) for beer. The yeast eats the starch or sugar over time and from it produces two things: alcohol and carbon dioxide (which is also how you can carbonate these drinks). So, lets start with attributes of a ```name```, a ```fermentable``` and a ```yeast```.

```
class Beverage

  attr_accessor :name, :fermentable, :yeast

  def initialize(name:, fermentable:, yeast:)
    @name = name
    @fermentable = fermentable
    @yeast = yeast
  end
	
end
```

Now we can create classes for specific types of fermentable beverages. To keep things simple still, let's start with wine which will inherit its attributes from the ```Beverage``` class using ```<```.

```
class Wine < Beverage
end
```

Let's create a new wine and initialize with its name and ingredients. We'll make a Merlot:

```
merlot = Wine.new(name: "Merlot", fermentable: "Merlot grapes", yeast: "Pasteur Red")
```

Now let's call some of the ```Beverage``` methods on ```merlot``` and see if it worked:

```
merlot.yeast
=> "Pasteur Red"

merlot.fermentable
=> "Merlot grapes"
```

Great! We didn't even have to define ```initialize``` for ```Wine``` since it inherited from ```Beverage``` correctly. That's all for now. Next time, we'll work on refactoring and expanding this code, and hopefully get to make one of my favorite beverages: beer!

Cheers!





