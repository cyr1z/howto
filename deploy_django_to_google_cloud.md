#  Deploy Django project to Ubuntu instance

We use Pipenv, Postgres, python3, Django 3, .env file for settings and git repository with project


## On our local project

### DB dump    

    dumpdata --exclude auth.permission --exclude contenttypes > db.json
### Pipenv lock

```sh
pipenv lock
```

------

## On server

### Install *python, nginx* and *postgresql*

```shell
sudo apt update
```
```sh
sudo apt install python3-pip python3-dev python3-pip libpq-dev postgresql postgresql-contrib nginx curl
```

```shell
pip install pipenv
```

### Create database

User: `myuser`

DB: `mydb`

password: `mypass`

```
sudo -u postgres psql
```

```sql
create database mydb with encoding 'UTF8';
```

```sql
create user myuser with password 'mypass';
```

```sql
grant all on database mydb to myuser;
```

### Clone project

```shell
git clone git@github.com:some_user/some_project.git
```

### Create virtual environment

```shell
cd some_project
pipenv shell
```

### Install the requirements

```shell
pipenv install --ignore-pipfile
pipenv install gunicorn
```

### Create `.env` file for environment variables

```shell
nano .env
```

```shell
DEBUG = False
SECRET_KEY = '---------=======django-secret=========---------'
DBNAME = 'mydb'
DBUSER = 'myuser'
DBPASSWORD = 'mypass'
DBHOST = 'localhost'
DBPORT = '5432'
ALLOWED_HOSTS = "127.0.0.1, localhost, somedomain.com"

```

in *settings.py*  I have these lines:

```python
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv("SECRET_KEY")

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv("DEBUG")

ALLOWED_HOSTS = [_.strip() for _ in os.getenv("ALLOWED_HOSTS").split(',')]

...

# Database
# https://docs.djangoproject.com/en/3.1/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.getenv("DBNAME"),
        'USER': os.getenv("DBUSER"),
        'PASSWORD': os.getenv("DBPASSWORD"),
        'HOST': os.getenv("DBHOST"),
        'PORT': os.getenv("DBPORT"),

    }
}
```

### Update database and load data

```shell
python manage.py migrate
python manage.py loaddata db.json
```

### Test django 

```shell
sudo ufw allow 8000
```

```shell
python manage.py runserver 0.0.0.0:8000
```

### Test gunicorn

```shell
gunicorn --bind 0.0.0.0:8000 Pizzeria_django.wsgi
```

### Demonization of the gunicorn

```shell
sudo nano /etc/systemd/system/gunicorn.service
```

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/pizzeria/
EnvironmentFile=/root/pizzeria/.env
ExecStart=/root/.local/share/virtualenvs/pizzeria-i1DZOXbP/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/root/pizzeria/run/gunicorn.sock \
          Pizzeria_django.wsgi:application

[Install]
WantedBy=multi-user.target


/root/.local/share/virtualenvs/pizzeria-i1DZOXbP - .venv to find use 'which python' 

sudo nano /etc/systemd/system/gunicorn.socket

#/etc/systemd/system/gunicorn.socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/root/pizzeria/run/gunicorn.sock

[Install]
WantedBy=sockets.target


sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl start gunicorn.socket
sudo systemctl status gunicorn.socket

sudo systemctl restart gunicorn

Nginx
sudo apt install nginx
sudo nano /etc/nginx/sites-available/pizzeria

server {
    listen 80;
    server_name pizza.zolotarev.pp.ua;
}

Welcome to nginx!

server {
    listen 80;
    server_name 10.116.0.2;
    location /static/ {
        root /root/pizzeria/;
        }
        location /media/ {
            root /root/pizzeria/;
        }
        location / {
            include proxy_params;
            proxy_pass http://unix:/root/pizzeria/run/gunicorn.sock;
        }

}

sudo systemctl restart nginx

gunicorn --bind 0.0.0.0:8000 /root/pizzeria/Pizzeria_django.wsgi


namei -l /root/pizzeria/run/gunicorn.sock

sudo apt install certbot python3-certbot-nginx

sudo certbot --nginx -d pizza.zolotarev.pp.ua

https://github.com/PonomaryovVladyslav/PythonCources/blob/master/lesson48.md
https://github.com/PonomaryovVladyslav/PythonCources/blob/master/lesson47.md
https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04-ru
https://the-bosha.ru/2016/06/29/django-delaem-damp-bazy-dannykh-i-vosstanavlivaem-iz-nego-s-dumpdata-i-loaddata/
[https://pizza.zolotarev.pp.ua/](https://pizza.zolotarev.pp.ua/)