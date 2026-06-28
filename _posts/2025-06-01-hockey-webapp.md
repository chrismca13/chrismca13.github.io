---
layout: post
title: "Building the Oak Hockey Web Application"
description: "A deep dive into the architecture, data pipeline, and interactive components behind my latest hockey tracking application."
date: 2026-06-27
categories: [AWS, Data Engineering]
---

I recently built and deployed a web application designed to track and visualize hockey stats for my team going back 15+ years:

https://oakhockey.streamlit.app/

<div style="width: 100%; overflow: hidden; border: 1px solid var(--rule); border-radius: 4px; margin: 1.5rem 0;">
  <iframe 
    src="https://oakhockey.streamlit.app/?embed=true"
    style="width: 125%; height: 750px; transform: scale(0.8); transform-origin: 0 0; border: none;"
    allow="fullscreen">
  </iframe>
</div>

### Motivation
Our hockey team's website only tracked our stats one season at a time, but we wanted to know who the all time leading scorer was. 

### Project Overview
We use a combination of GitHub Actions, Python, and S3 / AWS to automatically update the stats 3 times a week. 
Checkout my GitHub for a complete overview of the technical process:

https://github.com/chrismca13/oakland-hockey-stats

