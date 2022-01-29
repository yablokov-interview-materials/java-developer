### Spring

#### Чем отличается DI от IoC?

Inversion of Control (инверсия управления) — это некий абстрактный принцип, набор рекомендаций для написания слабо
связанного кода. Суть которого в том, что каждый компонент системы должен быть как можно более изолированным от других,
не полагаясь в своей работе на детали конкретной реализации других компонентов.

Dependency Injection (внедрение зависимостей) — это одна из реализаций этого принципа (помимо этого есть еще Factory
Method, Service Locator).

#### Циклические зависимости в Spring

Когда контекст Spring загружает все bean, он пытается создать их в порядке, необходимом для их полноценной работы.

При циклической зависимости Spring не может решить, какой из bean должен быть создан первым, поскольку они зависят друг
от друга. В этих случаях Spring вызовет исключение `BeanCurrentlyInCreationException` при загрузке контекста.

Это может произойти в Spring при использовании constructor dependency injection, т.е. используете передачу зависимостей
через конструктор bean'а.

Способы решения проблемы:

<details><summary markdown="span">Использовать constructor injection с аннотацией @Lazy</summary>

Простой способ разорвать цикл - сказать Spring лениво инициализировать один из bean, то есть вместо полноценной
инициализации bean он создаст proxy для его внедрения в другой bean. Внедренный bean будет полностью инициализирован
только тогда, когда он впервые понадобится.

```java
@Component
public class CircularDependencyA {
  private CircularDependencyB circularDependencyBean;

  @Autowired
  public CircularDependencyA(@Lazy CircularDependencyB circularDependencyBean) {
    this.circularDependencyBean = circularDependencyBean;
  }
}

@Component
public class CircularDependencyB {
  private CircularDependencyA circularDependencyBean;

  @Autowired
  public CircularDependencyB(CircularDependencyA circularDependencyBean) {
    this.circularDependencyBean = circularDependencyBean;
  }
}
```

</details>

<details><summary markdown="span">Использовать Setter/Field injection</summary>

Одним из самых популярных обходных путей - это использование других типов dependency injection. При внедрении
зависимостей поля (field) или setter'ы. При таком подходе зависимости не внедряются до тех пор, пока они не потребуются.

```java
@Component
public class CircularDependencyA {
    private CircularDependencyB circularDependencyBean;

    @Autowired
    public void setCircB(CircularDependencyB circularDependencyBean) {
        this.circularDependencyBean = circularDependencyBean;
    }

    public CircularDependencyB getCircularDependencyBean() {
        return circularDependencyBean;
    }
}

@Component
public class CircularDependencyB {
  private CircularDependencyA circularDependencyBean;

  @Autowired
  public void setCircB(CircularDependencyA circularDependencyBean) {
    this.circularDependencyBean = circularDependencyBean;
  }

  public CircularDependencyA getCircularDependencyBean() {
    return circularDependencyBean;
  }
}
```

</details>

<details><summary markdown="span">Использовать PostConstruct</summary>

Другой способ разорвать цикл - внедрить зависимость с помощью `@Autowired` в один из bean, а затем использовать метод,
аннотированный с помощью `@PostConstruct`, для установки другой зависимости.

```java
@Component
public class CircularDependencyA {
    @Autowired
    private CircularDependencyB circularDependencyBean;

    @PostConstruct
    public void init() {
      circularDependencyBean.setCircularDependencyBean(this);
    }

    public CircularDependencyB getCircularDependencyBean() {
        return circularDependencyBean;
    }
}

@Component
public class CircularDependencyB {
  private CircularDependencyA circularDependencyBean;

  public void setCircularDependencyBean(CircularDependencyA circularDependencyBean) {
    this.circularDependencyBean = circularDependencyBean;
  }
}
```

</details>

<details><summary markdown="span">Использование ApplicationContextAware и InitializingBean</summary>

Если один из bean реализует `ApplicationContextAware`, то он имеет доступ к контексту Spring и может извлечь из него
другой bean. Реализуя `InitializingBean`, мы указываем, что этот bean должен выполнить некоторые действия после того,
как все его свойства были установлены. В этом случае мы хотим вручную установить нашу зависимость.

```java
@Component
public class CircularDependencyA implements ApplicationContextAware, InitializingBean {
  private ApplicationContext context;
  private CircularDependencyB circularDependencyBean;

  public CircularDependencyB getCircularDependencyBean() {
    return circularDependencyBean;
  }

  @Override
  public void setApplicationContext(ApplicationContext context) throws BeansException {
    this.context = context;
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    circularDependencyBean = context.getBean(CircularDependencyB.class);
  }
}

@Component
public class CircularDependencyB {
  private CircularDependencyA circularDependencyBean;

  @Autowired
  public void setCircA(CircularDependencyA circularDependencyBean) {
    this.circularDependencyBean = circularDependencyBean;
  }
}
```

</details>

Подробнее см. [здесь](https://www.baeldung.com/circular-dependencies-in-spring).

#### @Autowired @Qualifier @Conditional

Данные аннотации предназначены для управления процессом внедрения зависимостей в Spring:
- [@Autowired](https://www.baeldung.com/spring-autowire):
    - включает автоматическое внедрение зависимостей для конструктора/поля/метода (там, где навешана);
    - выбор bean кандидата для внедрения происходит на основании типа (т.е. по интерфейсу/классу), при этом может быть
      сгенерировано исключение в следующих ситуациях:
        - если не был найден ни один подходящий кандидат:
            - `NoSuchBeanDefinitionException`;
        - если было найдено более одного подходящего кандидата (аннотация `@Qualified` помогает в решении данной проблемы):
            - `NoUniqueBeanDefinitionException`;
            - исключением является ситуация, когда имя поля/сеттера полностью соответствует имени одного из bean'ов кандидатов
              (в этом случае, поведение будет идентично тому, как если бы мы явно использования аннотацию `@Qualified` с
              указанием имени бина, равному имени поля/сетера);
- [@Qualifier](https://www.baeldung.com/spring-qualifier-annotation):
    - указывает идентификатор/имя bean, который необходимо использовать при внедрении зависимости;
    - необходима в ситуации, когда имеется несколько подходящих по типу (т.е. по интерфейсу/классу) bean кандидатов для
      внедрения зависимости;
- [@Conditional](https://www.baeldung.com/spring-conditional-annotations):
    - указывает на необходимость создания bean на основании условия, указанного в аннотации:
        - наличие property и/или определенное его значение:
            - `@ConditionalOnProperty`;
        - соответствие регулярному значению:
            - `@ConditionalOnExpression`;
        - наличие зарегистрированного bean определенного класса:
            - `@ConditionalOnBean`;
        - наличие определенного класса в classpath:
            - `@ConditionalOnClass`;
        - соответствие определенной версии java:
            - `@ConditionalOnJava`;
        - когда приложение стартует из war (для приложений со встроенным сервером приложений всегда будет `false`):
            - `@ConditionalOnWarDeployment`;
        - соответствие кастомным условиям:
            - `@Conditional`, указывающий на кастомную реализацию интерфейса `Condition` (подробнее см. [здесь](https://russianblogs.com/article/21241481285/)).

#### Расскажите про Transaction

В Spring используется аннотация `@Transactional` для оборачивания метода в транзакцию используемой БД.

Spring для выполнения этого либо используем паттерн проектирования proxy (создает наследника данного класса и проксирует
все вызовы ему, но предварительно оборачивая их в транзакции), либо манипулирует с байткодом классов
(через cglib) для управления созданием, commit и rollback транзакций. В случае использования proxy, Spring
игнорирует `@Transactional` в рамках internal method calls (вызовов внутри самого класса).

Оборачивание в транзакцию выглядит примерно следующим образом:

```java
createTransactionIfNecessary();
try {
    callMethod();
    commitTransactionAfterReturning();
} catch (exception) {
    completeTransactionAfterThrowing();
    throw exception;
}
```

#### Расскажите про Transaction Propagation (распространение транзакций)

Transaction propagation выставляется в виде атрибутов аннотации:

```java
@Transactional(propagation = Propagation.<значение>)
public void someMethod() {
    // some code
}
```

Типы transaction propagation:

- `REQUIRED` - значение по умолчанию:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет использована существующая транзакция;
        - если транзакции нет, то будет создана новая;
- `SUPPORTS`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет использована существующая транзакция;
        - если транзакции нет, то метод выполнится вне транзакции;
- `MANDATORY`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет использована существующая транзакция;
        - если транзакции нет, то будет сгенерировано исключение;
- `NEVER`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет сгенерировано исключение;
        - если транзакции нет, то метод выполнится вне транзакции;
- `NOT_SUPPORTED`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет выполнен suspend (транзакция будет прервана), после чего метод выполнится вне транзакции;
        - если транзакции нет, то метод выполнится вне транзакции;
- `REQUIRES_NEW`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет выполнен suspend (транзакция будет прервана), после чего будет создана новая;
        - если транзакции нет, то будет создана новая;
- `NESTED`:
    - Spring проверяет, существует ли активная транзакция:
        - если транзакция существует, то будет выполнено создание save point (точки отката внутри транзакции в случае
          получения исключения внутри вложенной транзакции);
        - если транзакции нет, то будет создана новая.

Подробнее см. [здесь](https://www.baeldung.com/spring-transactional-propagation-isolation).

#### Сколько существует уровней изоляции транзакций

- `DEFAULT`:
    - уровень изоляции транзакций, используемый СУБД по умолчанию;
- `READ_UNCOMMITTED`:
    - допускает "dirty read";
- `READ_COMMITTED`:
    - предотвращает "dirty read";
- `REPEATABLE_READ`:
    - предотвращает "dirty read" и "non-repeatable read";
- `SERIALIZABLE`:
    - предотвращает "dirty read" и "non-repeatable read" и "phantom reads".

#### Bean scopes
Из коробки Spring поддерживает два базовых scope:
- Singleton:
    - bean создаётся в единственном экземпляре на весь контекст и ссылка на этот экземпляр подсовывается каждому другому
      бину, который просит эту зависимость;
- Prototype:
    - создаётся отдельный для каждого другого бина, который просит его в зависимость.

Spring предоставляет ещё три scope, которые доступны только при использовании web специфичных ApplicationContext:
- Request:
    - bean создаётся для каждого HTTP request и уничтожается после завершения обработки;
- Session:
    - bean создаётся для HTTP сессии и уничтожается, после её завершения;
- GlobalSession:
    - bean создаётся на время существования глобальной сессии всего приложения с Portlets.

#### Опишите Spring Bean Lifecycle

Spring bean factory контролирует создание и уничтожение bean'ов. Для выполнения некоторого пользовательского кода фабрика
компонентов предоставляет методы обратного вызова, которые можно условно разделить на две группы:
- Post-initialization callback methods;
- Pre-destruction callback methods.

Spring framework предоставляет следующие варианты для контроля жизненного цикла бина:
- наследование bean от интерфейсов:
    - `InitializingBean` - метод `afterPropertiesSet()`;
    - `DisposableBean` - метод `destroy()`;
- использование аннотаций для методов:
    - `@PostConstruct` - метод будет вызываться после создания компонента с использованием конструктора по умолчанию и
      непосредственно перед тем, как его экземпляр будет возвращен запрашивающему объекту;
    - `@PreDestroy` - метод вызывается непосредственно перед тем, как bean будет уничтожен bean container'ом;
- указание методов в xml-файле конфигурации:
    - `init()` - аналог `@PostConstract`;
    - `destroy()` - аналог `@PreDestroy`;
- `*Aware` interfaces for specific behavior:
    - `ApplicationContextAware`;
    - `BeanNameAware`;
    - ...

Подробнее см. [здесь](https://howtodoinjava.com/spring-core/spring-bean-life-cycle/).

#### выполнится PreDestroy у bean со Scope=Prototype?

Нет. В отличие от других Scope, Spring не управляет полным life cycle Prototyp bean'а. Другими словами:
- initialization callback методы lifecycle вызываются у bean;
- а destruction lifecycle callbacks не вызываются у bean.

Подробнее см. [здесь](https://docs.spring.io/spring-framework/docs/3.1.x/spring-framework-reference/html/beans.html#beans-factory-scopes-prototype).

#### Как я могу изменить бин перед помещением его в контейнер не изменяя код класса?

Воспользоваться `BeanPostProcessor`, например:

```java
public class CustomBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        // some manipulation under bean
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        // some manipulation under bean
        return bean;
    }
}
```

Подробнее см. [здесь](https://stackoverflow.com/questions/29743320/how-exactly-does-the-spring-beanpostprocessor-work).

---

#### где я могу ставить аннотацию autowired
- конструктор bean'а:
    - будет применен для аргументов;
- поля bean'а.

---

#### В чем преимущество спринг для тестирования?

Приложение, ориентированное на инъекцию зависимостей, что предоставляет большие возможности для тестирования:
- ресурсы (например, bean'ы) легко заменяются тестовыми ресурсами (например, заглушками).

---

#### Чем Mock отличается от Spy?

- mock - полная заглушка объекта:
    - вместо всех полей / методов используется заглушка;
- spy - частичная заглушка объекта:
    - для частей полей / методов используется заглушка, а остальные используются из реального объекта.

---

#### В чем преимущество spring boot?

ToDo:

---

что будет результатом работы стартера?
#### чем лучше java 11 перед 8 для контейнеров

С Java 10 JVM теперь распознает ограничения, установленные группами управления контейнерами (cgroups).
Ограничения памяти и ЦП могут использоваться для управления приложениями Java непосредственно в контейнерах, в том
числе:
- соблюдение memory limits, установленных в контейнере;
- установка available cpus в контейнере;
- установка cpu constraints в контейнере.

Также, начиная с Java 11:
- `-XshowSettings` (Container Metrics): display the system or container configuration;
- `JDK-8197867`: improve CPU calculations for both containers and JVM hotspot (see PreferContainerQuotaForCPUCount).

Подробнее см. [здесь](https://www.docker.com/blog/improved-docker-container-integration-with-java-10/).

---

#### Расскажите зачем применяется аннотация Conditional

`@Conditional` позволяет указать условия, при которых должен создаваться bean.

---

работали ли с какими нибудь ORM?
расскажите сколько существует уровней кэширования?
могу как то я сказать JVM что этот объект нужно кэшировать?
какие знаете стратегии кэширования?
может быть с какими то сложными ошибками сталкивались в работе?
что такое уровни кэширования