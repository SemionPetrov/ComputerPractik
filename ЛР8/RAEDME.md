
# Лабораторная работа №8

## Цель
Разработать простое веб-приложение на Python с использованием FastAPI, которое принимает изображение формата PNG через `multipart/form-data` и возвращает его размеры в формате JSON. Также реализован маршрут, возвращающий логин автора.

## Используемые технологии
- **FastAPI** — фреймворк для создания API.
- **Pillow (PIL)** — библиотека для работы с изображениями.
- **Python 3.10+**

## код приложения

    from fastapi import FastAPI, UploadFile, File, HTTPException
    from fastapi.responses import JSONResponse
    from PIL import Image
    from io import BytesIO
    
    app = FastAPI()
    
    MOODLE_LOGIN = "1149806"
    
    @app.post("/size2json")
    async def get_image_size(image: UploadFile = File(...)):
        # Проверка расширения файла
        if not image.filename.lower().endswith(".png"):
            return JSONResponse(
                status_code=400,
                content={"result": "invalid filetype"},
                media_type="application/json"
            )
    
        # Проверка содержимого файла (что это действительно PNG)
        try:
            contents = await image.read()
            img = Image.open(BytesIO(contents))
            if img.format != "PNG":
                return JSONResponse(
                    status_code=400,
                    content={"result": "invalid filetype"},
                    media_type="application/json"
                )
            width, height = img.size
            return JSONResponse(
                content={"width": width, "height": height},
                media_type="application/json"
            )
        except Exception as e:
            return JSONResponse(
                status_code=400,
                content={"result": "invalid filetype"},
                media_type="application/json"
            )
    
    
    @app.get("/login")
    def get_login():
        return JSONResponse(
            content={"author": MOODLE_LOGIN},
            media_type="application/json"
        )

## Установка зависимостей

    pip install fastapi uvicorn pillow
## Маршруты

POST /size2json

Принимает файл в поле image формата PNG через multipart/form-data.

Если файл не является PNG — возвращает:

    {"result": "invalid filetype"}
Если всё верно — возвращает:

    {"width": 123, "height": 456}
GET /login

Возвращает логин в системе MOODLE:

    {"author": "1149806"}
Пример запуска
Запуск сервера:
    
    uvicorn main:app --reload
После запуска приложение будет доступно по адресу:

    https://localhost:8000 
## Вывод

Было создано работающее HTTPS-веб-приложение, удовлетворяющее всем заданным требованиям. Оно корректно обрабатывает загрузку PNG-изображений, проверяет их тип и возвращает ширину и высоту в формате JSON. Так же реализован маршрут /login, возвращающий логин MOODLE.
