## Set up the project

In a new project directory, create a new virtual environment
```
virtualenv venv
```

Install all of the things you'll need (psycopg2 is for PostgreSQL)
```
pip install django
pip install djangorestframework
pip install psycopg2
```

Put you're dependencies in a requirements.txt (basically a package.json)
```
pip freeze > requirements.tx
```

Don't forget to activate the venv or you'll probably get errors
```
source venv/bin/activate
```

Make project with 
```
django-admin startproject bookAPI
```

make a new app in that directory with 
```
python manage.py startapp books
```

### Create a database (using PostgreSQL)

(in terminal): 
```
createdb books
```

(if not installed already): 
```
pip install psycopg2
```

in settings.py, change DATABASES to: (and make sure to change name to your db's name)
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'books',
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

and add the app to INSTALLED_APPS (also in settings.py). Also add the rest\_framework. For example:
```
INSTALLED_APPS = [
    'rest_framework',
    'books.apps.BooksConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

in models.py: make a class for each table. For example: (btw the def \_\_str\_\_ will give a title in the admin interface)
```
class Book(models.Model):
    title = models.CharField(max_length=50)
    publication_date = models.integerField
    def __str__(self):
        return self.title
```

Make a migration for the model
```
python manage.py makemigrations books
```

Run the migration
```
python manage.py migrate
```

Check it out in psql to make sure!

## Make an admin user to easily edit stuff

Make a superuser
```
python manage.py createsuperuser
```
and fill out all the info

Add your app to the admin interface in admin.py
```
from .models import Book

admin.site.register(Book)
```

## Making a serializer

Make a serializers.py file in the app directory

Inside of it, import serializers and any models you need. For example:
```
from rest_framework import serializers
from books.models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ('id', 'title', 'publication_date')
```

Start writing routes! You will need to change views.py and urls.py (will need to be created) in your app folder and the urls.py that comes in your project directory.

### Example of the project with /books/ and /books/id route

Note: If you want it in json format add a .json at the end (e.g. /books/4.json)

view.py

```
from books.models import Book
from books.serializers import BookSerializer
from django.http import Http404
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status


class BookList(APIView):
    def get(self, request, format=None):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class BookDetail(APIView):
    def get_object(self, pk):
        try:
            return Book.objects.get(pk=pk)
        except Book.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        book = self.get_object(pk)
        serializer = BookSerializer(book)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        book = self.get_object(pk)
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        book = self.get_object(pk)
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

urls.py in book directory:
(format_suffix_patters lets you use .json or .api to see the data differently)
```
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from books import views

urlpatterns = [
    url(r'^books/$', views.BookList.as_view()),
    url(r'^books/(?P<pk>[0-9]+)/$', views.BookDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

urls.py in project directory:

```
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^', include('books.urls')),
]

```