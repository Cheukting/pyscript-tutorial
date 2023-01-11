# Chapter 2 - Visualisation with PyScript

In this chapter, we will start using PyScript for some data visualisation. Like in the last chapter, we will start with a [template.html](template.html) which is a continuation from [exercise 5 of Chapter 1](/chapter_1/chapter_1.md#exercise-5---loading-a-file).

Moving forward we will assume you have already got the basic knowledge from [Chapter 1](/chapter_1/chapter_1.md) and know how to use some popular Python data handling and visualisation library like [Pandas](https://pandas.pydata.org/) and [Matplotlib](https://matplotlib.org/).

There will also be some basic knowledge about HTML and JavaScript required. We will try to explain and link the reference when we go over them. You can also find the reference at the [W3 school website](https://www.w3schools.com/).

---

## Exercise 1 - Generate a static plot

Before we see what interactivity PyScript can offer in visualising data, we will first get ourselves familiar with generate a plot like we do normally, for example using Matplotlib.

First, we need to add Matplotlib into our Python packages in `<py-config>`:

```
<py-config>
  packages = ["pandas", "matplotlib"]
</py-config>
```

After that, we can import `pyplot` as `plt` like we usually do. We also take the opportunity to adjust the figure size here:

```
import matplotlib.pyplot as plt
plt.rcParams["figure.figsize"] = (20,30)
```

Keep the code we had reading the csv as `df`, we now instead of showing `df`, we make a horizontal bar plot. We also specify the bar labels and title of the plot:

```
fig, ax = plt.subplots()
bars = ax.barh(df["name"], df["rating"], height=0.7)
ax.bar_label(bars)
plt.title("Rating of ice cream flavours of your choice")
fig
```

To recap, the code within the `<py-script>` and `</py-script>` tags pair looks like this:

```
from pyodide.http import open_url
import pandas as pd

import matplotlib.pyplot as plt
plt.rcParams["figure.figsize"] = (20,30)

df = pd.read_csv(open_url("https://raw.githubusercontent.com/Cheukting/pyscript-ice-cream/main/bj-products.csv"))

fig, ax = plt.subplots()
bars = ax.barh(df["name"], df["rating"], height=0.7)
ax.bar_label(bars)
plt.title("Rating of ice cream flavours of your choice")
fig
```

Now open the html or refresh the page in the web browser. After a bit of a wait (for loading Matplotlib in the background) you should see a horizontal bar plot. It does not looks very pretty but it works.

## Exercise 2 - Creating elements for user input

Now, we can work towards making our visualisation better. Who does not want a bit of interactivity? Before we can make our plot interactive, we need some element so user can control the plot.

Right after the `<body>` tag, we add two `<div>` elements there, the first one serves as the input area that we are going to work on in this exercise, while the second one serves as the output which we will be using in the future.

```
<div id="input"></div>
<div id="output"></div>
```

Let's focus on the input `<div>` element. We would like to add a bunch of [radio buttons](https://www.w3schools.com/html/html_form_elements.asp) that a user can choose what flavour of ice cream that they fancy. Let's just add one button and its label as our first step:

```
<div id="input">
  <input type="radio" id="chocolate" name="flavour" value="COCOA">
  <label for="chocolate"> Chocolate 🍫</label>
</div>
<div id="output"></div>
```

Note that we have the `name` as `flavour` and `value` as `COCOA`. This is important as it will be used in our code later. Now let's refresh the page to see how the button looks like. After the page has loaded with the graph, you can try clicking on the button and can see that it is now highlighted.

Now let's add all the buttons and labels as follow:

```
<div id="input">
  <input type="radio" id="all" name="flavour" value="ALL">
  <label for="all"> All 🍧</label>
  <input type="radio" id="chocolate" name="flavour" value="COCOA">
  <label for="chocolate"> Chocolate 🍫</label>
  <input type="radio" id="cherrie" name="flavour" value="CHERRIES">
  <label for="cherrie"> Cherries 🍒</label>
  <input type="radio" id="berries" name="flavour" value="BERRY">
  <label for="berries"> Berries 🍓</label>
  <input type="radio" id="cheese" name="flavour" value="CHEESE">
  <label for="cheese"> Cheese 🧀</label>
  <input type="radio" id="peanut" name="flavour" value="PEANUT">
  <label for="peanut"> Peanut 🥜</label>
</div>
<div id="output"></div>
```

Refresh and see all the options pop up in the page. Also note that when you try clicking on the buttons after loading, only one can be highlighted at a time.

One last step before we move on, we can style the input area and add extra text so it looks nicer:

```
<div id="input" style="margin: 20px;">
  Select your 🍨 flavour: <br/>
  <input type="radio" id="all" name="flavour" value="ALL">
  <label for="all"> All 🍧</label>
  <input type="radio" id="chocolate" name="flavour" value="COCOA">
  <label for="chocolate"> Chocolate 🍫</label>
  <input type="radio" id="cherrie" name="flavour" value="CHERRIES">
  <label for="cherrie"> Cherries 🍒</label>
  <input type="radio" id="berries" name="flavour" value="BERRY">
  <label for="berries"> Berries 🍓</label>
  <input type="radio" id="cheese" name="flavour" value="CHEESE">
  <label for="cheese"> Cheese 🧀</label>
  <input type="radio" id="peanut" name="flavour" value="PEANUT">
  <label for="peanut"> Peanut 🥜</label>
</div>
<div id="output"></div>
```

Try refreshing again. And if we are happy with how it looks like, we can now move on to the next exercise which we will make a new plot according which option is highlighted.

## Exercise 3 - Interactive new plot

Now things are start getting interesting. We will filter our data depending on the flavour selected. Notice the `value`s in the buttons above appears in the ingredient list in the ice cream flavours? We are going to filter out the ice cream flavours that has the selected ingredient in their list and only showing those in the plot.

First, we have to put the code that involving making a plot into a function so we can "redraw" the plot when our data changes. Remember in the first exercise, we have created the plot with this:

```
fig, ax = plt.subplots()
bars = ax.barh(df["name"], df["rating"], height=0.7)
ax.bar_label(bars)
plt.title("Rating of ice cream flavours of your choice")
fig
```

Now let's change it into a function:

```
def plot(data):
    fig, ax = plt.subplots()
    bars = ax.barh(data["name"], data["rating"], height=0.7)
    ax.bar_label(bars)
    plt.title("Rating of ice cream flavours of your choice")
    pyscript.write("output",fig)
```

Notice that in the last line, we are using `pyscript.write("output",fig)` instead of just `fig` as we have now a dedicated output `<div>` element for the plot. We also change the name `df` to `data` within the function to avoid confusion with the original `df`.

If we call the `plot` function with the original `df` it should give us the same thing as before now:

```
def plot(data):
    fig, ax = plt.subplots()
    bars = ax.barh(data["name"], data["rating"], height=0.7)
    ax.bar_label(bars)
    plt.title("Rating of ice cream flavours of your choice")
    pyscript.write("output",fig)

plot(df)
```

Let's try save the code and refresh the page. If the horizontal bar plot still shows as before, we are doing it right.

Next, we have to check which flavour has been selected. To do that, first we will gather all the button elements. We do it before `plot(df)`:

```
flavour_elements = document.getElementsByName("flavour")

plot(df)
```

Here, using [document.getElementsByName](https://www.w3schools.com/js/js_htmldom_elements.asp), `flavour_elements` will be a list of elements that has `flavour` as `name`, which is all our buttons.

Next, we will write some code to check each of the elements in `flavour_elements` to see if they are selected and if so we will filter the data accordingly. However, since we want to repeat this process every time the user click on the buttons, we need to write these code in a function, we can put it after the `flavour_elements` definition:

```
flavour_elements = document.getElementsByName("flavour")

def select_flavour():
    for ele in flavour_elements:
        if ele.checked:
            current_selected = ele.value
            break
    if current_selected == "ALL":
        plot(df)
    else:
        filter = df.apply(lambda x: ele.value in x["ingredients"], axis=1)
        plot(df[filter])
```

There is a lot to cover here. In the first for loop we go through all the elements in the `flavour_elements` and if any is checked, we store it in the `current_selected` it works because only one button can be selected at a time. Then, if the `current_selected` is `ALL` we will plot the original unfiltered `df` otherwise we will filter those that has the ingredient in the ingredient list and plot the filtered data.

One last thing about this function, we have to make it an [event handler](https://www.w3schools.com/js/js_htmldom_eventlistener.asp) as it is doing things when an event, user clicking on the button, happened.

To make a event handler in Python that can work with DOM, we have to use the `create_proxy` provided by `pyodide.ffi`. But first, we need our `select_flavour` function to take one parameter `event`, like this:

```
def select_flavour(event):
```

The rest of the function is unchanged. Then we have to import `create_proxy`, let's put it at the top of the Python code, together with the other imports:

```
from pyodide.ffi import create_proxy
from pyodide.http import open_url
import pandas as pd
```

Then we can create the event handlers using this line after the definition of `select_flavour`:

```
def select_flavour(event):
    for ele in flavour_elements:
        if ele.checked:
            current_selected = ele.value
            break
    if current_selected == "ALL":
        plot(df)
    else:
        filter = df.apply(lambda x: ele.value in x["ingredients"], axis=1)
        plot(df[filter])

ele_proxy = create_proxy(select_flavour)
```

Now we have `ele_proxy` which is our event handler when someone click on the buttons. Nest, we have to add [event listeners](https://www.w3schools.com/js/js_htmldom_eventlistener.asp) to all the button elements so it will trigger the event handler when being clicked on. Remember that we have all the button elements stored as `flavour_elements`, so we add a for loop like this:

```
ele_proxy = create_proxy(select_flavour)

for ele in flavour_elements:
    ele.addEventListener("change", ele_proxy)

plot(df)
```

Now, let's refresh the page in the browser and see the result. After loading, you should be able to see the plot changes after clicking the button. Bring up the JavaScript Console for debugging if needed.

The final result from exercise 1 to 3 is saved as [viz_with_matplotlib.html](viz_with_matplotlib.html) for your reference.

Congratulation, you have made your first interactive visualisation with PyScript. However, it is not the prettiest visualisation. Personally I like using D3 as it provide a very nice transition when we change the plot. From this point on, we will be using D3 for visualisation.

## Exercise 4 - Interactive plot with D3

[D3](https://d3js.org/) is a popular JavaScript library for generating quality interactive graphs. Now with PyScript we can use D3 together with the Python libraries that we are familiar with and love. We will not be explaining how to use D3 in details in this tutorial. If you want to learn more, please visit the [D3 website](https://d3js.org/) for the example and references.

We will start by modifying the work we have done so far. We will be replacing Matplotlib with D3 for plotting our horizontal bar plots. So first of all, we will remove Matplotlib dependency in our code:

```
<py-config>
  packages = ["pandas"]
</py-config>
```

Instead, we will be importing D3. However, since D3 is a JavaScript library, we will do it the JavaScript way. Add these after the `</py-config>` tag:

```
<script type="importmap">
{
  "imports": {
    "d3": "https://cdn.skypack.dev/pin/d3@v7.6.1-1Q0NZ0WZnbYeSjDusJT3/mode=imports,min/optimized/d3.js"
  }
}
</script>
<script type="module">
import * as d3 from "https://cdn.skypack.dev/pin/d3@v7.6.1-1Q0NZ0WZnbYeSjDusJT3/mode=imports,min/optimized/d3.js";
</script>
```

After that, we work on the Python code within the `<py-script>` and `</py-script>` tags pair. Replace import `pyplot`:

```
import matplotlib.pyplot as plt
```

with `d3`:

```
import d3
```

Now the D3 library is available to use with the Python code, with a few necessary conversion of objects between JavaScript and Python using `to_js` in `pyodide.ffi`. To make `to_js` available, we have to import it right after `create_proxy`:

```
from pyodide.ffi import create_proxy, to_js
```

As you may guess, the parameter setting with `plt` and the `plot` function that we define need to be change as well. We will do that step by step.

First, we have some initial settings for the plot. In Matplotlib, we just simply set the figure size. However, we want a better looking visualisation this time so it is more complicated in this D3 example. We replace this line:

```
plt.rcParams["figure.figsize"] = (20,30)
```

with the following:

```
viz = d3.select("#output")
viz.select(".loading").remove()

margin = {"top": 30, "right": 30, "bottom": 70, "left": 260}
width = 1000 - margin["left"] - margin["right"]
max_height = 1200 - margin["top"] - margin["bottom"]

svg = (viz
  .append("svg")
    .attr("width", width + margin["left"] + margin["right"])
    .attr("height", max_height + margin["top"] + margin["bottom"])
  .append("g")
    .attr("transform",
          f"translate({margin['left']},{margin['top']})")
  )

x = d3.scaleLinear().domain([0, 5]).range([0, width])
xAxis = svg.append("g").call(d3.axisBottom(x))

y = d3.scaleBand()
yAxis = svg.append("g").call(d3.axisLeft(y))
```

The first 2 lines, we set the visualisation to be display at the `output` element in our HTML and remove the loading animation in D3. Then, we are setting the margin and the size of the figure. We also initialise 5 elements: `svg`, `x`, `xAxis`, `y` and `yAxis`, some of which we will manipulate when plotting.

Next, we will replace the code in the `plot` function. Instead of:

```
def plot(data):
    fig, ax = plt.subplots()
    bars = ax.barh(data["name"], data["rating"], height=0.7)
    ax.bar_label(bars)
    plt.title("Rating of ice cream flavours of your choice")
    pyscript.write("output",fig)
```

Now we have:

```
def plot(data):
    data_js = to_js(data.to_dict(orient="records"))
    height = data["name"].count() * 100 - margin["top"] - margin["bottom"]
    if height > max_height:
        height = max_height

    (
    svg.transition().duration(1000)
      .attr("width", width + margin["left"] + margin["right"])
      .attr("height", height + margin["top"] + margin["bottom"])
    )

    # x-axis
    xAxis.transition().duration(1000).attr("transform", f"translate(0,{height})").call(d3.axisBottom(x))

    # y-axis
    y.domain(list(data["name"])).range([0, height]).padding(.1)
    yAxis.transition().duration(1000).call(d3.axisLeft(y))

    # bars
    y_fn = create_proxy(lambda record, *_:y(record["name"]))
    x_fn = create_proxy(lambda record, *_:x(record["rating"]))
    u = svg.selectAll("rect").data(data_js)

    (
      u.enter()
      .append("rect")
      .merge(u)
      .transition()
      .duration(1000)
      .attr("x", x(0) )
      .attr("y", y_fn)
      .attr("width", x_fn)
      .attr("height", y.bandwidth() )
      .attr("fill", "#69b3a2")
    )
    u.exit().remove()

    # label
    label_fn = create_proxy(lambda record, *_:record["rating"])
    labels = svg.selectAll(".label").remove()
    l = svg.selectAll(".text").data(data_js)

    (
  	  l.enter()
  	  .append("text")
      .merge(l)
      .transition()
      .duration(1000)
  	  .attr("class","label")
  	  .attr("x", x_fn)
  	  .attr("y", y_fn)
  	  .attr("dy", "1.5em")
  	  .text(label_fn)
    )
    l.exit().remove()

    (
    svg.append("text")
        .attr("x", (width / 2))
        .attr("y", 0 - (margin["top"] / 2))
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .text("⭐ Ice cream flavour ratings ⭐")
    )
```

Which looks like a lot since we are controlling every single elements in the plot and their transition as well. We won't goes into details here, feel free to look up the examples and reference in the [D3 documentation](https://d3js.org/) for the explanation. However, there's one thing to pay attention here. The first line

```
data_js = to_js(data.to_dict(orient="records"))
```

is essential for what we are doing here. Since `data` is a Pandas `DataFrame`, therefore it is a Python object, we need to convert it to a JavaScript object so that the D3 library understand. We first convert into a dictionary in Python using `to_dict` method in Pandas `DataFrame`, then we use `to_js` to convert the dictionary into a JavaScript object.

Now, we may refresh the browser and see the interactive D3 graph. The transition animation of the plot is so addictive to watch. The completed result is at [viz_with_d3.html](viz_with_d3.html) for your reference.
