# Django running on Vercel


## Tutorial


### Install Django

```
$ mkdir vercel-django-example
$ cd vercel-django-example
$ pip install Django
$ django-admin startproject vercel_app .
```

### Add an app

```
$ python manage.py startapp example
```

Add the new app to your application settings (`project/settings.py`):
```python
# vercel_app/settings.py
INSTALLED_APPS = [
    # ...
    'portfolio',
    'authapp',

```

Be sure to also include your new app URLs in your project URLs file (`project/urls.py`):
```python
# vercel_app/urls.py
from django.urls import path, include

urlpatterns = [
    ...
    path('admin/', admin.site.urls),
    path('',include('portfolio.urls')),
    path('auth/',include('authapp.urls')),
```


#### Create the first view

Add the code below (a simple view that returns the current time) to `portfolio/views.py`:
```python
# example/views.py

from urllib import response
from django.shortcuts import render,redirect
from django.contrib import messages
from portfolio.models import Contact,Blogs
from django.http import FileResponse 
import os

def blog(request):
  posts=Blogs.objects.all()
  context={"posts":posts}
  return render(request,'blog.html',context)

def home(request):
  return render(request,'home.html')

def about(request):
  return render(request,'about.html')

def skils(request):
  return render(request,'skils.html')

def contact(request):
  if request.method=="POST":
    fname=request.POST.get('name')
    femail=request.POST.get('email')
    fphoneno=request.POST.get('num')
    fdesc=request.POST.get('desc')
    query=Contact(name=fname,email=femail,phonenumber=fphoneno,description=fdesc)
    query.save()
    messages.success(request,"Thanks for contacting us. We will get by you soon!")
    
    return redirect('/contact')
  
  return render(request,'contact.html')

def resume(request):
  file_path=os.path.join('static/assets/myapp/','resume.pdf')
  return
  FileResponse(open(file_path,'rb'),
               as_attachment=True,
               filename='resume.pdf')
  
```


#### Add the first URL

Add the code below to a new file `portfolio/urls.py`:
```python
# example/urls.py
from django.contrib import admin
from django.urls import path,include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include('portfolio.urls')),
    path('auth/',include('authapp.urls')),
    
] + static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT)

```


### Test your progress

Start a test server and navigate to `localhost:8000`, you should see the index view you just
created:
```
$ python manage.py runserver
```

### Get ready for Now

#### Add the Now configuration file

Create a new file `vercel.json` and add the code below to it:
```json
{
    "builds": [{
        "src": "project/wsgi.py",
        "use": "@ardnt/vercel-python-wsgi",
        "config": { "maxLambdaSize": "15mb" }
    }],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "project/wsgi.py"
        }
    ]
}

```
This configuration sets up a few things:
1. `"src": "project/wsgi.py"` tells Vercel that `wsgi.py` contains a WSGI application
2. `"use": "@ardnt/vercel-python-wsgi"` tells Now to use the `vercel-python-wsgi` builder (you can
   read more about the builder at https://github.com/ardnt/vercel-python-wsgi)
3. `"config": { "maxLambdaSize": "15mb" }` ups the limit on the size of the code blob passed to
   lambda (Django is pretty beefy)
4. `"routes": [ ... ]` tells Now to redirect all requests (`"src": "/(.*)"`) to our WSGI
   application (`"dest": "project/wsgi.py"`)


#### Add Django to requirements.txt

The `vercel-python-wsgi` builder will look for a `requirements.txt` file and will
install any dependencies found there, so we need to add one to the project:
```
# requirements.txt
asgiref==3.6.0
certifi==2022.12.7
charset-normalizer==3.0.1
coreapi==2.3.3
coreschema==0.0.4
Django==4.1.6
django-rest-swagger==2.2.0
djangorestframework==3.14.0
drf-yasg==1.21.5
idna==3.4
inflection==0.5.1
itypes==1.2.0
Jinja2==3.1.2
MarkupSafe==2.1.2
openapi-codec==1.3.2
packaging==23.0
psycopg2==2.9.5
psycopg2-binary==2.9.5
pytz==2022.7.1
requests==2.28.2
ruamel.yaml==0.17.21
ruamel.yaml.clib==0.2.7
simplejson==3.18.3
sqlparse==0.4.3
uritemplate==4.1.1
urllib3==1.26.14
```


#### Update your Django settings

First, update allowed hosts in `settings.py` to include `.now.sh`:
```python
# settings.py
ALLOWED_HOSTS = ['.vercel.app']
```

Second, get rid of your database configuration since many of the libraries Django may attempt to
load are not available on lambda (and will create an error when python can't find the missing
module):
```python
# settings.py
DATABASES = {}
```


### Deploy

With now installed you can deploy your new application:
```
$ vercel
Vercel CLI 21.3.3
? Set up and deploy “vercel-django-example”? [Y/n] y
...
? In which directory is your code located? ./
...
✅  Production: https://vercel-django-example.vercel.app [copied to clipboard] [29s]
```

Check your results in the [Vercel dashboard](https://vercel.com/dashboard).
