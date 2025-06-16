## Sequence-диаграмма авторизации

```mermaid
sequenceDiagram
  actor User as User
  participant UI as UI (React)
  participant Nginx as Nginx (Reverse-proxy)
  participant Bachend as Backend
  participant DB as Database

  User-->Browser: Открытие URL: https:localhost:3000
  Broser-->Nginx: GET/
  alt Запрос статики (HTML,CSS,JS)
    Nginx-->UI: Отдаёт index.html
    UI-->Nginx: Статические файлы
    Nginx-->Browser: 200 (HTML,CSS)
    Browser-->User: Отображает интерфейс
  end

  
    
