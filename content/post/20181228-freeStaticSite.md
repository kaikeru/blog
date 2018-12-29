---
title: "Free Website Hosting with Github and DevOps"
date: 2018-12-28
tags: ["azure pipelines", "github", "hugo", "devops"]
draft: true
---

Paying for webiste hosting is passé. You can easily host with Wordpress, Tumblr, or many other solutions. The downside to these free hosted services (applications, really) is  free means you're the product one way or another. Maybe they put adds on your blog, maybe they sell your data. Also, what happens if that company goes under? Where does your corner of the internet go?

I'm a proponent of not relying on other services to build my content if I can help it. I want to build my site so if any service I rely on goes under, I can shift to a new service with minimal fuss. It is more work, but the payoff is worth it to have a permanent home on the internet.

I created this site, the one you are reading, using free tools and hosting...or about as free as you can get on the internet without owing your soul to one specific company. Sure, some of the services might change so they are no longer free, but there are an abundant amount of alternatives available.

I like to use static sites where possible rather than dynamic web applications. Static sites don't use runtimes, databases, or other nasty exploitable services. They are simple. [Hugo](https://gohugo.io) is my static site generator of choice.

Next, we need a good place to the static site. Free and ad free hosting is best. Luckily, if you don't mind having the entire site public, you can use [Github Pages](https://pages.github.com). This is a free hosting service from Github that allows for static sites or Jekyll sites. I'm not a huge fan of Jekyll for Ruby reasons, but I'm bias. If you are using Jekyll, you can pretty much skip the rest of this article since [Github has you covered](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/). If you are using Hugo or some other static site generator, we need a good way to get the static site into the Github Pages repo. Because my background is in DevOps, let's add some (simple) [CI/CD](https://en.wikipedia.org/wiki/CI/CD) using [Azure Pipelines](https://azure.microsoft.com/en-us/services/devops/pipelines/).

## Github

### Get Going with Git
The basic container for our site, like all fun things, is a git repo. I'm using [Github](https://github.com/) for my cloud-based git hosting needs. Github's getting started with git tutorials are much better than anything I could write so start there. Sign up for Github and get started with git.

As a side note, Github is free if you are open sourcing your code. If you insist on not doing that but still don't want to pay, take a look at [Gitlab](https://gitlab.com). In either case, I recommend using [SSH key authentication](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).

### Create the Repo
Once everything is setup on the git side, create the site's new repo on Github. Click the `+` button and select 'New Repository'. Name the thing and finish the setup. Github will show instructions on how to clone that repo locally. If you are uncomfortable with command line interfaces, now's a great time to practice since this article won't use any GUIs outside of the Github and Azure websites.

Open up the terminal. Navigate to the parent directory where the repo is to be cloned.

```bash
# Clone our sites repo.
cd ~/repositories
git clone git@github.com:kaikeru/blog.git
```

This gives us our slate in which to chisel out our blog.

## Hugo

### Setup
[Hugo](https://gohugo.io) is a great static website builder that's simple, fast, and fairly intuitive. There are a fair amount of good tutorials about how to setup hugo and to get started. Try out the official [Quick Start](https://gohugo.io/getting-started/quick-start/) tutorial. We only care about having a functional compiled site directory that we can upload to Github Pages.

After following the _Hugo Quick Start_, we should have a working Hugo executable to run on our new repository. We want to create a site in our repo directory. To do that, move into your repo and create a new hugo project with the following. Once complete, continue on with the _Hugo Quick Start_.

```bash
# Create a new Hugo blog in the cloned repo.
cd ~/repositories/my-blog
hugo new site .
```

Woo! We have a new site to work with! The directory structure should be similar to below.

```txt
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

### Theme

A theme for Hugo may have been installed during the _Hugo Quick Start_ tutorial, but I'm using a forked version of the [Minimal](https://github.com/calintat/minimal) theme. We want to add this theme as a submodule in our `themes/` directory.

```bash
# Add a theme for us to use.
cd themes
git submodule add git@github.com:kaikeru/minimal.git
```

Now our theme will be pulled when we clone the parent blog repo.

Finally on the theme front, we need to make sure our `config.toml` has the right settings. The minimal config for the "Minimal" theme needs a few things. The main thing is a reference to the theme in the config.

```toml
# Our default config file.
baseURL = "http://localhost"
languageCode = "en-us"
title = "kaikeru"
theme = "minimal"

[author]
  homepage = "https://github.com/kaikeru"
  name = "Kyle Wagner"

[params]
  author = "Kyle Wagner"
  description = "Thoughts on happiness"
  githubUsername = "kaikeru"
  accent = "#2A603B"
  showBorder = true
  backgroundColor = "white"
  font = "Raleway" # should match the name on Google Fonts!
  highlight = true
  highlightStyle = "solarized-dark"
  highlightLanguages = ["java", "python", "yaml"]

[[menu.main]]
  url = "/"
  name = "Home"
  weight = 1

[[menu.main]]
  url = "/post/"
  name = "Posts"
  weight = 2
```

### Content

We want to add some simple content if it doesn't already have some.

```bash
# Create a new post with content.
hugo new post/test.md
echo "I'm a test" >> content/post/test.md
```

### Start the Test Server

Everything for Hugo should be ready. Testing this is easy.

```bash
# Start the Hugo test server.
 hugo server -D
```

This will output the server address. Most likely will be `http://localhost:1313`. Paste that into your browser, and you should see a page similar the home page of this blog.

The run server action compiles the site as well. The static content now lives in the `public/` directory. If we want to compile the site without the server, we just need to run the `hugo` command at the top level of our repo.

With the static content built, the content can be used to populate our Github Pages site. Before we do that, let's do some git housekeeping by adding the `public/` folder to our `.gitignore` so the folder doesn't accidentally creep into our repo. Create a `.gitignore` file if it doesn't exist and add a line to ignore the `public/` directory.

```.gitignore
# Ignore the public folder
public/
```

## Github Pages

With Hugo working, we have some content to put in our Github Pages site. Pages works on the premise we have a repository on Github named with the convention `<github-user>.github.io`. For my page I need a repo named `kaikeru.github.io`. We follow the exact same steps as creating the blog repo. We want to clone this repo as well but not into the blog repo. Put it in the `~/repositories` equivalent.

The content from the Hugo site needs to be placed in the Github Pages repo. This article uses `rsync`, but any other way to copy all the files is fine.

```bash
# Copy all static content to the Github Pages repo.
cd ~/repositories/my-blog
rsync -av public/* ~/repositories/kaikeru.github.io
```

The repo is ready for us to commit all the files to the repo.

```bash
# Add files to the commit.
cd ~/repositories/kaikeru.github.io
git add .
git commit -m "Add static content"
```

Our site is ready and primed for the public world. Push the commit up to Github and we should see the site at our Github Pages URL. The URL, for me, is https://kaikeru.github.io.

```bash
# From the Github Pages repo.
git push origin master
```

## Add Some DevOps

This entire process works just fine for a simple blog, but like any task, if we do it more than twice, then automation of that task is in order.

Let's break down the task into a pipeline that can be used to as a basis for our automation. The tasks that are always the same happen after we finish creating new content for the site.

_Repeated Tasks_

1. Compile the site with the `hugo` command.
2. Copy the static files to our Github Pages repo.
3. Add files to commit.
4. Commit the files.
5. Push commit to Github.

These steps never change.

### Azure Pipeline

It is common to use CI/CD tools to automate repetitive tasks and deploy that code to a production environment. A few popular tools are Gitlab CI/CD, Travis CI, Jenkins, AWS Pipelines, and Azure Pipelines. Most of these have incredibly good integrations with Github and will all do the job well. I choose Azure Pipelines more as an exercise in learning a cool new tool for myself than any other reason. One of the plus sides is Azure Pipelines are free if your project is open source. Gitlab and Travis have this benefit as well.

#### Setup Pipeline Add-On

First thing we need to do is add a trigger to our build process. We want to trigger whenever a commit goes into the `master` branch on our blog repo. Github makes this easy to do through the [Azure Pipeline add-on](https://github.com/marketplace/azure-pipelines) available in the Github Marketplace. Add this to your account using the free plan and that's pretty much it for setting up the pipeline. A pipeline will now trigger if we have an `azure-pipelines.yml` in our repo. Let's add that.

#### Create Pipeline Configuration

The `azure-pipelines.yml` dictates how the pipeline works. Each step needed for the build and deployment is described in that file.

```yaml
# A simple pipeline for Hugo builds in Azure.
pool:
  vmImage: 'Ubuntu 16.04'

variables:
  hugoUrl: https://github.com/gohugoio/hugo/releases/download/v0.51/hugo_0.51_Linux-64bit.tar.gz
  hugoTar: /tmp/hugo.tgz

steps:
- script: |
    wget -q -O $(hugoTar) $(hugoUrl)
    tar xzvf $(hugoTar)
    sudo rsync -av hugo /usr/local/bin/
  displayName: 'Install Hugo'
- script: |
    hugo
  displayName: 'Build Hugo Blog'
- script: |
    git config --global user.email "devops@kaikeru.com"
    git config --global user.name "Azure Build"
    git clone https://$(github.username):$(github.token)@github.com/kaikeru/kaikeru.github.io.git /tmp/blog
    rm -rf /tmp/blog/*
    rsync -av public/* /tmp/blog/
    cd /tmp/blog
    git add .
    git commit -m 'Autocommit from $(Build.DefinitionName) $(Build.BuildNumber)'
    git push origin
  displayName: 'Commit to kaikeru.github.io'
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master'))
```

The whole pipeline is broken down into three steps and uses an Ubuntu 16.04 virtual machine image as its base. Going through the steps, we start with the installation of Hugo on the VM.

```yaml
- script: |
    wget -q -O $(hugoTar) $(hugoUrl)
    tar xzvf $(hugoTar)
    sudo rsync -av hugo /usr/local/bin/
  displayName: 'Install Hugo'
```

The first thing we need to do is get the Hugo image. The tarball is unpacked and installed into the `/usr/local/bin` directory so that the Hugo binary is picked up on the `PATH` by the shell.

```yaml
- script: |
    hugo
  displayName: 'Build Hugo Blog'
```

Now that we have the Hugo binary available to our scripts, we run the `hugo` command to compile the static website into the `public/` directory.

```yaml
- script: |
    git config --global user.email "devops@kaikeru.com"
    git config --global user.name "Azure Build"
    git clone https://$(github.username):$(github.token)@github.com/kaikeru/kaikeru.github.io.git /tmp/blog
    rm -rf /tmp/blog/*
    rsync -av public/* /tmp/blog/
    cd /tmp/blog
    git add .
    git commit -m 'Autocommit from $(Build.DefinitionName) $(Build.BuildNumber)'
    git push origin
  displayName: 'Commit to kaikeru.github.io'
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master'))
```

The final step seems like it does a lot but most of it's fluff for git. We are mainly pushing our new static site to the Github Pages repo.

Easily missable parts are `$(github.username)` and `$(github.token)` in the clone action. Github allows us to create an OAuth token so we can clone and push to the repository. Navigate to https://github.com/settings/tokens on Github to create the token. It will need the `public_repo` OAuth scope. Save this token in a secure place. __You can't see it again on Github once you leave the page.__

Both `github.username` and `github.token` must be entered on the Azure Pipeline site. Take a look at how to [add secret variables](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=vsts&tabs=designer%2Cbatch#secret-variables) to your pipeline. You may need to have your pipeline run once (and fail) before you're given the option to add the variables. The names of the variables should be `github.username` and `github.token` with their corresponding values.

Notice that the last line is the `condition` on which this step will run. The triggering conditions are if the previous steps succeed and the branch for this build is `master`. This means any pull requests or other branches will not push their build results to the Github Pages repo.

We need to commit the `azure-pipelines.yml` to the repo and push the commit up. If everything is done properly, we can follow the build on Github's branches page for your blog's repo.

## Conclusion

The current site you are reading is built on the steps outlined above. There are a few other modifications to allow for some better performance. [Cloudflare's](https://www.cloudflare.com/) caching, SSL, and DNS provide more stability under load or attacks. That is probably overkill, but it's also free to use. I've also aliased the kaikeru.github.io to blog.kaikeru.com using a CNAME entry.

I hope this was helpful. Please contact me on  Twitter (see the link in the header) if you have any questions or suggestions.

