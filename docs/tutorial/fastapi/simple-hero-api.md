# Simple Hero API with FastAPI

Let's start by building a simple hero web API with **FastAPI**. ✨

## Install **FastAPI**

The first step is to install FastAPI.

FastAPI is the framework to create the **web API**.

Make sure you create a [virtual environment](../../virtual-environments.md){.internal-link target=_blank}, activate it, and then install them, for example with:

<div class="termy">

```console
$ pip install fastapi "uvicorn[standard]"

---> 100%
```

</div>

## **SQLModel** Code - Models, Engine

Now let's start with the SQLModel code.

We will start with the **simplest version**, with just heroes (no teams yet).

This is almost the same code we have seen up to now in previous examples:

{* ./docs_src/tutorial/fastapi/simple_hero_api/tutorial001_py310.py ln[2,5:20] hl[19:20] *}

There's only one change here from the code we have used before, the `check_same_thread` in the `connect_args`.

That is a configuration that SQLAlchemy passes to the low-level library in charge of communicating with the database.

`check_same_thread` is by default set to `True`, to prevent misuses in some simple cases.

But here we will make sure we don't share the same **session** in more than one request, and that's the actual **safest way** to prevent any of the problems that configuration is there for.

And we also need to disable it because in **FastAPI** each request could be handled by multiple interacting threads.

/// info

That's enough information for now, you can read more about it in the <a href="https://fastapi.tiangolo.com/async/" class="external-link" target="_blank">FastAPI docs for `async` and `await`</a>.

The main point is, by ensuring you **don't share** the same **session** with more than one request, the code is already safe.

///

## **FastAPI** App

The next step is to create the **FastAPI** app.

We will import the `FastAPI` class from `fastapi`.

And then create an `app` object that is an instance of that `FastAPI` class:

{* ./docs_src/tutorial/fastapi/simple_hero_api/tutorial001_py310.py ln[1:2,23] hl[1,23] *}

## Create Database and Tables on `startup`

We want to make sure that once the app starts running, the function `create_tables` is called. To create the database and tables.

This should be called only once at startup, not before every request, so we put it in the function to handle the `"startup"` event:

{* ./docs_src/tutorial/fastapi/simple_hero_api/tutorial001_py310.py ln[24:30] hl[27:30] *}

## Create Heroes *Path Operation*

/// info

If you need a refresher on what a **Path Operation** is (an endpoint with a specific HTTP Operation) and how to work with it in FastAPI, check out the <a href="https://fastapi.tiangolo.com/tutorial/first-steps/" class="external-link" target="_blank">FastAPI First Steps docs</a>.

///

Let's create the **path operation** code to create a new hero.

It will be called when a user sends a request with a `POST` **operation** to the `/heroes/` **path**:

{* ./docs_src/tutorial/fastapi/simple_hero_api/tutorial001_py310.py ln[23:37] hl[31:32] *}

/// info

If you need a refresher on some of those concepts, checkout the FastAPI documentation:

* <a href="https://fastapi.tiangolo.com/tutorial/first-steps/" class="external-link" target="_blank">First Steps</a>
* <a href="https://fastapi.tiangolo.com/tutorial/path-params/" class="external-link" target="_blank">Path Parameters - Data Validation and Data Conversion</a>
* <a href="https://fastapi.tiangolo.com/tutorial/body/" class="external-link" target="_blank">Request Body</a>

///

## The **SQLModel** Advantage

Here's where having our **SQLModel** class models be both **SQLAlchemy** models and **Pydantic** models at the same time shine. ✨

Here we use the **same** class model to define the **request body** that will be received by our API.

Because **FastAPI** is based on Pydantic, it will use the same model (the Pydantic part) to do automatic data validation and <abbr title="also called serialization, marshalling">conversion</abbr> from the JSON request to an object that is an actual instance of the `Hero` class.

And then, because this same **SQLModel** object is not only a **Pydantic** model instance but also a **SQLAlchemy** model instance, we can use it directly in a **session** to create the row in the database.

So we can use intuitive standard Python **type annotations**, and we don't have to duplicate a lot of the code for the database models and the API data models. 🎉

/// tip

We will improve this further later, but for now, it already shows the power of having **SQLModel** classes be both **SQLAlchemy** models and **Pydantic** models at the same time.

///

## Read Heroes *Path Operation*

Now let's add another **path operation** to read all the heroes:

{* ./docs_src/tutorial/fastapi/simple_hero_api/tutorial001_py310.py ln[23:44] hl[40:44] *}

This is pretty straightforward.

When a client sends a request to the **path** `/heroes/` with a `GET` HTTP **operation**, we run this function that gets the heroes from the database and returns them.

## One Session per Request

Remember that we should use a SQLModel **session** per each group of operations and if we need other unrelated operations we should use a different session?

Here it is much more obvious.

We should normally have **one session per request** in most of the cases.

In some isolated cases, we would want to have new sessions inside, so, **more than one session** per request.

But we would **never want to *share* the same session** among different requests.

In this simple example, we just create the new sessions manually in the **path operation functions**.

In future examples later we will use a <a href="https://fastapi.tiangolo.com/tutorial/dependencies/" class="external-link" target="_blank">FastAPI Dependency</a> to get the **session**, being able to share it with other dependencies and being able to replace it during testing. 🤓

## Run the **FastAPI** Server in Development Mode

Now we are ready to run the FastAPI application.

Put all that code in a file called `main.py`.

Then run it with the `fastapi` <abbr title="Command Line Interface">CLI</abbr>, in development mode:

<div class="termy">

```console
$ fastapi dev main.py

<span style="color: green;">INFO</span>:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

</div>

/// info

The `fastapi` command uses <a href="https://www.uvicorn.org/" class="external-link" target="_blank">Uvicorn</a> underneath.

///

When you use `fastapi dev` it starts Uvicorn with the option to reload automatically every time you make a change to the code, this way you will be able to develop faster. 🤓

## Run the **FastAPI** Server in Production Mode

The development mode should not be used in production, as it includes automatic reload by default it consumes much more resources than necessary, and it would be more error prone, etc.

For production, use `fastapi run` instead of `fastapi dev`:

<div class="termy">

```console
$ fastapi run main.py

<span style="color: green;">INFO</span>:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

</div>

## Check the API docs UI

Now you can go to that URL in your browser `http://127.0.0.1:8000`. We didn't create a *path operation* for the root path `/`, so that URL alone will only show a "Not Found" error... that "Not Found" error is produced by your FastAPI application.

But you can go to the **automatically generated interactive API documentation** at the path `/docs`: <a href="http://127.0.0.1:8000/docs" class="external-link" target="_blank">http://127.0.0.1:8000/docs</a>. ✨

You will see that this **automatic API docs <abbr title="user interface">UI</abbr>** has the *paths* that we defined above with their *operations*, and that it already knows the shape of the data that the **path operations** will receive:

<img class="shadow" alt="Interactive API docs UI" src="/img/tutorial/fastapi/simple-hero-api/image01.png">

## Play with the API

You can actually click the button <kbd>Try it out</kbd> and send some requests to create some heroes with the **Create Hero** *path operation*.

And then you can get them back with the **Read Heroes** *path operation*:

<img class="shadow" alt="Interactive API docs UI reading heroes" src="/img/tutorial/fastapi/simple-hero-api/image02.png">

## Check the Database

Now you can terminate that server program by going back to the terminal and pressing <kbd>Ctrl+C</kbd>.

And then, you can open **DB Browser for SQLite** and check the database, to explore the data and confirm that it indeed saved the heroes. 🎉

<img class="shadow" alt="DB Browser for SQLite showing the heroes" src="/img/tutorial/fastapi/simple-hero-api/db-browser-01.png">

## Recap

Good job! This is already a FastAPI **web API** application to interact with the heroes database. 🎉

There are several things we can improve and extend. For example, we want the database to decide the ID of each new hero, we don't want to allow a user to send it.

We will make all those improvements in the next chapters. 🚀
