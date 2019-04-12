---
title: "Introducing Interactive VDOM 2"
---

# Announcing event handler and React support in vdom

vdom now supports event handling. You can pass Python functions as event handlers to any vdom object.


```python
from vdom import button

count = 1

def handle_click(event):
    global count
    count += 1
    counter.update(render_counter())

def render_counter():
    return button(str(count), onClick=handle_click)

counter = display(render_counter(), display_id=True)

counter;
```


<button>5</button>


See [Event support](#event-support) below for more examples. See the [docs](https://github.com/nteract/vdom/blob/master/docs/event-spec.md) for more about the implementation, supported event types, and event objects reference.

Additionally, vdom now supports React components in addition to native HTML elements. The vdom spec now accepts a new `import` property that expects a `package` (the npm package name) and optional `module` (the _named import_ from that package, defaults to `default`).


```python
from IPython.display import HTML, display

display({
    'application/vdom.v1+json': {
        'tagName': 'Button', 
        'attributes': {}, 
        'children': ['click me'],
        'import': {
            'package': '@blueprintjs/core',
            'module': 'Button'
        }
    }
}, raw=True)
```



We can create an `import_component` function to improve things significantly. See [vdom_extras.py](./vdom_extras.py) for details.


```python
from vdom_extras import import_component

Button = import_component('@blueprintjs/core', 'Button')

Button('click me')
```




<div>click me</div>



See the [vdom spec](https://github.com/nteract/vdom/blob/master/docs/mimetype-spec.md) for more about the spec.

## Background and context

[vdom](https://github.com/nteract/vdom/) is a Python library for expressing UI (user interface) using _[virtual dom](https://reactjs.org/docs/faq-internals.html#what-is-the-virtual-dom)_, a concept popularized by Javascript libraries such as [React](https://reactjs.org/). 

vdom is designed to improve the experience of rendering custom UI within the context of a Jupyter notebook and is currently supported in [JupyterLab](https://github.com/jupyterlab/jupyterlab) and [nteract](https://github.com/nteract/nteract).

### A simple example

The following is an example of how to render some simple UI using vdom:


```python
from vdom import div, h1, p, ul, li

div(
    h1('A Header'),
    p('A list of numbers'),
    ul([li(str(num)) for num in range(10)])
)
```




<div><h1>A Header</h1><p>A list of numbers</p><ul><li>0</li><li>1</li><li>2</li><li>3</li><li>4</li><li>5</li><li>6</li><li>7</li><li>8</li><li>9</li></ul></div>



*Notice how the `li` elements are dynamically generated using a Python list comprehension expression.*

To fully appreciate vdom, we must understand how we accomplished the same results before vdom. The following is an example that uses ipython's `HTML` display function.


```python
from IPython.display import HTML

items = [f'<li>{num}</li>' for num in range(10)]

HTML(f'''<div>
<h1>A Header</h1>
<p>A list of random numbers</p>
<ul>{''.join(items)}</ul>
</div>''')
```




<div>
<h1>A Header</h1>
<p>A list of random numbers</p>
<ul><li>0</li><li>1</li><li>2</li><li>3</li><li>4</li><li>5</li><li>6</li><li>7</li><li>8</li><li>9</li></ul>
</div>



The above works but take a moment to ponder the differences...

In the vdom example, we use Python functions (vdom's helper functions) such as `div` and `h1` to construct a *virtual DOM tree*. In the HTML example, we construct a string of HTML markup. Not only is the vdom example more Pythonic, but it has many advantages.

### Advantages of vdom

Using vdom is like using React or other virtual dom Javascript libraries in Python. As a result, it has many of the advantages that React has over plain HTML:

- **Dynamic:** In the HTML example, we are simply constructing static HTML markup in Python. In the vdom example, we are creating live vdom objects (or *components*) that are interactive and updatable.
- **Interactive:** Using the HTML function, the only way to handle events such as clicks and keys is to inline Javascript in our static HTML markup. Using vdom, we can use Python functions to handle events by simply passing them as attributes to the component (e.g. `button('click me', onClick=lambda _: print('clicked'))`).
- **Stateful:** Using the HTML function, it would be very difficult to update the UI in response to a state change in Python and it would be impossible to update the state in Python in response to an event on the front-end (this requires communication between the front-end and kernel). Using vdom, it's straight-forward to both update the UI in response to a state change in Python and update the state in Python in response to an event on the front-end (see the [Interactive and stateful example](https://www.notion.so/Introducing-module-support-to-vdom-5ddc063236d6409db1be41da3f1af582#89f889f8962a4562979d71912e88b1fd) below).

### Interactive and stateful example

The following is an example of how to add interactivity and state to a vdom component:


```python
from vdom import button
from IPython.display import display

# The state of the counter
count = 1

# The click handler
def handle_click(event):
    # Increment the count
    global count
    count += 1
    # Use ipython's update_display to update the display
    counter.update(render_counter())

# Create a render function that will render the counter initially and when updating
def render_counter():
    return button(str(count), onClick=handle_click)

# Create a display handle that we can reference to update the output
counter = display(render_counter(), display_id=True)

# Return the display handle and render the counter
counter;
```


<button>3</button>


One great thing about React is its focus on *components*. A React component contains all the markup for a unit of UI (e.g. a drop-down box) in addition to any event handlers, local state, and/or local methods. This makes React components *composable* meaning that they can be easily dropped into a component tree and if given the required *props* (or properties), just work! We can turn this counter example into a React-like component so that it can be easily reused in other contexts.

First, we will create a base class that components and subclass or inherit from. This will implement React-like props and state, a render function that will be overridden, and an `_ipython_display_` method that will inform ipython about how to display this object.


```python
from vdom import button
from IPython.display import display
from vdom_extras import BaseComponent

# Create a counter component with local state and event handler
class Counter(BaseComponent):
    def __init__(self, **props):
        # Initialize the BaseComponent
        super().__init__(**props)
        # Set the initial component state
        self.state = {'count': 0}
        
    def handle_click(self, event):
        # Increment the count value in the component state
        self.set_state({'count': self.state['count'] + 1})
        
    def render(self):
        # Render a button that displays the count value and increments the count on click
        return button(str(self.state['count']), onClick=self.handle_click)
    
Counter()
```


<button>0</button>


We've successfully created an interactive and stateful vdom component that can be used like any other vdom component.


```python
from vdom import div

div([Counter(), Counter(), Counter()])
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    ~/anaconda/lib/python3.6/site-packages/IPython/core/formatters.py in __call__(self, obj, include, exclude)
        968 
        969             if method is not None:
    --> 970                 return method(include=include, exclude=exclude)
        971             return None
        972         else:


    ~/Sites/jupyter/vdom/vdom/core.py in _repr_mimebundle_(self, include, exclude, **kwargs)
        249 
        250     def _repr_mimebundle_(self, include, exclude, **kwargs):
    --> 251         return {'application/vdom.v1+json': self.to_dict(), 'text/plain': self.to_html()}
        252 
        253     @classmethod


    ~/Sites/jupyter/vdom/vdom/core.py in to_html(self)
        205 
        206     def to_html(self):
    --> 207         return self._repr_html_()
        208 
        209     def json_contents(self):


    ~/Sites/jupyter/vdom/vdom/core.py in _repr_html_(self)
        242                     out.write(escape(safe_unicode(c)))
        243                 else:
    --> 244                     out.write(c._repr_html_())
        245 
        246             out.write('</{tag}>'.format(tag=escape(self.tag_name)))


    AttributeError: 'Counter' object has no attribute '_repr_html_'



    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    ~/anaconda/lib/python3.6/site-packages/IPython/core/formatters.py in __call__(self, obj)
        343             method = get_real_method(obj, self.print_method)
        344             if method is not None:
    --> 345                 return method()
        346             return None
        347         else:


    ~/Sites/jupyter/vdom/vdom/core.py in _repr_html_(self)
        242                     out.write(escape(safe_unicode(c)))
        243                 else:
    --> 244                     out.write(c._repr_html_())
        245 
        246             out.write('</{tag}>'.format(tag=escape(self.tag_name)))


    AttributeError: 'Counter' object has no attribute '_repr_html_'





    <vdom.core.VDOM at 0x10e9553a8>



## vdom vs. ipywidgets

[the complexity of ipywidgets, creating an ipywidget in the notebook vs. a vdom object]


```python
import ipywidgets as widgets
from IPython.display import display

# Create a counter component with local state and event handler
class Counter:
    def __init__(self):
        # Set the initial component state
        self.count = 0
        # Create a button widget that displays the count value
        self.widget = widgets.Button(description=str(self.count))
        # ...and increments the count on click
        self.widget.on_click((self.handle_click))

    def handle_click(self, widget):
        # Increment the count value in the component state
        self.count += 1
        # Update the button description
        self.widget.description = str(self.count)

    def _ipython_display_(self):
        # Inform ipython how to display this object
        display(self.widget)

Counter()
```


    Button(description='0', style=ButtonStyle())


### Advantages over ipywidgets

- **Declarative:** asdf
- **Composable**: Static text is very flexible and can be wrangled to accomplish just about anything, but unlike Python objects, it lacks interfaces, scope, and context which enable features like autocomplete, hover tips, and other feedback that make our jobs as coders so much more manageable. vdom objects, like React components, can be composed to create complex UIs.
- **Compatibility with React**: 

## React components

In addition to rendering native HTML elements, we can now render React components in vdom. This is possible thanks to ES modules landing in ECMAScript (Javascript) and in most browsers and [jspm.io](https://jspm.io/). Javascript now has first-class support for modules, so we can use the new import (`import MyModule from './my-module.js'`) and dynamic import (`import('./my-module.js').then(MyModule => { })`) syntax. jspm.io is a CDN service that serves packages from [npm](http://npmjs.com) and transforms them from CommonJS to ES module format. This allows us to dynamically import and use most npm packages in the browser. 

The vdom spec now accept a new property called `import`. Import expects a `package` property that is the npm package name and an optional `module` property that is the _named import_ from that package (defaults to `default`). 

Let's look at an example:

The above button is not a native HTML button, it's a `Button` React component provided by @blueprintjs/core. When an `import` property is included, the `tagName` property (which is a string that refers to a native HTML tag name) is overriden by a React component.

A few things to notice:

* We have included a new `import` key in our vdom object
* `import` allows the user to specify a React component to import and use in lieu of a native HTML element
* In the above example, we are specifying an npm package (@blueprintjs/core) and named import or _module_ from that package (Button)
* When an `import` key is provided, transform-vdom dynamically imports the specified module from npm (jspm.io) and replaces the vdom component's `tagName` property with the imported React component.

Next, let's create a convenient function that will allow us to import React components and use them like we would any other vdom component. Let's call this function `import_component` and let's model it after vdom's own `create_component`:

[using React elements in vdom was not possible]

[using React elements in vdom using modules]


```python
from IPython.display import HTML, Markdown

Markdown('---')
```




---




```python
Markdown.
```