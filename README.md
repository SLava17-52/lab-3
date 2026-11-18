# ЛР: Redis + Rocket Counter. От `docker run` к `docker compose` и масштабированию


Создаем и запускаем контейнер с Redis с именем строго redis:
```
docker run -d --name redis redis
```

Проверка:

```
docker ps
```

<img width="974" height="243" alt="image" src="https://github.com/user-attachments/assets/3a19590d-1172-4f44-88c6-db851fef2125" />

Запуск приложения rocketcounter с --link

```
docker run -d --name rocketcounter -p 8085:5000 --link redis:redis rotorocloud/rocket-counter
```

<img width="974" height="548" alt="image" src="https://github.com/user-attachments/assets/d0f0323c-73c1-40a3-bec5-131534a476f3" />

Проверка:

```
docker ps
```
<img width="974" height="91" alt="image" src="https://github.com/user-attachments/assets/b012ef12-62f7-4b05-8c84-3511ece55dcf" />

Проверьте доступность страницы через curl:

в PowerShell команда curl является встроенным псевдонимом (alias) для командлета Invoke-WebRequest, а стандартная утилита head (выводящая первые строки) отсутствует в Windows по умолчанию
Использую команду, чтобы вывести содержимое:
```
(Invoke-WebRequest -Uri http://localhost:8085).Content
``` 
<img width="974" height="562" alt="image" src="https://github.com/user-attachments/assets/54a47038-a55b-4207-b2b6-2b6810ec3d82" />

Открываю в браузере http://localhost:8085

<img width="974" height="529" alt="image" src="https://github.com/user-attachments/assets/5b719759-faab-49d8-ba2d-456093454ace" />

Число кликов увеличивается:

<img width="974" height="527" alt="image" src="https://github.com/user-attachments/assets/fc72c5da-0ef3-471d-8dea-ed7ae0501b85" />

Удаляю созданные вручную контейнеры (остановить и удалить):

```
docker stop redis rocketcounter
docker rm redis rocketcounter
```
<img width="892" height="253" alt="image" src="https://github.com/user-attachments/assets/111a767b-199f-4520-8432-f400a80b9dbf" />

Создаю рабочую директорию:

```
mkdir -p /root/rocketcounter
cd /root/rocketcounter

```

Создаю файл docker-compose.yml через консоль

notepad docker-compose.yml

Содержимое файла:

```
version: '3.8'

services:
  redis:
    image: redis
    container_name: redis

  rocketcounter:
    image: rotorocloud/rocket-counter
    ports:
      - "0:5000"
    depends_on:
      - redis
```
Запускаю контейнер в фоновом режиме: 

```
docker compose up -d
```
<img width="974" height="88" alt="image" src="https://github.com/user-attachments/assets/eac539b9-4ac6-4346-ac02-71efa35b9a6a" />

Проверка:

```
docker ps
```

<img width="974" height="76" alt="image" src="https://github.com/user-attachments/assets/23eff7e5-c175-46f9-8b28-fd710186dbbc" />

Страница открывается и счетчик работает, если открыть сайт по нашему адресу:

<img width="974" height="528" alt="image" src="https://github.com/user-attachments/assets/2362d5ff-d279-4eec-a1ee-b836c92dee44" />

Масштабирование приложения


Важно: на одном хосте публиковать один и тот же фиксированный порт (8085) на нескольких репликах невозможно. Поэтому для масштабирования на одном узле используйте вариант ниже.
Измените публикацию портов на динамическую (эпемерные порты для каждого экземпляра):
```
ports:
  - "0:5000"     # хост выберет свободный порт автоматически
```

Применяю изменения и масштабирую:

```
docker compose up -d --scale rocketcounter=2
docker compose ps
```
<img width="974" height="164" alt="image" src="https://github.com/user-attachments/assets/f8ae3a89-e22f-42b1-a21c-c56fe22be5a4" />

Открываем: localhost:32768

<img width="974" height="527" alt="image" src="https://github.com/user-attachments/assets/fe5ed0be-fba4-40de-b716-b98e9e8877fc" />

Открываем: localhost:32769

<img width="974" height="528" alt="image" src="https://github.com/user-attachments/assets/ec1547e0-cd07-4346-bb6b-b9dd0be34eb5" />

Полное уничтожение стека

```
docker compose down -v
```
Проверка удаленных контейнеров (вместо grep):
```
docker ps -a
```
Проверка сетей (вместо grep):

```
docker network ls
```

<img width="974" height="170" alt="image" src="https://github.com/user-attachments/assets/932d78f4-599a-4fb0-b202-d35f8f4c5860" />

Очистка прошла успешно: старые контейнеры (redis и rocketcounter) удалены, а пользовательские сети стека полностью стерты.

# Ответы на контрольные вопросы:

# 1.	Зачем в Ч.1 использовался --link и почему в Compose он не нужен?
--link использовался для того, чтобы один контейнер мог обращаться к другому по имени (redis), а также для автоматической прописки записей в файл /etc/hosts и передачи переменных окружения. В Docker Compose это не нужно, так как Compose автоматически создает изолированную пользовательскую сеть для каждого проекта, внутри которой работает встроенный DNS-сервер. Контейнеры автоматически резолвят друг друга по именам сервисов (например: redis).

# 2.	Чем отличается image от build в Compose?
- image указывает готовый образ, который нужно скачать из реестра и запустить.
- build указывает путь к локальной директории с Dockerfile, из которой Docker Compose должен самостоятельно собрать образ перед запуском контейнера.

# 3.	Почему нельзя масштабировать сервис с публикацией фиксированного порта на одном хосте? Перечислите способы обойти это ограничение.
Сетевой стек операционной системы позволяет занять конкретный порт хоста (8085) только одному процессу (или одному контейнеру). Попытка запустить второй контейнер с тем же портом приведет к ошибке.

Способы обхода:
- Использование динамических (эпемерных) портов хоста ("0:5000"), как мы сделали в лабораторной.
- Использование обратного прокси-сервера, который принимает весь внешний трафик на фиксированном порту и балансирует запросы между внутренними репликами контейнеров.

# 4. Что делает docker compose down -v и чем отличается от down без -v?
-	docker compose down останавливает и удаляет контейнеры и сети, созданные стеком.
-	docker compose down -v (где -v означает --volumes) дополнительно удаляет все именованные тома (Volumes), объявленные в секции volumes файла docker-compose.yml (или созданные автоматически для баз данных), что приводит к безвозвратному удалению сохраненных данных. Без ключа -v данные в томах сохраняются.
  
# 5.	Как в Compose обеспечивается сетевое имя сервиса для обращения из другого сервиса?
Docker Compose создает для проекта выделенную пользовательскую сеть мостового типа (bridge network). Встроенный в эту сеть DNS-сервер Docker автоматически регистрирует каждое имя сервиса (например: redis) в качестве доменного имени, которое разрешается в IP-адрес соответствующего контейнера.

# UPDATE
1. Исправлено имя файла docker-compose.yml
2. Почему при масштабировании необходимо отказаться от фиксированного имени (container_name)?
Ответ: При масштабировании сервиса в Docker Compose необходимо отказаться от фиксированного имени контейнера (container_name), потому что архитектура Docker требует абсолютной уникальности имен всех контейнеров

Если в файле конфигурации жестко задано имя (например: container_name: rocketcounter), то при попытке запустить несколько реплик (--scale rocketcounter=2) оба новых контейнера попытаются получить абсолютно одинаковое имя. Демон Docker обнаружит конфликт имен, выдаст ошибку и откажется запускать стек.

Если убрать параметр container_name, Docker Compose будет автоматически генерировать уникальные имена для каждого экземпляра (например: rocketcounter-rocketcounter-1, rocketcounter-rocketcounter-2), добавляя к ним порядковый номер.



