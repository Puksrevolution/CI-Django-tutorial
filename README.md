```
RUN SERVER
-----------
python3 manage.py runserver
```

- pip3 install django

(This is just if you wanna see the index of the django packages
- cd /workspace/.pip-modules/lib/python3.7/site-packages/
  - ls -la
    - to see the index of all directories and files
- cd - )

- django-admin startproject django_todo .
- python3 manage.py runserver

- python3 manage.py startapp todo


- open todo folder
  - open views.py
```
from django.shortcuts import render, HttpResponse


# Create your views here.
def say_hello(request):
    """
    docstring
    """
    return HttpResponse("Hello!")

```


- open django_todo
  - open urls.py
```
from django.contrib import admin
from django.urls import path
from todo.views import say_hello


urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', say_hello, name='hello')
]
```

- python3 manage.py runserver

- inside of todo folder create templates foler
  - inside of templates folder create todo foler
  - inside todo folder create todo_list.html file
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Things I need to do:</h1>
</body>
</html>
```
- update views.py
```
from django.shortcuts import render


# Create your views here.
def get_todo_list(request):    
    return render(request, 'todo/todo_list.html')

```


- update urls.py
```
from django.contrib import admin
from django.urls import path
from todo.views import get_todo_list

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', get_todo_list, name='get_todo_list')
]

```


- open settings.py in django_todo folder
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'todo'
]
```
### Django's initial database migrations ###
- to set up our sequel Lite 3 database
- to created a superuser that we could use and manage the database
  - python3 manage.py makemigrations --dry-run
  - python3 manage.py showmigrations
  - python3 manage.py migrate --plan
  - python3 manage.py migrate
  - python3 manage.py createsuperuser
    - username: user
    - password: adminaccount

### built-in Django admin panel ###
- python3 manage.py runserver
  - open browser 8000
  - add /admin to:  https://8000-lavender-llama-4ksukh3w.ws-eu14.gitpod.io/
  - log in with created username and created password


- open todo/models.py
```
from django.db import models


# Create your models here.
class Item(models.Model):
    name = models.CharField(max_length=50, null=False, blank=False)
    done = models.BooleanField(null=False, blank=False, default=False)

    def __str__(self):
      return self.name

```
- Quit the server with CONTROL-C.
- python3 manage.py makemigrations --dry-run
  - --dry-run is just a check, if its realy do what you what 
- python3 manage.py makemigrations
  - check migrations folder if there is a new initial.py with the needed code for our database/table for webpage

- python3 manage.py showmigrations
- python3 manage.py migrate --plan
  - --plan is just a check, if its realy do what you what
- python3 manage.py migrate

- open todo/admin.py
```
from django.contrib import admin
from .models import Item


# Register your models here.
admin.site.register(Item)

```
- python3 manage.py runserver


- open views.py
```
from django.shortcuts import render
from .models import Item


# Create your views here.
def get_todo_list(request):
    items = Item.objects.all()
    context = {
        'items': items
    }
    return render(request, 'todo/todo_list.html', context)

```
- python3 manage.py runserver

- open todo_list.html
```
<table>
  {% for item in items %}
    <tr>
      <td> {{ item.name }} </td>
      <td> {{ item.done }} </td>
    </tr>
  {% endfor %}
</table>
```
- update to
```
<table>
    {% for item in items %}
        <tr>
            {% if item.done %}
            <td> <strike>{{ item.name }}</strike> </td>
            {% else %}
            <td> {{ item.name }} </td>
            {% endif %}
        </tr>
        {% empty %}
        <tr><td>You have nothing to do.</td></tr>
    {% endfor %}
</table>
```

- open views.py
```
def add_item(request):
    if request.method == 'POST':
        name = request.POST.get('item_name')
        done = 'done' in request.POST
        Item.objects.create(name=name, done=done)

        return redirect('get_todo_list')
    return render(request, 'todo/add_item.html')

```
- open url.py
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', get_todo_list, name='get_todo_list'),
    path('add', add_item, name='add')
]
```
- create new html add_item.html
```
<h1>Add a ToDo Item</h1>
    
    <form action="add" method="POST">
        {% csrf_token %}
        <div>
            <p>
                <label for="id_name">Name:</label>
                <input type="text" id="id_name" name="item_name">
            </p>
        </div>
        <div>
            <p>
                <label for="id_done">Done:</label>
                <input type="checkbox" id="id_done" name="done">
            </p>
        </div>
        <div>
            <p>
                <button type="submit">Add Item</button>
            </p>
        </div>
    </form>
```
- udate todo_list html
```
<h1>Things we need to do:</h1>
    
    <table>
    {% for item in items %}
        <tr>
            {% if item.done %}
            <td> <strike>{{ item.name }}</strike> </td>
            {% else %}
            <td> {{ item.name }} </td>
            {% endif %}
        </tr>
        {% empty %}
        <tr><td>You have nothing to do.</td></tr>
    {% endfor %}
    </table>

    <a href="/add">Add an Item</a>
```


- create forms.py in todo folder
```
from django import forms
from .models import Item


class ItemForm(forms.ModelForm):
    class Meta:
        model = Item
        fields = ['name', 'done']

```

- udate views.py
```
from django.shortcuts import render, redirect
from .models import Item
from .forms import ItemForm


# Create your views here.


def get_todo_list(request):
    items = Item.objects.all()
    context = {
        'items': items
    }
    return render(request, 'todo/todo_list.html', context)


def add_item(request):
    if request.method == 'POST':
        form = ItemForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('get_todo_list')
    form = ItemForm()
    context = {
        'form': form
    }
    return render(request, 'todo/add_item.html', context)

```

- update add_item.html
```
<form method="POST" action="add">
    {% csrf_token %}
    {{ form.as_p }}
        <p>
            <button type="submit">Add Item</button>
        </p>
    </div>
</form>
```

- update todo_list.html
```
<table>
    {% for item in items %}
        <tr>
            {% if item.done %}
            <td> <strike>{{ item.name }}</strike> </td>
            {% else %}
            <td> {{ item.name }} </td>
            {% endif %}
            <td>
                <a href="/edit/{{ item.id }}">
                    <button>EDIT</button>
                </a>
            </td>
        </tr>
        {% empty %}
        <tr><td>You have nothing to do.</td></tr>
    {% endfor %}
    </table>
```


- create edit_item.html
```
<body>
    <h1>Edit a ToDo Item</h1>
    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <div>
            <p>
                <button type="submit">Update Item</button>
            </p>

        </div>
    </form>
</body>
```
- update urls.py
```
from todo.views import get_todo_list, add_item, edit_item

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', get_todo_list, name='get_todo_list'),
    path('add', add_item, name='add'),
    path('edit/<item_id>', edit_item, name='edit')
]

```
- update views.py
```
from django.shortcuts import render, redirect, get_object_or_404


def edit_item(request, item_id):
    item = get_object_or_404(Item, id=item_id)
    if request.method == 'POST':
        form = ItemForm(request.POST, instance=item)
        if form.is_valid():
            form.save()
            return redirect('get_todo_list')
    form = ItemForm(instance=item)
    context = {
        'form': form
    }
    return render(request, 'todo/edit_item.html', context)
```

- toggle function
- update todo_list.html
```
<table>
    {% for item in items %}
        <tr>
            {% if item.done %}
            <td> <strike>{{ item.name }}</strike> </td>
            {% else %}
            <td> {{ item.name }} </td>
            {% endif %}
            <td>
                <a href="/edit/{{ item.id }}">
                    <button>EDIT</button>
                </a>
            </td>
            <td>
                <a href="/toggle/{{ item.id }}">
                    <button>TOGGLE</button>
                </a>
            </td>
        </tr>
        {% empty %}
        <tr><td>You have nothing to do.</td></tr>
    {% endfor %}
</table>
```
- update urls.py
```
from todo import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.get_todo_list, name='get_todo_list'),
    path('add', views.add_item, name='add'),
    path('edit/<item_id>', views.edit_item, name='edit'),
    path('toggle/<item_id>', views.toggle_item, name='toggle')
]
```
- update views.py
```
def toggle_item(request, item_id):
    item = get_object_or_404(Item, id=item_id)
    item.done = not item.done
    item.save()
    return redirect('get_todo_list')
```


- delete function
- update todo_list.html
```
<table>
    {% for item in items %}
        <tr>
        {% if item.done %}
            <td><strike>{{ item.name }}</strike></td>
        {% else %}
            <td>{{ item.name }}</td>
        {% endif %}
            <td>
                <a href="/edit/{{ item.id }}">
                    <button>EDIT</button>
                </a>
            </td>
            <td>
                <a href="/toggle/{{ item.id }}">
                    <button>TOGGLE</button>
                </a>
            </td>
            <td>
                <a href="/delete/{{ item.id }}">
                    <button>DELETE</button>
                </a>
            </td>
        </tr>
    {% empty %}
    <tr><td>You have nothing to do.</td></tr>
    {% endfor %}
</table>
```
- update urls.py
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.get_todo_list, name='get_todo_list'),
    path('add', views.add_item, name='add'),
    path('edit/<item_id>', views.edit_item, name='edit'),
    path('toggle/<item_id>', views.toggle_item, name='toggle'),
    path('delete/<item_id>', views.delete_item, name='delete')
]
```
- update views.py
```
def delete_item(request, item_id):
    item = get_object_or_404(Item, id=item_id)    
    item.delete()
    return redirect('get_todo_list')
```

### Testing ###

- open tests.py in todo folder
  - first test
```
class TestDjango(TestCase):

    def test_this_thing_works(self):
        self.assertEqual(1, 0)

```
- python3 manage.py test
  - [Test Resulte]()

  - second test
´´´
class TestDjango(TestCase):

    def test_this_thing_works(self):
        self.assertEqual(1, 1)

    def test_this_thing_works2(self):
        self.assertEqual(1, 3)

    def test_this_thing_works3(self):
        self.assertEqual(1, )

    def test_this_thing_works4(self):
        self.assertEqual(1, 4)

```
- python3 manage.py test
  - [Test Resulte]()

- third test
´´´
class TestDjango(TestCase):

    def test_this_thing_works(self):
        self.assertEqual(1, 1)

    def test_this_thing_works2(self):
        self.assertEqual(1, 1)

    def test_this_thing_works3(self):
        self.assertEqual(1, 1)

    def test_this_thing_works4(self):
        self.assertEqual(1, 1)

```
- python3 manage.py test
  - [Test Resulte]()


- create test_views.py
- create test_models.py
- create test_forms.py

- Testing forms.py
  - open test_forms.py
```
from django.test import TestCase
from .forms import ItemForm


class TestDjangoBase(TestCase):
    def test_this_thing_works(self):
        self.assertEqual(1, 1)


class TestItemForm(TestCase):
    def test_item_name_is_required(self):
        form = ItemForm({'name': ''})
        self.assertFalse(form.is_valid())
        self.assertIn('name', form.errors.keys())
        self.assertEqual(form.errors['name'][0], 'This field is required.')

    def test_done_field_is_not_required(self):
        form = ItemForm({'name': 'Test Todo Item'})
        self.assertTrue(form.is_valid())

    def test_fields_are_explicit_in_form_metaclass(self):
        form = ItemForm()
        self.assertEqual(form.Meta.fields, ['name', 'done'])

```
- python3 manage.py test todo.test_forms
- python3 manage.py test todo.test_forms.TestItemForm
- python3 manage.py test todo.test_forms.TestItemForm.test_fields_are_explicit_in_form_metaclass

- Testing views.py
  - open test_views.py
```
from django.test import TestCase
from .models import Item


class TestViews(TestCase):
    def test_get_todo_list(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'todo/todo_list.html')

    def test_get_add_item_page(self):
        response = self.client.get('/add')
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'todo/add_item.html')

    def test_get_edit_item_page(self):
        item = Item.objects.create(name='Test Todo Item')
        response = self.client.get(f'/edit/{item.id}')
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'todo/edit_item.html')

    def test_can_add_item(self):
        response = self.client.post('/add', {'name': 'Test Added Item'})
        self.assertRedirects(response, '/')

    def test_can_delete_item(self):
        item = Item.objects.create(name='Test Todo Item')
        response = self.client.get(f'/delete/{item.id}')
        self.assertRedirects(response, '/')
        existing_items = Item.objects.filter(id=item.id)
        self.assertEqual(len(existing_items), 0)

    def test_can_toggle_item(self):
        item = Item.objects.create(name='Test Todo Item', done=True)
        response = self.client.get(f'/toggle/{item.id}')
        self.assertRedirects(response, '/')
        updated_item = Item.objects.get(id=item.id)
        self.assertFalse(updated_item.done)

    def test_can_edit_item(self):
        item = Item.objects.create(name='Test Todo Item')
        response = self.client.post(f'/edit/{item.id}', {'name': 'Updated Name'})
        self.assertRedirects(response, '/')
        updated_item = Item.objects.get(id=item.id)
        self.assertEqual(updated_item.name, 'Updated Name')

```
- python3 manage.py test todo.test_views


- Testing models.py
  - open test_models.py
```
from django.test import TestCase
from .models import Item


class TestModels(TestCase):
    def test_done_default_to_false(self):
        item = Item.objects.create(name='Test Todo Item')
        self.assertFalse(item.done)

    def test_item_string_method_returns_name(self):
        item = Item.objects.create(name='Test Todo Item')
        self.assertEqual(str(item), 'Test Todo Item')

```
- python3 manage.py test todo.test_models

- pip3 install coverage
- coverage run --source=todo manage.py test
- coverage report
- coverage html
  - this command created the folder htmlcov
- python3 -m http.server
  - open browser 8000
  - click on htmlcov in browser
  - test report displayes on browser


### Run App with Heroku ###

**requirements and settings update**
- CLI ???

- pip3 install psycopg2-binary
- pip3 install gunicorn
- pip3 freeze --local > requirements.txt
- heroku apps:create ckz8780-django-todo-app --region eu
- heroku apps

Heroku
- Resourses
  - Add-ons: heroku postgres (Postgres is an Heroku Database)
  - click provision
- Settings
  - Config Vars

Terminal
- heroku addons


- pip3 install dj-database-url
- pip3 freeze --local > requirements.txt
- heroku config
- open settings.py
```
import os
import dj_database_url
from pathlib import Path

# Database
# https://docs.djangoproject.com/en/3.2/ref/settings/#databases

# DATABASES = {
#    'default': {
#        'ENGINE': 'django.db.backends.sqlite3',
#        'NAME': BASE_DIR / 'db.sqlite3',
#    }
# }


DATABASES = {
    'default': dj_database_url.parse('postgres://...')
}

```
- python3 manage.py migrate

**push to github**

- git remote -v
- create .gitignore
```
*.sqlite3
__pycache__/

```
- ls -la
- git add .
- git commit -m "prepared to deploy to Heroku"
- git push origin master
- git push heroku master
- heroku config:set DISABLE_COLLECTSTATIC=1
- git push heroku master
- heroku logs --tail
- create Procfile
```
web: gunicorn django_todo.wsgi:application

```
- git add Procfile
- git commit -m "Added Procfile"
- git push heroku master
- settings.py
```
ALLOWED_HOSTS = ['createdname.herokuapp.com']

```
- git add django_todo/settings.py
- git commit -m "fixed allowed hosts"
- git push heroku master
- git push origin master


Heroku
- open app
- deploy
- click GitHub
- search for repo
- click connect
- click enable automatic deploys

- settings.py
```
SECRET_KEY = os.environ.get('SECRET_KEY', 'django-insecure-=^tv8%qye_+$)hda674#erhfyn&qe8d57wggoc)l4&cgra-owq')


ALLOWED_HOSTS = [os.environ.get('HEROKU_HOSTNAME')]


DATABASES = {
        'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
    }
```

Heroku
- settings
  - config vars
    - (Key)HEROKU_HOSTNAME  (Value)createdname.herokuapp.com

- git add .
- git commit -m "Set up environment variables and automatic deployments"
- git push origin master


- settings.py
```
import os
import dj_database_url
from pathlib import Path

development = os.environ.get('DEVELOPMENT', False)


# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = development

if development:
    ALLOWED_HOSTS = ['localhost']
else:
    ALLOWED_HOSTS = [os.environ.get('HEROKU_HOSTNAME')]


# Database
# https://docs.djangoproject.com/en/3.2/ref/settings/#databases

if development:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': BASE_DIR / 'db.sqlite3',
    }
}
else:
    DATABASES = {
        'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
    }
```

- open Gitpod workspace
  - Settings / Variables
  - Add new Variable
    - Name: DEVELOPMENT Value: True
  - Workspaces
    - stopp  Workspace
    - start Workspace
- python3 manage.py runserver
- git add .
- git commit -m "setup development environment"
- git push origin master 


- settings.py
```
SECRET_KEY = os.environ.get('SECRET_KEY', '')

```

- https://miniwebtool.com/de/django-secret-key-generator/
  - copy & past generated key
    - open Gitpod workspace
    - Settings / Variables
    - Add new Variable
        - Name: SECRET_KEY Value: 96xdl+p46(98^i9^(kyi%mw^k91#lnjxd8d4(02x6dv8hn+hxp
    - Workspaces
        - stopp  Workspace
        - start Workspace

- python3 manage.py runserver

- https://miniwebtool.com/de/django-secret-key-generator/
  - copy & past generated key
  Heroku
    - Settings
      - Config Vars
        - Key: SECRET_KEY Value: 6$28e3_kcjwvtn*!%bf82b1^7^r73tf(r(plfyfb4*4z*m#0-0

- python3 manage.py runserver
- git add .
- git commit -m "removed secret key"
- git push origin master 