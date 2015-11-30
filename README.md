This repo holds the source for my blog, http://www.mfoot.com.

It uses the static site generator Jekyll to generate an HTML site from the list of posts. There are
two directories in the root: `jekyll` and `org`.

`jekyll` is the folder structure that Jekyll expects. It has some of my older posts in the `_posts`
subdirectory.

`org` is the more modern section. I switched to using Emacs' org-mode to write blog posts. The
directory structure here is similar to `jekyll`'s but contains org mode posts instead of yaml
posts. I use org mode's HTML export functionality to copy HTML exports of the posts into the
`jekyll` folder, and Jekyll understands both `.yaml` and `.html`. files when it generates the site.

More information on this can be found here:

http://www.mfoot.com/blog/2015/11/17/using-org-mode-to-write-jekyll-blogs/
