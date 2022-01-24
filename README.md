# test

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

```json
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

```json
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