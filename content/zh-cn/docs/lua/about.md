---
title: "关于"
linkTitle: ""
weight: 1
---

## 什么是 Lua?

Lua is a powerful, efficient, lightweight, embeddable scripting language.
It supports procedural programming, object-oriented programming, functional programming, data-driven programming, and data description.

Lua combines simple procedural syntax with powerful data description constructs based on associative arrays and extensible semantics.
Lua is dynamically typed, runs by interpreting bytecode with a register-based virtual machine, and has automatic memory management with incremental garbage collection, making it ideal for configuration, scripting, and rapid prototyping.

## Lua 来自哪里?

Lua is designed, implemented, and maintained by a team at PUC-Rio, the Pontifical Catholic University of Rio de Janeiro in Brazil.
Lua was born and raised in Tecgraf, formerly the Computer Graphics Technology Group of PUC-Rio.
Lua is now housed at LabLua, a laboratory of the Department of Computer Science of PUC-Rio.

## What's in a name?

"Lua" (pronounced LOO-ah) means "Moon" in Portuguese.
As such, it is neither an acronym nor an abbreviation, but a noun.
More specifically, "Lua" is a name, the name of the Earth's moon and the name of the language.
Like most names, it should be written in lower case with an initial capital, that is, "Lua".
Please do not write it as "LUA", which is both ugly and confusing, because then it becomes an acronym with different meanings for different people.
So, please, write "Lua" right!

## 加入社区

There are several meeting places for the Lua community where you can go to learn and help others and contribute in other ways.
One of the focal points is the mailing list, which is very active and friendly.

You can meet part of the Lua community in person by attending a Lua Workshop.

## 支持 Lua

You can help to support the Lua project by buying a book published by Lua.org and by making a donation.

You can also help to spread the word about Lua by buying Lua products at Zazzle.

Lua.org is an Amazon Associate and we get commissions for qualifying purchases made through links in this site.

## 为什么选择 Lua?

### Lua 是一种经过验证的、健壮的语言

Lua has been used in many industrial applications (e.g., Adobe's Photoshop Lightroom), with an emphasis on embedded systems (e.g., the Ginga middleware for digital TV in Brazil) and games (e.g., World of Warcraft and Angry Birds).
Lua is currently the leading scripting language in games.
Lua has a solid reference manual and there are several books about it.
Several versions of Lua have been released and used in real applications since its creation in 1993.
Lua featured in HOPL III, the Third ACM SIGPLAN History of Programming Languages Conference, in 2007.
Lua won the Front Line Award 2011 from the Game Developers Magazine.

### Lua 快速的

Lua has a deserved reputation for performance.
To claim to be "as fast as Lua" is an aspiration of other scripting languages.
Several benchmarks show Lua as the fastest language in the realm of interpreted scripting languages.
Lua is fast not only in fine-tuned benchmark programs, but in real life too.
Substantial fractions of large applications have been written in Lua.

If you need even more speed, try LuaJIT, an independent implementation of Lua using a just-in-time compiler.

### Lua 是可移植的

Lua is distributed in a small package and builds out-of-the-box in all platforms that have a standard C compiler.
Lua runs on all flavors of Unix and Windows, on mobile devices (running Android, iOS, BREW, Symbian, Windows Phone), on embedded microprocessors (such as ARM and Rabbit, for applications like Lego MindStorms), on IBM mainframes, etc.

For specific reasons why Lua is a good choice also for constrained devices, read this summary by Mike Pall.
See also a poster created by Timm Müller.

### Lua 是可嵌入的

Lua is a fast language engine with small footprint that you can embed easily into your application.
Lua has a simple and well documented API that allows strong integration with code written in other languages.
It is easy to extend Lua with libraries written in other languages.
It is also easy to extend programs written in other languages with Lua.
Lua has been used to extend programs written not only in C and C++, but also in Java, C#, Smalltalk, Fortran, Ada, Erlang, and even in other scripting languages, such as Perl and Ruby.

### Lua 功能强大(但简单)

A fundamental concept in the design of Lua is to provide meta-mechanisms for implementing features, instead of providing a host of features directly in the language.
For example, although Lua is not a pure object-oriented language, it does provide meta-mechanisms for implementing classes and inheritance.
Lua's meta-mechanisms bring an economy of concepts and keep the language small, while allowing the semantics to be extended in unconventional ways.

### Lua 是小的

Adding Lua to an application does not bloat it.
The tarball for Lua 5.4.3, which contains source code and documentation, takes 350K compressed and 1.3M uncompressed.
The source contains around 29000 lines of C.
Under 64-bit Linux, the Lua interpreter built with all standard Lua libraries takes 278K and the Lua library takes 466K.

### Lua 开源的

Lua is free open-source software, distributed under a very liberal license (the well-known MIT license).
It may be used for any purpose, including commercial purposes, at absolutely no cost.
Just download it and use it.
