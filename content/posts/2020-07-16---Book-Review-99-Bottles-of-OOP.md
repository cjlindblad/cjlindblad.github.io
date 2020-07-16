---
title: Book review - 99 Bottles of OOP (2nd Edition)
date: "2020-07-16T12:40:32.169Z"
template: "post"
draft: false
slug: "book-review-99-bottles-of-oop-2nd-edition"
category: "Software Design"
tags:
  - "Software Design"
description: "I read 99 Bottles of OOP (2nd Edition) and wrote some thoughts about it. Spoiler - I liked it alot!"
---

I read the latest edition of Sandi Metz et al's [99 Bottles of OOP](https://sandimetz.com/99bottles). I read the Ruby variant, but there's also a JavaScript one. I'll probably work through that one as well. It's that good!

### Don't Repeat Yourself

First, some context. I've been programming professionally for about five years. I read Clean Code and Pragmatic Programmer pretty early on. They appear in [pretty](https://www.serverless.com/blog/software-engineering-resources) [much](https://dev.to/awwsmm/20-most-recommended-books-for-software-developers-5578) [every](https://medium.com/better-programming/10-must-read-books-for-software-engineers-edfac373821b) [recommended](https://medium.com/@YogevSitton/the-ultimate-reading-list-for-developers-e96c832d9687) [software](https://sizovs.net/2019/03/17/the-best-books-all-software-developers-must-read/) [developer](https://www.freecodecamp.org/news/9-books-for-junior-developers-in-2019-e41fc7ecc586/) [reading](https://simpleprogrammer.com/best-programming-books-2019/) [list](https://bookauthority.org/books/best-software-development-books) out there, so they're kind of hard to miss.

The advice they contain seems very reasonable. Code should always be clean. Methods should be short, and then they should be even shorter than that. Knowledge should never be repeated ([DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)).

This IS good advice, but it isn't ALWAYS good advice. When I read the books, I didn't have the experience to say "yeah, well, it depends". Because it always depends. That's why you shouldn't get dogmatic about these things.

### Duplication might be OK

So, what's the problem with clean code?

To make code clean/dry, you'll most certainly have to add a layer of indirection. Or in other words, an abstraction. Abstractions will (hopefully) make the code easier to change, but might make the code harder to read.

Sandi Metz thinks that we should [prefer duplication over the wrong abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction). And that really resonates with me, at this point in my career. We tend to have a lot of respect for existing abstractions. We'll tack on another parameter and add one more conditional to a function that's almost suitable for our use case. This eventually leads to a tangled mess that you're afraid to touch.

### 99 bottles

99 Bottles of OOP is an expansion of this mindset. In essence, you'll be writing a program that returns the lyrics to the namesake song.

The book's first version of the program contains a lot of duplication, but it passes all the tests and is quite easy to read. And that is an excellent place to stop at. If no new requirements arrive at our desk, there's no need to abstract the duplication. We should move on and work on other things. I find this immensely refreshing and pragmatic.

The rest of the book hands you new requirements and teaches you how to find good abstractions in the case you should need them. You'll be TDD-ing new requirements, refactoring code smells, and renaming classes as you learn more and more about the problem. You'll also be led down a couple of bad paths to show you how that feels, and how you'll recover. It feels like pair programming with a master, teaching you a long career's worth of hard-earned knowledge.

There's a lot of discussion about how it _feels_ to write code. Tiny steps backed by tests make the experience stress-free. It makes me think about the [seminal](https://www.amazon.com/dp/0321278658/) [books](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530/) by Kent Beck, which talks a lot about making developers feel safe. If you've experienced the stress of diving too far into a hairy code change that just turns everything into an unreadable plate of spaghetti, you'll appreciate these values.

### Takeaways

This book probably isn't for everyone. For the right person though, it's excellent. A lot of the lessons within will be learned with experience. And if you haven't experienced the problems that the book addresses, it might not resonate with you. But for someone with a moderate amount of experience, I think this one really hits home.

The [SOLID](https://en.wikipedia.org/wiki/SOLID) principles (and relatives) never really stuck with me before. Now I can _feel_ the value of the [Open-Closed Principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle) because this book made use it to implement changing requirements with ease. I have an intuition for why we should conform to the [Law of Demeter.](https://en.wikipedia.org/wiki/Law_of_Demeter) The tests will be way easier to write!

But above all, I feel like I've given a license to not abstract stuff I don't yet understand. And more importantly, to tear up existing abstractions that smells bad!
