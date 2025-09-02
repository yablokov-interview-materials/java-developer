# Контейнерные рантаймы и их роль в Kubernetes

Эта шпаргалка описывает основные компоненты контейнерной экосистемы: **Docker**, **containerd**, **CRI-O**, а также
низкоуровневые рантаймы и альтернативы.

---

## Уровни контейнерной экосистемы

1. **High-level tool (UX)**  
   CLI и утилиты для работы с контейнерами (например, `docker run`, `docker build`).
2. **Container runtime**  
   Движок, управляющий жизненным циклом контейнеров (образы, снапшоты, запуск, сеть).
3. **CRI (Container Runtime Interface)**  
   gRPC-протокол, через который Kubernetes общается с рантаймами.
4. **OCI runtime**  
   Низкоуровневый рантайм, который запускает процессы в cgroups/namespaces.

---

## Docker

- **Что это:** изначально монолит: CLI + build system + runtime.
- **Внутри:** Docker → containerd → runc.
- **В Kubernetes:** до версии 1.24 kubelet общался с Docker напрямую через shim. Сейчас support удалён.
- **Сегодня:** используется в первую очередь как удобный CLI и UX-инструмент для разработчиков (локально, CI).

---

## containerd

- **Описание:** контейнерный runtime, вынесенный из Docker, CNCF-проект.
- **Функции:**
    - управление образами, снапшотами, сетями, lifecycle контейнеров;
    - интерфейс CRI для Kubernetes;
    - делегирование запуска в runc/crun.
- **Роль в Kubernetes:** дефолтный runtime в большинстве дистрибутивов (GKE, EKS, AKS, k3s).
- **Особенность:** универсальный — работает и в Kubernetes, и самостоятельно (например, через `nerdctl`).

---

## CRI-O

- **Описание:** контейнерный runtime, созданный специально для Kubernetes.
- **Функции:** минимальная реализация CRI без "лишнего" функционала.
- **Внутри:** CRI-O → runc/crun → ядро Linux.
- **Использование:** Red Hat OpenShift, OKD, Fedora.
- **Особенность:** простота и "чистота", всё крутится вокруг k8s.

---

## Низкоуровневые OCI рантаймы

- **runc**
    - Reference implementation Open Container Initiative (OCI).
    - Запускает контейнеры через syscalls, namespaces и cgroups.
    - Используется containerd и CRI-O как дефолтный backend.

- **crun**
    - Аналог runc, но написан на C (а не Go).
    - Легче и быстрее, лучше поддерживает cgroups v2.
    - По умолчанию используется в Fedora и Red Hat.

---

## Альтернативы и sandboxed рантаймы

- **Podman**
    - Альтернатива Docker CLI (аналог `docker run`).
    - Без демона, работает rootless.
    - В k8s напрямую не используется, но удобен локально для DevOps и CI.

- **Kata Containers**
    - Контейнеры в лёгких виртуальных машинах (sandbox).
    - Усиленная изоляция, подходит для multi-tenant окружений.

- **gVisor (Google)**
    - User-space реализация системных вызовов Linux.
    - Обеспечивает безопасность за счёт ограничения доступных syscalls.
    - Хорошо подходит для production с повышенными требованиями к изоляции.

---

## Сравнительная таблица

| Runtime        | Уровень       | Основное назначение            | Где используется            |
|----------------|---------------|--------------------------------|-----------------------------|
| **Docker**     | CLI + runtime | UX, локальная разработка       | Локально, CI                |
| **containerd** | Runtime (CRI) | Универсальный runtime          | Default в k8s (GKE, EKS…)   |
| **CRI-O**      | Runtime (CRI) | Минимализм, только под k8s     | OpenShift, OKD              |
| **runc**       | OCI runtime   | Запуск процессов               | В containerd / CRI-O        |
| **crun**       | OCI runtime   | Быстрее и легче runc           | Fedora, Red Hat             |
| **Podman**     | CLI           | Альтернатива Docker без демона | Локально, DevOps            |
| **Kata**       | Sandbox       | Контейнеры в лёгких VM         | Multi-tenant окружения      |
| **gVisor**     | Sandbox       | Syscall sandboxing             | Security-чувствительные pod |

---

## Слои контейнерной экосистемы

| Уровень     | Примеры реализации      | Роль / что делает                                   |
|-------------|-------------------------|-----------------------------------------------------|
| CLI / UX    | Docker, Podman, nerdctl | Интерфейс для разработчика, команды `run/build`     |
| CRI runtime | containerd, CRI-O       | Управляет жизненным циклом контейнеров, CRI для k8s |
| OCI runtime | runc, crun              | Запускает контейнеры через namespaces и cgroups     |
| Kernel / OS | Linux kernel            | Реализация namespaces, cgroups, сетевых стеков      |
