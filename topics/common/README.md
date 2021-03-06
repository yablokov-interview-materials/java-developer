## Общие вопросы

#### Если нажать поиск в гугл, что происходит под капотом?
<details><summary markdown="span">Ответ</summary>

ToDo: развернутый большой ответ

</details>

---

#### Расскажите про клиент-серверную архитектуру
Ответ:
Архитектура "Клиент-Сервер" предусматривает разделение сетевой нагрузки между потребителем услуг (*клиентом*) и
поставщиком услуг (*сервером*) на разных компьютерах сети, каждый из которых работает независимо от других.

<details><summary markdown="span">Типы реализаций</summary>:

1. **Одноуровневая архитектура (1-Tier)**: клиенты через интерфейс взаимодействуют с общим <u>сервером баз данных</u>
   или <u>файловым сервером</u> (только предоставление данных);
2. **Двухуровневая архитектура (2-Tier)**: клиенты через интерфейс взаимодействуют с общим <u>сервером приложений с
   БД</u>:
1. *fat client thin server* - данные хранятся на сервере, а логика их обработки и бизнес-логика - на клиенте;
2. *thin client fat server* - на сервере хранятся и данные, и логика их обработки, и бизнес-логика;
3. **Трёхуровневая архитектура (3-Tier)**: клиенты через интерфейс взаимодействуют с общемим сервером, который
   использует результат взаимодействия с сервером БД.
4. **Многоуровневая архитектура (N-Tier)**: клиенты через интерфейс взаимодействуют с одним из несколько серверов
   приложений, использующих результаты работы друг друга, а также данные от различных серверов БД, файловых серверов и
   других видов серверов.

</details>

<details><summary markdown="span">Практические примеры</summary>

* веб-сайт;
* автомобильный навигатор;
* корпоративная сеть (нпр. почтовый сервер);
* и др.

</details>

<details><summary markdown="span">Преимущества</summary>

* *Масштабируемость* - количество клиентов и серверов можно изменять независимо друг от друга;
* *Централизованность* - управление и данные сосредоточены на центральном сервере;
* *Информационная безопасность* - ресурсы общего пользования администрируются централизованно;
* *Производительность* - использование выделенного сервера повышает скорость работы ресурсов общего пользования;

</details>

<details><summary markdown="span">Недостатки</summary>

* *Перегрузку трафика в сети* - когда большое число клиентов одновременно запрашивают одну услугу на сервере, то число
  запросов может создать перегрузку в сети;
* *Наличие единой точки отказа* в небольших сетях с одним сервером. Если он отказывает, все клиенты остаются без
  обслуживания;
* *Превышение пределов ресурсов сервера*, когда новые клиенты, запрашивающие услугу, остаются без обслуживания. В таких
  случаях, требуется срочное расширение ресурсов сервера;
* Потенциально специфический интерфейс клиента, для использования которого требуются дополнительные настройки.

</details>

---

#### Что такое XML, как он связан с XSD?

**XML** расшифровывается как Extensible Markup Language. В качестве языка разметки он помогает создавать документы в
формате, понятном как людям, так и компьютеру. Он был разработан World Web Consortium (W3C). XML хранит данные в
текстовом формате и не зависит от платформы.

**XSD** — это язык описания структуры XML документа. Его также называют XML Schema. При использовании XML Schema XML
парсер может проверить не только правильность синтаксиса XML документа, но также его структуру, модель содержания и типы
данных.

---

#### Чем удобен JSON, есть ли у него инструменты для проведения валидации?

JSON - простой, основанный на использовании текста, способ хранить и передавать структурированные данные. С помощью
простого синтаксиса вы можете легко хранить все что угодно, начиная от одного числа до строк, массивов и объектов, в
простом тексте. Также можно связывать между собой массивы и объекты, создавая сложные структуры данных.

JSON имеет следующие преимущества:
- он компактен по сравнению с XML;
- его предложения легко читаются и составляются как человеком, так и компьютером;
- его легко преобразовать в структуру данных для большинства языков программирования (числа, строки, логические
  переменные, массивы и так далее).

Для валидации JSON существует стандарт [JSON Schema](http://json-schema.org/) (аналог XSD для XML).

---

#### Что такое сериализация и десериализация?
**Сериализация** - процесс преобразования структуры данных в линейную последовательность байтов для дальнейшей передачи
или сохранения.  
В Java, согласно спецификации Java Object Serialization существует два стандартных способа сериализации:
стандартная сериализация, через использование интерфейса `java.io.Serializable` и «расширенная» сериализация -
`java.io.Externalizable`.

**Десериализация** - это процесс восстановления сериализованного (см. "сериалиация") объекта из массива байтов.

---

#### Какие знаете HTTP методы?
- `GET` - получение ресурса (без изменения состояния);
- `HEAD` - получение информации о ресурсе без получения body (аналог `GET`, но без получения body);
- `PUT` - создание ресурса;
- `POST` / `PATCH` - обновление ресурса;
- `DELETE` - удаление ресурса;
- `OPTIONS` - получить список поддерживаемых методов ресурса.

---

#### Зачем нужны cookies?
Cookies -небольшой фрагмент данных, отправленный веб-сервером и хранимый на компьютере пользователя.
Веб-клиент (обычно веб-браузер) всякий раз при попытке открыть страницу соответствующего сайта пересылает этот фрагмент
данных веб-серверу в составе HTTP-запроса.
Применяется для сохранения данных на стороне пользователя, на практике обычно используется для:
- аутентификации пользователя;
- хранения персональных предпочтений и настроек пользователя;
- отслеживания состояния сеанса доступа пользователя;
- сведения статистики о пользователях.

---

#### идентификация / аутентификация / авторизация
- идентификация:
    - процесс распознавания пользователя по его идентификатору (проверка на наличие пользователя с таким логином);
- аутентификация:
    - процедура проверки подлинности, доказательство, что пользователь именно тот, за кого себя выдает (проверка на
      достоверность логина и пароля);
- авторизация:
    - проверка, либо предоставление определённых пользователю (проверка наличия прав доступа на выполнение той или иной
      операции прошедшему аутентификацию пользователю).

Каждый их последующих шагов включает в себя предыдущий:
- аутентификация включает в себя идентификацию;
- авторизация включает в себя аутентификацию, как следствие, транзитивно включает и идентификацию.

---

#### REST vs SOAP
REST — это архитектурный стиль. SOAP — это формат обмена сообщениями.

Специфика SOAP — это формат обмена данными. С SOAP это всегда SOAP-XML, который представляет собой XML, включающий:
- Envelope (конверт) – корневой элемент, который определяет сообщение и пространство имен, использованное в документе,
- Header (заголовок) – содержит атрибуты сообщения, например: информация о безопасности или о сетевой маршрутизации,
- Body (тело) – содержит сообщение, которым обмениваются приложения,
- Fault – необязательный элемент, который предоставляет информацию об ошибках, которые произошли при обработке сообщений. И запрос, и ответ должны соответствовать структуре SOAP.

Специфика REST — использование HTTP в качестве транспортного протокола. Он подразумевает наилучшее использование функций,
предоставляемых HTTP — методы запросов, заголовки запросов, ответы, заголовки ответов итд.

- SOAP не накладывает никаких ограничений на тип транспортного протокола. Вы можете использовать либо Web протокол HTTP, либо MQ.
- REST подразумевает наилучшее использование транспортного протокола HTTP
- SOAP использует WSDL (Web Services Description Language) — язык описания веб-сервисов и доступа к ним, основанный на языке XML.
- REST не имеет стандартного языка определения сервиса. Несмотря на то, что WADL был одним из первых предложенных стандартов,
  он не очень популярен. Более популярно использование Swagger или Open API.

---

#### RESTful api и JSON-pure api в чем отличие?

RESTful api и JSON-pure - это архитектурные стили взаимодействия компонентов распределённого приложения в сети
(распределенных веб-сервисов).

Не существует «официального» стандарта для RESTful веб-API. Несмотря на то, что REST не является стандартом сам по себе,
большинство RESTful-реализаций придерживаются следующих требований:
- в качестве средства интеграции использовать протокол HTTP:
    - явное указание ресурсов, над которыми выполняется действие (см. использование методов) в рамках URL;
    - явное использование методов HTTP в качестве глаголов (действия над ресурсом):
        - GET - для получения данных без изменения их состояния;
        - PUT - для добавления (создания/вставки) данных;
        - POST - для обновления данных;
        - DELETE - для удаления данных;
- в качестве структуры для передачи данных используетя JSON (реже XML);
- по возможности приложение не должно иметь состояния;
- отображает структуру папок как URls.

JSON-pure API
- использует только один метод для передачи данных — обычно POST для HTTP и SEND в случае использования Web Sockets;
- механизм передачи и содержимое запроса полностью независимы:
    - все ошибки, предупреждения и данные передаются в теле запроса, в формате JSON;
- механизм передачи и содержимое ответа полностью независимы:
    - все ошибки, предупреждения и данные передаются в теле ответа, в формате JSON;
    - используется лишь один код ответа, чтобы подтвердить успешную передачу, обычно это 200 ОК.

Из таких концепций JSON-pure API вытекают очень важные следствия:
- взаимодействие легко перенести на любой канал связи, например, HTTP/S, WebSockets, XMPP, telnet, SFTP, SCP, or SSH:
    - в связи с тем, что вся логика (включая ошибки) находится именно в теле запроса/ответа, а не в заголовках
      используемых сетевых протоколов.

---

#### Расскажите про CRUD
CRUD - аббревиатура от:
- Create - создать;
- Read - прочитать;
- Update - обновить;
- Delete - удалить.

Это не что иное, как основные операции для подавляющего большинства данных.

CRUD нужен для того, чтобы разделить программный код на отдельные части, каждая из которых отвечает за свои действия.

---

#### Как согласно CAP теореме я должен выбирать элементы для front и back систем (Базы данных, системы резервирования и проч)?

<details><summary markdown="span">Теорема CAP</summary>

Теорема CAP (теорема Брюера) - эвристическое утверждение о том, что в любой реализации распределённых вычислений
возможно обеспечить не более двух из трёх следующих свойств:
- согласованность данных (англ. consistency) — во всех вычислительных узлах в один момент времени данные не противоречат
  друг другу;
- доступность (англ. availability) — любой запрос к распределённой системе завершается корректным откликом, однако без
  гарантии, что ответы всех узлов системы совпадают;
- устойчивость к разделению (англ. partition tolerance) — расщепление распределённой системы на несколько изолированных
  секций не приводит к некорректности отклика от каждой из секций.

</details>

Согласно концепции из теоремы CAP best-practice можно обозначить следующим образом:
* фронтальные системы должны быть AP (доступна и устойчива к разделению);
* back системы должны быть CP (консистентна и устойчива к разделению).

---