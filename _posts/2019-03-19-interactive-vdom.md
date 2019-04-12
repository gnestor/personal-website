---
title: "Introducing Interactive VDOM"
---

[vdom](https://github.com/nteract/vdom/) is a Python library for expressing UI (user interface) using the [virtual dom](https://reactjs.org/docs/faq-internals.html#what-is-the-virtual-dom) popularized by Javascript libraries such as [React](https://reactjs.org/). 

vdom is designed to improve the experience of rendering custom UI within the context of a Jupyter notebook and is currently supported in [JupyterLab](https://github.com/jupyterlab/jupyterlab) and [nteract](https://github.com/nteract/nteract).

# A simple example

The following is an example of how to render some simple UI using vdom:

    from vdom import div, h1, p, ul, li
    
    div(
        h1('A Header'),
        p('A list of numbers'),
        ul([li(str(num)) for num in range(10)])
    )

![](Untitled-b18d6eea-044e-41f6-832b-cef0ed560c89.png)

*Notice how the `li` elements are dynamically generated using a Python list comprehension expression.*

To fully appreciate vdom, we must understand how we accomplished the same results before vdom. The following is an example that uses ipython's `HTML` display function.

    from IPython.display import HTML
    
    items = [f'<li>{num}</li>' for num in range(10)]
    
    HTML(f'''<div>
    <h1>A Header</h1>
    <p>A list of random numbers</p>
    <ul>{''.join(items)}</ul>
    </div>''')

![](Untitled-54152823-a869-41a8-99bb-e025495ac85f.png)

The above works but take a moment to ponder the differences...

In the vdom example, we use Python functions (vdom's helper functions) such as `div` and `h1` to construct a *virtual DOM tree*. In the HTML example, we construct a single string of HTML markup to create DOM tree. Not only is the vdom example more Pythonic, but it has many advantages.

# Advantages of vdom

Using vdom is like using React or other virtual dom Javascript libraries in Python. As a result, it has many of the advantages that React has over plain HTML:

- **Dynamic:** In the HTML example, we are simply constructing static HTML markup in Python. In the vdom example, we are creating live vdom objects (or *components*) that are interactive and updatable.
- **Interactive:** Using the HTML function, the only way to handle events such as clicks and keys is to inline Javascript in our static HTML markup. Using vdom, we can use Python functions to handle events by simply passing them as attributes to the component (e.g. `button('click me', onClick=lambda _: print('clicked'))`).
- **Stateful:** Using the HTML function, it would be very difficult to update the UI in response to a state change in Python and it would be impossible to update the state in Python in response to an event on the front-end (this requires communication between the front-end and kernel). Using vdom, it's straight-forward to both update the UI in response to a state change in Python and update the state in Python in response to an event on the front-end (see the [Interactive and stateful example](https://www.notion.so/Introducing-module-support-to-vdom-5ddc063236d6409db1be41da3f1af582#89f889f8962a4562979d71912e88b1fd) below).

# Interactive and stateful example

The following is an example of how to add interactivity and state to a vdom component:

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

One great thing about React is its focus on *components*. A React component contains all the markup for a unit of UI (e.g. a drop-down box) in addition to any event handlers, local state, and/or local methods. This makes React components *composable* meaning that they can be easily dropped into a component tree and if given the required *props* (or properties), just work! We can turn this counter example into a React-like component so that it can be easily reused in other contexts.

First, we will create a base class that components and subclass or inherit from. This will implement React-like props and state, a render function that will be overridden, and an `_ipython_display_` method that will inform ipython about how to display this object.

    class BaseComponent(object):
        def __init__(self, **props):
            self.props = props
            self.state = {}
            self.display_handle = None
            
        def set_state(self, state):
            self.state.update(state)
            if self.display_handle:
                self.display_handle.update(self.render())
            
        def render(self):
            return None
            
        def _ipython_display_(self):
            self.display_handle = display(self.render(), display_id=True)

Next, we can subclass `BaseComponent` and create a `Counter` component.

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

We've successfully create an interactive and stateful vdom component that can be used like any other vdom component.

# vdom vs. ipywidgets

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

[the complexity of ipywidgets, creating an ipywidget in the notebook vs. a vdom object]

- **Declarative:** asdf
- **Composable**: Static text is very flexible and can be wrangled to accomplish just about anything, but unlike Python objects, it lacks interfaces, scope, and context which enable features like autocomplete, hover tips, and other feedback that make our jobs as coders so much more manageable. vdom objects, like React components, can be composed to create complex UIs.

# vdom modules

[using React elements in vdom was not possible]

[using React elements in vdom using modules]

[the complexity of creating a ipywidget that depends on a javascript library, non-native elements]

[React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime)