---
layout: post
title:      "Data types, road blocks and how to work on them"
date:       2020-05-28 03:42:46 +0000
permalink:  data_types_road_blocks_and_how_to_work_on_them
---


After completing my CLI project, I wanted to take a  moment to reflect on how much I l have learned. Most of the learning came from making mistakes, and learning from those mistakes.  The other piece of learning was self discovery.  Binding.pry has become much more familiar to me and so have ruby enumerables. I believe I still have a long way to go, but what I have discovered somewhat is my process.

My process consists of two items: 
1. Speaking to myself in layman's terms, and 
2. Visual guides

So much of this project was difficult, because I was unsure of what I created in the first place. It took a lot of binging pry to figure out if I was passing in or creating an object that I could operate on, an array of items, an array of objects, a hash, or a string. Once I knew that information I felt like I could actually move forward. The other thing I struggled with was passing information from one method to another. If I needed the information to be stored and then passed into the next method, I found myself initially making too many instance variables to solve that problem. I realized how that made my code easy to break. 

After some thought, I realized the better way to code was to pass those saved items on through arguments inside the next method (e.g. def method (argument)). It was easy to access and pass information that way and get rid of instance variables. 

In talking to myself, I often asked:
-What am I passing into this method or what am I trying to work on?
- What am I returning by using this?

I found that I was often incorrect. For instance, there was one point in my code where I returned an array of objects. For some reason, it was difficult for me to conceptualize how to access objects inside of an array I had returned, of filtered items.  In order to access the object attributes that I needed, let's say object.name for example, I had to iterate over the array and pull out those items:

Array.each { |object| puts object.name}

I learned more about enumerables and iteration the hard way, so I always tried to break certain things down for myself in layman's terms. For example, collect and select. I have a difficult time differentiating the two, and when to use those over the 'each' iterator. Therefore, I devised:

"I will **select** (which is also find_all Kels) things based on a condition to return from an already existing array"
"I will **collect** things so that I can operate or DO something to them and put them in a new array"
"I will use **each** when I need to DO something to each item but I don't need to save the return values, like using puts"

Then, when I needed extra help -- using sort, especially when I realized it needed to compare items with 2 things in the block - I created a chart (likely still needs to be edited, but here it is) ** Note that it also includes things we have learned about strings and other items.
![](https://drive.google.com/file/d/1Z2afN9rdtr8xigeRL325xyfow5AbYa0a/view?usp=sharing)


I also noticed a few patterns that I started to follow, especially when accepting input to check for verification. 

Pattern for input verification:
input = gets.strip.downcase
1. until condition/input == exactly this || or this || or this || input.to_i.between(1, array.length)
2. puts "Please enter a valid input"
3. input = gets.strip.downcase
4. end

Then I setup the input values and what each one did should someone get past that loop - also very helpful. 

Lastly, after watching many restructuring your code videos I thought very carefully about when the responsibility should be the object and not inside the CLI code. Usually when the lines started to exceed 4 or 5 things that included that class name I decided to attempt to make methods inside that object class instead of calculating it inside the CLI. 

There are so many more things to cover, but all in all - I learned a ton of stuff! I hope this is helpful to anyone else who decides to embark on their coding journal. It is possible to boil things down to your level of understanding. I look forward to learning more and sharing that knowledge. 
