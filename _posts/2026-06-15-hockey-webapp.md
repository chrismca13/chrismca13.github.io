---
layout: post
title: "Deployed, Low Latency ML Model"
description: "Shipping a production grade ML Model using AWS infrastructure, served via API Gateway, and accessed through a static frontend website hosted on Amazon S3."
date: 2026-06-27
categories: [AWS, MLOps]
---

# Serverless Home Price Predictor

A 100% cloud-native, serverless Machine Learning web application that predicts home prices using a containerized Scikit-Learn model hosted on AWS Lambda, served via API Gateway, and accessed through a static frontend website hosted on Amazon S3.

**Disclaimer:**

The model is not intended to be particularly accurate (although predictions do seem reasonable), this is an exercise in deploying a low-latency, always on model in the cloud. 

**Link to website:**

[Front End - Home Price Predictor](http://homeprice-final-api-webpagebucket-fzofzswlrjkl.s3-website.us-east-2.amazonaws.com/)


<div style="width: 100%; overflow: hidden; border: 1px solid var(--rule); border-radius: 4px; margin: 1.5rem 0;">
  <iframe 
    src="http://homeprice-final-api-webpagebucket-fzofzswlrjkl.s3-website.us-east-2.amazonaws.com/?embed=true"
    style="width: 125%; height: 750px; transform: scale(0.9); transform-origin: 0 0; border: none;"
    allow="fullscreen">
  </iframe>
</div>


---

## Architecture Overview

The application utilizes a fully serverless, highly scalable infrastructure managed via AWS SAM (Serverless Application Model):

* **Frontend UI:** HTML and Javascript hosted on an **Amazon S3 Static Website Bucket**.
* **API Layer:** **Amazon API Gateway** handling REST endpoints and Cross-Origin Resource Sharing (CORS) configurations.
* **Compute (ML Logic):** An **AWS Lambda Function** packaged as an **ARM64 Docker Container image**. 
* **Model Object Storage:** **Amazon S3 Storage Bucket** holding the serialized serialization file (`home-price-model.joblib`).

## Model and Data
* Data Source
https://www.kaggle.com/competitions/house-prices-advanced-regression-techniques/data?select=train.csv

---

## Repository Structure

```text
├── Dockerfile
├── README.md
├── artifacts
│   └── train.csv
├── frontend
│   └── index.html # Stored in S3 bucket, website front end.
├── model
│   └── train.py # Trains model and drops trained model object in S3
├── requirements.txt
├── samconfig.toml
└── src
    ├── __init__.py
    ├── app.py # The code run by the lambda function. Imports predict() from inference.py
    └── inference.py # loads model and returns score
```







