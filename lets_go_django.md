Install Vagrant: 

* Install virtualbox
    * https://www.virtualbox.org/wiki/Downloads

* Install Vagrant
    * https://www.vagrantup.com/downloads

* Setup a newish ubuntu - it is an easy linux
    * See how to do that at vagrant cloud
    * https://vagrantcloud.com/ubuntu
    * https://vagrantcloud.com/ubuntu/boxes/trusty64

Then perform the following action:

    vagrant init ubuntu/trusty64; vagrant up --provider virtualbox

This will cause the Ubuntu trusty (released April, 2014) to be installed in your VM.  You can think of this as a new (slightly slow) computer.

This actually takes 15-30 minutes depending on the speed of your internet connection, and you'll need to be sure to have plenty of disk space.

Once installed, you will have a virtual linux "box."  You can control the parameters with VirtualBox, or by changing the VagrantFile.  

Once the installation is complete, you can log into the box like so:

    vagrant ssh

Once in the machine, you will be the Vagrant user, and an administrator on that machine.  

You will then want to check the python and pip versions:

    python --version

This should be *at least* 2.7!  If it is 2.6 or earlier, you should seek out some resources to upgrade your python to 2.7 or newer.

Then, you'll want to install pip if you don't have it already - pip is a python pacakge management tool.  It automatically downloads, configures and installs packages that you would otherwise find on the python package index.  This index is at https://pypi.python.org/pypi

You can use https://pypi.python.org/pypi to narrow down which packages you need to install, and what the appropriate version is for you.  It also usually includes links to documentation, so you can figure out how to use the packages you are installing!

    `% sudo apt-get install python-pip`

Now, let us check the version of pip that was installed.  If it is an older version, (prior to version 7) we should update pip with itself!
    
    `% pip --version`
    `% sudo pip install --upgrade pip`
    `% source ~/.bashrc # or logout and log back in`
    `% pip --version`

Now that pip is installed, we should verify that we have a recent version of virtualenv.  This is a tool that does 2 things: it creates a sandbox that allows you to have different versions of different packages installed for different projects that you may be using.  

For instance, you may have a website that requires Django 1.4 but will not work with more recent versions, and you may be working on a site that requires Django 1.8, and is incompatible with prior versions.  Virtualenv allows both projects to have their own environments, so as a developer, you don't have to worry about package versions conflicting with either what is installed on your system, or with packages installed for other projects.

You can also control which version of python should be installed for this project.  Since that's a slightly more advanced feature, we won't cover it here.

    `% virtualenv --version`

Let's make a new virtualenv.

    `% mkdir project`
    `% virtualenv .`
    `% ls`

What we see here are the following directories:

    ```
    drwxrwxr-x 2 vagrant vagrant 4096 Nov 15 19:24 bin
    drwxrwxr-x 3 vagrant vagrant 4096 Nov 15 19:24 lib
    drwxrwxr-x 2 vagrant vagrant 4096 Nov 15 19:24 local
    ```

These are all placed here by virtualenv.  Let's activate our virtualenvironment.

    `% . bin/activate`

This runs some bash shell commands that reorganize where your paths point, to prefer local resources when it comes to python.

You can see when you are in a virtualenv, because, by default, it adds a decoration to your prompt.  This just tells you which project you are working on.

You can deactivate a virtual environment by typing `deactivate`, but we won't do that just yet.


Now we're making a django project, so lets install Django.

    % pip install Django

Notice that we did not need to use `sudo` here - that's because we are not installing Django system-wide, we are only installing it into our project.  Also, Because I didn't tell pip what version to install, it will default to the most recent version available.

Now, when you do `ls bin`, you will see a bunch of new django commands that are ready for your use.  Any time you have a virutalenv active, and you do `pip install` everything is installed locally.  If you do a `pip install` with no virtual environment active, it will try to install packages system-wide, which is not usually what you want.

Now we have Linux, Python and Django ready to go.  The rest of this presentation is based a bit on the django tutorial, but we'll be building a "list of lists" app, instead of a poll.

    > You can see the Django tutorial here: https://docs.djangoproject.com/en/1.8/intro/tutorial01/

Now that Django's installed, lets create a new project:

    `% django-admin startproject list_of_lists`


This creates a brand new project, called list_of_lists in a directory of the same name.

    `% cd list_of_lists`

When we enter this directory, we see a manage.py file and a directory called list of lists.

First thing to do is make sure the database is ready, so lets migrate it:

    `% python manage.py migrate`

Once that's done, you should have a site you can use:

    `% python manage.py runserver 0:8000`

Now, in your web browser, try to navigate to localhost:8000.  

It didn't work!  Why?

Well, it turns out that since your system is running inside a VM, you need to configure
Vagrant to *forward the ports* to your host machine.  Let's do that now.  

Let's quit the vagrant machine, then in your project directory, modify the VagrantFile:

Add this line to your vagrant config, right underneath the `config.vm.box` line:

    `config.vm.network "forwarded_port", guest: 8000, host: 8000`

Let's restart the vagrant box, then log back in:

``` 
      % vagrant halt
      % vagrant up 
      % vagrant ssh
```

Now, let's do the following:

    % cd project
    % . bin/activate # virtualenv ON
    % cd list_of_lists
    % python manage.py runserver 0:8000

Now, we should see the *it works* page!  If so, you have a django project up and running.

Now, let's create a new app called "lister" - it will help us maintain our lists:

    % python manage.py startapp lister

When we `ls`, we will see a new directory created for this new app.

Lets add that app to our settings file, then set up the URLs for it.

In settings.py there is a tuple called INSTALLED_APPS, lets add our new app to that.  The installed apps tuple should now look like this:

    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'lister', # <-- our new app!
    )

All apps are made of a model, a view, and some templates.  There are other components, but these are the bare minimum to get started.  First, we should think about our model.

Since we're creating and managing lists, and those lists are lists of things, lets create two models.  One is a List, the other is a Thing.

Lets create the List model first:  
    
    class List(models.Model):
        name = models.CharField(max_length=64)
        slug = models.SlugField(max_length=64)

So, a list just has a name and a "slug" - the name is the name of the list and the slug is just a nice all-lower-case string that is useful for making cool URLs.  The Slug is usually based on the name, and is often autogenerated from the name.

Now let's make the "Thing"  these are the things that go in the list.

    class Thing(models.Model):
        my_list = models.ForeignKey(List)
        name = models.CharField(max_length=64)
        slug = models.SlugField(max_length=64)
        done = models.BooleanField()

What's cool about the Thing is that it knows which list it belongs to.  It does this through a *Foreign Key* relationship.  In this case, we are saying that each thing belongs to one list, but the relationship is NOT 2 way, which means that one of our lists can have lots of *Things* associated to it.

Ok! Our models are done!  Let's migrate the DB again, to make sure our database is ready for our new models:

    % python manage.py makemigrations
        Migrations for 'lister':
          0001_initial.py:
            - Create model List
            - Create model Thing

Now, let's run that migration:

    % python manage.py migrate

        Operations to perform:
          Synchronize unmigrated apps: staticfiles, messages
          Apply all migrations: admin, contenttypes, lister, auth, sessions
        Synchronizing apps without migrations:
          Creating tables...
            Running deferred SQL...
          Installing custom SQL...
        Running migrations:
          Rendering model states... DONE
          Applying lister.0001_initial... OK

So we have models - at this point, we could create a shell and mess around with the models, but lets wait until we have some views and URLs set up.

Now that we've figured out our model, we need to look at templates and views.

There are a couple of ways to create views in Django - one is to create them in the Object Oriented style, and the other is to create them as simple functions.  For bigger projects it makes a lot of sense to use Object Oriented views.  But for our simple project, we'll just use functions.

Here are some of the actions that we'll perform on our list of lists:

    * View the list of lists! (index)
    * Add a new list
    * Remove a list
    * View a List
    * Add an item to a list

So, it stands to reason that we'll create a view for each of these.  Don't worry!  It's actually really simple to do!  Every view just takes a request and returns an HTTP Response.  You can find the details for different kinds of responses here: https://docs.djangoproject.com/en/1.8/ref/request-response/

Let's set up the URLs first, so in `list_of_lists` there is a file called URLs.  There are lots of ways to add URLs that are more portable, but for now, lets just keep it simple and add our new apps URLs here:

    urlpatterns = [
        url(r'^$', view_lists),
        url(r'^/$', view_lists),
        url(r'^add/$', add_new_list),
        url(r'^(?P<list_slug>[a-z\-]+)$', view_list),
        url(r'^(?P<list_slug>[a-z\-]+)/add/$', add_new_thing),
        url(r'^admin/', include(admin.site.urls)),
    ]


Now that those are in, you can see that they are pointing to some views.  Let's define those views.  Notice the ModelForm here.  Often this will go into a separate file where you maintain all of your forms, but for now we're just keeping it in with the views.  This form can be used to help create new lists, and to verify and clean the input from the network.

    from django.shortcuts import render
    from django.http import HttpResponse, HttpResponseRedirect
    from django.utils.text import slugify
    from django.template import loader, RequestContext
    from django.forms import ModelForm

    from . models import List, Thing


    class ListModelForm(ModelForm):
        class Meta:
            model = List
            fields = ['name']

    class ThingModelForm(ModelForm):
         class Meta:
             model = Thing
             fields = ['name', 'done']

    def view_lists(request):

        lists = List.objects.all()
        template = loader.get_template('lister/index.html')
        form = ListModelForm()
        return HttpResponse(
            template.render(RequestContext(request, {'my_lists': lists, 'form': form}))
        )

    def view_list(request, list_slug):
        things = List.objects.get(slug=list_slug).thing_set.all()
        template = loader.get_template('lister/view_list.html')
        form = ThingModelForm()
        return HttpResponse(
            template.render(RequestContext(request, {'list_slug': list_slug, 'form': form, 'things': things}))
        )

    def add_new_list(request):
       # add the list, then redirect to the list of lists
       f = ListModelForm(request.POST)
       if f.is_valid():
           l = f.save(commit=False)
           l.slug = slugify(l.name)
           l.save()
       return HttpResponseRedirect('/')

    def add_new_thing(request, list_slug):
        f = ThingModelForm(request.POST)
        if f.is_valid():
            t = f.save(commit=False)
            t.slug = slugify(t.name)
            t.my_list = List.objects.get(slug=list_slug)
            t.save()
        return HttpResponseRedirect('/%s' % list_slug)


The views we created use templates, so lets add those

    # base.html

    <!doctype html>
    <html>
    <head><title>List of Lists!</title></head>
    <body>
    {% block contents %}
    {% endblock %}
    </body>
    </html>


    # index.html

    {% extends 'lister/base.html' %}
    {% block contents %}
    <p> Pick a list
            <ol>
        {% for l in my_lists %}
        <li><a href="/{{ l.slug }}"> {{l.name}} </a></li>
        {% empty %}
            <li>No lists available!</li>
            {% endfor %}
            </ol>
    </p>
    <p>
        Or, make a new one:
        <form method=POST action="/add/">
            {% csrf_token %}
        {{ form }}
            <input type="submit" value="Make a List"/>
        </form>
    </p>
    {% endblock %}


    # view_list.html

    {% extends 'lister/base.html' %}
    {% block contents %}
    <p>
        <ul>
         {% for t in things %}
             <li> {{ t.name }} </li>
         {% endfor %}
         <li> <a href="/"> Back </a> </li>
        </ul>
    </p>
    <p>
       Or make a new thing:
    <form method="POST" action="{{ list_slug }}/add/">
         {% csrf_token %}
         {{ form }}
         <input type="submit" value="Add a new thing to this list">
    </form>
    </p>
    {% endblock %}


