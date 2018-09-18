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

