---
title: "Visual Stream Processing System"
subtitle: "Drawing Programs"
date: 2024-07-10T18:44:58Z
tags:
- scala
- streaming
- visual_programming
---

## Add
{{< figure src="/vsps_add.gif" title="" alt="add">}}
{{< admonition type=note open=false title="" >}}
There is a new tldraw Tool called `Stream Component` (left-most icon: `</>` in the toolbar, short-key `S`). You can then click or draw a rectangle that will add a stream component to the canvas.

The component is not initialised at first - you need to select the component type by selecting from the drop-down. This UI will have to be improved to allow more discoverability, show description of components etc.

After selecting the component type, it will be initialised (started in the backend) and already start running - waiting for inputs on it's input(s) port(s). There is no separate step for starting the component, it is always running.  

I want the designer to have the feeling of working with just regular drawings - so normal things like moving, resizing, deleting, copying, pasting, undo, redo, etc. should work as expected.

{{< /admonition >}}

## Connect
{{< figure src="/vsps_connect.gif" title="" alt="connect">}}
{{< admonition type=note open=false title="" >}}
You can connect two components by adding arrows from an output port to an input port. Just regular tldraw arrows.
{{< /admonition >}}

### Type check on connect
{{< figure src="/vsps_typecheck.gif" title="" alt="connect">}}
{{< admonition type=note open=false title="" >}}
The backend that the streaming components are started in is a Scala program. This means that the connections between components can be type-checked. If the connection can be made, the arrow turns green, otherwise it turns red.

This can be an advantage over other visual programming systems that are usually dynamically typed. 

The system still allows the connection to me made, but you can expect a runtime error if the types are not compatible.
I chose to allow a more dynamical feel and allow the designer to do whatever they want (allow rough scribbles or scrappy fiddles), 
even if that means going into a temporary state where the types don't match. 
It is up to the designer to fix this later. 
{{< /admonition >}}

## Edit
{{< figure src="/vsps_edit.gif" title="" alt="edit">}}
{{< admonition type=note open=false title="" >}}
The component's code can be edited in any text editor.
I want to make it easy for the programmer to open the code directly from the canvas.

Currently, changing the code requires a restart of the backend - that is not ideal and will be improved. 
However, the restart only takes a few seconds, so it is not a big deal for now.
{{< /admonition >}}

## Logs
{{< figure src="/vsps_logs.gif" title="" alt="logs">}}
{{< admonition type=note open=false title="" >}}
Instead of adding some custom view for showing logs - why not just use an existing solution (ELK stack - ElasticSearch, Logstash, Kibana) and embed it in the tool.
After all tldraw allows embedding of any web content with iframes.
{{< /admonition >}}

## Proof of concept: Fibonacci
{{< figure src="/vsps_fibonacci.gif" title="" alt="fibonacci">}}
{{< admonition type=note open=false title="" >}}
This is a simple example of a Fibonacci stream component. It generates the Fibonacci sequence and prints it.
{{< /admonition >}}

## What's the big idea, anyway?
{{< figure src="https://imageproxy.ifunny.co/crop:x-20,resize:640x,quality:90x75/images/8a0902c12ee0ae61a9e61f035015ca2419b7a50f1d8d8eab34d735c9a64f58cf_1.jpg" title="" alt="draw-an-owl">}}
This image depicts how much of software is built today. Someone - architect, designer, senior dev ... starts by drawing some diagrams.
Those are representations of what the code for some application should do, it is not the actual code.

Then devs take those diagram and start implementing code. Most of the time, the diagrams are outdated as new information comes along.

###### What if the diagram can start out as a sketch, but can be gradually "filled out" with the actual code in the implemented.

And at the same time the diagram can be run as a program.

This is the idea behind Visual Stream Processing System (VSPS).

This is a highly experimental project that I've been working on in my (very limited) spare time, but I think it has potential.

If you are interested in this project, please reach out to me. I would love to hear your thoughts on it.
