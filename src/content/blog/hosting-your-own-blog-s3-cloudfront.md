---
title: "Deploy an Astro site with S3, CloudFront and GitHub Actions"
description: "Create a site with Astro framework and deploy it to AWS infrastructure sing GitHub Actions for CI/CD."
pubDatetime: 2023-03-01T05:17:19Z


---

If you're interested in starting a blog similar to the one you're on right now then this article is for you. Using the lightweight but powerful Astro framework and hosting it  through S3 and CloudFront on AWS is a great, simple and inexpensive way of hosting a site.

## What We're Doing:

Generating a static site using the Astro JS framework and deploying it to AWS. It will be hosted in an S3 bucket and served by CloudFront. You can use the CloudFormation template I've prepared to create the AWS infrastructure. You will update your website in your code editor, push it to GitHub where GitHub Actions will finish the deployment to AWS. Afterwards, use Route 53 for the domain name and ACM to generate an SSL certificate.

## Getting Started:
1. You need an <a href="https:/aws.amazon.com" target="_blank">AWS Account</a> and <a href="https:/github.com" target="_blank">GitHub account</a>.
2. Command line access on Mac, Linux or in Windows through something like VSCode's terminal, PowerShell or Windows Terminal.  
3. Have NodeJS 16+ installed. ``` node -v ``` to check. 
4. Install Astro Framework:
    ``` npm install astro ``` 

## Create an Astro Project
In a folder of your choice, do the following in the command line:

* 
    1. ```npm create astro@latest```
    2. Follow the prompt to finish installation. Choose a directory name and **choose yes to initialize a Git repo**, this will be used later.TypeScript or JavaScript - these are your personal preference and won't matter for the rest of this guide.
    3. ```cd``` into your folder and type: ```npm run dev``` to launch your project. It will be accessible from your browser at default http://127.0.0.1:3000
    4. If you see a blog, you're ready to move on to the next step...

## Create your AWS Infrastructure

I have provided a CloudFormation template in YAML format that will create the infrastructure explained below. See it on my GitHub <a href="https://github.com/ryanef/cloudformationtemplates/blob/main/cloudfront-s3-site.yml" target="_blank">cloudfront-s3-site.yml</a>. Save this in WhateverFile.yml on your computer.

This CloudFormation template will create:
* **S3 Bucket** - All your HTML, CSS, images and other files will be stored in this bucket. 
* **S3 Bucket Policy** - this will block public access straight to the public and only allow the CloudFront OAC to get objects. 
* **CloudFront Distribution** - This tells CloudFront where our content will be delivered from(the S3 bucket) and various configuration details.
* **CloudFront Function** - CloudFront doesn't natively support a multi-folder structure(like pages/about.html or posts/topic/index.html) and S3 is actually flat storage with no true folder structure. Our function will add /index.html to the viewer request so CloudFront can find our blog posts.
* **CloudFront Origin Access Control** - In older guides you will see OAIs used instead of OACs but OACs are recommended by AWS now. We are using this to restrict potentially bad actors from accessing anything in your S3 bucket directly. This forces all traffic to be handled by CloudFront and nobody can directly access files in our bucket.
* **Cache Policy** - the template provided uses a cache policy similar to the Caching Optimized managed policy. It can be edited to your preference.

* **Create the AWS infrastructure**

    1. Grab the CF template code from <a href="https://github.com/ryanef/cloudformationtemplates/blob/main/cloudfront-s3-site.yml" target="_blank">the file on Github</a> and save it in a .yml file 
    2. Go to your AWS account, search for **CloudFormation** in the services search bar at the top
    3. Click **"Create Stack"** and choose **"with new resources(standard)"**
    4. For template source, choose **"upload a template file"** and choose the YML file above.
    5. Stack name can be your choice, then proceed through the next few screens with default settings
    6. After clicking submit, you'll come to a screen where you see "CREATE_IN_PROGRESS" and you'll see the various resources start building out. This may take awhile. Most resources create very fast but occasionally a CloudFront Distrbution can take a very long time(15+ minutes) to create. 
    7. Wait until you see it says your stack name is CREATE_COMPLETE and then click on the **"Outputs"** tab near the top of your stack info. You should see the domain name for your CloudFront distribution that looks something like: d2wn80oq5c31.cloudfront.net

    ![Output](/aws-cloudformation-screen.png)

So the ugly cloudfront.net URL is actually a functional URL that points to our S3 Bucket but we don't have our website uploaded there yet. We'll do that in the following steps as well as cover how to point your own domain to the cloudfront URL.  

Also, take note of the other values CloudFrontDistributionARN and S3Bucket, we'll need these soon to configure out GitHub Actions secrets.


## Continuous Deployment with GitHub Actions
First we need to create an IAM User and policy for GitHub Actions. 

* **From your AWS console, search: IAM**
    1. Click **Policies** on the left side >> **Create Policy** >> click **JSON** tab
    2. Paste the following statement, but replace **<DISTRIBUTION_ARN>** and both **<BUCKET_NAME>** with the values you can see on your CloudFormation stack output. You can also go to CloudFront and S3 and find the values yourself. Of course remove the < > around DISTRIBUTION_ARN and BUCKET_NAME

        ```
        {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "InvalidationForCF",
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:ListBucket",
                    "cloudfront:CreateInvalidation"
                ],
                "Resource": [
                    "<DISTRIBUTION_ARN>",
                    "arn:aws:s3:::<BUCKET_NAME>/*",
                    "arn:aws:s3:::<BUCKET_NAME>"
                ]
            }
        ]
        }
        ```
    3. Click next, give the policy a name you can remember and find and click Create Policy


  * **Create a User** while still in IAM:
    1. Click **Users** on the left panel > click **Add Users**
    2. Enter a name you can remember and identify 
    3. Choose **"Attach policies directly"** and and search for the policy you just made above
        ![Output](/aws-user-policies.png)
    4. Once your user is created, you should see it in the list of users. Click on it.
    5. Inside your user, click the **Security Credentials** tab
    6. Click **Create Access Key**
    7. Choose Command Line Interface(CLI) and go to Next
    8. Save your Access Key and Secret Access Key for later. **Keep them safe as they allow access to your account**. You'll not be able to retrieve them later. If you lose them, it's not a huge deal but you'll have to generate new ones.




    * **From your GitHub Account**:
    1. Make a new repository, name it whatever you want and at the next screen you'll see:
        ![Output](/git.png)
    2. From the root of your Astro project, in the terminal copy and paste the line that looks like ```git remote add origin git@github.com:YourAccountName/repoName.git```
    3. After you've added the origin, back in your GitHub account, click **Settings** that I've circled near the top of the screenshot above
    4. Go to **Secrets and Variables** >> **Actions** on the left menu
    5. You need to make four different repository secrets, name them like this:
        - **BUCKET_ID** - the value will be the name of your S3 Bucket. You can see it in CloudFormation stack output as mentioned earlier, or get it from S3 yourself in the AWS Console.
        - **DISTRIBUTION_ID** - the value is CloudFront Distribution ID, also can be seen in CloudFormation stack output or get it from CloudFront in AWS console. It will look something like EOJKLEIAYSGO
        - **AWS_ACCESS_KEY_ID** - the value is what was shown when you made the IAM User
        - **AWS_SECRET_ACCESS_KEY** - the value is what was shown when you made the IAM User
    
    
    
    We're done in the GitHub Account for now, so back in the code editor and terminal:

    * **Make a GitHub Actions Workflow file**:
        1. In the root of your Astro project folder, make a folder: **.github** (include the . at the beginning)
        2. Inside of .github, make another folder called **workflows**
        3. Inside workflows, create a file named **deploy.yml**
        4. Inside deploy.yml, enter the following code:
            ```
            name: Deploy Website

            on:
            push:
                branches:
                - main

            jobs:
            deploy:
                runs-on: ubuntu-latest
                steps:
                - name: Checkout
                    uses: actions/checkout@v3
                - name: Configure AWS Credentials
                    uses: aws-actions/configure-aws-credentials@v1
                    with:
                    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
                    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
                    aws-region: us-east-1
                - name: Install modules
                    run: npm ci
                - name: Build application
                    run: npm run build
                - name: Deploy to S3
                    run: aws s3 sync ./dist/ s3://${{ secrets.BUCKET_ID }}
                - name: Create CloudFront invalidation
                    run: aws cloudfront create-invalidation --distribution-id ${{ secrets.DISTRIBUTION_ID }} --paths "/*"
            ```
        **NOTE** - Change **aws-region** to match the region your infrastructure was made. Also, this only deploys code when the **main** branch is pushed. If your main branch is called master, it won't be detected, so change that in the code above or change your branch name. 

        ## Almost done!
        
        Now push your project to Git. From the root of your Astro project enter the following:
        ```
        git add .
        git commit -m "First push"
        git push origin main
        ```

        Back in your GitHub account, if you go to your repo, then click **Actions** at the top, you'll see the workflow is running and pushing the files to your S3 Bucket. Now when you go to your CloudFront URL, you'll see the Astro site. 

        ## Add Your Own Domain and SSL:

        This can vary depending on where you registered your domain. 

        * **In AWS, go to CloudFront and select your Distribution**
            1. Click **edit** under **Settings**
                ![Output](/cf-edit.png)
            2. At CNAME, add the domains you want. You'll want to add yourdomain.com, www.yourdomain.com
                ![Output](/cf-cname.png)
            3. Click **Request Certificate** which you can see at the bottom of the above screenshot
            4. This takes you to the ACM(AWS Certificate Manager) service. Choose request a public certificate.
            5. Enter the domain and subdomains(such as www.) that you want the certificate to be valid for just like you did in CloudFront settings above. 
                ![Output](/acm.png)
            6. Choose DNS Validation then click Request to proceed

            This is where things change based on where you registered your domain. 

            **If your domain is setup in Route53:**

            At this screen, click the refresh button several times until you see a certificate show, then click on it.

            ![Output](/acm-cert.png)
       
            After clicking on the certificate, click **Create Records in Route 53** and click Create Records again.
            ![Output](/aws-record.png)

            This will automatically do the DNS validation for you. It can take anywhere from several minutes to 15 minutes for the certificate to validate but in your certificate status screen in ACM you'll see it change from *Pending* to *Issued*.

            After you see the certificate is issued, go back to your CloudFront distribution settings and press the refresh button beside the Custom SSL certificate menu and select the certificate from the menu.
            ![Output](/aws-ssl.png)

            Make sure you have your alternate domain CNAMES configured as explained above then click Save Changes at the bottom.


            ### * **Configure Route 53** 
            1. Go to Route 53 AWS Service
            2. On the left, click  **Hosted Zones**
            3. Click on your domain name's hosted zone
                
                **If you don't see a hosted zone**:  It should have been automatically made for you if you purchased your domain through AWS. If you bought your domain elsewhere, you'll need to make the hosted zone yourself, then follow your domain registrar's documentation to update the name servers for your domain to point at the name servers Route53 gives you when you make a public hosted zone.
            4. You'll need to **Create record** for each domain you want to be accessible. So in my example it is ryanf.dev and subdomain www.ryanf.dev
            5. Click **Create Record**
            6. Turn on **Alias**
            7. For the apex/root domain(ryanf.dev) leave the subdomain field empty
            8. Select **Record type**: A -- ROutes traffic to an IPv4 address and some AWS Resources
            9. Set **Route traffic to**: Alias to CloudFront Distribution
            10. You may have to manually enter your CloudFront distribution URL in the menu if the drop down menu doesn't automatically find it. That seems to be a small bug.
            ![Output](/route53.png)


            ### All done!

            You now have a website that is hosted on S3 and served by CloudFront with a continuous deployment setup through GitHub Actions. So every time you write a new blog or make edits to your website, simply push the changes to your Git repo's main branch then GitHub Actions will handle the rest for you. 

            Hosting a website in this way provides amazing speed at a very low cost. As always, refer to how pricing works with S3, CloudFront and Route 53 to know what your spend will be and **setup billing alarms** so you're never caught by surprise.

