## Hello-Django

[Completed App](https://github.com/Nditah/hello-django) | 
[Django Official Website](https://djangoproject.com/) | 
[Getting Started](https://docs.djangoproject.com/en/4.0/intro/tutorial01/)


1. Make a Directory for the Project and navigate into it
     > mkdir hello-django && cd hello-django

2. Create and activate a Python Virtual Environment
     > python3 -m venv venv 
     > source venv/bin/activate

3. Create a Django Project
     > django-admin startproject todoapp

4. Navigate into the django-project folder just created `todoapp` and Run the server
    > cd todoapp 
    > python manage.py runserver

5. Quit the Server with Ctrl + C and create a application component inside the project `todolist`
     > python manage.py startapp todolist

6. Add the application component to the lists of apps in `todoapp/settings.py`  _INSTALLED_APPS_

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'todolist', /* added line */
]
```

7. To add a view, go to `todolist/views.py` 
   
```py
# Create your views here.
def index(request):
    return render(request, "base.html", { "todo_list": todos })
```

8. For each view, register the route in a file `urls.py` created in the same todolist directory with the view.
   
```py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index")
]
```

9.  use the `include` function to add the todolist urls in the `todoapp/urls.py` on the main app.

```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('todolist.urls')),
]
```

10.  Create the `templates/base.html` file in the project root directory sitting at the same level with todoapp and todolist directory.
Make a small modification required by Django    
```html
                {% csrf_token %}
```

11.  Go to todoapp/settings.py and add the `templates` directory to the `DIR` array entry of the `TEMPLATES` array.
```py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ['templates'], /* modified */
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```


12.  Define the todolist model inside `todolist/models.py`

```py
from django.db import models

# Create your models here.
class Todo(models.Model):
    title=models.CharField(max_length=350)
    complete=models.BooleanField(default=False)

    def __str__(self):
        return self.title
```



13. Run `python manage.py makemigrations`
    - The command creates a migration for the model Todo

14. Then run `python manage.py migrate`

15. Create a Super user for the Admin panel  `python manage.py createsuperuser`
    Then follow the instruction to create name, email and password

16. Go to `todolist/admin.py` and register the model

```py
from django.contrib import admin
from .models import Todo

# Register your models here.
admin.site.register(Todo)
```

17. No go to the `todolist/views.py` and import the Todo model

```py
from django.shortcuts import render
from .models import Todo

# Create your views here.
def index(request):
    todos = Todo.objects.all()
    return render(request, "base.html", { "todo_list": todos })
```

18. Run the server again and test `python manage.py runserver`

Starting development server at http://127.0.0.1:8000/
If you try to add a record, it throws and error.
However, the TodoApp works find through the Admin Panel http://127.0.0.1:8000/admin 

19. to implement the missing views, go to `todolist/views.py` 

```py
from django.shortcuts import render, redirect
from django.views.decorators.http import require_http_methods

from .models import Todo

# Create your views here.
def index(request):
    todos = Todo.objects.all()
    return render(request, "base.html", { "todo_list": todos })

@require_http_methods(["POST"])
def add(request):
    title = request.POST["title"]
    todo = Todo(title=title)
    todo.save()
    return redirect("index")

def update(request, todo_id):
    todo = Todo.objects.get(id=todo_id)
    todo.complete = not todo.complete
    todo.save()
    return redirect("index")

def delete(request, todo_id):
    todo = Todo.objects.get(id=todo_id)
    todo.delete()
    return redirect("index")
            
```

21. Add the views to the urlpatterns in `todolist/urls.py`
```py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('add', views.add, name='add'),
    path('delete/<int:todo_id>/', views.delete, name='delete'),
    path('update/<int:todo_id>/', views.update, name='update'),
]
```




## templates/base.html

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Todo App - Django</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.css">
        <script src="https://cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.js"></script>
    </head>
    <body>
        <div style="margin-top: 50px;" class="ui container">
            <h1 class="ui center aligned header">Django ToDo App</h1>

            <form class="ui form" action="/add" method="post">
                <div class="field">
                    <label>Todo Title</label>
                    <input type="text" name="title" placeholder="Enter ToDo task...">
                    <br>
                </div>
                <button class="ui blue button" type="submit">Add</button>
            </form>

            <hr>

            {% for todo in todo_list %} 
            <div class="ui segment">
                <p class="ui big header">{{ todo.id }} | {{ todo.title }}</p>

                {% if todo.complete == False %}
                <span class="ui gray label">Not Complete</span>
                {% else %}
                <span class="ui green label">Complete</span>
                {% endif %}

                <a class="ui blue button" href="/update/{{ todo.id }}">Update</a>
                <a class="ui red button" href="/delete/{{ todo.id }}">Delete</a>

            </div>
            {% endfor %}

        </div>


    </body>

</html>

```

Finally, restart the App `python manage.py runserver` or refresh.
Review the App at http://127.0.0.1:8000/ and the out-of-the-box [Admin Panel](http://127.0.0.1:8000/admin)
Commit and push your code to github.com and Deactivate the Virtual Env.
   
    $ deactivate
    $ conda deactivate


### 📚 Sam's other blogs

- [Dev.to](https://dev.to/nditah) | [Hashnode.dev](https://nditah.hashnode.dev/) | [Medium.com](https://nditah.medium.com/)
- [Github](https://github.com/Nditah)