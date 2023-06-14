# Chapter 2 - Visualisation with PyScript

In this chapter, we will start using PyScript for some data visualisation. Like in the last chapter, we will start with a [template.html](template.html) which is a continuation from [exercise 5 of Chapter 1](/chapter_1/chapter_1.md#exercise-5---loading-a-file).

Moving forward we will assume you have already got the basic knowledge from [Chapter 1](/chapter_1/chapter_1.md) and know how to use some popular Python data handling and visualisation library like [Pandas](https://pandas.pydata.org/) and [Matplotlib](https://matplotlib.org/). We will also use [NetworkX](https://networkx.org/documentation/stable/index.html) in the last part of this chapter.

There will also be some basic knowledge about HTML and JavaScript required. We will try to explain and link the reference when we go over them. You can also find the reference at the [W3 school website](https://www.w3schools.com/).

In the later parts of this chapter, we will use the [D3 library]((https://d3js.org/). So it is required that you either have some basic knowledge of it or feel comfortable digging through their documentation to understand how the library works yourself.

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

```Python
import matplotlib.pyplot as plt
plt.rcParams["figure.figsize"] = (20,30)
```

Keep the code we had reading the csv as `df`, we now instead of showing `df`, we make a horizontal bar plot. We also specify the bar labels and title of the plot:

```Python
fig, ax = plt.subplots()
bars = ax.barh(df["name"], df["rating"], height=0.7)
ax.bar_label(bars)
plt.title("Rating of ice cream flavours of your choice")
fig
```

To recap, the code within the `<py-script>` and `</py-script>` tags pair looks like this:

```Python
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

```Python
fig, ax = plt.subplots()
bars = ax.barh(df["name"], df["rating"], height=0.7)
ax.bar_label(bars)
plt.title("Rating of ice cream flavours of your choice")
fig
```

Now let's change it into a function:

```Python
def plot(data):
    fig, ax = plt.subplots()
    bars = ax.barh(data["name"], data["rating"], height=0.7)
    ax.bar_label(bars)
    plt.title("Rating of ice cream flavours of your choice")
    Element("output").clear()
    display(fig,target="output")
```

Notice that in the last line, we are using `pyscript.write("output",fig)` instead of just `fig` as we have now a dedicated output `<div>` element for the plot. We also change the name `df` to `data` within the function to avoid confusion with the original `df`.

If we call the `plot` function with the original `df` it should give us the same thing as before now:

```Python
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

```Python
flavour_elements = js.document.getElementsByName("flavour")

plot(df)
```

Here, using [document.getElementsByName](https://www.w3schools.com/js/js_htmldom_elements.asp) from JavaScript (that's why we have the `js.` suffix), `flavour_elements` will be a list of elements that has `flavour` as `name`, which is all our buttons.

Next, we will write some code to check each of the elements in `flavour_elements` to see if they are selected and if so we will filter the data accordingly. However, since we want to repeat this process every time the user click on the buttons, we need to write these code in a function, we can put it after the `flavour_elements` definition:

```Python
flavour_elements = js.document.getElementsByName("flavour")

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

```Python
def select_flavour(event):
```

The rest of the function is unchanged. Then we have to import `create_proxy`, let's put it at the top of the Python code, together with the other imports:

```Python
from pyodide.ffi import create_proxy
from pyodide.http import open_url
import pandas as pd
```

Then we can create the event handlers using this line after the definition of `select_flavour`:

```Python
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

```Python
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

Instead, we will be importing D3. However, since D3 is a JavaScript library, we will do it the JavaScript way. Add this line after the `</py-config>` tag:

```
<script src="https://d3js.org/d3.v7.min.js"></script>
```

After that, we work on the Python code within the `<py-script>` and `</py-script>` tags pair. Replace import `pyplot`:

```Python
import matplotlib.pyplot as plt
```

with import `d3` from `js`:

```Python
from js import d3
```

Now the D3 library is available to use with the Python code, with a few necessary conversion of objects between JavaScript and Python using `to_js` in `pyodide.ffi`. To make `to_js` available, we have to import it right after `create_proxy`:

```Python
from pyodide.ffi import create_proxy, to_js
```

As you may guess, the parameter setting with `plt` and the `plot` function that we define need to be change as well. We will do that step by step.

First, we have some initial settings for the plot. In Matplotlib, we just simply set the figure size. However, we want a better looking visualisation this time so it is more complicated in this D3 example. We replace this line:

```Python
plt.rcParams["figure.figsize"] = (20,30)
```

with the following:

```Python
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

```Python
def plot(data):
    fig, ax = plt.subplots()
    bars = ax.barh(data["name"], data["rating"], height=0.7)
    ax.bar_label(bars)
    plt.title("Rating of ice cream flavours of your choice")
    pyscript.write("output",fig)
```

Now we have:

```Python
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

```Python
data_js = to_js(data.to_dict(orient="records"))
```

is essential for what we are doing here. Since `data` is a Pandas `DataFrame`, therefore it is a Python object, we need to convert it to a JavaScript object so that the D3 library understand. We first convert into a dictionary in Python using `to_dict` method in Pandas `DataFrame`, then we use `to_js` to convert the dictionary into a JavaScript object.

Now, we may refresh the browser and see the interactive D3 graph. The transition animation of the plot is so addictive to watch. The completed result is at [viz_with_d3.html](viz_with_d3.html) for your reference.

## Exercise 5 - D3 with NetworkX

Things are getting more interesting. Since now we can use D3, we can use it to create more complicated and interactive visualisation. One type of the visualisation would really benefit from interactivity is network graph.

I hope you have used [NetworkX](https://networkx.org/documentation/stable/index.html) before, it is a very good tool to analyse network graphs. However, as usually a network graph would involve a lot of nodes and edges, presenting them nicely would require a lot of customisation and interactivity.

So, what if we combine the usage of NetworkX and D3?

In this exercise, we will start with the [graph_template.html](graph_template.html). Since we already learnt how to import Python libraries and D3 we are not going to repeat here.

First, we will import the [Zachary’s Karate Club graph](https://networkx.org/documentation/stable/reference/generated/networkx.generators.social.karate_club_graph.html#karate-club-graph) as our example here. We also export lists of nodes and edges for our visualisation later.

```Python
G = nx.karate_club_graph()
node_list = [{"id": node[0], "name": node[1]} for node in G.nodes(data="club")]
edge_list = [{"source": edge[0], "target": edge[1]} for edge in nx.to_edgelist(G)]
```

Before doing any visualisation, we would like to display some graph analysis result from NetworkX, like average node connectivity, radius of the graph or the centre nodes etc. This can be done with the NetworkX functions and changing the inner test of the `stat` element:

```Python
avg_conn = nx.average_node_connectivity(G)
graph_radius = nx.radius(G)
center_nodes = nx.center(G)

stat = Element("stat")
stat.element.innerText = f"""Avg. node connectivity is {avg_conn}
Radius of the graph is {graph_radius}
Center nodes are: {center_nodes}"""
```

Now save and open the html page in the browser. We should now see the results display correctly.

Moving on, let's create our D3 network graph visualisation. We will follow [the example here on the D3 Graph Gallery](https://d3-graph-gallery.com/graph/network_basic.html). Please also note that the D3 version that we are using are [v7](https://devdocs.io/d3~7/).

To set up the "canvas" area in the `viz` div element, we do something similar with our previous D3 examples:

```Python
margin = {"top": 30, "right": 30, "bottom": 70, "left": 40}
width = 1000 - margin["left"] - margin["right"]
max_height = 800 - margin["top"] - margin["bottom"]

viz = d3.select("#viz")
viz.select(".loading").remove()

svg = (viz
  .append("svg")
    .attr("width", width + margin["left"] + margin["right"])
    .attr("height", max_height + margin["top"] + margin["bottom"])
  .append("g")
    .attr("transform",
          f"translate({margin['left']},{margin['top']})")
  )
```

Then we need to prepare the data. We have the `node_list` and `edge_list` that we created earlier, however, they are Python objects. So we need to convert them using `to_js`. We also create a proxy of a lambda function to extract the `id` of the nodes.

```Python
nodes_js = to_js(node_list)
edges_js = to_js(edge_list, dict_converter=js.Object.fromEntries)
id_fn = create_proxy(lambda d, *_:d["id"])
```

Now, let's create the links and nodes in the visualisation, one thing that is different here from the Graph Gallery is that instead of using simple circles as the nodes, we create an object `g`. We then add of a circle and a text label for each of them. This way we will have nodes that has the `id` as labels.

```Python
link = (svg
  .selectAll("line")
  .data(edges_js)
  .enter()
  .append("line")
    .style("stroke", "#aaa")
)

node = (svg
  .selectAll(".node")
  .data(nodes_js)
  .enter()
  .append("g")
    .attr("class", "node")
)

node.append("circle").attr("r", 20).style("fill", "#69b3a2")
node.append("text").attr("text-anchor", "middle").attr("dy", ".35em").text(id_fn)
```

Save and refresh the html page. Now we see the nodes, however, they are all stacked up on top of each others so we only see the node 33 hanging there.

Next, we want the nodes to be spread out and the links are at the right position. To do that, first we set up a listener called `ticked`, just like the example in Graph Gallery does:

```Python
def ticked():
    (link
      .transition().duration(1000)
      .attr("x1", create_proxy(lambda d, *_:d.source.x))
      .attr("y1", create_proxy(lambda d, *_:d.source.y))
      .attr("x2", create_proxy(lambda d, *_:d.target.x))
      .attr("y2", create_proxy(lambda d, *_:d.target.y))
    )
    (node
      .transition().duration(1000)
      .attr("transform", create_proxy(lambda d, *_:f"translate({d.x},{d.y})"))
    )

ticked_fn = create_proxy(ticked)
```

Here compare to the Graph Gallery example, we also added a transition animation and we have to use `create_proxy` since technically we are coding in Python.

Then we create the force simulation in D3. We use slightly difference setting than the Graph Gallery and if you want to know more about force simulation, you can check out [the documentation here](https://devdocs.io/d3~7-force/). Remember to use the listener proxy `ticked_fn` we created:

```Python
simulation = (d3.forceSimulation(nodes_js)
  .force("link", d3.forceLink().id(id_fn).links(edges_js))
  .force("charge", d3.forceManyBody().strength(-400))
  .force('collide',d3.forceCollide().radius(30).iterations(2))
  .force("center", d3.forceCenter(width / 2, max_height / 2))
  .on("tick", ticked_fn)
  )
```

Now save and refresh the page. After loading and the D3 calculation (it take some time for the D3 calculation), the nodes should spread out and there are links that connect the nodes that are connected. You can check the final result at [networkx_with_d3.html](networkx_with_d3.html).

## Exercise 6 - Interactive network graph

Now we have a nice looking graph, however, we are not stopping here. It would not be complete without some interactivity. It would be useful when click on a node, we would be able to see which nodes are its neighbours. So let's do that.

First, we would like to add event handlers `clicked` for the nodes. When a node is clicked, we will store the `id` of the node that has been clicked and its neighbours, provided by NetworkX, as global variables. We would also want to restart the simulation so the graph apperance can be updated. Add these code before we initialise the links and nodes:

```Python
highlight_node = None
neighbors = None

def clicked(event, d):
  global highlight_node, neighbors
  highlight_node = d["id"]
  neighbors = list(G.neighbors(highlight_node))
  simulation.restart().on("tick", ticked_fn)

clicked_proxy = create_proxy(clicked)
```

Then when we initialise the nodes, add the event handler proxy `clicked_proxy`:

```Python
node = (svg
  .selectAll(".node")
  .data(nodes_js)
  .enter()
  .append("g")
    .attr("class", "node")
    .on("click", clicked_proxy)
)
```

Next, we want to change the colour of the node label depending of if it's the node that has been clicked or it is one of the neighbours. To do that, we first create a function `highlight`. Add these definition before we define `ticked`:

```Python
def highlight(d, *_):
    id = d["id"]
    if (highlight_node is not None) and (id == highlight_node):
        return "#f00"
    elif (neighbors is not None) and (int(id) in neighbors):
        return "#00f"
    else:
        return "#000"
```

For the node transition in `ticked`, we will not just move them to their position, we will also change the `fill` colour depending of the `highlight` of the node (since `highlight` is a Python function, we need to create a proxy before passing to D3):

```Python
def ticked():
    (link
      .transition().duration(1000)
      .attr("x1", create_proxy(lambda d, *_:d.source.x))
      .attr("y1", create_proxy(lambda d, *_:d.source.y))
      .attr("x2", create_proxy(lambda d, *_:d.target.x))
      .attr("y2", create_proxy(lambda d, *_:d.target.y))
    )
    (node
      .transition().duration(1000)
      .attr("transform", create_proxy(lambda d, *_:f"translate({d.x},{d.y})"))
      .attr("fill", create_proxy(highlight))
    )
```

Now save and refresh the page. After the nodes are spread out, try clicking on one of the nodes to see the highlighting effect.

That's great! However, there is one last thing that we can improve the visualisation. If you have seen the [Karate Club Gallery in NetworkX](https://networkx.org/documentation/stable/auto_examples/graph/plot_karate_club.html#sphx-glr-auto-examples-graph-plot-karate-club-py), the nodes are arranged nicely in a circle. Now the position of the nodes looks very random. What if we add a toggle to change the formation of the nodes, between the position generated by D3 forces and forming a circle like the one in the NetworkX Gallery, when we click on the background of the visualisation.

First, we will need a few more global variables to help us. After defining `highlight_node` and `neighbors`, add these:

```Python
circle_mode = False
node_clicked = False
```

The `circle_mode` is used to keep track of the toggle state and `node_clicked` help us marked if we are clicking on the background or the nodes.

We also need a helper function `get_cir_coor` to calculate the coordinates of the nodes when they form a circle. Put its definition after we define `highlight`:

```Python
def get_cir_coor(id):
    node_num = len(node_list)
    radius = max_height / 2
    rad = 2 * math.pi * (id / node_num)
    return radius * math.sin(rad) + width / 2, radius * math.cos(rad) + max_height / 2
```

By having the `id` of the node, `get_cir_coor` returns the x and y coordinates of the node. Note that we use the Python standard library `math` to help us do the calculation here. So we need to import it. Put this at the very beginning of our Python code:

```Python
import math
```

Next we will modify `ticked`, as now the coordinates of the nodes and links are depending which mode that we are in. If we are not in `circle_mode`, we did what we did before, otherwise, we will use `get_cir_coor` to get the coordinates instead:

```Python
def ticked():
  if not circle_mode:
      (link
        .transition().duration(1000)
        .attr("x1", create_proxy(lambda d, *_:d.source.x))
        .attr("y1", create_proxy(lambda d, *_:d.source.y))
        .attr("x2", create_proxy(lambda d, *_:d.target.x))
        .attr("y2", create_proxy(lambda d, *_:d.target.y))
      )
      (node
        .transition().duration(1000)
        .attr("transform", create_proxy(lambda d, *_:f"translate({d.x},{d.y})"))
        .attr("fill", create_proxy(highlight))
      )
  else:
      (link
        .transition().duration(1000)
        .attr("x1", create_proxy(lambda d, *_:get_cir_coor(d.source["id"])[0]))
        .attr("y1", create_proxy(lambda d, *_:get_cir_coor(d.source["id"])[1]))
        .attr("x2", create_proxy(lambda d, *_:get_cir_coor(d.target["id"])[0]))
        .attr("y2", create_proxy(lambda d, *_:get_cir_coor(d.target["id"])[1]))
      )
      (node
        .transition().duration(1000)
        .attr("transform", create_proxy(lambda d, *_:f"translate{get_cir_coor(d['id'])}"))
        .attr("fill", create_proxy(highlight))
      )
```

We are almost done. The last thing to do the event handler. We will create `change` and its proxy to toggle the global variable `circle_mode` and restart the simulation. Add these at the end of our Python code:

```Python
def change(event):
    global circle_mode, node_clicked
    if not node_clicked:
      circle_mode = not circle_mode
      simulation.restart().on("tick", ticked_fn)
    else:
      node_clicked = False

change_proxy = create_proxy(change)
```

Note that we only toggle the `circle_mode` and restart the simulation when none of the nodes are clicked, otherwise we will reset the `node_clicked` instead. This reminds us to change the `node_clicked` to `True` when any nodes are clicked. That is done within `clicked`(remember to use the global `node_clicked`):

```Python
def clicked(event, d):
  global highlight_node, neighbors, node_clicked
  highlight_node = d["id"]
  neighbors = list(G.neighbors(highlight_node))
  node_clicked = True
  simulation.restart().on("tick", ticked_fn)
```

Then we can add the event handler to the element `viz`:

```Python
graph_ele = js.document.getElementById("viz")
graph_ele.addEventListener("click", change_proxy)
```

Finally, we can save and refresh the page to see our master piece. Click on the nodes to see the highlighting, it should works as before. Also click on the spaces other than the nodes to see the formation toggles. The finished visualisation is at [interactive_network_graph.html](interactive_network_graph.html) for your reference.

---

This concludes Chapter 2 of this workshop. To continue learning how to deploy machine learning modes in the frontend, please go to [Chapter 3](/chapter_3/chapter_3.md)
