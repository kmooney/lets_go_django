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

Well, it turns out that since your system is running inside a VM, you need to configure Vagrant to *forward the ports* to your host machine.  Let's do that now.  

Let's quit the vagrant machine, then in your project directory, modify the VagrantFile:

Add this line to your vagrant config, right underneath the `config.vm.box` line:

    `config.vm.network "forwarded_port", guest: 8000, host: 8080`

Let's restart the vagrant box, then log back in:

``` 
      % vagrant halt
      % vagrant up 
      % vagrant ssh
```

