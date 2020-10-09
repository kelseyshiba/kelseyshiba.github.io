---
layout: post
title:      "Easy Algorithm Tips & Tricks"
date:       2020-10-09 18:47:47 +0000
permalink:  easy_algorithm_tips_and_tricks
---


Although I'm at the end of my program, I know I am just beginning my journey. I am a very kinesthetic and visual learner, so the true challenge begins with learning algorithms.  I know that practice will help and I hope to acquire some patterns in problem solving, but the struggle is real!

To help solidify some knowledge, I am going to walk through a simple algorithm solve - to hopefully leave this here as I continue to develop so I can look back. 

Here is the problem I attempted to tackle today, along with notes and tips (mostly for myself).

https://leetcode.com/problems/reverse-integer/

Here are the details: 
Given a 32-bit signed integer, reverse digits of an integer.

Note:
Assume we are dealing with an environment that could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

Example 1:
Input: x = 123
Output: 321

Example 2:
Input: x = -123
Output: -321

Example 3:
Input: x = 120
Output: 21

Example 4:
Input: x = 0
Output: 0

When I first tackled this algorithm, I thought I could use some built-in methods, such as reverse. I did end up using that as part of the solution. 

**First Mistake / Tip**
**Know what type of data you are working with!**

Here is the blank code they leave you with:

```
var reverse = function(x) {

};
```

I immediately coded in this because I am more comfortable working with an array in this case. 

```
var reverse = function(x) {
     x.split(" ")
}
```

It immediately told me that x.split was not a function. Why? Because I am working with an integer, not a string! Good to notice that as a habit for myself as I know that could've been kind of embarassing had it been in a live interview situation. 

Second attempt at the code was to visualize each step of the solution. This is important for me because I have to see it to believe it :)

Here is the second attempt with notes:

```
var reverse = function(x) {
     let string = x.toString( )
		 let array = string.split(" ")
		 return array.reverse( ).join(" ")
}
```

Broken down in my head:

Given the first test case, 123 --
let string = x.toString() //  returns "123"
let array = string.split("") // returns ["1", "2", "3"]
array.reverse // ["3", "2", "1"]
array.join // "321"

I think the solution was allowing for a string return on Leetcode, but you'll see I return an integer in a later solution. This worked for the first test case, which bring me to my second self-tip.

**Always Think About Edge Cases/Other Cases**

What if the number is negative? If I run the code above on the integer -123, it would return 321-, which is not what we want. 

So, I could handle a few different ways, but decided to ask the number first, if it was positive or negative. Therefore, code now looks like this:

```
var reverse = function(x) {
     if (x > 0) {
		      let string = x.toString( )
		      let array = string.split(" ")
		      return Math.abs(array.reverse( ).join(""))
		 } else {
		      let positive = Math.abs(x)
					let string = positive.toString( );
					let array = string.split("")
					return -Math.abs(array.reverse( ).join(" "))
		 }
}
```

This solution solved the problem!  However, I wanted to address the final tip to prepare for interviews. 

**Is there a better, more efficient way to code this?**

The answer was most definitely yes. After viewing another solution, there is so much more to add to things to think about.  Here is one of the solutions that blew my mind a bit.

```
const reverse = x => {
     const limit = 2147483648
		 cont k = x < 0 ? -1 : 1;
		 const n = Number(String(Math.abs(x)).split(' ').reverse(  ).join(" "));
		 return n > limit ? 0 : n * k;
}
```

So, breaking this down:

**Setting the limit**

This is something I need to get more comfortable looking at and asking an interviewer. "Are there any constraints?" I missed the direction in the leetcode problem:

Constraints:

-2^31 <= x <= 2^31 - 1

The first const limit in this solution is addressing this constraint.  2^31 - 1 = 2147483648. 
The second line is then allowing the return value to be converted to negative or positive by multiplying it by either negative 1 or positive 1.

const k = x < 0 ? -1 : 1
In other words: If x is less than zero, return -1, if not, return 1.

The third line is a super consolidated way of doing a chained calculation, which I really love! 

const n = Number(String(Math.abs(x)).split(" ").reverse( ).join(" "));
In other words: Give me a number as the final result of taking the absolute number--
    let's say test case of 123 // absolute is 123
		turning it into a string  // "123"
		splitting it into an array //["1", "2", "3"]
		reversing it // ["3", "2", "1"]
		joining it // "321"
		running Number("321") = 321
		

This is incredible and I learned that I need to be more familiar with these built-in Javascript operations in order to be able to use them. 

**Do it the Way It Makes Sense To You**

However, my brain does not necessarily work that way right now. I hope someday it will.  Here is my final submission that works with these new constraints.


```
var reversed = function(x) {
     if (x > 0) {
		     let string = x.toString( )
				 let array = string.split("")
				 let result = Math.abs(array.reverse( ).join(""))
				 if (result < 2147483648) {
				      return result
					} else {
					     return 0
					}
			} else {
			     let positive = Math.abs(x)
					 let string = positive.toString( )
					 let array = string.split( )
					 let result = -Math.abs(array.reverse( ).join(""))
					 if (result > -2147483648) {
					      return result
					 } else {
					      return 0
					 }
			}
}
```

**Identify Your Weaknesses**

LeetCode is a great resource for checking runtime and memory storage.  While the runtime of my solution is about 100ms, which is faster than 48% of other submissions, the memory usage 40.5MB is only about 9% less than most of the solutions submitted. I need to rework it again and see if I can make it better in a way that I understand it. 


**Grit**

There's only one thing I truly know at this point. I am determined to solve things and get better at them. Maybe employers will not be able to see algorithms as one of my strongest qualities. However, I will keep at it until I improve, and I will keep on a problem until I can find a solution. To my friends and colleagues and fellow coders, Keep at it!

-- Kelsey
