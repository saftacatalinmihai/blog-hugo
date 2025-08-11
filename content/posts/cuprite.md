---
title: "Cuprite"
subtitle: "Experimental Ruby on Rails inspired web framework in C"
date: 2025-08-01
draft: true
tags:
  - C
  - ruby
  - rails
  - htmx
---

What do you get when you binge watch DHH talks[^1] and podcasts[^2] [^3], but are also in a low level programming mood from watching @Better Software Conference talks ?

What I got is an experiment to write a Ruby on Rails like web framework in C.
TL;DR - [Github link](https://github.com/saftacatalinmihai/cuprite)

# Why?
<!-- 
{{< rawhtml >}}
<div class="class">
<blockquote class="twitter-tweet" data-theme="dark"><p lang="en" dir="ltr">any sufficiently complicated web app i build in java contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of ruby on rails</p>&mdash; htmx.org / CEO of Rizz&#39;em w/the &#39;Tizm (same thing) (@htmx_org) <a href="https://twitter.com/htmx_org/status/1950348285786661031?ref_src=twsrc%5Etfw">July 30, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
{{< /rawhtml >}} -->

Ruby on Rails is already known to be the best web framework for developer happiness.
Rails emphasizes the use of other well-known software engineering patterns and paradigms, including convention over configuration (CoC), don't repeat yourself (DRY), and the active record pattern.
I don't want to reinvent the wheel for developer experience given that there is already a good model, but wanted to see how close I could get to the Ruby version using the C programming language.

My benchmark for the proof of concept is to replicate as close as possible the code from the [Rails getting started page](https://guides.rubyonrails.org/getting_started.html).

# How?
I did not want to start completely from scratch and also build the http server code, so I looked around for C http server libraries. 
Landed on facil.io - which I later learned is also used for a Rails server backend implementation ( just one of the coincidences that happened during this project )

The first step is to define a model, which you can do in C by just defining a struct.

'''c
typedef struct {
   id: int,
   name: char*,
} Product;
'''

TODO:

- Acknowledge an already existing cuprite project in Ruby
- Facil.IO - alreay used as a ruby server 
- talk about the coincidence from the 2 above of finding links to ruby in choices made, before knowing about them.
- adress #no-build. 
- c is fast - perf test ab show
- i use arch, omarchi , neovim , btw.
- How to use:
  - model generation (using ruby)
  - run migrations
  - run app
  - htmx (preload, view transitions, mustache)

[^1]: [Rails World 2023 Opening Keynote](https://youtu.be/iqXjGiQ_D-A?si=Wk-lCifYGY9pqSp0)
[^2]: [Lex Fridman Podcast - DHH: Future of Programming, AI, Ruby on Rails, Productivity & Parenting](https://www.youtube.com/watch?v=vagyIcmIGOQ)
[^3]: [ThePrimeagen - DHH IS RIGHT ABOUT EVERYTHING (Again)?](https://www.youtube.com/watch?v=EIBxRMH4bvs)