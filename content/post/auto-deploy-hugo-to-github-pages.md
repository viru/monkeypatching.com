+++
Categories = ["hugo", "github", "circleci"]
date = "2016-07-25T20:16:14+02:00"
title = "Auto Deploy Hugo to Github Pages"

+++

Want to know how to create a static website using [Hugo](https://gohugo.io/) and deploy it automatically using [CircleCI](https://circleci.com/) to [GitHub Pages](https://pages.github.com/)? Everything free of charge? Read on!

This post covers exactly the same setup I'm using for this blog, so if TL;DR and you know your way around Hugo, you can jump directly to my [repository](https://github.com/viru/monkeypatching.com).

Hugo is a static website generator. You choose a theme (or create your own), create the content using convinient Markdown format and call Hugo to generate your website. It produces HTML files along with statics ready to upload to your hosting provider's server. Typical workflow looks like this:

```sh
$ hugo new site myblog && cd myblog
$ git clone https://github.com/dplesca/purehugo.git themes/purehugo # or other theme
$ edit config.toml # set theme
$ hugo new post/first-post.md
$ edit post/first-post.md
$ hugo -v
```

Now you should have your static website generated in a `public` directory.

Next we will setup GitHub Pages. First you need to create a repository called `<username>.github.io`. Everything that is inside will be publicly visible under http://<username>.github.io/. So if you create a hello world `index.html` there, it will work right away. If you want to have it under a custom domain, you only have to crate a CNAME record pointing at `<username>.github.io` and commit a `CNAME` file to your repository with your custom domain. You can check for example at my [github pages repository](https://github.com/viru/viru.github.io).

Now the fun part: automation! We need two repositories. We already have `<username>.github.io` which is our destination. But we need a source for it, so let's do `git init` in `myblog` directory we've created earlier with Hugo. We don't want to commit build files inside the `public` directory, so let's remove it and add it to `.gitignore`. Of course we also need to create that repository on GitHub, add a remote and push your files.

Now we have two public GitHub repositories (as our blog website is public anyway and we don't have any secrets / DB credentials that should be hidden). This gives us possibility to also use a public CircleCI project that is free of charge. Our automation workflow will be as follows:

1. Create a new blog post locally.
2. Push changes to the `myblog` repository.
3. CircleCI picks it up and runs `hugo -v` to generate static files.
4. CircleCI pushes them to `<username>.github.io` repository.

CircleCI is configured by the [circleci.yml](https://github.com/viru/monkeypatching.com/blob/master/circle.yml) file in root of the `myblog` repository. It installs Hugo on a build machine, clones the `<username>.github.io` repository to the `public` directory, runs `hugo -v` to generate static files and finally runs [ci-deploy.sh](https://github.com/viru/monkeypatching.com/blob/master/ci-deploy.sh) that adds all new files generated by Hugo, commits them and pushes to the `<username>.github.io` repository.

There's one more step to make it all work... CircleCI needs a write permission to the `<username>.github.io` repository. In order to do that, generate a new SSH key pair with `ssh-keygen`, add the public key as a deployment key to that repository and the private key to CircleCI under `myblog` project settings. It is called "SSH Permissions" and you should provide `github.com` as a hostname for that key.

You're all set now. Commit changes to `myblog` repository and changes will appear on your website few minutes after. Everything based on free GitHub and CircleCI accounts!