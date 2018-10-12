---
layout: post
title:  "The case for classes in Python"
date:   2018-05-25 17:00:35
categories: OOP partial
---

I have learned something and would like to share.

# Introduction

My story starts in 2013.
I was not very familiar with the basics of OOP back then and was getting into my first big programming adventures. I had taken java at high school and it's verbosity scared me off. I strongly preferred python for it's expressiveness.

Then I saw a "great" talk from pycon 2012 in which all my naive assumptions seemed to be confirmed by a professional:

* OOP is dead, evil and unnecessary for mere mortals
* OOP leads to verbose **(bad!!!)** code
* If you're writing classes in Python you didn't understand the language and are writing OOP **(bad!!!!)** code

And as a corollary:

* Functional programming is the holy grail and if you are using partial  then you're doing functional **(good!!!)** programming

I seemed to find confirmation by some former colleagues that
functools.partial is amazing and met other colleagues who were similarly
 amazed by the talk from PyCon. I then went on to apply "smart" patterns
in many places, but inevitably faced the shortcomings and poor results
of a dogma-driven-programming style.

It has taken me almost five years to come to the conclusion that the
talk is devoid of value, that I was wrong and that others were right.
While it's hard to pin down why and when I came to the realization
that partials are bad and classes are (often) preferable, the roots run
deep. So deep in fact they reach to the fundamentals of programming
themselves.

### Detour on the nature of Object Oriented Programming

*Please bear with me. I promise ... not much, but maybe you'll be entertained.*

A common definition of OOP runs around the following lines(source):
Object Oriented Programming adds four new tools to your arsenal:
* Encapsulation
* Abstraction
* Inheritance
* Polymorphism

And whenever something around these lines is said, the inevitable follow up is(from the comments to the source):

    Abstraction and polymorphism are tools provided by many "orientations" of programming. OOP actually offers a weaker form of encapsulation than other approaches because  inheritance encourages leaky abstraction designs. The only thing OOP really adds to the toolkit is inheritance, which is widely seen as a bad thing.

Robert Martin (Uncle Bob) makes this point very strongly in almost every talk he gives.(metasource)

I no longer think classes are terrible, especially in cases where you'd use partial otherwise

On the subject of partial, I think it still has its applications and usecases, especially when dealing with library functions that you can't (or shouldn't want to ) subclass.


Furthermore I think that many of the GangOfFour design patterns are still relevant

I apologize for any possible cr*p related to this I have said during the last years. ( I might apologize for this letter in the future ;-) )

