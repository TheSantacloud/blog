---
draft: "true"
title: Large-Model inference on Mac
date: 2024-07-06T11:06:24+03:00
---
This is a log of my journey on how do large-model inference on my Macbook, rather than having to use a cloud service or buying an expensive PC with a more expensive GPU. And then seeing how fast I can make it go.

## Introduction 

So in case you haven't noticed - everybody's talking about AI. And as a software developer, I wanted to test some stuff out and play around with different frameworks, methods, algorithms. But the thing is, that like a lot of developers, I own a Macbook. And to my surprise almost **NOTHING** is supported, and you're just stuck with "Oh well, this is for data scientists and it requires a GPU and/or the cloud... so either buy those or leave the job to the pros". That bothered me.

I mean, I've been a software developer since 2nd grade, I tinker with things. We didn't have resources then and we made things works. Why shouldn't this be the case now?
When I got my first Macbook somewhere around 8 years ago from one of my earlier jobs, I was surprised by the product efficiency and quality - and only looked back once when they had that god-awful flat keyboard somewhere around 2020. I've used this Macbook to design things in 3D, played games, and created some pretty great software on this machine. This is a great piece of technology with the Apple Silicon chip, with both a CPU and GPU, and it performs like a champ. 
And then comes this thing, which is basically running a freaking program, and suddenly that is "Data Science" domain. Doesn't seem right. I think that (generally speaking) I should be able to use this machine to tinker and work and try out new things. And from that tinkering there might come great things, with great value and scale for other people.

I'm a firm believer that innovation comes from constraints, rather than abundance. Constraints make you think outside the box.
And I wanted to see how much I could push it. I wanted to learn.

## Initial research of existing solutions

After some investigation (and a lot of trials with a lot of different tools and engine), I came up with:

* [[llama.cpp]] - lets you run some models using MacOS that works **great(!)** but it wasn't easily hackable with its insane llama.cpp file, and I wanted to experiment with different optimization algorithms (and perhaps try some of my own)
* [[Ollama]] - lets you play around with some models, and its very high level just inferencing (and it uses llama.cpp under the hood). This makes llama.cpp more approachable with a [[Docker]]-like experience (which is awesome). But still you have no control over the actual throughput of inference.
* Using paid serverless GPU providers like [[Modal]] or [[baseten]] - I wanted to be able to run stuff and iterate locally, and this just doesn't do that. I'm sure that these solutions are great, but this failed to meet my criteria. Also I hate DSLs for deployment. 
* [[Google Colab]] - playing around with actual code with model, however - in their extremely janky notebook experience, and also it's on somebody else's machine.

**Not viable for Macs (or somewhat viable, but REALLY slow and inefficient):** 
- [[DeepSpeed-MII]]
- [[CTranslate2]]
- [[vLLM]]
- [[pytorch]]
- [[text-generation-inference]]
## What language to pick?

This was another consideration 