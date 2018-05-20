[![English](https://thumb.ibb.co/jDrVkd/gb.png)](README.md) [![Russian](https://thumb.ibb.co/cjYMrJ/ru.png)](README.ru.md)

# utransnet-faucet
Python-based faucet for Transnet

## Установка

* edit `config.py` and provide private keys and settings
* `python manage.py install`

## Usage

* `python manage.py runserver`

The faucet is then available at URL `http://localhost:5000`

## Nginx configuration

Run `uwsgi --ini wsgi.ini`

and use a configuration similar tothis

```
user transnet;
worker_processes  4;

events {
    worker_connections  2048;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    access_log  /www/logs/access.log;
    error_log  /www/logs/error.log;
    log_not_found off;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;

    upstream websockets {
      server localhost:9090;
      server localhost:9091;
    }

    server {
        listen       80;
        if ($scheme != "https") {
                return 301 https://$host$request_uri;
        }

        listen       443 ssl;
        server_name  transnet-wallet.com;
        ssl_certificate      /etc/nginx/ssl/your-cert.crt;
        ssl_certificate_key /etc/nginx/ssl/your-cert.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location ~ /ws/? {
            access_log on;
            proxy_pass http://websockets;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_next_upstream     error timeout invalid_header http_500;
            proxy_connect_timeout   2;
        }
        location ~ ^/[\w\d\.-]+\.(js|css|dat|png|json)$ {
            root /www/wallet;
            try_files $uri /wallet$uri =404;
        }
        location / {
            root /www/wallet;
        }
        location /api {
                include uwsgi_params;
                uwsgi_pass unix:/tmp/faucet.sock;
        }

    }
}
```

## Связанные проекты
- [python-utransnet](https://github.com/u-transnet/python-utransnet)

## Сотрудничество
Мы будем рады вашей помощи в развитии проекта! Откройте [CONTRIBUTING.ru.md](CONTRIBUTING.ru.md) для того, чтобы узнать чем Вы можете поможете помочь проекту и как присоединиться

## Лицензия
Проект использует MIT лицензию. Откройте [LICENSE](LICENSE) для подробностей

