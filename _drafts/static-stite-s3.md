---
layout: post
title: "Host a Static Stite Using AWS S3 and CloudFront"
date: 2017-01-30 20:48:00 -0600
categories: aws
---

How to set up a static site with AWS hosted with an S3 bucket. For these
examples, I'll just use my domain `kylegengler.com`. Make sure you are
registered with [Amazon Web Services](https://aws.amazon.com/)

## Register Domain Name

Do this with whatever domain registrar you choose. There are a lot of offerings
here such as [Route 53](https://aws.amazon.com/route53/) or 
[Namecheap](https://www.namecheap.com/). All that's important here is that you
get ownership of the domain name you want to use, and that you have the ability
to set custom nameservers.

## Create the S3 Bucket

Navigate to the [S3 console](https://console.aws.amazon.com/s3/). On this screen
you will want to select `Create Bucket`

```
S3 Console Image
```

Choose a descriptive name for the bucket, and then you can basically just click
through the defaults to create the bucket. For the sake of this example, we'll
say I created a bucket called `s3-kylegengler`.

```
S3 Bucket Created Image
```

## Upload Site Content to S3 Bucket

Creating the actual site is outside of the scope of this post. All that's
important is that this guide assumes that there is a root file that will be 
called `index.html` which will act as the site homepage. For example:

```
├── assets
|   ├── styles.css
|   └── app.js
└── index.html 
```

Uploading all these files individually to S3 through the web interface is kind
of a pain. Luckily, you can use the aws-cli to sync your site directory with the
S3 bucket.

`aws s3 sync site-dir/ s3://s3-kylegengler`

> Make sure to substitute `s3-kylegengler` with the name of your bucket, and to
> substitute `site-dir` with the path of your static site on your machine.

