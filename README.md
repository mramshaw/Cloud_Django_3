# Cloud_Django_3

[![Known Vulnerabilities](https://snyk.io/test/github/mramshaw/Cloud_Django_3/badge.svg?style=plastic&targetFile=requirements.txt)](https://snyk.io/test/github/mramshaw/Cloud_Django_3?style=plastic&targetFile=requirements.txt)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

This project follows on from my [Writing_Django_3](http://github.com/mramshaw/Writing_Django_3) project, which is a simple Hello World in Django 3.

[As might be expected, there were breaking changes between Django 2 and Django 3. So rather than upgrade my
 [Cloud_Django_2](http://github.com/mramshaw/Cloud_Django_2) repo, it seemed to be a better idea to create
 the __polls__ app in Django 3 and proceed from there, this time with Django 3.]

It will use [gunicorn](http://gunicorn.org/) which is a web server for [Django](http://docs.djangoproject.com/en/3.0/howto/deployment/wsgi/gunicorn/).
Specifically, it is a [WSGI](http://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) server.

One of the new features that Django 3 supports is [ASGI](http://asgi.readthedocs.io/en/latest/), which will
be tested here with [uvicorn](http://www.uvicorn.org). ASGI is *supposed* to be faster than WSGI, although
this remains to be tested. As is generally usual with these types of claims, it probably depends upon the
specific workload.

[For an explanation of why ASGI should be faster than WSGI,
 [this article](http://towardsdatascience.com/a-better-way-for-asynchronous-programming-asyncio-over-multi-threading-3457d82b3295)
 is worth a read. The benchmarking code will be transparent to anyone with recent experience with `node.js` (i.e. async/await experience).
 While I would personally take issue with the way the benchmarks are run, the results appear conclusive.]

We will run `uvicorn` behind `gunicorn`:

> For production deployments we recommend using gunicorn with the uvicorn worker class.

From: http://www.uvicorn.org/#running-with-gunicorn

The plan of attack is as follows:

* [Install and test 'gunicorn'](#gunicorn)
* [Configure 'gunicorn'](#configure-gunicorn)
* [Install and test 'uvicorn'](#uvicorn)
* [Run 'uvicorn' behind 'gunicorn'](#uvicorn-behind-gunicorn)

## gunicorn

To install locally:

    $ pip3 install --user gunicorn

Or simply use the `requirements.txt` file:

    $ pip3 install --user -r requirements.txt

Verify the version:

```bash
$ gunicorn --version
gunicorn (version 20.0.4)
$
```

Lets see if it runs (this needs to be in the same folder as `manage.py`):

```bash
$ cd polls
$ gunicorn polls.wsgi
[2020-02-19 16:32:10 -0500] [16182] [INFO] Starting gunicorn 20.0.4
[2020-02-19 16:32:10 -0500] [16182] [INFO] Listening at: http://127.0.0.1:8000 (16182)
[2020-02-19 16:32:10 -0500] [16182] [INFO] Using worker: sync
[2020-02-19 16:32:10 -0500] [16185] [INFO] Booting worker with pid: 16185
^C[2020-02-19 16:32:12 -0500] [16182] [INFO] Handling signal: int
[2020-02-19 21:32:12 +0000] [16185] [INFO] Worker exiting (pid: 16185)
[2020-02-19 16:32:12 -0500] [16182] [INFO] Shutting down: Master
$
```

## Configure gunicorn

Now we need to open up `gunicorn` with a `gunicorn.conf.py` file (for some reason,
this config file needs to be tagged as a Python file). By default, `gunicorn` runs
locally and will only accept local connections. We will configure it to run in
__promiscuous mode__ (which is a terrible practice, we should really run it behind
a front-end [__nginx__ is recommended], but we can fix this later).

We will create this file as follows:

```python3
import multiprocessing

bind = "0.0.0.0:8000"
workers = multiprocessing.cpu_count() * 2 - 1
```

Once more, with our config file (the `-c gunicorn.conf.py` part is not actually needed):

```bash
$ gunicorn -c gunicorn.conf.py polls.wsgi
[2020-02-19 16:35:58 -0500] [16239] [INFO] Starting gunicorn 20.0.4
[2020-02-19 16:35:58 -0500] [16239] [INFO] Listening at: http://0.0.0.0:8000 (16239)
[2020-02-19 16:35:58 -0500] [16239] [INFO] Using worker: sync
[2020-02-19 16:35:58 -0500] [16242] [INFO] Booting worker with pid: 16242
[2020-02-19 16:35:58 -0500] [16243] [INFO] Booting worker with pid: 16243
[2020-02-19 16:35:58 -0500] [16245] [INFO] Booting worker with pid: 16245
[2020-02-19 16:35:58 -0500] [16248] [INFO] Booting worker with pid: 16248
[2020-02-19 16:35:58 -0500] [16249] [INFO] Booting worker with pid: 16249
[2020-02-19 16:35:58 -0500] [16250] [INFO] Booting worker with pid: 16250
[2020-02-19 16:35:58 -0500] [16254] [INFO] Booting worker with pid: 16254
^C[2020-02-19 16:36:06 -0500] [16239] [INFO] Handling signal: int
[2020-02-19 21:36:06 +0000] [16245] [INFO] Worker exiting (pid: 16245)
[2020-02-19 21:36:06 +0000] [16242] [INFO] Worker exiting (pid: 16242)
[2020-02-19 21:36:06 +0000] [16243] [INFO] Worker exiting (pid: 16243)
[2020-02-19 21:36:06 +0000] [16254] [INFO] Worker exiting (pid: 16254)
[2020-02-19 21:36:06 +0000] [16249] [INFO] Worker exiting (pid: 16249)
[2020-02-19 21:36:06 +0000] [16248] [INFO] Worker exiting (pid: 16248)
[2020-02-19 21:36:06 +0000] [16250] [INFO] Worker exiting (pid: 16250)
[2020-02-19 16:36:06 -0500] [16239] [INFO] Shutting down: Master
[2019-11-27 09:57:14 -0500] [9960] [INFO] Shutting down: Master
$
```

Okay, everything runs.

## uvicorn

To install locally:

    $ pip3 install --user uvicorn

Or simply use the `requirements.txt` file:

    $ pip3 install --user -r requirements.txt

Verify the version:

```bash
$ uvicorn --version
Running uvicorn 0.11.3 with CPython 3.6.9 on Linux
$
```

Lets see if it runs (this needs to be in the same folder as `manage.py`):

```bash
$ cd polls
$ uvicorn polls.asgi:application
INFO:     Started server process [7899]
INFO:     Waiting for application startup.
INFO:     ASGI 'lifespan' protocol appears unsupported.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
^CINFO:     Shutting down
INFO:     Finished server process [7899]
$
```

[Note that the `uvicorn` syntax differs slightly from the `gunicorn` syntax.]

Or, to run with the 'lifespan' protocol disabled:

```bash
$ uvicorn polls.asgi:application --lifespan off
INFO:     Started server process [9590]
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
^CINFO:     Shutting down
INFO:     Finished server process [9590]
$
```

## uvicorn behind gunicorn

The recommendation is to run `uvicorn` behind `gunicorn`.

Even if it is not supplied, `gunicorn` will read `gunicorn.conf.py` (so it needs to be re-named).

This should look as follows (using WSGI):

```bash
$ mv gunicorn.conf.py gunicorn.conf
$ gunicorn polls.wsgi -k uvicorn.workers.UvicornWorker
[2020-02-20 11:49:29 -0500] [9237] [INFO] Starting gunicorn 20.0.4
[2020-02-20 11:49:29 -0500] [9237] [INFO] Listening at: http://127.0.0.1:8000 (9237)
[2020-02-20 11:49:29 -0500] [9237] [INFO] Using worker: uvicorn.workers.UvicornWorker
[2020-02-20 11:49:29 -0500] [9240] [INFO] Booting worker with pid: 9240
[2020-02-20 16:49:29 +0000] [9240] [INFO] Started server process [9240]
[2020-02-20 16:49:29 +0000] [9240] [INFO] Waiting for application startup.
[2020-02-20 16:49:29 +0000] [9240] [INFO] ASGI 'lifespan' protocol appears unsupported.
[2020-02-20 16:49:29 +0000] [9240] [INFO] Application startup complete.
^C[2020-02-20 11:49:32 -0500] [9237] [INFO] Handling signal: int
[2020-02-20 16:49:32 +0000] [9240] [INFO] Shutting down
[2020-02-20 16:49:32 +0000] [9240] [INFO] Finished server process [9240]
[2020-02-20 16:49:32 +0000] [9240] [INFO] Worker exiting (pid: 9240)
[2020-02-20 11:49:32 -0500] [9237] [INFO] Shutting down: Master
$
```

Okay, everything runs.

[The `ASGI 'lifespan' protocol appears unsupported.` line is ___informational___, so can be ignored.]

And again, this time with ASGI:

```bash
$ gunicorn polls.asgi -k uvicorn.workers.UvicornWorker
[2020-02-20 12:52:37 -0500] [9524] [INFO] Starting gunicorn 20.0.4
[2020-02-20 12:52:37 -0500] [9524] [INFO] Listening at: http://127.0.0.1:8000 (9524)
[2020-02-20 12:52:37 -0500] [9524] [INFO] Using worker: uvicorn.workers.UvicornWorker
[2020-02-20 12:52:37 -0500] [9527] [INFO] Booting worker with pid: 9527
[2020-02-20 17:52:37 +0000] [9527] [INFO] Started server process [9527]
[2020-02-20 17:52:37 +0000] [9527] [INFO] Waiting for application startup.
[2020-02-20 17:52:37 +0000] [9527] [INFO] ASGI 'lifespan' protocol appears unsupported.
[2020-02-20 17:52:37 +0000] [9527] [INFO] Application startup complete.
^C[2020-02-20 12:52:40 -0500] [9524] [INFO] Handling signal: int
[2020-02-20 17:52:40 +0000] [9527] [INFO] Shutting down
[2020-02-20 17:52:41 +0000] [9527] [INFO] Finished server process [9527]
[2020-02-20 17:52:41 +0000] [9527] [INFO] Worker exiting (pid: 9527)
[2020-02-20 12:52:41 -0500] [9524] [INFO] Shutting down: Master
$
```

Also fine.

[As `gunicorn` is a WSGI application, I believe specifying `.wsgi` is __probably__ correct.]

The `gunicorn.conf.py` config file may be reinstated as follows:

```bash
$ mv gunicorn.conf gunicorn.conf.py
$
```

## Reference

Some useful references follow.

#### Django with Gunicorn

For a production implementation of Django (WSGI or ASGI), `gunicorn` is recommended.

How to use Django with Gunicorn: http://docs.djangoproject.com/en/3.0/howto/deployment/wsgi/gunicorn/

#### Django with Uvicorn

For a production implementation of ASGI, `uvicorn` is recommended.

How to use Django with Uvicorn: http://docs.djangoproject.com/en/3.0/howto/deployment/asgi/uvicorn/

#### Django Deployment Checklist

Django has a nice utility for checking if the app is ready for deployment: http://docs.djangoproject.com/en/3.0/howto/deployment/checklist/

#### Uvicorn

Uvicorn (even behind Gunicorn) should be faster than Gunicorn by itself:

> ASGI should help enable an ecosystem of Python web frameworks that are highly competitive
> against Node and Go in terms of achieving high throughput in IO-bound contexts. It also
> provides support for HTTP/2 and WebSockets, which cannot be handled by WSGI.

[The implication here is that Python web frameworks are NOT competitive against Node or Go.]

And:

> Uvicorn currently supports HTTP/1.1 and WebSockets. Support for HTTP/2 is planned.

From: http://www.uvicorn.org

Uvicorn settings: http://www.uvicorn.org/settings/

One frustration with running `uvicorn` behind `gunicorn` is that it is not possible
to specify `--lifespan off` (which would prevent the `ASGI 'lifespan' protocol appears
unsupported.` INFO message).

#### Lifespan Protocol

This is mainly to allow for life-cycle events for the worker (or workers):

> The Lifespan ASGI sub-specification outlines how to communicate lifespan events
> such as startup and shutdown within ASGI.

And:

> The lifespan messages allow for an application to initialise and shutdown
> in the context of a running event loop. An example of this would be creating
> a connection pool and subsequently closing the connection pool to release
> the connections.

From:

    http://asgi.readthedocs.io/en/latest/specs/lifespan.html

[Within the context of ASGI, the Lifespan Protocol appears to be __optional__.]

## Versions

* Django __3.0.3__
* gunicorn __20.0.4__
* Python __3.6.9__
* uvicorn __0.11.3__

## To Do

- [ ] Investigate whether specifying WSGI or ASGI to `gunicorn` makes a difference
- [ ] Benchmark Django 3 ASGI performance against Django 3 WSGI performance
- [x] Add a badge for `Black` formatting style
