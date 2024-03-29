### Java Base

#### Назовите принципы ООП (с примерами)

<details><summary markdown="span">Наследование</summary>

Наследование - это механизм, который позволяет описать новый класс на основе существующего (родительского).
Свойства и функциональность родительского класса заимствуются новым классом.

Это главный механизм для повторного использования кода. Наследственное отношение классов четко определяет их иерархию.

</details>

<details><summary markdown="span">Инкапсуляция</summary>

Инкапсуляция в Java означает ограничение доступа к данным и возможностям их изменения.

В его основе лежит слово «капсула». В эту «капсулу» мы прячем какие-то важные для нас данные, которые не хотим, чтобы
кто-то менял.

</details>

<details><summary markdown="span">Полиморфизм</summary>

Полиморфизм - это возможность работать с несколькими типами так, будто это один и тот же тип.

При этом поведение объектов будет разным в зависимости от типа, к которому они принадлежат.

</details>

<details><summary markdown="span">Абстракция</summary>
Абстракция - это принцип выделение главных, наиболее значимых характеристик предмета и наоборот - отбрасывание 
второстепенных, незначительных.

Скажем, мы создаем картотеку работников компании. Для создания объектов «работник» мы написали класс `Employee`. Какие
характеристики важны для их описания в картотеке компании? ФИО, дата рождения, номер социального страхования, ИНН. Но
вряд ли в карточке такого типа нам нужны его рост, цвет глаз и волос. Компании эта информация о сотруднике ни к чему.

</details>

Подробнее см. [здесь](https://javarush.ru/groups/posts/1966-principih-obhhektno-orientirovannogo-programmirovanija).

---

#### Расскажите про Object Pools (Boolean/Short/Integer/Long/String)

Для более эффективного использования памяти, в Java используются так называемые пулы (`Boolean`, `Short`, `Integer`,
`Long` и `String`). Когда мы создаем объект, не используя оператор `new` (посредством литералов, либо с использованием
механизма autoboxing), объект помещается в пул, и в последствии, если мы захотим создать такой же объект (опять же,
без использования оператора `new`), то новый объект создан не будет, а мы просто получим ссылку на наш объект из пула,
т.е. по факту один и тот же объект будет переиспользован в нескольких местах (такое поведение допустимо для объектов
вышеописанных типов, т.к. они являются immutable).

Стоит заметить, что использование любого Pool'а за исключением `String` будет осуществляться при любой автоупаковке
(если, конечно, значение соответсвует определенному диапазону, зависящему от типа, см. autoboxing).
В случае же Pool'а String интернирование (intern) работает только для литералов.

<details><summary markdown="span">Boolean pool</summary>

Boolean pool хранит оба возможных значения, т.е. объектные Wrapper'ы для обоих примитивов (true и false).

Контрольный пример:

```java
Boolean valueFromPool1 = true; // произойдет вызов: Boolean.valueOf(true);
Boolean valueFromPool2 = true; // произойдет вызов: Boolean.valueOf(true);

System.out.println(valueFromPool1 == valueFromPool2); // true
System.out.println(true == valueFromPool1);           // true (произойдет вызов: valueFromPool1.booleanValue();)
System.out.println((Boolean) true == valueFromPool1); // true (произойдет вызов: Boolean.valueOf(true);)
```

</details>

<details><summary markdown="span">Short / Integer / Long pool</summary>

Особенность целочисленных (`Short`, `Integer` и `Long`) pool'ов состоит в том, что, по умолчанию, они хранят только
числа, которые помещаются в тип данных `byte`, т.е. числа в интервале от `-128` до `127` (который начиная с Java 7
(раньше это было захардкожено внутри классов `java.lang.Short`, `java.lang.Integer` и `java.lang.Long`, соответственно)
можно расширить с помощью опции JVM, пример для Integer: `-Djava.lang.Integer.IntegerCache.high=<size>` или
`-XX:AutoBoxCacheMax=<size>`). Для остальных чисел данных типов pool не работает.

**Контрольный пример**

```java
Integer valueFromPool1 = 127;    // произойдет вызов: Integer.valueOf(127);
Integer valueFromPool2 = 127;    // произойдет вызов: Integer.valueOf(127);
Integer valueNotFromPool1 = 128; // произойдет вызов: Integer.valueOf(128);
Integer valueNotFromPool2 = 128; // произойдет вызов: Integer.valueOf(128);

System.out.println(valueFromPool1 == valueFromPool2);       // true
System.out.println(valueNotFromPool1 == valueNotFromPool2); // false

System.out.println(127 == valueFromPool1);              // true  (произойдет вызов: valueFromPool1.intValue();)
System.out.println(128 == valueNotFromPool1);           // true  (произойдет вызов: valueNotFromPool1.intValue();)
System.out.println((Integer) 127 == valueFromPool1);    // true  (произойдет вызов: Integer.valueOf(127);)
System.out.println((Integer) 128 == valueNotFromPool1); // false (произойдет вызов: Integer.valueOf(128);)
```

</details>

<details><summary markdown="span">String pool</summary>

Выделим основные нюансы строкового пула в Java:
* строковые литералы (в одном/разных классе(ах) и в одном/разных пакете(ах)) представляют собой ссылки на один и
  тот же объект;
* строки, получающиеся сложением констант, вычисляются во время компиляции и далее смотри пункт первый;
* строки, создаваемые во время выполнения НЕ ссылаются на один и тот же объект;
* метод intern(), в любом случае, возвращает объект из пула, вне зависимости от того, когда создается строка,
  на этапе компиляции или выполнения (если на этапе выполнения, то объект сначала принудительно разместится в pool,
  а затем на него будет получена ссылка).

В контексте данных пунктов речь шла об "одинаковых" строковых литералах.

**Контрольный пример**

```java
String hello = "Hello", hello2 = "Hello";
String hel = "Hel", lo = "lo";

System.out.println("Hello" == "Hello");           // true
System.out.println("Hello" == "hello");           // false
System.out.println(hello == hello2);              // true
System.out.println(hello == ("Hel" + "lo"));      // true
System.out.println(hello == (hel + lo));          // false
System.out.println(hello == (hel + lo).intern()); // true
```

</details>

<details><summary markdown="span">Почему отсутствуют `Float` и `Double` pool</summary>

Для чисел с плавающей точкой (`Float` и `Double`) отсутствуют специфичные pool'ы вследствие особенностей хранения таких
чисел согласно стандарту [IEEE 754](https://habr.com/ru/post/112953/).
Использование для них pool'ов оказалось бы крайне неэффективно (хотя бы, в виду огромного количества чисел с плавающей
точкой, находящихся в интервале от `-128.0` до `127.0`).

</details>

---

#### Расскажите про контракт между методами equals() и hashCode()

Для сравнения двух экземпляров объектных типов в Java помимо оператора `==` также существуют такие методы как:
`equals()` и `hashCode()`, определенные на уровне класса `java.lang.Object`, который является родительским классом
для объектов Java, поэтому все Java-объекты наследуют от этих методов реализацию по умолчанию.

Метод equals(), как и следует из его названия, используется для простой проверки равенства двух объектов.
Реализация этого метода, по умолчанию, проверяет равенство ссылок двух объектов, т.е. по умолчанию,
поведение идентично работе оператора `==` за исключением того, что оператор `==` не позволяет сравнивать операнды
неприводимых друг к другу типов.

Метод `hashCode()` обычно используется для получения уникального целого числа, что на самом деле не является правдой,
т.к. результат работы данного метода - это целое число примитивного типа `int` (называемое хеш-кодом), полученное
посредством работы хэш-функции входным параметром которой является объект, вызывающий данный метод, но множество
возможных хеш-кодов ограничено примитивным типом `int` (всего `2^32` вариантов значений), а множество объектов
ограничено только нашей фантазией.

Из вышеописанного между методами `hashCode()` и `equals()` следует следующий контракт:
* если хеш-коды разные, то и объекты гарантированно разные;
* если хеш-коды равны, то входные объекты могут быть неравны (это обусловленно тем, что множество объектов мощнее
  множества хеш-кодов, т.к. множество возможных хеш-кодов ограничено размером примитивного типа `int`).

Создавая пользовательский класс следует переопределять методы `hashCode()` и `equals()`, чтобы они корректно работали
и учитывали необходимые для вас данные (поля) объекта. Кроме того, если не переопределить реализацию из `Object`, то,
например, при использовании таких объектов в `java.util.HashMap` у вас наверняка возникнут проблемы, поскольку `HashMap`
активно использует `hashCode()` и `equals()` в своей работе (подробнее см. [здесь](https://habr.com/ru/post/168195/)).

Подробнее про данные методы можно прочитать [здесь](https://ru.stackoverflow.com/questions/493576/%D0%9C%D0%B5%D1%82%D0%BE%D0%B4%D1%8B-equals-%D0%B8-hashcode).

---

#### Как следует сравнивать вещественные примитивы

Вещественные примитивы (`float` и `double`) стоит сравнивать с определенной точностью.
Например, округлять их до 6-го знака после запятой (`1E-6` для double, либо `1E-6F` для `float`), либо,
что предпочтительнее, проверять абсолютное значение разницы между ними.

**Контрольный пример**

```java
float float1 = 0.7F;
float float2 = 0.3F + 0.4F;
final float EPS = 1E-6F;

System.out.println(Math.abs(float1 - float2) < EPS); // true
```

Стоит понимать, что существуют и [другие более точные](https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/),
но при этом куда более изощренные варианты сравнения чисел с плавающей точкой.

---

#### Что будет если вызвать String#intern() у строки, которой еще нет в пуле?
Ответ: Строка будет размещена в String Pool.

Если в String Pool уже присутствовала данная строка, то будет возвращена ссылка на строку из Pool, а текущая позднее
будет "собрана" GC.

Стоит заметить, что область памяти (PermGen / Heap) размещения String Pool зависит от JDK.
Например, в рамках Oracle HotSpot JDK:
* до JDK 7 (не включая) Pool хранился в PermGen;
* [начиная с JDK 7](https://www.oracle.com/java/technologies/javase/jdk7-relnotes.html) (включая) Pool
  хранится в Heap (в области old generation).

Подробнее см. [здесь](https://www.baeldung.com/string/intern) и [здесь](https://stackoverflow.com/questions/10578984/what-is-java-string-interning).

---

#### Что хранится в Stack, Heap, PermGen / MetaSpace

<details><summary markdown="span">Stack</summary>

Стек (stack) - область ОЗУ, выделенная под поток и содержащая список элементов (стек вызовов / stack frames),
организованных по принципу LIFO (Last In First Out).

Содержимое элемента стека вызова (stack frame):
* примитивы:
    * `byte`, `char`, `boolean`, `short`, `int`, `long`, `float`, `double`;
* ссылки на объекты и кучи, используемые в методе:
    * наследники класса `Object`.

Основные опции JVM, отвечающие за размер стека:
* `-Xss` - объем выделяемый под стек памяти.

</details>

<details><summary markdown="span">Heap</summary>

Куча (heap) – область памяти в ОЗУ, выделенная под процесс и используемая для размещения создаваемых на этапе
выполнения объектов.

Содержимое объекта кучи:
* заголовок объекта:
    * ссылка на таблицу виртуальных методов, hashCode, ...;
* примитивы:
    * поля объектов, имеющие один из следующих типов: `byte`, `char`, `boolean`, `short`, `int`, `long`, `float`, `double`;
* ссылки на другие объекты:
    * поля наследники класса Object.

Основные опции JVM, отвечающие за размер кучи:
* `-Xms` - исходный/минимальный размер heap;
* `-Xmx` - максимальный размер heap.

</details>

<details><summary markdown="span">PermGen / MetaSpace</summary>

PermGen (до JDK 8) / MetaSpace (с JDK 8) – область памяти в ОЗУ, выделенная под процесс и используемая
ClassLoader’ами для загрузки мета-информации о классах.

Содержимое PermGen/MetaSpace:
* экземпляры классов: `Class`, `Constructor`, `Method`, `Field`, `Modifier`, ...;
* до JDK 7 (Oracle HotSpot) в PermGen хранился String Pool.

Основные опции JVM, отвечающие за размер Permanent Generation:
* `-XX:PermSize` - исходный/минимальный размер PG;
* `-XX:MaxPermSize` - максимальный размер PG.

Основные опции JVM, отвечающие за размер MetaSpace:
* `-XX:MetaspaceSize` - исходный/минимальный размер MT;
* `-XX:MaxMetaspaceSize` - максимальный размер MT.

</details>

---

#### Расскажите, как работает GC

Сборщик мусора – часть JVM, отвечающая за освобождение и перераспределение памяти,
которая занята более неиспользуемыми объектами (мусором).

Что считается мусором?
* мусор (минорная сборка мусора) – объект младшего поколения (young generation),
  недоступный/недостижимый из корней (roots), либо из ссылок на объекты старшего
  поколения (old generation);
* мусор (мажорная сборка мусора) - объект, недоступный/недостижимый из корней
  (roots).

Что является корнем?
* ссылка на объект из стека потока;
* ссылка на объект из статического поля класса.

Когда запускается сборка мусора?
* когда становится недостаточно памяти для создания нового объекта.

Большая часть сборщиков мусора опирается на «слабую гипотезу о поколениях».

В общем виде, гипотеза о поколениях гласит, что вероятность смерти как функция от возраста снижается очень быстро.
Ее приложение к сборке мусора в частности означает, что подавляющее большинство объектов живут крайне недолго.
По людским меркам, большинство даже в детский сад не пойдут. Также это означает, что чем дольше прожил объект,
тем выше вероятность того, что он будет жить и дальше.

Большинство приложений имеют распределение времен жизни объектов, схематично описываемое примерно такой кривой:

![GC Object Lifetime](images/gc_object-lifetime.png)

Подавляющее большинство объектов создаются на очень короткое время, они становятся ненужными практически сразу после
их первого использования. Итераторы, локальные переменные методов, результаты боксинга и прочие временные объекты,
которые зачастую создаются неявно, попадают именно в эту категорию, образуя пик в самом начале графика.

Далее идут объекты, создаваемые для выполнения более-менее долгих вычислений. Их жизнь чуть разнообразнее — они обычно
гуляют по различным методам, трансформируясь и обогащаясь в процессе, но после этого становятся ненужными и
превращаются в мусор. Благодаря таким объектам возникает небольшой бугорок на графике следом за пиком временных объектов.

И, наконец, объекты-старожилы, переживающие почти всех — это постоянные данные программы, загружаемые часто в самом
начале и проживающие долгую и счастливую жизнь до остановки приложения.

Все это навело разработчиков на мысль, что в первую очередь необходимо сосредотачиваться на очистке тех объектов,
которые были созданы совсем недавно. Именно среди них чаще всего находится бóльшее число тех, кто уже отжил свое,
и именно здесь можно получить максимум эффекта при минимуме трудозатрат.

Вот тут и возникает идея разделения объектов на младшее поколение (young generation) и старшее поколение (old generation).
В соответствии с этим разделением и процессы сборки мусора разделяются на малую сборку (minor GC), затрагивающую только
младшее поколение, и полную сборку (full GC), которая может затрагивать оба поколения. Малые сборки выполняются
достаточно часто и удаляют основную часть мертвых объектов. Полные сборки выполняются тогда, когда текущий объем
выделенной программе памяти близок к исчерпанию и малой сборкой уже не обойтись.

При этом разделение объектов по поколениям не просто условное, они физически размещаются в разных регионах памяти.
Объекты из младшего поколения по мере выживания в сборках мусора переходят в старшее поколение.
В старшем поколении объект может прожить до окончания работы приложения, либо будет удален в процессе одной из
полных сборок мусора.

![GC Object Generations](images/gc_object-generations.png)

Особенности работы сборки мусора зависят от конкретного GC, поэтому подробнее смотрите в серии статей
[здесь](https://habr.com/ru/post/269621/).

Сборщики мусора HotSpot VM:
1. Serial (последовательный);
2. Parallel (параллельный);
3. Concurrent Mark Sweep (CMS);
4. Garbage-First (G1);
5. ZGC (экспериментальный);
6. Epsilon.

Опция JVM для явного использования определенного GC (на примере Serial):
* `-XX:+UseSerialGC`

---

#### Назовите методы класса Object
  
<details><summary markdown="span">clone()</summary>
  
  **protected native Object clone() throws CloneNotSupportedException** - возвращает shallow (поверхностную) копию объекта. Для использования обязательно должен быть переопределен в классе. Класс должен имплементить маркерный интерфейс Cloneable, иначе метод выкинет CloneNotSupportedException. Типовая реализация:
```
  public YourClass clone() { 
    return (YourClass) super.clone(); 
}
```  
</details>

<details><summary markdown="span">equals(Object o)</summary>
  
**public boolean equals(Object o)** - возвращает результат сравнения объектов. Текущий объект сравнивается с объектом, который передается в параметр. По умолчанию в Object сравнивает ссылки через ```==```:
```
 public boolean equals(Object obj) {
     return (this == obj);
 }
 ```
Обязателен для переопределения вместе с hashCode(). Переопределенный метод должен возвращать true, если объекты логически равны.

Контракт equals:

1. рефлексивность: ```a.equals(a) == true```
2. симметричность: ```a.equals(b) == b.equals(a)```
3. транзитивность: ```if a.equals(b) == true and b.equals(c) == true -> a.equals(c) == true```
4. консистентность: повторный вызовов ```a.equals(b)``` возвращает тот же результат
5. ```a.equals(null) == false```  
  
</details>
  
  
<details><summary markdown="span">hashCode()</summary>
  
**public int hashCode()** - возвращает хэш объекта. Нужен чтобы с объектом могли работать хэш-таблицы (например, HashMap). Должен предопределяться вместе с equals().

Контракт hashCode:

1. Если ```a.hashCode()``` вызывается несколько раз, то результат должен быть одинаков
2. В определении ```hashCode()``` должны участвовать те же поля, что и в ```equals()```
2. Если ```a.equals(b) == true -> a.hashCode() == b.hashCode()```
2. Если ```a.equals(b) == false -> a.hashCode() != b.hashCode()``` - это не обязательное требование, но желательное т.к. это повышает эффективность хэш-таблиц
  
</details>

<details><summary markdown="span">getClass()</summary>
  
**public final native Class<?> getClass()** - возвращает runtime class объекта. Инстансы Class репрезентуют любой тип класса или интерфейса а также Enum, массив, аннотацию, примитивы и void. На том, что не является объектом и поэтому не может быть вызван метод ```getClass()``` объект Class можно получить с помощью литерала ```class```.  
  
</details>

<details><summary markdown="span">toString()</summary>

**public String toString()** - возвращает строковую презентацию объекта. Реализация в Object:
```
getClass().getName() + '@' + Integer.toHexString(hashCode())
```
где ```toHexString()``` - приводит хэш к шестнадцатеричному виду.

</details>

<details><summary markdown="span">wait()</summary>

**public final void wait() throws InterruptedException** - текущий поток переходит из состояния Runnable в состояние ожидания и ожидает до тех пор, пока другой поток не вызовет ```notify()``` на этом же lock-объекте. Должен вызываться только в **synchronized**-блоке: 
``` 
synchronized(lock) { 
  lock.wait(); 
}
 ```  
В перегруженные методы можно передать максимальное время ожидания. Кроме ```notify()``` и истекшего лимита времени ожидание потоком может быть прервано вызовом ```interrupt()``` у ожидающего потока из другого потока.

</details>


<details><summary markdown="span">notify()</summary>

**public final void notify()** - будит поток, который ожидает по монитору lock-объекта, на котором вызван метод. Или 1 из потоков, если ожидающих потоков > 1. Какой именно - мы предсказать не можем. Должен вызываться только в **synchronized**-блоке.

</details>

<details><summary markdown="span">notifyAll()</summary>

**public final void notifyAll()** - будит все потоки, синхронизированные по объекту у которого вызван метод

</details>

<details><summary markdown="span">finalize()</summary>

**protected void finalize() throws Throwable** - вызывается, когда больше нет причин для того, чтобы объект был доступен из любого живого потока.

Вызывается за время существования объекта в памяти только 1 раз.
В Object не делает ничего (у метода пустое тело).
В наследуемых классах может быть переопределен, если есть какие-то операции, которые должны быть обязательно выполнены перед уничтожением объекта.
Метод может сделать объект снова доступным, повесив ссылку на ```this```.
Использовать явный вызов этого метода - очень плохая практика.
Переопределять также не нужно практически никогда. Разве что как средство последнего шанса на высвобождение системных ресурсов. Но именно как последнего. Штатно это должно делаться при помощи конструкции **try-with-resources**.
Если все-таки нужно переопределить, то надо не забыть в конце метода вызвать ```super.finalize()``` в блоке ```finally```.
В HotSpot VM GC не вызывает ```finalize()``` напрямую, а только добавляет объекты в специальный список, вызывая статический метод ```Finalizer.register(Object)```. В список добавляются только объекты с переопределенным ```finalize()```. Объект ```Finalizer``` представляет собой ссылку на объект, для которого надо вызвать ```finalize()```, и хранит ссылки на следующий и предыдущий ```Finalizer```, формируя двусвязный список. Вызов ```finalize()``` происходит в отдельном потоке ```Finalizer```, кот. создается при запуске VM. Исключения, брошенные в ```finalize()```, не обрабатываются потоком-финализатором, т.е. данный стектрейс скорее всего нельзя будет отследить.
В Java 9 **deprecated**.
в качестве альтернативы переопределения ```finalize()``` можно использовать фантомную ссылку.

</details>

---
