---
title: "Software Architecture Horror Story"
subtitle: "Why modern software is slow and unreliable" 
date: 2025-10-09T19:35:31+03:00
---

This is a tale of software architecture decision-making in large companies.

### The setup

As part of my job as a software architect, I had to design a solution which involved the integration between 2 systems: let's call them `A` and `B`. They were part of a new instant payment processing flow that was beeing developed. System `B` had a legacy integration mechanism that was no longer supported at the architectural level. A proxy system - call it `P` was implemented to facilitate the translation between the legacy interface into a more modern - JSON over HTTP - API.

{{< mermaid >}}
sequenceDiagram
    participant A
    participant P
    participant B

    A ->> P: API Request
    P ->> B: Legacy Request
    B ->> B: Process
    B ->> P: Legacy Response
    P ->> A: API Response

{{< /mermaid >}}

### The problem

Both `A` and `B` were hosted in the same data center ( `DC-1` ), but the team that implemented `P` used a different data center (`DC-2`) - a more cloud-like environment - to implement their automations for deploying, alerts, metrics etc. 

The `P` team wanted to keep their automation setup and asked if we would be okay with them deploying `P` in `DC-2`,  even though A and B were still in DC-1.

This suggestion perplexed me from the start. The distance between the 2 data centers was considerable (30 ms ping time). I engaged with them and explained - politely - that this setup would introduce unnecessary latency and create a dependency on another data center for the system’s functionality. It would be much better if they deployed the proxy app `P` closer to the `B` system or even on the `B` system server. 

This to me seems obvious. The idea of adding a dependency on a whole other data center - that's hundreds of kilometers away - just to transform some data, seems preposterous. It goes against everything I understand as an software architect or engineer in general.

Unfortunately, the discussion didn’t go well. Things escalated, and I had to schedule a full **Architecture Decision Meeting**, inviting around 10–15 architects, tech leads, and even the CIO.

I thought surely, presenting the two options—(1) `A` and `B` in `DC-1` with `P` in `DC-2` vs. (2) all systems in `DC-1` — would be convincing to any architect, especially infrastructure architects. The setup with all apps in the same DC would have near-zero latency, while the two-DC setup would introduce a minimum of 120 ms latency due to two synchronous requests: `A` to `P` and `P` to `B`, each round trip requiring 60 ms (30 ms for the request, 30 ms for the response). This latency would be added to `B`’s processing time, which was very fast (around 5–10 ms). So, total latency would be an order of magnitude higher than necessary.

### The decision 

**How did the meeting go, you ask?**

This is where things got weird—and baffling. Top management had ulterior motives for pushing the use of `DC-2`, and the architects knew it. Because of this, no one (except me) raised the concerns as strongly as they should have. Their reasoning was that we had reliability mechanisms in place for the `DC-1` to `DC-2` connection, and that we could tolerate the extra latency given the maximum allowed end-to-end processing time.

We ended the meeting with the decision to deploy `P` in `DC-2`, while `A` and `B` remained in `DC-1`.

After the meeting, I sat there bewildered—how could all these architects willingly bypass basic architectural principles for political reasons?

### Aftermath

This is why so many large companies end up with complex software architectures that result in slow user experiences. Instead of optimizing for performance, architects often prioritize arbitrary factors—sometimes political.

{{< figure src="https://media1.tenor.com/m/PQreEw8d3fQAAAAC/tom-delonge-blink182.gif">}}

There are many [studies](https://www.gigaspaces.com/blog/amazon-found-every-100ms-of-latency-cost-them-1-in-sales) showing that performance directly impacts user experience, which in turn affects revenue—by retaining and attracting users. In my view, performance should be a top priority in architectural requirements.

It’s also true that many architectures are overly complicated due to [organisational structure](https://en.wikipedia.org/wiki/Conway's_law).

Luckily, a higher-level architect who later reviewed the decision said (and I paraphrase):

{{< admonition type=quote open=true >}}
"Are you crazy? Don't put a proxy system that just transforms data into a completely separate data-center".
{{< /admonition >}}

He reversed the decision, and in the end, we implemented the proxy system in the same data center as the other systems.

I’m glad such architects still exist—but we got lucky. The decision could have easily slipped under his radar.

This happened a few years ago. If anyone involved in that decision is reading this and is offended by it—good, you should! Hopefully, this story will change some minds about what truly matters in architecture.

