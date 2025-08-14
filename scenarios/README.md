# Техническое интервью Java-разработчика

## Полезные ссылки
* [live-coding](https://code.yandex-team.ru/)

## 1. Общие вопросы про Computer Science и REST API

Наше interview построено следующим образом:
* не будем спрашивать технические вопросы напрямую (это слишком нудно);
* вместо этого, мы с нуля спроектируем сервис;
* после чего попробуем найти у устранить его узкие места.

Представьте себе, что мы проектируем сервис некоторого вымышленного банка, 
который хочет предоставить своим партнёрам возможность интеграции с его системой через REST API.

### 1.1 Практическое задание про REST API

Цель интеграции по REST API заключается в предоставлении:
* API для создания и редактирования счетов и карт клиента;
* API для получения информации о банковских счетах и картах клиента.

**Шаблон для live-coding REST API:**
```
# API для получения списка счетов (accounts) клиента банка. 
# Это могут быть как вклады, расчетные счета, накопительные счета

# Пример DTO:
{
    "type": "DEPOSIT",
    "number": "4054474869669569703",
    "clientId": "fca1d622-d1d7-49be-936a-f771e7030dc7",
    "currency": "RUR",
    "balance": 1000,
    "savingCounter": {
      "endDate": "...",
      "targetSum": 20000 
    }
}

# Получение всех счетов клиента
<METHOD> <base-url>/<path>
<STATUS>

# Получение конкретного счета клиента по номеру 
<METHOD> <base-url>/<path>
<STATUS>

# Получение всех карт счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# Создание счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# Редактирование счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# Удаление счета клиента
<METHOD> <base-url>/<path>
<STATUS>
```

**Шаблон для live-coding Spring Controller REST API:**
```java
@RestController
class AccountController {
    private int apiCallCounter = 0;
    @Autowired
    private AccountMapper dtoMapper;
    @Autowired
    private AccountRepository accountRepository;

    @RequestMapping(path = "/clients/{clientId}/accounts", method = RequestMethod.POST)
    @ResponseStatus(HttpStatus.CREATED)
    public AccountRsDto createAccount(@PathVariable("clientId") String clientId,
                                       @RequestBody AccountRqDto accountDto) {
        apiCallCounter++;
        AccountEntity accountEntity = accountMapper.mapCreateAccountRq(clientId, accountDto);
        accountEntity = accountRepository.createAccount(accountEntity);
        return accountMapper.mapGetAccountEntity(accountEntity);
    }

    @RequestMapping(path = "/clients/{clientId}/accounts/{accountId}", method = RequestMethod.PUT)
    @ResponseStatus(HttpStatus.OK)
    public AccountRsDto updateAccount(@PathVariable("clientId") String clientId,
                                       @PathVariable("accountId") String accountId,
                                       @RequestBody AccountRqDto account) {
        apiCallCounter++;
        AccountEntity accountEntity = accountMapper.mapUpdateAccountRq(clientId, accountId, accountDto);
        accountEntity = accountRepository.updateAccount(accountEntity);
        return accountMapper.mapGetAccountEntity(accountEntity);
    }

    @RequestMapping(path = "/clients/{clientId}/accounts/{accountId}", method = RequestMethod.DELETE)
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteAccount(@PathVariable("clientId") String clientId,
                              @PathVariable("accountId") String accountId) {
        apiCallCounter++;
        accountRepository.deleteAccount(clientId, accountId);
    }

    @RequestMapping(path = "/clients/{clientId}/accounts", method = RequestMethod.GET)
    @ResponseStatus(HttpStatus.OK)
    public List<AccountRsDto> getAccounts(String clientId) {
        apiCallCounter++;
        List<AccountEntity> accountsEntity = accountRepository.getAccounts(clientId);
        List<AccountDto> accountsDto = new ArrayList();
        for (AccountEntity accountEntity: accountsEntity) {
            accountsDto.add(dtoMapper.mapGetAccountEntity(accountEntity));
        }
        return accountsDto;
    }
}

interface AccountMapper {
    AccountEntity mapCreateAccountRq(String clientId, AccountDto accountDto);
    AccountEntity mapUpdateAccountRq(String clientId, String accountId, AccountDto dto);
    AccountRsDto mapGetAccountEntity(AccountEntity entity);
}

interface AccountRepository {
    AccountEntity createAccount(AccountEntity entity);
    AccountEntity updateAccount(AccountEntity entity);
    AccountEntity deleteAccount(String clientId, String accountId);
    List<AccountEntity> getAccounts(String clientId);
}
```

**Шаблон для live-coding SQL:**
```
CLIENTS (
    client_id
    ...
)

ACCOUNTS (
    account_id
    client_id
    ...
)

CARDS (
    account_id,
    card_id,
    balance,
    expire_date
    ...
)
```

**Шаблон для live-coding для GC:**
```java
interface ThreadManager {
    List<Thread> getThreads();
}

interface Thread {
    void pause();
    void resume();
    Stack getStack();
}

class Stack {
    List<StackFrame> frames;
}

class StackFrame {
    Method caller;
    List<Variable> variables;
}

class Heap {
    Map<Long, Object> objects;
}

class Object {
    boolean isVisited;
    List<Field> fields;
}

class Field {
    String name;
    Variable variable;
}

class Variable {
    boolean isPrimitive;
    long valueOrRef;
}

@RequiredArgsConstructor
class GarbageCollector {
    private final Heap heap;
    private final ThreadManager threadManager;

    void clean() {
        // ToDo:
    }

    void markHeapObjects() {
        // ToDo:
    }

    void cleanUnmarkedHeapObjects() {
        // ToDo:
    }
}
```

### 1.2. Общие вопросы по сетям и HTTP
#### Что происходит после ввода google.com в браузере?
Браузер парсит URL, определяет протокол, домен, путь, инициирует DNS-запрос.

#### Что такое DNS-резолвинг?
Процесс получения IP-адреса по доменному имени через DNS-серверы.

#### Откуда DNS берёт информацию об IP?
Из кэша, резолвера ОС, провайдера или корневых/авторитетных серверов.

#### Как формируется GET-запрос?
Браузер формирует HTTP-запрос и отправляет его по TCP/IP на полученный IP-адрес.

#### Чем отличается HTTP от HTTPS?
HTTPS использует TLS/SSL для шифрования трафика, обеспечивает конфиденциальность и целостность.

#### Как работает TLS handshake?
Клиент и сервер договариваются о параметрах шифрования, обмениваются ключами, проверяют сертификат.

#### Как устроены цепочка доверия и проверка сертификата?
Сертификат подписан доверенными центрами сертификации, клиент проверяет подписи до корневого CA.

#### Способы аутентификации в HTTP?
Basic (логин/пароль), Bearer (токены), JWT (подписанные токены).

#### Мини-задачка: Как реализовать отзыв JWT-токенов без смены ключа?
Например, хранить идентификаторы (jti) отозванных токенов в Redis и проверять их наличие при каждом запросе.

## 2. Инфраструктура
#### Что происходит при поступлении запроса в инфраструктуру банка?
Запрос проходит через балансировщик, маршрутизируется на сервисы, может попадать в kubernetes.

#### Что такое L4-балансировщик?
Распределяет трафик на уровне TCP/UDP, не смотрит на содержимое пакетов.

#### Kubernetes: что такое ingress, service, pod?
Ingress — входная точка HTTP(S), service — абстракция для доступа к подам, pod — минимальная единица развертывания контейнеров.

#### Что такое replica set?
Контроллер для поддержания заданного числа идентичных подов.

#### Как работает kube-dns?
Разрешает имена сервисов и подов внутри кластера, управляет внутренним DNS на хосте.

#### Что такое node в Kubernetes, что на ней запускается?
Node — физический или виртуальный сервер с агентом kubelet, запускает поды, каждый под стартует на определенном порту.

#### Что такое Docker-образ и как он создается?
Образ — снэпшот файловой системы и зависимостей; создаётся с помощью Dockerfile.

#### Какие есть основные команды в Dockerfile?
FROM, COPY, RUN, CMD

#### Базовые команды Linux
ls, cd, cat, grep, ps, top, docker ps, kubectl get pods.

## 3. Java
### 3.1 Реактивщина
#### Что такое синхронный и асинхронный сервер?
Синхронный обрабатывает запросы по одному, асинхронный — параллельно, не блокируя потоки.

#### Thread-per-request модель и thread pool — различия?
В первом случае на каждый запрос выделяется поток, во втором — пул потоков, которые переиспользуются.

#### Почему мы не можем создавать бесконечное количество потоков в синхронном сервере?
Каждый поток требует собственного стека памяти, контекст-переключения и ресурсов ОС.

### 3.2 Java Memory Model
#### Stack vs Heap — различия?
Stack — область памяти для локальных переменных и вызовов методов, heap — для объектов.

#### Что такое volatile?
Ключевое слово для переменных, обеспечивающее видимость изменений между потоками.

#### Что такое happens-before?
Гарантия порядка видимости изменений между потоками в JMM.

#### Race condition — что это?
Состояние, когда результат зависит от порядка выполнения потоков.

#### Garbage Collector и Stop-the-World — что это?
GC — автоматическое освобождение памяти, Stop-the-World — приостановка приложения для выполнения GC.

#### Мини-задача: простая модель обхода объектов для GC
Опишите алгоритм обхода объектов для имитации GC.
Использовать граф объектов, начиная с корневых ссылок, помечать достижимые объекты, затем удалять недостижимые.

**Шаблон для live-coding:**
```java

```

## 4. Kotlin
#### Nullable-типизация, пример?
В Kotlin переменная типа String? может быть null, а String — нет.

#### Что такое data-класс?
Класс с автоматической генерацией equals, hashCode, toString.

#### Что такое extension-функции?
Extension — добавление методов к существующим типам

## 5. Spring
#### Чем отличается Spring от Spring Boot?
Spring Boot — надстройка для облегчения конфигурации и запуска приложений со встроенным сервером.

#### Что такое автоконфигурация и conditional annotations?
Автоматическая настройка компонентов на основе зависимостей; conditional-аннотации позволяют создавать бины при определённых условиях.

#### Что такое контекст Spring, бины, @Component, @Configuration, @Bean?
Контекст — контейнер для бинов, @Component — аннотация для автоматического создания бина, @Configuration/@Bean — явное определение бинов.

#### Какие бывают скопы бинов?
Singleton, Prototype, Request, Session, Application и WebSocket.
_**Приведите пример, зачем нужно Prototype.**_

#### Мини задача: программно создать несколько бинов одного сервиса с разными параметрами?
**Шаблон для live-coding:**
```
# Содержимое файла application.yaml:
my-app.myparam:
    - test
    - test1
    - test4

# Класс параметров
@ConfigProperties(prefix="my-app")
class Properties{
    private List<String> myparam = new ArrayList()
}

# Класс конфигурации
@Configuration
class MyConfig{
    @Bean
    public MyService myService1(){
          return new MyService("test")
    }
    
    @Bean
    public MyService myService1(){
          return new MyService("test1")
    }
    
    @Bean
    public MyService myService1(){
          return new MyService("test4")
    }
}
```

**Ожидаемое решение:**
```
@Bean
public static BeanDefinitionRegistryPostProcessor myServiceRegistrar(Properties props) {
    return new BeanDefinitionRegistryPostProcessor() {
        @Override
        public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
            props.getServices().stream()
                .map(name -> Map.entry(
                    "myService_" + name,
                    BeanDefinitionBuilder.genericBeanDefinition(MyService.class)
                        .addConstructorArgValue(name)
                        .getBeanDefinition()
                ))
                .forEach(entry -> registry.registerBeanDefinition(entry.getKey(), entry.getValue()));
        }
    };
}
```

#### Как работают транзакции в Spring?
С помощью аннотации @Transactional, управление commit/rollback через прокси

#### Как работает атрибут propagation в @Transactional и почему он важен?
propagation определяет, как метод транзакции должен выполняться в контексте существующей транзакции. 
Он определяет, будет ли метод использовать существующую транзакцию, всегда создавать новую, или выполнять метод без транзакции. 
Это важно для правильно обработки логики взаимодействия между методами в многослойных приложениях.
* REQUIRED (по умолчанию) — использовать существующую или создать новую транзакцию.
* REQUIRES_NEW — всегда создавать новую транзакцию, при этом текущая приостанавливается.
* SUPPORTS — использовать транзакцию, если она есть, иначе работать без неё.
* Другие: MANDATORY, NEVER, NOT_SUPPORTED, NESTED.

#### Что произойдет, если метод с @Transactional выбросит необработанное исключение?
По умолчанию, необработанное непроверяемое исключение (например, RuntimeException) приведет к откату транзакции. 
Проверяемые исключения (Exception) не вызовут откат, если только не указаны в rollbackFor.

## 6. Тестирование
#### Мок, стаб, spy — различия?
Mock — имитация поведения, stub — возвращает предопределённые ответы, spy — обёртка вокруг настоящего объекта.

#### Что такое Testcontainers?
Библиотека для запуска контейнеров (например, PostgreSQL) в тестах.

## 7. Реляционные базы данных
#### Что такое индекс, как он устроен?
Структура данных (обычно B-дерево), ускоряющая поиск по столбцу.

#### Причины медленной работы таблицы — как найти?
Использовать EXPLAIN, анализировать индексы, статистику, блокировки.

#### Издержки от большого количества индексов?
Замедление записи и обновления, увеличение размера БД.

#### Что такое партиционирование?
Деление большой таблицы на физические части для улучшения производительности.

#### Как оптимизировать место на диске у сервера БД?
VACUUM очищает "мертвые" строки, REINDEX пересоздаёт индексы для повышения производительности.

## 8. Нереляционные базы данных
#### Главные различия реляционных и нереляционных БД?
Реляционные используют таблицы и SQL, NoSQL — документы, ключ-значение, графы, проще масштабируются.

#### Базовые понятия MongoDB?
Коллекция, документ, BSON, индекс, replica set, sharding.

#### Как осуществляется индексация в MongoDB?
Индексы создаются по полям документов для ускорения поиска, поддерживаются составные и уникальные индексы.

#### Read/Write concern
* **Write Concern** определяет, сколько реплик должно подтвердить запись, чтобы она считалась успешной. 
Например, w:1 — подтверждение от одного узла, w:majority — от большинства.
* **Read Concern** задаёт, насколько “свежими” должны быть читаемые данные. 
Например, local — данные с текущего узла, majority — только подтверждённые большинством реплик.
Используются для балансировки между производительностью и надежностью данных.

## 9. Kafka
#### Что такое топик, брокер, продюсер, консюмер, партиция?
Топик — поток сообщений, брокер — сервер Kafka, продюсер — пишет сообщения, консюмер — читает, партиция — раздел топика.

#### Гарантии доставки в Kafka?
At most once, at least once, exactly once (в зависимости от конфигурации).

#### Задача: сколько партиций нужно?
Количество партиций >= количество инстансов-читателей для максимального параллелизма.

#### Split-brain в Kafka — что это?
Ситуация, когда кворум брокеров разделился, может привести к потере или дублированию данных.
