Отчет по Лабораторной Работе 11

Задание

Разработать веб-приложение, которое:

По маршруту /decypher принимает приватный ключ и зашифрованное содержимое в формате multipart/form-data и выдает результат расшифровки.
Реализует фронтенд с использованием Vue.js для асинхронной отправки данных на сервер и отображения результата без перезагрузки страницы.
Отвечает на маршрут /login с JSON-ответом, содержащим логин автора.
Использует Flask как backend-фреймворк.
Структура Проекта

├── templates

│   ├── decypher.html          # HTML-шаблон для страницы дешифрования

│   └── encrypt.html           # HTML-шаблон для страницы генерации ключей и шифрования

└── app.py                     # Основной файл приложения на Flask

Содержимое Файлов

1. app.py
   
Это основной файл приложения, реализующий логику работы сервера на Flask.

        from flask import Flask, jsonify, request, render_template, send_file
        from cryptography.hazmat.primitives import serialization, hashes
        from cryptography.hazmat.primitives.asymmetric import rsa, padding
        from io import BytesIO
        
        app = Flask(__name__)
        
        # Ограничение размера загружаемых файлов
        app.config['MAX_CONTENT_LENGTH'] = 2 * 1024 * 1024
        
        # Маршрут для страницы дешифрования
        @app.route('/decypher')
        def decypher_page():
            return render_template('decypher.html')
        
        # Маршрут для страницы шифрования
        @app.route('/encrypt')
        def encrypt_page():
            return render_template('encrypt.html')
        
        # Маршрут для проверки авторизации
        @app.route('/login')
        def login():
            return jsonify({"author": "1149806"})
        
        # Маршрут для генерации RSA-ключей
        @app.route('/generate_keys', methods=['POST'])
        def generate_keys():
            private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
            public_key = private_key.public_key()
        
            # Сериализация приватного и публичного ключей в PEM-формат
            private_pem = private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption()
            )
            public_pem = public_key.public_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            )
        
            return jsonify({
                "private_key": private_pem.decode('utf-8'),
                "public_key": public_pem.decode('utf-8')
            })
        
        # Маршрут для шифрования сообщения
        @app.route('/encrypt_message', methods=['POST'])
        def encrypt_message():
            data = request.get_json()
            public_key_str = data.get('public_key')
            message = data.get('message')
        
            if not public_key_str or not message:
                return jsonify({"error": "Both public key and message are required"}), 400
        
            try:
                # Загрузка публичного ключа
                public_key = serialization.load_pem_public_key(public_key_str.encode('utf-8'))
        
                # Шифрование сообщения
                encrypted = public_key.encrypt(
                    message.encode('utf-8'),
                    padding.OAEP(mgf=padding.MGF1(algorithm=hashes.SHA256()), algorithm=hashes.SHA256(), label=None)
                )
        
                # Возврат зашифрованного сообщения как файла
                return send_file(BytesIO(encrypted), mimetype='application/octet-stream', as_attachment=True, download_name='secret.bin')
        
            except Exception as e:
                return jsonify({"error": str(e)}), 500
        
        # Маршрут для дешифрования сообщения
        @app.route('/decypher', methods=['POST'])
        def decypher():
            if 'key' not in request.files or 'secret' not in request.files:
                return jsonify({"error": "Both key and secret fields are required"}), 400
        
            key_file = request.files['key']
            secret_file = request.files['secret']
        
            try:
                # Загрузка приватного ключа
                private_key_data = key_file.read()    
Объяснение:

Маршрут /decypher: Обрабатывает POST-запрос с приватным ключом и зашифрованным содержимым, выполняет дешифрование и возвращает результат.

Маршрут /encrypt: Предоставляет страницу для генерации ключей и шифрования сообщений.

Маршрут /login: Возвращает JSON-ответ с логином автора.

Маршрут /generate_keys: Генерирует RSA-ключи и возвращает их в PEM-формате.

Маршрут /encrypt_message: Принимает публичный ключ и сообщение, шифрует его и возвращает зашифрованный файл.

2. templates/decypher.html

HTML-шаблон для страницы дешифрования с интеграцией Vue.js.

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>RSA Decryptor</title>
        <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>   
        <style>
            body { font-family: Arial, sans-serif; line-height: 1.6; margin: 0; padding: 20px; }
            .container { max-width: 800px; margin: 0 auto; }
            .form-group { margin-bottom: 15px; }
            label { display: block; margin-bottom: 5px; font-weight: bold; }
            textarea, input[type="file"] { width: 100%; padding: 8px; }
            button { background: #42b983; color: white; border: none; padding: 10px 15px; cursor: pointer; }
            #result { margin-top: 20px; padding: 15px; border: 1px solid #ddd; white-space: pre-wrap; }
            .error { color: red; }
        </style>
    </head>
    <body>
        <div id="app" class="container">
            <h1>🔐 RSA Decryptor</h1>
    
            <div class="form-group">
                <label for="key">Приватный ключ (PEM):</label>
                <input type="file" id="key" @change="handleFileUpload" accept=".pem">
            </div>
    
            <div class="form-group">
                <label for="secret">Зашифрованный текст:</label>
                <textarea id="secret" v-model="secret" rows="5"></textarea>
            </div>
    
            <button @click="decrypt">Расшифровать</button>
    
            <div v-if="result" id="result">
                <h3>Результат:</h3>
                <div v-if="error" class="error">{{ error }}</div>
                <pre v-else>{{ result }}</pre>
            </div>
        </div>
    
        <script>
            const { createApp, ref } = Vue;
    
            createApp({
                setup() {
                    // Данные состояния
                    const secret = ref('');
                    const result = ref('');
                    const error = ref('');
                    const keyFile = ref(null);
    
                    // Обработчик загрузки файла приватного ключа
                    const handleFileUpload = (event) => {
                        keyFile.value = event.target.files[0];
                    };
    
                    // Метод для дешифрования
                    const decrypt = async () => {
                        if (!keyFile.value || !secret.value) {
                            error.value = "Загрузите ключ и введите зашифрованный текст";
                            return;
                        }
    
                        // Создание FormData для отправки файлов и текста
                        const formData = new FormData();
                        formData.append('key', keyFile.value);
                        formData.append('secret', secret.value);
    
                        try {
                            // Отправка POST-запроса на сервер
                            const response = await fetch('/decypher', {
                                method: 'POST',
                                body: formData
                            });
    
                            const data = await response.json();
    
                            if (response.ok) {
                                result.value = data.decrypted;
                                error.value = '';
                            } else {
                                throw new Error(data.error || "Ошибка дешифровки");
                            }
                        } catch (err) {
                            error.value = err.message;
                            result.value = '';
                        }
                    };
    
                    return { secret, result, error, handleFileUpload, decrypt };
                }
            }).mount('#app');
        </script>
    </body>
    </html>
Объяснение:

Vue.js: Используется для создания динамического интерфейса.

Форма: Позволяет пользователю загрузить приватный ключ и ввести зашифрованное сообщение.

Асинхронная отправка данных: При нажатии кнопки "Расшифровать", данные отправляются на сервер через fetch, и результат отображается на странице без перезагрузки.

3. templates/encrypt.html

HTML-шаблон для страницы генерации ключей и шифрования сообщений.

    <!DOCTYPE html>
    <html lang="ru">
    <head>
      <meta charset="UTF-8">
      <title>Генерация и шифрование</title>
    </head>
    <body>
      <h1>Генерация ключей</h1>
      <button onclick="generateKeys()">Сгенерировать</button>
    
      <h3>Приватный ключ</h3>
      <textarea id="private" readonly style="background-color: #f0f0f0;"></textarea>
      <h3>Публичный ключ</h3>
      <textarea id="public" readonly style="background-color: #f0f0f0;"></textarea>
    
      <h1>Шифрование сообщения</h1>
      <textarea id="message" placeholder="Введите сообщение..."></textarea><br>
      <button onclick="encrypt()">Зашифровать и скачать</button>
    
      <div class="result" id="result"></div>
    
      <script>
        async function generateKeys() {
          const response = await fetch('/generate_keys', { method: 'POST' });
          const data = await response.json();
          document.getElementById('private').value = data.private_key;
          document.getElementById('public').value = data.public_key;
        }
    
        async function encrypt() {
          const message = document.getElementById('message').value;
          const publicKey = document.getElementById('public').value;
    
          const response = await fetch('/encrypt_message', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ public_key: publicKey, message: message })
          });
    
          if (response.ok) {
            const blob = await response.blob();
            const url = window.URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'secret.bin';
            a.click();
          } else {
            const error = await response.json();
            document.getElementById('result').textContent = error.error || 'Ошибка шифрования';
          }
        }
      </script>
    </body>
    </html>
Объяснение:

Генерация ключей: Пользователь может сгенерировать RSA-ключи, которые отображаются в текстовых полях.

Шифрование сообщения: Пользователь вводит сообщение и использует публичный ключ для его шифрования. Зашифрованное сообщение сохраняется как файл secret.bin.

Для тестирования можно запустить приложение локально:

    http://localhost:5000/
Ссылка на GitHub
Ссылка на репозиторий

Вывод
В данном проекте был разработан функциональный сервис для генерации RSA-ключей, шифрования и дешифрования сообщений. Фронтенд реализован с использованием Vue.js, что обеспечивает асинхронную работу и удобство взаимодействия с пользователем. Backend выполнен на Flask, который обрабатывает запросы и выполняет криптографические операции.
