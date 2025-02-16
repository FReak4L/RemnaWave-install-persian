# راهنمای نصب Node ها

پس از نصب پنل اصلی، یک سرور مجازی دیگر با لوکیشن دلخواه ایجاد کنید.

ابتدا با دستور زیر بروزرسانی های سرور را دریافت و اعمال کنید:
```
sudo apt update && sudo apt upgrade -y
```

با دستور زیر یک پوشه برای Node و یک فایل داکر ایجاد نمایید:
```
mkdir /remnanode && cd remnanode && nano docker-compose.yml
```

محتوای <a href="https://raw.githubusercontent.com/remnawave/node/refs/heads/main/docker-compose-prod.yml">این فایل</a> را کپی و در فایل داکری که ایجاد شد الصاق کنید: (در تاریخ 28 بهمن 1403 محتوای فایل به شرح زیر است، به علت وجود احتمال در تغییر محتوا، محتوای فایل را به صورت بروز از <a href="https://raw.githubusercontent.com/remnawave/node/refs/heads/main/docker-compose-prod.yml">لینک فایل ارائه شده توسط توسعه دهنده</a> دریافت کنید.)
```
services:
  remnanode:
    container_name: remnanode
    hostname: remnanode
    image: remnawave/node:latest
    env_file:
      - .env
    network_mode: host
```

دستور زیر را برای ورود به فایل کانفیگ env وارد نمایید:
```
nano .env
```
وارد پنل شوید و Node را اضافه کنید؛ روی گزینه Important information کلیک کنید. کلید را کپی و در فایل env خود الصاق کنید:
```
### APP ###
APP_PORT=2222
```

برای اجرای فایل داکر و فعالسازی Node از دستور زیر استفاده کنید:
```
docker compose up -d && docker compose logs -f
```

موفق باشید.
