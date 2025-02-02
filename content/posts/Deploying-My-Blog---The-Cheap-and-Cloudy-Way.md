+++
title = 'Deploying My Blog   the Cheap and Cloudy Way'
date = 2025-02-02T16:13:14Z
draft = true
+++

A step-by-step guide on how I deployed this website using Hugo and AWS.

---

# Creating your first Hugo blog

1. Spin up a terminal and do the following:
```bash
sudo apt install hugo
cd
hugo new site mysite
```

2. This will prompt Hugo to create the required files for a base blog. Now you need to find a theme that you like. You can find plenty [here](https://themes.gohugo.io/). I chose **Hugo-Ficurinia**. Once you have found a theme you are happy with, you have to set it up as follows:
```bash
cd ~/mysite/
git clone <github url of your theme> /themes/
cd ..
```

3. Now move the example site into your website root so that Hugo knows how to display it:
```bash
rm -rf content/ static/ config.toml
cp -r themes/<yourThemeName>/exampleSite/* .
```

4. Now all you need to do is generate the site.
```bash
# Generate the site
hugo
# Generate the site and start a local server to preview it
hugo server -D
```
Open your web browser and in the URL bar type `localhost:1313` You should now see your site!

5. You can now further customize your site accoding to your theme. For more information on Hugo you can check:
- [Hugo Quick Start Guide](https://gohugo.io/getting-started/quick-start/)
- [FreeCodeCamp Hugo Guide](https://www.freecodecamp.org/news/your-first-hugo-blog-a-practical-guide/)
- [Kinsta Guide To Hugo](https://kinsta.com/blog/hugo-static-site/)

---

# Buying a domain name

Once you have your website ready you will need to buy a domain name. For this, and since we will be using AWS S3 to host our website, we will also be using AWS [Route53's](https://us-east-1.console.aws.amazon.com/route53/v2/home?region=eu-west-1#Dashboard) for our domain needs.

1. Head over to AWS Route53 page.
2. Under **Registered Domains** click on Register Domain.
3. Choose the domain you want. In this case, our `.com` costed ~$14.
4. Go back to **Route53**->**Hosted zones**->**Your domain**. This is where we will add our DNS records later on.

---

# Creating an S3 Bucket
![Creating an S3 bucket](/img/Deploying-My-Blog/bucket.png)

1. After creating your bucket, go to the **Properties** tab.
2. Go all the way down to "**Static website hosting**" and click *Edit*.

![Enable S3 Web hosting](/img/Deploying-My-Blog/bucketwebhosting.png)

3. Go to **Permissions** tab.
4. *Edit* **Block public access** and make sure it's not blocked.

![Allow S3 Publick access](/img/Deploying-My-Blog/bucketallow.png)

5. *Edit* the **Bucket policy** and add the following:
```json
{
    "Version":"2012-10-17",
    "Statement": [
        {
            "Sid":"PublicReadGetObject",
            "Effect":"Allow",
            "Principal":"*",
            "Action":"s3:GetObject",
            "Resource":"arn:aws:s3:::<YOUR-BUCKET-NAME>/*"
        }
    ]
}
```

Our bucket is now read-only for The World!

---

# Route traffic using Route53
Our S3 bucket link `http://YOUR_BUCKET_NAME.s3-website-eu-west-1.amazonaws.com/` works, but now we want to route the traffic through the domain that we have just bought. For this, we will be using AWS Route53.

1. Go to **Route53** -> **Hosted zones** -> **Create hosted zone**

![Create a Hosted Zone](/img/Deploying-My-Blog/hostedzone.png)

2. Click on your new Hosted Zone and then **Create record**.

![Create record](/img/Deploying-My-Blog/createrecord.png)

We will revisit the remaining configuration once we have created our CloudFront distribution, for now, you should leave it just like this.

---

# Secure and Fast with CloudFront

Now it's time to configure our site to use **HTTPs**. There are many ways that we could do this, but again, if we are on AWS, we will keep it so.

1. Open **AWS Certificate Manager**
2. Click on **Request Certificate**

![Certificate manager](/img/Deploying-My-Blog/certificate.png)
![Certificate manager](/img/Deploying-My-Blog/certificate2.png)

3. Click on your newly requested certificated.
4. Take note of the values in "*CNAME name*".
5. Go again to **Route53** -> **Hosted zones** and click on the one we just created.
6. Click on **Create record**.
7. Add a new CNAME record using the name of one of one of the two domains you registered in the last step.

**Note:** In the *subdomain* you want only the prefix of the proof record.
For ex, if the *CNAME name* value is `_78978897897eeeeqwewqe78970787.itsorphic.com.`, the value in the subdomain field should be entered as: `_78978897897eeeeqwewqe78970787`.

Create the record and within a few seconds to minutes the certificate status should change to **Issued**.

Now that we have a valid certificate, we can configure **HTTPS** using CloudFront.

1. Go to **CloudFront** -> **Distributions** -> **Create**
![Distributions](/img/Deploying-My-Blog/distribution.png)
2. Redirect http trafic to https
![alt text](/img/Deploying-My-Blog/distribution2.png)

We will leave WAF disabled for know, but you should give it some thought if it's worth it for your usecase.
![alt text](/img/Deploying-My-Blog/waf.png) 
![alt text](/img/Deploying-My-Blog/distribution3.png)

---

# Direct Traffic to CloudFront distribution

Now we can revisit our DNS configuration.
1. Go to **Route53** -> **Hosted zones** -> **Your hosted zone** -> **Create record**
2. First we will add (or replace) the A Record for the `**www.subdomain **`

![alt text](/img/Deploying-My-Blog/arecord.png)

3. Now we create another record:
![alt text](/img/Deploying-My-Blog/arecord2.png)

We are all set, the internet traffic should now flow to the CloudFront distribution. You should be able to test and observe that when you enter `http://` you get redirected automatically to `https://`.

---

# Storing sourcecode in GitHub

Straightforward. Push your code to GitHub.
```bash
cd mysite
git add .
git commit -m "my first commit"
git remote add origin git@gitHub.com:itsorphic/mysite.git
git branch -M main
git push -u origin main
```
