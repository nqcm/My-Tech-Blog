---
title: "⚡️ Making a Static Site Generator with Python - part 1"
date: 2019-04-17T11:40:47+02:00
tags: ["Python", "Web", "Static Site", "Jinja2"]
summary: "Make your own static site generator with Python to learn how they work. A step by step tutorial. Part 1/2"
cover:
  image: "/img/static-site/static-site-tn.jpg"
---

Static site generators are a secure and fast alternative to traditional content management systems like Wordpress.

Static Site Generators or SSG are pretty simple to use once you get the hang of them. But initially there may be a learning curve. One way of learning (my preferred way) is to look under the hood and see how things work and try to recreate them. So when I saw [this post](https://rahmonov.me/posts/static-site-generator/) I wanted to create my own static site generator using Python.

It turned out to be a lot of fun to make!

If you are new to static site generators, you can first read about [what is a static site generator](https://davidwalsh.name/introduction-static-site-generators), [when to use one](https://learn.cloudcannon.com/jekyll/why-use-a-static-site-generator/), and [when not to use one](https://www.sitepoint.com/7-reasons-not-use-static-site-generator/). Also have a look at [a list of many static site generators](https://github.com/myles/awesome-static-generators) written in different programming languages.

So without further ado, let’s start.

## Understanding how static site generators work

The concept behind statistic site generators is pretty simple. You write your posts in markdown and the script in the static site generators turn your markdown into HTML using pre-created templates. These static HTML files can be uploaded to any web server and served to your readers.

Let’s say you are making a recipe blog. You write your recipe in markdown like this:

```markdown
title: Turkish Pide
date: 17-04-19

**Ingredients for the dough:**
500 gm bread flour
1 tblspoon yeast
1 tblspoon honey
1 tsp salt
2 tblsp olive oil
250 ml or as req water.

**Ingredients for the filling:**
1 cup grated Mozerella
½ cup crumbled feta

Combine the ingredients of the dough and knead until smooth and elastic. Let rest 45 minutes. Divide in two. Roll each in a rectangle. Spread the cheese filling over top. And roll the sides in to form a boat shape. Let rise for 10-15 minutes. Brush the sides with olive oil and bake at 230C for 10-15 minutes or until nicely browned.
```

And you have a template like this:

```HTML
<html>
    <head>
        <title>{{ title }}</title>
    </head>

    <body>
        <p>
            Published at: {{ date }}
            {{ content }}
        </p>
    </body>
</html>
```

The static site generator will parse the markdown text and insert each component where it belongs, so you will end up with something like this:

```HTML
<html>
    <head>
        <title>Turkish Pide</title>
    </head>

    <body>
        <p>
            Published at: 17-04-19
            <strong>Ingredients for the dough:</strong><br>
            <ul>
                <li>500 gm bread flour</li>
                <li>1 tblspoon yeast</li>
                <li>1 tblspoon honey</li>
                <li>1 tsp salt</li>
                <li>2 tblsp olive oil</li>
                <li>250 ml or as req water.</li>
            </ul>

            <strong>Ingredients for the filling:</strong>
            <ul>
                <li>1 cup grated Mozerella</li>
                <li>½ cup crumbled feta</li>
            </ul>

            Combine the ingredients of the dough and knead until smooth and elastic. Let rest 45 minutes. Divide in two. Roll each in a rectangle. Spread the cheese filling over top. And roll the sides in to form a boat shape. Let rise for 10-15 minutes. Brush the sides with olive oil and bake at 230C for 10-15 minutes or until nicely browned.
        </p>
    </body>
</html>
```

So we need to find a way to parse the text from markdown to HTML and to insert HTML into pre-created templates.

## Converting Markdown to HTML

Create a new folder where you want your code to live:

```console
mkdir recipe-ssg
```

SSG stands for Static Site Generator by the way. And as good as it is for the SEO of my site, I am pretty tired of typing Static Site Generators over and over so I will refer to Static Site Generators as SSG from now on. Phew!

In this folder, create the content folder where we will write our blog posts in markdown files:

```console
cd recipe-ssg & mkdir content
```

In the content folder create a markdown file. You can copy the contents of [this file](https://raw.githubusercontent.com/nqcm/static-site-generator/master/content/turkish-pide.md) and call it `turkish-pide.md`.

You will notice that the file is divided into two parts. There is some metadata between the two dash separators. And the actual content is below the separators.

Next we want a way to parse this markdown, convert it into HTML and store this data so it can be consumed by Python. But we don't need to reinvent the wheel. The awesome Python community has already written a [Markdown Parser](https://github.com/trentm/python-markdown2) for us. We will install it and play around a bit to see how it works.

Within your [virtual environment ](https://docs.python-guide.org/dev/virtualenvs/) install `markdown2` using pipenv or pip.

```bash
pipenv install markdown2
```

In the interactive Python shell try:

```python
from markdown2 import markdown
markdown("**This is a very important message**")

# outputs '<p><strong>This is a very important message</strong></p>\n'
```

Great! Now we can create a simple script to convert our markdown format files into HTML files.

Create a `main.py` file in the root of your project (not inside the content) and type the following:

```python
from markdown2 import markdown

with open('content/turkish-pide.md', 'r') as file:
    parsed_md = markdown(file.read(), extras=['metadata'])


print('Metadata: ', parsed_md.metadata)
print('Content: ', parsed_md)
```

Run `main.py` and you will see the contents of the `turkish-pide.md` file printed on the console with HTML tags. Also notice that the metadata is converted into a Python dictionary. We can now use these variables to create pages using templates.

But first we need to create a template.

## Creating and Using Templates

Templates are ordinary HTML files with placeholders for the variable data. We will use [Jinja2](http://jinja.pocoo.org/docs/2.10) templating language for Python which will simply insert the data from our variables in the HTML template. If you are new to templating languages or Jinja2, you can first read more about it [here](https://realpython.com/primer-on-jinja-templating/).

Install `Jinja2` within your virtual environment using pipenv or pip:

```console
pipenv install jinja2
```

To try Jinja2, create a folder inside your project root and call it `templates`. Now create a `test.html` inside `templates` and put the following code in it:

```html
<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>{{ post.title }}</title>
  </head>
  <body>
    <h1>{{ post.title }}</h1>
    <small>{{ post.date }}</small>
    <p>{{ post.content }}</p>
  </body>
</html>
```

Great! Now we have a template and we just have to extract our variables from the parsed markdown and hand them over to Jinja2.

In your `main.py` add the following code:

```python
from jinja2 import Environment, PackageLoader

env = Environment(loader=PackageLoader('main', 'templates'))
test_template = env.get_template('test.html')

data = {
    'content': parsed_md,
    'title': parsed_md.metadata['title'],
    'date': parsed_md.metadata['date']
}

print(test_template.render(post=data))
```

After importing the Jinja2, we create an environment to show Jinja2 where the templates folder is located. Make sure that the first argument for `PackageLoader` is the name of your python file, in my case it is `main`. Next we get the template we need and after extracting data from our parsed file we hand it over to Jinja2 while calling the `render()` function.

Your final code should look like this now:

```python
from markdown2 import markdown
from jinja2 import Environment, PackageLoader

with open('content/turkish-pide.md', 'r') as file:
    parsed_md = markdown(file.read(), extras=['metadata'])

    env = Environment(loader=PackageLoader('main', 'templates'))
    test_template = env.get_template('test.html')

    data = {
    'content': parsed_md,
    'title': parsed_md.metadata['title'],
    'date': parsed_md.metadata['date']
    }

    print(test_template.render(post=data))
```

If you run your `main.py` now you should get all the code from your `test.html` file with the contents from your `turkish-pide.md` file inserted in suitable places.

Now that we know how to render HTML pages using Jinja2, in the [next part](https://blog.naveeraashraf.com/posts/make-static-site-generator-with-python-2/) we will put all the pieces together and make our SSG.

<br>
