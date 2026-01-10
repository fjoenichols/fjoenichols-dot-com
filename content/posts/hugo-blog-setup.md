+++
title = "Hugo Blog Setup"
date = 2026-01-10T12:00:00-07:00
draft = false
categories = ["Knowledge Log"]
tags = ["tech", "hugo"]
+++

Setting up a Personal Site Using Hugo and Cloudflare Pages

To get a site up quickly I am going to Hugo 

## Installing Hugo for Local Development
> [!NOTE]
> The instructions provided in this document will be for Ubuntu 24.04. 
> Installation instructions for other operating systems are available [here](https://gohugo.io/installation/). 

Following best practices of not testing in production, I will be setting up a local development environment for my Hugo site. Step-by-step instructions are provided below.

### Installing Hugo
Luckily there is already an apt package for Hugo so no need to add additonal repositories or anything like that. Just simply install using apt. As of this writing the version installed via apt is 0.151.

``` bash
$ sudo apt install hugo
```

### Create your site directory
Hugo's cli allows for the easy creation of new sites using the new site command. I will be naming my site directory fjoenichols-dot-com because the domain for my website is fjoenichols.com.

``` bash
$ hugo new site fjoenichols-dot-com
```

> [!NOTE]
> If you prefer to use yaml for configuration instead of toml you can append the new site command with `--format yaml`
> `hugo new site <your-site-name> --format yaml`

Once the site directory is created it should look something like the following

``` bash
$ ls
archetypes  assets  content  data  hugo.toml  i18n  layouts  static  themes
```

### Add a Theme
The first thing you will want to do after creating the site directory is to identify a suitable theme for your Hugo site. There are a number of prebuilt themes available from the Hugo community at [themes.gohugo.io](https://themes.gohugo.io/).

I will be using the [PaperMod](https://themes.gohugo.io/themes/hugo-papermod/) theme as it is fairly minimal and I like the style.

The recommended way of installing PaperMod is as a git submodule. Since I'm planning to keep the site's code on GitHub I will go ahead and initialize the site directory as a git repository.

``` bash
$ git init
Initialized empty Git repository in /home/jnichols/projects/hugo/fjoenichols-dot-com/.git/
```

Once I have initialized the git repository I can install the theme as a git submodule.

``` bash
$ git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
$ git submodule update --init --recursive
Cloning into '/home/jnichols/projects/hugo/fjoenichols-dot-com/themes/PaperMod'...
remote: Enumerating objects: 140, done.
remote: Counting objects: 100% (140/140), done.
remote: Compressing objects: 100% (99/99), done.
remote: Total 140 (delta 37), reused 115 (delta 36), pack-reused 0 (from 0)
Receiving objects: 100% (140/140), 256.53 KiB | 1.26 MiB/s, done.
Resolving deltas: 100% (37/37), done.
$ git submodule update --remote --merge
```

To use the theme with the site I need to edit the hugo.toml file to add the theme. While in the hugo.toml file I will go ahead and set the `baseURL` and `title`.

> [!NOTE]
> Some theme installation instructions might say to edit config.toml. 
> As of version 0.110, hugo.toml is the default config file.

``` toml
baseURL = 'https://fjoenichols.com/'
languageCode = 'en-us'
title = 'F. Joe Nichols'
theme = ["PaperMod"]
```

### Switching PaperMod to Profile Mode
To use this site as a profile for myself I want to use profile mode which allows me to have an avatar, social links, and a subtitle under my name. To do this some more changed need to be made to hugo.toml.

Under everything in hugo.toml I am adding the following lines:
``` toml
[params.profileMode]
    enabled = true
    title = "F. Joe Nichols"
    subtitle = "SRE Manager | Agency Owner | 21-Day Sprint Builder"
    imageUrl = "images/profile.jpg"
    imageWidth = 120
    imageHeight = 120

[[params.socialIcons]]
    name = "github"
    url = "https://github.com/fjoenichols"
[[params.socialIcons]]
    name = "youtube"
    url = "https://www.youtube.com/@fjoenichols"
[[params.socialIcons]]
    name = "linkedin"
    url = "https://www.linkedin.com/in/fjoenichols/"
```

I also need to create an `images` directory in `static` and add a profile picture there as specified in `imageUrl`. 

``` bash
$ pwd
/home/jnichols/projects/hugo/fjoenichols-dot-com/static/images
$ ls
profile.jpg
```

### Adding Taxonomies for 21-Day Sprints 
To support my 21-Day Sprint methodology, I need to tell Hugo and PaperMod how to group my posts. I'll add the following to hugo.toml:

``` Ini, TOML
[taxonomies]
    category = "categories"
    tag = "tags"
    sprint = "sprints"
```

### Adding Menu Items to the Nav Bar
I also want these to be easily accessible from the main navigation bar:
``` Ini, TOML
[[menu.main]]
    name = "Knowledge Log"
    url = "categories/knowledge-log"
    weight = 10
[[menu.main]]
    name = "21-Day Sprints"
    url = "categories/21-day-sprints"
    weight = 20
```

## Deploying to Cloudflare Pages
Now that the local environment is verified using hugo server -D, it's time to push to production.

### Push to GitHub
Cloudflare Pages works best with a git-integrated workflow.

``` bash
$ git add .
$ git commit -m "Initial site setup with PaperMod"
$ git remote add origin https://github.com/fjoenichols/your-repo-name.git
$ git push -u origin main
```

### Configure Cloudflare Pages
This is a lot of point and click. 

I was looking for Workers & Pages and I couldn't find it so I search for Pages instead and then it popped up. 

Then I clicked on Create application and `Continue with GitHub`

Selected my repository and set the following Build settings:

* Framework preset: Hugo
* Build command: hugo --gc --minify
* Build output directory: public
* Environment Variables: Add HUGO_VERSION and set it to 0.151.0 (or whatever version you are running locally) to ensure the build environment matches your dev environment.

After hitting Save and Deploy, Cloudflare provisioned a container, installed Hugo, and give me a pages.dev URL. From this point I mapped the page to fjoenichols.com via the "Custom Domains" tab.