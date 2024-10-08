# llm-style-guides

Collection of style guides that can be pasted in ChatGPT, Claude, Cursor, etc to help code based on my prefrences and notes.

- FastHTML Style Guide

For example FastHTML supports different ways of developing apps including `app = FastHTML()` vs `app,rt = fast_app()` vs `app = FastHTML(); rt = app.route`.

The style guide aims to explain the difference and therefore can help human readers and LLMs to convert examples to my preferred coding style.

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

## FastHTML() vs fast_app()

You should use always use FastHTML(). If you see a fast_app() example like this:

```python
app,rt,rooms,Room = fast_app('data/drawapp.db', render=render, id=int, name=str, created_at=str, pk='id')
```

Convert it to the FastHTML() version

```
db = database('data/drawapp.db')
rooms = db.t.rooms
if rooms not in db.t:
    rooms.create(id=int, name=str, created_at=str, pk='id')
Room = rooms.dataclass()

@patch
def __ft__(self:Room):
    return Li(A(self.name, href=f"/rooms/{self.id}"))
```

This is because my preferred style is to seperate the database creation from the app and route creation

This is also because prefer to use 

```python
app.get()
def home():
```

vs

```python
@rt('/')
def get(): 
```

## Apps and routes (TODO)

- Explain the differemce between app 3 options

There are 3 supported ways for creating apps and defining routes

- Option 1: `app = FastHTML()` << Preferred. Use this!
- Option 2: `app,rt = fast_app()` 
- Option 3: ```app = FastHTML()
  rt = app.route```

### Option 1: (Preferred. Use this!)

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

## Styling with PicoCSS

Always include PicoCSS styling using

```python
from fasthtml.common import *
css = Style(':root {--pico-font-size:90%,--pico-font-family: Pacifico, cursive;}')
app = FastHTML(hdrs=(picolink, css))
```

## Layout with grids (TODO)

- Update advanced to use [http://flexboxgrid.com/](http://flexboxgrid.com/)

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

### Advanced example (TODO)

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

## Charts

Using Plotly

Adapted from: [tutorials >quickstart_for_web_devs > a-minimal-charting-application](https://docs.fastht.ml/tutorials/quickstart_for_web_devs.html#a-minimal-charting-application)

```python
import json
from fasthtml.common import *

css = Style(':root {--pico-font-size:90%,--pico-font-family: Pacifico, cursive;}')

app = FastHTML(hdrs=(
    picolink,
    css,
    Script(src="https://cdn.plot.ly/plotly-2.32.0.min.js")
))

data = json.dumps({
    "data": [
        {"x": [1, 2, 3, 4], "type": "scatter"},
        {"x": [1, 2, 3, 4], "y": [16, 5, 11, 9], "type": "scatter"}
    ],
    "layout": {
        "title": "Plotly chart in FastHTML",
        "xaxis": {"title": "X Axis"},
        "yaxis": {"title": "Y Axis"}
    }
})

@app.get("/")
def home():
    return (
        Title("Plotly Chart Demo"),
        Main(
            H1("Plotly Chart Demo"),
            P("This is a demo dashboard using Plotly in FastHTML"),
            Div(id="myDiv"),
            Script(f"var data = {data}; Plotly.newPlot('myDiv', data.data, data.layout);"),
            cls="container"
        )
    )

serve()
```

Using Observable plot

```python
from fasthtml.common import *

css = Style(':root {--pico-font-size:90%,--pico-font-family: Pacifico, cursive;}')

app = FastHTML(hdrs=(
    picolink, css,
    Script(src="https://cdn.jsdelivr.net/npm/d3@7"),
    Script(src="https://cdn.jsdelivr.net/npm/@observablehq/plot@0.6")
))

@app.get("/")
def home():
    return (
        Title("D3 and Observable Plot Demo"),
        Main(
            H1("D3 and Observable Plot Demo"),
            Div(id="myplot"),
            Script("""
                const plot = Plot.rectY({length: 10000}, Plot.binX({y: "count"}, {x: Math.random})).plot();
                const div = document.querySelector("#myplot");
                div.append(plot);
            """, type="module"),
            cls="container"
        )
    )

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

## SQLite Database (TODO)

- Explain the dirrefent methods for using SQLite databases

- Option 1: `db = database('data/drawapp.db')` << Preferred. Use this!
- Option 2: `app,rt,rooms,Room = fast_app('data/drawapp.db', render=render, id=int, name=str, created_at=str, pk='id')`

Source: https://docs.fastht.ml/tutorials/e2e.html#drawing-rooms

### Option 1: `db = database('data/drawapp.db')` << Preferred. Use this!

```python
db = database('data/drawapp.db')
rooms = db.t.rooms
if rooms not in db.t:
    rooms.create(id=int, name=str, created_at=str, pk='id')
Room = rooms.dataclass()

@patch
def __ft__(self:Room):
    return Li(A(self.name, href=f"/rooms/{self.id}"))
```

Option 2: If you see this convert to Option 1: `db = database('data/drawapp.db')`

Note: Issue with option 2 is that you can only create 1 table for the whole app (Need to confirm).
This might be useful in some scenarios.

```python
def render(room):
    return Li(A(room.name, href=f"/rooms/{room.id}"))

app,rt,rooms,Room = fast_app('data/drawapp.db', render=render, id=int, name=str, created_at=str, pk='id')
```

## Forms

Source:  https://docs.fastht.ml/tutorials/e2e.html#drawing-rooms



```python
from fasthtml.common import *
from datetime import datetime
from fastcore.utils import patch

css = Style('''
    :root {--pico-font-size:90%,--pico-font-family: Arial, sans-serif;}
    .url-form { margin-bottom: 2rem; }
    .url-list { list-style-type: none; padding: 0; }
    .url-item { margin-bottom: 0.5rem; }
''')

app = FastHTML(hdrs=(picolink, css))

# Database setup
db = database('data/urlsaver.db')
urls = db.t.urls
if urls not in db.t:
    urls.create(id=int, url=str, created_at=str, pk='id')
URL = urls.dataclass()

@patch
def __ft__(self: URL):
    return Li(A(self.url, href=self.url, target="_blank"), cls="url-item")

@app.get("/")
def home():
    return (
        Title("URL Saver"),
        Main(
            H1("URL Saver"),
            Form(
                Input(type="url", name="url", placeholder="Enter a URL", required=True),
                Button("Save URL", type="submit"),
                cls="url-form",
                hx_post="/save-url",
                hx_target="#url-list",
                hx_swap="afterbegin"
            ),
            H2("Saved URLs"),
            Ul(
                *[URL.__ft__(url) for url in urls()],  # Changed to use urls()
                id="url-list",
                cls="url-list"
            ),
            cls="container"
        )
    )

@app.post("/save-url")
def save_url(url: str):
    new_url = URL(url=url, created_at=datetime.now().isoformat())
    urls.insert(new_url)
    return URL.__ft__(new_url)

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
