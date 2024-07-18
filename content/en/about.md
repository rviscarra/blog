---
title: 'About'
date: 2019-06-23T22:43:14-06:00
layout: 'about'
weight: 10
comments: false
---

I'm a Salvadoran software engineer living in Canada. I enjoy tinkering with new technologies, weightlifting and 
learning new things. Writing clean, efficient and correct software is my passion.

I have been a professional software engineer for more than 10 years, these are 
some of the most interesting project I've worked on:

{{<project-card title="Ad-tech platform" tech-icons="python golang docker aws" other-tech="Kubernetes">}}
  Spearheaded the development of a highly scalable ad-tech platform for a prominent US-based media company that handled 
  hundreds of millions of requests per day. 

  **Main challenges**
  - High volume of requests.
  - Sub-second latency was a hard requirement.
  - Logging and monitoring at scale.
{{</project-card>}}

{{<project-card title="Emoji keyboard rendering cluster" tech-icons="node docker aws" other-tech="C++">}}
  Implemented an auto scalable, animation rendering cluster in AWS.

  **Main challenges**
  - Correctly handling traffic spikes, since a single user could re-render 500+ animations at a time.
  - GPU-based rendering in headless servers.
  - Optimizing the renderer by implementing the critical paths in C++.
{{</project-card>}}

{{<project-card title="Bitcoin inscription platform" tech-icons="python rust bitcoin docker aws" other-tech="Pulumi">}}
  Designed and implemented a Bitcoin inscription platform.

  **Main challenges**
  - Security, given the platform dealt with cryptocurrencies and digital assets.
  - Technical complexity of dealing with Bitcoin constructs such as transactions, signatures, SegWit, Taproot, etc ...
  - Interacting with a distributed mesh network.
{{</project-card>}}

{{<project-card title="Game backends" tech-icons="erlang docker aws unity" other-tech="C# F# Elixir">}}
  Several backend services for different video games.

  **Main challenges**
  - Dealing with in-memory state in distributed systems.
  - Realtime updates to a global shared state (e.g. tiles in a map)
  - Rich domains that need to be shared between backend and clients, supporting old clients.
{{</project-card>}}

I'm also an open-source enthusiast, these are some of open-source projects of mine:

{{<project-card title="rviscarra / webrtc-speech-to-text" tech-icons="golang" other-tech="WebRTC" repository="rviscarra/webrtc-speech-to-text">}}
  Performs real-time speech transcription in the browser via WebRTC and Google Speech.
{{</project-card>}}

{{<project-card title="rviscarra / webrtc-remote-screen" tech-icons="golang" other-tech="WebRTC" repository="rviscarra/webrtc-remote-screen">}}
  Allows the user to watch the screen of a remote system in the browser via WebRTC.
{{</project-card>}}

If you want to get in touch you can send a message to `{{<param email>}}`.
