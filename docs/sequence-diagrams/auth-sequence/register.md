![image](https://github.com/user-attachments/assets/4d84eed5-a1f6-4004-ae5d-7019a0fa231b)


    sequenceDiagram
    actor User as Пользователь
    participant Nginx as Nginx (Reverse-proxy)
    participant UI as UI (React)
    participant Bachend as Backend
    participant DB as Database
    
    User->>Nginx: GET /register (Открытие формы) 
    Nginx->>UI: Запрос статики (HTML,CSS,JS) 
    UI->>Nginx: Ответ (200 0K) 
    Nginx->>User: 200 OK (Отображение формы)
    
    
    User->>UI: Ввод email и пароля и повторенного пароля
    UI->>UI: Валидация полей(regex)
    alt Данные невалидны ❌
      UI->>User: Ошибка "Неккоретный email или пароль"
    else Данные валидны ✅
      UI->>Nginx: POST /api/register {email, password}
      Nginx->>Backend: Проксирование запроса
      Backend->>DB: Проверка существования email
      alt Email уже занят ⚠️
          DB->>Backend: Ошибка 409
          Backend->>Nginx: 409 Conflict
          Nginx->>UI: "Email уже существует"
          UI->>User: 🔴 Показать ошибку 
      else Email свободен ✅
          Backend->>Backend: Хеширование пароля (bcrypt)
          Backend->>DB: Сохранение пользователя
          DB->>Backend: 201 Created
          Backend->>Nginx: 201 JSON {status: "ok"}
          Nginx->>UI: Успешный Ответ
          UI->>User: 🟢 Редирект /
        end  
       end
       Note over User,DB: Верификация email
        User->>UI: Ввод кода из письма #FF5733
        UI->>Nginx: POST /api/verify {email, code} #2ECC71
        Nginx->>Backend: Проксирование #3498DB
        Backend->>DB: Проверка кода #9B59B6
        alt Код неверен/истёк ❌
            DB-->>Backend: 400 #F39C12
            Backend-->>Nginx: 400 Bad Request #9B59B6
            Nginx-->>UI: "Неверный код" #3498DB
            UI-->>User: 🔴 Ошибка + кнопка повторной отправки #FF0000
        else Код верный ✅
            Backend->>DB: Активация аккаунта #9B59B6
            DB-->>Backend: 200 #2ECC71
            Backend-->>Nginx: 200 OK {token} #9B59B6
            Nginx->>UI: Успешная активация #3498DB
            UI->>User: 🟢 Редирект /dashboard #2ECC71
        end

