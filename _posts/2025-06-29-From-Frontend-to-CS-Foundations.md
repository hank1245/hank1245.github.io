---
layout: post
title: Studying Computer Science as a Frontend developer
subtitle: Why I went back to study CS fundamentals as a working developer
tags: [Computer Science, Frontend Development]
comments: true
author: Hank Kim
thumbnail-img: /assets/img/cs.jpg
---

# Studying Computer Science as a Frontend developer

I wanted to share a bit about my journey as a front-end developer and why I decided to dive deep into computer science fundamentals. For a while, I worked as a front-end developer at an IoT startup, and it was quite different from typical web development roles.

## More Than Just Web Development

Working at a startup meant wearing many hats. I wasn't just maintaining websites or tweaking existing features. The role demanded a lot of initiative and covered a wide range of responsibilities.

One of my first major projects was building a careers feature for our company website from scratch - everything from job postings to the application process. I also led the gradual adoption of TypeScript across our codebase, which was a great learning experience in improving code reliability.

The most challenging project was a floor plan application I built for our customer service team. Think of it as a specialized tool similar to Figma, but for mapping IoT sensor locations. After installing sensors at client locations, the team needed a way to visualize exactly where each sensor was placed.

The complexity of this project included:

- **Data Visualization**: Using D3.js to create and overlay heatmaps on floor plans, involving complex data interpolation for smooth visualizations
- **3D Graphics**: Implementing Three.js to transform 2D floor plans into interactive 3D models - my first real experience with 3D web development
- **Mobile Development**: Contributing to our React Native mobile app alongside the web applications

It was an incredible period of growth. I was shipping features, working with powerful libraries, and building tools that had real impact. But there was something missing.

## The Knowledge Gap

Despite my successes, I constantly felt like I was missing fundamental understanding. I could get things done, but I didn't truly grasp how things worked under the hood.

This became a real limitation:

**Performance Issues**: When dealing with performance problems, concepts like threads and processes would come up. Without understanding how operating systems work, I couldn't fully comprehend what was happening. I was applying solutions without understanding the underlying principles.

**Mathematical Barriers**: My attempt to build custom heatmap shaders hit a wall due to my weak mathematical foundation. Later, when trying to create cloud effects in React Native Skia, I'd encounter functions like vector projections but couldn't understand the math behind them.

**System Understanding**: I struggled to see how my front-end work fit into the larger system. What exactly did our gateway do? How did the NAS server work? Why were logs so important? Where did the ML models actually run? This disconnect was frustrating - I felt like a small part of a machine I couldn't see.

I realized that to break through this ceiling, I needed to go back to fundamentals.

## Back to the Fundamentals

So I decided to formally study computer science. I dove into calculus, linear algebra, operating systems, server architecture, microservices, Docker, and more.

It was genuinely exciting. Learning about the mathematics behind 3D graphics, understanding how an OS manages processes, and finally seeing how all pieces of a distributed system connect - it felt like a series of puzzle pieces clicking into place. Concepts that had seemed like cryptic jargon suddenly made sense.

## The Real Impact

Did studying computer science immediately make me a dramatically better front-end developer? Honestly, no. My day-to-day coding skills didn't take a sudden leap.

But that wasn't the point.

What I gained was a solid foundation. I now have the mental models to understand entire systems, not just the browser layer. I can have more meaningful conversations with backend and infrastructure engineers. I can approach complex problems with a deeper understanding of the trade-offs involved.

This knowledge feels like a long-term investment. It's the foundation I'll build the rest of my career on. While it may not have immediately changed what I do day-to-day, it has fundamentally changed how I think about problems. And I believe that will ultimately make me a much more complete developer.

The journey from practical coding to theoretical understanding and back has been one of the most rewarding parts of my career so far. Sometimes going back to basics is exactly what you need to move forward.
