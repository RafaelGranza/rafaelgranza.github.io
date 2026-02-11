---
layout: single
title: "Reverse Feynman Study Technique"
date: 2026-02-07
author_profile: true
author: granza
categories: [Learning, Systems]
tags: [Learning, Engineering]

sidebar:
  - title: "Further Reading"
    text: |
      **Who made this possible:**<br>
      [Richard Feynman](https://en.wikipedia.org/wiki/Richard_Feynman)<br>
      [Diogenes](https://en.wikipedia.org/wiki/Diogenes)

header:
  overlay_image: /assets/images/inverted_image_of_feynman.png
  overlay_filter: 0.8
  caption: "Inverted Image of Feynman"
excerpt: "How to use AI to help you absorb knowledge quickly"
---

<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
<script>
  mermaid.initialize({ startOnLoad: true });
</script>

As an engineer, computer scientist, student, professor, researcher, or any other role, it is **SACRED** (as moral guidance) that you keep improving your knowledge and studying.

You will face challenges that may seem impossible, but the beauty of things is that we, as homo sapiens (and maybe some reptilians), can learn to overcome these challenges.

At some point in your job or your life, you will need to learn something new. Nowadays, with the pressure from stakeholders and in such a fast-paced engineering environment, you need to learn enough, as quickly as possible.

**Learning as fast as possible is an engineering requirement!**

Fortunately, there are tons of very good learning techniques that we can easily use!

In this post, we are going to focus on a very famous one, and extrapolate how to go even further with this technique in the AI era (which I hope never comes, but here we are).

## Feynman Technique
Have you ever heard of the Feynman Learning Technique?<br>
Named after [Richard Feynman](https://en.wikipedia.org/wiki/Richard_Feynman), it is a very powerful learning technique to study ANY subject.

It uses the concept of learning by teaching, and it can be represented by the following flowchart:

<div class="mermaid" style="display: flex; justify-content: center;">
%%{init: {'theme': 'neutral', 'themeVariables': { 'fontSize': '16px'}, "flowchart": {"htmlLabels": true, "useMaxWidth": false}} }%%
graph TD
    T["Teach in very<br/>simple words"]
    D["Discover your Gaps"]
    S["Study"]

    T --> D
    D --> S
    S --> T
</div>

- **Teach**: Start by exposing what you already know. If you know very little about a subject, it is not a problem; you will soon realize where to improve. Use simple words, as if you were explaining to a kid.
- **Discover your Gaps**: You will realize a lot of things you don't know about the subject when you are unable to explain it simply.
- **Study**: Now you know what you should focus your study on, and it's time to do so.
- **Repeat**: Go back to teaching and iterate until you realize you know enough about the subject.

## Reverse Feynman Technique


The main idea here is to reverse the way you explain things!

The "Reverse" here takes place by not explaining to a kid, but explaining to an **"Expert"** (The AI). This one is also known as "Feynman Technique whit Feedback".

You should still keep your language simple to ensure *you* understand the core, but you gain velocity by having your errors pointed out instantly by an AI agent. In the usual Feynman Technique, you can get lost while trying to "Discover your Gaps". Think about it: you could possibly be learning (and explaining) everything wrong the whole time, and you only find out after many iterations!

**It protects the learner from hallucination.**

This "Reverse Feynman Technique" should be called "Feynman Technique with Feedback". The AI is used to review things you missed and to make sure you are not missing the point.

The flow is exactly the same as the previous, but with a small change in efficiency:

<div class="mermaid" style="display: flex; justify-content: center;">
%%{init: {'theme': 'neutral', 'themeVariables': { 'fontSize': '16px'}, "flowchart": {"htmlLabels": true, "useMaxWidth": false}} }%%
graph TD
    T["Teach in very<br/>simple words"]
    D["Discover your Gaps<br/>(with Feedback)"]
    S["Study"]

    T --> D
    D --> S
    S --> T
  
    style D fill:#add8e6,stroke:#333,stroke-width:2px
</div>

- **Discover your Gaps (with Feedback)**: You not only realize things you don't know, but by giving context to the AI (books, repos, etc.), you can be more assertive about what you need to study. This avoids "unknown unknowns."
- **Study**: AI can also help here to find specific content for your identified gaps.

## Example: Learning lvalues and rvalues

My very brief abstract about lvalues and rvalues:

> - **`lvalues` are your average Joe variables**:<br>they have owners, names, and fixed addresses
- **`rvalues` are the disciples of Diogenes**:<br>they have no names, their addresses aren’t fixed, and they can move around freely.

The AI feedback showed me some critical points I was missing:

**Conceptual inaccuracies found by AI:**
* **“Rvalues’ address is not fixed”** ⚠️: Not all rvalues are completely “address-less.” Temporary rvalues do exist in memory, even if only briefly. A more precise phrasing: *“Their address is temporary / not meant to be relied upon.”*
* **“They can move freely”** ⚠️: In C++, "move" refers to move semantics—transferring resources from one object to another—not literally wandering in memory.
* **Missing subcategories**: For full technical accuracy, I should eventually look into *prvalues* and *xvalues*.

And here I rest my case. You iterate it yourself if you want.

## When to stop iterating?

- **From an engineering perspective**: Learn enough to understand the implications of your solution.
- **From a human perspective**: Learn until your soul is satiated.

## P.S.
This post was developed using this technique.