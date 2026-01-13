# Otus-DB__5
 DML: вставка, обновление, удаление, выборка данных

1) SELECT с регулярным выражением + пояснение

Ищем корректные email-адреса 
```
SELECT
  cc.id,
  cc.value AS email,
  c.full_name,
  e.name AS entity_name,
  e.entity_type
FROM mdm.contact_channels cc
JOIN mdm.contacts c ON c.id = cc.contact_id
JOIN mdm.entities e ON e.id = c.entity_id
WHERE cc.channel_type = 'email'
  AND cc.value ~* '^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$'
ORDER BY cc.id;
```
<img width="791" height="351" alt="image" src="https://github.com/user-attachments/assets/13537d54-9d93-4d24-87d7-988bde1b6f1c" />

2. LEFT JOIN и INNER JOIN + как порядок в FROM влияет на результат

2.1 LEFT JOIN: показать всех контрагентов + primary контакт
```
SELECT
  e.id,
  e.entity_type,
  e.name AS entity_name,
  c.full_name AS primary_contact
FROM mdm.entities e
LEFT JOIN mdm.contacts c
  ON c.entity_id = e.id
 AND c.is_primary = true
ORDER BY e.id;
```
<img width="614" height="351" alt="image" src="https://github.com/user-attachments/assets/07494091-c7a7-4204-b0fb-75ebd6a9068b" />

2.2 INNER JOIN: показать только тех, у кого есть primary контакт

```
SELECT
  e.id,
  e.entity_type,
  e.name AS entity_name,
  c.full_name AS primary_contact
FROM mdm.entities e
JOIN mdm.contacts c
  ON c.entity_id = e.id
 AND c.is_primary = true
ORDER BY e.id;
```
<img width="632" height="309" alt="image" src="https://github.com/user-attachments/assets/1ccbd5bf-20fb-42a5-8860-f9e812121d17" />

3) INSERT INTO с выводом добавленных строк

```
INSERT INTO catalog.customers(name, type, inn, contact_name, phone, email, address, is_active)
VALUES ('ООО Тест-Клиент', 'b2b', '7722334455', 'Кузнецов Кирилл', '+7 (495) 777-88-99',
        'kuznetsov@test-client.ru', 'Москва, Тестовая, 12', true)
RETURNING id, name, inn, created_at;
```
<img width="1128" height="186" alt="image" src="https://github.com/user-attachments/assets/49ae558c-19bd-4c4e-84a4-ff1c28214d23" />

4. UPDATE FROM обновление на основе другой таблицы

```
UPDATE sales.orders o
SET total_amount = s.sum_total,
    updated_at = now()
FROM (
  SELECT
    oi.order_id,
    SUM(
      COALESCE(
        oi.line_total,
        oi.quantity * oi.unit_price * (1 - oi.discount_percent / 100.0)
      )
    )::numeric(14,2) AS sum_total
  FROM sales.order_items oi
  GROUP BY oi.order_id
) s
WHERE s.order_id = o.id
RETURNING o.id, o.total_amount, o.updated_at;
```
<img width="745" height="479" alt="image" src="https://github.com/user-attachments/assets/291d3806-a80f-40f2-92e3-496964a76ccd" />


5. DELETE USING удаление через join с другой таблицей

```
DELETE FROM catalog.prices p
USING catalog.customers c
WHERE p.customer_id = c.id
  AND c.is_active = false
RETURNING p.id, p.product_id, p.customer_id, p.valid_from;
```
<img width="646" height="228" alt="image" src="https://github.com/user-attachments/assets/b61a5b16-0ecb-4a74-8f94-c7ccc70ebc5f" />

6. COPY (серверный файл; в Docker, внутри контейнера)

  ```
 COPY catalog.customers
TO '/tmp/customers_export.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ';');
```
\copy (клиентский файл; работает в psql и пишет на локальную/виртуальную машину, либо как сейчас в WSL)
```
\copy catalog.customers TO 'customers_export.csv' WITH (FORMAT csv, HEADER true, DELIMITER ';');
```
<img width="1170" height="151" alt="image" src="https://github.com/user-attachments/assets/04a432fb-5889-42fc-87fc-1b382c38017a" />


