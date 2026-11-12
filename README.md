# API для Yatube

REST API для социальной сети Yatube. Через него можно публиковать посты,
комментировать их, смотреть сообщества и подписываться на других авторов.
API пригодится, если нужно подключить к Yatube мобильное приложение, бота
или любой другой клиент, которому удобнее работать с JSON, а не с HTML.

Что умеет API:

- посты: просмотр, создание, редактирование и удаление, есть пагинация
  через `limit` и `offset`;
- комментарии к постам;
- сообщества (группы) — только чтение, создаются через админку;
- подписки на авторов с поиском по `search`;
- аутентификация по JWT-токену.

Без токена API доступен только на чтение. Исключение — `/follow/`, туда
пускают только аутентифицированных пользователей. Изменять и удалять пост
или комментарий может только его автор.

## Технологии

- Python 3.9
- Django 3.2
- Django REST Framework 3.12
- Simple JWT + Djoser
- SQLite

## Как запустить проект

Клонировать репозиторий и перейти в него в командной строке:

```bash
git clone https://github.com/Danieltukg/api-final-yatube.git
cd api-final-yatube
```

Создать и активировать виртуальное окружение:

```bash
python3 -m venv venv
source venv/bin/activate
```

На Windows:

```bash
python -m venv venv
source venv/Scripts/activate
```

Установить зависимости из файла requirements.txt:

```bash
python3 -m pip install --upgrade pip
pip install -r requirements.txt
```

Выполнить миграции:

```bash
cd yatube_api
python3 manage.py migrate
```

Создать суперпользователя (нужен, чтобы заводить сообщества в админке):

```bash
python3 manage.py createsuperuser
```

Запустить проект:

```bash
python3 manage.py runserver
```

После запуска документация будет доступна по адресу
http://127.0.0.1:8000/redoc/, админка — http://127.0.0.1:8000/admin/.

## Примеры запросов

### Получить токен

`POST /api/v1/jwt/create/`

```json
{
    "username": "regular_user",
    "password": "iWannaBeAdmin"
}
```

Ответ:

```json
{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

Токен `access` передаётся в заголовке всех запросов, где нужна авторизация:

```
Authorization: Bearer <access>
```

Обновить токен можно через `POST /api/v1/jwt/refresh/`, проверить —
через `POST /api/v1/jwt/verify/`.

### Создать пост

`POST /api/v1/posts/`

```json
{
    "text": "Первый пост через API",
    "group": 1
}
```

Ответ:

```json
{
    "id": 1,
    "author": "regular_user",
    "text": "Первый пост через API",
    "pub_date": "2026-11-12T09:15:42.381507Z",
    "image": null,
    "group": 1
}
```

### Получить посты с пагинацией

`GET /api/v1/posts/?limit=2&offset=4`

```json
{
    "count": 12,
    "next": "http://127.0.0.1:8000/api/v1/posts/?limit=2&offset=6",
    "previous": "http://127.0.0.1:8000/api/v1/posts/?limit=2&offset=2",
    "results": [
        {
            "id": 8,
            "author": "regular_user",
            "text": "Текст поста",
            "pub_date": "2026-11-12T09:10:03.114209Z",
            "image": null,
            "group": null
        }
    ]
}
```

Без параметров `limit` и `offset` возвращается обычный список всех постов.

### Добавить комментарий

`POST /api/v1/posts/1/comments/`

```json
{
    "text": "Отличный пост!"
}
```

Ответ:

```json
{
    "id": 1,
    "author": "regular_user",
    "text": "Отличный пост!",
    "created": "2026-11-12T09:20:11.502416Z",
    "post": 1
}
```

### Подписаться на автора

`POST /api/v1/follow/`

```json
{
    "following": "root"
}
```

Ответ:

```json
{
    "user": "regular_user",
    "following": "root"
}
```

Подписаться на самого себя или подписаться повторно не получится — API
вернёт ошибку `400`.

### Найти подписку

`GET /api/v1/follow/?search=root`

```json
[
    {
        "user": "regular_user",
        "following": "root"
    }
]
```

Полный список эндпоинтов с описанием параметров и кодов ответа — в
документации `/redoc/`.

## Тесты

Из корня проекта:

```bash
pytest
```

Ещё в папке `postman_collection` лежит коллекция запросов для Postman,
инструкция по запуску — в [postman_collection/README.md](postman_collection/README.md).
