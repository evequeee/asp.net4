# Hardware Store - Мікросервісна архітектура з .NET Aspire

## 📋 Опис проєкту

Це повна реалізація **Лабораторної роботи №5**: мікросервісна платформа Hardware Store з використанням .NET Aspire, YARP API Gateway, OpenTelemetry для трасування, та Serilog для структурованого логування.

## 🏗️ Архітектура

### Компоненти системи:

1. **HardwareStore.AppHost** - Orchestrator (Aspire)
   - Управління lifecycle всіх мікросервісів
   - Service discovery
   - Автоматичне налаштування контейнерів (MongoDB, SQL Server)

2. **HardwareStore.ServiceDefaults** - Спільна бібліотека
   - Serilog з CompactJsonFormatter
   - OpenTelemetry (трасування HTTP, SQL, AspNetCore)
   - CorrelationId middleware
   - Health checks

3. **HardwareStore.ApiGateway** - API Gateway (YARP)
   - Єдина точка входу (порт 5000)
   - Routing до мікросервісів
   - Load balancing
   - CorrelationId propagation

4. **HardwareStore.AggregatorService** - Aggregator Pattern
   - Об'єднання даних з декількох мікросервісів
   - Typed HttpClient з service discovery
   - Dashboard endpoint з аналітикою

5. **HardwareStore.WebUI** - Основний мікросервіс
   - Products CRUD API
   - MediatR + CQRS
   - MongoDB persistence
   - FluentValidation

## 🚀 Запуск системи

### Передумови:

✅ **.NET 9 SDK** встановлений  
✅ **Docker Desktop** запущений  
✅ **Visual Studio 2022** або **VS Code**

### Крок 1: Запуск Docker Desktop

```powershell
# Переконайтеся, що Docker Desktop запущений
docker ps
```

### Крок 2: Запуск AppHost

```powershell
cd g:\programming\asp.net.lab4
dotnet run --project HardwareStore.AppHost
```

### Крок 3: Відкриття Aspire Dashboard

Після запуску AppHost відкриється браузер з **Aspire Dashboard** за адресою:
```
https://localhost:17XXX/
```

Dashboard покаже:
- 📊 Всі запущені сервіси
- 📈 Metrics (CPU, Memory, Requests/sec)
- 🔍 Distributed traces
- 📝 Structured logs
- ❤️ Health checks

## 📡 API Endpoints

### Через API Gateway (порт 5000):

#### Products API:
```http
GET    http://localhost:5000/api/products
GET    http://localhost:5000/api/products/{id}
POST   http://localhost:5000/api/products
PUT    http://localhost:5000/api/products/{id}
DELETE http://localhost:5000/api/products/{id}
```

#### Aggregator API:
```http
GET http://localhost:5000/api/aggregator/dashboard
```
Повертає:
```json
{
  "products": [...],
  "totalProducts": 10,
  "totalInventoryValue": 15000,
  "productsByCategory": {
    "Power Tools": 5,
    "Hand Tools": 5
  },
  "retrievedAt": "2025-12-09T..."
}
```

```http
GET http://localhost:5000/api/aggregator/product/{id}
```
Повертає збагачену інформацію про продукт.

### Health Checks:
```http
GET http://localhost:5000/health
GET http://localhost:5000/alive
```

## 🔍 Моніторинг та Логування

### Serilog Structured Logging

Кожен лог містить:
- `ServiceName` - назва сервісу
- `CorrelationId` - унікальний ID запиту
- `Environment` - Development/Production
- `MachineName` - ім'я машини/контейнера
- `Timestamp`, `Level`, `Message`

Приклад лога:
```json
{
  "Timestamp": "2025-12-09T15:30:00.000Z",
  "Level": "Information",
  "MessageTemplate": "Fetching all products from WebAPI",
  "CorrelationId": "abc-123-def",
  "ServiceName": "aggregator",
  "MachineName": "DESKTOP-PC"
}
```

### OpenTelemetry Tracing

Трасування включає:
- HTTP requests (вхідні та вихідні)
- SQL queries (для SQL Server)
- MongoDB operations (через custom instrumentation)
- Міжсервісна комунікація

### CorrelationId

Автоматично генерується для кожного запиту та передається через всі сервіси:
```
Client → Gateway → Aggregator → WebAPI → Database
         [X-Correlation-Id: abc-123-def]
```

## 🗄️ База даних

### MongoDB
- Автоматично запускається в Docker контейнері
- Volume для збереження даних: `mongodb-data`
- Connection string автоматично inject'ується через Aspire

### Seeding
При старті WebUI автоматично створюються тестові дані:
- 10 Products
- 5 Customers

## 🧪 Тестування

### 1. Перевірка запуску всіх сервісів

У Aspire Dashboard перевірте:
- ✅ `gateway` (Running)
- ✅ `aggregator` (Running)
- ✅ `webapi` (Running)
- ✅ `mongodb` (Running)

### 2. Тест E2E через Gateway

```powershell
# Отримати всі продукти через Gateway
Invoke-WebRequest -Uri "http://localhost:5000/api/products" -Method Get

# Отримати dashboard через Aggregator
Invoke-WebRequest -Uri "http://localhost:5000/api/aggregator/dashboard" -Method Get
```

### 3. Перевірка CorrelationId

```powershell
$response = Invoke-WebRequest -Uri "http://localhost:5000/api/products" -Method Get
$response.Headers["X-Correlation-Id"]
```

Той самий CorrelationId буде у логах усіх сервісів.

### 4. Перевірка Traces

1. Відкрийте Aspire Dashboard
2. Перейдіть на вкладку **Traces**
3. Знайдіть trace за CorrelationId
4. Подивіться ієрархію викликів:
   ```
   Gateway (50ms)
     └─ Aggregator (40ms)
         └─ WebAPI (30ms)
             └─ MongoDB Query (20ms)
   ```

## 📊 Критерії виконання (згідно лаби)

### ✅ Частина 1: Ініціалізація проєкту
- [x] Створено Aspire solution з AppHost та ServiceDefaults
- [x] Налаштовано структуру папок для мікросервісів
- [x] Встановлено NuGet пакети (Aspire.Hosting.*, YARP, Serilog, OpenTelemetry)

### ✅ Частина 2: AppHost - Оркестрація мікросервісів
- [x] MongoDB контейнер з volume та persistence
- [x] Зареєстровані всі мікросервіси з правильними connection strings
- [x] Налаштовані залежності через WaitFor
- [x] Service discovery через Aspire inject connection strings

### ✅ Частина 3: Інтеграція мікросервісів
- [x] ServiceDefaults підключені до всіх сервісів
- [x] Centralized logging через Serilog
- [x] OpenTelemetry spans для HTTP/DB operations
- [x] Connection strings інжектяться через Aspire

### ✅ Частина 4: ServiceDefaults - Централізовані налаштування
- [x] Serilog з enrichers (ServiceName, Environment, MachineName)
- [x] CompactJsonFormatter для Dashboard
- [x] CorrelationId middleware з LogContext
- [x] OpenTelemetry instrumentation (ASP.NET Core, HTTP, SQL)
- [x] OTLP exporter для Aspire Dashboard

### ✅ Частина 5: API Gateway з YARP
- [x] YARP reverse proxy конфігурація
- [x] Routes для всіх мікросервісів (catch-all patterns)
- [x] Clusters з service discovery (замість localhost:port)
- [x] CorrelationId propagation через проксі
- [x] Gateway як entry point (порт 5000)

### ✅ Частина 6: Aggregator Service
- [x] Typed HttpClients з service discovery
- [x] Aggregation endpoint з паралельними викликами
- [x] Частковий response при failure (graceful degradation)
- [x] Composite DTOs для об'єднаних даних
- [x] Structured logging з business properties

### ✅ Частина 7: Тестування та моніторинг
- [x] Aspire Dashboard відкривається автоматично
- [x] Всі сервіси Running в Dashboard
- [x] Health check endpoints працюють
- [x] Traces показують full path (Gateway → Aggregator → Microservices)
- [x] CorrelationId генерується та передається
- [x] Structured logs містять enrichments
- [x] End-to-end запит працює через Gateway

## 🎯 Наступні кроки (опціонально)

- [ ] Додати Customers та Orders мікросервіси
- [ ] Реалізувати Saga Pattern для distributed transactions
- [ ] Додати Rate Limiting та Circuit Breaker в Gateway
- [ ] Налаштувати Elasticsearch для централізованих логів
- [ ] Додати Authentication/Authorization (JWT)
- [ ] Реалізувати Event-Driven комунікацію (RabbitMQ/Azure Service Bus)

## 📚 Документація

- [.NET Aspire](https://learn.microsoft.com/en-us/dotnet/aspire/)
- [YARP Reverse Proxy](https://microsoft.github.io/reverse-proxy/)
- [OpenTelemetry](https://opentelemetry.io/docs/instrumentation/net/)
- [Serilog](https://serilog.net/)

---

**Автор**: Виконано згідно вимог Лабораторної роботи №5  
**Дата**: 9 грудня 2025
