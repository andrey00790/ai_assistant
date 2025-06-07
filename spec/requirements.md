📝 AI Assistant System — Сводка требований (v1.0, 2025‑06‑07)

Документ фиксирует весь контекст проекта так, чтобы из любой новой сессии разработчик или ассистент мог продолжить следующую итерацию без потерь.

👥 Роли (универсальный профиль)

Developer (web, iOS, Android, backend)

System Analyst / Business Analyst

System Architect / Business Architect

DevOps / SRE

QA Engineer / SDET

Admin / Security Engineer

Tech Support

Все компетенции объединены → ассистент должен отвечать и генерировать контент для каждой роли.

🎯 Бизнес‑ценность

Сокращение времени на поиск, проектирование и ревью архитектуры.

Автоматическое производство стандартизированных документов (SRS, NFR, Use Cases, RFC, ADR, диаграммы) по лучшим практикам FAANG/Enterprise.

Полный цикл discovery → delivery → support → feedback → re‑train в одном инструменте.

Офлайн‑режим (данные не утекают наружу).

🔑 Ключевые функциональные блоки

1️⃣ Семантический поиск + советы

Источники: Confluence, GitLab, PDF/DOCX/TXT, видео-транскрипты, книги, RFC.

Уточняющие вопросы при score < 0.7.

Авто-ген SRS/NFR/Use Cases и диаграмм из результатов поиска.

Отображение результатов пользователю с возможностью лайка/дизлайка и текстового комментария, если результат не удовлетворяет.

Оценка используется для дообучения модели (feedback loop с сохранением фидбека в базу).

Метрика отзывов агрегируется и участвует в приоритетах переобучения и уточнения выдачи.

2️⃣ Проектирование / анализ / изменение архитектуры

Диалоговый сбор требований.

Генерация документов: SRS, NFR, Use Cases, API spec, C4 (C1–C4), UML (Class/Sequence/Activity), RFC/ADR.

Форматы: Markdown, DOCX, PDF, SVG.

Экспорт пригоден для Confluence ≥ 8, Jira, e-mail.

После генерации пользователь может поставить лайк/дизлайк и оставить комментарий по результату.

Эта информация сохраняется в базу и участвует в дообучении модели, включая приоритеты генерации и уточнения шаблонов.



 3️⃣ Инфраструктура и DevOps

Модульная архитектура: api, services, llm, vectorstore, database, models, core, gui

Плагинная LLM-подсистема — смена модели без переписывания кода; подключение через llm_plugins/, конфигурация через .env.local или llm.yaml

Хранилища: Qdrant (векторная БД), PostgreSQL (метаданные)

Инструменты: Docker Compose, Helm Chart, Makefile, Chainlit GUI, Kubernetes

🔌 Плагинная LLM-подсистема — пример реализации

**Структура **llm_plugins/

llm_plugins/
├── __init__.py
├── base.py       # BaseLLM (abstract .generate())
├── openai.py     # OpenAIChat(BaseLLM)
├── mistral.py    # MistralChat(BaseLLM)
└── local_ggml.py # LlamaCppChat(BaseLLM) — локальная модель

**Пример конфигурации **.env.local

# Выбор провайдера (openai / mistral / local_ggml)
LLM_PROVIDER=openai

# OpenAI
OPENAI_API_KEY=sk-***
OPENAI_MODEL=gpt-4o

# Mistral
MISTRAL_API_KEY=
MISTRAL_MODEL=mistral-large

# Локальный GGML
GGML_MODEL_PATH=/models/phi-3.gguf
GGML_CTX=8192

Как работает:

core/llm_loader.py читает LLM_PROVIDER

Импортирует модуль из llm_plugins через importlib

Создаёт экземпляр класса, передаёт параметры (API-ключ или путь к модели)

Сервисы используют метод BaseLLM.generate(prompt, **kwargs)

Добавление новой модели:

Создать llm_plugins/your_model.py, наследуя BaseLLM

Реализовать метод .generate()

Добавить переменные в .env.example или llm.yaml

Установить LLM_PROVIDER=your_model — остальной код менять не нужно



### 🛡️ CI-Gate `verify_testplan.py`



**Назначение**  

Скрипт гарантирует, что тест-план всегда синхронизирован с актуальными требованиями и OpenAPI-спекой.



**Алгоритм**



1. Парсит:

   - `docs/openapi_spec_mvp1.yaml` (эндпоинты, методы)

   - `requirements_summary.md` (по ключевым словам `/search`, `/generate` …)

   - `test_plan.md` (таблица «Область тестирования» и «Классификация тестов»)

2. Формирует diff:

   - *Новые* эндпоинты или модули **без** соответствующего теста ⇒ `missing_tests.md`

   - Удалённые эндпоинты, которые ещё есть в тест-плане ⇒ `obsolete_tests.md`

3. Если есть расхождения ⇒  

   - выводит список в консоль + сохраняет markdown-отчёт  

   - завершает работу **exit code ≠ 0** (CI блокирует PR)

4. Если всё синхронно ⇒ exit 0, вывод «✅ test_plan актуален».



**Запуск**  

```bash

python scripts/verify_testplan.py --spec docs/openapi_spec_mvp1.yaml \

       --req requirements_summary.md --plan test_plan.md



---



#### Что в нём реализовать (в коде)



- YAML-парсер (`openapi-schema-parser` или `yaml.safe_load`) — собрать список `path + method`.  

- Markdown-парсер (`markdown-it-py` или простые регексы) — вытащить таблицы из test-плана.  

- Сравнить множества эндпоинтов / модулей.  

- Напечатать diff, записать `missing_tests.md` / `obsolete_tests.md`.  

- `sys.exit(1)` если diff не пустой.



---



С таким описанием в requirements любой разработчик (или ассистент в новой сессии) сразу понимает, что именно должен делать `verify_testplan.py`.





4️⃣ Импорт знаний, обучение и инкрементальное дообучение

СкриптНазначениеЗапуск





``

Однократное первичное обучение на curated-датасете (книги, статьи, FAANG-RFC, стандарты и т.д.)

Ручной запуск при первом развёртывании

``

Ежесуточное инкрементальное обновление: добавляет новые, перезаписывает изменённые, удаляет удалённые источники

Cron Job / K8s CronJob (0 3 * * *)

📚 Первичное обучение (bootstrap_train.py)

Используется один раз при первом развёртывании или сбросе.

Индексирует проверенный датасет: FAANG RFC/ADR/SRS, архитектура, DevOps, QA, базы данных, стандарты ISO/IEEE, интервью и лекции (SREcon, re:Invent).

Скрипт автоматически скачивает открытые материалы (arXiv, ACM, блоги FAANG, RFC, ISO PDF, YouTube) и сохраняет их в datasets/bootstrap_raw/. Если скачивание не удалось (например, файл недоступен или требует авторизации) — он пропускает источник и продолжает работу.

После загрузки скрипт конвертирует документы (PDF → TXT/Markdown, DOCX → Markdown, видео → транскрипт), тегирует их по ролям (например, developer, architect, QA), присваивает категории (архитектура, тестирование и др.), а затем отправляет данные в индексатор. Готовые эмбеддинги помещаются в Qdrant, метаданные записываются в PostgreSQL. } ] }



🔗 Дополнительные открытые источники

КатегорияПримеры материалов



Cloud & Scalability

AWS Well-Architected Labs, Google Cloud Architecture Center, Azure Architecture Center

Networking & CDN

Cloudflare Blog, Fastly Engineering, Google Quic / BBR white-papers

Observability

Honeycomb Blog, Datadog Tech Blog, OpenTelemetry Specs

AI / ML Ops

Papers With Code SOTA summaries, Google Vertex AI docs, MLOps.dev guides

Security

Cloud Security Alliance docs, NIST SP-800 series, Google BeyondProd

DevEx / Platform Eng

Spotify Backstage RFC, Etsy Engineering Blog, Stripe Dev Productivity

Community Q&A

StackOverflow Developer Survey, StackExchange data-dump (filtered)

Public GitHub

Trending OSS repos (Star ≥ 1K) — README, ADR, CODE_OF_CONDUCT

Academic

arXiv categories: cs.SE, cs.DB, cs.DC, cs.CR (System Engineering, Databases, Distributed Computing, Cryptography)

Standards & RFC

IETF: gRPC over HTTP/2 (RFC 9113), QUIC (RFC 9000), HTTP/3 (RFC 9114); ISO/IEC 25010

{

"date": "2025-06-07T03:17:42Z",

"schedule": "0 3 * * *",

"stats": {

"added": 123,

"updated": 45,

"deleted": 12,

"skipped": 8,

"failed": 3

},

"failures": [

{

"source": "https://arxiv.org/pdf/2402.12345.pdf",

"reason": "HTTP 403 Forbidden"

},

{

"source": "https://example.com/missing.pdf",

"reason": "Connection timeout"

},

{

"source": "s3://videos/talk.mp4",

"reason": "Whisper transcription error"

}

],

"duration_sec": 742,

"vector_time_ms": 11405,

"notes": "High number of new docs from Cloudflare Blog; 5 docs re-indexed due to feedback weight."

}





added / updated / deleted / skipped — количественная сводка.

failures — список URL/пути и причина ошибки (не останавливает процесс).

vector_time_ms — время генерации эмбеддингов.

notes — краткая аналитика, полезна для мониторинга CronJob-ов.







🔄 Ежесуточное дообучение (retrain_daily.py)

Источники: Confluence API, GitLab API, каталог datasets/, видео (Whisper/WhisperX).

Алгоритм:

Сканирует документы, вычисляет hash/etag.

Новые → добавить, изменённые → перезаписать, удалённые → исключить.

Индексирует в Qdrant без удаления существующих эмбеддингов.

Учитывает лайк/дизлайк + комментарий пользователя для приоритизации дообучения.

Логирует результат в logs/import_YYYYMMDD.json.

# Пример docker-compose для запуска по расписанию
services:
  retrain-cron:
    image: python:3.11-slim
    command: >
      bash -c "pip install -r /work/requirements.txt && python scripts/retrain_daily.py"
    volumes:
      - ./:/work
    env_file:
      - .env.local
    restart: unless-stopped







⚙️ Архитектура модулей

ai_assistant/
├── app/                     # ⬅️ точка входа FastAPI
│   ├── main.py              #   – инициализация приложения, middleware, DI
│   └── api/                 #   – роуты, схемы OpenAPI
│       ├── v1/
│       │   ├── search.py    #     /search
│       │   ├── generate.py  #     /generate
│       │   ├── settings.py  #     /settings
│       │   └── feedback.py  #     /feedback (лайк/дизлайк)
│       └── health.py        #   /health
│
├── services/                # Сервис-слой (use-cases, оркестрация)
│   ├── search_service.py    #   семантический поиск → VectorStore + LLM
│   ├── generate_service.py  #   генерация SRS/NFR/UseCases/диаграмм
│   ├── settings_service.py  #   чтение/запись конфигов
│   └── feedback_service.py  #   запись фидбека + приоритизация дообучения
│
├── llm/                     # Плагинная LLM-подсистема
│   ├── llm_loader.py        #   динамическая загрузка по LLM_PROVIDER
│   ├── base.py              #   BaseLLM (абстрактный интерфейс)
│   └── llm_plugins/         #   плагины моделей
│       ├── openai.py
│       ├── mistral.py
│       └── local_ggml.py
│
├── vectorstore/             # Обёртки работы с Qdrant
│   ├── client.py            #   подключение, коллекции
│   └── similarity.py        #   запросы и ranking
│
├── database/                # SQLAlchemy + Alembic
│   ├── models.py            #   таблицы (feedback, configs, sources)
│   ├── crud.py              #   CRUD-операции
│   └── migrations/
│
├── models/                  # Pydantic-схемы API
│   ├── search.py
│   ├── generate.py
│   ├── settings.py
│   └── feedback.py
│
├── core/                    # Утилиты и общая логика
│   ├── config.py            #   чтение env / yaml
│   ├── cron/                #   задачи cron-job
│   │   ├── bootstrap_train.py   #   первичное обучение
│   │   └── retrain_daily.py     #   ежедневное дообучение
│   └── utils.py             #   вспомогательные функции
│
├── gui/                     # Chainlit GUI (UI вкладки)
│   ├── components.py
│   └── main_gui.py
│
├── scripts/                 # CLI-скрипты (импорт, диагностика, seed)
├── datasets/                # bootstrap_raw/ + подготовленные выборки
├── docs/                    # OpenAPI, requirements, test-plan, kanban
├── helm/                    # Helm-chart для Kubernetes
├── tests/                   # unit, integration, e2e
└── docker-compose.yaml      # локальный стек (API + Qdrant + PG + cron)



Зависимости и потоки данных

GUI ➜ API (FastAPI) ─┬─► services/ ──► llm/ (модель)

                     │              └─► vectorstore/ (Qdrant)

                     └─► database/ (feedback, configs)





API-уровень принимает запросы, валидирует через Pydantic-схемы,

передаёт их в сервисы.

Service-слой — единое место бизнес-логики: orchestration, пост-/пре-процесс.

LLM-подсистема динамически подключает выбранный плагин, обеспечивая единый .generate().

VectorStore обслуживает семантический поиск (embedding ↔ Qdrant).

Database хранит метаданные, фидбек, статусы импорт-задач.

Cron-job импортирует/дообучает данные и запускается либо как k8s-CronJob, либо через docker-compose-сер-cron-сервис.



Связь с требованиями

ТребованиеМодуль/директория



Семантический поиск

services/search_service.py + vectorstore/

Генерация SRS/NFR/UseCases

services/generate_service.py + llm/

Лайк/дизлайк + дообучение

services/feedback_service.py + cron/retrain_daily.py

Плагинная смена модели

llm/llm_loader.py, папка llm_plugins/

Квалификация 100k Q&A

datasets/ + tests/e2e/

Deployment (Docker/K8s)

docker-compose.yaml, helm/

🛠️ Deployment

Локально (MacBook Pro M3 Pro 24 GB)



Запуск одной командой: make run или docker-compose up --build.

Все тесты, линтеры (pytest, flake8, bandit, mypy) — зелёные.



Kubernetes / Helm Chart

helm install ai-assistant ./helm -f values-prod.yaml

replicaCount ≥ 1; HPA; Ingress + TLS.

SecurityContext, PodSecurityStandards, imagePullSecrets.

Возможность деплоя по 1 реплике в каждом DC (antiaffinity / topologyKey).

🔧 Настройки окружения (.env.local)

# Qdrant / PlantUML / LLM
QDRANT_URL=http://localhost:6333
PLANTUML_URL=http://localhost:8080
MODEL_URL=http://localhost:11434
ORCHESTRATOR_URL=http://localhost:8000

# Jira
JIRA_URL=
JIRA_USERNAME=
JIRA_PASSWORD=

# Confluence ≥ 8.0 (PAT)
CONFLUENCE_URL=
CONFLUENCE_BEARER_TOKEN=

# GitLab Instances\...

Файл .env.example хранится в репозитории.



📑 OpenAPI 3.1 (YAML)



Смотри openapi.yaml



    📊 Метрики и тест-план



QT-dataset 100 000 вопросов: Precision ≥ 95 %, Recall ≥ 90 %.

BLEU/ROUGE ≥ 0.9 на генерируемых SRS/NFR документах.

Время ответа ≤ 5 с.

Code-coverage ≥ 90 %.

End-to-End: загрузка → поиск → проектирование → ревью → изменение → rollout.

Тест план смотри в  test_plan.md

📅 Команды (pipeline)

КомандаДействия



Начни новую итерацию

1) git clone … → 2) прочитать requirements, OpenAPI, test_plan, Kanban → 3) прогнать тесты/линтеры → 4) взять первый wait в Kanban → 5) реализовать инкремент → 6) протестировать → 7) собрать архив zip → 8) base64 → Canvas Iteration Archive → 9) git-patch → Canvas Iteration Patch → 10) ждать приёмки

Покажи статус проекта

Канбан, текущий прогресс, активная задача

Покажи документацию

Вывести requirements_summary.md, test_plan.md

Запусти тесты

Полный pytest (unit + integration + e2e)

Собери отчёт о тестировании

Allure/coverage/bandit

Проведи квалификационные тесты

Прогон 10 000/100 000 вопросов

📜 Документация в репозитории

requirements.md (этот файл)

  openapi.yaml (контракт API)

test_plan.md (подробные тесты + traceability)

mvp_Iteration_kanban.md (план с версионированием)

resume_prompt.txt (универсальный prompt для новой сессии)





**🟢 Алгоритм возобновления **

Возобнови работу над проектом AI Assistant System.

	1.	pull / clone репозитория
    
    git clone https://github.com/andrey00790/ai_assistant.git
    cd ai_assistant

2.	Прочитать ключевые файлы
	•	requirements.md ― требования, команды, Kanban
	•	openapi.yaml ― контракт API
	•	test_plan.md ― область тестирования + метрики
	•	mvp_Iteration_kanban.md ― статусы итераций (WAIT / IN PROGRESS / DONE)
	•	.env.example или .env.local ― переменные окружения

3.	Синхронизация тест-плана

make verify-testplan         # запускает scripts/verify_testplan.py

Если обнаружены новые эндпоинты без тестов → выход 1, работа останавливается до обновления test_plan.

4.	Быстрый sanity-check

pytest -q && flake8 . && bandit -r app && mypy app

Все проверки должны быть зелёными; покрытие ≥ текущего порога (см. метрики).

5.	Найти первый WAIT в Kanban
	•	Если это строка 0 («OpenAPI Stub-генерация») и stubs ещё не созданы → выполнить

    npx @openapitools/openapi-generator-cli generate \
    -i docs/openapi_spec_mvp1.yaml -g python-fastapi -o generated/fastapi_stubs
pytest tests/test_contract.py

	•	Иначе → перейти к соответствующей итерации.
6.	Перевести строку Kanban в IN PROGRESS и запустить таймер 30 мин.
7.	Реализовать инкремент
	•	Писать код, тесты, доки строго по Exit-criteria выбранной строки.
	•	В случае изменения OpenAPI → обновить spec и запустить шаг 0 вновь.
	•	Покрытие кода не должно падать.
8.	Прогнать тесты / линтеры / perf

pytest -m "not perf" --cov=app --cov-report=xml
locust -f tests/perf/locustfile.py --headless -u 10 -r 5 -t1m

9.	Собрать архив
zip -r iteration_<N>.zip .
base64 iteration_<N>.zip > iteration_<N>.b64

  •	Проверь, что архив разворачивается base64 -d … | unzip - без ошибок.
  • В Canvas “Iteration Archive ” создаётся новый документ и вставляется содержимое файла iteration_<N>.b64 (только base64-строки и короткая инструкция восстановления).
10.	Сформировать git-патч изменений

git add -A
git diff --staged > iteration_<N>.patch

	•	В Canvas “Iteration Patch ” создаётся второй документ; туда помещается содержимое iteration_<N>.patch.

11.	Ожидать приёмки
	•	После моего «Принято» → статус строки Kanban = DONE.
	•	Если найдены ошибки → исправить код и повторить шаги 8 → 9 → 10 (обновив оба Canvas-документа).

‼️ Всегда два Canvas-артефакта на спринт:
• Iteration Archive  — цельный base64-архив проекта.
• Iteration Patch  — diff-патч.
Это гарантирует сохранность как diff-истории, так и рабочего снапшота.





📅 Kanban‑процесс (30‑мин итерации)

См. файл mvp_Iteration_kanban.md

Каждая строка = 1 итерация ≤ 40 мин.

После приёмки статус → done.

⚡ Pipeline‑команды

Команда

Действие

Начни новую итерацию

1. pull repo → 2. прочитать файлы в папке spec → 3. сверка → 4. pytest+lint → 5. взять первый wait в Kanban → 6. реализовать → 7. pytest → 8. archive (base64) → 9. git‑patch → 10. ждать приёмки

Покажи статус проекта

Kanban‑таблица, активная задача, % готовности

Покажи документацию

requirements, test_plan, OpenAPI

Запусти тесты

pytest (+coverage)

Собери отчёт о тестировании

Allure/coverage отчёт

Проведи квалификационные тесты

Запуск 100k Q&A оценки



git remote -v