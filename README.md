
# Simple CRUD API with Django & Django REST Framework (DRF)

## Table of Contents
1. [Introduction](#introduction)
2. [Phase 1: Setting Up the Environment](#phase-1-setting-up-the-environment)
3. [Phase 2: Installing Necessary Packages](#phase-2-installing-necessary-packages)
4. [Phase 3: Creating the Django Project and App](#phase-3-creating-the-django-project-and-app)
5. [Phase 4: Configuring Django Settings](#phase-4-configuring-django-settings)
6. [Phase 5: Database Setup and Migration](#phase-5-database-setup-and-migration)
7. [Phase 6: Building API Views](#phase-6-building-api-views)
8. [Phase 7: Configuring URLs](#phase-7-configuring-urls)
9. [Phase 8: Running and Testing the API](#phase-8-running-and-testing-the-api)
10. [Django Request-Response Cycle](#django-request-response-cycle)
11. [Final Folder and File Structure](#final-folder-and-file-structure)

## Tutor Details:
**Jawad Khan**  
Senior Django Developer  
[jawadkhan31849@gmail.com](mailto:jawadkhan31849@gmail.com)

Part 1: Introduction to Django/DRF and Django request response cycle: https://www.loom.com/share/e44e19c4e4374a798067de122a7945f5?sid=f0f1127d-13d3-4bae-a06d-fdf6ddbd2fdf
part 2: Installing dependences and creating a new project and app in django: https://www.loom.com/share/cedddcc54ec84f89a597fc75ee2ff0f4?sid=75959022-4fae-452b-b4b3-484c5e7b867d
part 3: Database models: https://www.loom.com/share/2d4ee70be81a43aba9d45bf005de978b?sid=641279cb-21cd-4d8c-bb4e-7bd471d0f560
part 4: APIs for saving items and getting all the saved items form databse: https://www.loom.com/share/cf27504ce59149cc9386ab5f30f18f49?sid=3ed0be8e-be14-4ab6-8c71-3d56b46904b4
part 5: testing apis with postman: https://www.loom.com/share/a380a22fc2d44863ac4e2896838242a3?sid=a5687c3a-9c34-41f4-8ae0-922d09e71444

---

## Introduction
In this tutorial, we will create a simple CRUD (Create, Read, Update, Delete) API using Django and Django REST Framework (DRF). We will use SQLite as the database. The goal is to learn how to build APIs using function-based views, work with models and serializers, and understand the request-response cycle in Django.

---

## Phase 1: Setting Up the Environment

### Step 1: Install Python
Ensure that Python is installed on your system. You can download the latest version of Python from the official website:  
[Python Download](https://www.python.org/downloads/).

- After installation, verify it by running:
  ```bash
  python --version
  ```

### Step 2: Set Up a Virtual Environment
A virtual environment is a self-contained Python environment that isolates project dependencies.

- Create a virtual environment:
  ```bash
  python -m venv venv
  ```

- Activate the virtual environment:
  - On Windows:
    ```bash
    venv\Scriptsctivate
    ```
  - On macOS/Linux:
    ```bash
    source venv/bin/activate
    ```

---

## Phase 2: Installing Necessary Packages

### Step 3: Install Django and DRF
To work with Django and Django REST Framework (DRF), you need to install these packages inside your virtual environment:
```bash
pip install django djangorestframework
```

---

## Phase 3: Creating the Django Project and App

### Step 4: Create a Django Project
A Django project is the main container for your web application. Create a new project by running:
```bash
django-admin startproject inventory
cd inventory
```

### Step 5: Create the Django App
Create a new app inside your project by running:
```bash
python manage.py startapp api
```

---

## Phase 4: Configuring Django Settings

### Step 6: Add the App and DRF to `INSTALLED_APPS`
Open `inventory/settings.py` and add `'api'` and `'rest_framework'` to the `INSTALLED_APPS` list.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'api',
]
```

---

## Phase 5: Database Setup and Migration

### Step 7: Define the Model
A model represents the structure of your database tables. Define a simple model in `api/models.py`:

```python
class Item(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()

    def __str__(self):
        return self.name
```

### Step 8: Create and Apply Migrations
Run the following commands to create and apply migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Phase 6: Building API Views

### Step 9: Create Serializers
A serializer is used to convert complex data types like Django models into native Python data types, which can be easily converted into JSON. Create a serializer for the `Item` model in `api/serializers.py`:

```python
from rest_framework import serializers
from .models import Item

class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = '__all__'
```

### Step 10: Create Function-Based Views
Define views for list, create, retrieve, update, and delete operations in `api/views.py`:

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Item
from .serializers import ItemSerializer

@api_view(['GET', 'POST'])
def item_list(request):
    if request.method == 'GET':
        items = Item.objects.all()
        serializer = ItemSerializer(items, many=True)
        return Response(serializer.data)

    if request.method == 'POST':
        serializer = ItemSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'PATCH', 'DELETE'])
def item_detail(request, pk):
    try:
        item = Item.objects.get(pk=pk)
    except Item.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = ItemSerializer(item)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = ItemSerializer(item, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'PATCH':
        serializer = ItemSerializer(item, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        item.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

---

## Phase 7: Configuring URLs

### Step 11: Set Up URL Routing
Define URL patterns for the views in `api/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('items/', views.item_list, name='item-list'),
    path('items/<int:pk>/', views.item_detail, name='item-detail'),
]
```

Include the app URLs in the project’s `inventory/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),
]
```

---

## Phase 8: Running and Testing the API

### Step 12: Run the Development Server
Run the development server with the following command:
```bash
python manage.py runserver
```

### Step 13: Test the API
Use tools like Postman or cURL to test the following endpoints:
- **GET** `/api/items/`: Get a list of all items.
- **POST** `/api/items/`: Create a new item.
- **GET** `/api/items/{id}/`: Get a single item by ID.
- **PUT** `/api/items/{id}/`: Update an item by ID.
- **PATCH** `/api/items/{id}/`: Partially update an item by ID.
- **DELETE** `/api/items/{id}/`: Delete an item by ID.

---

## Django Request-Response Cycle
1. User makes a request to a URL.
2. Django URL routing processes the request and triggers the appropriate view function.
3. The view interacts with the model to retrieve or modify data from the database.
4. The serializer converts the model data into JSON format.
5. Middleware (if any) processes the request and response for tasks like security, authentication, etc.
6. The response (in JSON format) is returned to the user.

---

## Final Folder and File Structure

```
inventory/
│
├── manage.py           # Command-line utility for managing the project
├── inventory/
│   ├── __init__.py
│   ├── settings.py     # Settings configuration
│   ├── urls.py         # URL routing for the entire project
│   └── wsgi.py
└── api/                 # Django app
    ├── __init__.py
    ├── models.py       # Database models
    ├── views.py        # Views for handling requests (function-based views)
    ├── serializers.py  # Serializers for converting models to JSON
    └── urls.py         # URL routing for the app
```
