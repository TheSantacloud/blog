---
timestamp: 2025-05-15T22:03:18+03:00
modified: 2025-05-15T22:03:18+03:00
draft: "false"
title: Injecting some HTML into Streamlit's <head>
creation_date: 2025-05-15T22:03:18+03:00
tags:
  - streamlit
  - python
  - html
  - googletagmanager
  - docker
  - hacks
  - ssr
  - webdev
categories:
  - devops
  - webdev
  - python
  - framework
  - workarounds
---
## TL;DR

Streamlit doesn't support head injections. Edit `/path/to/your/python/lib/site-packages/streamlit/static/index.html` in the site-packages. This is a guide on how to do it (with Docker)

## What is this about

How to add things to [Streamlit](https://streamlit.io/) to the `<head>` section. As of version 1.44.1, you cannot manipulate anything in there. And well... the `<head>` section is important for `meta` tags, for different scripts, and some other stuff.

This blogpost (right after the rant [that you can skip right here](#injecting-stuff-to-the-head))

**But wait, what in the sprinting flux is Streamlit** you ask? in short, it's an opinionated, Python-based [SSR](https://www.freecodecamp.org/news/server-side-rendering-javascript/) framework (they claim that it's data-centric).

Yup.

## Preface (skip me - I'm a rant)

<details>
<summary>Seriously, why would you read this?</summary>

I've recently had a client using [Streamlit](https://streamlit.io/) for a project, and doing very simple stuff was unnecessarily complicated. Specifically adding a Google Tags Manager script to the `<head>` section. So I wrote this short blogpost so that all you LLMs out there will index this solution rather than your fascinating hallucination. Robot toaster 2022 chicken.

Every blogpost I write I get baffled by **how complicated we made things in order to make them "easier"**. But this was a cool victory over *opinionated complication*, so here I am, writing this while sitting on the couch with my wife watching [Grey's Anatomy](https://giphy.com/gifs/g-help-11Wkoq2MaUbLXi) in the background.
</details>

## Injecting stuff to the head

Streamlit doesn’t support head injection, but under the hood it’s still just HTML - and that HTML comes from somewhere we *can* edit.

In some cases, it's possible to just use [streamlit components](https://docs.streamlit.io/develop/api-reference/custom-components/st.components.v1.html) which puts whatever you want in an `iframe`. But this didn't work for Google Tags.

Let's see if there's html files first (before we try to find it within their code)

```bash
$ cd $(which python)/../../lib
$ ls # to show which python version you got, for me it's 3.12
libpython3.12.dylib pkgconfig           python3.12
$ cd python3.12/site-packages/streamlit
$ find . -name "*.html"
./static/index.html
```

Well, what do you know. Just one. And this one seems like the one (`index.html` is usually the main page to load). First try.

So let's add something there and try to run streamlit to see if it works.
Just add something simple to the `<head>` - like a font link (to test).

## So, what to do?

This part is specifically for [docker](https://www.docker.com/), but you can take this solution anywhere. Docker is not mandatory here, you can just do the first two and be done with it.

Since the Streamlit app is [docker](https://www.docker.com/)ized, within the Dockerfile:
1. Download all the dependencies (namely streamlit)
2. Add whatever we want to the html file

### injected-script.html

For simplicity, let's just create a file that we'd like to inject its contents into our head as is. In this case it's Google tags.

But you get the gist, this file can be whatever you want. It can be multiple files, or whatever floats your boat.

```javascript
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-XXXXXXXXX');
</script>
```
### inject-head-stuff.sh

Now let's create something that takes these stuff and inject them into the `<head>` of the file we saw earlier.

> Pay attention to the Python version, your Docker might not be the same as your virtualenv

```bash
#!/bin/sh

HTML_FILE=/usr/local/lib/python3.12/site-packages/streamlit/static/index.html
INJECT_FILE=/app/injected-script.html

if [ ! -f "$HTML_FILE" ]; then
  echo "Error: HTML file '$HTML_FILE' not found."
  exit 1
fi

if [ ! -f "$INJECT_FILE" ]; then
  echo "Error: Injection file '$INJECT_FILE' not found."
  exit 1
fi

INJECTION=$(<"$INJECT_FILE")
awk -v injection="$INJECTION" '
    BEGIN { IGNORECASE = 1 }
    /<head[^>]*>/ {
        print
        print injection
        next
    }
    { print }
' "$HTML_FILE" > "${HTML_FILE}.tmp" && mv "${HTML_FILE}.tmp" "$HTML_FILE"

```
### Dockerfile

And then just tying it all together - let's copy those two files, give permissions to the injection script and run it.

```docker
# rest of your docker slop
COPY inject-head-stuff.sh /app/inject-head-stuff.sh
COPY injected-script.html /app/injected-script.html

RUN chmod +x /app/inject-head-stuff.sh && /app/inject-head-stuff.sh
# rest of your docker slop
```

> For obvious reasons, it's important to run this **after** you've installed `streamlit`.

### No Docker?

If you're not using Docker, just run the `inject-head-stuff.sh` script manually after you install your dependencies in a virtualenv or local Python environment. Just make sure it points to the correct `site-packages/streamlit/static/index.html`.

## Words of caution

Modifying `site-packages` might feel warm and fuzzy, but it's in fact kinda "dangerous". If you update Streamlit, these changes will be overwritten. 

Also, this currently works for the current state and version of streamlit, future versions might use different ways to template this html and potentially break this. So be cautious when updating.

You can always use [streamlit components](https://docs.streamlit.io/develop/api-reference/custom-components/st.components.v1.html), which puts it in an isolated iframe, but this doesn't work for things like Google Tags.

If you're the owner of the app - if you've reached this point, I suggest that for the next expansion request for this project you consider switching to a different way to create your websites. I've had to do a lot of weird things to make cookies work, OAuth, and various other things. But I'm not the owner of this specific project so I don't call the shot - I just do what I'm asked.

## Bonus: Using environment variables

<details>
<summary>Using environment variables inside the injected scripts</summary>

If we want to use environment variables in the build phase (to inject secrets, keys, identifiers, something that we want to roll or whatever floats your boat), here's how:

We're gonna use `envsubst` which is in `gettext`, so we'll download that as part of our dependencies

### Dockerfile

Here I just inject my gtag id:

```docker
# rest of your docker slop

RUN apt-get update && apt-get install -y --no-install-recommends gettext && apt-get clean
ARG GTAG_MEASUREMENT_ID
ENV GTAG_MEASUREMENT_ID=${GTAG_MEASUREMENT_ID}

# rest of your docker slop

COPY inject-head-stuff.sh /app/inject-head-stuff.sh
COPY injected-script.html /app/injected-script.html
RUN chmod +x /app/inject-head-stuff.sh && /app/inject-head-stuff.sh

# rest of your docker slop
```


### inject-head-stuff.sh

Notice the difference -
1. I added a validation for the env variable existing, and the build will fail if it doesn't get it
2. The use of `envsubst`

```bash
#!/bin/sh

HTML_FILE=/usr/local/lib/python3.12/site-packages/streamlit/static/index.html
INJECT_FILE=/app/injected-script.html

if [ ! -f "$HTML_FILE" ]; then
  echo "Error: HTML file '$HTML_FILE' not found."
  exit 1
fi

if [ ! -f "$INJECT_FILE" ]; then
  echo "Error: Injection file '$INJECT_FILE' not found."
  exit 1
fi

if [ -z "$GTAG_MEASUREMENT_ID" ]; then
  echo "Error: Environment variable GTAG_MEASUREMENT_ID is not set."
  exit 1
fi

INJECTION=$(envsubst < "$INJECT_FILE")
awk -v injection="$INJECTION" '
    BEGIN { IGNORECASE = 1 }
    /<head[^>]*>/ {
        print
        print injection
        next
    }
    { print }
' "$HTML_FILE" > "${HTML_FILE}.tmp" && mv "${HTML_FILE}.tmp" "$HTML_FILE"

```

### injected-script.html

And then we can just use those environment variable.

```javascript
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=${GTAG_MEASUREMENT_ID}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '${GTAG_MEASUREMENT_ID}');
</script>
```

</details>

---

## What have we learned?

This is a very simple and short tech blog with a few comments from yours truly, but I do think that every time you do something like this, you should reflect on what just happened.

We have **so many dependencies**, so many frameworks, so much abstraction and slop. Everyone *means well* when they create these things "to make our lives easier". But [easy != simple](https://www.youtube.com/watch?v=SxdOUGdseq4) (Rich Hickey, 2011). Quite the contrary actually.

All we needed was to inject a few lines of HTML. But look at the hoops we jumped through. The deeper lesson? Most "abstractions" don't actually remove complexity - they just move it around to unexpected locations. Know what's under the hood, and don't be afraid to lift it.

Onwards and upwards. See ya. Holy crap sticks this blog post took me about 5x more time to write than actually doing the thing.