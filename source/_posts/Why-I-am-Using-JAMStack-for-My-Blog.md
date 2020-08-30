---
title: Why I am Using JAMStack for My Blog
date: 2020-07-05 23:21:23
categories:
    - Solution Architecture 
tags: 
    - JAMStack
    - Tech Blog
    - CMS
---


In the [previous post](./2020-06-28-Why-I-Started-a-Tech-Blog.html) I explained my reasons of having a tech blog. To continue my tech blogging journey, I am going to discuss the design decisions I made when selecting which blogging framework or software I should use.

## Which CMS to Use
Instead of using the traditional CMS like WordPress, Drupal or Joomla, I decided to create and maintain my blog using JAMStack.

## What is JAMStack
What is JAMStack? It simply means building website using a stack of web technologies consist of JavaScript, APIs and Markup. In recent years, this approach has gained quite a lot of tractions. Many have deemed JAMStack is more suitable for static content management like blogging, software documentations and change logging than its traditional server side CMS counter parts.

## Server Side CMS vs JAMStack
In a traditional CMS' setting, the web application which has the content management functionalities is deployed to the server, static contents like blog posts are uploaded and maintain in the database, then dynamically serve to the client upon request. 

In contrast, in JAMStack, static contents are not store in the database. Content creators use markup language to create and maintain the static contents and use a static site generator like NEXT.js, Hugo or Hexo...etc. to generate the whole website as static html, css and JavaScript, then deploy them to CDN (content delivery network). This way client will consume the content from CDN without the need of html rendering and data binding. Let say if dynamic functions like comments or site analytics is required, serverless APIs or Microservices can be used. 

## Advantages of Using JAMStack for Blogging

* Better Site Performance
Html are pre-build by generator before the deployment, and they can be served via CDN without going through binding of the data from database. Hence the web pages can be loaded much faster speed.

* More Secure
Without the need of server side processing and database operations, points of failures and security exploits are reduced. 

* Cheaper to Host
Static sites are cheaper to host as compare to dynamic sites. There are many cheap or even free static web hosting solutions around like github pages, AWS S3 and Alibaba OSS. 

* Easier to Scale
If the demand for the site suddenly spike, the CDN able to seamlessly compensate with its scale. On the other hand, server side CMS needs to increase the number of server instances or scale up the the server capacity to meet the demand. 

## Conclusion
As compare to tradition server side CMS, JAMStack is more suitable for static content management like blogging, software documentation and change logging. Fast, secure, cheap and easier to maintain are the main reasons I choose to use it for my tech blog. 