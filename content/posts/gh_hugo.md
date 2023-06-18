---
title: "GitHub Pages + Hugo"
date: 2023-06-18T12:00:17+02:00
draft: false
---

### Introduction
In this post we will create a website using Hugo and host it as a GitHub Pages project and automate the whole process with Github Actions.

This is an update from an earlier post I wrote on my Medium blog -> ["Hugo in 10 Minutes"](https://medium.com/@magstherdev/hugo-in-10-minutes-2dc4ac70ee11)



### Prerequisites
A [GitHub account](https://github.com)

[Hugo](https://gohugo.io/installation/)



### GitHub Pages
_[GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages) is a static site hosting service that takes HTML, CSS, and JavaScript files straight from a repository on GitHub, optionally runs the files through a build process, and publishes a website._

In addition to that, it's also free.

The requirements for using GitHub pages in our case are:

- To publish a user site, you must create a repository owned by your personal account that's named `<username>.github.io`.

- The source files for a project site are stored in the same repository as their project.

- You can only create one user or organization site for each account on GitHub.

There are also some limitations outlined [here](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#usage-limits).

GitHub provides a good [quick-start](https://docs.github.com/en/pages/quickstart), so I won't go much in detail about that.
It usually comes down to these steps:

1. Create a repository and name it `username.github.io` as the repository name. (In my case, it will be `magsther.github.io`) The website will be published at https://magsther.github.io/
2. Clone the repository to your computer.
3. Cd into the folder
4. Add an index.html
5. Add, commit, and push your changes
6. Enable and configure Pages (in GitHub)

After a little while, you can visitusername.github.io to view your new website. 

Next step is to setup a GitHub Pages site with `Hugo`.


### Hugo
[Hugo](https://gohugo.io/about/what-is-hugo/) is a fast and modern static site generator written in Go, and designed to make website creation fun again. It is a popular static site generator, designed to provide an optimal viewing experience for your website's end users and an ideal writing experience for website authors.

> A [static site generator](https://www.cloudflare.com/en-gb/learning/performance/static-site-generator/) is a tool that generates a full static HTML website based on raw data and a set of templates.

Hugo works on macOS, Linux, Windows and you can follow the installation instructions for your platform [here](https://gohugo.io/installation/
).
Once the installation is done, we can just go through the [Quick-start](https://gohugo.io/getting-started/quick-start/) guide, which more or less comes down to this:

These are the steps that we will do:

1. Create a Hugo project (site)
2. Choose a theme
3. Install and configure the theme
4. Customising the theme
5. Add content by creating a post
6. Run Hugo


### Creating a Hugo Project (Site)
Start by cloning your repository to your computer.

`git clone git@github.com:magsther/magsther.github.io.git`

Cd into the folder and run the following command:
`hugo new site o11ycloud -f yml # creates the directory structure for your project`


```
Just a few more steps and you're ready to go:
1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".
Visit https://gohugo.io/ for quickstart guide and full documentation.
```

Now we need to add a [theme](https://themes.gohugo.io/) to your Hugo site, which is what we will do next.


### Themes
Hugo provides a list of [themes](https://themes.gohugo.io/) that you can use for your new website. If you are familiar with Wordpress then you will know how useful this is. Let's first to go the themes page and select a theme that you like.

I like the [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) theme, so I'll use that one. 



### Installing a Theme
Follow the [instructions](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation) for the theme on how to install it. You can also watch a [YouTube video](https://www.youtube.com/playlist?list=PLeiDFxcsdhUrzkK5Jg9IZyiTsIMvXxKZP) where the author will guide you through the process.

When you select a theme it will give you instructions about the installation and configuration for that particular theme.
I cloned the repository and copied the PaperMod themes folder into the root themes folder
`git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1`

I also replaced the config.yml file (in the root module) and made some changes to it.

```
baseURL: "http://magsther.github.io/"
title: O11yCloud.com
paginate: 5
theme: PaperMod
...
....
```

### Run Hugo
To view the website locally, we can use Hugo's development server. Just type in hugo server at the prompt, and it will start building the website. At the bottom of the output we can find a link on how we can access the website.

```
Start building sites …

                   | EN
-------------------+-----
  Pages            |  7
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  0
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Built in 32 ms

Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

Open a browser and go to http://localhost:1313


### Making changes on the "fly"
If you open a new terminal and modify some of the content in the config.yml file you will notice that website is updated automatically.


This happened, because the Hugo process was still running in the other terminal and will build any changes that you make. 

```
Change of config file detected, rebuilding site.
2023-06-11 23:15:55.936 +0200
Rebuilt in 63 ms
```

Tips, if you run `hugo server -D` then it will also included any posts that are in "draft" mode.


### Adding content
To add content to your website, I first added a post.md file that was provided in the repository to the archtypes folder. You can think of that file as a template for all future posts. To create a new post, I typed in `hugo new posts/example.md`

This added an example.md file in the content folder.
You can now open this file with your editor to create your blog post.
If the Hugo process is still running and you are using the `-D `flag, then you will see that a new blog post is now added to your website.


Once you are done with the changes to your post, you can change draft: true to draft: false to have it deployed. This is depending on the theme you are using. If you are using the default theme, then Hugo does not publish draft content when you build the site
This should give you an understanding on how to create new blog posts.


### Customizing Hugo
To further customize your new blog, just play around with the configuration file called config.yml located in the root folder.
The first things that you probably want to change are the baseURL, title and the theme

```
baseURL: "https://magsther.github.io/" #This value must begin with the protocol and end with a slash, as shown above.
title: O11y Cloud.com
paginate: 5
theme: PaperMod
```


### Publish your blog to GitHub Pages
When you are happy with how the website works on the local environment you can decide on how you want to host it. Here, we want to publish the blog to GitHub Pages. 

To do that, we need to run hugo to generate the blog. That will create a docs folder that you will need to add an entry for in the config.yml file for.

After that, we add, commit and push them to the GitHub repository.

`git add . && git commit -m 'new hugo blog' -a && git push`

Next, we need to enable GitHub Pages, which we can do by going to our GitHub repository and from there click on `Pages`. Enable it by changing the `Source` from `None` to `Deploy from a branch`

Save your changes.
After a short while, you will see that your blog is now published at https://username.github.io


### GitHub Actions
You may have noticed that a workflow was created for you in GitHub Actions.

If you click on the workflow, you can see a summary of the jobs that are part of this workflow.

In the future, whenever you push a change from your local repository, GitHub will rebuild your site and deploy the changes.


### Updating your blog
To add a new post and publish it on your website, you can follow this workflow:

1. create a new post with `hugo new posts/example2.md`
2. build the site with `hugo`
3. Add, commit and push the changes to GitHub `git add . && git commit -m 'creating my second post' -a && git push`
4. Watch the workflow in GitHub Actions
5. New posts are published at https://username.github.io


### Conclusion
In this post, we looked at Hugo; how to create a blog and how to publish it to GitHub Pages. In the next post, we will use GitHub Pages with a custom domain name.