# Cloud_Django_3

[![Known Vulnerabilities](https://snyk.io/test/github/mramshaw/Cloud_Django_3/badge.svg?style=plastic&targetFile=requirements.txt)](https://snyk.io/test/github/mramshaw/Cloud_Django_3?style=plastic&targetFile=requirements.txt)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

This project follows on from my [Writing_Django_3](https://github.com/mramshaw/Writing_Django_3) project, which is a simple Hello World in Django 3.

[As might be expected, there were breaking changes between Django 2 and Django 3. So rather than upgrade my
 [Cloud_Django_2](https://github.com/mramshaw/Cloud_Django_2) repo, it seemed to be a better idea to create
 the __polls__ app in Django 3 and proceed from there, this time with Django 3.]

It will use [gunicorn](http://gunicorn.org/) which is a web server for [Django](https://docs.djangoproject.com/en/3.0/howto/deployment/wsgi/gunicorn/).
Specifically, it is a [WSGI](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) server.

The plan of attack is as follows:

* [Install and test 'gunicorn'](https://github.com/mramshaw/Cloud_Django#gunicorn)

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

## Reference

Some useful references follow.

#### Django with Gunicorn

For a production implementation of Django (WSGI or ASGI), `gunicorn` is recommended.

How to use Django with Gunicorn: http://docs.djangoproject.com/en/3.0/howto/deployment/wsgi/gunicorn/

#### Django with Uvicorn

For a production implementation of ASGI, `uvicorn` is recommended.

How to use Django with Uvicorn: http://docs.djangoproject.com/en/3.0/howto/deployment/asgi/uvicorn/

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

- [ ] Benchmark Django 3 ASGI performance against Django 3 WSGI performance
- [x] Add a badge for `Black` formatting style
