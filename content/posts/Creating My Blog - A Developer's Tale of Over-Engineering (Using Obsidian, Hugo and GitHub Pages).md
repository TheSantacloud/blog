---
timestamp: 2024-08-10T20:48:37+03:00
modified: 2024-08-10T20:48:37+03:00
draft: "false"
title: "Creating My Blog: A Developer's Tale of Over-Engineering (Using Obsidian, Hugo and GitHub Pages)"
creation_date: 2024-08-10T20:48:37+03:00
---
## Intro

[TL;DR:](#dead-eyed-tutorial) If you just wanna know how to do this, and don't care about the actual post.

I've wanted to start a blog for quite a while now, and never got around to it because of various nonsense excuses. When I finally decided to start, I struggled to choose the content for the first post. And so I did what every decent software developer would do when they begin: over-engineered the shit out of the blog itself rather than focus on what matters - the content. But this over-engineering was quick, integrated smoothly with my workflow, was relatively isolated and not super opinionated, and presented an integration issue that, once solved, ended up being pretty damn convenient.

*Lightbulb* - this would be the topic of my first post! Instead of just a tutorial, it will be a walkthrough on how an actual development process of something pretty simple actually goes and how I personally approach things (I'll save the obligatory 'what I think about all this AI shenanigans' post for later. This is not LinkedIn after all.)

**P.S.:** I'm well aware that there are probably more blog creation platforms available out there than the amount of stars in the visible universe. And I guess that with some research I would probably find something pretty damn fantastic. But I wanted do it myself because of reasons. So there. 

## What I want

I want a blog that will blend seamlessly with my daily driver for note taking, and feel seamless and smooth.

**My requirements:**
1. Use [Obsidian](https://obsidian.md/) exclusively for writing, editing, and managing posts. No extra [vaults](https://help.obsidian.md/Files+and+folders/Manage+vaults). The blog should integrate smoothly with my existing workflow (because I have automatic backups and versioning, and because I said so.)
2. Nice looking blog - plain HTML with won't cut it because of my Obsidian requirement, so I need a markdown parser of some sort.

## How I chose the tools 

1. [Obsidian](https://obsidian.md/) - My go-to for all note-taking. I use a [Zettlekasten](https://zettelkasten.de/introduction/)-esque method, dumping all notes into one directory with timestamps and links. It's markdown-based and has vim-mode, plugins, [internal linking](https://help.obsidian.md/Linking+notes+and+files/Internal+links) and [searching](https://help.obsidian.md/Plugins/Search) capabilities. This made it a no-brainer for me.
3. [GitHub Pages](https://pages.github.com/) - I've been using [GitHub](https://github.com) for years, so it was the obvious choice for hosting and publishing. Static pages fit my needs perfectly, avoiding unnecessary complexity. I was thinking about hosting it on one of my Raspberry PIs I have lying around but then I figured... nah.
4. [Hugo](https://gohugo.io/) - This Go-based website framework won out after some research (Googling "Github static blog serving" and then "Jekyll vs. "). I considered [Jekyll](https://jekyllrb.com/), [11ty](https://www.11ty.dev/), and [Astro](https://astro.build/), and since this is a website building framework, I was pretty sure Hugo's use of Go will use its native templating language. Plus, Go > Ruby, tsx, and JavaScript in my book. 

## Hello blorgd (nailed it) - initial setup

Nice, We've got the tools. Let's put them to use. 

A [quick intro to Hugo](https://gohugo.io/getting-started/quick-start/) yielded the following to get started:

```bash
hugo new site quickstart
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

But hold on a minute, we can't have just any old regular theme. A quick search got led me to [this lovely gallery](https://themes.gohugo.io/tags/blog) to choose from. [hugo-blog-awesome](https://themes.gohugo.io/themes/hugo-blog-awesome/) looks nice, scrolling down the page a bit we get how to install it instead of that `ananke` theme (which I didn't even look at **once**, because fuck default values). Nice.

```bash
hugo mod get github.com/hugo-sid/hugo-blog-awesome # good old Go
```

and also adding this to the `hugo.toml` file:
```toml
[module]
  [[module.imports]]
    path = "github.com/hugo-sid/hugo-blog-awesome"
```

Now that I've set this up, I wandered around the directory to understand the file structure:

```bash
➜ website (main) ✗ ls -lh
total 16
drwxr-xr-x@ 16 santacloud  staff   512B Jul  6 23:20 .
drwxr-xr-x  24 santacloud  staff   768B Jul  6 19:58 ..
drwxr-xr-x@  3 santacloud  staff    96B Jul  6 21:39 archetypes
drwxr-xr-x@  2 santacloud  staff    64B Jul  6 19:58 assets
drwxr-xr-x@  2 santacloud  staff    64B Jul  6 21:45 content
drwxr-xr-x@  2 santacloud  staff    64B Jul  6 19:58 data
-rw-r--r--@  1 santacloud  staff   1.3K Jul  6 23:20 hugo.toml
drwxr-xr-x@  2 santacloud  staff    64B Jul  6 19:58 i18n
drwxr-xr-x@  4 santacloud  staff   128B Jul  6 23:26 layouts
drwxr-xr-x@ 17 santacloud  staff   544B Jul 14 21:21 public
drwxr-xr-x@  3 santacloud  staff    96B Jul  6 20:03 resources
drwxr-xr-x@  2 santacloud  staff    64B Jul  6 19:58 static
drwxr-xr-x@  3 santacloud  staff    96B Jul  6 22:34 themes
```

Quickly visiting some directories showed me that:

- `content` is where the markdown files go.
- `hugo.toml` contains the site configuration.
- `themes` is for theme-related files.
* `archetypes` is interesting. It has a `default.md` file with some following content:
```toml
+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName `-` ` ` | title }}'
+++
```
This is interesting. Because Obsidian has a similar metadata section called [properties](https://help.obsidian.md/Editing+and+formatting/Properties). I should make a note of this.
* `layouts` has a single directory called `_default`, with a single item called (conveniently) `single.html` that looks like HTML (duh) and our beloved Go template (called it!). This looks like the template for the actual blogpost page. Oki-doki
- The other directories are currently empty or not immediately relevant.

This directory exploration helped me understand how Hugo organizes its files and gave me a head start on integrating it with Obsidian.

### Accumulating experience

There's a reason I examined the directory structure, even though it wasn't immediately necessary.

Throughout our careers we encounter many tools and frameworks, and we need to use, create and integrate with them. This process helps us develop a sense for how things work. That's raw experience right there, but it needs to be indexed. Ultimately, this is what helps us estimate better, break down tasks easier and be more confident.

Understanding a tool's structure **before** reading the documentation (notice how I didn't say instead?) helps me develop an intuition about how it works and what the developer's intentions were because at this point in time, the docs are just a bunch of words and terminology I don't know yet. The more I do this, the better I get at predicting how tools function and how to operate them.

I've seen many developers ask, "How would you know that? We've never seen this before." The (annoying) answer often is, "I just had a feeling." (or "I don't know.") This is why. It's not because they know everything but because they've developed an intuition from past experiences.

## Obsidian integration

OK we got something that looks like something. And I think that I have a rough understanding of how Hugo works (enough to get started). But before that - I formulate a rough plan in mind:
1. I'll create a dedicated directory for the blog in my Obsidian vault
2. Find some plugin/script that allows me to automatically move notes that are tagged (or something of that sort) to that directory automatically
3. I'll [symlink](https://en.wikipedia.org/wiki/Symbolic_link) `./content` and my `./obsidian-vault/blog` directory (remember this moment)
4. Find some way to inject the necessary metadata to each note (probably using Obsidian [templates](https://help.obsidian.md/Plugins/Templates) that I've been using for a long time)
5. Automate the publish process using [GitHub Actions](https://docs.github.com/en/actions)

Cool. Now it's time for the interesting part - integration.

### First page

Now let's create a dummy markdown file. But I need to understand what kind of file... I have a rough idea, but lets just `hugo --help` just in case.

```
➜ website (main) ✗ hugo --help
hugo is the main command, used to build your Hugo site.
...
Available Commands:
  completion  Generate the autocompletion script for the specified shell
  ... lots of commands
  new         Create new content for your site
  server      A high performance webserver
  version     Print Hugo version and environment info
Flags:
  -b, --baseURL string             hostname (and path) to the root, e.g. https://spf13.com/
  -D, --buildDrafts                include content marked as draft
  ... lots of flags
  -w, --watch                      watch filesystem for changes and recreate as needed
```

Nice. I like good help menus.
First of all - I learned from the flags that in order to see the drafts (I remember seeing this keyword in the archetypes) I need to pass on the `-D` flag to the `hugo server -D`. Alright, good to know.

Anyway - `new` looks like the right thing to do. And indeed this seems to do the job.
```bash
hugo new content first-post.md
cat ./content/first-post.md
```

Shows the following:

```toml
+++
date = '2024-07-14T22:20:52+03:00'
draft = true
title = 'First Post'
+++
```

OK. I understand, it uses the default `archetype` of the single page.

Serving the page (like we did before when we first started Hugo) shows the post, and adding some markdown down seems to update it. Nice.

### First page using Obsidian

OK, we got a page. Now lets just make a directory in my obsidian vault (which is easy enough), and symlink that.

```bash
mkdir /location/of/obsidian-vault/blog # create the directory in my vault
mv ./content/first-page.md /location/of/obsidian-vault/blog # moving the existing page to that directory
rm -r ./content # removing the existing one before creating the symlink
ln -s /location/of/obsidian-vault/blog ./content # creating the symlink
```

Nice. Let's serve and see the page.

Aaaand.... It doesn't work. I mean, it does - the site is served, but there are no pages there.

### Solving the issue

This is why I wanted to write this blog post- and not just another [dead-eyed tutorial](#dead-eyed-tutorial). To me, this is the essence of programming: solving unexpected issues.

There were no logs or errors (even when using debug logs), **yet it didn't work**. This moment is crucial in programming. Developers often create tools without anticipating every possible use case, leading to unexpected issues.

When I encounter these situations, it's important to:
1. **Isolate the unknowns**
2. **Revisit plans and adjust**

#### Isolating unknowns

I gotta admit, I cheated here a little bit.

In those drawing tutorials, artists start with basic shapes and build from there. When you watch them work, they skip these steps because it's already ingrained in them. They see the whole picture.

When I broke down the steps of creating this blog, I kinda did the same thing. Notice the pattern:
1. Install Hugo
2. Learn just enough to serve it
3. Ensuring I actually like it (theme, folder structure)
4. Creating a first page with Hugo in the recommended and well documented way
5. Move that same first page to another (Obsidian) directory and symlink it

These steps might seem trivial, but many developers jump straight into coding without testing along the way. This leads to too many unknowns piling onto one another, making debugging a nightmare. When integrating tools, people often overlook the connection as a component of complexity. In this case, we have Hugo, Obsidian, and the connection between them—those are three points of failure, not just two!

The issue arose when integrating Obsidian. Let's gather the facts:
1. Hugo worked with its default setup
2. The issue occurred when I tried to use a symlinked directory.
3. Obsidian wasn't actually used yet

So what went wrong? **the symlink**.
1. Google searches yielded nothing.
2. ChatGPT is almost never helpful in these situations (especially when you can't really describe what's wrong).

If Hugo isn't the problem, it might be macOS handling symlinks or Go itself.
1. **Permissions**: Gave all permissions with `chmod 777 <location>`—Nothing.
2. **Go handling symlinks**: Googling "Golang symlink" led to [this result](https://stackoverflow.com/questions/18062026/resolve-symlinks-in-go), showing Go's way to follow symlinks with `filepath.EvalSymlink` or `os.Readlink` (after checking with `os.Lstat`):

```go
filePath := "path/to/your/file"

fileInfo, err := os.Lstat(filePath)
if err != nil {
	// error reading file
	return
}

target := filePath
if fileInfo.Mode() & os.ModeSymlink != 0 {
	// It is a symbolic link
	target, err := os.Readlink(filePath)
}

data, err := ioutil.ReadFile(target)
if err != nil {
	return nil, err
}
```

_(Illustrative, not actually implemented)_

I didn't need to know these details in Go, just that reputable languages have ways to handle symlinks. I could dive into Hugo's code or use my experience.
Since it didn't work, I assumed:
1. Either there's a flag for following symlinks, or there's no support.
2. Or there's a built-in way to do it.

This is the moment my experience "betrayed" me. I **know** about symlinks, which should solve the problem seamlessly. But this way was not supported. This doesn't make my experience irrelevant. I now have a new marker in my knowledge of "linking between files and directories."

My options:
1. Using a builtin function - preferable if found
2. Automate copying files from Obsidian—not ideal due to more moving parts and potential sync issues.
4. Use [gitmodules](https://git-scm.com/docs/gitmodules) - this would require me to manage my blog posts in a separate git, and have an automation to update it. Lets leave this as an absolute last resort

The assumptions rabbit-hole ends here. **Time to read the docs**. I'm looking for a flag for symlinks/shortcuts/modules/links/mounts or ways to import another directory. 

The docs' side-panel showed (along many more items):
1. **Content management** - Mainly about formatting.
2. **Hugo modules** - Interesting! 
3. Hosting and deployment - remember for later. 

> **Hugo Modules** are the core building blocks in Hugo. A _module_ can be your main project or a smaller module providing one or more of the 7 component types defined in Hugo: **static**, **content**, **layouts**, **data**, **assets**, **i18n**, and **archetypes**.

Yep, that’s it. Configuring a content module within `hugo.toml`:

```toml
[module]
  [[module.mounts]]
    source = '/path/to/source/content'
    target = '/path/to/target/content'
```

Got it.
### Revisit plans and adjust

- [x] I'll create a dedicated directory for the blog in my Obsidian vault
- [x] ~~I'll [symlink](https://en.wikipedia.org/wiki/Symbolic_link) `./content` and my `./obsidian-vault/blog` directory~~  Mount my blog source directory to Hugo
- [ ] Find some way to inject the necessary metadata to each note (probably using Obsidian [templates](https://help.obsidian.md/Plugins/Templates) that I've been using for a long time)
- [ ] Find some plugin/script that allows me to automatically move notes that are tagged (or something of that sort) to that directory automatically
- [ ] Automate the publish process using [GitHub Actions](https://docs.github.com/en/actions)

### Automatically injecting the metadata 

OK now that I know that this works, all I need to do is to make Obsidian create the files like Hugo wants it to. Since the files can be created within Obsidian I have 3 ways to go about it:
1. Make Hugo read Obsidian metadata—preferable.
2. Inject metadata automatically on push—possible but error-prone.
3. Make Obsidian behave like Hugo's metadata—last resort.

Now let's use some experience, and the docs again - this format that we've seen before
```toml
+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName `-` ` ` | title }}'
+++
```

Looks awful similar to the Obsidian format for metadata, isn't it?
```yaml
---
date: <date>
draft: true
title: Some title
---
```

How was this thing called when we went over the directory? archetypes? Yeah lets find that in the docs under Content Management.

What do you know? it appears that this is the default [TOML](https://toml.io/en/) format, clicking on the YAML format shows me exactly what I want to see. Nice.

OK. Now - either there's a flag to use TOML/YAML/JSON, or it just works. Let's try YAML just because its easy before trying to find it in the docs. What do you know x2? It works right off the bat.

Obsidian integration almost complete, all I have to do is massage the metadata like Hugo expects it to be, and we're done. gg ez.

### Injecting metadata content

I've been using Obsidian long enough, and have been using [templates](https://help.obsidian.md/Plugins/Templates) for a while now, I know what I have to do -
```yaml
---
date: "{{date:YYYY-MM-DDTHH:mm:ss+03:00}}"
title: "{{title}}"
draft: true
---
```

All I need to do is move the file to the `/path/to/obsidian-vault/blog` directory and serve Hugo to test it. Yep - works. Next.

### Moving the note automatically

Since I've been using Obsidian for a while I know that it has multiple functions that I can access pressing CMD+O, and just searching what I want. I see that there's a move file to directory option so I know that it's possible. I also know that Obsidian is written in Javascript and all of its plugins are as well, and I also know that you can inject a script to do what I want.

Let's Google: "move files automatically Obsidian template." This yielded [this result](https://forum.obsidian.md/t/templater-rename-a-file-and-move-it-to-a-folder/39287).

Oh I remember this plugin! I used to use [Templater](https://github.com/SilentVoid13/Templater) before there was an official support for templates in Obsidian. I thought it was redundant but if it's really as easy as just putting this in the file - let's just get back to it.

A quick install and going over the [Templater documentation](https://silentvoid13.github.io/Templater/), and a new Blog template was born:

```
---
timestamp: <% tp.file.creation_date("YYYY-MM-DDTHH:mm:ss+03:00") %>
modified: <% tp.file.last_modified_date("YYYY-MM-DDTHH:mm:ss+03:00") %>
draft: "true"
title: <% tp.file.title %>
creation_date: <% tp.file.creation_date("YYYY-MM-DDTHH:mm:ss+03:00") %>
---
<% tp.file.move('/blog/' + tp.file.title) %>%

```

Also this plugin is seriously cool. It has some features like [tp.file.cursor](https://silentvoid13.github.io/Templater/internal-functions/internal-modules/file-module.html#tpfilecursororder-number) that puts the cursor where I tell it to rather than having it in the end - which helped out my other templates.

This took me on a side-quest of making my other templates better that I won't get into here, because by now you're probably hungry and tired. It was important for me to note this, because we often get distracted, and often link things to other things. The important things is to know yourself - should this be done later? should I write a note to myself to maybe do this in the future? should I even do this? is it small enough for me to do it now? There are no good answers here and it really depends on the project, its priority, your goal and yourself. But we're humans - and we get excited, and that's fine. We just have to be self aware about it.

## Automatic publish 

So to recap - in order to create a post, I  need to:
1. Create a new note/navigate to an existing note in Obsidian
2. CMD+T (my binding for templates) and choose Blog - this automatically moves the file to the blog directory, which is in turn mounted to Hugo's content 

Nice. Simple.

Lets set up a [pre-commit hook in git](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) in `.git/hooks/pre-commit`:
```bash
hugo 
if [ $? -ne 0 ]; then 
	echo "Hugo build failed. Aborting commit." 
	exit 1 
fi 
git add public resources
```

All I have to do it push the website whenever I want (using git) and for it to automatically publish it to the blog.

Enter GitHub Actions. Luckily, we saw earlier in the docs the exact thing we need - Hosting and deployment (in the side bar), and what do you know - [Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

Their tutorial is a bit out-dated, I had to switch to GitHub actions and select a Hugo template that is given there - a quick comparison between what was in their instructions and the actual template from GitHub showed the the template is fine and I just committed it using the GitHub UI.

## Summary

That does it. We've got a blog!

By isolating issues, revisiting plans, and leveraging existing tools, we've turned a potential roadblock into a learning experience. Now, creating and publishing a blog post is seamless and integrated into my daily workflow.

While I skipped some details like hooking automatic local build before push, add a "Sync Blog" command to Obsidian, implementing analytics, handling Obsidian inner links, customizing and making the blog look nicer and registering the domain name, but I invite you to customize and do things however the hell you wanna do them.

This felt like a great blog post to start my writing journey (which took WAY too long) because it was about the nature of programming and why I love it so much, using a very simple and isolated example. I went into each detail in-depth, and it might seem intimidating at first - but these things become second nature pretty quickly, and this is how experience is built!

If you've reached this far, thank you very much! I appreciate you very much, and hope to see you again in the next post!

## Dead-eyed tutorial

For all you people who just wanna follow the recipe, here it is:

### Tools Overview
- **Obsidian:** For writing and managing notes.
- **GitHub:** For hosting and publishing the blog.
- **Hugo:** For generating the static site.

### Step-by-Step Instructions

```bash
# install hugo
brew install hugo

# create new hugo website
hugo new site deadeye-tut 
cd deadeye-tut
git init

#  install theme
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.gitthemes/ananke
echo "theme = 'ananke'" >> hugo.toml

# serve
hugo server -D

# create dedicated blog directory in obsidian
mkdir /path/to/obsidian-vault/blog
```

Add the following to `hugo.toml`:
```toml
[module]
  [[module.mounts]]
    source = '/path/to/obsidian-vault/blog'
    target = 'content'
```

Install Templater in Obsidian:
- Go to Obsidian Settings
- Community Plugins
- Browse
- Search for "Templater"
- Install and enable it

Create a Templater template:
```yaml
---
date: <% tp.file.creation_date("YYYY-MM-DDTHH:mm:ss+03:00") %>
draft: "true"
title: <% tp.file.title %>
---
<% tp.file.move('/blog/' + tp.file.title) %>
```

GitHub integration and automation:
1. Create a GitHub repo
2. Push the code from your local repo
3. In the repo's page, go to "Settings" and "Pages" in the side-bar
4. Change "Source" to GitHub actions and choose the Hugo template -> next next next until you're done and commit

Add pre-commit hook in `.git/hooks/pre-commit`
```bash
hugo 
if [ $? -ne 0 ]; then 
	echo "Hugo build failed. Aborting commit." 
	exit 1 
fi 
git add public resources
```

Lastly:
```bash
git push -u origin main
```