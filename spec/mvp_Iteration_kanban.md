# 📋 MVP Iteration Kanban (Sprint ≈ 30 мин, CI-gate — все тесты зелёные)

| #  | Итерация (~30 мин)                       | Артефакты / Код                                               | **Критерии приёмки + типы тестов** (см. test_plan.md)                                               | Статус |
|----|-----------------------------------------|--------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|--------|
| 0  | **OpenAPI Spec + Stub-генерация**       | `openapi_spec_mvp1.yaml`, FastAPI stubs, `tests/test_contract.py` | ✔ Spec валиден <br>✔ Stubs 200 OK (unit)                                                            | WAIT |
| 1  | **Init + Skeleton API**                 | FastAPI app, docker-compose, Helm stub, интеграция stubs      | ✔ `make run` запускает стек <br>✔ Все эндпоинты 200 OK <br>coverage ≥ 70 % (unit)                   | WAIT |
| 2  | **DB + VectorStore**                    | PostgreSQL ORM, Alembic, Qdrant client                        | ✔ PG + Qdrant поднимаются <br>Integration-тесты зелёные <br>coverage ≥ 75 %                         | WAIT |
| 3  | **Semantic Search core**                | `search_service.py`, `/upload`, embedding                     | ✔ Precision ≥ 60 % (stub) <br>p95 ≤ 5 с @20 RPS (Locust)                                             | WAIT |
| 4  | **Generate docs & architecture**        | `generate_service.py`, PlantUML proxy, Markdown/PDF           | ✔ SRS/NFR/UC валидны (doc-validator) <br>C4/UML PNG/SVG создаются <br>coverage ≥ 80 %               | WAIT |
| 5  | **Feedback API & loop**                 | `/feedback`, DB table, retrain-приоритет                      | ✔ POST /feedback 204 <br>Запись в PG <br>coverage ≥ 82 %                                             | WAIT |
| 6  | **Cron bootstrap / datasets**           | `bootstrap_train.py`, `fetch_datasets.py`                     | ✔ Первая индексация OK, повтор — skip <br>`validate_datasets.py` зелёный                            | WAIT |
| 7  | **Cron retrain_daily + verify_plan**    | `retrain_daily.py`, `verify_testplan.py`                      | ✔ Hash/etag логируется <br>PR без актуальных тестов — ❌ <br>coverage ≥ 85 %                        | WAIT |
| 8  | **Chainlit GUI MVP**                    | UI “Поиск”, “Проектирование”, upload-виджет                   | ✔ Playwright E2E upload→search→generate→feedback зелёный                                            | WAIT |
| 9  | **CI/CD + Static security**             | GitHub Actions, Allure, Codecov, Flake8, MyPy, Bandit         | ✔ Coverage ≥ 90 % <br>Bandit High/Medium = 0                                                         | WAIT |
| 10 | **Helm-prod readiness**                 | values-prod.yaml, HPA, ingress, TLS, readiness-probe          | ✔ `helm install` на Kind OK <br>/health 200 <br>k6 smoke 10 RPS × 5 мин OK                           | WAIT |
| ✅ | **MVP 1 ready**                         | README, diagrams, user/dev manuals                            | ✔ Все тесты зелёные, coverage ≥ 90 % <br>Канбан строки = DONE                                        | — |

_Общие правила_:
* Перед каждым спринтом — `verify_testplan.py`.
* Патч + base64-архив выкладываются **только** после выполнения exit-criteria.
* Coverage не должен падать между итерациями.
* Один спринт ≈ 30 мин реального времени разработки.