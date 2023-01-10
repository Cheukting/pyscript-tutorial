flavour# Chapter 2 - Visualisation with PyScript

In this chapter, we will start using PyScript for some data visualisation. Like in the last chapter, we will start with a [template.html](template.html) which is a continuation from exercise 5 of [Chapter 1](/chapter_1/chapter_1.md#exercise-5---loading-a-file).

Moving forward we will assume you have already got the basic knowledge from [Chapter 1](/chapter_1/chapter_1.md) and know how to use some popular Python data handling and visualisation library like [Pandas](https://pandas.pydata.org/) and [Matplotlib](https://matplotlib.org/).

There will also be some knowledge about html elements required. You can find the reference of them at the [W3 school website](https://www.w3schools.com/html/).

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
