---
author: Kent Hua
comments: true
date: "2016-06-30T16:42:00Z"
modified_time: "2017-04-10T17:30:00.000-07:00"
title: Migration from blogger to Jekyll (hosted on github)
---

I've updated the underlying blog hosted on blogger to jekyll hosted on github.  
Migrating the content was fairly easy.  Unfortunately I wasn't able to migrate previous comments, it will now be hosted on disqus.  The one advantage I have now is being able to write content with markdown instead of 
html which made things a bit more difficult.  Plus everything is just a file now.
I will eventually start using the jekyll provided `rogue` feature for syntax highlighting. 

Eventually I'll look into some theming as well, or leave it as is, nice and clean. 

  

0. Export your blog under settings and backup content (from blogger) [linky](https://support.google.com/blogger/answer/41387?rd=1)
0. Use the jekyll import blogger format [linky](https://import.jekyllrb.com/docs/blogger/) 
0. Create your page on github [linky](https://pages.github.com/)
0. Follow setup instructions [here](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)
0. `_config.yml` is your friend
0.  `bundle exec jekyll serve` is also your friend for local testing of your content
0. Current gems so far
  * jekyll-feed
  * jekyll-seo-tag
  * jekyll-sitemap
  * jekyll-mentions
  * jemoji
0. Setup disqus for commenting [linky](https://help.disqus.com/customer/portal/articles/472138-jekyll-installation-instructions)    
0. Update your site to the latest gems
  * `bundle update` or `bundle update github-pages`
0. Push your site
  * `git add *`, `git commit -m 'updates'`, `git push origin master`

* A good [link](https://help.github.com/categories/customizing-github-pages/) for specific jekyll functions supported within github
