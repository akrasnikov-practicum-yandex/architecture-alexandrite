# Жизненный цикл заказа — Sequence Diagram

```mermaid
%%{init: {'theme':'base','themeVariables':{'fontSize':'14px','actorBkg':'#dae8fc','actorBorder':'#5b7fb0','actorTextColor':'#0b2545','actorLineColor':'#5b7fb0','signalColor':'#1f2d3d','signalTextColor':'#0b2545','labelBoxBkgColor':'#dae8fc','labelBoxBorderColor':'#5b7fb0','labelTextColor':'#0b2545','loopTextColor':'#0b2545','noteBkgColor':'#fff2cc','noteBorderColor':'#d6b656','noteTextColor':'#0b2545','sequenceNumberColor':'#ffffff','activationBkgColor':'#e2e8f0','activationBorderColor':'#7a8aa0'}}}%%
sequenceDiagram
    autonumber

    actor Customer as Покупатель
    participant Shop as Internet Shop<br/>(Vue + Three.js)
    participant ShopAPI as Shop API<br/>(Java Spring Boot)
    participant S3 as S3 Storage<br/>(3D модели)
    participant ShopDB as Shop DB<br/>(PostgreSQL)
    participant MQ as RabbitMQ
    participant MesAPI as MES API<br/>(C# .NET)
    participant MesDB as MES DB<br/>(PostgreSQL)
    participant CrmAPI as CRM API<br/>(Java Spring Boot)
    participant CRM as CRM<br/>(Vue)
    actor Seller as Продавец
    actor Operator as Оператор
    participant MES as MES<br/>(React)

    rect rgb(230, 245, 255)
        Note over Customer,ShopDB: 🛒 Фаза 1 — Создание заказа [Online Shop]

        Customer->>Shop: Открывает корзину / начинает заказ
        Shop->>ShopAPI: POST /orders
        ShopAPI->>ShopDB: INSERT order (status=INITIATED)
        ShopAPI-->>Shop: order_id
        Shop-->>Customer: Заказ создан ✓

        Customer->>Shop: Загружает 3D-модель или использует конструктор
        Shop->>ShopAPI: POST /orders/{id}/file
        ShopAPI->>S3: Upload 3D file
        S3-->>ShopAPI: file_url
        ShopAPI->>ShopDB: UPDATE order (status=FILE_UPLOADED, file_url)
        ShopAPI-->>Shop: FILE_UPLOADED ✓
        Shop-->>Customer: Файл принят ✓

        Customer->>Shop: Нажимает «Оформить заказ»
        Shop->>ShopAPI: POST /orders/{id}/submit
        ShopAPI->>ShopDB: UPDATE order (status=SUBMITTED)
        ShopAPI-->>Shop: SUBMITTED ✓
        Shop-->>Customer: Заказ передан в производство ✓
    end

    rect rgb(255, 245, 230)
        Note over ShopDB,CrmAPI: ⚙️ Фаза 2 — Передача в очередь и расчёт цены [CRM API → MES API]

        Note over ShopAPI,CrmAPI: Shop API не подключён к очереди (см. C4-модель).<br/>Мост к RabbitMQ — CRM API: он делит Shop DB с Shop API<br/>и забирает новые заказы в статусе SUBMITTED.
        CrmAPI->>ShopDB: SELECT новые заказы (status=SUBMITTED)
        ShopDB-->>CrmAPI: order {order_id, file_url}
        CrmAPI->>MQ: Publish → order.submitted {order_id, file_url}

        MQ-->>MesAPI: Consume order.submitted
        MesAPI->>S3: GET 3D file (file_url)
        S3-->>MesAPI: 3D model data

        Note over MesAPI: Расчёт стоимости<br/>(CPU-интенсивно, 2–30 мин)

        MesAPI->>MesDB: INSERT manufacturing_order (status=PRICE_CALCULATED, price)
        MesAPI->>MQ: Publish → order.price_calculated {order_id, price}
    end

    rect rgb(230, 255, 230)
        Note over MQ,Seller: ✅ Фаза 3 — Подтверждение производства [CRM]

        MQ-->>CrmAPI: Consume order.price_calculated
        CrmAPI->>ShopDB: UPDATE order (status=PRICE_CALCULATED, price)

        Seller->>CRM: Просматривает заказ с рассчитанной ценой
        CRM->>CrmAPI: GET /orders/{id}
        CrmAPI->>ShopDB: SELECT order
        ShopDB-->>CrmAPI: order data
        CrmAPI-->>CRM: order details

        Seller->>CRM: Подтверждает запуск производства
        CRM->>CrmAPI: POST /orders/{id}/approve
        CrmAPI->>ShopDB: UPDATE order (status=MANUFACTURING_APPROVED)
        CrmAPI->>MQ: Publish → order.approved {order_id}
        CrmAPI-->>CRM: APPROVED ✓
    end

    rect rgb(255, 230, 255)
        Note over MQ,MesDB: 🏭 Фаза 4 — Производство [MES]

        MQ-->>MesAPI: Consume order.approved
        MesAPI->>MesDB: UPDATE manufacturing_order (status=MANUFACTURING_APPROVED)

        Operator->>MES: Выбирает заказ из очереди
        MES->>MesAPI: GET /manufacturing-orders?status=MANUFACTURING_APPROVED
        MesAPI->>MesDB: SELECT orders (status, created_at)
        MesDB-->>MesAPI: orders list
        MesAPI-->>MES: orders list
        MES-->>Operator: Список заказов

        Operator->>MES: Берёт заказ в работу
        MES->>MesAPI: POST /manufacturing-orders/{id}/start
        MesAPI->>MesDB: UPDATE manufacturing_order (status=MANUFACTURING_STARTED)
        MesAPI->>MQ: Publish → order.status_changed {order_id, status=MANUFACTURING_STARTED}

        Note over Operator,MES: Оператор выполняет изготовление

        Operator->>MES: Завершает изготовление
        MES->>MesAPI: POST /manufacturing-orders/{id}/complete
        MesAPI->>MesDB: UPDATE manufacturing_order (status=MANUFACTURING_COMPLETED)
        MesAPI->>MQ: Publish → order.status_changed {order_id, status=MANUFACTURING_COMPLETED}

        Operator->>MES: Начинает упаковку
        MES->>MesAPI: POST /manufacturing-orders/{id}/package
        MesAPI->>MesDB: UPDATE manufacturing_order (status=PACKAGING)
        MesAPI->>MQ: Publish → order.status_changed {order_id, status=PACKAGING}

        Operator->>MES: Передаёт в доставку
        MES->>MesAPI: POST /manufacturing-orders/{id}/ship
        MesAPI->>MesDB: UPDATE manufacturing_order (status=SHIPPED)
        MesAPI->>MQ: Publish → order.status_changed {order_id, status=SHIPPED}
    end

    rect rgb(240, 240, 240)
        Note over MQ,Seller: 📦 Фаза 5 — Закрытие заказа [CRM]

        MQ-->>CrmAPI: Consume order.status_changed (все статусы МЕС)
        CrmAPI->>ShopDB: UPDATE order (status=SHIPPED / PACKAGING / ...)

        Note over CrmAPI,Seller: После подтверждения доставки<br/>от транспортной компании<br/>или ручное закрытие продавцом

        alt Автоматическое закрытие
            CrmAPI->>ShopDB: UPDATE order (status=CLOSED)
            CrmAPI-->>CRM: Заказ закрыт автоматически
        else Ручное закрытие продавцом
            Seller->>CRM: Закрывает заказ вручную
            CRM->>CrmAPI: POST /orders/{id}/close
            CrmAPI->>ShopDB: UPDATE order (status=CLOSED)
            CrmAPI-->>CRM: CLOSED ✓
        end

        CRM-->>Seller: Заказ завершён ✓
    end
```

## Статусы заказа

| # | Статус | Система | Триггер |
|---|--------|---------|---------|
| 1 | `INITIATED` | Online Shop | Покупатель создал корзину |
| 2 | `FILE_UPLOADED` | Online Shop | Загружена/создана 3D-модель |
| 3 | `SUBMITTED` | Online Shop | Покупатель нажал «Оформить заказ» |
| 4 | `PRICE_CALCULATED` | MES | Расчёт стоимости завершён (2–30 мин) |
| 5 | `MANUFACTURING_APPROVED` | CRM | Продавец подтвердил производство |
| 6 | `MANUFACTURING_STARTED` | MES | Оператор взял заказ в работу |
| 7 | `MANUFACTURING_COMPLETED` | MES | Оператор завершил изготовление |
| 8 | `PACKAGING` | MES | Оператор начал упаковку |
| 9 | `SHIPPED` | MES | Заказ передан в доставку |
| 10 | `CLOSED` | CRM | Подтверждение от ТК или ручное закрытие |

## Участники

| Компонент | Технология | Роль |
|-----------|-----------|------|
| Internet Shop | Vue + TypeScript + Three.js | UI покупателя |
| Shop API | Java Spring Boot | Backend магазина |
| CRM | Vue + TypeScript | UI продавца |
| CRM API | Java Spring Boot | Backend CRM |
| MES | React + TypeScript | UI оператора |
| MES API | C# .NET | Расчёт цены, производство |
| RabbitMQ | Message Broker | Асинхронная интеграция между сервисами |
| Shop DB | PostgreSQL | Заказы, покупатели |
| MES DB | PostgreSQL | Производственные заказы, статусы |
| S3 Storage | Object Storage | 3D-модели |
