reasonable# Chapter 3 - Deploying ML model

Before we start this chapter, there are some preparation work to do. First of all, we need a ML model to deploy. More precisely, a KNN recommender model to deploy. In this chapter's file, there is a `knn_recommender.py` and `requirements.txt` already prepared for you. We will show you the steps to train the model and get the needed pickle files, however we are not going to explain the construct of the model in details.

After training our model, we will deploy it on a `html` file using pyscript and let users to get movie recommendations.

We will assume you have already completed [Chapter 1](/chapter_1/chapter_1.md) and [Chapter 2](/chapter_2/chapter_2.md) got the basic knowledge of using pyscript.

---

## Exercise 0 - Prepare the data and train the model

In this example, we will be using [MovieLens Datasets](https://grouplens.org/datasets/movielens/latest/) to build a recommender system based on KNN Item-Based Collaborative Filtering. Get the full data set from [MovieLens Datasets](https://files.grouplens.org/datasets/movielens/ml-latest.zip) and put the movies.csv and ratings.csv in this folder.

Create a new Python environment, for example, using `venv`:

```
python3 -m venv .venv
```

then you can activate it, like this in Unix system (macOS and Linux):

```
source .venv/bin/activate
```

or like this in Windows:

```
.venv\Scripts\activate
```

After that, install all the requirements:
```
pip install -r requirements.txt
```

Run the code in `knn_recommender.py` to train the model. For example using this command:

```
python knn_recommender.py --movie_name "Iron Man" --top_n 10
```

It may take a while to run. When it is finished, it will show you the recommendataion and there will be two files, `hashmap.p` and `movie_user_mat_sparse.p`, generated. The option `--movie_name` and `--top_n` does not matter as we do not care about the result. We only want to run this code once to generate the `hashmap.p` and `movie_user_mat_sparse.p` which we will be using for the deployment in the following exercises.

## Exercise 1 - Using fetch and launching a server

We can start with `template.html`. The first thing we want to is to add the `hashmap.p` and `movie_user_mat_sparse.p` to our application (this will load the files into the web browser VM, without doing it, the application will not be able to access those files). In pyscript, it is straightforward. It can be done with `[[fetch]]` in `<py-config>`. In `<py-config>` tag pairs:

```
<py-config>
  packages = ["pandas", "scikit-learn", "fuzzywuzzy"]
  [[fetch]]
  files = ["hashmap.p", "movie_user_mat_sparse.p"]
</py-config>
```

Now if we save and open the html file, it will take a while to load in everything, however, there is also an error message:

```
(PY0001): PyScript: Access to local files (using "Paths:" in <py-config>) is not available when directly opening a HTML file; you must use a webserver to serve the additional files.
```

Don't panic and let me explain. What we are doing here is to ask pyscript to load in files from your local computer, in web browser world it is a big no no for security reason. Imagine you visit a website and it can see all the files you have on your computer.

So what are we going to do? This web app is created by us and is not "insecure", how can we load in a file that we need? Now think of what a normal website would be. The webpages are served from a server but not your local drive. So what we can do is to start a local server on your computer to serve your own application (for local use only).

To do that, we can simply use the `http.server` standard library module. Run `python3 -m http.server` at the `chapter_3` directory. Now open your browser and go to `http://0.0.0.0:8000/`. You should see all the files in the directory, click on the html file that you were working on.

Now it will take it's time to load again. But when it finished, the error message should not be there and the problem has been fixed. At the point the web app is not finished yet so it does nothing, we will keep working on it in the following exercises.

## Exercise 2 - Set the action for the button

When we start with the template, we have already added some interactive elements. One of them is the `Recommend!` button. Before we make the recommendation work, lets make sure we can have interactivity of this button with our Python code.

Remember in [Chapter 2](/chapter_2/chapter_2.md) when we work with interactive elements, we need event handlers. To do that, we will add a function that just print out "Recommended!" like this:

```
def recommend(evt=None):
  print("Recommend!")
```

Notice that we need the `evt=None` argument for this function as it is expected for all the event handler to have it. Without it, we will have an error saying we are missing an argument.

Next, we need to create the element in the code, as the element id of the button is `run-btn` we will create the element like this:

```
run_btn = Element('run-btn')
```

After that, we need to add the `onclick` event, our `recommend` function, to the `run_btn` element:

```
run_btn.element.onclick = recommend
```

Combining them, the code with in the `<py-script>` tag pairs looks like this:

```
<py-script>
  def recommend(evt=None):
    print("Recommend!")
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
```

Save and refresh the page, press the `Recommend!` button and see what happened.

## Exercise 3 - Taking input values

Besides the `Recommend!` button, we also have the input boxes `fav-movie` and `top-n`, lets see how we can get the values from them.

You can do similar thing like in the previous exerces, create 2 elements corresponding to the boxes:

```
fav_movie_box = Element('fav-movie')
top_n_box = Element('top-n')
```

and get their values:

```
fav_movie = fav_movie_box.value
top_n = top_n_box.value
```

However, I prefer to do it in one go:

```
movie_name = Element('fav-movie').value
top_n = Element('top-n').value
```

Then, we can use those values in our output:

```
print("Recommend!")
print(f"{top_n} recommendation based on {movie_name}")
```

Now code with in the `<py-script>` tag pairs looks like this:
```
<py-script>
  def recommend(evt=None):
    movie_name = Element('fav-movie').value
    top_n = Element('top-n').value
    print("Recommend!")
    print(f"{top_n} recommendation based on {movie_name}")
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
```

Refresh the page and try it out, first with "Iron man" and "10", it looks alright, however, you may spot a problem that we may have. Now try "Iron man" and "random" as input. The code will take the value as `string` by default.

There are many ways of handling it, for example try converting `top_n` to integer if failed give user a warning. For simplicity here, we will assume the default of `top_n` is 10 and if anything failed, we will just use 10 as the `top_n` value:

```
try:
  top_n = int(top_n)
except:
  top_n = 10
```

So we have now:

```
<py-script>
  def recommend(evt=None):
    movie_name = Element('fav-movie').value
    top_n = Element('top-n').value
    try:
      top_n = int(top_n)
    except:
      top_n = 10
    print("Recommend!")
    print(f"{top_n} recommendation based on {movie_name}")
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
```

Save and refresh and try again. See what "Iron man" and "random" gives you this time.

## Exercise 4 -- Give recommendations

To use our trained recommender, we need to first include the `knn_recommender.py` in `fetch` so our application have access to it:


```
<py-config>
  packages = ["pandas", "scikit-learn", "fuzzywuzzy"]
  [[fetch]]
  files = ["hashmap.p", "movie_user_mat_sparse.p", "knn_recommender.py"]
</py-config>
```

This allow us to import the `KnnRecommender`:

```
from knn_recommender import KnnRecommender
```

Save and refresh the page, you will see a warning about installing python-Levenshtein, ignore it for now as we will not be using it.

Next, in the `recommend` function, we want to create the `KnnRecommender` instances and set the model parameters

```
recommender = KnnRecommender()
recommender.set_model_params(20, 'brute', 'cosine', -1)
```

Now, instead of printing the input:

```
print("Recommend!")
print(f"{top_n} recommendation based on {movie_name}")
```

We want to make recommendations:

```
print(recommender.make_recommendations(movie_name, top_n))
```

However, if `movie_name` is empty, there will be an error in the recommender. To avoid it, we better check if the `movie_name` is empty:

```
if movie_name:
  print(recommender.make_recommendations(movie_name, top_n))
else:
  print("Sorry we need a movie name.")
```

Combining everything, now in the `<py-script>` tag pairs we should have:

```
<py-script>
  from knn_recommender import KnnRecommender
  def recommend(evt=None):
    movie_name = Element('fav-movie').value
    top_n = Element('top-n').value
    try:
      top_n = int(top_n)
    except:
      top_n = 10
    recommender = KnnRecommender()
    recommender.set_model_params(20, 'brute', 'cosine', -1)
    if movie_name:
      print(recommender.make_recommendations(movie_name, top_n))
    else:
      print("Sorry we need a movie name.")
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
````

Save and refresh and get some recommendation from the recommender.

## Exercise 5 -- Better presentation

Now everything works, however, the presentation is not the best, it just print out the result in a command line style. To make it worse, it prints out everything including the warning where the users should not be seeing. We are going to fix it in the exercise.

Now instead of printing the output like this:

```
print(recommender.make_recommendations(movie_name, top_n))
```

We turn on the `recommendation_only` flag and store the result:

```
recommendation = recommender.make_recommendations(movie_name, top_n, recommendation_only=True)
```

As we are developing, let first print out `recommendation` and see how it looks like. Now we have:

```
<py-script>
  from knn_recommender import KnnRecommender
  def recommend(evt=None):
    movie_name = Element('fav-movie').value
    top_n = Element('top-n').value
    try:
      top_n = int(top_n)
    except:
      top_n = 10
    recommender = KnnRecommender()
    recommender.set_model_params(20, 'brute', 'cosine', -1)
    if movie_name:
      recommendation = recommender.make_recommendations(movie_name, top_n, recommendation_only=True)
      print(recommendation)
    else:
      print("Sorry we need a movie name.")
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
```

Save and refresh, try with "Iron man" again and now you see instead of a listed output, we got a list back from the `make_recommendations`

Next, we want to present the Python list that we got as a `List` element in `html`. Here's why we have these tags in our `template.html` when we start:

```
<div class="text-center w-full mb-8"><ol id="output"></ol></div>
<template id="movie-template"><li class="movie"></li></template>
```

So what we will do is to generate a clone of the `<li>` element for each of the recommended movies, customise the text, then we will add them to the `output` `<ol>` element. Before we do that, we need to select those elements so we can use them in the code:

```
movie_template = Element("movie-template").select(".movie", from_content=True)
output_list = Element("output")
```

You can add them to the top after the `import` statment. Then we will replace the `print(recommendation)` with:

```
for i, recommend in enumerate(recommendation):
  movie_html = movie_template.clone(f"movie_{i}")
  movie_html.element.innerText = recommend
  output_list.element.appendChild(movie_html.element)
```

Then, we replace the `print` statement when we have no `movie_name`:

```
output_list.element.innerText = "Sorry we need a movie name."
```

Last, since we only want the most recent result to be shown, we will clear the last result when we press the button. Add this to the top of the `recommend` function:

```
output_list.element.innerHTML = ""
```

To double check, now we have:

```
<py-script>
  from knn_recommender import KnnRecommender
  movie_template = Element("movie-template").select(".movie", from_content=True)
  output_list = Element("output")
  def HTML(text):
    return text.replace("[[", "<").replace("]]", ">")
  def recommend(evt=None):
    output_list.element.innerHTML = ""
    movie_name = Element('fav-movie').value
    top_n = Element('top-n').value
    try:
      top_n = int(top_n)
    except:
      top_n = 10
    recommender = KnnRecommender()
    recommender.set_model_params(20, 'brute', 'cosine', -1)
    if movie_name:
      recommendation = recommender.make_recommendations(movie_name, top_n, recommendation_only=True)
      for i, recommend in enumerate(recommendation):
        movie_html = movie_template.clone(f"movie_{i}")
        movie_html.element.innerText = recommend
        output_list.element.appendChild(movie_html.element)
    else:
      output_list.element.innerText = "Sorry we need a movie name."
  run_btn = Element('run-btn')
  run_btn.element.onclick = recommend
</py-script>
```

Now save and refresh and have a play. The only thing that get in the way is the warning that got printed on the py-terminal. To switch it off, we just need to add `terminal = false` to the `<py-config>`

```
<py-config>
  terminal = false
  packages = ["pandas", "scikit-learn", "fuzzywuzzy"]
  [[fetch]]
  files = ["hashmap.p", "movie_user_mat_sparse.p", "knn_recommender.py"]
</py-config>
```

Now for the final time, save and refresh. This time, we have a reasonable recommender app.

---

This concludes Chapter 3 and the end this workshop. There is so much more you can do with pyscript. Check out their official documentation and example at https://pyscript.net/.
