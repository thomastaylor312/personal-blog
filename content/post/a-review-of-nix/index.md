---
title: A Review of Nix
description: My deep dive into the world of Nix
date: 2024-12-09 10:00:00-0700
draft: false
image: images/nix-snowflake-colours.svg
categories:
    - Explorations
tags:
    - nix
    - explorations
---

This post is a summary of my learnings of Nix, its pros and cons, and a bonus section on using it as
a monorepo build tool. This came out of some investigations I did in context of a work project.

## What I did

For a long I have not been a huge fan of Nix. It often comes off as far too academic and clever for
its own good – a stereotype often confirmed by current power users and the community. Every time
I've tried to approach it, it has made me want to run away as fast as I can. The solution to this
was to dive head first into Nix and use it a whole bunch of ways. Below is a mostly complete list of
all the things I did to try Nix out

- Installed Nix using the [Determinate Systems
  installer](https://zero-to-nix.com/concepts/nix-installer/), both open source and their
  Determinate Nix version (which is also open source Nix but with some wrappers for enterprise-y
  convenience)
- [Created a
  flake](https://github.com/thomastaylor312/wadm/blob/aee23f12830355a87e62bc91f8617ec1c2e58ad4/flake.nix)
  for the wadm subproject of wasmCloud
- Added flakes for all of my personal projects, including a frontend UI, a Go project, and a Rust
  project
- Figured out [how to run some E2E
  tests](https://github.com/thomastaylor312/snas/blob/aa81f10c504473b060845979c685e9ea84f03e5a/flake.nix#L166)
  that needed dependent services running at the same time
- Messed around with Nix profiles
- Removed homebrew entirely on my personal and work MacBooks and created a [multi-layered
  config](https://github.com/thomastaylor312/home-manager) that enables different packages for
  personal and work using [home-manager](https://github.com/nix-community/home-manager)
- Set up/connected to various Nix binary caches
- Set up an s3 binary cache and a binary cache using [Attic](https://github.com/zhaofengli/attic),
  which I host over my personal [Tailscale](https://tailscale.com/) for caching things

Basically, I tried to do a bit of everything, so let's start off with the positives.

## The Positives

This section breaks down what I consider to be the biggest strengths of Nix based on my experience
with the above. I broke down each thing into a subheader, even if it was small, for ease of
navigation

### Home-manager and package management

One of the biggest surprises I had when trying things out was how much I loved
[home-manager](https://github.com/nix-community/home-manager). It is far and above the best
experience for both a) installing software and b) managing a home directory. Other solutions like
chezmoi were something I could never get behind because it required specific formats/layouts and was
just complicated. Getting started with home manager was easy with their init tool and on another
machine it was a simple git clone followed by an init. Honestly, you probably don’t even need to
know much about Nix to see what it is I’m doing with my [home-manager
config](https://github.com/thomastaylor312/home-manager). 

After using home-manager, I actually think it is one of the best entrypoints into the Nix ecosystem
as it shows off what Nix is actually very good at: package/tool management layered with
configuration.

### Dev Environment

I’m going to come right out and say it: Unless you’re talking about remote dev environments (and
even then that might be better with Nix too), Flakes (or nix-shell) with `devShells` are hands down
the best way to do dev environments. Definitely better than devcontainers, and often less heavy due
to the caching and large package set that is Nixpkgs. Between [direnv](https://direnv.net/) and a
flake (plus the direnv extension for VSCode), it was very easy to get the exact environment needed
for any project.

One downside to mention here because it isn’t big enough to be in the negatives section: there is
quite a bit of fragmentation in this space because you have normal flakes (honestly just fine IMO),
but you also have Flox, devenv, devbox, and more (at least in the Nix-based space).

### Language tooling

At least for my two most used languages (Go and Rust), but also for many others, the support for
those languages is quite robust. The tooling already knows how to split up and/or hash all of the
dependencies for better cachability and integrates in fairly smoothly. Rust has the
[crane](https://github.com/ipetkov/crane) library and Go has the nixpkgs `buildGoModule` function
that works really well ([gomod2nix](https://github.com/nix-community/gomod2nix) is a community
supported tool for it as well). Many of these tools are basically needed to make sure you can cache
dependencies and not rebuild everything.

### The idea itself

I wasn’t sure how else to describe this, but essentially, the absolute core part of Nix and what it
is trying to do is an extremely solid and helpful idea. It is a package manager, but I think it
sells itself short (which makes me sad that is what it often touts itself as). Boiled down to its
essence, Nix is essentially a set of inputs and outputs (and I don’t just mean that with Flakes,
which use the terms "inputs" and "outputs"). Every single thing, whether it be a file/string or it
is a whole project, can be passed around as a “dependency,” as it were, to anything else. This makes
it both powerful and flexible. To really see this in action, I highly recommend at least looking at
my aforementioned home-manager config (or trying one out yourself) as it is pretty basic. This is
when it really clicked for me as you could see how everything layered together

## The Negatives

This section breaks down what I consider to be the most relevant weaknesses of Nix based on my
experiences with it.

### Documentation and Examples

As advertised, Nix’s documentation is pretty terrible and/or disorganized. There are some bright
spots that I found. Home Manager was pretty good for getting started, but really lacked middle
ground examples (i.e. more complex than Hello World, but less complex than [enterprise level
code](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition)). Crane was also pretty
good at documenting its functions, but didn’t have a ton of other more complex examples either. In
fact, one of the most worrying things is how most examples were just above “Hello World” or were
overly complex, super layered, hundreds of lines of Nix nested among 10 subdirectories of other Nix
files. There was nothing in between. Maybe I was searching for the wrong thing since I’m new, but
this happened with everything I tried: home manager, a personal Rust project, wasmCloud, a personal
Go project, and Wadm. It seems like the community is very biased towards very complex things and I
couldn’t find anything that was along the lines of “I need something moderately flexible, but not
infinitely tweakable.” However, based on what I got to with my Home Manager config, I don’t think it
always has to be this way. So I am hopeful all Nix isn’t complex, but it sure has been hard to find
those middle of the road examples.

Overall, I think better docs would help Nix as a project be much more approachable, but if a team
wants to do anything with it, it needs to be heavily documented.

### The Language

Obviously, I’m still new to the language, so this can be taken with a grain of salt. The core
language itself is quite simple, but in practice, it is very hard to follow a lot of Nix code. This
differs from project to project, but when you have things like `inherit`, the `//` operator, and
imported modules that automatically merge, it quickly becomes very hard to follow what is accessible
where and how it works. That isn’t even beginning to factor in things like overlays and other flake
things. Then you throw in builtins and the nixpkgs functions, which are documented in single web
page walls of text (often in different places). When you do something fairly simple (like a basic
build along with a devShell), it is pretty easy to follow along. So much of this just feels like
“wizardry” or “being clever.” Is it a cool way of doing things for sure and I still really want Nix
to be a more popular/approachable thing, but it takes sooooo much mental overhead to set up builds
right and then to modify something. Unless you’re writing Nix all the time, you end up forgetting
some of the more obtuse incantations until all of a sudden you need to do it again. So really the
only solution here is very good documentation, both user docs and function level doc comments.

### Slowness and caching

Contrary to popular belief (at least what I've heard argued from Nix fans and what I see on the
internet), Nix is not very fast. Derivations (the way Nix figures out what the dependency tree is)
are, for all intents and purposes, done serially. Yes there’s caching but it sometimes
just...doesn’t work if some inputs change in the wrong way (especially with something that doesn’t
really matter to the code). I kept running into this in my own builds and I've seen it in other
projects too..

Tools like crane recommend a separate tool that does cargo workspace hacks and you need special
things for languages like Go (gomod2nix) as well to get the right hashes in place for things in
order to do caching properly. It is definitely possible to work around this, but I don’t think many
teams want to waste their precious dev time constantly figuring out how to update their build tool
to perfectly set things up so caching works properly. Just keep in mind that Nix is not a silver
bullet just because “it caches.”

### Testing

I noticed that End-to-end/integration tests are a bit wonky to set up and use. I’ve gotten away with
it [by using a `preCheck`
script](https://github.com/thomastaylor312/snas/blob/aa81f10c504473b060845979c685e9ea84f03e5a/flake.nix#L166),
but that feels slightly hacky. Everything online is like “just use a NixOS server for testing” so it
is hard to find prior art on what is the right thing here. FWIW, if I ever have a project that
deeply uses Nix, using NixOS servers wouldn’t be a bad idea, but that would be waaaaay down the line
for me personally. 

For unit tests, those run fairly easily but sometimes it is a bit odd. For example, when using
crane, if I have a repo that has multiple binaries, but uses some shared libraries (where most of
the tests are), you end up running the tests twice without splitting it up into a separate test
check (which then doesn’t necessarily run during build).

This isn’t a super huge point, but I did find it an annoying rough edge.

### The community

This is a minor, but could eventually become concerning, note about the state of the community. This
year, the Nix community had two fairly large blowups around changing their governance and pushing
out the creator of Nix (as well as a few others). I don’t have skin in the game that makes me want
to comment on the virtues of those issues, but after reading through some of the threads and various
analyses out there, I am concerned. There seems to be a distinct lack of solid leadership and things
generally seem to be a mess. That hasn’t seemed to completely impact the growth of the community or
usage, but it could become a problem if there isn’t leadership to help move the project forward.
This is even reflected in the attitude taken towards Flakes. You have these two camps of Flakes and
Anti-Flakes. Honestly, as a n00b to all of this, Flakes are just easier IMO. I understand there is
nuance, but the whole way it is handled is just odd.

## Summary

My really blunt TL;DR is Nix the tool is amazing, but Nix the language can still be painful to use
and I desperately would love for it to be better because I love the features it provides.

The longer version of that is that I actually came away quite impressed from my time with Nix. I am
planning on continuing use at a bare minimum of home-manager and dev shells via flakes. As a dev
environment tool and a package manager, it succeeds very well. As a build tool (especially for
complex projects) it half way fails. I mean that it does a really good job (probably better than
other tools I’ve seen) when it is set up but it can be a monster to manage. It says something that
Nix makes other complex build tools like Buck2 and Bazel look much more simple to understand in some
aspects. Overall, I recommend everyone at least look at it because it continues to grow and will be
very powerful in the future if it can fix some of its current issues.

## Appendix: Additional Resources

A list of extra resources I found useful:

- [Adding Binary Cache Servers | NixOS & Flakes
  Book](https://nixos-and-flakes.thiscute.world/nix-store/add-binary-cache-servers)
- [https://nixcloud.io/tour/](https://nixcloud.io/tour/)
- [https://github.com/numtide/build-go-cache/](https://github.com/numtide/build-go-cache/) - Go
  binary caching tool
- [Wombat’s Book of Nix](https://mhwombat.codeberg.page/nix-book/) - Probably one of the best guides
  to the language and flakes I’ve found

<!-- TODO: Put this in a callout box -->

{{< quote>}}
### Bonus Feature: Using Nix for a Monorepo

One of the reasons I did these explorations was evaluating Nix as a possible tool for monorepos. In
addition to Nix, I looked into tools like Bazel and Buck2. Obviously if all you want is a monorepo
tool, it _might_ just be easier to use something like Bazel and Buck2. For Nix, I don’t think there
is built in support for things like git LFS and what not that you would need for a giant monorepo.
However, Nix is much better at being able to do things like select the right dependencies for
caching. For example, with Buck2, I had to use their `reindeer` tool to get dependencies set up.
This requires a bunch of annoying patches/overrides and you have to maintain a separate dir with a
different `Cargo.toml` file. If you were to go down the route of mostly using Nix, I think it would
need to focus on the following things:

- Having a single flake does make it hard to install a specific version of a tool from that flake.
  This probably involves a whole bunch of other Nix that the flake would act as an entry point to.
  It could also be that you need a separate flake per project combined with the previous option
- You will need to make some heavily documented vanity wrappers around common operations (such as
  cross compiling, running e2e tests, and so on)
- Work around some of the caching/slowness issues by running your own cache using something like
Attic. It would probably be cheaper and easier until your usage of Nix is big enough that you'd want
to use something like Cachix enterprise or FlakeHub. You can then aggressively cache everything so
many of the team's local cache items (like dependencies) could be shared quite easily {{< /quote >}}
