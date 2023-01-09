---
title: "Designing idempotent Streaming system"
date: 2023-01-07T13:39:16+02:00
draft: true
tags:
- scala
- cats
- akka
- streaming
- idempotency
---

When working with Akka Streams, you might find yourself in a situation where you need to bypass a stage in the graph. 

This is a good pattern if we want the whole stream to be idempotent. 
If we get the same message again, we want the stream to be able to process it - but only the stages that were not already processed.


We usually save the extra information after each processing stage. 
At the start of the stream we can get the state for a message


