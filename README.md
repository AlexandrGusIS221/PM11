# Словарь данных для SQL Server

Я проанализировал твой файл и выделил **13 таблиц**. Давай разберём каждую с типами данных и ограничениями. Я объясню логику выбора типов, чтобы ты понимал, почему именно так.

---

## 📋 Общие принципы выбора типов

Прежде чем перейти к таблицам, объясню логику:

1. **`INT`** — для ID и числовых внешних ключей (до ~2 млрд значений, с запасом)
2. **`NVARCHAR(N)`** — для русского текста (поддерживает Unicode). Размер выбираем "с запасом", но не избыточно
3. **`DECIMAL(10,2)`** — для денег (точность важна, `FLOAT` не подходит из-за погрешностей)
4. **`DATE`** — для дат без времени
5. **`DATETIME2`** — для дат со временем (современнее `DATETIME`)
6. **`TINYINT`** — для маленьких чисел (0-255), например для `Sale` (скидка в %)

---

## 🗂 Таблицы

### 1. **Category** (Категории товаров)
| Поле | Тип | Ограничения | Примечание |
|------|-----|-------------|------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) | Автоинкремент |
| Name | NVARCHAR(50) | NOT NULL, UNIQUE | "Женская обувь" = 13 симв., 50 с запасом |

### 2. **Unit** (Единицы измерения)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(20) | NOT NULL, UNIQUE |

### 3. **Manufacture** (Производители)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL |

### 4. **Supplier** (Поставщики)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(100) | NOT NULL |

> ⚠️ Здесь данные подозрительные: `Name = "1"` — возможно, в исходнике ошибка. Уточни.

### 5. **Type** (Типы обуви)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL, UNIQUE |

> ⚠️ `Type` — зарезервированное слово в SQL. Лучше переименовать в `ProductType` или экранировать ``.

### 6. **Role** (Роли пользователей)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL, UNIQUE |

### 7. **User** (Пользователи)
| Поле | Тип | Ограничения | Примечание |
|------|-----|-------------|------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) | |
| RoleID | INT | NOT NULL, FK → Role(ID) | |
| Name | NVARCHAR(50) | NOT NULL | Имя |
| SurName | NVARCHAR(50) | NOT NULL | Фамилия |
| LastName | NVARCHAR(50) | NULL | Отчество (может отсутствовать) |
| Login | NVARCHAR(100) | NOT NULL, UNIQUE | Email до 100 символов |
| Password | NVARCHAR(255) | NOT NULL | С запасом под хеш (bcrypt = 60, SHA256 = 64) |

> ⚠️ `User` тоже зарезервированное слово — используй `` или переименуй в `Users`.
> 💡 Пароли в открытом виде хранить **нельзя**! В дипломе обязательно используй хеширование.

### 8. **Status** (Статусы заказов)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL, UNIQUE |

### 9. **Order** (Заказы)
| Поле | Тип | Ограничения | Примечание |
|------|-----|-------------|------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) | |
| CreateAt | DATE | NOT NULL | Дата создания |
| DeliveryAt | DATE | NULL | Дата доставки |
| Code | INT | NOT NULL, UNIQUE | Код заказа (3 цифры — но лучше INT с запасом) |
| AddressID | INT | NOT NULL, FK → Address(ID) | |
| ClientID | INT | NOT NULL, FK → User(ID) | |
| StatusID | INT | NOT NULL, FK → Status(ID) | |

> ⚠️ `Order` — **зарезервированное слово**! Обязательно используй `` или переименуй в `Orders`.
> ⚠️ В данных у строки 7 дата `30.02.2025` — некорректная (февраль не имеет 30 дней). Уточни.

### 10. **Product** (Товары)
| Поле | Тип | Ограничения | Примечание |
|------|-----|-------------|------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) | |
| Article | NVARCHAR(20) | NOT NULL, UNIQUE | Артикул, у тебя 6 симв., с запасом |
| Cost | DECIMAL(10,2) | NOT NULL, CHECK (Cost >= 0) | Цена |
| Sale | TINYINT | NOT NULL DEFAULT 0, CHECK (Sale BETWEEN 0 AND 100) | Скидка в % |
| Quantity | INT | NOT NULL DEFAULT 0, CHECK (Quantity >= 0) | Остаток |
| Description | NVARCHAR(500) | NULL | Описание |
| Photo | NVARCHAR(255) | NULL | Имя файла |
| UnitID | INT | NOT NULL, FK → Unit(ID) | |
| TypeID | INT | NOT NULL, FK → Type(ID) | |
| SupplierID | INT | NOT NULL, FK → Supplier(ID) | |
| ManufactureID | INT | NOT NULL, FK → Manufacture(ID) | |
| CategoryID | INT | NOT NULL, FK → Category(ID) | |

### 11. **OrderProduct** (Состав заказа — связь многие-ко-многим)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| OrderID | INT | NOT NULL, FK → Order(ID) |
| ProductID | INT | NOT NULL, FK → Product(ID) |
| Quantity | INT | NOT NULL, CHECK (Quantity > 0) |

### 12. **Address** (Адреса)
| Поле | Тип | Ограничения | Примечание |
|------|-----|-------------|------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) | |
| PostCode | CHAR(6) | NOT NULL | Индекс РФ — ровно 6 цифр |
| HouseNumber | NVARCHAR(10) | NOT NULL | Может быть "12А", "5/2" |
| CityID | INT | NOT NULL, FK → City(ID) | |
| StreetID | INT | NOT NULL, FK → Street(ID) | |

> 💡 Для индекса использую `CHAR(6)` — фиксированная длина экономнее, чем `NVARCHAR`.

### 13. **Street** (Улицы)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL |

### 14. **City** (Города)
| Поле | Тип | Ограничения |
|------|-----|-------------|
| ID | INT | PRIMARY KEY, IDENTITY(1,1) |
| Name | NVARCHAR(50) | NOT NULL |

---

## ⚠️ На что обратить внимание

1. **Зарезервированные слова**: `Order`, `User`, `Type` — рекомендую переименовать (`Orders`, `Users`, `ProductType`)
2. **Даты в CSV в формате Excel**: числа вроде `45715` — это serial date Excel. Нужно конвертировать при импорте в `DATE`
3. **Странность в Supplier**: запись `1, "1"` выглядит как ошибка экспорта
4. **Пароли**: в дипломе обязательно используй `password_hash()` в PHP, а в БД храни хеш в `NVARCHAR(255)`
5. **Кодировка**: создавай БД с `COLLATE Cyrillic_General_CI_AS` для корректной работы с русским

---

Хочешь, чтобы я следом написал **готовый SQL-скрипт `CREATE TABLE`** со всеми ограничениями и внешними ключами в правильном порядке создания? Или сначала уточним спорные моменты (например, по Supplier и датам)?


USE [ИмяТвоейБазы];
GO

-- ============================================
-- 1. Category
-- ============================================
ALTER TABLE [Category] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 2. Unit
-- ============================================
ALTER TABLE [Unit] ALTER COLUMN [Name] NVARCHAR(20) NOT NULL;
GO

-- ============================================
-- 3. Manufacture
-- ============================================
ALTER TABLE [Manufacture] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 4. Supplier
-- ============================================
ALTER TABLE [Supplier] ALTER COLUMN [Name] NVARCHAR(100) NOT NULL;
GO

-- ============================================
-- 5. Type (зарезервированное слово - экранируем)
-- ============================================
ALTER TABLE [Type] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 6. Role
-- ============================================
ALTER TABLE [Role] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 7. User (зарезервированное слово)
-- ============================================
ALTER TABLE [User] ALTER COLUMN [Name]     NVARCHAR(50)  NOT NULL;
ALTER TABLE [User] ALTER COLUMN [SurName]  NVARCHAR(50)  NOT NULL;
ALTER TABLE [User] ALTER COLUMN [LastName] NVARCHAR(50)  NULL;
ALTER TABLE [User] ALTER COLUMN [Login]    NVARCHAR(100) NOT NULL;
ALTER TABLE [User] ALTER COLUMN [Password] NVARCHAR(255) NOT NULL;
GO

-- ============================================
-- 8. Status
-- ============================================
ALTER TABLE [Status] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 9. Order (зарезервированное слово)
-- ============================================
ALTER TABLE [Order] ALTER COLUMN [CreateAt]   DATE NOT NULL;
ALTER TABLE [Order] ALTER COLUMN [DeliveryAt] DATE NULL;
ALTER TABLE [Order] ALTER COLUMN [Code]       INT  NOT NULL;
GO

-- ============================================
-- 10. Product
-- ============================================
ALTER TABLE [Product] ALTER COLUMN [Article]     NVARCHAR(20)   NOT NULL;
ALTER TABLE [Product] ALTER COLUMN [Cost]        DECIMAL(10,2)  NOT NULL;
ALTER TABLE [Product] ALTER COLUMN [Sale]        TINYINT        NOT NULL;
ALTER TABLE [Product] ALTER COLUMN [Quantity]    INT            NOT NULL;
ALTER TABLE [Product] ALTER COLUMN [Description] NVARCHAR(500)  NULL;
ALTER TABLE [Product] ALTER COLUMN [Photo]       NVARCHAR(255)  NULL;
GO

-- ============================================
-- 11. OrderProduct
-- ============================================
ALTER TABLE [OrderProduct] ALTER COLUMN [Quantity] INT NOT NULL;
GO

-- ============================================
-- 12. Address
-- ============================================
ALTER TABLE [Address] ALTER COLUMN [PostCode]    CHAR(6)      NOT NULL;
ALTER TABLE [Address] ALTER COLUMN [HouseNumber] NVARCHAR(10) NOT NULL;
GO

-- ============================================
-- 13. Street
-- ============================================
ALTER TABLE [Street] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO

-- ============================================
-- 14. City
-- ============================================
ALTER TABLE [City] ALTER COLUMN [Name] NVARCHAR(50) NOT NULL;
GO


