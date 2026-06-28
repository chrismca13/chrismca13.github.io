---
layout: post
title: "Hockey Stats Web Application"
description: "Using GitHub Actions, S3, and Python to create a weekly refresh of career hockey stats for my team going back up to 15 years."
date: 2026-06-27
categories: [AWS, Data Engineering]
---

I recently built and deployed a web application designed to track and visualize hockey stats for my team going back 15+ years.

Try it out by doing a search for "McAllister" in the panel on the left!

[https://oakhockey.streamlit.app/]

<div style="width: 100%; overflow: hidden; border: 1px solid var(--rule); border-radius: 4px; margin: 1.5rem 0;">
  <iframe 
    src="https://oakhockey.streamlit.app/?embed=true"
    style="width: 125%; height: 750px; transform: scale(0.7); transform-origin: 0 0; border: none;"
    allow="fullscreen">
  </iframe>
</div>

### Motivation

Our hockey team's website only tracked our stats one season at a time, but we wanted to know who the all time leading scorer was. 

### Project Overview

We use a combination of GitHub Actions, Python, and S3 / AWS to automatically update the stats 3 times a week. 
Checkout my [GitHub](https://github.com/chrismca13/oakland-hockey-stats) for a complete overview of the technical process!



