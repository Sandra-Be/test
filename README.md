# GitHub

* Add this repository to your local workspace:
    * Click on the **Crystal Energy** repository on [GitHub link]().
    * Click on the **Code** button, and copy the URL.
    * Go into your local workspace and open a new terminal.
    * Type `git clone` and paste the URL you copied from GitHub, and press Enter. It shpuld look like this:

```
git clone https://github.com/*username*/*repository*
```
The process of cloning is now completed.
* Open your IDE and run the following command to install the dependencies:
```
pip install -r requirements.txt
```
* Create an env.py file in your project root, and set your environment variables:
```
DEVELOPMENT=1
SECRET_KEY=YOUR_SECRET_KEY
STRIPE_PUBLIC_KEY=YOUR_STRIPE_PUBLIC_KEY
STRIPE_SECRET_KEY=YOUR_STRIPE_SECRET_KEY
STRIPE_WH_SECRET=YOUR_STRIPE_WH_SECRET
```
* In your settings.py file, add your environment settings:
```
import os
    
STRIPE_PUBLIC_KEY = os.getenv('STRIPE_PUBLIC_KEY', '')
STRIPE_SECRET_KEY = os.getenv('STRIPE_SECRET_KEY', '')
STRIPE_WH_SECRET = os.getenv('STRIPE_WH_SECRET', '')
SECRET_KEY = os.getenv('SECRET_KEY', '')
```
* After setting your environment varibles, run the following commands to migrate models:
```
python3 manage.py makemigrations --dry-run
python3 manage.py makemigrations
python3 manage.py migrate --plan
python3 manage.py migrate
```
* Load the data to the database from the db.json file by running following command:
```
python3 manage.py loaddata db.json
```
* Create a superuser for your app:
```
python3 manage.py createsuperuser
```
* Run the following command to start the project:
```
python3 manage.py runserver
```

# Heroku Deployment

* Log into [Heroku.com](https://www.heroku.com/).
* Select `New` on your dashboard and then select `Create new app`.
* Choose a name for your application, select the region closest to your, and then click `Create app`.
* After you created your app click on Resources tab, using the `Add ons` search field, find and select `Heroku Postgres`.
* Select your plan and click confirm.
* In order to use Heroku Postgres you need to install two dependencies `dj_database_url` and `psycopg2-binary`:
```
pip3 install dj_database_url
pip3 install psycopg2-binary
```
* After installing the dependencies, freeze your requirements into `requirements.txt`:
```
pip3 freeze > requirements.txt
```
* In your settings.py file import `dj_database_url` and replace your current Database settings to: 
```
import dj_database_url
    
DATABASES = {
    'default': dj_database_url.parse('DATABASE_URL')
}
```
Your `DATABASE_URL` can be found in your Heroku Apps `Config Var`.
* After setting your Database, run the following commands to migrate models:
```
python3 manage.py makemigrations --dry-run
python3 manage.py makemigrations
python3 manage.py migrate --plan
python3 manage.py migrate
```
* Load the data to the database from the db.json file by running following command:
```
python3 manage.py loaddata db.json
```
* Create a superuser for your app:
```
python3 manage.py createsuperuser
```
* Heroku setup is complete, now add an if statement in your settings.py file to set the DATABASES:
```
if 'DATABASE_URL' in os.environ:
    DATABASES = {
        'default': dj_database_url.parse(os.environ.get('DATABASE_URL'))
    }
    else:
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }
```
* Install `gunicorn` and freeze your requirements:
```
pip3 install gunicorn
pip3 freeze > requirements.txt
```
* Create a `Procfile` and add the following code into this file:
```
echo web: gunicorn <app-name>.wsgi:application
```
* Connect to Heroku fom the terminal:
```
heroku login -i
```
* Go to back to the Settings tab on your Heroku dashboard, and click `Reveal Config Vars` and add the following Config Variable, to temporarily disable `COLLECTSTATIC`:
```
DISABLE_COLLECTSTATIC = 1
```
* In your settings.py file add your Heroku app, and `localhost`:
```
ALLOWED_HOSTS = ['YOUR_HEROKU_APP_NAME.herokuapp.com', 'localhost']
```
* In your Heroku app dashboard, click on `Settings` tab on your Heroku dashboard, and click `Reveal Config Vars` and add the following Config Variables:

| Key | Value |
|:----|:------|
| SECRET_KEY | YOUR_SECRET_KEY |
| STRIPE_PUBLIC_KEY | YOUR_STRIPE_PUBLIC_KEY |
| STRIPE_SECRET_KEY | YOUR_STRIPE_SECRET_KEY |
| STRIPE_WH_SECRET | YOUR_STRIPE_WH_SECRET |
| EMAIL_HOST_USER | YOUR_EMAIL_HOST_USER |
| EMAIL_HOST_PASS | YOUR_EMAIL_APP_PASSWORD |

* Then push to Heroku:
```
heroku git:remote -a <your heroku app name>
git push heroku main
```
* Navigate to `Deploy` tab on your Heroku apps Dashboard, and click on `Enable Automatic Deploys`.
* Site is successfully deployed, and any futher changes on the app will automatically be updated everytime they are commited and pushed on GitHub.

# AWS Deployment

* Login to your [AWS Management Console](https://aws.amazon.com), and click on Amazon S3.
* Create a new bucket that matches your App, and uncheck `Block all public access`.
* Enable `Static website hosting` from `Properties` tab.
* Go to `Permissions tab`, and set your CORS (Cross-origin resouce sharing) settings to `enable access` between Heroku and your S3 Bucket by:
```
[
    {
        "AllowedHeaders": [
            "Authorization"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```
* Set your bucket policy using `Policy Generator`.
    * `Select Type of Policy` - S3 Bucket Policy.
    * `Principal` - `*` to allow all principles.
    * `Actions` - Select `Get Object`.
    * `Amazon Resource Name (ARN)` - Paste your Bucket ARN and add `*` at the and of your Bucket Resource key `arn:aws:s3:::bucket_name/*` and then save.
* Click on Access Control List (ACL), and enable `List` on `Everyone (public access)` tab.
* On the top of your AWS Management Console, search for `IAM (Identity Access Management)`.
    * Click on `User Groups` on the left panel, and `Create Group`.
    * Click on `Policies` and `Create Policy`.
    * Click on JSON and select `Import Managed Policy` and search for `AmazonS3FullAccess` and click import.
    * Copy your `Bucket ARN` and paste it into the `Resource`:
```
    "Resource": [
        "arn:aws:s3:::bucket_name",
        "arn:aws:s3:::bucket_name/*"
    ]
```
    * Click on `Review Policy`.
* Go back to `User Groups` and click on the group name you just created, click on the `Permissions` then `Attach Policies` and search for the policy you've just created and then click on `Attach Policy` to attach the policy to the group.
* Click on `Users` and then click on `Add Users`.
    * Set your user's name and give `Programmatic Access`.
    * Click `Next` and add the user in your New Group, and `Create User`.
    * After you created the user download user's `.csv` file which contains user's access key and secret access key.
* Go back to your IDE and install the following dependencies in order to connect Django to AWS S3:
```
pip3 install boto3
pip3 install django-storages
```
* Freeze your requirements:

```
pip3 freeze > requirements.txt
```
* Add it to your installed apps in your settings.py
* Create `custom_storages.py` file in your project root and add the following code, and then save:
```
from django.conf import settings
from storages.backends.s3boto3 import S3Boto3Storage
    
    
class StaticStorage(S3Boto3Storage):
    location = settings.STATICFILES_LOCATION
    
    
class MediaStorage(S3Boto3Storage):
    location = settings.MEDIAFILES_LOCATION
```
* In your `settings.py` file add the following code:

```
# AWS Backend Configuration

    if 'USE_AWS' in os.environ:
        # Cache control
        AWS_S3_OBJECT_PARAMETERS = {
            'Expires': 'Thu, 31 Dec 2099 20:00:00 GMT',
            'CacheControl': 'max-age=94608000',
        }
        
        # Bucket Config
        AWS_STORAGE_BUCKET_NAME = 'YOUR_BUCKET_NAME'
        AWS_S3_REGION_NAME = 'YOUR_BUCKET_REGION'
        AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
        AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'
    
        # Static and media files
        STATICFILES_STORAGE = 'custom_storages.StaticStorage'
        STATICFILES_LOCATION = 'static'
        DEFAULT_FILE_STORAGE = 'custom_storages.MediaStorage'
        MEDIAFILES_LOCATION = 'media'
    
        # Override static and media URLs in production
        STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/{STATICFILES_LOCATION}/'
        MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/{MEDIAFILES_LOCATION}/'
```
* Add the following config variables in your Heroku App, and remove `DISABLE_COLLECTSTATIC=1`:

| Key | Value |
|:----|:------|
| USE_AWS | True |
| AWS_ACCESS_KEY_ID | YOUR_AWS_ACCESS_KEY_ID |
| AWS_SECRET_ACCESS_KEY | YOUR_AWS_SECRET_ACCESS_KEY |

* Your deployment is now complete.