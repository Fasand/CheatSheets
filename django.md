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

# mysite/
#    manage.py
#    mysite/
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
