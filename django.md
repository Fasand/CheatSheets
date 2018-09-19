![Django](https://www.djangoproject.com/s/img/logos/django-logo-positive.svg)

Heavily inspired by https://docs.djangoproject.com/en/2.1/intro/

# Table of contents
- [Installation](#installation)
- [Setup](#setup)
- [Settings](#settings)
- [Models](#models)
- [Admin](#admin)
- [Views](#views)
- [Urls](#urls)

## Installation
Create virtual environment with Python3 and activate
```bash
$ virtualenv -p `which python3` ~/.virtualenvs/djangodev
$ source ~/.virtualenvs/djangodev/bin/activate
```

Install Django: 
```bash
$ pip install Django
```

## Setup
Create a new project
```bash
$ django-admin startproject myproject

# myproject/
#    manage.py
#    myproject/
#        __init__.py
#        settings.py
#        urls.py
#        wsgi.py
```

Run development server
```bash
$ python manage.py runserver [port]
```

Create a new app
```bash
$ python manage.py startapp myapp

# myapp/
#    __init__.py
#    admin.py
#    apps.py
#    migrations/
#        __init__.py
#    models.py
#    tests.py
#    views.py
```

Migrate (create or update) databases, e.g. initial db creation
```bash
$ python manage.py migrate
```

Create a superuser for administration
```bash
$ python manage.py createsuperuser
```

## Settings
INSTALLED_APPS: any apps used by the project, including the ones you create
- e.g. `django.contrib.{admin|auth|sessions|...}`
- to include your app, reference its configuration class
    - e.g. `myapp.apps.MyappConfig`

DATABASES.default:
- ENGINE: `django.db.backends.{sqlite3|mysql|postgresql|...}`
- NAME: absolute path for sqlite3, database name for others

TIME_ZONE = e.g. UTC, GMT, Europe/London, ...
- see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

## Models
Basic model
```python
from django.db import models
from django.utils import timezone
import datetime

class MyModel(models.Model):
    name = models.CharField(max_length=50)
    pub_date = models.DateTimeField('date published')

    # String representation for administration
    def __str__(self):
        return self.name

    # Define functions as usual
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

Prepare migrations for changes in myapp, don't change DB yet
```bash
$ python manage.py makemigrations myapp

# Migrations for 'myapp':
#   myapp/migrations/0001_initial.py
#     - Create model MyModel
```

View sql of a migration
```bash
$ python manage.py sqlmigrate myapp 0001
```

Migrate a migration
```bash
$ python manage.py migrate

# Operations to perform:
#   Apply all migrations: admin, auth, contenttypes, myapp, sessions
# Running migrations:
#   Applying myapp.0001_initial... OK
```

Play around with your models
```bash
$ python manage.py shell
```
```python
>>> from myapp.models import MyModel
>>> from django.utils import timezone
>>> m = MyModel(name="Fancy name", pub_date=timezone.now())
>>> m.save()
>>> m.was_published_recently()
True
```

### Fields (TODO)

### Database API (TODO)
e.g. MyModel.objects.all()

## Admin
Register a model in administration
```python
# myapp/admin.py
from django.contrib import admin
from .models import MyModel

admin.site.register(MyModel)
```

## Views
Basic view
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello world.")
```

## Urls
Basic template
```python
# myproject/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('myapp/', include('myapp.urls')),
    path('admin/', admin.site.urls),
]

# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    # resolves to /myapp/
]
```
- Always use **include()**, only exception is **admin.site.urls**


path(route, view, kwargs, name)
- **route**: e.g. `"index/"`, `"article/<int:articleid>"`, `"bio/<username>"`
- **view**: either a specific view or include()
- **kwargs**: additional arguments for the view
- **name**: identifier that is used by **reverse()**
- *also exists re_path(): using regex*

