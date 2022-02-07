## Контейнеры и микросервисы

#### Отличие виртуализации от контейнеризации

<details><summary markdown="span">Виртуализация и гипервизор (hypervisor)</summary>

Виртуализация - предоставление набора вычислительных ресурсов или их логического объединения, абстрагированное от
аппаратной реализации, и обеспечивающее при этом логическую изоляцию друг от друга вычислительных процессов,
выполняемых на одном физическом ресурсе.

Гипервизор (Hypervisor, он же virtual machine monitor / VMM / virtualizer) - это аппаратная схема (native/bare-metal
hypervisor type 1), либо программа (hosted hypervisor type 2), обеспечивающая или позволяющая одновременное,
параллельное выполнение нескольких операционных систем на одном и том же хост-компьютере посредством использования
технологии виртуализации.

Гипервизор обеспечивает изоляцию операционных систем друг от друга, защиту и безопасность, разделение ресурсов
между различными запущенными ОС и управление ресурсами.

Гипервизор также обязан предоставлять работающим под его управлением на одном хост-компьютере ОС средства связи и
взаимодействия между собой (например, через обмен файлами или сетевые соединения) так, как если бы эти ОС
выполнялись на разных физических компьютерах.

Гипервизор сам по себе в некотором роде является минимальной операционной системой (микроядром или наноядром).  
Он предоставляет запущенным под его управлением операционным системам службу виртуальной машины, виртуализируя или
эмулируя реальное (физическое) аппаратное обеспечение конкретной машины.  
Управляет этими виртуальными машинами, выделением и освобождением ресурсов для них.  
Гипервизор позволяет независимое «включение», перезагрузку, «выключение» любой из виртуальных машин с той или иной ОС.  
При этом операционная система, работающая в виртуальной машине под управлением гипервизора, может, но не обязана «знать»,
что она выполняется в виртуальной машине, а не на реальном аппаратном обеспечении.

Среди гипервизоров аппаратного типа (native/bare-metal
[hypervisor type 1](https://ru.wikipedia.org/wiki/%D0%93%D0%B8%D0%BF%D0%B5%D1%80%D0%B2%D0%B8%D0%B7%D0%BE%D1%80))
можно отметить следующие: VMware ESXi, Hyper-V, Xbox One system software.

Среди гипервизоров программного типа, т.е. работающих поверх ОС (hosted
[hypervisor type 2](https://ru.wikipedia.org/wiki/%D0%93%D0%B8%D0%BF%D0%B5%D1%80%D0%B2%D0%B8%D0%B7%D0%BE%D1%80)),
можно отметить следующие: VirtualBox, Parallels Desktop for Mac, QEMU, VMware Player & Workstation.

![VM](images/hypervisor_types.png)

</details>

<details><summary markdown="span">Виртуальные машины (VM)</summary>

Виртуальные машины (VM) работают посредством технологии виртуализации, предоставляемой hypervisor.
Как следствие, каждая VM требует запуск собственной операционной системы (OS), а также виртуальной копии hardware,
которые необходимо OS для запуска и работы.

Операционная система и все приложение, запущенные в рамках одной VM, разделяют hardware ресурсы host'а (физического
устройство, на котором запущена VM) с учетом квотирования ресурсов между VM на стороне hypervisor.

![VM](images/vm.png)

К особенностям VM можно отнести следующие:
* возможность запуска любой операционной системы (OS);
* возможность запуска различных OS на одном host'е (физическом устройстве);
* обеспечивается высокий уровень безопасности относительно изоляции между собой несколько VM, запущенных на одном host'е;
* хорошо известная технология:
    * имеет множество надежных средств для управления и контроля уровня безопасности;
* куда более эффективная утилизация ресурсов по сравнению с управлением несколькими физическими устройствами:
    * влечет экономию средств.

</details>

<details><summary markdown="span">Контейнеры (Containers)</summary>

Контейнеры используют несколько иной уровень виртуализации.
Если в случае VM единицей виртуализации было hardware (создавать виртуальное физическое устройство), то в случае
контейнеров единицей виртуализации выступает операционная система.

Другими словами, если VM в качестве средства изоляции использовали hypervisor, то контейнеры поднимаются на уровень выше
и для изоляции используют средства host'ов ОС, поверх которой работают.

![containers](images/containers.png)

Контейнеры размещаются поверх физического сервера и его host'овой операционной системы.  
Каждый контейнер разделяет ядро host'овой операционной системы, а также, обычно, ряд библиотек.

Разделяемые компоненты являются read-only для контейнера.  
Контейнер получает изолированное, а в некоторых случаях виртуализированное, представление системы.  
Например, контейнер может обращаться к виртуализированной версии файловой системы и реестра, но любые изменения
затрагивают только контейнер и удаляются при его остановке.  
Чтобы сохранить данные, контейнер может подключить постоянное хранилище, например общую папку (монтируемое устройство).

Контейнер собирается поверх ядра, но ядро не предоставляет все интерфейсы API и службы, необходимые для запуска
приложения. Большинство из них предоставляются системными файлами (библиотеками), которые работают на уровне выше ядра в
пользовательском режиме. Поскольку контейнер изолирован от среды пользовательского режима сервера, контейнеру требуется
собственная копия этих системных файлов пользовательского режима, которые упаковываются в базовый образ.
Базовый образ выступает в качестве основного уровня, на котором собирается контейнер, предоставляя ему службы
операционной системы, не предоставляемые ядром.

К наиболее популярным приложениям для работы с контейнерами, использующими OS-level virtualization, можно отнести:
* Docker;
* Podman;
* LXC (Linux Containers);
* systemd-nspawn;
* [Windows Containers](https://docs.microsoft.com/ru-ru/virtualization/windowscontainers/).

Рассмотрим средства изоляции различных ОС, на которых базируется работа вышеописанных приложений:

Linux:
* [cgroups](https://en.wikipedia.org/wiki/Cgroups):
    * группа процессов в Linux, для которой механизмами ядра наложена изоляция и установлены ограничения на некоторые
      вычислительные ресурсы (процессорные, сетевые, ресурсы памяти, ресурсы ввода-вывода);
    * механизм позволяет образовывать иерархические группы процессов с заданными ресурсными свойствами и обеспечивает
      программное управление ими;
* [namespaces](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D1%81%D1%82%D0%B2%D0%BE_%D0%B8%D0%BC%D1%91%D0%BD_(Linux)):
    * это функция ядра Linux, позволяющая изолировать и виртуализировать глобальные системные ресурсы множества процессов:
        * примеры ресурсов, которые можно виртуализировать:
            * ID процессов;
            * имена хостов;
            * ID пользователей;
            * доступ к сетям;
            * межпроцессное взаимодействие и файловые системы.

С недавних пор в Windows также появилась поддержка контейнеров, но технология проприетарна, поэтому детали неизвестны,
но вероятнее всего используются технологии аналогичные Linux cgroups и namespaces.
Подробнее см. [здесь](https://docs.microsoft.com/ru-ru/virtualization/windowscontainers/about/) и
[здесь](https://habr.com/ru/post/465057/).

</details>

<details><summary markdown="span">VM vs Containers</summary>

Чем виртуальные машины отличаются от контейнеров?
Имеется ли смысл в использовании виртуальных машин вместо контейнеров в какой-либо ситуации?

Ответ:
Виртуальные машины могут использоваться вместо контейнеров в ситуации, когда:

Требуется:
* максимальный уровень изоляции с точки зрения безопасности
  (относительно других процессов, запущенных на том же hardware);
* максимальная оптимизация в коде приложения с учетом низкоуровневых особенностей операционной системы и hardware,
  в рамках которого запущено приложения.

Допускается:
* дополнительные затраты ресурсов (каждая VM довольно тяжеловесна, т.к. включает в себя всю
  операционную систему, включая её ядро);
* время старта измеримое в минутах (а не в миллисекундах, как в случае с контейнерами).

</details>

Подробнее см. [здесь](https://www.backblaze.com/blog/vm-vs-containers/).

---

#### Docker / Docker Compose / Swarm - k8s - OpenShift

Docker:
- приложение для работы с контейнерами, использующими OS-level virtualization (подробнее см. в вопросе выше);
- позволяет:
    - посредством файла конфигурации (dockerfile) собирать образ (docker image);
    - запускать контейнеры - экземпляры изолированных на уровне OS-level virtualization процессов, созданных на базе
      образа (docker image) в рамках одной host машины.

Docker Compose:
- утилита, работающая поверх Docker;
- позволяет:
    - посредством файла конфигурации (docker-compose.yaml) собирать и управлять сразу несколькими docker
      контейнерами в рамках одной host машины.

Swarm / k8s / OpenShift:
- оркестраторы - распределенные средства для управления контейнерами на нескольких host машинах:
    - типичный механизм работы оркестратора:
        - одна машина является master node (управляет остальными node'ами):
            - получает команды на применение deployment:
                - группы совместно работающих на одной машине контейнеров;
                - с указанием минимального и максимального количества экземпляров таких групп контейнеров;
            - на основании команды на применение deployment, master node формирует управляющие запросы на разворачивание
              группы контейнеров на worker node - формирование pod'а;
        - остальные машины являются worker node:
            - от master node на отдельно взятый worker node приходит команда на поднятие pod:
                - группы совместно работающих на одной машине контейнеров;
            - от master node получает и обрабатывает иные управляющие и мониторящие состояние pod'ов запросы.

---

#### Зачем нужен Dockerfile?
Dockerfile — это инструкция для сборки образа. Это простой текстовый файл, содержащий по одной команде в каждой строке.
В нем указываются все программы, зависимости и образы, которые нужны для разворачивания образа.

Основные команды:
- `FROM` - задаёт базовый (родительский) образ;
- `LABEL` - описывает метаданные:
    - например, сведения о том, кто создал и поддерживает образ;
- `ENV` - устанавливает постоянные переменные среды;
- `RUN` - выполняет команду и создаёт слой образа. Используется для установки в контейнер пакетов;
- `COPY` - копирует в контейнер файлы и папки;
- `ADD` - копирует файлы и папки в контейнер, может распаковывать локальные ".tar"-файлы;
- `CMD` - описывает команду с аргументами, которую нужно выполнить когда контейнер будет запущен:
    - аргументы могут быть переопределены при запуске контейнера. В файле может присутствовать лишь одна инструкция CMD;
- `WORKDIR` - задаёт рабочую директорию для следующей инструкции;
- `ARG` - задаёт переменные для передачи Docker во время сборки образа;
- `ENTRYPOINT` - предоставляет команду с аргументами для вызова во время выполнения контейнера. Аргументы не
  переопределяются;
- `EXPOSE` - декларирует о "выставляемых" из контейнера портах:
    - данная инструкция, на самом деле, не публикует порт(ы), а лишь является best practice для декларирования публикуемых
      из контейнера портах;
- `VOLUME` - создаёт точку монтирования для работы с постоянным хранилищем.

Подробнее см. [здесь](https://docs.docker.com/engine/reference/builder/).

---

#### Какие инструменты для оркестрации контейнеров использовали. Зачем они нужны?
OpenShift и Kubernetes:
- Мониторинг сервисов и распределение нагрузки Kubernetes может обнаружить контейнер, используя имя DNS или собственный IP-адрес.
  Если трафик в контейнере высокий, Kubernetes может сбалансировать нагрузку и распределить сетевой трафик, чтобы развертывание было стабильным.
- Оркестрация хранилища Kubernetes позволяет вам автоматически смонтировать систему хранения по вашему выбору, такую как локальное хранилище, провайдеры общедоступного облака и многое другое.
- Автоматическое развертывание и откаты Используя Kubernetes можно описать желаемое состояние развернутых контейнеров и изменить фактическое состояние на желаемое.
  Например, вы можете автоматизировать Kubernetes на создание новых контейнеров для развертывания, удаления существующих контейнеров и распределения всех их ресурсов в новый контейнер.
- Автоматическое распределение нагрузки Вы предоставляете Kubernetes кластер узлов, который он может использовать для запуска контейнерных задач.
  Вы указываете Kubernetes, сколько ЦП и памяти (ОЗУ) требуется каждому контейнеру. Kubernetes может разместить контейнеры на ваших узлах так, чтобы наиболее эффективно использовать ресурсы.
- Самоконтроль Kubernetes перезапускает отказавшие контейнеры, заменяет и завершает работу контейнеров, которые не проходят определенную пользователем проверку работоспособности, и не показывает их клиентам, пока они не будут готовы к обслуживанию.
- Управление конфиденциальной информацией и конфигурацией Kubernetes может хранить и управлять конфиденциальной информацией, такой как пароли, OAuth-токены и ключи SSH.
  Вы можете развертывать и обновлять конфиденциальную информацию и конфигурацию приложения без изменений образов контейнеров и не раскрывая конфиденциальную информацию в конфигурации стека.

---

#### Сколько запускается рутовых процессов если я на linux-машине разворачиваю 3 контейнера?
Запускается только базовый процесс Docker из под рута, контейнеры не плодят процессы из под рута.
```shell
root     27942  0.0  3.2 743316 32800 ?        Ssl   2021  98:15 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

---

#### Что такое Istio и зачем он нужен?
Istio — Open Source-проект, который решает сложности, возникающие в приложениях, основанных на микросервисной архитектуре
и предлагает специализированное решение, полностью отделённое от сервисов и функционирующее путём вмешательства в сетевое взаимодействие. Таким образом реализует:
- Отказоустойчивость: опираясь на код статуса в ответе, оно понимает, произошёл ли сбой в запросе, и выполняет его повторно.
- Канареечные выкаты: перенаправляет на новую версию сервиса лишь фиксированное процентом число запросов.
- Мониторинг и метрики: за какое время сервис ответил?
- Трассировка и наблюдаемость: добавляет специальные заголовки в каждый запрос и выполняет их трассировку в кластере.
- Безопасность: извлекает JWT-токен, аутентифицирует и авторизует пользователей.

---

#### Что такое Service, Route, Pod?
- `Pod`: Pod это группа контейнеров с общими разделами, запускаемых как единое целое.
- `Service`: Сервис в Kubernetes это абстракция, которая определяет логический объединённый набор pod и политику доступа к ним.
- `Route (Ingress)`: Ресурс для добавления правил маршрутизации трафика из внешних источников в службы в кластере kubernetes. Выпускает сервис во внешнюю сеть путем выделения DNS имени.

---

#### Зачем нужен replica set?
`ReplicaSet` гарантирует, что определенное количество экземпляров подов будет запущено в кластере Kubernetes в любой момент времени.

---

#### У меня есть новая версия приложения, возможно с багами, я хочу только 10% нагрузки передать на новую версию, что мне делать?
Сделать можно с помощью Istio, для этого необходимо создать `VirtualService`:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: backend
  namespace: default
spec:
  gateways: []
  hosts:
  - "backend.default.svc.cluster.local"
  http:
  - match:
    - {}
    route:
    - destination:
        host: backend.default.svc.cluster.local
        subset: v1
        port:
          number: 80
      weight: 90 # 90% нагрузки
    - destination:
        host: backend.default.svc.cluster.local
        subset: v2
        port:
          number: 80
      weight: 10 # 10% нагрузки
```

---

#### Мне нужно закрыть обмен между микросервисами tls. Как я могу это реализовать?
С помощью Istio можно закрыть все взаимодействия mTLS, путем добавления `PeerAuthentication`:
```yaml
kind: PeerAuthentication
metadata:
  name: "default"
spec:
  mtls:
    mode: STRICT
```

---