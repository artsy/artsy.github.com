---
layout: epic
title: "An Intro to Palette, Artsy's Design System"
date: "2019-04-30"
author: justin
categories: [culture, process, tooling, design]
series: Design Systems at Artsy
---

On May 3rd, 2018 [Luc]() made the first commit to [Palette](). The intent at the time was create a way to easily
share styles between [Reaction](), our React web components, and [Emission](), our React Native components. As
understanding of our needs evolved, Palette began it's transition into a full blown design system. In this post I'm
going to tell you the story of Palette and how we use it to ship products faster with tighter collaboration between
design and engineering.

<!-- more -->

## Design Systems, A Primer

Before digging into Palette itself, I want to take some time to talk about what a design system is. A design
system, as I see it, is a collection of guidelines, examples, and UI implementations used to build digital products
that cater to a specific theme or brand identity. It's a toolkit of design _and_ engineering resources that makes
building products for you company (that feel like your brand) easier. The term itself is most often encountered in
the design world, but I really want more engineers to understand both the technical and cultural merits that design
systems have.

Mozilla has a pretty good [glossary](https://mozilla.github.io/styleguide/resources/glossary.html) of terms related
to design systems world. I disagree with some of the terms in that I don't see the scope of design systems as
limited to the web, but it'll help you understand conversations happening in the community.

## Approachable documentation

Design systems are a collaboration between design and engineering. If either parts of your team can't contribute as
they need then the whole system suffers. It's critical to keep this in mind when building infrastructure for a
design system.

## The Tools Behind the Technology

At its heart, Palette is a set of brand specific but product agnostic components that engineers can use to rapidly
build products that _feel_ like Artsy. I think it's important to re-emphasis the fact that none of these components
are product specific. There's a button, but not a purchase button. Palette tries to make as few assumptions about
the systems that consume it so it can have the maximum utility across all of our digital products.

## Freedom through constraints

## Other topics...

- styled-components
- styled-systems

- responsive props allow for easy responsiveness
- make it easy to do the right thing with constraints
- document all the interfaces
