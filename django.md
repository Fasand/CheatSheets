# Django (2.1)

## Installation
Create virtual environment with Python3 and activate
```
virtualenv -p `which python3` ~/.virtualenvs/djangodev
source ~/.virtualenvs/djangodev/bin/activate
```

Install Django: 
```
pip install Django
```

## Setup
Create a new project
```
django-admin startproject myproject

# myproject/
#    manage.py
#    myproject/
#        __init__.py
#        settings.py
#        urls.py
#        wsgi.py
```

Run development server
```
python manage.py runserver [port]
```

Create a new app
```
python manage.py startapp myapp

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
```
python manage.py migrate
```

## Settings
INSTALLED_APPS: any apps used by the project, including the ones you create
- e.g. `django.contrib.{admin|auth|sessions|...}`
- to include your app, reference its configuration class
    - e.g. `myapp.apps.MyappConfig`

DATABASES.default:
- ENGINE: `django.db.backends.{sqlite3|mysql|postgresql|...}`
- NAME: absolute path for sqlite3, database name for others

## Models
Basic model
```
from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=50)
    pub_date = models.DateTimeField('date published')
```

Prepare migrations for changes in myapp, don't change DB yet
```
python manage.py makemigrations myapp

# Migrations for 'myapp':
#   myapp/migrations/0001_initial.py
#     - Create model MyModel
```

View sql of a migration
```
python manage.py sqlmigrate myapp 0001
```

Migrate a migration
```
python manage.py migrate

# Operations to perform:
#   Apply all migrations: admin, auth, contenttypes, myapp, sessions
# Running migrations:
#   Applying myapp.0001_initial... OK
```

Play around with your models
```python
$ python manage.py shell
>>> from myapp.models import MyModel
>>> from django.utils import timezone
>>> m = MyModel(name="Fancy name", pub_date=timezone.now())
>>> m.save()
```

### Fields (TODO)

## Views
Basic view
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello world.")
```

## Urls
Basic template
```
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

