# Техническое видение проекта

*LLM-ассистент для первичной консультации клиентов в Telegram*

## Технологии

**Backend & Bot:**
- **Python 3.11+** - простой, быстрый старт, отличная экосистема для LLM
- **aiogram 3.x** - современная асинхронная библиотека для Telegram Bot API
- **python-dotenv** - для конфигурации через переменные окружения

**LLM Integration:**
- **OpenRouter API** - доступ к различным моделям по выгодным ценам
- **openai** Python библиотека - для работы через совместимый API

**Development & Tools:**
- **uv** - современное управление зависимостями и окружением Python
- **pytest** - тестирование
- **make** - автоматизация сборки, запуска, деплоя
- **Git** для версионирования

**Deployment:**
- **Docker** - контейнеризация приложения
- **Docker Compose** - для простого локального развертывания

**Infrastructure:**
- **Один Python файл** для старта (monolith approach)
- **Данные компании и услуг** - в системном промпте (без БД)

## Принципы разработки

**KISS - Keep It Simple, Stupid**
- Решаем только текущую задачу, не планируем на 10 лет вперед
- Минимум абстракций - прямолинейный код
- Если функцию можно не писать - не пишем

**Функциональный подход без ООП**
- Разбиваем код на логические функции по доменам (telegram, llm, config)
- Каждая функция делает одну вещь и делает её хорошо
- Данные передаем как параметры, избегаем глобального состояния
- Чистые функции где возможно - легче тестировать и понимать

**Модульная организация**
- Логическое разделение на компоненты: `bot.py`, `llm_client.py`, `handlers.py`
- Каждый модуль отвечает за свою область ответственности
- Простые импорты без сложных зависимостей
- Тестируем каждый компонент изолированно

**Быстрая итерация**
- Запускаем MVP за 1-2 дня разработки
- Тестируем с реальными пользователями как можно раньше
- Фокус на функциональности, не на сложной архитектуре

**Явность над магией**
- Явные конфигурации через переменные окружения
- Простая обработка ошибок без сложных фреймворков
- Читаемый код важнее "умного" кода
- Дублирование кода лучше преждевременной абстракции

**MVP-мышление**
- Делаем только то, что проверяет гипотезу
- Откладываем все "хорошо бы иметь" фичи
- Идеальный код появится после проверки идеи

## Структура проекта

```
llm-intense-01/
├── main.py                  # Основной файл запуска бота
├── config.py                # Конфигурация и переменные окружения
├── bot/                     # Telegram-функциональность
│   ├── __init__.py          # Инициализация модуля
│   └── handlers.py          # Обработчики сообщений
├── llm/                     # LLM-функциональность
│   ├── __init__.py          # Инициализация модуля
│   └── client.py            # Работа с OpenRouter API
├── tests/                   # Тесты
│   ├── test_bot_handlers.py # Тесты bot-обработчиков
│   ├── test_llm_client.py   # Тесты LLM интеграции
│   └── conftest.py          # Общие настройки тестов
├── docker/                  # Docker-файлы
│   ├── Dockerfile           # Образ приложения
│   └── docker-compose.yml   # Локальное развертывание
├── doc/                     # Документация
│   ├── vision.md            # Техническое видение (этот файл)
│   └── product_idea.md      # Описание идеи продукта
├── .env.example             # Пример переменных окружения
├── .gitignore              # Исключения Git
├── pyproject.toml          # Конфигурация uv и проекта
├── Makefile                # Автоматизация сборки и запуска
└── README.md               # Описание проекта и инструкции
```

**Преимущества доменной структуры:**
- **Логическое разделение** - bot/ и llm/ как отдельные области ответственности
- **Простота расширения** - каждый домен развивается независимо
- **Четкие границы** - понятно, где что искать
- **Легкое тестирование** - каждый домен тестируется изолированно

## Архитектура проекта

**Тип архитектуры:** Линейная пайплайн-архитектура с доменным разделением

**Схема потока данных:**
```
Пользователь → Telegram Bot API → bot/handlers.py → llm/client.py → OpenRouter API
     ↑                                                                        ↓
     ↓                                                                  Ответ LLM
Ответ пользователю ← Telegram Bot API ← bot/handlers.py ←──────────────────────┘
```

**Основные компоненты:**

1. **main.py** - точка входа
   - Инициализация бота aiogram
   - Регистрация обработчиков из bot/
   - Запуск polling

2. **bot/** - директория Telegram-функциональности
   - **handlers.py** - основные обработчики сообщений
   - **__init__.py** - инициализация модуля

3. **llm/** - директория LLM-функциональности  
   - **client.py** - работа с OpenRouter API
   - **__init__.py** - инициализация модуля

4. **config.py** - конфигурация
   - Переменные окружения
   - Системные промпты
   - Настройки бота и LLM

**Принципы архитектуры:**
- **Доменное разделение** - bot/ и llm/ как отдельные области ответственности
- **Без состояния** - каждый запрос независим
- **Простые интерфейсы** - функции как точки входа в каждый домен
- **Отказоустойчивость** - простая обработка ошибок через try/except

## Модель данных

**Принцип:** Stateless с ограниченным контекстом диалога

**Основные типы данных:**

1. **Конфигурационные данные** (в config.py)
   ```python
   # Настройки бота
   TELEGRAM_TOKEN: str
   OPENROUTER_API_KEY: str
   OPENROUTER_MODEL: str  # например, "anthropic/claude-3.5-haiku"
   
   # Настройки контекста
   MAX_DIALOG_MESSAGES: int = 10  # максимум сообщений в истории
   MAX_CONTEXT_TOKENS: int = 4000  # максимум токенов контекста
   
   # Системный промпт
   SYSTEM_PROMPT: str
   
   # Информация о компании
   COMPANY_INFO: str
   SERVICES_LIST: str
   ```

2. **Данные сообщения** (временные, в памяти)
   ```python
   # Входящее сообщение пользователя
   user_message: str
   user_id: int
   chat_id: int
   
   # История диалога (ограниченная)
   dialog_history: list[dict]  # последние N сообщений
   # Формат OpenAI API:
   # [
   #   {"role": "system", "content": "системный промпт"},
   #   {"role": "user", "content": "сообщение пользователя"},
   #   {"role": "assistant", "content": "ответ бота"},
   #   {"role": "user", "content": "следующее сообщение"}
   # ]
   ```

3. **Данные ответа LLM** (временные)
   ```python
   # Ответ от OpenRouter
   llm_response: str
   tokens_used: int
   model_used: str
   ```

**Принципы обработки данных:**
- **Без состояния** - каждый запрос независим
- **Ограниченный контекст** - максимум N последних сообщений
- **Простые типы** - строки, числа, словари
- **Автоматическая обрезка** - при превышении лимита удаляем старые сообщения

**Управление контекстом диалога:**
- **По количеству:** максимум `MAX_DIALOG_MESSAGES` сообщений
- **По размеру:** максимум `MAX_CONTEXT_TOKENS` токенов
- **Стратегия обрезки:** FIFO (первые удаляются первыми)
- **Сохранение системного промпта:** всегда первое сообщение с `role="system"`

**Алгоритм ограничения контекста:**
1. Добавляем новое сообщение в историю
2. Проверяем лимит по количеству сообщений
3. Проверяем лимит по токенам
4. Удаляем старые сообщения при превышении
5. Отправляем на LLM

## Работа с LLM

**Основные принципы:**
- Простые синхронные вызовы через OpenAI-совместимый API  
- Использование стандартного формата сообщений OpenAI
- Сохранение контекста диалога в памяти
- Простая обработка и отправка ответов

**Сохранение истории диалога:**
```python
# bot/handlers.py - глобальный словарь для хранения истории
dialog_contexts: dict[int, list[dict]] = {}  # chat_id -> messages

def get_or_create_context(chat_id: int) -> list[dict]:
    """Получает или создает контекст диалога для chat_id"""
    if chat_id not in dialog_contexts:
        dialog_contexts[chat_id] = [
            {"role": "system", "content": SYSTEM_PROMPT}
        ]
    return dialog_contexts[chat_id]

def add_message_to_context(chat_id: int, role: str, content: str):
    """Добавляет сообщение в контекст с проверкой лимитов"""
    context = get_or_create_context(chat_id)
    context.append({"role": role, "content": content})
    
    # Обрезаем контекст если превышен лимит
    while len(context) > MAX_DIALOG_MESSAGES + 1:  # +1 для system prompt
        # Удаляем старые сообщения, но сохраняем system prompt
        if context[1]["role"] in ["user", "assistant"]:
            context.pop(1)
```

**Интеграция с OpenRouter:**
```python
# llm/client.py
def generate_response(messages: list[dict], model: str = "anthropic/claude-3.5-haiku") -> dict:
    """Генерирует ответ через LLM с историей диалога"""
    client = create_llm_client()
    try:
        response = client.chat.completions.create(
            model=model,
            messages=messages,
            temperature=0.7,
            max_tokens=1000,
        )
        return {
            "content": response.choices[0].message.content,
            "tokens_used": response.usage.total_tokens,
            "model": response.model,
            "success": True
        }
    except Exception as e:
        return {
            "content": "Извините, произошла ошибка. Попробуйте позже.",
            "success": False,
            "error": str(e)
        }
```

**Простая обработка ответов:**
```python
# bot/handlers.py
async def send_llm_response(message: types.Message, llm_response: str):
    """Отправляет ответ LLM с обработкой длинных сообщений"""
    MAX_MESSAGE_LENGTH = 4000
    
    if len(llm_response) <= MAX_MESSAGE_LENGTH:
        await message.reply(llm_response, parse_mode="Markdown")
    else:
        # Разбиваем длинное сообщение на части
        parts = [llm_response[i:i+MAX_MESSAGE_LENGTH] 
                for i in range(0, len(llm_response), MAX_MESSAGE_LENGTH)]
        
        for part in parts:
            await message.reply(part, parse_mode="Markdown")
            await asyncio.sleep(0.5)  # небольшая задержка между частями
```

**Основной flow обработки:**
1. Получаем сообщение пользователя
2. Добавляем в контекст диалога
3. Отправляем весь контекст в LLM  
4. Получаем ответ и добавляем в контекст
5. Обрабатываем и отправляем ответ пользователю

## Мониторинг LLM

**Принцип:** Простое логгирование ключевых метрик для контроля расходов и качества

**Основные метрики для отслеживания:**
- **Количество запросов** - сколько обращений к LLM
- **Использование токенов** - расход токенов по запросам
- **Время ответа** - скорость работы LLM
- **Ошибки** - сбои в работе с API
- **Стоимость** - приблизительная оценка затрат

**Простая реализация:**
```python
# llm/client.py
import logging
import time
from datetime import datetime

# Настройка логгера для LLM метрик
llm_logger = logging.getLogger("llm_metrics")

def generate_response(messages: list[dict], model: str = "anthropic/claude-3.5-haiku") -> dict:
    """Генерирует ответ через LLM с метриками"""
    start_time = time.time()
    
    try:
        response = client.chat.completions.create(...)
        
        duration = time.time() - start_time
        tokens_used = response.usage.total_tokens
        
        # Логгируем успешный запрос
        llm_logger.info(f"LLM_SUCCESS model={model} tokens={tokens_used} duration={duration:.2f}s")
        
        return {
            "content": response.choices[0].message.content,
            "tokens_used": tokens_used,
            "model": response.model,
            "success": True,
            "duration": duration
        }
    except Exception as e:
        duration = time.time() - start_time
        llm_logger.error(f"LLM_ERROR model={model} duration={duration:.2f}s error={str(e)}")
        
        return {
            "content": "Извините, произошла ошибка. Попробуйте позже.",
            "success": False,
            "error": str(e),
            "duration": duration
        }
```

**Инструменты мониторинга:**
- **Стандартный Python logging** - запись в файлы логов
- **Простые скрипты анализа** - ежедневная статистика из логов
- **Базовые алерты** - превышение лимитов токенов/ошибок

**Что НЕ делаем (пока):**
- Сложные системы мониторинга (Prometheus, Grafana)
- Real-time дашборды
- Детальная аналитика пользователей
- A/B тестирование промптов

## Сценарии работы

**Основные пользовательские сценарии:**

**1. Первый контакт с ботом**
- Пользователь пишет `/start` или любое сообщение
- Бот приветствует и представляется как консультант компании
- Бот предлагает рассказать о своих потребностях
- Бот начинает первичную консультацию

**2. Выяснение потребностей клиента**
- Пользователь описывает свою проблему/задачу
- Бот задает уточняющие вопросы (по одному за раз)
- Бот собирает ключевую информацию:
  - Тип задачи/проблемы
  - Ожидаемый результат
  - Примерные сроки и бюджет (опционально)
- Бот подтверждает понимание запроса

**3. Рекомендация услуг**
- На основе собранной информации бот предлагает 1-3 релевантные услуги
- Бот кратко объясняет пользу каждой услуги
- Бот отвечает на дополнительные вопросы об услугах

**4. Завершение консультации**
- Бот резюмирует результаты консультации
- Бот предлагает следующие шаги:
  - Связь с менеджером для детального обсуждения
  - Получение коммерческого предложения
  - Назначение консультации/встречи
- Бот собирает контактные данные (при необходимости)

**5. Обработка ошибок и edge cases**
- Неясные/размытые запросы → бот просит уточнить
- Нерелевантные вопросы → бот вежливо перенаправляет на основную тему
- Технические проблемы → бот извиняется и предлагает повторить позже
- Агрессивные пользователи → бот остается вежливым и профессиональным

**Примеры диалогов:**
```
User: Привет, мне нужно продвинуть свой бизнес в интернете
Bot: Привет! Я консультант [Компания]. Помогу разобраться с задачами продвижения. 
     Расскажите, что именно вы хотите продвигать и какую цель преследуете?

User: У меня интернет-магазин одежды, хочу больше продаж
Bot: Понятно, интернет-магазин одежды. Уточните:
     - Сколько примерно сейчас заказов в месяц?
     - Откуда в основном приходят клиенты?
```

## Деплой

**Принцип:** Простой запуск через Docker на локальной машине

**Docker setup:**
```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install uv && uv sync
CMD ["uv", "run", "python", "main.py"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  llm-bot:
    build: .
    restart: unless-stopped
    env_file:
      - .env
```

**Makefile для удобства:**
```makefile
# Makefile
.PHONY: run stop logs build

# Запуск бота
run:
	docker-compose up -d

# Остановка бота  
stop:
	docker-compose down

# Просмотр логов
logs:
	docker-compose logs -f llm-bot

# Пересборка при изменениях
build:
	docker-compose up --build -d
```

**Быстрый старт:**
```bash
# 1. Настройка окружения
cp .env.example .env
# заполнить TELEGRAM_TOKEN и OPENROUTER_API_KEY

# 2. Запуск
make run

# 3. Готово! Бот работает
```

**Управление:**
- `make run` - запустить бота
- `make stop` - остановить бота  
- `make logs` - посмотреть логи
- `make build` - пересобрать после изменений

**Требования:**
- Docker + Docker Compose
- Интернет для Telegram API и OpenRouter

## Подход к конфигурированию

**Принцип:** Все через переменные окружения + один файл конфигурации

**Структура конфигурации:**

**1. Переменные окружения (.env):**
```bash
# .env
# Обязательные настройки
TELEGRAM_TOKEN=your_telegram_bot_token
OPENROUTER_API_KEY=your_openrouter_key

# Настройки LLM
OPENROUTER_MODEL=anthropic/claude-3.5-haiku
LLM_TEMPERATURE=0.7
LLM_MAX_TOKENS=1000

# Настройки контекста диалога
MAX_DIALOG_MESSAGES=10
MAX_CONTEXT_TOKENS=4000

# Настройки логирования
LOG_LEVEL=INFO
```

**2. Конфигурационный модуль (config.py):**
```python
import os
from dotenv import load_dotenv

load_dotenv()

# Обязательные настройки
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
OPENROUTER_API_KEY = os.getenv("OPENROUTER_API_KEY")

# Настройки LLM
OPENROUTER_MODEL = os.getenv("OPENROUTER_MODEL", "anthropic/claude-3.5-haiku")
LLM_TEMPERATURE = float(os.getenv("LLM_TEMPERATURE", "0.7"))
LLM_MAX_TOKENS = int(os.getenv("LLM_MAX_TOKENS", "1000"))

# Настройки контекста
MAX_DIALOG_MESSAGES = int(os.getenv("MAX_DIALOG_MESSAGES", "10"))
MAX_CONTEXT_TOKENS = int(os.getenv("MAX_CONTEXT_TOKENS", "4000"))

# Системный промпт
SYSTEM_PROMPT = """
Ты — LLM-ассистент компании [Название] для первичной консультации клиентов.
Цель: понять задачу клиента, рекомендовать услуги, собрать информацию.
Правила: краткость, 1 вопрос за раз, не выдумывай факты.
"""

# Информация о компании (можно вынести в отдельный файл при росте)
COMPANY_INFO = "Краткое описание компании и услуг"
```

**3. Пример файла (.env.example):**
```bash
# .env.example - шаблон для разработчиков
TELEGRAM_TOKEN=your_telegram_bot_token_here
OPENROUTER_API_KEY=your_openrouter_key_here
OPENROUTER_MODEL=anthropic/claude-3.5-haiku
LLM_TEMPERATURE=0.7
LLM_MAX_TOKENS=1000
MAX_DIALOG_MESSAGES=10
MAX_CONTEXT_TOKENS=4000
LOG_LEVEL=INFO
```

**Принципы конфигурирования:**
- **Все секреты через .env** - никаких токенов в коде
- **Разумные дефолты** - работает из коробки
- **Валидация при старте** - проверяем обязательные переменные
- **Никаких сложных форматов** - только строки и числа

## Подход к логгированию

**Принцип:** Простое логгирование в файлы + вывод в консоль для разработки

**Структура логирования:**

**1. Основные логи (основные события):**
```python
# Настройка в main.py
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/app.log'),
        logging.StreamHandler()  # вывод в консоль
    ]
)

logger = logging.getLogger(__name__)
```

**2. Специализированные логгеры:**
```python
# bot/handlers.py - логи бота
bot_logger = logging.getLogger("bot")

# llm/client.py - логи LLM (уже описанные в разделе мониторинга)  
llm_logger = logging.getLogger("llm_metrics")
```

**3. Что логгируем:**
- **INFO:** Старт/остановка бота, входящие сообщения пользователей
- **DEBUG:** Детали обработки сообщений (только в разработке)
- **ERROR:** Ошибки Telegram API, ошибки LLM, критические сбои
- **LLM_METRICS:** Отдельные логи для мониторинга LLM (как описано выше)

**4. Пример логирования в коде:**
```python
# bot/handlers.py
async def handle_message(message: types.Message):
    bot_logger.info(f"New message from user {message.from_user.id}: {message.text[:50]}...")
    
    try:
        # обработка сообщения
        llm_response = await get_llm_response(...)
        bot_logger.info(f"Response sent to user {message.from_user.id}")
    except Exception as e:
        bot_logger.error(f"Error handling message from {message.from_user.id}: {e}")
```

**5. Ротация логов (простая):**
```python
# config.py - расширенная настройка при необходимости
from logging.handlers import RotatingFileHandler

# Максимум 10MB на файл, 5 backup файлов
handler = RotatingFileHandler(
    'logs/app.log', 
    maxBytes=10*1024*1024, 
    backupCount=5
)
```

**Принципы логгирования:**
- **Простота:** Стандартный Python logging
- **Читаемость:** Понятный формат с временными метками
- **Минимализм:** Логгируем только важное
- **Отладка:** Консольный вывод в разработке, файлы в продакшене

---

**Документ готов!** 🎉

*Техническое видение проекта LLM-ассистента для первичной консультации клиентов завершено. Все разделы проработаны с учетом принципов KISS и максимальной простоты для быстрого MVP.*
