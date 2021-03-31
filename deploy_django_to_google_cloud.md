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
sudo apt install python3-pip python3-dev python3-pip libpq-dev postgresql postgresql-contrib nginx certbot python3-certbot-nginx curl
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
sudo nano /etc/systemd/system/gunicorn.socket
```

```shell
#/etc/systemd/system/gunicorn.socket

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/home/user/some_project/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

The `Unit` block is responsible for describing the demon

The `Socket` block is responsible for where the socket file will be located, `some_project` in this case is the name of the folder with the project, `run` is the name of the folder with the socket file (for some reason it is customary to call this)

`Install` block is responsible for automatic start at system startup



```shell
sudo nano /etc/systemd/system/gunicorn.service
```

```shell
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/home/user/some_project/
EnvironmentFile=/home/user/some_project/.env
ExecStart=/home/user/.local/share/virtualenvs/some_project-zzzxxx333/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/home/user/some_project/run/gunicorn.sock \
          some_project.wsgi:application

[Install]
WantedBy=multi-user.target
```

`/home/user/.local/share/virtualenvs/some_project-zzzxxx333/` - .venv directory. Use `which python` for find them

`some_project.wsgi` '*some_project*' is the folder of basic application



```shell
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl start gunicorn.socket
```

test

```shell
sudo systemctl status gunicorn.socket

sudo systemctl restart gunicorn
gunicorn --bind 0.0.0.0:8000 /home/user/some_project/some_project.wsgi
```



### Nginx  config

```shell
sudo nano /etc/nginx/sites-available/some_project
```

```nginx
server {
    listen 80;
    server_name 10.110.0.10;
}
```

`10.110.0.10` - is a intro network instance ip

```shell
sudo systemctl restart nginx
```



> Welcome to nginx!

```shell
sudo nano /etc/nginx/sites-available/some_project
```

```nginx
server {
    listen 80;
    server_name somedomain.com;
    location /static/ {
        root /home/user/some_project/;
        }
        location /media/ {
            root /home/user/some_project/;
        }
        location / {
            include proxy_params;
            proxy_pass http://unix://home/user/some_project/run/gunicorn.sock;
        }

}
```

```shell
sudo systemctl restart gunicorn

sudo systemctl restart nginx
```

Setup certbot for our domain

```shell
sudo certbot --nginx -d somedomein.com
```
[Link text](URL)
