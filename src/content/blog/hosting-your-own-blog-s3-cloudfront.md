---
title: "Deploy an Astro site with S3, CloudFront and GitHub Actions"
description: "Create a site with Astro framework and deploy it to AWS infrastructure sing GitHub Actions for CI/CD."
pubDatetime: 2023-03-01T05:17:19Z


---

If you're interested in starting a blog similar to the one you're on right now then this article is for you. Using the lightweight but powerful Astro framework and hosting it  through S3 and CloudFront on AWS is a great, simple and inexpensive way of hosting a site.

### What We're Doing:

Generating a static site using the Astro JS framework and deploying it to AWS. It will be hosted in an S3 bucket and served by CloudFront. You can use the CloudFormation template I've prepared to create the AWS infrastructure. You will update your website in your code editor, push it to GitHub where GitHub Actions will finish the deployment to AWS. Afterwards, use Route 53 for the domain name and ACM to generate an SSL certificate.

### Getting Started:
1. You need an [AWS Account](https://aws.amazon.com/) and [GitHub](https://www.GitHub.com) account.
2. Command line access on Mac, Linux or in Windows through something like VSCode's terminal, PowerShell or Windows Terminal.  
3. Have NodeJS 16+ installed. ``` node -v ``` to check. 
4. Install Astro Framework:
    ``` npm install astro ``` 

### Create an Astro Project
In a folder of your choice, do the following in the command line:

* Create an Astro project:
    1. ```npm create astro@latest```
    2. Follow the prompt to finish installation. Choose a directory name and to initialize  and TypeScript or JavaScript. These are your personal preference and won't matter for the rest of this guide.
    3. ```cd``` into your folder and type: ```npm run dev``` to launch your project. It will be accessible from your browser at default http://127.0.0.1:3000
    4. If you see a blog, you're ready to move on to the next step...

### Create your AWS Infrastructure

I have provided a CloudFormation template in YAML format that will create the infrastructure explained below. See it on my GitHub [cloudfront-s3-site-yml](https://github.com/ryanef/cloudformationtemplates/blob/main/cloudfront-s3-site.yml) 

This CloudFormation template will create:
* S3 Bucket - All your HTML, CSS, images and other files will be stored in this bucket. 
* S3 Bucket Policy - this will block public access straight to the public and only allow the CloudFront OAC to get objects. 
* CloudFront Distribution - This tells CloudFront where our content will be delivered from(the S3 bucket) and various configuration details.
* CloudFront Function - CloudFront doesn't natively support a multi-folder structure(like pages/about.html or posts/topic/index.html) and S3 is actually flat storage with no true folder structure. Our function will add /index.html to the viewer request so CloudFront can find our blog posts.
* CloudFront Origin Access Control - In older guides you will see OAIs used instead of OACs but OACs are recommended by AWS now. We are using this to restrict potentially bad actors from accessing anything in your S3 bucket directly. This forces all traffic to be handled by CloudFront and nobody can directly access files in our bucket.
* Cache Policy - the template provided uses a cache policy similar to the Caching Optimized managed policy. It can be edited to your preference.

* Create the AWS infrastructure
    1. Right click the github .yml link above and save it. 
    2. Go to your AWS account, search for CloudFormation
    3. Click "Create Stack" and choose "with new resources(standard)"
    4. For template source, choose "upload a template file" and choose the YML file above.
    5. Stack name can be your choice, then proceed through the next few screens with default settings
    6. After clicking submit, you'll come to a screen where you see "CREATE_IN_PROGRESS" and you'll see the various resources start building out. This may take awhile. Most resources create very fast but occasionally a CloudFront Distrbution can take a very long time(15+ minutes) to create. 
    7. Wait until you see it says your stack name is CREATE_COMPLETE and then click on the "Outputs" tab near the top of your stack info. You should see the domain name for your CloudFront distribution that looks something like: d2wn80oq5c31.cloudfront.net

    ![Output](/aws-cloudformation-screen.png)

So this ugly cloudfront.net URL is actually a functional URL that points to our S3 Bucket however we haven't added anything to our bucket to see. Our CloudFront Distribution is configured to look for DefaultRootObject: index.html - if that isn't there, access denied. We will remedy this in the upcoming steps by deploying our Astro project. If you have your own domain, we'll later cover how to connect that random CloudFront URL to your own .com or whatever it may be.