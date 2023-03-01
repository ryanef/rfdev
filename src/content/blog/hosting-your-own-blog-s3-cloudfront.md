---
title: "Astro with S3, CloudFront and GitHub Actions to Host a Site"
description: "Using S3 and CloudFront to host your own blog"
pubDatetime: 2022-09-21T05:17:19Z

---

If you're interested in starting a blog similar to the one you're on right now then this article is for you. Using the lightweight but powerful Astro framework and hosting it  through S3 and CloudFront on AWS is a great, simple and inexpensive way of hosting a site.

### What we're doing:

Generating a static site using the Astro JS framework and deploying it to AWS. It will be hosted in an S3 bucket and served by CloudFront. You can use the CloudFormation template I've prepared to create the S3 bucket and CloudFront distribution or do it yourself. Afterwards, use Route 53 for the domain name and ACM to generate an SSL certificate.

### Getting Started:
1. You need an AWS Account - https://aws.amazon.com/
2. Have NodeJS 16+ installed. ``` node -v ``` to check. if not installed: https://nodejs.dev/en/download/
3. Once Node is installed, install Astro Framework:
    ``` npm install astro ``` 

### Creating an Astro Project


* Create an Astro project:
    1. ```npm create astro@latest```
    2. Follow the quick prompt to finish installation. Enter a directory name or accept default of a random name and choose if you want TypeScript or JS. These are your personal preference and won't matter for the rest of this guide.
    3. cd into your folder and ```npm run dev``` to launch your project at default http://127.0.0.1:3000
    4. If you see a blog, you're ready to move on to the next step...


