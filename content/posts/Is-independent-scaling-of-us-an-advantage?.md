---
title: Microservice scaling advantage?
subtitle: Is independent scaling of microservices an advantage ?
date: 2022-06-12
lastmod: 2022-06-12
draft: true
tags:
  - draft-post
  - microservice
---

**"Just split it into microservices and scale them independently."**

How many times have you heard this phrase in architecture meetings? It's become the default answer to scaling challenges, repeated with such conviction that questioning it feels almost heretical.

But what if this widely accepted wisdom is fundamentally flawed?

## The Scaling Scenario: A Common Dilemma

Picture this: You have a service that runs a number of steps for its business process. One of those steps consumes high CPU—something to do with cryptographic functions, let's say. You run this service on 3 nodes (you need 3 at least for high availability, no less), and you get concerned that with increasing traffic to this service, the CPU starts to go above 89-90%.

The alarm bells are ringing. Performance is degrading. Your team is under pressure to find a solution.

## The Microservice Promise: Independent Scaling

What are your options?

One answer comes from the much-praised solution of using microservices and scaling them independently. The logic seems sound:

1. **Identify the bottleneck**: The cryptographic step
2. **Extract it**: Create a dedicated microservice
3. **Scale independently**: Add more instances of just that service
4. **Celebrate**: You've solved the problem with surgical precision

This approach has become so popular that it's often the first—and sometimes only—solution considered.

## The Uncomfortable Questions

But before we embrace this solution, let's ask some uncomfortable questions:

- What happens to network latency when you split a process?
- How do you handle distributed transactions?
- What's the operational overhead of managing multiple services?
- Are you solving a scaling problem or creating a complexity problem?

The microservice scaling advantage might not be as advantageous as it first appears...
