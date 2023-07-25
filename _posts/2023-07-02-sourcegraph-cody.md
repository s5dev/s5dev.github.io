---
layout: post
title: "From ArcGIS to Mapbox: How Cody AI Made My Web App Shine"
categories: [cody, sourcegraph, ai]
tags: [programming, productivity]
description: Discover how Cody AI, the magic AI Assistant, helped me seamlessly upgrade my web app from ArcGIS to Mapbox map, making it responsive for mobile users.
comments: true
---

Imagine being a newcomer to Canada around 2019, relying on public transit to navigate the Waterloo region. Like many others, I found myself frustrated with the occasional unreliability of Google Maps when searching for public transit options. However, my luck changed when I stumbled upon a cool command line tool developed by a UWaterloo student that predicted the next bus or LRT arrival time within seconds. This discovery led me to GRT.ca, a website providing real-time transit feed updates, allowing me to track the exact location of buses and LRTs. Inspired by this newfound resource, I created a naive map using ArcGIS ESRI Maps, loading location information from the protobuf feed and deploying it on my domain, livemap.shivasurya.me. This map became my go-to tool for finding the precise location of the next bus or LRT.

But recently, I decided it was time to give my web app a fresh look and enhance its responsiveness for mobile and tablet users. After some research, I discovered Mapbox, a fantastic alternative to Google Maps. Excited about the possibilities, I wanted to migrate my map to Mapbox but wasn't sure where to start. That's when I turned to Sourcegraph Cody AI, an incredible assistant that allowed me to seamlessly upgrade my UI without leaving my beloved VSCode. In this blog post, I'll share my journey of leveraging Cody AI's power, eliminating the need for extensive Googling, and successfully migrating to the new shiny Mapbox map. Get ready to dive into the world of reliable transit tracking and UI upgrades, all made possible with the help of Cody AI!

### Overview of LiveMap

Before we get started, let's take a look at the LiveMap repo. The repo contains a simple web app that displays the location of buses and LRTs in the Waterloo region. The map is built using ArcGIS ESRI Maps, and the location information is loaded from the protobuf feed. The repo is available [here](https://github.com/shivasurya/waterloo-grt-live-map).

### Getting Started with Cody AI

Sourcegraph Cody AI provides code suggestions and autocompletes, allowing you to quickly implement features, find and fix bugs. You can learn more about Cody AI [here](https://about.sourcegraph.com/blog/cody-ai). In order to feed the context about the project, the embeddings plays important role. Cody AI uses the embeddings to understand the context of the project and provide relevant suggestions. To generate the embeddings, you can headover to [Sourcegraph Discord](https://srcgr.ph/discord-server) and run the command `/embeddings github.com/shivasurya/waterloo-grt-live-map`. This will generate the embeddings and you can start using Cody AI from your VSCode. If you're on Macbook or Linux, you can use [Cody Desktop app](https://sourcegraph.com/cody) to get started which can generate embeddings automatically when you add the project.

![Cody Embedding Bot](/assets/media/cody-embedding-bot.png)

### Migrating to Mapbox

I had around 10 interactions with Cody side-by-side and made changes to the project. You can checkout the Miro board below to see the interactions and changes made to the project.

<iframe width="100%" height="432" src="https://miro.com/app/live-embed/uXjVM5LRZcY=/?moveToViewport=-2376,-2079,8780,4938&embedId=655405980729" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

Finally, Cody answers were less hallucinating after enabling embeddings and giving as much as context about the project. That's why Cody was able to recommend adding `ViewPort` `width` and `Height` to map div tag and suggest adding viewport meta tags to make the map responsive for mobile users. You can check the latest version deployed at [livemap.shivasurya.me](https://livemap.shivasurya.me).

### Closing Note:

 As Sourcegraph Cody AI continues to evolve and impress with its remarkable code writing and suggestion capabilities, it's important to remember that even the most advanced AI systems can sometimes experience moments of "hallucination." In such cases, it's crucial to assess the quality of embeddings and provide sufficient context to guide Cody towards the right answer. With each passing day, Cody is becoming an invaluable companion for developers, offering unparalleled assistance and streamlining the coding process. So, embrace the power of Cody AI, harness its potential, and enjoy the journey of coding with an intelligent and reliable partner by your side.

I hope this post is helpful for developers & engineers 💻. For bugs or hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
