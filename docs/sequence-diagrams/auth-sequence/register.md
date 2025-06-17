![image](https://github.com/user-attachments/assets/b8ab9ac0-2cca-48aa-8689-5875ac6f6ff1)

    sequenceDiagram
    actor User as Пользователь #FF5733
    participant Nginx as Nginx (Reverse-proxy)
    participant UI as UI (React)
    participant Bachend as Backend
    participant DB as Database
  
    User->>Nginx: GET /register (Открытие формы) #FF5733
    Nginx->>UI: Запрос статики (HTML,CSS,JS) 3498DB
    UI->>Nginx: Ответ (200 0K) #2ECC71
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
