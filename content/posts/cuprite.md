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

{{< rawhtml >}}<span style="font-size: xxx-large; font-family: serif; font-style: italic">W</span>{{< /rawhtml >}}hat do you get when you binge watch [DHH](https://dhh.dk/) talks[^1] and podcasts[^2] [^3], but are also in a low level programming mood from watching [Better Software Conference](https://www.youtube.com/@BetterSoftwareConference) talks and other sources like [Wookash](https://www.youtube.com/@WookashPodcast) and [Casey Muratori](https://caseymuratori.com/)'s [Computer Enhance](https://www.computerenhance.com/about) courses ?

Well... what I got is an urge to write a Ruby on Rails like - web framework - in C.

TL;DR - [Cuprite Github link](https://github.com/saftacatalinmihai/cuprite)

## Why

<!-- {{< rawhtml >}}
<div class="class">
<blockquote class="twitter-tweet" data-theme="dark"><p lang="en" dir="ltr">any sufficiently complicated web app i build in java contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of ruby on rails</p>&mdash; htmx.org / CEO of Rizz&#39;em w/the &#39;Tizm (same thing) (@htmx_org) <a href="https://twitter.com/htmx_org/status/1950348285786661031?ref_src=twsrc%5Etfw">July 30, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
{{< /rawhtml >}} -->

Ruby on Rails is already known to be the best web framework for developer happiness.

Rails emphasizes the use of other well-known software engineering patterns and paradigms, including convention over configuration (CoC), don't repeat yourself (DRY), and the active record pattern.

I don't want to reinvent the wheel for developer experience given that there is already a good model, but wanted to see how close I could get to the Ruby version using the C programming language.

This project does not mean to say there is anything wrong with using Rails in Ruby - it only tries to see if the same experience (or close enough) can be replicated in C which has the performance advantage.

It seems to me that the software industry had evolved in terms of frameworks and libraries, but not in C, in other higher level languages - as it is seen to be an old programming language... and it has the perceived disadvantage of being manual memory managed - which can cause memory leaks. All of this can be easily managed today with tools that check for memory leaks, but also using some simple programming techniques, the issue can be managed quite easily. I think it is time to go back to C but take the learnings from the other languages.

My benchmark for the proof of concept is to replicate as close as possible the code from the [Rails getting started page](https://guides.rubyonrails.org/getting_started.html).

## How

### Web framework library

I did not want to start completely from scratch and build an http server code in C, so I looked around for C http server libraries.

I landed on [facil.io](https://facil.io/) which provides high performance TCP/IP network services by using an evented design that was tested to provide an easy solution to [the C10K problem](https://www.kegel.com/c10k.html).

I later learned is also used for a Rails server backend implementation (just one of the coincidences that links this project with Ruby on Rails - happened multiple times during this project)

Looking at the Rails Getting Started page - the first step (jumping over the project setup) is to define a model, which you can do in C by just defining a struct.

### Model / ActiveRecord

In Rails you do this by generating the model from the CLI with `bin/rails generate model Product name:string`
The model in code can leave out the fields, as they are automatically retrieved from the DB:

{{< rawhtml >}}
<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
{{< /rawhtml >}}

{{< rawhtml >}}
<div>
{{< /rawhtml >}}

```ruby
class Product < ApplicationRecord
end
```

> You might be surprised that there is no code in this class. How does Rails know what defines this model?  {{< rawhtml >}} </br> {{< /rawhtml >}}
> When the Product model is used, Rails will query the database table for the column names and types and automatically generate code for these attributes. Rails saves us from writing this boilerplate code and instead takes care of it for us behind the scenes so we can focus on our application logic instead.[^4]

{{< rawhtml >}}
</div><div>
{{< /rawhtml >}}

```C
typedef struct {
   id: int,
   name: char*,
} Product;
```

{{< rawhtml >}}
</div></div>
{{< /rawhtml >}}

The Rails way is a bit too magical for me (at list for the first MVP, could look into this later), so I decided to require the programmer to define a struct for the model with all the fields.

```bash
make generate_model model=Product
```

Having the model struct, you can run the model code-generation script that generates all the `ActiveRecord` functions - the database access code for interacting with the model in the database. For example: `product_new`, `product_find`, `product_all`

After the model code generation, you run run the db migration to create the model table in the database (in that order):


and

```sh
make migrate
```

The first time you run a make command, it also downloads and build the `facio.io` library

The `generate_model` script will generate C code that takes the given model struct and create all the db access functions - similar to the ActiveRecord methods in Rails.

For comparison:

{{< rawhtml >}}
<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
{{< /rawhtml >}}

{{< rawhtml >}}
<div>
{{< /rawhtml >}}

```ruby
product = Product.new(name: "T-Shirt")
```

Nice and clean.

{{< rawhtml >}}
</div><div>
{{< /rawhtml >}}

```C
Product* p = product_new();
```

Notice the `product_new()` function does not take parameters. I left this for a later implementation. For now you can set fields on the product struct and then call the save function with it:

```C
p->name = strdup("T-Shirt"); 
int id = product_save(p);
```

{{< rawhtml >}}
</div></div>
{{< /rawhtml >}}

We need `strdup` because the product needs to own the memory for the `name` string so it can free it. If we did just `p->name = "T-Shirt"`, the "T-Shirt" string is allocated on the stack so it can't be safely freed later with `product_free(p)`. This is one of the things you need to keep track of when using C - but for this case you quite easily find the issue because the app will break on the first `free` call when `strdup` is not used.

We can start the server with
```sh
make start
```

<!-- To do this I wanted to replicate the experience of following the [ruby getting started tutorial](https://guides.rubyonrails.org/getting_started.html) and try to keep the code as similar as possible with the ruby one. -->

TODO:

- Acknowledge an already existing cuprite project in Ruby
- [x] Facil.IO - already used as a ruby server
- talk about the coincidence from the 2 above of finding links to ruby in choices made, before knowing about them.
- address #no-build.
- c is fast - perf test ab / hey show
- i use arch, omarchi , neovim , btw.
- How to use:
  - model generation (using ruby)
  - run migrations
  - run app
  - htmx (preload, view transitions, mustache)

[^1]: [Rails World 2023 Opening Keynote](https://youtu.be/iqXjGiQ_D-A?si=Wk-lCifYGY9pqSp0)
[^2]: [Lex Fridman Podcast - DHH: Future of Programming, AI, Ruby on Rails, Productivity & Parenting](https://www.youtube.com/watch?v=vagyIcmIGOQ)
[^3]: [ThePrimeagen - DHH IS RIGHT ABOUT EVERYTHING (Again)?](https://www.youtube.com/watch?v=EIBxRMH4bvs)
[^4]: [Active Record Model Basics](https://guides.rubyonrails.org/getting_started.html#active-record-model-basics:~:text=You%20might%20be,application%20logic%20instead.)