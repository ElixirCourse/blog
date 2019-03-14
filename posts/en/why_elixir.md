---
title_image_path: why_elixir.jpg
created_at: 2019-03-14T16:19:23
category: Програма
author: Ivan, Jordan
tags:
  - elixir
  - introduction
  - erlang
  - beam
  - functional programming
---

# Why Learn And Use Elixir?

This blog post is meant to accompany the first lecture<sup>[1](http://gitpitch.com/ElixirCourse/presentations-2019/?p=welcome)</sup> of our course _Functional Programming with Elixir_,
that takes place in FMI (Faculty Of Mathematcis And Informathics of The Sofia University) during the summer semester of 2018/2019.
We will discuss why one would want to learn and use Elixir.
But first, a bit of history:

### Erlang

Erlang is a functional language, developed in the mid-80s by Ericsson with the intent to build telecom software.
Although, the motivations behind its creation were primarily driven by the field&#39;s specific requirements, Erlang can be applied to many other needs.
One of least cited reasons<sup>[2](https://www.erlang-factory.com/upload/presentations/416/MikeWilliams.pdf)</sup> behind its development was the need to replace the highly specialized hardware and software used at the time (including the operating system).
Erlang does not explicitly support the programming requirements of a telephone or otherwise telecom venture, nor does it pose any specialized hardware or OS requirements.
Instead, the language offers solutions to the technical needs and problems encountered while developing this type of software:

- concurrency;
- scalability;
- distributability;
- error tolerance;
- updating without service interruption;
- quick query response even when under high stress;

All of this happened at a time, when internet was not yet widespread and most programs worked on a single computer with no connection.
This type of software does not require any distributability or horizontal scalability.
At the time, personal computers did not have 2-, 4-, or 16-core processors – the need for the software to automatically accelerate with addition of more processors/cores was not yet present. This directly impacted the design and purpose of the programming languages developed at this time or those earlier ones they were based on.

Even if you never use Erlang, its ideas and approach<sup>[3](https://ferd.ca/the-zen-of-erlang.html)</sup> to solving a specific type of problems is easily applicable when using other languages.

### Where does Erlang stand today?

All of the above sounds good, but how many of us write telecom software or, in fact, any software with such requirements?
Well, a lot of us, actually.
The requirements of today&#39;s web applications overlap completely with the concerns addressed by Erlang&#39;s creators at the time of its development.
Having a language and a platform based on meeting those specifics without the need to load external libraries to implement the missing functionality of the language is a huge advantage.

![Ad-hoc Erlang implementation](https://raw.githubusercontent.com/ElixirCourse/blog/master/assets/erlang_ad_hoc_implement.png)

If you want to know more about the guarantees provided by BEAM<sup>[4](http://www.erlang-factory.com/upload/presentations/708/HitchhikersTouroftheBEAM.pdf)</sup> – the virtual machine in which Erlang runs, you would do best to watch [this video](https://www.youtube.com/watch?v=5SbWapbXhKo).

### Elixir

Why do talk so much about Erlang in this article named &quot;Why Learn And Use Elixir?&quot;
![Best part of Elixir tweet](https://raw.githubusercontent.com/ElixirCourse/blog/master/assets/elixir_most_important_part.png)



Elixir is a functional language. Its development started in 2011, with version 1.0 going live in 2014.

Elixir runs on Beam.
It can use everything that was written and everything that could be written in Erlang.
This allows us to combine a new and modern language with libraries that stood the test of time.
Elixir&#39;s Erlang basis does not come with any additional hassle or the need to use cryptic syntax or type transformations.
Let&#39;s compare the different ways Elixir and Erlang call a function from a given module:

Elixir:

```elixir
DateTime.utc_now()
```

Erlang:

```erlang
:crypto.strong_rand_bytes(64)
```

Elixir modules always start with an upper case, while Erlang modules start with : and a lower case.

Let&#39;s look at just some of the things WE like about Elixir/Erlang:

- Functional Language – immutable and persistent data structures, pattern matching, higher order functions, function compositions
- Balance between pure and side-effect functions. At this point, some of the sworn Haskell fans might disagree. But some things have to sacrificed at the altar of productivity.
- Language-level processes – it may sound dull, but it&#39;s the basis for Elixir&#39;s famous concurrency and distributability
- OTP (Open Telecom Platform) – this is a platform that is no longer used solely for telecom solutions, but also for software that has similar requirements, which, as mentioned earlier in the article, a lot of today&#39;s web applications do. During this course, we will explore some of the libraries that OTP provides. For now, you can look at additional information [here](https://learnyousomeerlang.com/what-is-otp#its-the-open-telecom-platform).

It&#39;s wrong to think of Elixir as a competitor that will tank Erlang. On the contrary, Elixir is one of the best things to happen to it. Erlang, as the basis, has seen massive development due to the interest in Elixir and the influx of new people to the community.

But since Elixir emerged and we&#39;re running a course about it and not Erlang, it must bring something new to the table:

- Elixir is Erlang. Everything advantage of Erlang is an advantage of Elixir.
- Elixir is more.
- Better tools than Erlang.
- Good documentation. The language&#39;s creator claims that documentation errors should be handled as a program error.
- There are experienced, responsive and intelligent people behind the language and its ecosystem.
- Convenient and visually pleasing syntax. Erlang has a minimalistic syntax, even too much so. This could sometimes lead to boilerplate and repetitive code. The lack of macros exacerbates this problem.
- Powerful macros. As opposed to the ones in C/C++, macros in Elixir do not work with strings, but with AST (Abstract Syntax Tree). Macros in Elixir are mainly influenced by those in Lisp and Clojure.
- Polymorphism through protocols.
- Proper libraries. A good number of libraries have been written in Erlang and have been used long enough to make a safe claim that they&#39;re stable and well tested. But there are also a good number of libraries written relatively recently in Elixir. This gives the advantage of avoiding errors present in older libraries of other languages. Good examples would be [Plug](https://hexdocs.pm/plug/readme.html), [Ecto](https://hexdocs.pm/ecto/Ecto.html), [Phoenix](http://phoenixframework.org), [GenStage](https://github.com/elixir-lang/gen_stage) and many others.
- Pipe operator `|>`. With its help, code is unquestionably more pleasing, convenient, readable, as well as easy to change. This provides the not so obvious advantage of consistency in the standard library of the language. In order to make using `|>` easier, functions take the data they&#39;re working on as their first argument.

Let&#39;s look at an example of the difference between a code that uses `|>` and code that does not.

The goal is to execute a number of transformations on a list of numbers.

One way to write this out is:

```elixir
data = [1,2,3,4,5]
data_squared = Enum.map(data, fn n -> n*n end)
data_filtered = Enum.filter(data_squared, fn n -> n >10 end)
sum = Enum.reduce(data_filtered, 1, &(&1+&2))
```

What if we don&#39;t want to use so many temporary variables? We can implement the functions:

```elixir
Enum.reduce(Enum.filter(Enum.map([1,2,3,4,5], fn n -> n*n end),fn n ->n > 10 end), 1, &(&1 + &2))
```

In this example, readability is near-zero and adding further transformations is hardly convenient.

This is where `|>` comes in handy. It presents the result on its left side as a first argument in the expression on its right. Nothing more, nothing less, but just enough to allow us to write code in the following manner:

```elixir
result =
  [1, 2, 3, 4, 5]
  |> Enum.map(fn n -> n * n end)
  |> Enum.filter(fn n -> n > 10 end)
  |> Enum.reduce(1, &(&1 + &2))
```

### What about the downsides of Erlang/Elixir?

Just so we&#39;re not blindly spewing superlatives about the language, it would be appropriate if we mention some of its disadvantages. Some of them are in the below list, just because they&#39;re unseemly at first glance, but do carry value that starts to show over time and we will explore why.

- Lack of libraries found in many other languages. Since Elixir is a fairly new language, and Erlang does not benefit from the popularity of languages like Java, Python or Ruby, sometimes you won&#39;t be able to locate something you intuitively expect to find. Sometimes, you&#39;ll only be able to find an Erlang library that could do the job, but this will require learning some Erlang.
- The delusion, brought on by people not truly familiar with the language. A common complaint is that the syntax concerning concurrency is not as simple and minimalistic as in Go. The reason for this is that Elixir&#39;s concurrency is meant to solve problems related to high availability, while in Go it is simply used for concurrency&#39;s sake.
- At first glance, it&#39;s not obvious how to locate certain functions. If you want to return the length of a list, would you find the function in the `List` module? You&#39;ll actually discover there is no `List.size`, nor `List.length`. The length function is called with `Enum.count`, which runs `:erlang.length` (accessible as `Kernel.length` or simply `length`) under the hood. However, using `Enum.count` we can find the number of elements of everything the `Enumerable` protocol implements.

The library problem is one that every language suffers at this stage of its development. To clarify, we are not talking about important libraries like HTTP server/client, database, web framework, JSON decoders, etc; libraries like that are available, and they&#39;re quite good, too. We&#39;re talking about small things that aren&#39;t really a problem of the language or the platform and is fixable, albeit slowly over time. During this FMI Elixir course we will take into account contributions to open source libraries and credit will be given accordingly. Even more so for newly created libraries.

### Sources:

[1] Tsvetinov, Nikolay (Meddle). Functional Programming With Elixir. Gitpich, 18.02.2019. Available from: https://gitpitch.com/ElixirCourse/presentations-2019/?p=welcome [cited 22.02.2019].

[2] Williams, Mike. The True story about why we invented Erlang and A few things you don’t want to tell your Manager. Erlang Factory, 26.02.2011. Available from: https://www.erlang-factory.com/upload/presentations/416/MikeWilliams.pdf [cited 22.02.2019].

[3] Hebert, Fred. The zen of Erlang. Ferd, 08.02.2016. Available from: https://ferd.ca/the-zen-of-erlang.html [cited 22.02.2019].

[4] Virding, Robert. Hitchhiker’s Tour of the BEAM. Erlang Factory, 2012. Available from: http://www.erlang-factory.com/upload/presentations/708/HitchhikersTouroftheBEAM.pdf [cited 22.02.2019].

Jurić, Saša. ElixirDaze 2017- Solid Ground by Saša Juric. YouTube, 16.03.2017. Availalble from: https://www.youtube.com/watch?v=5SbWapbXhKo [cited 22.02.2019].

[Official site of Erlang](http://www.erlang.org/)

[Official site of Elixir](https://elixir-lang.org/)
