# Техническое интервью Java-разработчика

## 1. Вступление

### 1.1. Знакомство с кандидатом

Добрый день.

Сегодня у нас техническая часть собеседования.  
Меня зовут <your_name> (если вы не единственный собесущующий, то также представьте остальных).

Как вам комфортнее будет общаться: на "вы" или на "ты"?

Расскажите, пожалуйста, о себе.  
В частности, интересуют наиболее релевантные для вас технологии и проекты, в рамках которых вы с ними взаимодействовали.

---

Подсказка:

* посмотреть в резюме кандидата ключевую информацию (в случае необходимости направлять рассказ по важным для вас
  пунктам);
* по ходу рассказа можете:
    * либо запоминать для себя основные моменты, по которым в дальнейшей части сценария собеседования задавать вопросы;
    * либо задавать наводящие вопросы, раскрывающие техническую составляющую той функциональности, о которой
      рассказывает
      человек, чтобы выяснить реальный уровень понимания и вовлеченности человека в реализацию этой функциональности;
* в ходе этого этапа желательно понять наиболее релевантные темы и технологии для человека.

### 1.2. Введение кандидата в структуру собеседования

Наше interview построено следующим образом:

* не будем спрашивать технические вопросы напрямую (это слишком нудно);
* вместо этого, мы с нуля спроектируем сервис;
* после чего попробуем найти у устранить его узкие места.

Для более продуктивной комуникации, а также чтобы вам не держать некоторые наши вопросы в голове,  
предлагаю перейти по ссылке, где мы будем отражать ключевую часть некоторых вопросов:

* [live-coding](https://code.yandex-team.ru/)

## 2. Практическое задание по REST API

Представьте себе, что мы проектируем сервис некоторого вымышленного банка,
который хочет предоставить своим партнёрам возможность интеграции с его системой через REST API.

Цель интеграции по REST API заключается в предоставлении API для
создания, редактирования и получения счетов и карт клиента.

**Шаблон для live-coding REST API:**

```
# API для получения списка счетов (accounts) клиента банка. 
# Это могут быть как вклады, расчетные счета, накопительные счета

# иерархия сущностей:
клиенты (clients) -> счета (accounts) -> карты (cards)

base-url = https://bank.test

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

# body
{
  "currency": "RUB",
  "type": "SAVINGS", 
  "branchCode": "7701",
  "initialDeposit": 10000.00
}

# Редактирование счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# Удаление счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# Создание карты счета клиента
<METHOD> <base-url>/<path>
<STATUS>

# body
{
  "type": "DEBIT",
  "design": "CLASSIC",
  "deliveryAddress": {
    "country": "RU",
    "city": "Москва",
    "street": "ул. Арбат, д. 15",
    "postalCode": "119019"
  },
  "holderName": "IVAN IVANOV"
}
```

## 3. Flow запроса из браузера до нашего сервера приложений

Теперь когда мы спроектировали REST API, перейдем к вызову одного из наших endpoint'ов.

Допустим мы вставили в адрессную строку браузера следующий URL:  
https://bank.test/api/v1/clients/{clientId}/accounts/{accountId}

Как браузер поймет куда отправлять запрос?

___

**_Ожидания от кандидата по данному пункту:_**

* укажет какая часть URL используется для маршрутизации запроса (host);
* расскажет как host преобразуется в IP (в частности, хочется услышать общие знания по DNS):
    * для резолвинга используются DNS сервера;
    * откуда бурутся адреса DNS серверов (в идеале услышать что-то про DHCP и прочее);
* укажет куда ведет полученный IP (с учетом предположения, что наш сервис развернут в нескольких экземплярах):
    * MVP: укажет на балансировщик;
    * идеально: информация про VRRP (или альтернативу) + NAT + LoadBalancer;
* расскажет на какой port пойдет запрос (443);
* расскажет про отличия HTTPS от HTTP:
    * информация про TLS handshake:
        * информация про цепочки сертификатов, и как они проверяются;
        * информация про сессионные ключи шифрования, и откуда они берутся;
* расскажет про то, как в K8S добраться до нашего сервиса:
    * MVP: ingress -> service -> pod
    * идеально: также услышать общее понимание того, как в k8s это все работает (локальный DNS с настройкой linux
      resolv.conf и прочее).

___

## 4. Практическое задание по Spring Controller

HTTP запрос прилетел в наш сервис на Spring:

* чем Spring отличается от Spring Boot;
* как в Spring помечается класс, который принимает HTTP запросы?
* как помечаются его методы, принимающие HTTP запросы?

### 4.1. Конкурентность в рамках Controller

Предположим, что у нас имеется следующий Controller для обработки запросов (см. код из шаблона ниже).

Для подсчета метрики общего количества вызовов endpoint'ов (допустим, для мониторинга) мы решили завести
специальный счетчит - apiCallCounter.

Правильно ли мы его завели?
Если нет, то что сделано некорректно?
Можете, пожалуйста, поправить код так, чтобы все работало корректно.

___

**_Ожидания от кандидата по данному пункту:_**

* объяснит почему здесь будет проблема, во время объяснения через доп вопросы, чтобы узнать понимает ли он:
    * что это Controller имеет Spring scope=Singleton, поэтому счетчик общий на все вызовы;
    * что HTTP запросы будут обрабатываться параллельно;
    * что для параллельной обработки используется pool потоков:
        * также можно спросить его почему бы не создавать поток на горячую, зачем иметь ограниченный в размере пул;
        * в процессе объяснения узнать у человека что за ресурсы потребляет поток (хочется услышать, что помимо CPU еще
          и Stack - потом это пригодится в вопросе про GC);
    * что поток от начала до конца будет заниматься обработкой HTTP запроса, включая ожидаение на блокирующие IO
      вызовы (в классической синхронной модели сервера приложений).

___

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
        for (AccountEntity accountEntity : accountsEntity) {
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

### 4.2. Особенности Spring в рамках Controller

Как в примере кода выше (без его изменения) сделать так, чтобы:

* при запуске на локали AccountRepository выдавал заглушечные значения;
* при запуске на стенде AccountRepository обращался к БД.

---

В ответе ожидается информация про две реализации данного интерфейса (в виде двух Spring Bean'ов), нужная из которых
выбирается либо через Spring Profile, либо через OnCondition.

### 5. Практическое задание по SQL

Перейдем к реализации AccountRepository, которая обращается к БД.

Предположим, что нам пришел запрос на удаление счета клиента:

* как нам атомарно удалить все связанные с ним сущности?
    * хотим изежать ситуации, когда мы удалили карточку, а на удалении клиента вылетела ошибка (а карточки уже нет);
* допустим мы из Controller вызвали метод класса AccountRepository, который не помечен аннотацией @Transactional, а он
  уже внутри себя вызвал соседний метод, но с данной аннотацией. Вызовы из данного метода будут обернуты в транзакцию?

---

**_Ожидания от кандидата по данному пункту:_**

* услышать информацию про @Transactional
* услышать информацию:
    * про создаваемый Spring'ом proxy-класс (обертку) над наим Bean'ом;
    * про то, что вызова из неаннотированного метода данного класса другого аннотируемого (@Transactional) метода этого
      же класса не приведет к созданию транзакции;
    * про self-inject (или иной вариант решения проблемы);

---

Предположим, у нас имеются следующие таблицы:

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
    account_id
    card_id
    balance
    expire_date
    ...
)
```

Давайте напишем SQL запрос на получение всех карт клиента (т.е. у нас имеется client_id)

---

**_Ожидания от кандидата по данному пункту:_**

```sql
SELECT *
FROM CLIENTS
         JOIN ACCOUNTS ON CLIENTS.client_id = ACCOUNTS.client_id
         JOIN CARDS ON ACCOUNTS.account_id = CARDS.account_id
WHERE CLIENTS.client_id = :client_id;
```

---

Теперь, когда мы написали SQL запрос:

* На что обратить, если он отрабатывает медленно? (предположим, что он написан правильно)
* Если дело в индексах, то давай пометим в каждой из таблиц на какие столбцы нужно навесить индекс в рамках текущего SQL
  запроса (т.е. какие важны именно в этом запросе)
* Почему бы нам не навесить индексы на все столбцы, чтобы не задумываться каждый раз касательно индексируемых столбцов?

---

**_Ожидания от кандидата по данному пункту:_**

* услышать информацию про индесы;
* услышать, что нужны индексы на следующие поля:
    * CLIENTS.client_id - используется в WHERE
    * ACCOUNTS.client_id - используется в JOIN с CLIENTS
    * CARDS.account_id - используется в JOIN с ACCOUNTS
* услышать информацию о том, что индексы:
    * замедляют операции записи (т.к. обновляют файл индексов);
    * увеличивают объем занимаемого на диске места (т.к. хранят информацию в файле индексов).

---

Со временем в нашей табличке стало очень много просрочившихся карточек (см. CARDS.expire_date).  
Из-за этого у нас стали замедляться любые запросы к таблице CARDS.  
Есть ли какие-то варианты решения проблемы?

---

**_Ожидания от кандидата по данному пункту:_**

* услышать информацию про:
    * partial index;
    * table partitions (или ручной аналог, т.е. самому создать архивную таблицу и вынести в неё неактуальные данные);

---

Проблемы с оптимизацией БД мы решили, но со временем у нас стало столько клиентов, что наша БД стала насколько большой,
что нам перестали помогать все предпринятые нами решения.  
Есть ли какие-либо варианты решения проблемы масштабируемости?

---

**_Ожидания от кандидата по данному пункту:_**

* услышать информацию про:
    * репликацию БД (write to master -> read only from slave)
    * HA (high availability) кластер PostgreSQL (например, за счет Patroni или Stolon):
      * поднимается несколько инстансов PostgreSQL
      * один из инстансов является лидером, остальные - реплики
      * репликация происходит за счет механизма PG - streaming replication
    * шардирование БД (штатного решения нет, можно воспользоваться, например, Citus)

---

В какой-то момент мы решили, что записи по давно неактуальным счетам и карточкам клиентов мы можем выгрузить в
архивную БД, а из основной удалить, чтобы они не занимали место в основной БД.

После переноса данных к нам пришло сопровождение БД и сказало, что почему-то место на жестком диске все еще занято.  
С чем это может быть связано?

---

**_Ожидания от кандидата по данному пункту:_**

* услышать информацию про:
  * PG реализует MVCC (Multi-Version Concurrency Control), поэтому старые данные не удаляются сами по себе
  * для удаления неактуальных данных нужно вызывать vacuum

### 5. Практическое задание по Message Brokers

ToDo:
