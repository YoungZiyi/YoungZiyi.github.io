## How to use Hexo and deploy to GitHub Pages
* https://github.com/hexojs/hexo
* https://hexo.io/docs/

This tutorial has been updated to include additional steps for linking to GitHub pages.

### 1. Install Hexo
```
$ npm install -g hexo-cli

$ hexo -v
hexo-cli: 0.1.9
os: Darwin 14.3.0 darwin x64
http_parser: 2.3
node: 0.12.7
v8: 3.28.71.19
uv: 1.6.1
zlib: 1.2.8
modules: 14
openssl: 1.0.1p
```

### 2. Create a project for your GitHub Pages
```
$ cd ~/Desktop/
$ mkdir hexo-sample
$ cd hexo-sample
$ hexo init
INFO  Copying data to ~/***/hexo-sample
INFO  You are almost done! Don't forget to run 'npm install' before you start blogging with Hexo!

$ npm install
```

### 3. Run a test server for your page on Mac
```
$ hexo server
INFO  Hexo is running at [http://localhost:4000/](http://localhost:4000/). Press Ctrl+C to stop.
```

### 4. Set information to use Git
https://github.com/hexojs/hexo-deployer-git
```
$ npm install hexo-deployer-git --save
$ atom .

~~~~~~~~~~~~~~~~~~ _config.yml ~~~~~~~~~~~~~~~~~~
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: YOUR_GITHUB_REPO.git
  branch: master
```

### 5. Deploy your new blog
https://hexo.io/docs/deployment.html
```
$ hexo clean
$ hexo deploy
```

### 6. Set information for your new blog
https://hexo.io/docs/configuration.html
```
~~~~~~~~~~~~~~~~~~ _config.yml ~~~~~~~~~~~~~~~~~~
# Site
title: My Blog
subtitle:
description: Personal blog
author: YOUR NAME
language:
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: URL_FROM_GITHUB_PAGES
root: /REPO_NAME/
permalink: :year/:month/:day/:title/
permalink_defaults:
```

### 6. Deploy your new blog again
https://hexo.io/docs/deployment.html
```
$ hexo clean
$ hexo deploy
```

### 7. Set "watch" before starting your work
"watch" command can monitor your files.  
https://hexo.io/docs/generating.html
```
$ hexo generate --watch
```

### 8. Create a new post file
Leave watch running and open a new terminal tab.

```
$ hexo new first-post
INFO  Created: ~/***/yt8yt.github.io/source/_posts/first-post.md
```

### 8. Edit the above file with Markdown or Hexo's Helper
Hexo's Helper  
https://hexo.io/docs/helpers.html  

I use Atom with "shift + control + m" when I use Markdown :-)  
https://atom.io/

Markdown resources [https://guides.github.com/features/mastering-markdown/](https://guides.github.com/features/mastering-markdown/)

### 9. Deploy your new blog!!
https://hexo.io/docs/deployment.html

No need to clean since we have watch running in a separate tab. 

```
$ hexo deploy
```
After writting the above command, you can see your new blog on GitHub Pages.  
http://******.github.io/REPO_NAME