---
title: "Visual Stream Processing System"
subtitle: "Drawing Programs"
date: 2024-07-06T08:44:58Z
draft: true
tags:
- scala
- streaming
- visual_programming
---

## Introduction
{{< figure src="https://imageproxy.ifunny.co/crop:x-20,resize:640x,quality:90x75/images/8a0902c12ee0ae61a9e61f035015ca2419b7a50f1d8d8eab34d735c9a64f58cf_1.jpg" title="" alt="draw-an-owl">}}
This is how much of software is built today. Someone - architect, designer, senior dev ... starts by drawing some diagrams. 
Those are representations of what the code for some application should do, it is not the actual code.

Then devs take those diagram and start implementing code. Most of the time, the diagrams are outdated as new information comes along.

###### What if the diagram can start out as a sketch, but can be "filled out" with the actual code as it is implemented.

And at the same time the diagram can be run as a program.

This is the idea behind Visual Stream Processing System (VSPS).

This is a highly experimental project that I've been working on in my (very limited) spare time, but I think it has potential.

## Add
{{< figure src="/vsps_add.gif" title="" alt="add">}}
{{< admonition type=note open=false title="" >}}
There is a new tldraw Tool called `Stream Component` (S in the toolbar). After selecting the tool will allow you to create a new stream component.

The component is not initialised at first - you need to select the component type by selecting from the drop-down. This UI will have to be improved to allow more discoverability, show description of components etc.

{{< /admonition >}}

## Connect
{{< figure src="/vsps_connect.gif" title="" alt="connect">}}

### Type check on connect
