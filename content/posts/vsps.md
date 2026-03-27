---
title: "Visual Stream Processing System"
subtitle: "Drawing Programs"
date: 2024-07-10T12:44:58Z
tags:
- scala
- streaming
- visual_programming
---

## Add
{{< figure src="/vsps_add.gif" title="" alt="add">}}
There is a new [tldraw](https://www.tldraw.com/) tool called `Stream Component` (left-most icon: `</>` in the toolbar, shortcut key `S`). You can click or draw a rectangle to add a stream component to the canvas.

The component is not initialized at first - you need to select the component type from the drop-down menu. This UI will be improved to allow more discoverability, show descriptions of components, etc.

After selecting the component type, it will be initialized (started in the backend) and begin running immediately, waiting for inputs on its input port(s). There is no separate step for starting the component; it is always running.  

I want the designer to have the feeling of working with regular drawings - so normal actions like moving, resizing, deleting, copying, pasting, undo, redo, etc., should work as expected.

## Connect
{{< figure src="/vsps_connect.gif" title="" alt="connect">}}
You can connect two components by adding arrows from an output port to an input port using regular tldraw arrows.

### Type check on connect
{{< figure src="/vsps_typecheck.gif" title="" alt="connect">}}
The backend where the streaming components are started is a Scala program[^1]. This means that the connections between components can be type-checked. If the connection can be made, the arrow turns green; otherwise, it turns red.

This can be an advantage over other visual programming systems that are usually dynamically typed. 

The system still allows the connection to be made, but you can expect a runtime error if the types are not compatible.
I chose to allow a more dynamic feel and enable the designer to do whatever they want (allowing rough scribbles or [scrappy fiddles](https://www.todepond.com/wikiblogarden/scrappy-fiddles/sharing/normalising/live/)), 
even if that means entering a temporary state where the types don't match. 
It is up to the designer to fix this later. 

## Edit
{{< figure src="/vsps_edit.gif" title="" alt="edit">}}
The component's code can be edited in any text editor.
I want to make it easy for the programmer to open the code directly from the canvas, so I added a context button that opens the code in a default editor.

Currently, changing the code requires a restart of the backend - that is not ideal and will be improved. 
However, the restart only takes a few seconds, so it is not a significant issue for now.

## Logs
{{< figure src="https://s10.gifyu.com/images/StqmJ.gif" title="" alt="logs">}}
Instead of adding a custom view for showing logs, I decided to use an existing solution (ELK stack - ElasticSearch, Logstash, Kibana) and embed it in the tool.
After all, tldraw allows embedding of any web content with iframes.

## Proof of concept: Fibonacci
{{< figure src="https://s10.gifyu.com/images/StqmP.gif" title="" alt="fibonacci">}}
This is a simple example of a Fibonacci stream component. It generates the Fibonacci sequence and prints it.

## What's the big idea, anyway?
{{< figure src="https://imageproxy.ifunny.co/crop:x-20,resize:640x,quality:90x75/images/8a0902c12ee0ae61a9e61f035015ca2419b7a50f1d8d8eab34d735c9a64f58cf_1.jpg" title="" alt="draw-an-owl">}}
This image depicts how much of software is built today. Someone - architect, designer, senior dev - starts by drawing some diagrams.
These are representations of what the code for an application should do, not the actual code.

Then developers take those diagrams and start implementing code. Most of the time, the diagrams become outdated as new information comes along.

###### What if the diagram can start as a sketch, but can be gradually "filled out" with the actual implemented code?

And at the same time, the diagram can be run as a program.

This is the idea behind Visual Stream Processing System (VSPS).

I'm trying to capture a feeling I have when developing streaming applications. 
I usually start with a high-level component graph (using [Pekko Graph DSL](https://pekko.apache.org/docs/pekko/current///stream/stream-graphs.html)). 
The components are just mocks - only the types are defined, but the graph actually runs. 
Then I start implementing the components one by one, slowly filling out the graph with actual code, and refactoring the shape as I gather new information about what I actually need to build.
The implementation also guides the graph design by surfacing some paths I haven't thought of before - especially error cases, edge cases, etc.
But I always have a running system, even if it is not fully implemented.

This is a highly experimental project that I've been working on in my (very limited) spare time, but I think it has potential.

If you are interested in this project, please reach out to me. I would love to hear your thoughts on it.

[^1]: Pekko [Streams](https://pekko.apache.org/docs/pekko/current/stream/) to be precise