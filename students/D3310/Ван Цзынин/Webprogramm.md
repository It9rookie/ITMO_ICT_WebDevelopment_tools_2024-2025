# Лабораторная работа 1. Реализация серверного приложения FastAPI.

Необходимо реализовать полноценное серверное приложение с помощью фреймворка FastAPI с применением дополнительных средств и библиотек.

## pra1

Создание конечных точек api и виртуальных баз данных

![image-20250609000107020](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000107020.png)



## pra2

Настройка БД, SQLModel и миграции через Alembic

![image-20250609000210684](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000210684.png)![image-20250609000214710](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000214710.png)![image-20250609000220703](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000220703.png)

## pra3

Миграции, ENV, GitIgnore и структура проекта

![image-20250609000601012](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000601012.png)

## Лабораторная работа 1

![image-20250609000755467](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000755467.png)![image-20250609000825174](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000825174.png)![image-20250609000836424](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000836424.png)![image-20250609000845312](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000845312.png)![image-20250609000859044](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000859044.png)![image-20250609000907285](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609000907285.png)

# Лабораторная работа 2. Потоки. Процессы. Асинхронность.

Цель работы: понять отличия потоками и процессами и понять, что такое ассинхронность в Python.

Работа о потоках, процессах и асинхронности поможет студентам развить навыки создания эффективных и быстродействующих программ, что важно для работы с большими объемами данных и выполнения вычислений.

## Задача 1. Различия между threading, multiprocessing и async в Python

Задача: Напишите три различных программы на Python, использующие каждый из подходов: threading, multiprocessing и async. Каждая программа должна решать считать сумму всех чисел от 1 до 10000000000000. Разделите вычисления на несколько параллельных задач для ускорения выполнения.

**threading.py:**

![image-20250609001245890](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001245890.png)

**Multiprocessing — Процессы，multiprocessing.py:**

![image-20250609001240135](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001240135.png)

**Async - Асинхронность，async.py:**

![image-20250609001235414](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001235414.png)

## Задача 2. Параллельный парсинг веб-страниц с сохранением в базу данных

Задача: Напишите программу на Python для параллельного парсинга нескольких веб-страниц с сохранением данных в базу данных с использованием подходов threading, multiprocessing и async. Каждая программа должна парсить информацию с нескольких веб-сайтов, сохранять их в базу данных.

**threading.py:**

![image-20250609001348943](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001348943.png)

**Multiprocessing — Процессы，multiprocessing.py:**

![image-20250609001355143](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001355143.png)

**Async - Асинхронность，async.py:**

![image-20250609001403347](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609001403347.png)

# Лабораторная работа 3: Упаковка FastAPI приложения в Docker, Работа с источниками данных и Очереди

Цель: Научиться упаковывать FastAPI приложение в Docker, интегрировать парсер данных с базой данных и вызывать парсер через API и очередь.

![image-20250609002315421](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002315421.png)

**код из файла Dockerfile:**

```python
FROM python:3.9-slim
WORKDIR /app
COPY Requirements.txt .
RUN pip install --no-cache-dir -r Requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**код из файла Docker-compose.yml**

```python
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Docker/Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=mysql+pymysql://root:wzn001021@db/finance_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db 
      - redis

  worker:
    build:
      context: .
      dockerfile: Docker/Dockerfile
    command: celery -A app.celery_config.celery_app worker --loglevel=info
    environment:
      - DATABASE_URL=mysql+pymysql://root:wzn001021@db/finance_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=wzn001021 
      - MYSQL_DATABASE=finance_db  
    volumes:
      - mysql_data:/var/lib/mysql 

  redis:
    image: redis:7.0-alpine
    ports:
      - "6379:6379"

volumes:
  mysql_data: 
```

**код из файла celery_config.py:**

```python
from celery import Celery
from os import getenv

celery_app = Celery(
    "lab3_worker",
    broker=getenv("REDIS_URL", "redis://localhost:6379/0"),  
    backend=getenv("REDIS_URL", "redis://localhost:6379/0"),
    include=["app.routers.tasks"]  
)

celery_app.conf.timezone = "UTC"
celery_app.conf.task_serializer = "json"
```

**Запуск бэкенда celery и клиента FASTApi：**

![image-20250609002425695](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002425695.png)![image-20250609002428935](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002428935.png)

**Выполните функциональное тестирование：**

![image-20250609002507556](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002507556.png)![image-20250609002511289](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002511289.png)

**База данных бэк-офиса：**

![image-20250609002542945](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002542945.png)

**Терминал выполнения и сельдерейный дисплей：**

![image-20250609002705210](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002705210.png)![image-20250609002709441](/Users/not_hungry_king/Library/Application Support/typora-user-images/image-20250609002709441.png)