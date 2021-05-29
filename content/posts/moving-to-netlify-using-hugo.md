---
title: "Moving to Netlify and Using Hugo"
date: 2019-12-08T17:55:56-05:00
---

I've moved my blog from a [DigitalOcean](https://www.digitalocean.com/) droplet running Wordpress to [Netlify](https://www.netlify.com/) running [Hugo](https://gohugo.io/).  Now, I'm very new to both Hugo and Netlify still, but I'll try to explain them at a high level.  

The concept of Netlify is sites built directly from a git repository, with branches being able to be quickly spun up as separate sites for testing and editing before pushing the changes to your primary site.  

Hugo, is a static site generator and framework.  I haven't delved much into static sites in the past, but figured with a site like my blog, which isn't frequently updated, a static site is a great solution for it.  I don't need to have the features of Wordpress, or other larger Content Management Systems, just to write a few posts.  Another big benefit of static sites in general is the performance.  Because the site is static, there is not much for the app/hosting server to do aside from serve the static content.  Of course, that doesn't mean every static site is as boring as mine.  While I can do web site development, I'm not as creative a designer.  

For my blog, the content is updated through a GitHub repository, which integrates very easily with Netlify.  When updates are commited to a pull request branch, Netlify creates a 'preview' site for you to check how things look on their site before merging the changes to be deployed on the main site.  Netlify also offers free SSL certificates through Let's Encrypt, as well as using a custom domain for the site.  Netlify also supports several other technology stacks, including Jamstack, React, and Vue.js, or full platforms like Wordpress, Drupal, and Sitecore.  

When running Hugo locally, the `hugo serve` will create the site locally, and automatically update the site as file updates are made, including refreshing any browser pages you have open when they are updated.  

Hugo and Netlify are both used by major brands and companies.  Hugo is used for the Let's Encrypt site, among others.  Netlify is used by several large companies, such as Mailchimp, Nike, Verizon, Citrix, and Pelaton.  

Overall, I highly recommend Hugo and Netlify for anyone that has a personal site and wants to simplify the hosting.  