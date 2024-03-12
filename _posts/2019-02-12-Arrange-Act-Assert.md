---
title: "Arrange Act Assert"
author: sham
date: 2019-02-12 19:00:00 +0800
categories: [article]
tags: [testing]
---

A better way to write unit tests.

I have recently started to care more and more about test-coverage and unit testing. It is something that I am starting to promote more and more, not only for numbers but because unit tests are important.
Why, you ask? Firstly because of TDD, but mostly because when you create a PR with some functional changes or development on an existing code-base, I have no idea what you might actually be doing. Unit tests
however, will help me understand and I here is what has helped me a great deal!

# Arrange Act Assert

A simple way of representing the different actions of each unit test.

1. *Arrange* - All of the necessary inputs as required.
2. *Act* - on the subject of the test e.g an object or an array.
3. *Assert* - The expectations match.

When laying out a test in this format, it makes it extremely simple to understand what is going on. It helps to organise the test in such a way where code isn't mixed up and the reader cannot discern between the "Arrange" and "Act".
It also helps to lay out the test in a logical format, a step-by-step outlook on what is being tested.

# Example

Let's take a look at an example and see how nicely this works. Let's create a simple function that transforms an array using a given function. I will be using JavaScript for this example but this can be applied to any language/test framework.

Suppose we have the following function in JavaScript.

```
const myFunction = arr => arr.map(x => x*2);
```

A simple function that multiplies each element in an array by `2`. Let's write a test for this, again, I'm using JavaScript so I'll use the jest test framework for the demonstration.

```
describe('', () => {

	test('myFunction', () => {
  
		// Arrange
		const myArr = [1,2,3,4];

		// Act
		const result = myFunction(myArr)

		// Assert
		expect(result).toEqual([2,4,6,8]);
    
	})
})
```

We can see how logical this unit test looks. First, we arrange the data that we want to play around with. Then we can execute `myFunction` and do *something* and finally we can assert our expectations to see what we get back.

Not only is this helpful to understand what we started with, what we did and what we got, but for someone who was to just look at `myFunction` they would be able to tell what the function does without looking at it. Of course, this doesn't
mean that every unit test will be so self-explanatory but it helps the reader to understand the general idea behind some functionality.
