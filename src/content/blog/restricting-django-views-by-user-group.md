---
title: "Restricting Django Views by User Group with a Decorator Factory"
description: "How to build a reusable Django decorator that restricts view access to one or more user groups."
pubDate: "Jan 15 2020"
heroImage: "/blog-placeholder-2.jpg"
---

In Django, you sometimes need to wrap a view function in a decorator that checks whether the current user belongs to certain groups.

There are a few things we want from this decorator:

- It should be able to check several groups at once.
- The list of groups should be configurable through the decorator's arguments.
- It should be able to carry some custom behavior. For example, only admins should even know that a given page exists — every other user gets a 404.

Let's create a module for our decorators, call it `decorators.py`, and start with a simple one. It checks whether the user is in the `admin` group; if that check fails, we return a 404.

```python
from django.http import Http404

def check_user_able_to_see_page(function):
    def wrapper(request, *args, **kwargs):
        if request.user.groups.filter(name="admin").exists():
            return function(request, *args, **kwargs)
        raise Http404

    return wrapper
```

We can use it in `views.py` like this:

```python
@check_user_able_to_see_page
def hidden_page(request):
    ....
```

## The problem

There's a catch. If we want to grant access to a different group, we have to modify the decorator or write a new one. And if we have several pages, each needing its own group check, we'd end up creating a separate decorator for every one of them.

A decorator receives a single argument: the function it decorates. There's no room to pass anything else. The way around this is a *decorator factory* — a function that returns a decorator.

```python
from django.http import Http404

def check_user_able_to_see_page(*groups):

    def decorator(function):
        def wrapper(request, *args, **kwargs):
            if request.user.groups.filter(name__in=groups).exists():
                return function(request, *args, **kwargs)
            raise Http404

        return wrapper

    return decorator
```

Now you can use it in any of these ways:

```python
@check_user_able_to_see_page("admin")
def hidden_page(request):
    ....
```

or

```python
@check_user_able_to_see_page("manager")
def hidden_page(request):
    ....
```

or even with several groups at once:

```python
@check_user_able_to_see_page("admin", "manager")
def hidden_page(request):
    ....
```

## Making it nicer with an enum

This already works well, but let's polish it a little. First, create a module that enumerates the groups:

```python
# constants.py
from enum import Enum

class Group(Enum):
    admin = "Administrator"
    manager = "Manager"
```

Then rewrite the decorator to accept those enum members:

```python
from django.http import Http404
from .constants import Group

def check_user_able_to_see_page(*groups: Group):

    def decorator(function):
        def wrapper(request, *args, **kwargs):
            if request.user.groups.filter(
                name__in=[group.name for group in groups]
            ).exists():
                return function(request, *args, **kwargs)
            raise Http404

        return wrapper

    return decorator
```

Now the usage reads cleanly, with no magic strings to mistype:

```python
from .constants import Group

@check_user_able_to_see_page(Group.admin)
def hidden_page(request):
    ...
```
