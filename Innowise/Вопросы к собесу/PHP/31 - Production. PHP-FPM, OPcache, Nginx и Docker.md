---
custom-width: 80
---
# 31 — Production. PHP-FPM, OPcache, Nginx и Docker

## 1. Теория

### Связка Nginx + PHP-FPM
1. **Nginx**: Принимает входящий HTTP-запрос, статические файлы отдает напрямую, а динамические запросы `.php` перенаправляет в PHP-FPM по протоколу FastCGI (через TCP-сокет или Unix-сокет).
2. **PHP-FPM**: Управляет пулом процессов-воркеров (`pm = dynamic` или `pm = static`), которые исполняют PHP-код.

### Docker-контейнеризация
- Разделение сервисов на контейнеры (`nginx`, `php-fpm`, `postgres`/`mysql`, `redis`).
- Оптимизация образов (production build без лишних инструментов разработки).

---

## 2. Примеры кода

```nginx
# Конфигурация Nginx для проксирования в PHP-FPM
server {
    listen 80;
    server_name example.com;
    root /var/www/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php-fpm:9000; # Контейнер PHP-FPM
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

---

## 3. Вопросы на собеседовании

1. **В чем разница между Unix-сокетом и TCP-сокетом при связи Nginx и PHP-FPM?**
   *Ответ*: Unix-сокет работает через файл файловой системы, что быстрее на одном сервере, так как исключает накладные расходы сетевого стека TCP/IP. TCP-сокет (`127.0.0.1:9000`) необходим, если Nginx и PHP-FPM запущены на разных физических серверах или в разных хостах Docker.
2. **Как работает директива `try_files $uri $uri/ /index.php?$query_string;` в Nginx?**
   *Ответ*: Она проверяет существование запрошенного файла (`$uri`), затем папки (`$uri/`), а если ничего не найдено, перенаправляет весь трафик на единую точку входа фреймворка (`/index.php`), что является основой Front Controller паттерна.
3. **Какие ключевые настройки OPcache важны для production?**
   *Ответ*: `opcache.enable=1`, `opcache.memory_consumption=256`, `opcache.max_accelerated_files=20000`, и главное — `opcache.validate_timestamps=0` (для максимальной производительности на production, требующая сброса кеша при деплое).
