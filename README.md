<<<<<<< HEAD
=======
# Docker ELK Stack

این پروژه برای اجرای مجموعه ELK با استفاده از Docker Compose ساخته شده است.

اجزای پروژه:

- **Elasticsearch** برای ذخیره‌سازی و جست‌وجوی داده‌ها
- **Logstash** برای دریافت، پردازش و ارسال لاگ‌ها
- **Kibana** برای مشاهده، جست‌وجو و ساخت داشبورد
- Docker Compose برای مدیریت سرویس‌ها

---

## ساختار پروژه

```text
.
├── docker-compose.yml
├── .env.example
├── .gitignore
├── README.md
├── elasticsearch
│   ├── Dockerfile
│   └── config
│       └── elasticsearch.yml
├── kibana
│   ├── Dockerfile
│   └── config
│       └── kibana.yml
└── logstash
    ├── Dockerfile
    ├── config
    │   └── logstash.yml
    └── pipeline
        └── logstash.conf
```

---

## پیش‌نیازها

قبل از اجرا، موارد زیر باید روی سرور نصب باشند:

- Docker Engine
- Docker Compose Plugin
- Git

بررسی نسخه‌ها:

```bash
docker --version
docker compose version
git --version
```

---

## دریافت پروژه

```bash
git clone git@github.com:Parvanak/docker-elk-stack.git
cd docker-elk-stack
```

در صورتی که پروژه از قبل روی سرور موجود است:

```bash
cd /opt
```

---

## تنظیم متغیرهای محیطی

فایل نمونه Environment را کپی کنید:

```bash
cp .env.example .env
```

سپس فایل `.env` را ویرایش کنید:

```bash
nano .env
```

نمونه محتوا:

```dotenv
ELK_VERSION=7.10.1
ELASTIC_PASSWORD=CHANGE_THIS_PASSWORD
KIBANA_SYSTEM_PASSWORD=CHANGE_THIS_PASSWORD
```

> فایل `.env` شامل اطلاعات حساس است و نباید در Git Commit شود.

برای محدودکردن دسترسی به فایل:

```bash
chmod 600 .env
```

---

## بررسی تنظیمات Docker Compose

قبل از اجرا، ساختار Compose را بررسی کنید:

```bash
docker compose config
```

توجه داشته باشید که خروجی این دستور ممکن است مقادیر Secretها را نمایش دهد. خروجی آن را در محیط عمومی منتشر نکنید.

---

## ساخت و اجرای سرویس‌ها

ساخت Imageها:

```bash
docker compose build
```

اجرای سرویس‌ها:

```bash
docker compose up -d
```

بررسی وضعیت:

```bash
docker compose ps
```

خروجی مورد انتظار:

```text
elasticsearch   Up
kibana          Up
logstash        Up
```

---

## دسترسی به سرویس‌ها

### Kibana

```text
http://SERVER-IP:5601
```

نام کاربری اولیه:

```text
elastic
```

پسورد از فایل `.env` خوانده می‌شود:

```text
ELASTIC_PASSWORD
```

### Logstash TCP Input

```text
SERVER-IP:5000
```

### Elasticsearch

در تنظیم فعلی، پورت Elasticsearch فقط داخل شبکه Docker قابل دسترسی است:

```text
http://elasticsearch:9200
```

این حالت از نظر امنیتی بهتر از انتشار مستقیم پورت 9200 روی Host است.

---

## مشاهده لاگ‌ها

مشاهده لاگ تمام سرویس‌ها:

```bash
docker compose logs -f
```

لاگ Elasticsearch:

```bash
docker compose logs -f elasticsearch
```

لاگ Kibana:

```bash
docker compose logs -f kibana
```

لاگ Logstash:

```bash
docker compose logs -f logstash
```

مشاهده 100 خط آخر:

```bash
docker compose logs --tail=100 kibana
```

---

## توقف سرویس‌ها

توقف کانتینرها بدون حذف آن‌ها:

```bash
docker compose stop
```

اجرای مجدد:

```bash
docker compose start
```

توقف و حذف کانتینرها:

```bash
docker compose down
```

توقف و حذف کانتینرها به همراه Volumeها:

```bash
docker compose down -v
```

> اجرای `docker compose down -v` داده‌های Elasticsearch را حذف می‌کند.

---

## راه‌اندازی مجدد یک سرویس

راه‌اندازی مجدد Kibana:

```bash
docker compose restart kibana
```

Recreate کردن Kibana پس از تغییر Environment:

```bash
docker compose up -d --force-recreate kibana
```

Recreate کردن تمام سرویس‌ها:

```bash
docker compose up -d --force-recreate
```

---

## بررسی سلامت سرویس‌ها

بررسی وضعیت کانتینرها:

```bash
docker compose ps
```

بررسی پورت‌ها:

```bash
ss -lntp | grep -E '5000|5601|9200'
```

تست Kibana:

```bash
curl -I http://127.0.0.1:5601
```

تست Elasticsearch از داخل کانتینر:

```bash
docker exec elasticsearch curl \
  -u elastic:"${ELASTIC_PASSWORD}" \
  http://localhost:9200/_cluster/health?pretty
```

در صورتی که متغیر Shell تعریف نشده است:

```bash
docker exec -it elasticsearch bash
```

سپس داخل کانتینر:

```bash
curl -u elastic:'YOUR_PASSWORD' \
  http://localhost:9200/_cluster/health?pretty
```

---

## شبکه‌های Docker

این پروژه از دو شبکه استفاده می‌کند:

- `app_net` برای ارتباط داخلی سرویس‌های ELK
- `web_net` به‌عنوان شبکه خارجی

بررسی شبکه‌ها:

```bash
docker network ls
```

بررسی شبکه خارجی:

```bash
docker network inspect web_net
```

اگر شبکه `web_net` وجود نداشت:

```bash
docker network create web_net
```

---

## Volume مربوط به Elasticsearch

داده‌های Elasticsearch داخل Volume زیر نگه‌داری می‌شوند:

```text
elastic_data
```

بررسی Volume:

```bash
docker volume inspect elastic_data
```

لیست Volumeها:

```bash
docker volume ls
```

از حذف این Volume بدون Backup خودداری کنید.

---

## فایل Pipeline مربوط به Logstash

فایل Pipeline باید در مسیر زیر قرار داشته باشد:

```text
logstash/pipeline/logstash.conf
```

نمونه ساده:

```conf
input {
  tcp {
    port => 5000
    codec => json_lines
  }
}

output {
  stdout {
    codec => rubydebug
  }

  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

پس از تغییر Pipeline:

```bash
docker compose restart logstash
```

مشاهده خطاهای Pipeline:

```bash
docker compose logs --tail=200 logstash
```

---

## تنظیمات امنیتی Kibana

برای محیط عملیاتی، بهتر است Kibana به‌جای کاربر `elastic` از کاربر داخلی زیر استفاده کند:

```text
kibana_system
```

همچنین باید Encryption Keyهای ثابت در `kibana.yml` تعریف شوند:

```yaml
xpack.security.encryptionKey: "REPLACE_WITH_RANDOM_VALUE"
xpack.encryptedSavedObjects.encryptionKey: "REPLACE_WITH_RANDOM_VALUE"
xpack.reporting.encryptionKey: "REPLACE_WITH_RANDOM_VALUE"
```

ساخت کلید تصادفی:

```bash
openssl rand -base64 48
```

برای هر تنظیم یک کلید جداگانه تولید کنید.

---

## نکات امنیتی

موارد زیر نباید در GitHub قرار بگیرند:

- فایل `.env`
- Passwordها
- API Tokenها
- Private Keyها
- فایل‌های `.p12`
- فایل‌های `.pfx`
- فایل‌های `.jks`
- فایل‌های `.keystore`
- Certificateهای دارای Private Key

قبل از Commit بررسی کنید:

```bash
git status
git diff
```

بررسی فایل‌هایی که قرار است Commit شوند:

```bash
git diff --cached --name-only
```

فایل `.env` نباید در لیست قرار داشته باشد.

---

## اعمال تغییرات در Git

پس از تغییر فایل‌ها:

```bash
git status
git diff
git add .
git commit -m "Describe the change"
git push
```

نمونه:

```bash
git add kibana/config/kibana.yml
git commit -m "Configure Kibana encryption keys"
git push
```

---

## مشکلات متداول

### Kibana در حالت Restarting است

لاگ‌ها را بررسی کنید:

```bash
docker compose logs --tail=200 kibana
```

یکی از دلایل متداول، تعریف متغیر در `kibana.yml` و نبود آن داخل Environment کانتینر است.

بررسی متغیرها:

```bash
docker inspect kibana \
  --format '{{range .Config.Env}}{{println .}}{{end}}' |
grep -E 'ELASTIC|KIBANA'
```

### خطای Mount فایل تنظیمات

بررسی کنید مسیر روی Host واقعاً فایل باشد، نه Directory:

```bash
file kibana/config/kibana.yml
file elasticsearch/config/elasticsearch.yml
file logstash/config/logstash.yml
```

### خطای شبکه `web_net`

اگر شبکه از قبل وجود دارد، باید در Compose به‌صورت external تعریف شود:

```yaml
networks:
  web_net:
    name: web_net
    external: true
```

### خطای اتصال به Registry Elastic

تست اتصال:

```bash
curl -Iv https://docker.elastic.co/v2/
```

اگر اتصال Timeout شد، دسترسی خروجی TCP/443 به دامنه زیر باید در Firewall بررسی شود:

```text
docker.elastic.co
```

---

## هشدار حذف داده

دستورات زیر ممکن است داده‌های Elasticsearch را حذف کنند:

```bash
docker compose down -v
docker volume rm elastic_data
docker system prune --volumes
```

قبل از اجرای این دستورات از اطلاعات مهم Backup تهیه کنید.

---

## License

این Repository صرفاً برای نگه‌داری تنظیمات Docker Compose و فایل‌های Configuration است.

شرایط استفاده از Elasticsearch، Kibana و Logstash تابع License رسمی Elastic است.
>>>>>>> 7495e79 (Add project documentation)
