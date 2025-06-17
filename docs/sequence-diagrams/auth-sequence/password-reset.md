![image](https://github.com/user-attachments/assets/5fbb4a8f-a96b-46c6-93c4-362573ef2fe6)

    sequenceDiagram
        actor User as Пользователь
        participant Nginx as Nginx
        participant UI as UI (React)
        participant Backend as Backend
        participant DB as Database
        participant MailService as Mail Service
        Note over User,MailService: Фаза 1 - Инициация сброса пароля
        User->>UI: Клик "Забыли пароль?"
        UI->>Nginx: GET /password-reset
        Nginx->>UI: Статика формы сброса
        UI->>User: Форма ввода email
        User->>UI: Ввод email
        UI->>UI: Валидация email
        alt Email невалиден ❌
            UI-->>User: 🟥 "Некорректный email"
        else Email валиден ✅
            UI->>Nginx: POST /api/password-reset {email}
            Nginx->>Backend: Проксирование запроса
            Backend->>DB: Поиск пользователя
            alt Пользователь не найден ⚠️
                DB-->>Backend: 404
                Backend-->>Nginx: 200 (без указания ошибки)
                Nginx-->>UI: "Если email существует, письмо отправлено"
                UI-->>User: 🟡 Сообщение о проверке почты
            else Пользователь найден ✅
                Backend->>Backend: Генерация 6-значного кода
                Backend->>DB: Сохранение кода (с TTL 15 мин)
                Backend->>MailService: Отправка кода на email
                MailService-->>User: ✉️ Письмо с кодом
                Backend-->>Nginx: 200 OK
                Nginx->>UI: Подтверждение отправки
                UI->>User: Редирект на /verify-code
            end
        end
        Note over User,MailService: Фаза 2 - Ввод кода подтверждения
        User->>UI: Ввод 6-значного кода
        UI->>Nginx: POST /api/verify-code {email, code}
        Nginx->>Backend: Проксирование запроса
        Backend->>DB: Проверка кода
        alt Код неверен/истёк ❌
            DB-->>Backend: 400
            Backend-->>Nginx: 400 Bad Request
            Nginx-->>UI: "Неверный код"
            UI-->>User: 🔴 Ошибка + кнопка повторной отправки
        else Код верный ✅
            Backend->>Backend: Генерация токена сброса (JWT, 10 мин)
            Backend-->>Nginx: 200 {reset_token}
            Nginx->>UI: Успешная верификация
            UI->>User: Редирект на /new-password
        end
        Note over User,DB: Фаза 3 - Установка нового пароля
        User->>UI: Ввод нового пароля
        UI->>UI: Валидация пароля
        alt Пароль невалиден ❌
            UI-->>User: 🟥 "Пароль слишком простой"
        else Пароль валиден ✅
            UI->>Nginx: POST /api/new-password {reset_token, new_password}
            Nginx->>Backend: Проксирование запроса
            Backend->>Backend: Верификация токена
            alt Токен невалиден ⚠️
                Backend-->>Nginx: 401
                Nginx-->>UI: "Сессия истекла"
                UI-->>User: 🔴 Ошибка + возврат на шаг 1
            else Токен валиден ✅
                Backend->>Backend: Хеширование пароля
                Backend->>DB: Обновление пароля
                DB-->>Backend: 200 OK
                Backend->>DB: Удаление использованных кодов
                Backend-->>Nginx: 200 OK
                Nginx->>UI: Успех
                UI->>User: 🟢 "Пароль изменён" → /login
            end
        end
