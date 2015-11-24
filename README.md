# django-modelclone

Allows users to duplicate a model in admin.

## Installation

    $ pip install django-modelclone

then:

 1. Add `'modelclone'` to `INSTALLED_APPS`
 2. In your `admin.py` files extend from `modelclone.ClonableModelAdmin` instead of
    Django's `ModelAdmin`

The models that have admin configuration extending `modelclone.ClonableModelAdmin` will
have a new link on the Change page to duplicate that object

![Screenshot Duplicate link](images/duplicate-link.png)

This links redirects to a page similar to an Add page but with all the fields already
filled with the values from the original object.

Note that you still need to save to get a new object. And make sure to edit fields
that must be unique otherwise you will get a validation error.

## Advanced usage

Extends `modelclone.ClonableModelAdminMix` class.

```python
from django.contrib import admin
from modelclone import ClonableModelAdminMix
from .models import MyModel, MyOtherModel

class MyCustomAdminMix(object):
    pass

class MyBaseAdmin(MyCustomAdminMix, ClonableModelAdminMix, admin.ModelAdmin):
    def post_clone(self, request, original_obj, new_obj):
        '''callback called of post clone'''
        print("Clone of %r (#%s) is %r (#%s)" % (original_obj, original_obj.pk, new_obj, new_obj.pk))

class MyAdmin(MyBaseAdmin):
    list_search = ['name']

admin.site.register(MyModel, MyAdmin)

class NotClonableAdmin(MyBaseAdmin):
    clonable = False

admin.site.register(MyOtherModel, NotClonableAdmin)
```

In my template `templates/admin/myapp/mymodel/change_form.html`

```html
{% extends "admin/change_form.html" %}

{% block object-tools-items %}
    {% if include_clone_link %}
        <li><a href="clone/">{{ clone_verbose_name }}</a></li>
    {% endif %}
    {{ block.super }}
{% endblock %} 
```

## But Django already has a 'save as'

Yes, I know. Django Admin has a [`save_as`](https://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.save_as)
feature that adds a new button to your Change page to save a new instance of that
object.

I don't like the way this feature works because you will save an identical copy of the
original object (if you don't get validation errors) as soon as you click that link, and
if you forget to make the small changes that you wanted in the new object you will end up
with a duplicate of the existing object.

On the other hand, django-modelclone offers an intermediate view, that basically pre-fills
the form for you. So you can modify and then save a new instance. Or just go away without
side effects.

## Requirements

Tested with Python 2.6 and 2.7. Django 1.4 up to 1.7. See `tox.ini`.

## Hacking

Fork the [repository on github](http://github.com/realgeeks/django-modelclone), make your
changes (don't forget the tests) and send a pull request.

To run the tests, install and run [Tox](http://tox.readthedocs.org/):

    $ pip install tox
    $ tox

You can also run the sample project to test manually. In this case you'll need to
install Django, or just use one of the virtualenvs tox creates, for example:

    $ source .tox/py27-django15/bin/activate

then start the server

    (py27-django15) $ ./manager serve

The app is available on [http://localhost:8000/admin/](http://localhost:8000/admin/),
username and password "admin".
