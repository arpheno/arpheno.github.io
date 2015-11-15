---
layout: post
title:  "Working with an existing MSSQL database in django"
date:   2015-11-15 14:43:18
categories: django mssql inspectdb
---

Django is awesome.

I inspired a friend who is working on a administration tool for his municipal transportation
agency to have a look at python and django. 

I think the endless tirade of buzzwords like *multiple inheritance*, *context managers*,
*list comprehensions* or *generators* made the seasoned `C#` programmer curious to see
what the cool kids do these days.

Inspired by Uncle Bob's [Clean Architecture talks] I tend to fend off all questions about
which database to use with an uninvolved _"It's an implementation detail, I don't care"_,
however it quickly became clear that it doesn't work when the database has already been implemented,
and at that in a database framework I was unfamiliar with: **MSSQL**

I was working on his computer running Microsoft Windows 7,without my carefully tuned Pycharm IDE,
with a fresh installation of Python2.7.10 and the windows command line. I felt a little bit
like David Beazley in his _"Vault"_ in [Discovering Python]. 

I had one chance to impress my friend with django, and I had virtually no tools available...

But as I said, **django is awesome**.

My friend watched me struggle with pip for a while and then went to a nearby shop to get some snacks.

To import an existing database into django we need two things:

  1. Be able to talk to the database
  2. Translate the database format into models for django
  
django-mssql
------------

I didn't know there was a database adapter for mssql, but it turns out there is one. [django-mssql]
allows you to connect to the mssql2014 instance running on localhost. To install it just do:

    pip install django==1.7 django-mssql

Note that at the time of writing django-mssql only ran on django 1.7 for me.

Then you can hook up you database into django:

    DATABASES = {
        'default': {
            'NAME': 'transactions',
            'ENGINE': 'sqlserver_ado',
            'HOST': '127.0.0.1',
            'PORT': '50824',
            'USER': 'root',
            'PASSWORD': 'root',
        }
    }
Note that `HOST` must be an IP address if you're specifying `PORT`.

inspectdb
---------
After installing [django-mssql] I was more than just a little anxious to type `python manage.py shell`, 
but django connected to the database just fine.

`python manage.py inspectdb` is a superpower of django that is going to parse the tables
in a connected database and spit out python classes describing them:

    class transactions2(models.Model):
        id = models.IntegerField(null=True, blank=True)
        bill_number = models.IntegerField(null=True, blank=True)
        ....
so this little gem saved the day: `python manage.py inspectdb > testapp/models.py`

Typing `python manage.py migrate` right away is going to fail unfortunately because the
inspectdb tool doesn't know that `id` is special. You should modify the id line in your 
newly acquired `models.py`:
    
    class transactions2(models.Model):
        ...
        id = models.IntegerField(primary_key=True)
        ...
`python manage.py migrate` will now work just fine and djangofy the database, giving it 
tables for migration history and user management.

To get the most impressive effect right away I modified `testapp/admin.py`:

    from django.contrib import admin
    from models import transactions2
    admin.site.register(transactions2)

To shock my friend entirely I also created a superuser:

    python manage.py createsuperuser
   
When my friend came back about 20 minutes later I had his application imported into django
with the admin interface giving him fully editable tables, the ability to add new entries
to his tables and user management already implemented. He was speechless.

Now he's bombarding me with questions about templates and views and on a great journey 
from `C#` to Python.

Today was a good day.


[Clean Architecture talks]: https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html
[Discovering Python]: https://www.youtube.com/watch?v=RZ4Sn-Y7AP8
[django-mssql]: https://django-mssql.readthedocs.org/en/latest/