---
layout: post
title: "The hiring bottleneck: why I built TestReviewer.ai"
date: 2025-12-06 02:00:00 +0100
comments: true
image: /images/logo_testreviewer.png
categories: [Startups, Engineering]
tags:
    - Startups
    - Engineering
---

I did the math recently, and honestly, it was depressing.

If you are a Tech Lead, you know the struggle: your calendar looks like a game
of Tetris where you are losing. Instead of building apps, I found myself
constantly reviewing them.

The workflow is a killer. You stop your actual work, pull a repo, and pray that
running the candidate's code doesn't hijack your local environment (a genuine
security risk these days).

My colleagues and I tried using LLM prompts to speed things up, but that only
solved the _reading_ part, not the _process_. HR was still chasing us on Slack,
the Tech team was swamped, and candidates were left in the dark.

I realized we didn't just need a code analyzer; we needed a tool to bridge the
gap between the **Recruiter**, the **Candidate**, and the **Technical
Evaluator**.

## Enter TestReviewer.ai

Today, I am launching **[TestReviewer.ai](https://testreviewer.ai)** in Early
Access.

The goal is simple: **HR shouldn't waste time chasing feedback, and Tech
shouldn't waste time on setup and basic code review.**

I built a flow that respects everyone's time.

**Step 1: The Setup -** The recruiter creates a test (e.g., "Senior Android
Developer"), creates a candidate profile, and sends the test link. No friction.

**Step 2: The Submission -** The candidate opens the link, completes the task,
and uploads a Zip file or a Git link.

But here is the "magical" part: We also allow candidates to **upload a video
walkthrough**. If you've ever reviewed a UI/Frontend test, you know that code
doesn't tell the whole story. Seeing the dev explain their choices in a video?
That's a game changer.

**Step 3: The Analysis -** Before a human ever looks at the code, our tool
analyzes the project for **security concerns, anti-cheat, and project quality.**

**Step 4: The Feedback -** The tech team receives a detailed report. They don't
need to run the code locally to know if it's secure or functional. They get a
deep analysis, add their final feedback, and the recruiter gets the green light
to hire (or pass).

## Why different?

The project is in the early stages, but the vision is to save you money and time
immediately.

Most ATS platforms charge you a fortune in monthly subscriptions for seats you
don't use. I hate that model.

**TestReviewer.ai is pay-per-completed-review.** You don't pay for idle seats;
you only pay when you are actually interviewing.

## I need your eyes on this

I’m launching this today, slightly under the radar, because I want **honest
feedback**.

If you are a Recruiter tired of bottlenecks, or a Dev tired of reviewing messy
code, give it a spin.

Check it out here: **[https://testreviewer.ai](https://testreviewer.ai)**

If you find a bug, or if you think a feature is missing, join our [Discord](https://discord.gg/S4WAZsNTPt) or shoot me a mail at
**hi@testreviewer.ai**.

**Happy reviewing,**
