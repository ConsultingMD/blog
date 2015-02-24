---
layout: post
title: "A Brief Introduction to TDD"
date: 2015-02-24 10:51:51 -0800
comments: true
categories: testing tdd
author: Grace Li <grace.li@grandroundshealth.com>
---

A couple of weeks ago, my colleague posted some good tips about using Rspec effectively. In this post, I'd like to take a step back and talk about test-driven development (TDD), a software development technique popularized by Kent Beck [1], the inventor of extreme programming. In a nutshell, TDD advocates that tests should be written before production code. The general process of TDD follows a Red - Green - Refactor cycle [1]:

1. *Red* - Write a failing test. Ensure that the test is failing for the right reasons.
2. *Green* - Write the minimum amount of code neccessary to pass the test.
3. *Refactor* - Refactor the existing code to remove duplication while ensuring that existing tests pass.

Why use TDD? Writing tests first promotes having more testible code, which facilitates building highly cohesive and loosely coupled components. TDD also helps shorten the amount of time you spend writing code before knowing whether your code works or not.

Starting a new project using TDD seems relatively straightforward -- apply the steps of red-green-refactor. However, software developers are often faced with building features on top of an already mature code base. In this case, writing tests first may be a daunting task, especially when we work with code that we are not yet familiar with. To alleviate this uncertainty, we can first write code to explore the landscape and reduce uncertainty (called a "spike"). Once we have gained a better understanding of how things work, we delete this exploratory code and return to using TDD to write the code for production.

Moreover, TDD can be applied naturally for bug fixing tasks. First, we can write a failing test which reproduces the behavior of the bug. Once the bugfix is done having had a previously failing test that now passes provides us with confidence that the defect was indeed repaired.

In this post, I've only touched the tip of the iceberg of TDD and software testing. If you are curious to learn more, I highly encourage you to check out the following resources:

1. Test-Driven Development by Example by Kent Beck
2. Rails 4 Test Prescriptions by Noel Rappin
3. Growing Object-oriented Software Guided by Tests by Steve Freeman
4. [Is TDD dead? A debate.](http://martinfowler.com/articles/is-tdd-dead)
5. [SaaS course from BerkeleyX on edx](https://www.edx.org/course/engineering-software-service-uc-berkeleyx-cs169-1x) - an excellent course which introduces Rails, agile development and TDD
6. Magic Tricks of Testing (Rails Conf) by Sandi Metz [video][] [slides][]
[video]: http://www.youtube.com/watch?v=URSWYvyc42M
[slides]: https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf
