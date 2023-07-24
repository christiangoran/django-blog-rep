# Django Blog

## SETUP

1. Install Gunicorn
- pip3 install 'django<4' gunicorn

2. Then two database libraries are needed

- pip3 install dj_database_url==0.5.0 psycopg2

3. Then install library needed to run Cloudinary

- pip3 install dj3-cloudinary-storage

4. Create requirements.txt file

- pip3 freeze --local > requirements.txt

5. Now we can create a new Django project

- django-admin startproject blog .

6. Then we start a new app

- python3 manage.py startapp blogapp

7. Now we need to add it to installed app in the settings.py file

8. We also need to migrate the changes. Whenever add a new app it is automatically added to migrate and we need to give the command

- python3 manage.py migrate

7. Now we can do the runserver command. We need to add the Django server that comes up into our allowed hosts in settings.py

8. Now, we want to prepare it for deployment to Heroku. We’re into the third step of our “new project checklist” which is setting the project up to use Cloudinary and an external database.

To do this we need to create the app on Heroku, provision a database, configure some environment variables, and modify the settings of the project to get all this working.

9. So first we create a Heroku app

10. Then we create a ElephantSQL database

11. Now we create an env.py file and import os at the top. After this we add:

-  os.environ["DATABASE_URL"]="<copiedURL>" (where url is db url from ElephantSQL)

-  os.environ["SECRET_KEY"]="my_super^secret@key" (secret key can be whatever, but maybe take the one Django generated)

12. Now we need to modify the settings.py file
first we import env.py

import os
import dj_database_url
if os.path.isfile('env.py'):
  import env

Then:

- SECRET_KEY = os.environ.get('SECRET_KEY')

13. Now that is taken care of, we need to hook up the database. We are going to use the dj_database_url import for this, so scroll down in your settings file to the database section.

Comment out the original DATABASES variable and add the code below, as shown

DATABASES = {
    'default': dj_database_url.parse(os.environ.get("DATABASE_URL"))
}

The code that has been commented out connects your Django application to the created db.sqlite3 database within your repo. However, as we know, that database is not suitable for production. This line of code separates the database URL stored by your env.py file into the relevant name and password etc.

14. We should be able to connect to the ElephantSQL database now. First we need to migrate it.

- python manage.py migrate

15.Time to push our changes to Github

16. Now comes the tricky part. Setting up Heroku

We start off by adding the database and secret key as config vars in Heroku:
Add two config vars:

- DATABASE_URL, and for the value, copy in your database URL from ElephantSQL, no need to add quotation marks.

- SECRET_KEY containing your secret key

We also add

- PORT : 8000

17. Then we add the API environment key from Cloudinary into the env.py

- os.environ['CLOUDINARY_URL'] = 'secret url' (and remove cloudinary_url= from the code)

18. Add the CLOUDINARY_URL to Heroku. The one without cloudinary_url

19. Also add DISABLE_COLLECTSTATIC and 1 to the config vars

20. Then under installed apps in settings.py add right above django.contrib.staticfiles add:

- cloudinary_storage

and then under staticfiles add:

- 'cloudinary',

21. Then near the end of our settings.py  file we can add these few lines.
First of all, "STATICFILES_STORAGE" and in here we can tell it to use,
"Cloudinary_storage.storage.StaticHashedCloudinaryStorage",
so this is coming from the  library that we installed above,
and put into our installed apps section.

22. THen right under I add

STATICFILES_DIRS = [os.path.join(BASE_DIR, 'staticfiles')]
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

MEDIA_URL = '/media/'
DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'

23. Then further up under BASE_DIR add:

TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')

24. Then still in settings.py under TEMPLATES change the value of 'DIRS'
to

[TEMPLATES_DIR]

25. then under allowed hosts, write

'heroku_app_name.herokuapp.com', 'localhost'

26. add folders media, static, templates at root

27. Finally add a Procfile and within write:

web: gunicorn blog.wsgi