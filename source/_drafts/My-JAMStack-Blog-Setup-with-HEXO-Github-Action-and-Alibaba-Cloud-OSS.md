---
title: 'JAMStack Blog With HEXO, Github Action and Alibaba Cloud OSS'
date: 2020-07-05 20:21:23
tags:
---

In this post I am going to share the design decisions and the steps I took to set up my tech blog nefelibata.tech.

## Which CMS to Use
Instead of using the traditional CMS like WordPress, Drupal and Joomla to create and maintain my blog, I decided to create my blog with JAMStack.

### Server Side CMS vs JAMStack
What is JAMStack? It simply means building website using a stack of web technologies consist of JavaScript, APIs and Markup. In recent years, this approach has gained quite a lot of tractions in the blogging communities. Many have deemed JAMStack is more suitable for static content management like blogging, software documentations and change logging than its traditional server side CMS counter parts.

In a traditional CMS' setting, which underneath can be using stack like LAMP, MEAN, XAMPP, ect., the web application which has the content management functionalities is deployed to the server, static contents like blog posts are added and maintain in the database,then dynamically serve to the client upon request. In contrast, in JAMStack, static contents are not store in the database. Content creators use markup language to create the static contents and use a static site generator to generate the whole website as plain html, css and JavaScript, deploy them to Server and CDN(content delivery network), then client will consume the content from CDN instead of the origin server whenever possible. If dynamic functions like comments, and site analytics are required, APIs or Microservices are used. 

--graph here

### Advantages of Using JAMStack

Here are some of the advantages using JAMStack for blogging:

* Better Site Performance

* More Secure

* Cheaper to Host

* Easier and Cheaper to Scale


## My Blogging Workflow 

With some researches I came up with following workflow to set up my blog:

### Static Site Generator - Hexo

### CI/CD Automation - GitHub Action

### Static Site Hosting - Alibaba Cloud OSS

### Dynamic Contents - Google Analytic and Valine

## Conclusion

