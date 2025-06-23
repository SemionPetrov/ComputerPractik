Отчёт по лабораторной работе №10
Тема: Создание веб-приложения с использованием Flask и Vue.js
Цель работы
Научиться создавать простые веб-приложения с использованием фреймворка Flask (Python), а также внедрять клиентскую логику с помощью Vue.js. Практиковаться в генерации изображений на стороне сервера и их передаче на клиент.

Структура проекта

LR10/

├── templates/

│   ├── makeimage.html

│   └── result.html

└── app.py
Описание функциональности
В рамках лабораторной работы реализовано веб-приложение, позволяющее пользователю:

Вводить параметры изображения: ширина, высота и текст.
Генерировать изображение на сервере с использованием библиотеки Pillow.
Получать результат в формате Base64 и отображать его в браузере.

Форма создания изображения 
(templates/makeimage.html)

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Создание изображения</title>
        <script src="https://cdn.jsdelivr.net/npm/vue@2"></script> 
    </head>
    <body>
        <div id="app">
            <h1>Создание изображения</h1>
            <form @submit.prevent="generateImage">
                <label for="width">Ширина:</label>
                <input type="text" v-model="width" id="width" required>
                <br>
                <label for="height">Высота:</label>
                <input type="text" v-model="height" id="height" required>
                <br>
                <label for="text">Текст:</label>
                <input type="text" v-model="text" id="text" required>
                <br>
                <input type="submit" value="Создать">
            </form>
            <p v-if="message" style="color: red;">Неверные размеры изображения</p>
            <div v-if="imageData">
                <h2>Созданное изображение</h2>
                <img :src="imageData" alt="Созданное изображение">
            </div>
        </div>

    <script>
        new Vue({
            el: '#app',
            data: {
                width: '',
                height: '',
                text: '',
                imageData: null,
                message: false
            },
            methods: {
                async generateImage() {
                    const response = await fetch('/makeimage', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded',
                        },
                        body: new URLSearchParams({
                            width: this.width,
                            height: this.height,
                            text: this.text
                        })
                    });

                    if (!response.ok) {
                        this.message = true;
                        return;
                    }

                    const data = await response.json();
                    this.message = false;
                    sessionStorage.setItem('imageData', data.image);
                    window.location.href = '/result';
                }
            }
        });
    </script>
    </body>
    </html>

Описание:
Это HTML-страница с формой, связанной с JavaScript через Vue.js. При отправке формы происходит POST-запрос на сервер, который генерирует изображение и возвращает его в виде Base64-кодированной строки.

Страница результата (templates/result.html)

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Созданное изображение</title>
    </head>
    <body>
        <h1>Созданное изображение</h1>
        <img id="result-image" alt="Generated Image">
        <br>
        <script>
            document.addEventListener('DOMContentLoaded', () => {
                const data = sessionStorage.getItem('imageData');
                if (data) {
                    document.getElementById('result-image').src = 'data:image/jpeg;base64,' + data;
                } else {
                    document.body.innerHTML = '<p style="color: red;">Данные изображения не найдены. Пожалуйста, сначала создайте изображение.</p>';
                }
            });
        </script>
    </body>
    </html>
Описание:
После генерации изображения пользователь перенаправляется на страницу /result, где изображение отображается через sessionStorage. Если данные отсутствуют — выводится сообщение об ошибке.
    
Серверная часть (app.py)
    
    from flask import Flask, request, jsonify, render_template
    from PIL import Image, ImageDraw, ImageFont
    import io
    import base64
    
    app = Flask(__name__)
    
    
    @app.route('/login')
    def login():
        return jsonify({"author": "1149806"})
    
    
    @app.route('/makeimage', methods=['POST', 'GET'])
    def make_image():
        if request.method == "GET":
            return render_template('makeimage.html')

        width = request.form.get('width')
        height = request.form.get('height')
        text = request.form.get('text')
    
        if not width.isdigit() or not height.isdigit() or int(width) <= 0 or int(height) <= 0:
            return jsonify({'error': 'Invalid image size'}), 400
    
        width, height = int(width), int(height)
    
        # Создаем черное изображение
        image = Image.new('RGB', (width, height), color=(0, 0, 0))
        draw = ImageDraw.Draw(image)
    
        font = ImageFont.load_default()
        draw.text((10, 10), text, fill=(255, 255, 255), font=font)
    
        img_bytes = io.BytesIO()
        image.save(img_bytes, format='JPEG')
        img_bytes.seek(0)
    
        encoded_img = base64.b64encode(img_bytes.getvalue()).decode('utf-8')
    
        return jsonify({'image': encoded_img})


    @app.route('/result')
    def result():
        return render_template('result.html')
    
    
    if __name__ == '__main__':
        app.run(debug=True)
Описание:
Файл app.py содержит основную логику Flask-приложения:

Обрабатывает GET- и POST-запросы к маршруту /makeimage.
Проверяет корректность входных данных.
Создает изображение с помощью библиотеки Pillow.
Кодирует изображение в формат Base64 и отправляет его клиенту.
Реализует маршруты /login, /makeimage, /result.
Как запустить проект
Убедитесь, что установлены зависимости:
bash



    pip install flask pillow
Запустите приложение:
bash


    python app.py
Перейдите по адресу:


    http://127.0.0.1:5000/makeimage

Вывод
В ходе выполнения лабораторной работы были освоены:

Работа с Flask: маршруты, обработка запросов.
Использование библиотеки Pillow для генерации изображений.
Интеграция Vue.js для интерактивного взаимодействия с пользователем.
Передача изображений в формате Base64 между сервером и клиентом.
Проект демонстрирует базовый уровень взаимодействия клиента и сервера и может быть расширен за счёт добавления новых возможностей: выбор цвета, загрузки шрифтов, сохранения изображений и т.д.

