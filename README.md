# flask-devops
how to deploy flask web app to gcp + domain [UNFINISHED]

# stack

- flask as web app framework

- virtualenv create new python environment

- nginx as proxy server

- gunicorn as wsgi server

- supervisord as task runner

- gcp as cloud

# goal

the goal is to create a web that accesible using domain name, running on GCP instance.
the web app is created with Flask (micro)framework. we're using gunicorn as wsgi server on port 8000 and proxy the connection using nginx on port 80. and the gunicorn process is watched by supervisord. 

let say, we have this script as our **awesomewebappproject** 

server.py
```
from flask import Flask

app = Flask()

@app.route("/")
def index_view():
   return "hello world"
```

# setup

- install all the stack

```
sudo apt install nginx
```

- install pip3/pip 

```
sudo apt install python-pip3
```

- install virtualenv, so we can create new python env 

```
sudo pip3 install virtualenv
```

- create new python env

```
virtualenv env 
```

- activate new env

```
source env/bin/activate
```

- install app related dependencies (don't forget to install flask!)

- install supervisord (use python2.7)

first, we need to install the python2.7 pip

```
sudo apt install python-pip
```

then install supervisord

```
pip install supervisord
```

nuff with the installation part, time to config the proxy server, nginx!

- remove the default nginx config

```
sudo rm /etc/nginx/sites-enabled/default
```

- create new config on `/etc/nginx/sites-enabled/my_awesome_app`

```
server {
    # listen on port 443 (https)
    listen 80;
    server_name myawesomeproject.org;

    # location of the self-signed SSL certificate
    #ssl_certificate /home/dasta/prfnational/certs/cert.pem;
    #ssl_certificate_key /home/dasta/prfnational/certs/key.pem;

    # write access and error logs to /var/log
    access_log /var/log/myawesomeaccess.log;
    error_log /var/log/myawesomeerror.log;
    
    gzip on;
	 gzip_disable "msie6";
	 gzip_vary on;
	 gzip_proxied any;
	 gzip_comp_level 6;
	 gzip_buffers 16 8k;
	 gzip_http_version 1.1;
	 gzip_types application/javascript application/rss+xml application/vnd.ms-fontobject application/x-font application/x-font-opentype application/x-font-otf application/x-font-truetype application/x-font-ttf application/x-javascript application/xhtml+xml application/xml font/opentype font/otf font/ttf image/svg+xml image/x-icon text/css text/javascript text/plain text/xml;

    location / {
        # forward application requests to the gunicorn server
        proxy_pass http://localhost:8000;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        # handle static files directly, without forwarding to the application
        alias /home/dasta/prfnational/static;
        expires 30d;
    }
}

```

- supervisord config

put this config under `/etc/supervisor/conf.d/oa.conf

```
[program:oa]
command=/home/vicky/oa/env/bin/gunicorn -b localhost:8000 -w 3 server:app
directory=/home/vicky/oa
user=vicky
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
```

- then run `sudo supervisorctl update all`


