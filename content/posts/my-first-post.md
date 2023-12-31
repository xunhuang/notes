+++
title = 'Setting up this site with Hugo and Github'
date = 2023-09-02T12:26:05-07:00
draft = false
+++

## Inspired by Lillian's blog
 
I came across [Lillian Weng's ML Blog](https://lilianweng.github.io/) and enjoyed the style a lot so I decided to create a page to keep track of random note-worthy stuff I ran into withe same [Hugo](https://gohugo.io) tool. 

## Why do I like this

- Clean style
- Markdown based 
- Github to track changes
- Github Pages for free hosting
- Streamlined publishing workflow (commit -> push -> publish) with Github Action. 

## How

I work on a Mac, assuming git is already install. 

[Hugo Quick Start](https://gohugo.io/getting-started/quick-start/) is good.

```
brew install hugo
```

Then create the site locally 

```
hugo new site notes
cd notes
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

Add content 

```
hugo new content posts/my-first-post.md
```

to enable hot reload (edit-see-change-immediately)

```
hugo server -D
```

Now you can edit content and commit locally. 

## Push to github remotely 

### install and configure gh (github CLI)
```
brew install gh
gh auth login
```

this will pop up a browser with link the GitHub and ask you to authroize to create your credentials for this repo. 

### Create the remote repo

```
gh repo create                               
```

Select use the local directory for content  instead of creating a new empty repo.

Afterwards your site should be pushed to the Github site

### Github Page publishing

Follow the instructions [here](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

Steps mainly perform the following

- Setting up Github Page
- Deploy from Main Branch
- Setting up Github Action so that future commits will be publish automatically within 2-3 minutes.  


## Adding Disqus for Comments


## git submodule needs to pull

When setting up update on this on a different computer

```
git clone xxxxx
cd xxxx
git submodule update --init --recursive
# assuming hugo in installed (or 'brew install hugo')
hugo server -D
```

## Oh, you don't need to use a computer, just use Codespace on Github

Believe it or not, just press "." on this repo, and github will launch a codespace for you to access via your browser and VSCode. 

You can sync your settings and start editing. However, you must run this first

```
git submodule update --init --recursive
hugo server -D
```

Codespace is smart enough to do auto-port mapping and you can access the draft version webpage in a browser to see how it looks. 
