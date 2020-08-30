---
title: 'Blog setup using Hexo, GitHub Action and Alibaba Cloud OSS+CDN'
date: 2020-07-11 23:00:00
tags:
---

In the [previous post](./2020-07-05-Why-I-am-Using-JAMStack-for-My-Blog.html) I have discussed what is JAMStack and why I am using for my tech blog. In this post I am going to share the workflow I used to set up my tech blog.

The simplest way setup a JAMStack blog is to use a static site generator like Hexo, NEXT.js or Hugo, write your content in a markup that the generator supports, generate the site into html, js and css files, then use the generator's build-in deploy function to deploy it on to some static web server like Github pages.

However, I decided to spice up things a little. According to [jamstack.wtf] (https://jamstack.wtf/#best-practices), there are some best practices when doing JAMStack development:

1. The use of CDN.
2. Atomic build, which means always deploy a full snapshot of the site.
3. Cache invalidation at CDN, when there is a new build.
4. Everything should be version controlled.
5. Automated Build.

Creating a personal blog may not need to follow all these, but for learning purpose I decided to use the follow tools to help me achieve these best practices:

- Hexo, a popular node js based static site generator
- GitHub for version control
- GitHub Action to automate the build and deployment of the site
- Alibaba OSS and CDN for hosting the of the blog

## Setup Hexo 
Before setting up Hexo make sure you have node.js and Git installed.
More info: [Hexo Requirements](https://hexo.io/docs/#Requirements)

### Install Hexo
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
### Create a new post
``` bash
$ hexo new "My New Post"

```
### Write post content
Newly created post can be found in "folder_name/source/posts/". In Hexo, you need to use markdown language to write the content.  

More info on writing in markdown and Hexo: 
[Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
[Hexo Writing](https://hexo.io/docs/writing.html)

### Local test run
``` bash
$ hexo server
```
 
### Install themes
Select a theme from [here](https://hexo.io/themes/), follows the instruction given in the theme's readme to setup the theme.


## Setup GitHub Repository
Create a new repository in you Github account.
{% asset_img new_repo.png new_repo %}


#### Init and push the blog source code to GitHub
In your blog directory.
``` bash
git init
git commit -m "commit comments"
git remote add origin https://github.com/<github account>/<repository name>.git
git push -u origin master
```
Go to you repositories page at GitHub to verify source code are checked in.

## Setup Alibaba Cloud OSS
### Create OSS Bucket
Create an OSS bucket in Alibaba Cloud OSS, you can use the default settings except to set the **Access Control List (ACL)** to **Public Read**.

### Static pages settings in OSS
To host static pages in OSS, click on the newly created Bucket and go to basic settings, under the **Static Pages** section change the **Default Homepage** to **index.html** and **Default 404 Page** to **404.html**
{% asset_img oss_static.png oss_static %}

## Set up Alibaba CDN
The advantage of using CDN is it will accelerate the content delivery as it caches the static contents in nodes near the users. This works particularly well with JAMStack, as the generated content are all static files.
Before using CDN, you need to have an existing domain name. The setup will be easier if your domain name is with Alibaba Cloud.
To purchase domain name at Alibaba Cloud click [here](https://www.alibabacloud.com/domain/search)

### Add domain names
Go to Alibaba Cloud CDN, Domain Names, add your domain name with the following settings:
{% asset_img cdn_domain.png cdn_domain %}

Set Origin Info to **OSS Domain** and select the OSS url which points to your newly created OSS bucket from the Domain Name list.

For **Port** set to **433** if you intend to use https (will show the steps later), if not chose **80**.

Region change to Global (Excluding Mainland China).

After the Domain Name is added, a CNAME will be generated (it will take awhile). Copy this value for DNS setting later. 
{% asset_img cdn_cname.png cdn_cname %}

### DNS settings
Go to Alibaba Cloud DNS, select your Domain Name and add the @ and * CNAME records.
{% asset_img dns_cname.png dns_cname %}
---
{% asset_img dns_cname2.png dns_cname2 %}

Set the **Value** to the CNAME you copied from the CDN earlier.

### Set up HTTPS
There are a few ways to setup SSL for your Domain, the easiest way is to use Alibaba Cloud CDN's free Https Certificate.

In Alibaba Cloud CDN click on the your Domain Name, under HTTPS section, chose Modify.
{% asset_img cdn_https.png cdn_https %}

Enable HTTPS Secure, and chose Free Certificate. By doing this Alibaba Cloud will apply the SSL for you (the approval process will take a few hours).
{% asset_img cdn_ssl.png cdn_ssl %}

After that change **Force Redirect** to **HTTP->HTTPS**.

### Check if CDN setup is correct
Go to OSS, click on the your Bucket, then **Transmission**, **Domain Names**. 
{% asset_img cdn_bind.png cdn_bind %}

{% asset_img oss_bind.png oss_bind %}

Make sure your custom domain name is pointing to the Alibaba Cloud CDN Domain Name and then points to the OSS Domain Name.

## Set up GitHub action for CI/CD
### Create secrets for Github Action
Go to your Github repository, then **Settings**. Create two secrets as the follows:
{% asset_img git_secret.png git_secret %}
You can find the access id and secret key of your Alibaba Cloud Account in ALibaba Cloud console Access Key Management.
{% asset_img ali_key.png ali_key %}

### Create a Github Action
Go to your blog directory, create a folder name ".github", and a yaml file action_name.yml, and put the following content inside:

``` yml
# This is a basic workflow to help you get started with Actions
name: Deploy

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on: [push]  

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - uses: actions/checkout@v2
      with:
        ref: master
       
    - name: Update Submodule   # use if there is submodule.
      run: |
        git submodule init
        git submodule update --remote
    - name: Setup Node            # install node
      uses: actions/setup-node@v1
      with:
        node-version: "10.x"

    - name: Hexo Generate         # install Hexo and generate files
      run: |
        rm -f .yarnclean
        yarn --frozen-lockfile --ignore-engines --ignore-optional --non-interactive --silent --ignore-scripts --production=false
        rm -rf ./public
        yarn run hexo clean
        yarn run hexo generate
    
    - name: Deploy to OSS # deploy to AliCloud OSS 
      env:
        OSS_AccessKeyID: ${{ secrets.AC_ACCESS_KEY_ID }}
        OSS_AccessKeySecret: ${{ secrets.AC_ACCESS_KEY_SECRET }}
        OSS_EndPoint: oss-<region id>.aliyuncs.com  # Your Alibaba Cloud Region End Point 
        OSS_Bucket: <your bucket name>
      run: |
        wget -q http://gosspublic.alicdn.com/ossutil/1.6.10/ossutil64
        chmod +x ./ossutil64
        ./ossutil64 config -e $OSS_EndPoint -i $OSS_AccessKeyID -k $OSS_AccessKeySecret -L CH
        ./ossutil64 rm -r -f oss://$OSS_Bucket/
        ./ossutil64 cp -r -f ./public oss://$OSS_Bucket/
```
Once this file is checked into Github, a Github Action will be created. This action script will be trigger upon every push of code to your Github master branch, and it will generate the blog html files then push them to your OSS bucket. 

## Going Live
### Hexo tweaks for Alibaba OSS
When hosting static html files in OSS, it can only identify the index.html at the root folder, but Hexo normally will ignore the index.html in the its generated paths. Example: Hexo's path - "your_blog_domain/post_1/", actual path - "your_blog_domain/post_1/index.html"

To fix this, you need to change some of the paths generated by Hexo. In the Hexo config file or Hexo's theme's config file change this:

``` yml
#find the permalink setting in config.yml and change the the path as the follows. 
permalink: :year-:month-:day-:title.html
```
There are also some pages paths defined in theme configs, make sure they are set to the full path with index.html at the end.

Example: change "your_blog_domain/about/" to "your_blog_domain/about/index.html"

### Deploy
After push your blog content to your Github master branch, at your Github Repository, Actions page, you should see the Action Script starts to run. Once completed, your blog is finally ONLINE!

## Conclusion

Using Hexo + Github + Github Action + Alibaba Cloud OSS+ Alibaba Cloud CDN is a good demonstration of JAMStack in action. 
In terms of the cost of using Alibaba Cloud OSS and CDN, you can find them [here](https://www.alibabacloud.com/product/oss/pricing?spm=a3c0i.7950270.1834322160.1.742dab91KeUDPE) (OSS) and [here](https://www.alibabacloud.com/product/cdn/pricing?spm=a3c0i.7958120.7305017250.1.680d7b744o2ZyD) (CDN). For an average traffic blog, it is very cheap. In fact, for OSS, it is free of charge if your blog's storage and traffic is under 5GB. The only thing you need to pay is the traffic in CDN (using CDN is optional). 
