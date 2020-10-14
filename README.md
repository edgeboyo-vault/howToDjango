# Setting up new django programs

This guide will explain how to set up new Django application on a multi-VirtualHost Apache2 setup.

## Creating a new project directory

To start you'll need to set up a project directory and it's virtual enviroment

```
mkdir projectname
cd projectname
mkdir projectname
```

For `projectname` you can use anything unique. I'd suggest the domain name or some variation of that.
Inside said project you also create another folder for the source of program. That's where the wsgi.py and other django settings files go.

## Creating a virtual enviroment

Now you need to create a virtual enviroment that'll hold all the dependacies and module of this project.
```
virtualenv projectnameenv
source ./projectnameenv/bin/activate
```
For the name you can use the project name + *env*.
Now you should see a `(projectnameenv)` in the start line of you shell. That means you're modifying this enviroment.
It's time to install django and any other app modules.
```
pip install django
[...]
```
You should install all needed modules now, those will only be used by this app in order to compartmentlise it.
Now to leave the venv.
```
deactivate
```
Any time you need to install a new module, use `source ./projectnameenv/bin/activate` and install them using pip.

## Setting up a new apache VirtualHost

Use this as a tempate and add it to the oneapp.conf or any **activated** sites-avalible site (you can activate a new site using `a2ensite newsite` and use that instead).

```apacheconf
<VirtualHost *:80>

    ServerName www.example.com
    ServerAlias example.com
    ServerAdmin webmaster@example.com

    DocumentRoot /path/to/projectname

    <Directory /path/to/projectname>
    <IfVersion < 2.4>
        Order allow,deny
        Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
        Require all granted
    </IfVersion>
    </Directory>
    
    WSGIDaemonProcess projectname python-home=/path/to/projectname/projectnameenv python-path=/path/to/projectname
    WSGIProcessGroup projectname
    WSGIScriptAlias / /path/to/projectname/projectname/wsgi.py

    ErrorLog /var/log/projectname.log
</VirtualHost>
```

Now to explain a couple of these:

`DocumentRoot` - is the location where this VHost will look for resources you need to provide the absolute path of your **project**, so the initial *projectname* directory.
to find the absolute path you can use the `pwd` command.

`<Directory /path/to/projectname>` - same as `DocumentRoot`

`WSGIDaemonProcess` - can be broken down like so:
* first you declare the daemon name so for simplicity *projectname* 
* then after the `python-home=` directive you declare the 
absolute path of the virtual enviroment folder. Again can be found with `pwd` but it should be within the *projectname* project folder 
* then after the `python-path=`directive you declare the absolute path of **project folder** (same as `DocumentRoot`)

`WSGIProcessGroup` - same as the daemon name from `WSGIDaemonProcess`

`WSGIScriptAlias` - first a slash, then a whitespace, then the location of the `wsgi.py` which should be in the **source** folder within the **project** folder.

Lastly the `ErrorLog` is an optional like, but is very handy as it creates a file that contains all the error messages created by this particular VHost. 
Any path you like can be used, but */var/log* is used usually.
