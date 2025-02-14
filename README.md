# آموزش نصب RemnaWave در 18 مرحله

بهتر است جهت نصب پنل از سیستم عامل Ubuntu 22.04 استفاده نمایید.

1. ابتدا آپدیت های مورد نیاز سرور را انجام دهید:
```
sudo apt update && apt upgrade -y
```

2. سپس داکر را روی سرور نصب کنید:
```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update
sudo apt install -y docker-ce
docker --version
```

3. فایل تولید کننده JWT را بسازید
```
nano jwtgen.py
```

 محتوای زیر را در کپی نموده و در فایل الصاق کنید:
```
import secrets

# Generate JWT_AUTH_SECRET
jwt_auth_secret = secrets.token_hex(32)
print("JWT_AUTH_SECRET:", jwt_auth_secret)

# Generate JWT_API_TOKENS_SECRET
jwt_api_tokens_secret = secrets.token_hex(32)
print("JWT_API_TOKENS_SECRET:", jwt_api_tokens_secret)
```
سپس با دستور زیر فایل را اجرا کنید تا سکرت های JWT برای شما ساخته شود:
```
python3 jwtgen.py
```
کد های HEX تولید شده را در یک فایل متنی یادداشت کنید (در ادامه نیاز دارید)

5. یک پوشه به نام remnawave بسازید و وارد آن شوید:
```
mkdir remnawave && cd remnawave
```

5. فایل های پروژه را دانلود کنید:
```
curl -o .env https://raw.githubusercontent.com/remnawave/backend/refs/heads/main/.env.sample
```

6. با دستور زیر فایل کانفیگ پروژه را ویرایش کنید:
```
nano .env
```

مواردی که به زبان فارسی نوشته را با مقادیر صحیح جایگزین کنید:
```
### APP ###
APP_PORT=3000

DATABASE_URL="postgresql://postgres:postgres@remnawave-db:5432/postgres"

### JWT ###
### CHANGE DEFAULT VALUES ###
JWT_AUTH_SECRET=کد HEX اولی که یادداشت کردید
JWT_API_TOKENS_SECRET=کد HEX دومی که یادداشت کردید


### TELEGRAM ###
TELEGRAM_BOT_TOKEN=یک ربات از بات فادر بسازید و توکن آن را اینجا وارد کنید
TELEGRAM_ADMIN_ID=آیدی عددی اکانت شما در تلگرام
NODES_NOTIFY_CHAT_ID=آیدی عددی اکانت شما در تلگرام

### FRONT_END ###
FRONT_END_DOMAIN=*

### SUBSCRIPTION ###
SUB_SUPPORT_URL=https://ساب دامینی که رکورد کردید به سرور
SUB_PROFILE_TITLE=Subscription Profile
SUB_UPDATE_INTERVAL=12
SUB_WEBPAGE_URL=https://ساب دامینی که رکورد کردید به سرور

### SUBSCRIPTION PUBLIC DOMAIN ###
### RAW DOMAIN, WITHOUT HTTP/HTTPS, DO NOT PLACE / to end of domain ###
SUB_PUBLIC_DOMAIN=rw.guilanit.com

EXPIRED_USER_REMARKS=["⚠️ Subscription expired","Contact support"]
DISABLED_USER_REMARKS=["❌ Subscription disabled","Contact support"]
LIMITED_USER_REMARKS=["🔴 Subscription limited","Contact support"]

### SUPERADMIN ###
### CHANGE DEFAULT VALUES ###
SUPERADMIN_USERNAME=نام کاربری مدیریت پنل
SUPERADMIN_PASSWORD=رمزعبور مدیریت پنل

### SWAGGER ###
SWAGGER_PATH=/docs
SCALAR_PATH=/scalar
IS_DOCS_ENABLED=true

### PROMETHEUS ###
METRICS_USER=admin
METRICS_PASS=admin

### WEBHOOK ###
WEBHOOK_ENABLED=true
### Only https:// is allowed
WEBHOOK_URL=https://webhook.site/1234567890
### This secret is used to sign the webhook payload, must be exact 64 characters. Only a-z, 0-9, A-Z are allowed.
WEBHOOK_SECRET_HEADER=vsmu67Kmg6R8FjIOF1WUY8LWBHie4scdEqrfsKmyf4IAf8dY3nFS0wwYHkhh6ZvQ

### CLOUDFLARE ###
# USED ONLY FOR docker-compose-prod-with-cf.yml
# NOT USED BY THE APP ITSELF
CLOUDFLARE_TOKEN=ey...

### Database ###
### For Postgres Docker container ###
# NOT USED BY THE APP ITSELF
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=postgres
```
فایل را ذخیره کنید و از آن خارج شوید.

7. فایل داکر را ایجاد کنید:
```
nano docker-compose.yml
```

محتوای زیر را کپی و در فایل الصاق کنید سپس فایل را ذخیره کنید.
```
services:
    remnawave-db:
        image: postgres:17
        container_name: 'remnawave-db'
        hostname: remnawave-db
        restart: always
        env_file:
            - .env
        environment:
            - POSTGRES_USER=${POSTGRES_USER}
            - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
            - POSTGRES_DB=${POSTGRES_DB}
            - TZ=UTC
        ports:
            - '127.0.0.1:6767:5432'
        volumes:
            - remnawave-db-data:/var/lib/postgresql/data
        networks:
            - remnawave-network
        healthcheck:
            test: ['CMD-SHELL', 'pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}']
            interval: 3s
            timeout: 10s
            retries: 3

    remnawave:
        image: remnawave/backend:latest
        container_name: 'remnawave'
        hostname: remnawave
        restart: always
        ports:
            - '127.0.0.1:3000:3000'
        env_file:
            - .env
        networks:
            - remnawave-network

networks:
    remnawave-network:
        name: remnawave-network
        driver: bridge
        external: false

volumes:
    remnawave-db-data:
        driver: local
        external: false
        name: remnawave-db-data
```

8. دستور زیر را اجرا کنید:
```
docker compose up -d
```

9. با اجرای دستور زیر لاگ را بررسی کنید:
```
docker compose logs -f
```

تا اینجا پنل به صورت ssh portforwarding نصب شده و از مسیر http://127.0.0.1:3000 در دسترس است. برای اجرای پنل به صورت پابلیک با استفاده از دامنه نیاز است گواهی SSL دریافت کنید.
10. جهت نصب پیشنیاز های دریافت SSL دستور زیر را اجرا نمایید:
```
sudo apt-get install cron socat
```

11. جهت نصب acme.sh دستور زیر را اجرا نمایید (نکته: آدرس ایمیل خود را جایگزین example@gmail.com نمایید):
```
curl https://get.acme.sh | sh -s email=example@gmail.com && source ~/.bashrc

```

12. جهت ساخت پوشه برای گواهی ها دستور زیر را اجرا نمایید:
```
mkdir -p ~/remnawave/nginx && cd ~/remnawave/nginx
```

13. جهت درخواست گواهی SSL دستور زیر را اجرا نمایید( نکته: آدرس دامنه خود را جایگزین sub.domain.ir نمایید):
```
acme.sh --issue --standalone -d 'sub.domain.ir' --key-file ~/remnawave/nginx/privkey.key --fullchain-file ~/remnawave/nginx/fullchain.pem --alpn --tlsport 8443
```

14. با اجرای دستور زیر یک پوشه و فایل کانفیگ برای Nginx بسازید:
```
cd ~/remnawave/nginx && nano nginx.conf
```

15. محتوای زیر را به صورت کامل کپی کرده و در فایل فوق الصاق نمایید (نکته: آدرس دامنه خود را جایگزین sub.domain.ir نمایید):
```
user                 nginx;
pid                  /var/run/nginx.pid;
worker_processes     auto;
worker_rlimit_nofile 65535;

# Load modules
include              /etc/nginx/modules-enabled/*.conf;

events {
    multi_accept       on;
    worker_connections 65535;
}

http {
    charset                utf-8;
    sendfile               on;
    tcp_nopush             on;
    tcp_nodelay            on;
    server_tokens          off;
    log_not_found          off;
    types_hash_max_size    2048;
    types_hash_bucket_size 64;
    client_max_body_size   16M;

    # MIME
    include                mime.types;
    default_type           application/octet-stream;

    # Logging
    access_log             off;
    error_log              /dev/null;

    # SSL
    ssl_session_timeout    1d;
    ssl_session_cache      shared:SSL:10m;
    ssl_session_tickets    off;

    # Mozilla Intermediate configuration
    ssl_protocols          TLSv1.2 TLSv1.3;
    ssl_ciphers            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # OCSP Stapling
    ssl_stapling           on;
    ssl_stapling_verify    on;
    resolver               1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 valid=60s;
    resolver_timeout       2s;

    # Connection header for WebSocket reverse proxy
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    map $remote_addr $proxy_forwarded_elem {

        # IPv4 addresses can be sent as-is
        ~^[0-9.]+$        "for=$remote_addr";

        # IPv6 addresses need to be bracketed and quoted
        ~^[0-9A-Fa-f:.]+$ "for=\"[$remote_addr]\"";

        # Unix domain socket names cannot be represented in RFC 7239 syntax
        default           "for=unknown";
    }

    map $http_forwarded $proxy_add_forwarded {

        # If the incoming Forwarded header is syntactically valid, append to it
        "~^(,[ \\t]*)*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*([ \\t]*,([ \\t]*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*)?)*$" "$http_forwarded, $proxy_forwarded_elem";

        # Otherwise, replace it
        default "$proxy_forwarded_elem";
    }

    # Load configs
    include /etc/nginx/conf.d/*.conf;

    # your domain
    server {
        listen                               443 ssl  reuseport;
        listen                               [::]:443 ssl reuseport;
        http2                                on;
        server_name                          sub.domain.ir;

        # SSL
        ssl_certificate                      /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key                  /etc/nginx/ssl/privkey.key;

        # security headers
        add_header X-XSS-Protection          "1; mode=block" always;
        add_header X-Content-Type-Options    "nosniff" always;
        add_header Referrer-Policy           "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy   "default-src 'self' http: https: ws: wss: data: blob: 'unsafe-inline'; frame-ancestors 'self';" always;
        add_header Permissions-Policy        "interest-cohort=()" always;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # . files
        location ~ /\.(?!well-known) {
            deny all;
        }

        # logging
        access_log /var/log/nginx/access.log combined buffer=512k flush=1m;
        error_log  /var/log/nginx/error.log warn;

        # reverse proxy
        location / {
            proxy_pass                         http://127.0.0.1:3000;
            proxy_set_header Host              $host;
            proxy_http_version                 1.1;
            proxy_cache_bypass                 $http_upgrade;

            # Proxy SSL
            proxy_ssl_server_name              on;

            # Proxy headers
            proxy_set_header Upgrade           $http_upgrade;
            proxy_set_header Connection        $connection_upgrade;
            proxy_set_header X-Real-IP         $remote_addr;
            proxy_set_header Forwarded         $proxy_add_forwarded;
            proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host  $host;
            proxy_set_header X-Forwarded-Port  $server_port;

            # Proxy timeouts
            proxy_connect_timeout              60s;
            proxy_send_timeout                 60s;
            proxy_read_timeout                 60s;
        }

        # favicon.ico
        location = /favicon.ico {
            log_not_found off;
        }

        # robots.txt
        location = /robots.txt {
            log_not_found off;
        }

        # gzip
        gzip            on;
        gzip_vary       on;
        gzip_proxied    any;
        gzip_comp_level 6;
        gzip_types      text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;
    }




}
```

فایل را ذخیره کرده، از آن خارج شوید و مراحل را ادامه دهید.

16. نیاز است با اجرای دستور زیر یک فایل داکر دیگر ایجاد کنید:
```
cd ~/remnawave/nginx && nano docker-compose.yml
```

17. محتوای زیر را کپی نموده و در فایل ایجاد شده الصاق کنید:
```
services:
    remnawave-nginx:
        image: nginx:latest
        container_name: remnawave-nginx
        hostname: remnawave-nginx
        volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf:ro
            - ./fullchain.pem:/etc/nginx/ssl/fullchain.pem:ro
            - ./privkey.key:/etc/nginx/ssl/privkey.key:ro
        restart: always
        network_mode: host
```

18. با دستور زیر فایل داکر را اجرا نمایید:
```
docker compose up -d && docker compose logs -f -t
```


تمام شد! حالا می توانید از نشانی sub.domain.ir به پنل خود دسترسی داشته باشید.
(نکته: sub.domain.ir صرفا یک مثال می باشد، با آدرس دامنه خود تغییر دهید)

موفق باشید.
