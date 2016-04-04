---
layout: post
title: "Improve Names in Your Code"
date: 2016-04-04 11:13:39 -0700
comments: true
categories: code-style clean-code names
author: Belda Chan
---
## Clean Code
[Clean Code by Robert C. Martin](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) is a classic book that can be helpful to programmers of all experience levels, programming in any language. Since the advice is universal to coding, whenever I read a chapter, I find practices to use the very next time I code. Here are some thoughts from *Chapter 2: Meaningful Names*. I have highlighted the sub-headings used in the book in **bold**.

**Use Intention Revealing Names**
- As the author says, "The name of a variable, function, or class, should answer all the big questions. It should tell you why it exists, what it does, and how it is used."
- The names you create in the process of coding can help organize your thoughts. For example, when coding a game of Tic-tac-toe, 'determine_winner' can be a placeholder for a method you will need in the future.

**Avoid Disinformation**
- Only use words that have meaning to programmers if you take care to use them correctly. Examples include 'list' and 'factory'.

**Make Meaningful Distinctions**
- Avoid redundant naming, such as 'username' in the User class.
- If you add to a concept, take care to distinguish the original name from what you have added. For example, 'address' and 'address2' have more meaning when named 'mailing_address' and 'work_address'.

**Use Searchable and Pronounceable Names**
- As the author says, "If you can't pronounce it, you can't discuss it without sounding like an idiot."

**Pick One Word Per Concept**
- Minimizing these words lends consistency to your codebase. For example, synonyms like 'fetch', 'retrieve', and 'get' should not be littered throughout your classes. Read through a similar file in the codebase to observe the convention, then follow it.

## Now What?
These are the concepts that resonate with me. So, how can you use this information when coding tomorrow (or right now)?

- **Ask for help.** If you are stumped on a name for a new class or library, talking with a co-worker can help. A quick search online can also help. Recently, I came across StackExchange's "English Language & Usage" site when searching for [a word that describes both 'encoding' and 'decoding'](http://english.stackexchange.com/questions/76549/a-word-that-describes-both-encoding-and-decoding).

- **Comment on pull requests.** If you are wondering if a class, method, or test can have a better name, it probably can! You do not need to provide the ultimate solution; raising the concern can lead someone else to brainstorm something better.

- **Improve names as you code.** If you add to, or change, a method significantly, ask yourself if the name continues to be suitable. If you encounter a confusing name as you code, take the time to search the codebase and replace with a better name.

Better naming leads to self-documenting code. It clearly communicates your intent and decreases the cognitive load for everyone who works in the codebase. Good luck!
