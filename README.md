# llm-style-guides

Collection of style guides that can be pasted in ChatGPT, Claude, Cursor, etc to help code based on my prefrences and notes.

For example FastHTML supports different ways of developing apps including `app,rt = fast_app()` vs `app = FastHTML()`.

This style guide explains the difference and can then help LLMs to convert examples to my preferred style.

# FastHTML Style Guide

## Virtual Env

Create a venv in VSCode by: 

- Creating a main.py file 
- Open and save main.py file
- Click in bottom right on python version
- Panel opens in top middle
- Click on "+ Create Virtual Env..."
- Right click on explorer in VScode (left panel) in blank space
- Select "Open in Integrated Terminal"
- venv should be enabled now in terminal ready for installation

Note: I use this manual process as sometimes have issues with VSCode recognising the venv when I create it via the terminal

## Installation

Since fasthtml is a Python library, you can install it with:

```
pip install python-fasthtml
```

## Apps and routes

There are 3 supported ways for creating app and defining routes

### Option 1: (Prefered. Use this!)

Source: [Tutorials > FastHTML By Example > Constructing HTML (https://docs.fastht.ml/tutorials/by_example.html#constructing-html)](https://docs.fastht.ml/tutorials/by_example.html#constructing-html)

```python
# Don't use from fasthtml.common import FastHTML, serve
# Use from fasthtml.common import *
# Why use import *
# https://docs.fastht.ml/explains/faq.html#why-use-import
from fasthtml.common import *

css = Style(':root {--pico-font-size:90%,--pico-font-family: Pacifico, cursive;}')
app = FastHTML(hdrs=(picolink, css))

@app.get("/")
def home():
    return (Title("Hello World"), Main(
        H1('Hello, World'), 
        cls="container"))

serve()
```

### Option 2: 
#### Convert this style to Option 1

Source: [Get Started > Usage (https://docs.fastht.ml/#usage)](https://docs.fastht.ml/#usage)

```python
from fasthtml.common import *

app,rt = fast_app()

@rt('/')
def get(): return Div(P('Hello World!'), hx_get="/change")

serve()
```

### Option 3:
#### If you see this convert to option 1

Source: [Tutorials > FastHTML By Example > Defining Routes (https://docs.fastht.ml/tutorials/by_example.html#defining-routes)](https://docs.fastht.ml/tutorials/by_example.html#defining-routes)

```python
from fasthtml.common import *
app = FastHTML()
rt = app.route

@rt("/")
def get():
    page = Html(
        Head(Title('Some page')),
        Body(Div('Some text, ', A('A link', href='https://example.com'), Img(src="https://placehold.co/200"), cls='myclass')))
    return page

serve()
```

## Layout with grids

### Simple example

```
grid = Html(
    Link(rel="stylesheet", href="https://cdnjs.cloudflare.com/ajax/libs/flexboxgrid/6.3.1/flexboxgrid.min.css", type="text/css"),
    Div(
        Div(Div("This takes up the full width", cls="box", style="background-color: #800000;"), cls="col-xs-12"),
        Div(Div("This takes up half", cls="box", style="background-color: #008000;"), cls="col-xs-6"),
        Div(Div("This takes up half", cls="box", style="background-color: #0000B0;"), cls="col-xs-6"),
        cls="row", style="color: #fff;"
    )
)
show(grid)
```

### Advanced example

```python
from fasthtml.common import *

css = Style('''
    :root {--pico-font-size:90%,--pico-font-family: Arial, sans-serif;}
    .grid-container { display: grid; grid-template-columns: 20% 80%; gap: 20px; }
    .title-list { background-color: #f0f0f0; padding: 20px; }
    .title-item { cursor: pointer; margin-bottom: 10px; }
    .title-item:hover { text-decoration: underline; }
    .card { background-color: #ffffff; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
''')

app = FastHTML(hdrs=(picolink, css))

titles = [
    {"id": 1, "title": "Amazing Landscapes", "description": "Breathtaking views from around the world"},
    {"id": 2, "title": "Cute Animals", "description": "Adorable creatures that will melt your heart"},
    {"id": 3, "title": "Futuristic Technology", "description": "Glimpses into the world of tomorrow"}
]

@app.get("/")
def home():
    return (
        Title("Dynamic Content App"),
        Main(
            H1("Dynamic Content App"),
            Div(
                Div(
                    H2("Titles"),
                    *[P(title["title"], 
                        cls="title-item", 
                        hx_get=f"/card/{title['id']}", 
                        hx_target="#card-container",
                        hx_swap="innerHTML") for title in titles],
                    cls="title-list"
                ),
                Div(
                    card(titles[0]),
                    id="card-container",
                    cls="card"
                ),
                cls="grid-container"
            ),
            cls="container"
        )
    )

def card(content):
    return Div(
        H2(content["title"]),
        P(content["description"]),
        Img(src=f"https://placehold.co/300", alt=content["title"])
    )

@app.get("/card/{id}")
def get_card(id: int):
    content = next((title for title in titles if title["id"] == id), None)
    if content:
        return card(content)
    return P("Content not found")

serve()
```

## Adding Interactivity using HTMX

You don't need to include anything as HTMX is automatically bundled

This will swap the contect of the element that is clicked

```python
from fasthtml.common import *

css = Style(':root {--pico-font-size:90%,--pico-font-family: Pacifico, cursive;}')
app = FastHTML(hdrs=(picolink, css))

@app.get("/")
def home():
    return (Title("Hello World"), Main(
        H1('Hello, World'), 
        Div(P('Hello World!'), hx_get="/change"),
        cls="container"))

@app.get('/change')
def change(): return P('Nice to be here!')

serve()
```

Use htmx hx_target="#card-container" and hx_swap to correctly update content in another location

The possible values of the hx_swap attribute are:
* innerHTML - Replace the inner html of the target element
* outerHTML - Replace the entire target element with the response
* textContent - Replace the text content of the target element, without parsing the response as HTML
* beforebegin - Insert the response before the target element
* afterbegin - Insert the response before the first child of the target element
* beforeend - Insert the response after the last child of the target element
* afterend - Insert the response after the target element
* delete - Deletes the target element regardless of the response
* none- Does not append content from response (out of band items will still be processed).

```python
from fasthtml.common import *

css = Style('''
    :root {--pico-font-size:90%,--pico-font-family: Arial, sans-serif;}
    .grid-container { display: grid; grid-template-columns: 20% 80%; gap: 20px; }
    .title-list { background-color: #f0f0f0; padding: 20px; }
    .title-item { cursor: pointer; margin-bottom: 10px; }
    .title-item:hover { text-decoration: underline; }
    .card { background-color: #ffffff; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
''')

app = FastHTML(hdrs=(picolink, css))

titles = [
    {"id": 1, "title": "Amazing Landscapes", "description": "Breathtaking views from around the world"},
    {"id": 2, "title": "Cute Animals", "description": "Adorable creatures that will melt your heart"},
    {"id": 3, "title": "Futuristic Technology", "description": "Glimpses into the world of tomorrow"}
]

@app.get("/")
def home():
    return (
        Title("Dynamic Content App"),
        Main(
            H1("Dynamic Content App"),
            Div(
                Div(
                    H2("Titles"),
                    *[P(title["title"], 
                        cls="title-item", 
                        hx_get=f"/card/{title['id']}", 
                        hx_target="#card-container",
                        hx_swap="innerHTML") for title in titles],
                    cls="title-list"
                ),
                Div(
                    card(titles[0]),
                    id="card-container",
                    cls="card"
                ),
                cls="grid-container"
            ),
            cls="container"
        )
    )

def card(content):
    return Div(
        H2(content["title"]),
        P(content["description"]),
        Img(src=f"https://placehold.co/300", alt=content["title"])
    )

@app.get("/card/{id}")
def get_card(id: int):
    content = next((title for title in titles if title["id"] == id), None)
    if content:
        return card(content)
    return P("Content not found")

serve()
```

## Deploy app locally

```sh
python main.py
```

You should see link to open locally

```
Link: http://localhost:5001
INFO:     Will watch for changes in these directories: ['/Users/user.name/your/dev/folder']
INFO:     Uvicorn running on http://0.0.0.0:5001 (Press CTRL+C to quit)
INFO:     Started reloader process [69908] using WatchFiles
INFO:     Started server process [69910]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```
