# SQLTestQ

**Задание А**
Нужно сделать одну таблицу (Divisions) для хранения иерархической структуры (дерева), например ВУЗ, Факультет, Кафедра (ВУЗ может иметь несколько факультетов, а факультет несколько кафедр). Cостав данных подразделений идентичен: "Наимнование", "Адрес".

Какие поля в таблице создадите?

**Ответ A**
```sql
CREATE TABLE Divisions(
  Division_ID INT PRIMARY KEY,
  Parent_ID INT,
  Name VARCHAR(255),
  Address VARCHAR(255)
);
```
Поля:
- Division_ID - первичный ключ для идентификации подразделения.
- Parent_ID - внешний ключ, для связывания с подразделений друг с другом.
- Name - имя подразделения.
- Address - адрес подразделения

---
**Задание В**
Таблица клиентов: Customers, поля: CustId, CustName
Содержимое:
| CustId| CustName     |
| ------|:------------:|
| 1     | Компания1    |
| 2     | Компания2    |
| 3     | Компания3    |
| 4     | Компания4    |

Таблица заказов: Orders поля: OrderId, CustId, OrderDate, Amount
Содержимое:
| OrderId| CustId    | OrderDate     | Amount    |
| -------|:---------:|--------------:|----------:|
| 1      |1          |     15.01.2021|    4300.00|
| 2      |1          |     12.02.2020|    1200.00|
| 3      |2          |     21.04.2019|     300.00|
| 4      |2          |     29.08.2020|    4800.00|
| 5      |3          |     02.12.2017|    1100.00|
| 6      |3          |     14.03.2021|    2300.00|
  
**Задача B.1**: Вывести запросом список клиентов (код, наименование), у которых нет ни одного заказа.  
**Ответ B.1**
```sql
SELECT c.CustId, c.CustName
FROM Customers c
LEFT JOIN Orders o ON c.CustId = o.CustId
WHERE o.OrderId IS NULL;
```
  
**Задача B.2**: Со временем таблица Orders стала очень большой (несколько миллионов записей), и запрос стал медленно работать. Что нужно сделать, чтобы ускорить выборку?
Медленно работающий запрос:
```sql
select CustId, count(1) as Orders, sum(Amount) as TotalAmount from Orders where CustId = 15 group by CustId
```
**Ответ B.2**
- Оптимизировать запрос. Заменить некоторые операции на более эффективные, например ```COUNT(*)```
  ```sql
  SELECT CustId, COUNT(*) AS Orders, SUM(Amount) AS TotalAmount
  FROM Orders
  WHERE CustId = 15
  GROUP BY CustId;
  ```
- Создать индекс по столбцу ```CustId```. Позволит БД быстро находить строки с нужным значением ```CustId```.
  ```sql
  CREATE INDEX idx_cust_id ON Orders (CustId);
  ```

  
**Задача B.3**: Вывести запросом три поля - код клиента, его наименование и номер последнего по дате заказа. 
По возможности, выбор даты сделать без использования вложенного запроса в списке полей, то есть избегая конструкций типа:
```sql
select CustId, (select top 1 OrderDate from ...) from...
```
**Ответ B.3**
```sql
WITH LastOrders AS(
  SELECT CustId, MAX(OrderDate) AS LastOrderDate
  FROM Orders
  GROUP BY CustId
)

SELECT c.CustId, c.CustName, o.LastOrderDate
FROM Customers c
INNER JOIN LastOrders o ON c.CustId = o.CustId;
```

---
**Задание С**
Ваша программа работает с базой данных в СУБД. Пользователей много и в пиковые часы нагрузка на базу данных в СУБД довольно велика. В то же время на основе данных из этой базы пользователи запускают построение отчётов, что еще больше нагружает СУБД. Что, в свою очередь, приводит к тому, что пользователи начинают жаловаться на снижение производительности системы (простыми словами - всё тормозит).

Какие варианты решения проблемы Вы бы предложили?
**Ответ C**
Основные решения:  
- Оптимизация запросов. К примеру добавление индексов или использование эффективных операций.
- Кэширование. Возможно выявить часто запрашиваемые данные и реализовать механизм кэширования данных для запросов. Позволит снизить нагрузку на саму БД.
- Шардирование. Проще говоря, можно "разрезать БД на куски". Разделить хранилище на несколько независимых частей. Это позволит уменьшить нагрузку на отдельные серверы.
- Масштабирование. Если есть возможность увеличить ресурсы сервера БД, можно рассмотреть и такой вариант.

---
**Задание D**
Перед Вами поставлена задача выгрузки документов во внешнюю информационную систему (ИС). Внешняя ИС реализована и поддерживается сторонними разработчиками, и получает данные через интернет с использованием web-сервиса (то есть надо учесть возможные ситуации, что не работает интернет или внешняя ИС находится на обслуживании).

Оператор формирует документ в Вашей программе, и нужно выгрузить этот документ во внешнюю ИС.

Какое архитектурное программное решение Вы бы предложили? (Программу писать не надо, только основные тезисы).
**Ответ D**
Основные решения:
- Асинхронная обработка. Реализовать асинхронный механизм выгрузки документов, чтобы оператор мог формировать документы в программе без блокировки интерфейса.
- Очередь задач. Вдобавок к асинхронному выполнению можно использовать механизм очереди задач для планирования выгрузки, если нет возможности мгновенной выгрузки данных. Необходимо так же реализовать повторную отправку, если появились ошибки или сбои при первоначальной выгрузке. Позволит обеспечить надежность и сохранность доставки данных.
- Локальное хранение. Использовать временное локальное хранение документов, которые необходимо выгрузить во внешнюю ИС. Это так же позволит обеспечить сохранность данных в случае недоступности внешней ИС(отсутствие подключения к сети и т.д.).

