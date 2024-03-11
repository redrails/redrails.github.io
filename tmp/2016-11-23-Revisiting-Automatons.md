---
title: "Revisiting Automatons"
author: sham
categories: [article]
tags: [java, automata, theory, computer science]
---

In my second year of studying Computer Science I was introduced to the theory of computation as well as the very important notions of Automata.

### What is Automata?

From Wikipedia:

> Automata theory is the study of abstract machines and automata, as well as the computational problems that can be solved using them.

#### Why is this important?

It is important because computers rely on internal machinery to be able to parse languages and to construct any computational model by interpreting the input. For example: trying to parse in a program written in C may have brackets, to parse this program the compiler will need to parse the brackets to verify if the program has correct syntax by checking for properly bracketed code. This is done by creating an Automaton which can be used for verification. 

#### Types of Automatons

In my study, we focused on the three main types of automatons:

- Deterministic finite automatons
- Non-deterministic finite automatons
- Pushdown automatons

A DFA or deterministic finite automaton provides some very useful tools that we can use such as parsing regular languages and being able to accept desired inputs. An example of this can be something like accepting a language such that $${α ∈ {0,1}∗ : |α|_{0} is even}$$.. Given this we can construct a DFA like the following:

![](https://i.imgur.com/B07kgEI.png)

This machine we have constructed has two states. One marked with a double circle denoting an "accept" state. We take our input from the left (accept) state and the vertices denote which state we land on given the input. If we reach the end of the input but are not on the accept state then our input is not accepted. More specifically in our example, if we do not have an even number of `0`s then our input will not be accepted.

#### Restrictions

The restriction with this is that we cannot keep a track of what symbols we are reading. How do we solve this? It's easy, we use a way of keeping a track; and a very simple way to do this is by utilising a stack which we can throw symbols onto and pop when we are checking for something. This is where Pushdown Automata helps us.

#### What is a PDA?

A PDA is very similar to a DFA as we have covered above, but adds the capability of using a stack. A PDA introduces a notion of "pushing" and "popping" symbols as we read inputs. It also introduces the notion of having a "stack symbol" which is just used to denote the bottom of the stack, i.e. where the stack finishes. 

We can construct a PDA for the language we mentioned above:

![](https://i.imgur.com/clTzgh2.png)

In this case, our stack symbol is represented by the `#` symbol and we denote an "empty" symbol by `λ`.

Now, using this notion we can solve a problem mentioned earlier about "properly bracketed expressions". Because we only need to know what symbols are pushed on i.e. `{`, `}`, `(`, `)`, we can use a symbol to represent these and pop accordingly.

### Solving the issue of properly bracketed expressions

#### What is the problem?

When we parse in a program like:
```
int main(){
	if((1+2) == 3){
		return 0;
	}
}
```
We know that this program will compile in the sense that it includes no syntactical errors in regards to brackets. So we want to be able to output a "yes" from this. On the other hand if we have a program like the following:
```
int main){
	if}(1+2)) == 3){
		return 0;
	}
}
```
This program should not compile, and should give us an output of "no" because the brackets aren't matching.

More specifically we need to have some sort of a hierarchy of symbols like:
```
(
	{ ()
		[
			(
			)
		]
	}
)
```
Where we can interpret the blocks of brackets and if they are in the correct order.

To do this, we can revisit our notions of pushdown automata and design a machine which can parse an expression like: `(())` or `(({[()]}))`

We can form the following to do this:

![](https://i.stack.imgur.com/sloQk.png)

The above PDA shows how we can process the brackets `(` and `)` but we can further expand this to accepting other types of brackets by simply doing the following:

![](https://i.imgur.com/TPHLTXx.png)

This enables us to use all the brackets we can.

#### Representing this in code form

As this is a simple machine just using one form of memory (stack) we can model it in Java by using the [java.util.Stack](https://docs.oracle.com/javase/7/docs/api/java/util/Stack.html) library which provides us with functions such as `pop`, `push` and `peek`. 

Here is the code I came up with:
```
import java.util.Stack;

class BracketMatching {

	private static boolean isValid(String input){

		Stack<Character> stack = new Stack<>();

	    char s;
	    for(int i=0; i < input.length(); i++) {

	        s = input.charAt(i);

	        if(s == '(') {
	            stack.push(s);
	        } else if(s == '{'){
	            stack.push(s);
	        } else if(s == ')'){
	            if(stack.empty()){
	                return false;
	            } else if(stack.peek() == '(') {
	                stack.pop();
	            } else {
	                return false;
	            }
	        } else if(s == '}') {
	            if(stack.empty()){
	                return false;
	            } else if(stack.peek() == '{') {
	                stack.pop();
	            } else {
	                return false;
	            }
	        }
	    }

	    return stack.empty();
	}

	public static void main(String []args){

		System.out.println(isValid("{}(){(())}"));			// matching
		System.out.println(isValid("{}(){())}"));			// not matching
		System.out.println(isValid("{25 + (3 – 6) * 8}"));	// matching
		System.out.println(isValid(" }{25 + (3 – 6)"));		// no matching

	}

}
```

In this program I have basically duplicated the process of what I modelled in the PDA I formed. For each `(` or `{` symbols I read I push them onto the stack. And for all the closing forms of those brackets (i.e. `)` or `}`) I pop the pushed symbols to see if they match. If at any point I fail to pop the correct symbol, then I can return a `false` response indicating that the bracket matching failed.

### Lessons learned

By revisiting this whole notion of designing machines, it's very helpful to visualise problems sometimes in the model of automatons because it helps us understand the underlying processes happening to compute some arbitrary input. Of course, the types of automatons I have covered in this post DO NOT address all of the issues that are presented to us in the real world, and in this case we look to using models of the Turing Machine which grants us a lot more power and control over what we know and what we need, when we need it.