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

ROOT_URLCONF: pointer to main urlconf, first point of contact for dealing with url requests

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
- MyModel.objects.get(pk=...) (use primary key)

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

# args come from path route, e.g. "/detail/<article_id>"
def detail(request, article_id):
    return HttpResponse("Article details for: {}".format(article_id))
```

Using a template
```python
# The hard way
from django.http import HttpResponse
from django.template import loader

def index(request):
    template = loader.get_template('myapp/index.html')
    context = { 'message': 'Hello world' }
    return HttpResponse(template.render(context, request))

# The easy way
from django.shortcuts import render

def index(request):
    context = { 'message': 'Hello world' }
    return render(request, 'myapp/index.html', context)
```

Raising a 404
```python
# The hard way
from django.http import Http404
from django.shortcuts import render
from .models import MyModel

def detail(request, article_id):
    try:
        article = MyModel.objects.get(pk=question_id)
    except MyModel.DoesNotExist:
        raise Http404("Article does not exist")
    return render(request, 'myapp/detail.html', {'article': article})

# The easy way
from django.shortcuts import render, get_object_or_404

def detail(request, article_id):
    article = get_object_or_404(MyModel, pk=question_id)
    return render(request, 'myapp/detail.html', {'article': article})
```
- `get_list_or_404()` works the same but for database `filter()` instead of `get()`

Access POST data
```python
# request.POST is a dictionary-like object
request.POST['choice']
```
- `request.GET` also exists

Redirect to another URL
```python
from django.http import HttpResponseRedirect
from django.urls import reverse

def index(request):
    return HttpResponseRedirect(reverse('myapp:detail', args=(1,)))
```
- always return a redirect when dealing with POST forms
- `reverse()` takes the name of a view and a tuple of arguments

### Templates
- Store in `myapp/templates/myapp/template.html`
    - Adjustable in settings/TEMPLATES
    - Requires APP_DIRS to be True, which it is by default

Sample template
```django
{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'myapp:vote' article.id %}" method="post">
{% csrf_token %}
{% for choice in article.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```
- `{{ forloop.counter }}` prints the current for loop iteration
- `{% csrf_token %}` should always be used in a POST form

Dynamic URLs
```django
Use the urlconf of the app containing the template
{% url 'index' %}
{% url 'detail' article.id %}

Use a namespace defined in the app's "myapp/urls.py"
{% url 'myapp:detail' article.id %}
```

### Generic views
TemplateView
```python
from django.views import generic

class HomePageView(generic.base.TemplateView):
    template_name = "myapp/home.html"

    def get_context_data(self, **kwargs):
        # get original context before modifying it
        # contains parameters captured in the URL
        context = super().get_context_data(**kwargs)
        # add your own stuff and render template_name
        context['fascinating_fact'] = "Dead people can get goose bumps."
        return context
```

ListView
```python
from django.views import generic

class IndexView(generic.ListView):
    # not necessary if overriding get_queryset()
    model = MyModel
    # default is <app name>/<model name>_list.html
    template_name = "myapp/index.html"
    # rename the template variable, default is mymodel_list
    context_object_name = 'latest_articles_list'

    # return a filtered queryset instead of the whole thing
    def get_queryset(self):
        return MyModel.objects.order_by('-pub_date')[:5]

# myapp/urls.py 
path('', views.IndexView.as_view(), name='index')
```

DetailView
```python
from django.views import generic

class DetailView(generic.DetailView):
    # necessary this time
    model = Question
    # default is <app name>/<model name>_detail.html
    template_name = 'myapp/detail.html'

    # add your own data to template context
    def get_context_data(self, **kwargs):
        # Call the base implementation first to get a context
        context = super().get_context_data(**kwargs)
        # Add in a QuerySet of all the books
        context['message'] = "Whoa, this is a message."
        return context

# myapp/urls.py
# 'pk' will be used as the filter for the database
path('detail/<int:pk>', views.DetailView.as_view(), name='detail')
```

also Redirect, Form, Create, Update, Delete, ... views
- [Django 2.1 built-in generic views](https://docs.djangoproject.com/en/2.1/ref/class-based-views/)

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

# Declare the app's namespace for url reverse
app_name = 'myapp'

urlpatterns = [
    path('detail/<int:article_id>', views.detail, name='detail'),
]
```
- Always use **include()**, only exception is **admin.site.urls**


path(route, view, kwargs, name)
- **route**: e.g. `"index/"`, `"article/<int:article_id>"`, `"bio/<username>"`
- **view**: either a specific view or include()
- **kwargs**: additional arguments for the view
- **name**: identifier that is used by **reverse()**
- *also exists re_path(): using regex*

