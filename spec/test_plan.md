# 🧪 Test Plan — AI Assistant System (MVP 1)
*(актуализация: 2025-06-07)*

---

## 1. Обзор

| Раздел            | Детали                                                                    |
|-------------------|---------------------------------------------------------------------------|
| **Компоненты**    | FastAPI API (`/search`, `/generate`, `/settings`, `/feedback`, `/upload`) |
|                   | LLM-плагины, Qdrant, PostgreSQL, Chainlit GUI, cron-скрипты               |
| **Цель**          | Проверка функционала, производительности, безопасности, DX               |
| **Инструменты**   | `pytest`, TestClient, Playwright, Locust, Allure, Coverage, Flake8, Bandit, MyPy |
| **Среды**         | *Local*: Mac M3 Pro 24 GB + Docker-Compose<br>*CI*: GitHub Actions<br>*Prod*: K8s + Helm |

---

## 2. Область тестирования

| Модуль / Фича          | Проверки                                                                                         |
|------------------------|---------------------------------------------------------------------------------------------------|
| **API /search**        | 200 OK, сортировка `score`, сложные запросы, `top_k ≤ 50`, пустой результат                       |
| **API /generate**      | Валидность Markdown/PDF/DOCX, корректность SRS/NFR/UseCases, наличие диаграмм, тайм-аут ≤ 5 с      |
| **API /settings**      | GET/PUT, неверное поле → 422, ACL                                                                  |
| **API /feedback**      | like/dislike, enum, длинный комментарий > 1 КБ → 400                                               |
| **API /upload**        | PDF, DOCX, видео > 100 MB → 413, MIME-check                                                        |
| **Cron bootstrap_train** | Наличие `datasets/bootstrap_raw/*`, повторный запуск — skip                                     |
| **Cron retrain_daily** | Новые/изменённые/удалённые документы, лог-файл, переменная `CRON_SCHEDULE`                         |
| **LLM Loader**        | Смена `LLM_PROVIDER` (openai ↔ mistral ↔ local) без даунтайма                                      |
| **Feedback Loop**     | Накопление лайков/дизлайков, приоритизация дообучения                                              |
| **Security**          | OWASP headers, rate-limit 100 r/m, Bandit High/Medium = 0                                          |
| **Infrastructure**    | Helm `replicaCount`, readiness-probe ≤ 200 мс                                                      |

---

## 3. Классификация тестов

| Тип                 | Папка / Файл                 | Описание                                                  |
|---------------------|------------------------------|-----------------------------------------------------------|
| **Unit**            | `tests/unit/`                | Бизнес-логика сервисов, Pydantic-схемы                    |
| **Integration**     | `tests/integration/`         | API ↔ Qdrant ↔ PostgreSQL                                 |
| **Cron-Jobs**       | `tests/cron/`                | bootstrap / retrain с мок-источниками                     |
| **E2E (API+GUI)**   | `tests/e2e/` + Playwright    | upload → search → generate → feedback (голова GUI headless)|
| **Dataset QA**      | `tests/datasets/test_10k.py` | 10 000 вопросов / ответов → precision/recall              |
| **Doc-Validator**   | `tests/doc/validator.py`     | Структура SRS/NFR/UseCases, PlantUML, C4-diagram проверка |
| **Perf / Load**     | `tests/perf/locustfile.py`   | 100 RPS, p95 latency ≤ 5 с                                |
| **Static Analysis** | Flake8, MyPy, Bandit         | Lint, typing, security audit                              |

---

## 4. Метрики и критерии приёмки

| Метрика                      | Порог / Цель |
|------------------------------|--------------|
| Precision (поиск)            | **≥ 95 %**   |
| Recall (поиск)               | **≥ 90 %**   |
| BLEU / ROUGE (генерация)     | **≥ 0.90**   |
| p95 latency API              | **≤ 5 с**    |
| Code-coverage                | **≥ 90 %**   |
| Bandit High/Medium           | **0**        |
| Lint (Flake8)                | **0** W/E/F  |
| Док-валидатор                | **100 %** docs|

---

## 5. Тестовые данные

| Набор            | Объём    | Описание                                        |
|------------------|---------:|-------------------------------------------------|
| **QT-dataset**   | 100 000  | Q&A: System Design, DevOps, QA, Security        |
| **Bootstrap docs** | 500   | Markdown + PlantUML (архитектура, ADR)          |
| **Sample videos** | 10     | SREcon talks (≈ 300 КБ транскрипта)             |

---

## 6. Процедуры запуска

```bash
# Unit + Integration
pytest -m "unit or integration" --cov=app --cov-report=xml

# Cron-скрипты (dry-run)
pytest -m cron

# End-to-End GUI (headless)
playwright install
pytest -m e2e --base-url http://localhost:8000

# Performance (10 мин)
locust -f tests/perf/locustfile.py --headless -u 50 -r 10

# Lint & static checks
flake8 .
mypy app/
bandit -r app/