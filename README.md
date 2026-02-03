# Лабораторна робота 1
## Розробка систем обміну повідомленнями


Роботу виконав Сич Станіслав Олегович
Варіант 2
---
# Виконання

## Завдання 1: Створити схему компонентів
```mermaid
graph LR
    S[Клієнт-Відправник] -->|1. POST /messages| G[API Gateway]
    G -->|2. createMessage| M[Message Service]
    M -->|3. setStatus sending| T[Status Service]
    M -->|4. save to DB| DB[(Database)]
    M -->|5. publish| Q[Message Queue]
    Q -->|6. consume| D[Delivery Service]
    D -->|7. setStatus sent| T
    D -->|8. deliver| R[Клієнт-Отримувач]
    R -->|9. ACK delivered| G
    G -->|10. updateStatus delivered| T
    R -->|11. ACK read| G
    G -->|12. updateStatus read| T
    T -->|13. log history| DB
    T -->|14. notify sender| S
```
## Завдання 2: Діаграма послідовності

```mermaid
sequenceDiagram
    participant S as Sender
    participant GW as API Gateway
    participant MS as Message Service
    participant SS as Status Service
    participant DB as Database
    participant DS as Delivery Service
    participant R as Receiver

    S->>GW: POST /messages {to, text}
    GW->>MS: Створити повідомлення
    MS->>DB: Зберегти (status: "sending")
    MS->>SS: Оновити статус "sending"
    SS->>DB: Записати в історію
    MS-->>GW: Повернути ID
    GW-->>S: 200 OK
    
    Note over MS,DS: Асинхронна доставка
    
    MS->>DS: Надіслати через чергу
    DS->>SS: Оновити статус "sent"
    SS->>DB: Записати "sent"
    
    alt Отримувач онлайн
        DS->>R: Доставити через WebSocket
        R->>SS: ACK "delivered"
        SS->>DB: Записати "delivered"
        R->>SS: ACK "read"
        SS->>DB: Записати "read"
    else Отримувач офлайн
        DS->>DS: Зберегти для подальшої доставки
        Note over R: При підключенні синхронізація
    end
```

## Завдання 3: Діаграма стану

```mermaid
stateDiagram-v2
    [*] --> sending
    sending --> sent
    sent --> delivered
    delivered --> read
    read --> [*]
    
    sending --> failed
    sent --> failed
    delivered --> failed
    
    failed --> retrying
    retrying --> sent
    retrying --> [*]
    ```

## Завдання 4: ADR

##Status
Under consideration

##Desicion
Централізований Status Service з наступними обов'язками:

- Єдине джерело правди для всіх статусів повідомлень
- Валідація переходів між станами
- Зберігання повної історії змін статусів
- Публікація подій про зміни статусів

##Consequences
- Усі зміни статусів проходять через одну точку
- Унеможливлення конфліктуючих оновлень
- Легкий аудит та дебаггінг
- Якщо Status Service недоступний, система не може оновлювати статуси


---