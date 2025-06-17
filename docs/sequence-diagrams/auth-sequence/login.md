![image](https://github.com/user-attachments/assets/af03d893-c5be-42cf-a7b2-686e57bf1445)
`sequenceDiagram
    actor User as Пользователь
    participant Nginx as Nginx (Reverse-proxy)
    participant UI as UI (React) #2ECC71
    participant Backend as Backend #9B59B6
    participant DB as Database #F1C40F`



    User->>Nginx: GET /login (Открытие формы)
    Nginx->>UI: Запрос статики (HTML,CSS,JS) 
    UI->>Nginx: Ответ (200 OK) 
    Nginx->>User: Отображение формы входа 

    User->>UI: Ввод email и пароля
    UI->>UI: Клиентская валидация

    alt Данные невалидны ❌
        UI-->>User: 🟥 "Некорректный email или пароль" 
    else Данные валидны ✅
        UI->>Nginx: POST /api/login {email, password} 
        Nginx->>Backend: Проксирование запроса 
        Backend->>DB: Поиск пользователя по email 

        alt Пользователь не найден ⚠️
            DB-->>Backend: 🟨 404 Not Found 
            Backend-->>Nginx: 401 Unauthorized 
            Nginx-->>UI: "Неверные учетные данные" 
            UI-->>User: 🔴 Ошибка авторизации 

        else Пользователь найден ✅
            Backend->>Backend: Сравнение хеша пароля (bcrypt) 
            alt Пароль неверный ⚠️
                Backend-->>Nginx: 401 Unauthorized 
                Nginx-->>UI: "Неверный пароль" 
                UI-->>User: 🔴 Ошибка авторизации 
            else Пароль верный ✅
                Backend->>Backend: Генерация JWT-токена 
                Backend-->>Nginx: 200 OK {token: "jwt"} 
                Nginx->>UI: Успешный ответ #3498DB
                UI->>User: 🟢 Редирект в / 
            end
        end
    end





