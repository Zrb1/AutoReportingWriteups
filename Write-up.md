
# Automatic Reporting in Python
## Part 1 - From planning, to Hello World

The goal of this project is to create an automatic reporting tool.

The simple outcome would, in any particular case, be a single stand-alone HTML file. An HTML file is a great reporting tool: even stripped of a back-end server, you can package up a good volume of info in a page and have handy-dandy interactivity.

<img src="static/BroadConcept.png">

Personally, I mainly intend to use this to automate reporting of model performance in the machine learning space, but you could of course use this for any context! 

While not a terribly exciting or sexy tool, automatically producing reports is a fantastic capability, especially in business environments. [Automate the boring stuff,](https://automatetheboringstuff.com/) as they say.

**Naturally, tools like this already exist**. But I want to make my own, for the experience and flexibility it offers me.

### How is this guide presented?

Rather than just providing a dump of code that works, I'd like to incrementally work through the development, step-by-step. [You'll see what I see - code what I code!](https://www.youtube.com/watch?v=l1_bp8YKUPU)

Personally, I've always found this kind of tutorial more effective as:
* It gives an insight of how someone else approaches a problem
* It actually reflects how I code - feature by feature, commit by commit.

### Who is this for?

A user "knee-deep" in Python is assumed. I'll not run over standard syntax and the like, but I'll be sure to stop and explain anything particularly tricky. Wherever possible, I'll refer back to cleverer and clearer explanations.

___

## Step Zero - Stop for a minute

I'm actually a chemical engineer by background. While I certainly gained and applied a lot of useful and/or esoteric knowledge from this background, by far the most useful bit of advice I ever learnt was probably also the most simple:

> When approaching a new problem, draw a picture.

By "draw a picture", I mean to externalise the problem. Sketch it out and get thoughts and ideas out on paper.

Before tackling this (relatively small-scale) project, I sat down and spent 10-15 minutes sketching out how I thought I might approach it, what parts would move, what parts would stay static, and a (very) rough structure of the project. 

My own nigh-incomprehensible sketch is as follows.

<img src="static/ConceptSketchII.png">

### Some key learnings from early planning

Based on my early "planning", some things came clear:
* An [HTML templating system](https://en.wikipedia.org/wiki/Web_template_system) makes a lot of sense. These systems are designed to input dynamic content into static template: this sounds exactly like what we're after. In Python, [Jinja](http://jinja.pocoo.org/) is a common choice, and may be familiar to anyone who has played with [Flask](http://flask.pocoo.org/).
* In the machine-learning space, I'm constantly finding new things I might want to look out for, or new ways to compare models, or new ways to analyse a dataset - so both the *content of a report* and the *features I can pack into any report* should be flexible.
* A lot of the data I work with is in .csv format, so having tools to work with that and generate interrogatable HTML blocks from it would be neat.

___

## Step One - Lay a foundation

For pretty much any size project bigger than tinkering in the command line for a couple of minutes, I'm fond of setting up a Github repo and a consistent environment. 

This, I think, will be a pretty small-scale project, so I'm not terribly concerned with laying down a [full project structure](http://docs.python-guide.org/en/latest/writing/structure/), but it's worth being comfortable with this for further reference.

This:
* Encourages good habits in keeping the project clean
* Helps me jump from my Mac laptop to my Linux desktop (or any other enviornment, for that matter) with a minimum of fuss

A few quick goals to start us off:

### Set up a virtual environment

We want to create an isolated instance of Python for this project - or "virtual environment" - rather than using whatever general installation is present on this machine. This isolates our dependencies and makes it easier to move the project around. I'm working in Pycharm, which gives you the option to create new virtual environment from the start.

To make this transferable, I'll keep a `requirements.txt` file in the root of my project directory which details what packages are required in the environment.

There are [many](https://docs.python.org/3/tutorial/venv.html), [many](https://realpython.com/python-virtual-environments-a-primer/), [many](http://docs.python-guide.org/en/latest/dev/virtualenvs/) guides on this topic, and they're good to refer to for more specific details. 

### Set up a Git/Github repo

To implement version tracking and allow ready transport to and fro from Github, we'll initialise a Git repo locally (in the project folder) and create a new repo in Github. The local Git repo can then pushed to Github. 

A succint guide can be found [on Github.com](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/).

As a matter of course of working in a Git/Github repo, we'll add the following to the root project directory:
* A `readme.md` file that briefly describes the project, and
* A `.gitignore` file that indicates which files and folders should not be tracked by Git and pushed to Github. This might include automatically generated files (from the IDE, or from the virtual environment, for instance), sensitive files and config data, or anything else you've created locally that doesn't need to be shared with the world. For me, this looks like:

```
# .gitignore

# Don't add the virtual environment, IDE, and Jupyter notebook info
venv
.idea
.ipynb_checkpoints
Write-up.ipynb
```

### Github status
At this point, the skeleton of your project should look a bit like [this](https://github.com/goyder/autoreporting/tree/c1f1ba68cae351d0de851f0f4831b0f2caa0a407).

___
## Step Two - "Hello, World" templating

At this point, we have a very simple project structure that we can start populating with code and templates. To carry out a bit of a smoke-test and give me somewhere to build off of, I'd like to create the simplest possible report.

This first implementation should input no information, and output a simple "Hello, world!" HTML page.

### A brief primer on templating

This project is built on the concept of templating and [template processors](https://en.wikipedia.org/wiki/Template_processor). This is an enormous but very crucial field in web publishing.

In a crude explanation: we separate the *structure* of an HTML page and the *content* that is to be published. When it comes time to generate a new webpage (or in this context, a report!) we can input the content into a standardised structure and get a new webpage out. 

<img src="static/TemplatingEngine.jpg">

Templating is well and truly ubiquitous now, but if you'd like an example of life without consistent manual or automatic templating, you can find many unironically charming examples in a [Geocities archive](http://www.oocities.org/).

### Building 'hello, world' functionality

Building on this distinction between *structure* and *content*, we can create very simple examples for both to test the system.

#### Structure

Our structure, in this case, is a near-empty HTML page with locations for data to be input. We'll call this a "template", and we'll store this and others under a `template` folder in our root project directory. We'll create a new HTML file under this folder - `{project_folder}/template/report.html`. The contents of this file are as follows:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ content }}</title>
</head>
<body>
{{ content }}
</body>
</html>
```

Note the elements here within curly braces (in this case *double* curly braces - `{{` and `}}`). This is where dynamic content will be entered. 

Template processors such as Jinja are very powerful and allow many features to be present in templates beyond simple insertion. Take a look at the first example on the [Jinja homepage](http://jinja.pocoo.org/docs/2.10/) and see if you can make sense of it.

#### Content

Our content, for now, can be as simple as `hello, world!`. 

We'll create a new file in the root of the project directory called `{project_folder}/autoreporting.py` and hardcode our 'content' to begin with.

```python
content = "Hello, world!"
```

#### Functionality

We have structure and content - now we just need to get them combined using Jinja.

Firstly, create an `outputs` directory and add it to your `.gitignore` file. We'll dump what we create in this folder, but there's no reason to share whatever ends up in here in your repo.

Jinja requires an `Environment` object to store and define things like the config and loader - i.e. where we will pull our templates from. In this case, we're running off the local filesystem, so a `FileSystemLoader` specified to search in the `templates` directory will do us just fine. 

With our `env` instance, we can render our basic template and write this to file.

Our `autoreporting.py` script now looks like this:

```python
from jinja2 import FileSystemLoader, Environment

# Content to be published
content = "Hello, world!"

# Configure Jinja and ready the template
env = Environment(
    loader=FileSystemLoader(searchpath="templates")
)
template = env.get_template("report.html")


def main():
    """
    Entry point for the script.
    Render a template and write it to file.
    :return:
    """
    with open("outputs/report.html", "w") as f:
        f.write(template.render(content=content))


if __name__ == "__main__":
    main()

```

Note the argument to `template.render`, `content=content`. This links the template and our code. The first `content` refers to the `{{ content }}` in the template, while the second is the string defined in the script. We define this relationship and the string is fed into the template to make a substitution.

### Results so far

If you run this script and all goes well, a new `report.html` file will be written under `outputs/`. Open this file and behold what you have created!

<img src="static/ItWorks.png">
This certainly meets the criteria of "very simple, but functional."

If you inspect the HTML of the page, you'll note that the `{{ content }}` tags in the template were neatly replaced by the `Hello, world!` string, which is the crux of this affair.

## Github status

You've developed a new feature, so it's a good point to commit and push to Github. 

Your repo should probably look a bit like [this](https://github.com/goyder/autoreporting/tree/271790cb0e6ed8d2e07b585fa43d03b1e1fa5b1b).

## Next Steps

The reporter walks and works, but we certainly don't have any sort of meaningful input or flexibility at this point. Our next steps will be to explore how we can read in data and report on it, and start working with JavaScript and CSS.
