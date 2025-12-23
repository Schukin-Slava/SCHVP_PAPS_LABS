# Лабораторная работа №3

**Тема:** Использование принципов проектирования на уровне методов и классов
**Цель работы:** Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.
**Ожидаемые результаты:**
1.  Добавить ранее созданные или создать диаграммы контейнеров и компонентов нотации C4 model (1 балл).
2.  Построить диаграмму последовательностей для выбранного варианта использования (показать взаимодействие C4-компонентов для реализации выбранного варианта использования) (2 балла).
3.  Построить модель БД в виде диаграммы классов UML. Если по заданию не предусмотрена БД, то самостоятельно продумать возможное хранилище данных, связанное с заданием. Минимально количество сущностей: 5 (1 балл).
4.  Реализовать требуемый клиентский и серверный код с учетом принципов KISS, YAGNI, DRY и SOLID. Пояснить, каким образом были учтены эти принципы (4  балла).
5.  Самостоятельно ознакомиться, кратко изложить и обосновать применимость или отказ по каждому принципу разработки в отдельности (2 балла). Рассмотреть следующие принципы разработки: BDUF. Big design up front («Масштабное проектирование прежде всего»); SoC. Separation оf concerns (принцип разделения ответственности); MVP. Minimum viable product (минимально жизнеспособный продукт); PoC. Proof of concept (доказательство концепции).

---

**Тема работы:**  "Разработка веб-приложения для удалённого мониторинга и управления парком 3D-принтеров"

---



## Диаграмма контейнеров

![Image](https://github.com/user-attachments/assets/11c94c58-3105-4d53-bb4b-1f1607660b2a)

**Описание:**
Ключевые контейнеры:
*   Web UI (Single-Page Application, React)
*   PrinterHub Controller (API Server, FastAPI / Python)
*   Database (PostgreSQL)
*   Object Storage (Filesystem / MinIO)
*   Printer Agent (Gateway)
*   Message Broker (Redis / RabbitMQ)
*   Background Worker (Celery / rq)
*   Внешние системы: Slicing Service, Push Service, 3D-printers

Контейнерная диаграмма показывает распределение ответственности: 
1.  Фронтенд - интерфейс.
2.  API - бизнес-логика.
3.  DB - метаданные.
4.  Storage - файлы.
5.  Agent - шлюз к принтерам.
6.  Broker/Worker - асинхронная обработка (Распределение задач, слайсинг, анализ и пр.).



## Диаграмма компонентов

##### API-сервер PrinterHub Controller

![Image](https://github.com/user-attachments/assets/d5fe79e2-ed2e-4b3a-9371-898d81c10a5c)

Компоненты API-сервера (ключевые):
*   REST / WebSocket Controller - HTTP/WebSocket эндпоинты.
*   Auth & User Management - аутентификация, сессии, роли.
*   Job Orchestrator - создание/планирование/контроль задач печати.
*   Slicing Orchestrator - отправка задач на слайсер, получение G-code.
*   Print Job Manager - управление жизненным циклом печати.
*   Repository / DB Adapter - слой доступа к БД (ORM).
*   Notification / Push Adapter - интеграция с push-сервисами.
*   Model Manager - загрузка/валидация 3MF, сохранение в объектное хранилище.
*   Filament Inventory - учёт расходных материалов.

##### Printer Agent

![Image](https://github.com/user-attachments/assets/88f07349-9b45-4a7e-9a37-aebf151981e5)

Компоненты Printer Agent (ключевые):
*   API Client (to PrinterHub) - двунаправленные соединения с API-сервером.
*   Local Job Queue - устойчивость/буферизация локально.
*   Klipper Adapter / Printer Adapter - сетевой адаптер к конкретным принтерам.
*   Telemetry Collector - сбор телеметрии и отправка её на сервер.
*   Camera Proxy - проксирование RTSP/MJPEG, создание снимков.
*   Object Storage Client - загрузка снимков и логов в Storage.



## Диаграмма последовательностей

**Выбранный вариант использования**: Загрузка модели - Создание задания печати - Слайсинг - Отправка задания агенту - Печать - Телеметрия.

![Image](https://github.com/user-attachments/assets/8c286ab7-616c-4c06-9cda-caf9f98e54c1)

**Краткое пояснение**:
1.  Загрузка и подготовка: Администратор загружает 3D-модель через веб-интерфейс - файл сохраняется в хранилище - создается запись задания в БД - задача на слайсинг отправляется в очередь сообщений.
2.  Слайсинг (преобразование в G-код): Фоновый воркер забирает задание из очереди - загружает модель - генерирует G-код - сохраняет результат - уведомляет о завершении через брокера сообщений.
3.  Отправка на печать: API получает уведомление о готовности G-кода - отправляет задание в очередь печати - агент принтера забирает задание - передает G-код на физический принтер - обновляет статус на "выполняется".
4.  Мониторинг в реальном времени: Принтер отправляет телеметрию (прогресс, температуру) - данные сохраняются в БД - веб-интерфейс получает обновления через WebSocket - видеопоток может записываться в хранилище.
5.  Завершение: Принтер сообщает об окончании печати - система обновляет статус задания - веб-интерфейс и администратор получают уведомления.



## Модель Базы Данных

![Image](https://github.com/user-attachments/assets/0de176f8-e6dd-4c66-a6ff-c26285e4b19f)

**Краткое пояснение**:

1.  User - аккаунт пользователя. Может создавать задания и отчёты.
2.  PrintJob - задача печати. Хранит статусы, временные метрики, связь с профилем печати, автором and т.д.
3.  Model3D - загруженная 3MF модель. Путь хранится в Object Storage, в БД-метаданные.
4.  Printer - устройство. Хранит информацию подключения и состояние.
5.  Filament - расходный материал (катушка). Хранит остатки и характеристики.
6.  PrintProfile - профиль печати (температуры, скорость, слой и т.п.), используется PrintJob.
7.  Report - отчёты по печати (ежедневные).



## Применение основных принципов разработки

Ниже демонстрационные фрагменты кода на Python / FastAPI, которые иллюстрируют применение принципов проектирования на уровне методов и классов.


#### Контроллер FastAPI (KISS - простые и понятные эндпоинты)
Эндпоинт делает только то, что нужно: принимает запрос, вызывает сервис и возвращает результат. Вся бизнес-логика вынесена в сервисный слой, что делает контроллер простым, понятным и легко тестируемым.

```py
# app/routers/jobs.py

router = APIRouter()

async def get_job_service(session=Depends(get_async_session)):
    repo = JobRepository(session)
    # dispatcher: конкретная реализация передаётся здесь (инъекция зависимостей)
    from app.adapters.rabbit_dispatcher import RabbitDispatcher
    dispatcher = RabbitDispatcher()
    return JobService(repo, dispatcher)

@router.post("/jobs", response_model=PrintJobRead)
async def create_job(payload: PrintJobCreate, svc: JobService = Depends(get_job_service)):

    # KISS: метод делает ровно то, что нужно - создаёт и запускает задачу
    gcode_path = "/path/to/generated.gcode"  
    job = await svc.create_and_dispatch(payload, gcode_path)
    return job
```


#### YAGNI - пример сознательного упрощения
В проекте не создаются избыточные абстракции и инфраструктурные уровни «на будущее». Например, если сейчас поддерживается только один брокер (RabbitMQ), пока не вводим фабрики для множества брокеров, не создаём сложную конфигурацию переключения провайдеров - вводим её, когда действительно появится необходимость. Это экономит время и уменьшает технический долг сейчас.

```py
class RabbitDispatcher(Dispatcher):
    async def dispatch_print_job(self, job_id: int, gcode_path: str):
        # publish to rabbitmq
        pass
```


#### ORM-модель и Pydantic-схема (DRY - единое место описания данных)
Модель SQLAlchemy и Pydantic-схемы синхронизированы через orm_mode=True, что позволяет использовать одни и те же валидационные правила на всех уровнях приложения. Это устраняет дублирование описания структуры данных и гарантирует консистентность между API, бизнес-логикой и базой данных.

```py
# app/models/orm.py

Base = declarative_base()

class PrintJob(Base):
    __tablename__ = "print_jobs"
    job_id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.user_id"), nullable=False)
    profile_id = Column(Integer, ForeignKey("print_profiles.profile_id"))
    status = Column(String, default="queued")
    priority = Column(Integer, default=0)
    created_date = Column(DateTime, default=datetime.utcnow)
    started_date = Column(DateTime, nullable=True)
    completed_date = Column(DateTime, nullable=True)
    estimated_time = Column(Integer, nullable=True)
    actual_time = Column(Integer, nullable=True)
    cost = Column(Numeric, nullable=True)
# app/schemas/job_schema.py
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

class PrintJobCreate(BaseModel):
    user_id: int
    model_id: int
    profile_id: Optional[int]
    priority: Optional[int] = 0

class PrintJobRead(BaseModel):
    job_id: int
    user_id: int
    status: str
    created_date: datetime
    class Config:
        orm_mode = True
```


#### SOLID
##### Репозиторий (SRP - принцип единственной ответственности)
JobRepository имеет единственную ответственность - выполнять CRUD-операции с сущностью PrintJob в базе данных. Это отделяет логику доступа к данным от бизнес-логики, что упрощает тестирование и позволяет менять реализацию хранилища без изменения сервисов.

```py
# app/repositories/job_repo.py

class JobRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def create(self, payload: PrintJobCreate) -> PrintJob:
        job = PrintJob(
            user_id=payload.user_id,
            profile_id=payload.profile_id,
            priority=payload.priority
        )
        self.session.add(job)
        await self.session.commit()
        await self.session.refresh(job)
        return job

    async def get(self, job_id: int) -> PrintJob:
        return await self.session.get(PrintJob, job_id)
```


##### Сервис: бизнес-логика и инверсия зависимостей (DIP, OCP, SRP)
JobService зависит от абстракции Dispatcher, а не от конкретной реализации, что позволяет легко заменять способ отправки заданий. Система открыта для расширения (можно добавить новые диспетчеры), но закрыта для модификации основного сервиса.

```py
# app/services/job_service.py

# Интерфейс отправки задач на брокера или агента (DIP)
class Dispatcher(ABC):
    @abstractmethod
    async def dispatch_print_job(self, job_id: int, gcode_path: str):
        pass

class JobService:
    def __init__(self, repo: JobRepository, dispatcher: Dispatcher):
        self.repo = repo
        self.dispatcher = dispatcher

    async def create_and_dispatch(self, payload: PrintJobCreate, gcode_path: str):
        # SRP: JobService отвечает за создание задания и его запуск
        job = await self.repo.create(payload)
        await self.dispatcher.dispatch_print_job(job.job_id, gcode_path)
        return job
```

*   Dispatcher - абстракция (интерфейс). Rонкретная реализация может быть подставлена без изменения (DIP).
*   Если появится новый способ доставки (например, прямой HTTP к агенту), достаточно добавить новую реализацию Dispatcher (OCP - класс открыт для расширения).
*   JobService имеет только одну ответственность - создать задачу и инициировать доставку (SRP).


##### Абстракция принтера (LSP и ISP)
Интерфейс BasePrinter определяет минимальный контракт для работы с любым принтером, что позволяет заменять реализации без изменения клиентского кода. Каждый принтерный адаптер предоставляет только необходимые методы, избегая избыточности.

```py
# app/core/printers.py

class BasePrinter(ABC):
    @abstractmethod
    async def send_gcode(self, gcode_path: str) -> str:
        """Отправить gcode на принтер, вернуть job_id у принтера."""
        pass

    @abstractmethod
    async def get_status(self, remote_job_id: str) -> dict:
        """Получить статус задания у принтера."""
        pass

# Конкретная реализация (например, для Klipper API)
class KlipperPrinter(BasePrinter):
    async def send_gcode(self, gcode_path: str) -> str:
        # реализация отправки (HTTP POST файл/URL)
        return "printer-job-123"

    async def get_status(self, remote_job_id: str) -> dict:
        return {"status":"running", "progress": 42}
```


#### Кратко по всему
1.  KISS: простые и маленькие функции/эндпоинты (create_job), нет перегруженной логики в контроллерах.
2.  YAGNI: не вводим универсальные абстракции до реальной потребности.
3.  DRY: общие функции доступа к БД в JobRepository, общие схемы, повторно используемые модули.
4.  SOLID:
    *   SRP: JobRepository, JobService, routers.jobs - разные обязанности.
    *   OCP: добавление нового Dispatcher не требует изменения JobService.
    *   LSP: KlipperPrinter подставляется вместо BasePrinter.
    *   ISP: интерфейсы минимальны (BasePrinter предоставляет только необходимые методы).
    *   DIP: JobService зависит от абстракции Dispatcher, а не от конкретного Rabbit dispatcher.



## Дополнительные принципы разработки

#### BDUF (Big Design Up Front)
**Суть**: Детальное проектирование всей системы перед началом кодирования.
**Применимость**: Частично.
**Обоснование**: В проекте подготовлены диаграммы контекста, контейнеров и компонентов, а также модель данных - этого достаточно для начала реализации. Полный BDUF (детализировать каждую мелочь) нецелесообразен: требования и детали реализации уточняются в процессе. Поэтому применили лишь «достаточный дизайн» на уровне архитектуры, а детали реализуем итеративно (lean-подход). Это снижает риск переработок и потери времени.

#### SoC (Separation of Concerns)
**Суть**: Деление системы на независимые зоны ответственности.
**Применимость**: Применяется.
**Обоснование**: Производим явное разделение на слои: UI / API / Services / Repositories / Agent / Worker / Storage. Это повышает тестируемость, удобство сопровождения и локализацию изменений. Этот принцип согласуется с SOLID (SRP) и реализован в структуре проекта.

#### MVP (Minimum Viable Product)
**Суть**: Реализовать минимально необходимый функционал для валидации идеи.
**Применимость**: Применяется.
**Обоснование**: Проект ориентирован на реализацию ключевых сценариев (загрузка модели, запуск печати, телеметрия). Дополнительный функционал (расчёт стоимости, сложные отчёты, расширенная поддержка типов принтеров) откладывается на потом. Это позволяет быстрее получить рабочую систему и получить обратную связь.

#### PoC (Proof of Concept)
**Суть**: Быстрый прототип для проверки технической реализуемости.

**Применимость**: Не применяется.

**Обоснование**: Технологии (FastAPI, RabbitMQ/Redis, взаимодействие с принтерами) достаточно распространены и проверены. Проект не содержит совершенно новых или рискованных технических гипотез, требующих отдельного PoC. Вместо этого сразу выполняется реализация согласно архитектуре и iteratively дорабатывается.
