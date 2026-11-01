# Raw → Clean → Analytics: Трехслойный DWH пайплайн

Демонстрация классической трехслойной архитектуры хранилища данных (DWH) с оркестрацией в Apache Airflow.

## Что это такое?

**Трехслойная архитектура** — это стандарт для корпоративных хранилищ данных:
1. **Raw-слой (сырой):** данные сохраняются «как есть» (без изменений). Это страховка: если что-то пойдет не так, у нас всегда есть исходник.
2. **Clean-слой (очищенный):** данные нормализуются (убираются дубли, приводятся типы, объединяются источники).
3. **Analytics-слой (витрины):** готовые таблицы для отчетов и BI.

## Что реализовано

- **Raw-слой:** Python-скрипт забирает данные из API (курсы валют) и сохраняет их в `raw.currency_rates` (JSONB).
- **Clean-слой:** Python-скрипт разворачивает JSON в нормализованную таблицу `clean.currency_rates_clean`.
- **Analytics-слой:** Python-скрипт строит витрину `analytics.currency_rates_mart` (последние курсы, топ-5 валют по курсу к USD).
- **Оркестрация:** DAG в Apache Airflow (`raw_clean_analytics_pipeline`), который запускает все три слоя по цепочке.

## Стек

- Python (Pandas, SQLAlchemy, psycopg2)
- PostgreSQL
- Apache Airflow (Docker)
- REST API

## Архитектура

```mermaid
graph LR
    A[REST API] -->|Python| B[Raw-слой<br/>raw.currency_rates]
    B -->|Python| C[Clean-слой<br/>clean.currency_rates_clean]
    C -->|Python| D[Analytics-слой<br/>analytics.currency_rates_mart]
    E[Airflow DAG] -->|Оркестрация| B
    E -->|Оркестрация| C
    E -->|Оркестрация| D
```
Как это работает
Airflow запускает DAG raw_clean_analytics_pipeline (ежедневно).

Задача raw_layer вызывает raw_loader.py (данные из API → raw.currency_rates).

Задача clean_layer вызывает clean_loader.py (JSON → clean.currency_rates_clean).

Задача analytics_layer вызывает analytics_builder.py (витрина → analytics.currency_rates_mart).

Скриншоты
Успешное выполнение DAG (все три слоя): dag_success.png

Витрина в PostgreSQL: analytics_mart.png

Ссылки
Основной проект DWH Marketplace Analytics

Data Contracts Demo

dbt CI/CD Demo
