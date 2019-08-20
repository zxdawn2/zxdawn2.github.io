---
title: "Hugo+Staticman+Travis"
date: 2019-08-17
tags: ["hugo", "web"]
categories: ["sci-tech"]
img: "/images/sci-tech/2019-08/Hugo_Staticman_Travis_4.png"
draft: false
toc: true
---

This blog post documents the setup of deploying Hugo with Travis CI to the GitHub Pages.

<!--more-->

## Prerequisites

- [GitHub](https://github.com/) account;
- [Travis CI](https://travis-ci.org/) account (login with GitHub account);
- Installing [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [hugo](https://gohugo.io/getting-started/installing/) on your local system

## Create Basic Site

Here I want to create a site named `demo`:

```
$ hugo new site demo
```

Generated directory structure by Hugo:

```
└── demo
    ├── archetypes
    ├── config.toml
    ├── content
    ├── data
    ├── layouts
    ├── static
    └── themes
```

Since I decide to combine the modified [AllinOne](https://github.com/zxdawn/AllinOne) and [hugo-resume](https://github.com/zxdawn/hugo-resume) theme, I need to clone the two theme first:

```
$ cd demo/themes
$ git clone https://github.com/zxdawn/AllinOne_resume.git AllinOne
$ git clone https://github.com/zxdawn/hugo-resume.git resume
$ cd ..
```

Then, the configuration file `config.toml` should be modified. Here's my example:

```
baseURL                    = "https://zxdawn2.github.io/"
builddrafts                = false
languageCode               = "zh-Hans"
canonifyurls               = true
contentdir                 = "content"
layoutdir                  = "layouts"
publishdir                 = "public"
enableEmoji                = true
hasCJKLanguage             = true
summaryLength              = 200
Paginate                   = 5

#theme                      = "AllinOne"
theme                      = ["AllinOne", "resume"]

[outputs]
home = [ "HTML", "RSS", "JSON" ]
section = [ "HTML", "RSS"]
taxonomy = [ "HTML", "RSS"]

[outputFormats.resume]
baseName = "index"
mediaType = "text/html"
isHTML = true

title                      = "Dreambooker"

pygmentsuseclasses         = true
disablePathToLower         = true 

[permalinks]
sci-tech                 = ":year/:month/:day/:slug/"
essay                     = ":year/:month/:day/:slug/"

[taxonomies]
  tag                      = "tags"
  series                   = "series"
  category                 = "categories"


[menu]
  [[menu.main]]
    name                   = "Sci-Tech"
    weight                 = 1
    identifier             = "sci-tech"
    url                    = "sci-tech/"

  [[menu.main]]
    name                   = "Essay"
    weight                 = 3
    identifier             = "essay"
    url                    = "essay/"

  [[menu.main]]
    name                   = "About"
    weight                 = 9
    identifier             = "about"
    url                    = "about/"

[params]
  faviconfile              = "img/logo.jpg"
  avatar                   = "img/me.jpg"             # path to image in static dir e.g img/avatar.png (do not use in the same time as gravatar)
  author                   = "Xin Zhang"
  description              = ["Life is a dream,", "Dream is a life."]           # appears in the site header when set to a non-empty string
  welcome_head             = "Hello, Dreambooker!"
  welcome_word             = "~ Chase your dream ~"

  latestpostscount         = 5                             # how many posts to display on the home page
  bloggroupby              = 'month'
  dateform                 = "Jan 2, 2006"
  dateformfull             = "2006-01-02  Monday  15:04:05"
  noshowreadtime           = false

  # slides
  slidesDirPath            = "static/img/header-slides"    # path to image in local dir (for hugo)
  slidesDirPathURL         = "img/header-slides"    # path to image in static dir (for static pages)

  # highlighting 
  highlightjs              = true

  # links
  email                    = "xinzhang1215@gmail.com"
  github                   = "//github.com/zxdawn"
  twitter                  = "//twitter.com/zhangxin_dawn"
  instagram                = "//instagram.com/zhangxin_dawn/"
  include_rss              = true                         # include RSS <link> tag in <head> and show RSS icon


  # analytics
  # googleAnalytics          = "UA-90057853-1"

  # counter
  busuanzi                 = true

  firstName = "Xin"
  lastName = "Zhang"
  address = "Nanjing, CN"
  phone = "+86 15895938361"
  contactNote = "Mr."
  profileImage = "img/portrait.jpg"
  favicon = "images/favicon.ico"

  showSkills = true
  showProjects = false
  showOpenSource = true
  showPublications = true
  showExperience = false
  showEducation = true
  showQr = true

[params.staticman]
  username = "zxdawn2"
  repository = "zxdawn2.github.io"
  branch = "source"
  notifications = true

[[params.handles]]
    name = "GitHub"
    link = "https://github.com/zxdawn/"

[[params.handles]]
    name = "Twitter"
    link = "https://twitter.com/zhangxin_dawn/"

[[params.handles]]
    name = "Youtube"
    link = "https://www.youtube.com/channel/UC2y2YOGOWZXmOKzR1h_8wGw"

[[params.handles]]
    name = "Instagram"
    link = "https://www.instagram.com/zhangxin_dawn/"

[[params.handles]]
    name = "Facebook"
    link = "https://www.facebook.com/zxdawn"
```

Then, put your image files under `/static/img` folder.

Create `about`, `essay`, `sci-tech` directories under `/content` and add contents.

To check the effect, just run `hugo server` and open `http://localhost:1313/` in browser.

## Create repository

Starting from here there are two possible ways:

1. to use one repository for sources (hugo repository) and publishing (generated HTML)
2. to use two repositories: one for sources and another for publishing

I prefer the first option which needs only one repository and create the empty `<user/org name>.github.io` repository without README file. I create a new account named `zxdawn2` for this purpose. So, the corresponding repository name should be `zxdawn2.github.io`:

![repository](/images/sci-tech/2019-08/Hugo_Staticman_Travis_1.png)

Two branches will be created: `master` and `code`.

- `master` branch will be used for hosting of generated HTML
- `code` branch will be used as the sources of my page

Here're all steps to set these branches:

### Branch source

```
# Initialized empty Git repository
$ git init

# Create new branch named source
$ git checkout -b source

# Exclude the `public` directory which is deployed to `master` branch
$ echo '/public/' >> .gitignore

# Add themes correctly
$ git rm --cached themes/AlliOne
$ git rm --cached themes/resume
$ git commit -m "remove submodule"
$ git add themes/AllinOne/
$ git add themes/resume/
$ git commit -m "added codebase"

# Commit all source codes to `source` branch
$ git add .
$ git commit -m "Hugo generator source code"
$ git remote add origin git@github.com:zxdawn2/zxdawn2.github.io.git
$ git push -u origin source
```

### Branch master

```
# Generate codes of website
$ hugo

# Copy contents to temporary directory
$ HUGO_TEMP_DIR=$(mktemp -d)
$ cp -R public/* "$HUGO_TEMP_DIR"

# Create `master` branch
$ git checkout --orphan master

# Remove unnecessary contents
$ rm .git/index
$ git clean -fdx
$ echo '/themes/' >> .gitignore

# Copy files of website
$ cp -R "$HUGO_TEMP_DIR"/. .

# Commits
$ git add .
$ git commit -m 'initial website content'
$ git push -u origin master

$ [[ -d "$HUGO_TEMP_DIR" ]] && rm -rf "$HUGO_TEMP_DIR"
```

It's time to check whether the website works in browser: `https://zxdawn2.github.io/`.

If you got `404` error, you need to initialize the `README` file of `master` branch.

## Travis CI

There're two possible ways:

1. To use Github token;
2. To use HTTPS for custom domains

I prefer the second method which is simpler and easier.

### Add variable to Travis

Now that we need to set a private variable name `GITHUB_AUTH_SECRET` on Travis:

(update the placeholders with your data)

```
Name  = GITHUB_AUTH_SECRET
Value = https://<GITHUB USERNAME>:<GITHUB PASSWORD>@github.com
```



![variable](/images/sci-tech/2019-08/Hugo_Staticman_Travis_2.png)

### .travis.yml

Switch to `source` branch, write the script below into file `.travis.yml`:

```
---
install:
  - wget -O /tmp/hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.52/hugo_0.52_Linux-64bit.deb
  - sudo dpkg -i /tmp/hugo.deb

script:
  - hugo

deploy:
  - provider: script
    script: chmod +x ./deploy.sh && ./deploy.sh
    skip_cleanup: true
    on:
      branch: source # or master if you are using the two-repository approach
```

### deploy.sh

Create corresponding `deploy.sh`:

```
#!/bin/bash

set -e

echo $GITHUB_AUTH_SECRET > ~/.git-credentials && chmod 0600 ~/.git-credentials
git config --global credential.helper store
git config --global user.email "<USERNAME>@users.noreply.github.com"
git config --global user.name "<USERNAME>"
git config --global push.default simple

rm -rf deployment
git clone -b master https://github.com/<GITHUB HTTPS PATH TO YOUR PUBLISHING REPO> deployment
rsync -av --delete --exclude ".git" public/ deployment
cd deployment
git add -A
# we need the || true, as sometimes you do not have any content changes
# and git woundn't commit and you don't want to break the CI because of that
git commit -m "rebuilding site on `date`, commit ${TRAVIS_COMMIT} and job ${TRAVIS_JOB_NUMBER}" || true
git push

cd ..
rm -rf deployment
```

Push both of them to the `source` branch of remote repository.

```
$ git add .
$ git commit -m "add travis"
$ git push
```

## Staticman

Add `staticman.yml` under root directory and add `@staticmanlab` to the collaborator on github:

![variable](/images/sci-tech/2019-08/Hugo_Staticman_Travis_3.png)

Access the link below to accept the invitation:

https://staticman3.herokuapp.com/v3/connect/github/zxdawn2/zxdawn2.github.io/

You will see `OK!` on that page.

## Reference

1. [Using Hugo and Travis CI To Deploy Blog To Github Pages Automatically](https://axdlog.com/2018/using-hugo-and-travis-ci-to-deploy-blog-to-github-pages-automatically/)
2. [Hugo on GitHub Pages with Travis CI](https://www.sidorenko.io/post/2018/12/hugo-on-github-pages-with-travis-ci/)
3. [Hugo + Staticman](https://networkhobo.com/2017/12/30/hugo-staticman-nested-replies-and-e-mail-notifications/)
4. [Threaded comments for Hugo with Staticman v3](https://jameskiefer.com/posts/threaded-comments-for-hugo-with-staticman-v3/)