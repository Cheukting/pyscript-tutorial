# Chapter 1 - Python in the browser

In this chapter, we will explore the basics of PyScript with some exercises. We will use the [template.html](template.html) as a base.

## Exercise 1 - Hello World

First, make a copy of the [template.html](template.html), you can name it something else like `hello_world.html`.

Then we inspect the file. See the line

```html
<link rel="stylesheet" href="https://pyscript.net/latest/pyscript.css" />
```
It is calling the stylesheet that PyScript may need to display some of the default elements like the Python REPL (which we will talk about in other exercises). It only affect the style of the REPL so if we are not using it you can opt-out of it or use your own style sheet.

The line that makes the PyScript magic happened is

```html
<script defer src="https://pyscript.net/latest/pyscript.js"></script>
```

And is needed if you want to use PyScript. In this chapter we are using the CDN version of it as above as it will always be the latest version. If you want to keep a certain version for your application, you can self host it. (Details to follow)

Now it's time to write some code. Python code can be written within the tags `<py-script>` and `</py-script>`.

Let's try to print the current time. Do you know how to do it? Try it before you see how it is done below.

```python
from datetime import datetime
print("The time now is:")
print(datetime.now())
```

When you are done, try opening the `hello_world.html` file on your browser of choice.

As you refresh, you see the time got updated. What if we want to have the time to be updated constantly? We will discover that in later exercises.

## Exercise 2 - Python REPL

Now, let's try to add a Python REPL in the webpage. If you want to keep the work of the previous exercises, feel free to make a copy. Now I will assume we just reuse the `hello_world.html` and continue from there.

Now below the `</py-script>` tag. Add a pair of tags like this:

```html
<py-repl>
</py-repl>
```

This will create a Python REPL on the webpage. Let's look at the browser to find out. If you are on a new file, open that file with a browser, otherwise you can just refresh to see the new changes.

Now by seeing the REPL, you can try to code Python in it. Try printing the time now from the REPL? Then click on the green arrow or press `shirt + enter`.

You can also add Python code between the the tags `<py-repl>` and `</py-repl>`, but they won't get executed right away like the previous exercise. They will be added to the REPL and you have to run them with the green arrow or press `shirt + enter`.

Now try adding the following line between the `<py-repl>` and `</py-repl>` tags:

```python
print(datetime.now()) # press shirt + enter to run
```

and refresh the page.

What if I want to keep the result of the previous REPL and have a new one after I executed the old one. You can activate that creature by adding `auto-generate="true"` inside the `<py-repl>` tag like this:

```html
<py-repl auto-generate="true">
```

Now save and refresh again to see the changes.

## Exercise 3 - Async

Remember we can print out the time but it does not update automatically? If we want to do so we need to do a bit of async code. Don't worry if you are not familiar with async code, we will do it here together and you can look deeper into [async code in Python](https://docs.python.org/3/library/asyncio.html) later if you are interested.

Now, let's go to the `hello_world.html` and use it as a starting point. If you had a copy of exercise 1, make a copy of it and we can start form there. If you are continuing form the last exercise, just delete the `<py-repl>` and `</py-repl>` tags and anything between them, we will start from there.

We need to modify the code inside the `<py-script>` and `</py-script>` tags. But first, we need to add a div element as `output` for the output, after the `<py-script>` and `</py-script>` tags, add this:

```html
<div id=output></div>
```

Then, we need to change our code, first, we need to import asyncio after importing datetime.

```python
from datetime import datetime
import asyncio
```

We can keep the first print statement but for the time itself, we need to update it, so, we will have to write into the div element `output` as `print` statement does not allow us to rewrite it.

```python
Element("output").write(str(datetime.now()))
```

Note that we have to convert `datetime.now()` into string as it is done automatically by the `print` statement but not he `wirte` method here.

Now, let's create an async function (or it is called coroutine) to make it looping over and over again. An async function (coroutine) is just like a function but with `async` in front of the `def`, like this:

```python
async def tick():
```

In this function, it want it to run and never end, so we:

1) use a while `True` loop;
2) then we do the `.write` into `output` (the line above) and finally;
3) sleep for a second before the next iteration

just like this:

```python
async def tick():
    while True:
      Element("output").write(str(datetime.now()))
      await asyncio.sleep(1)
```

Now we have our `tick`, we just need to put it in the PyScript event loop so it is running in the background non-stop. But before, remember the PyScript loader? The spinning thing when PyScript is loading. If we just put our `tick` in the event loop technically PyScript is never finish executing and the spinning will not go away. So we have to tell it to stop.

```python
pyscript_loader.close()
```

Then now we can put `tick` into the event loop:

```python
pyscript.run_until_complete(tick())
```

Did you follow? If you are not sure here is all the code within the `<py-script>` and `</py-script>` tags:


```python
from datetime import datetime
import asyncio
print("The time now is:")
async def tick():
    while True:
      Element("output").write(str(datetime.now()))
      await asyncio.sleep(1)
pyscript_loader.close()
pyscript.run_until_complete(tick())
```

When you are done, save and refresh (or open the file) and see the async magic happened.

## Exercise 4 - Adding packages

Up until now we can use Python with browser, but we have not include any extra Python packages where the true power of Python lies. Here we will see how we can use Pandas in the browser.

Let's start with the [template.html](template.html) again, make a copy of it. You can call it `using_pandas.html`.

First, if we would like to use any packages that does not come with Python by default (here by default Pyodide include most standard libraries of CPython). We would like to add that in. To do that, we will use the `<py-config>` tag, you can see the details in [the documentation here](https://github.com/pyscript/pyscript/blob/main/docs/tutorials/getting-started.md#the-py-config-tag).

Let's add this before the `<py-script>` tag:

```
<py-config>
  packages = ["pandas"]
</py-config>
```

Now we have make Pandas avaliable to our code within the `<py-script>` and `</py-script>` tags, but don't forget to import it. Then we can create a DataFrame. Put the following between the `<py-script>` and `</py-script>` tags:

```Python
import pandas as pd
d = {'col1': [1, 2], 'col2': [3, 4]}
df = pd.DataFrame(data=d)
df
```
