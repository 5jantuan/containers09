## Лабораторная работа: Оптимизация Docker-образов

### Цель работы
Познакомиться с различными методами оптимизации Docker-образов для уменьшения их размера и повышения эффективности.

---

### Задание
Сравнить эффективность следующих методов оптимизации Docker-образов:

1. Удаление неиспользуемых зависимостей и временных файлов  
2. Уменьшение количества слоев  
3. Использование минимального базового образа  
4. Перепаковка образа  
5. Комбинация всех методов  

---

### Выполнение

#### Подготовка
- Установлен Docker
- Создан репозиторий `containers09`
- В папке `containers09` создана директория `site/`, содержащая HTML, CSS и JS файлы

#### Базовый образ `mynginx:raw`
**Dockerfile.raw:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Сборка:
```bash
docker image build -t mynginx:raw -f Dockerfile.raw .
```

---

#### Удаление временных файлов и кэша: `mynginx:clean`
**Dockerfile.clean:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y
RUN apt-get install -y nginx
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Сборка:
```bash
docker image build -t mynginx:clean -f Dockerfile.clean .
```

---

#### Уменьшение количества слоев: `mynginx:few`
**Dockerfile.few:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y nginx && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Сборка:
```bash
docker image build -t mynginx:few -f Dockerfile.few .
```

---

#### Минимальный базовый образ: `mynginx:alpine`
**Dockerfile.alpine:**
```dockerfile
FROM alpine:latest
RUN apk update && apk upgrade
RUN apk add nginx
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Сборка:
```bash
docker image build -t mynginx:alpine -f Dockerfile.alpine .
```

---

#### Перепаковка образа: `mynginx:repack`
```bash
docker container create --name mynginx mynginx:raw
docker container export mynginx | docker image import - mynginx:repack
docker container rm mynginx
```

---

#### Использование всех методов: `mynginx:min`
**Dockerfile.min:**
```dockerfile
FROM alpine:latest
RUN apk update && apk upgrade && \
    apk add nginx && \
    rm -rf /var/cache/apk/*
COPY site /var/www/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Сборка и перепаковка:
```bash
docker image build -t mynginx:minx -f Dockerfile.min .
docker container create --name mynginx mynginx:minx
docker container export mynginx | docker image import - mynginx:min
docker container rm mynginx
```

---

### Сравнение размеров образов

```bash
docker image list
```
REPOSITORY              TAG           IMAGE ID       CREATED              SIZE
myngin                  min           53ba4f0baaca   About a minute ago   2.99MB
mynginx                 minx          edb91b0b5cb5   About a minute ago   14.4MB
mynginx                 alpine        f241af03536a   9 minutes ago        19.5MB
mynginx                 few           db8eb06fc20c   12 minutes ago       130MB
mynginx                 clean         bad5c3ea20a6   14 minutes ago       211MB
mynginx                 raw           64c37554291a   16 minutes ago       211MB

---

### Ответы на вопросы

#### 1. Какой метод оптимизации образов вы считаете наиболее эффективным?
**Наиболее эффективным** является **использование минимального базового образа (например, Alpine)** в сочетании с **удалением временных файлов и перепаковкой**. Это позволяет максимально уменьшить размер образа без потери функциональности.

#### 2. Почему очистка кэша пакетов в отдельном слое не уменьшает размер образа?
Docker сохраняет каждый слой как дельту, и даже если в следующем слое вы удаляете файлы, они всё ещё присутствуют в предыдущем слое. Только объединение операций в один `RUN` позволяет навсегда удалить лишнее и сократить итоговый размер.

#### 3. Что такое перепаковка образа?
Перепаковка — это процесс, при котором контейнер создается и экспортируется как tar-архив, а затем импортируется обратно в Docker как новый образ. Это **удаляет историю слоев**, сжимает файловую систему и может сократить размер за счёт упрощения структуры образа.

---

### Выводы
- Оптимизация Docker-образов помогает **сократить занимаемое место**, **ускорить развертывание** и **повысить безопасность**.
- Использование **Alpine** даёт наилучшие результаты по размеру.
- Важно **объединять команды RUN** и **удалять временные файлы в том же слое**.
- Перепаковка может быть полезна как финальный шаг для сжатия образа.

---
