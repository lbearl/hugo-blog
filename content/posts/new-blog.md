+++ 
draft = false
date = 2024-09-17T20:16:27-05:00
title = "New Blog"
description = ""
slug = "new-blog"
tags = ["blog", "general"]
categories = []
+++

# New Blog

I decided to move off of WordPress and move to a static blog. Most of the time I prefer writing things in Markdown, so moving to a solution where everything is natively in Markdown makes a lot of sense to me. Before settling on Hugo I looked at a few other options (including of course contemplating writing a generator in C#, but that would be a terrible case of bikeshedding). 

## Why Hugo?

Hugo has a benefit over a lot of other static blog generators (like Jekyll or Gatsby or one of the others out there), namely:
 
  1. It's mature enough where it can be trivially hosted via basically any service which can host static pages (I'm using CloudFlare pages)
  2. It has a wide variety of free themes available (no one wants to see what happens when I try to make a design)
  3. The hugo binary is both fast and space efficient (one of the biggest advantages Go has over .NET is still how small the binaries can be)

## How it is setup

Setup was pretty simple. Just follow the [docs](https://gohugo.io/getting-started/quick-start/) on the Hugo website to install Hugo:

  1. Install Hugo (`brew install hugo` if on a Mac)
  2. Create a new Hugo project `hugo new site <your project name>`
  3. `cd` into the directory made by the previous step
  4. Open your text editor of choice in the directory (i.e. `code .`)
  5. Find a theme (I'm using [hugo-coder](https://github.com/luizdepra/hugo-coder))
  6. Follow the steps to install the theme as a git submodule
  7. Edit the `hugo.toml` file to set things up. Reference the theme documentation to make sure you have everything necesssary
  8. Run `hugo server -D` and make sure everything looks good
  9. Add the blog into Github (other SCM solutions may or may not work depending on where you are hosting)
  10. In CloudFlare go to "Workers and Pages" > "Pages" > "Connect to Git"
  11. Add your repository to CloudFlare
  12. Set the build command to `hugo -b $CF_PAGES_URL`, the build output directory to `/public` and set the production branch to `main`
  13. Depending on your theme, you may need to set the `HUGO_VERSION` env var to `v0.134.0+extended`
  14. Deploy the site, wait for it finish.
  15. Go back to the site, and set the custom domain (assuming you have your domain already with CloudFlare)

## Migrating old content

I have a number of blog posts that were in the previous WordPress install, and I really didn't want to lose them. Thankfully, someone wrote a fantastic little tool which can export a wordpress export.XML file to markdown (which Hugo consumes with minimal additional editing).

  1. Go to your old wordpress site and find the Export option
  2. `npx wordpress-export-to-markdown --input=export.xml --post-folders=false --save-attached-images=true --save-scraped-images=true` will pretty much do everything for you. The one big issue I did encounter was that images with the same file name will overwrite each other (i.e. if you have PostA and PostB that both have a different image named img1.jpg, whichever image is downloaded last wins).

## First Impressions

Overall I'm happy with my choice to migrate to Hugo so far (granted this is the first post I've written since the conversion). I also added a few things that are a bit more fresh like my [about](/about) page or my [now](/now) page which will give you a bit more information about who I am. I also re-did my resume with [jsonresume](https://jsonresume.org).