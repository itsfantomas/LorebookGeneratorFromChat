<p align="center">
  <img src="https://img.shields.io/badge/SillyTavern-Extension-blueviolet?style=for-the-badge" alt="SillyTavern Extension"/>
  <img src="https://img.shields.io/badge/version-1.0.1-blue?style=for-the-badge" alt="Version 1.0.1"/>
  <img src="https://img.shields.io/badge/license-AGPL--3.0-green?style=for-the-badge" alt="License AGPL-3.0"/>
  <img src="https://img.shields.io/badge/LLM_cost-$0-brightgreen?style=for-the-badge" alt="Zero LLM Cost"/>
</p>

<h1 align="center">📖 Lorebook Generator (From Chat)</h1>

<p align="center">
  <b>Transform your chat history into smart, vector-searchable memory — instantly and for free.</b><br/>
  A SillyTavern third-party extension that converts dialogue into World Info / Lorebook entries<br/>with zero token cost and full vector search support.
</p>

---

### 🌐 Language / Язык

- [🇬🇧 English](#-english)
- [🇷🇺 Русский](#-русский)

---

# 🇬🇧 English

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [How It Works — Under the Hood](#-how-it-works--under-the-hood)
- [Technology Stack](#-technology-stack)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [Usage Guide](#-usage-guide)
- [Integration with SillyTavern](#-integration-with-sillytavern)
- [Configuration Options](#-configuration-options)
- [Tips & Best Practices](#-tips--best-practices)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔍 Overview

**Lorebook Generator** is a third-party extension for [SillyTavern](https://github.com/SillyTavern/SillyTavern) that automatically converts chat history into a vector-ready Lorebook (World Info) file. Unlike LLM-based summarization solutions, this extension operates entirely **client-side** with **zero token cost** — it directly processes raw chat messages, cleans them, and packages them into structured lorebook entries optimized for semantic search.

This is ideal for:

- Building **long-term character memory** from existing roleplay sessions
- Creating a **knowledge base** from accumulated dialogue
- Preserving **narrative context** across conversations without manual data entry

---

## ✨ Key Features

| Feature                       | Description                                                                                                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **💸 Zero Token Cost**        | No LLM API calls required. Conversion is performed entirely through client-side text processing — fast, free, and offline-capable.                                  |
| **🔗 Vector-Ready Output**    | Every generated entry has the `vectorized: true` flag enabled, allowing SillyTavern to perform **semantic similarity search** (not just keyword matching).          |
| **🧹 Smart Content Cleaning** | Multi-layer sanitization pipeline removes code blocks, HTML tags, inline CSS styles, `<script>` / `<style>` elements, and other noise that degrades AI performance. |
| **✂️ Intelligent Chunking**   | Messages are grouped into logical dialogue blocks based on user responses, preserving conversational context within each entry.                                     |
| **💾 Dual Save Strategy**     | Attempts direct server save via SillyTavern API first; automatically falls back to JSON file download if server restrictions prevent file creation.                 |
| **🏷️ Auto-Naming**            | Automatically suggests a lorebook name based on the active character (e.g., `CharacterName_Lore`).                                                                  |

---

## ⚙️ How It Works — Under the Hood

### 1. Content Cleaning Pipeline (`cleanMessageContent`)

Each message passes through a multi-stage sanitization process:

````
Raw Message → Remove code blocks (```...```) → Strip <style> tags
            → Strip <script> tags → Remove all remaining HTML tags
            → Trim whitespace → Clean Text
````

This ensures that CSS rules, JavaScript snippets, stat blocks, and HTML formatting artifacts do not pollute the lorebook entries.

### 2. Smart Chunking Algorithm (`generateLorebookData`)

Messages are grouped into dialogue chunks using a conversational boundary approach:

```
[Bot msg #1] ─┐
[Bot msg #2] ─┤── Chunk 1 (ends at User response)
[User msg]   ─┘
[Bot msg #3] ─┐
[Bot msg #4] ─┤── Chunk 2 (ends at User response)
[User msg]   ─┘
[Bot msg #5] ─── Chunk 3 (last chunk, ends at chat boundary)
```

Each chunk is split when a **user message** (`is_user: true`) is encountered. This groups a character's continuous narrative with the user's reply that triggered the next turn, preserving **cause-and-effect context**.

### 3. Entry Generation (`createLorebookEntry`)

Each chunk is transformed into a lorebook entry with:

| Field         | Value         | Purpose                                       |
| ------------- | ------------- | --------------------------------------------- |
| `vectorized`  | `true`        | Enables semantic (vector) search              |
| `enabled`     | `true`        | Entry is active by default                    |
| `selective`   | `true`        | Works with selective matching logic           |
| `constant`    | `false`       | Not permanently injected                      |
| `position`    | `before_char` | Inserted before character's response          |
| `probability` | `100`         | Always included when matched                  |
| `order`       | `100`         | Default priority ordering                     |
| `key`         | `[]`          | Empty — relies on vector search, not keywords |

### 4. Save Strategy

```
Generate JSON → Try POST /api/worldinfo/create
              → Try POST /api/worldinfo/edit
              → Success? → Reload World Info
              → Failure? → Fallback: download .json file
```

CSRF protection is automatically handled via `X-CSRF-Token` header if available.

---

## 🛠 Technology Stack

| Technology                     | Usage                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **JavaScript (ES Modules)**    | Core logic — import-based module using SillyTavern's extension API (`getContext`)                      |
| **jQuery**                     | DOM manipulation, event handling, AJAX requests to SillyTavern's backend API                           |
| **HTML5**                      | Settings panel UI with form inputs, rendered within SillyTavern's extension drawer                     |
| **CSS3**                       | Custom "Nightwing" theme with glassmorphism, neon glow effects, and CSS variables for theme adaptation |
| **SillyTavern Extensions API** | Integration via `getContext()`, World Info endpoints (`/api/worldinfo/create`, `/api/worldinfo/edit`)  |
| **Font Awesome**               | Icon for the generate button (`fa-book-journal-whills`)                                                |
| **Blob API**                   | Fallback file generation for browser-side `.json` download                                             |

---

## 📁 Project Structure

```
LorebookGeneratorFromChat/
├── manifest.json      # Extension metadata (name, version, author, entry points)
├── index.js           # Core logic: cleaning, chunking, generation, save/download
├── settings.html      # UI panel injected into SillyTavern's extensions sidebar
├── style.css          # Nightwing-themed styles with glassmorphism and neon accents
├── LICENSE            # GNU Affero General Public License v3.0
└── README.md          # This file
```

---

## 📦 Installation

### Via SillyTavern UI (Recommended)

1. Open **SillyTavern** in your browser.
2. Navigate to **Extensions** → **Install Extension**.
3. Paste the repository URL:
   ```
   https://github.com/itsfantomas/LorebookGeneratorFromChat
   ```
4. Click **Install** and **refresh** the page.

### Manual Installation

1. Clone the repository into your SillyTavern extensions folder:
   ```bash
   cd SillyTavern/public/scripts/extensions/third-party/
   git clone https://github.com/itsfantomas/LorebookGeneratorFromChat.git
   ```
2. Restart SillyTavern and refresh the browser.

---

## 🚀 Usage Guide

1. **Open a chat** with the character whose dialogue you want to convert into memory.
2. Open the **Extensions** menu from the top panel.
3. Find the **Lorebook Generator** panel in the extensions sidebar.
4. **Configure the parameters:**
   - **Lorebook Name** — auto-filled with `CharacterName_Lore` when you expand the panel; editable.
   - **Start From** — first message index to include (default: `0`, the beginning).
   - **End At** — last message index (leave blank to include everything until the end).
   - **Scan Depth** — controls the `scan_depth` metadata in the output (default: `10`).
5. Click **"Create and Save"** (Создать и Сохранить).
6. ✅ **Done!**
   - Refresh the page.
   - Go to the **World Info** menu (Globe icon 🌐).
   - Your new lorebook will be there — attach it to the chat or character.

---

## 🔌 Integration with SillyTavern

### API Endpoints Used

| Endpoint                | Method | Purpose                                          |
| ----------------------- | ------ | ------------------------------------------------ |
| `/api/worldinfo/create` | `POST` | Creates a new World Info file on the server      |
| `/api/worldinfo/edit`   | `POST` | Writes the generated lorebook data into the file |

### Extension API

The extension uses `getContext()` from SillyTavern's extensions API to access:

- `context.chat` — the full chat message history
- `context.chatId` — current active chat identifier
- `context.characterId` / `context.characters` — active character data for auto-naming
- `context.user_name` — the user's display name
- `context.loadWorldInfo()` — triggers a UI refresh of the World Info panel

### Vector Search Requirement

> [!IMPORTANT]
> For the generated lorebook to work effectively, you **must** have a **Vectorization Source** configured in your SillyTavern settings. This requires either:
>
> - **SillyTavern Extras** API with a vector provider, or
> - A **local embedding model** (e.g., via `sentence-transformers`)
>
> Without vectorization, entries won't be semantically matched — they'll only work if you manually add keywords.

---

## ⚙️ Configuration Options

| Parameter         | Default              | Description                                                                                            |
| ----------------- | -------------------- | ------------------------------------------------------------------------------------------------------ |
| **Lorebook Name** | `CharacterName_Lore` | Output file name. Auto-sanitized (special characters replaced with `_`).                               |
| **Start From**    | `0`                  | Index of the first message to process. Use to skip intro/system messages.                              |
| **End At**        | _(empty)_            | Index of the last message. Leave blank for full chat. Useful for segmenting arcs.                      |
| **Scan Depth**    | `10`                 | Metadata value for `scan_depth` in the lorebook. Determines how deep SillyTavern searches for matches. |

---

## 💡 Tips & Best Practices

### Optimizing Lorebook Quality

1. **Segment long chats by arc.** If your chat spans 500+ messages with multiple story arcs, generate separate lorebooks per arc using the Start/End range (e.g., messages 0–150, 151–300). This improves vector search relevance.

2. **Clean up before generating.** Delete or swipe away broken/repeated messages before generating. The extension cleans code blocks and HTML, but it can't fix incoherent text.

3. **Combine with manual entries.** After generation, review the lorebook in World Info and add high-priority manual entries with specific keywords for critical lore (character names, locations, important events).

### Vector Search Optimization

4. **Use a quality embedding model.** The semantic search quality depends entirely on your vectorization provider. Models like `all-MiniLM-L6-v2` offer a good balance of speed and quality.

5. **Adjust scan depth wisely.** A higher `scan_depth` means SillyTavern looks further back in the conversation for matching entries, but increases processing time. The default of `10` is good for most use cases.

### Workflow Integration

6. **Use with [Janitor Exporter](https://github.com/itsfantomas/janitor-exporter).** If you need to extract chat metadata from JanitorAI for bulk export before processing with this tool, the Janitor Exporter companion tool can help.

7. **Attach lorebooks strategically.** You can attach the generated lorebook globally (to the character) or locally (to a specific chat). For ongoing stories, local attachment avoids cross-contamination between different story branches.

---

## 🔧 Troubleshooting

| Problem                                  | Solution                                                                                                                           |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **"Open a chat!" error**                 | Make sure you have an active chat open with a character before clicking generate.                                                  |
| **"Enter lorebook name!" error**         | Type a name in the input field, or expand the panel to auto-fill it.                                                               |
| **Save fails, downloads .json instead**  | Your server's security settings may block file creation. Import the downloaded `.json` manually via World Info → "Import content". |
| **Lorebook entries don't match in chat** | Ensure a vectorization source is configured. Without it, entries with empty `key[]` arrays won't trigger.                          |
| **Generated content contains garbage**   | The extension strips code blocks, HTML, and scripts. If garbage remains, check if the source messages contain unusual formatting.  |

---

## 🤝 Contributing

Contributions are welcome! This project is licensed under AGPL-3.0, so any modifications distributed must also be open source.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **GNU Affero General Public License v3.0** — see the [LICENSE](LICENSE) file for details.

---

---

# 🇷🇺 Русский

## 📋 Содержание

- [Обзор](#-обзор)
- [Ключевые возможности](#-ключевые-возможности)
- [Как это работает — изнутри](#-как-это-работает--изнутри)
- [Стек технологий](#-стек-технологий)
- [Структура проекта](#-структура-проекта)
- [Установка](#-установка)
- [Руководство по использованию](#-руководство-по-использованию)
- [Интеграция с SillyTavern](#-интеграция-с-sillytavern)
- [Параметры настройки](#-параметры-настройки)
- [Советы и лучшие практики](#-советы-и-лучшие-практики)
- [Решение проблем](#-решение-проблем)
- [Участие в разработке](#-участие-в-разработке)
- [Лицензия](#-лицензия-1)

---

## 🔍 Обзор

**Lorebook Generator** — стороннее расширение для [SillyTavern](https://github.com/SillyTavern/SillyTavern), которое автоматически конвертирует историю чата в файл Lorebook (World Info), готовый к векторному поиску. В отличие от LLM-решений, расширение работает полностью **на стороне клиента** с **нулевыми затратами токенов** — оно напрямую обрабатывает сырые сообщения, очищает их и упаковывает в структурированные записи лорбука, оптимизированные для семантического поиска.

Идеально для:

- Создания **долговременной памяти персонажа** из существующих ролевых сессий
- Формирования **базы знаний** из накопленного диалога
- Сохранения **нарративного контекста** между разговорами без ручного ввода данных

---

## ✨ Ключевые возможности

| Возможность                           | Описание                                                                                                                                           |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **💸 Нулевая стоимость**              | Не требуются вызовы LLM API. Конвертация выполняется исключительно на стороне клиента — быстро, бесплатно и без интернета.                         |
| **🔗 Готовность к векторному поиску** | Каждая запись имеет флаг `vectorized: true`, позволяющий SillyTavern выполнять **семантический поиск по смыслу** (а не только по ключевым словам). |
| **🧹 Умная очистка контента**         | Многоуровневый конвейер удаляет блоки кода, HTML-теги, CSS-стили, `<script>` / `<style>` элементы и прочий мусор, ухудшающий работу нейросети.     |
| **✂️ Интеллектуальная нарезка**       | Сообщения группируются в логические блоки на основе ответов пользователя, сохраняя контекст диалога.                                               |
| **💾 Двойная стратегия сохранения**   | Сначала пытается сохранить через API SillyTavern; при ошибке автоматически предлагает скачать JSON-файл.                                           |
| **🏷️ Авто-именование**                | Автоматически подставляет имя лорбука на основе активного персонажа (напр., `ИмяПерсонажа_Lore`).                                                  |

---

## ⚙️ Как это работает — изнутри

### 1. Конвейер очистки (`cleanMessageContent`)

Каждое сообщение проходит через многоступенчатую санитизацию:

````
Сырое сообщение → Удаление блоков кода (```...```) → Удаление <style>
                → Удаление <script> → Удаление HTML-тегов
                → Очистка пробелов → Чистый текст
````

Это гарантирует, что CSS-правила, JavaScript-код, блоки со статами и артефакты HTML-форматирования не попадут в записи лорбука.

### 2. Алгоритм нарезки (`generateLorebookData`)

Сообщения группируются в диалоговые чанки по границам разговора:

```
[Бот сообщ. #1] ─┐
[Бот сообщ. #2] ─┤── Чанк 1 (заканчивается на ответ пользователя)
[Пользователь]  ─┘
[Бот сообщ. #3] ─┐
[Бот сообщ. #4] ─┤── Чанк 2 (заканчивается на ответ пользователя)
[Пользователь]  ─┘
[Бот сообщ. #5] ─── Чанк 3 (последний, заканчивается на границе чата)
```

Каждый чанк разделяется при обнаружении **сообщения пользователя** (`is_user: true`). Это группирует непрерывное повествование персонажа с ответом пользователя, сохраняя **контекст причинно-следственных связей**.

### 3. Генерация записей (`createLorebookEntry`)

Каждый чанк преобразуется в запись лорбука:

| Поле          | Значение      | Назначение                                  |
| ------------- | ------------- | ------------------------------------------- |
| `vectorized`  | `true`        | Включает семантический (векторный) поиск    |
| `enabled`     | `true`        | Запись активна по умолчанию                 |
| `selective`   | `true`        | Работает с логикой выборочного соответствия |
| `constant`    | `false`       | Не внедряется постоянно                     |
| `position`    | `before_char` | Вставляется перед ответом персонажа         |
| `probability` | `100`         | Всегда включается при совпадении            |
| `key`         | `[]`          | Пусто — опирается на векторный поиск        |

### 4. Стратегия сохранения

```
Генерация JSON → POST /api/worldinfo/create
               → POST /api/worldinfo/edit
               → Успех? → Перезагрузка World Info
               → Ошибка? → Фоллбэк: скачивание .json файла
```

Защита CSRF автоматически обрабатывается через заголовок `X-CSRF-Token`.

---

## 🛠 Стек технологий

| Технология                     | Применение                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **JavaScript (ES Modules)**    | Основная логика — модуль с импортами через API расширений SillyTavern (`getContext`)                   |
| **jQuery**                     | Манипуляция DOM, обработка событий, AJAX-запросы к бэкенду SillyTavern                                 |
| **HTML5**                      | UI панель настроек с формами ввода, рендерится в боковой панели расширений                             |
| **CSS3**                       | Кастомная тема «Nightwing» с glassmorphism, неоновым свечением и CSS-переменными для адаптации         |
| **SillyTavern Extensions API** | Интеграция через `getContext()`, эндпоинты World Info (`/api/worldinfo/create`, `/api/worldinfo/edit`) |
| **Font Awesome**               | Иконка кнопки генерации (`fa-book-journal-whills`)                                                     |
| **Blob API**                   | Генерация файла на стороне браузера для скачивания `.json` при ошибке сервера                          |

---

## 📁 Структура проекта

```
LorebookGeneratorFromChat/
├── manifest.json      # Метаданные расширения (имя, версия, автор, точки входа)
├── index.js           # Основная логика: очистка, нарезка, генерация, сохранение
├── settings.html      # UI-панель, внедряемая в боковую панель расширений SillyTavern
├── style.css          # Стили Nightwing: glassmorphism и неоновые акценты
├── LICENSE            # GNU Affero General Public License v3.0
└── README.md          # Этот файл
```

---

## 📦 Установка

### Через интерфейс SillyTavern (рекомендуется)

1. Откройте **SillyTavern** в браузере.
2. Перейдите в **Extensions** → **Install Extension**.
3. Вставьте ссылку на репозиторий:
   ```
   https://github.com/itsfantomas/LorebookGeneratorFromChat
   ```
4. Нажмите **Install** и **обновите** страницу.

### Ручная установка

1. Клонируйте репозиторий в папку расширений SillyTavern:
   ```bash
   cd SillyTavern/public/scripts/extensions/third-party/
   git clone https://github.com/itsfantomas/LorebookGeneratorFromChat.git
   ```
2. Перезапустите SillyTavern и обновите браузер.

---

## 🚀 Руководство по использованию

1. **Откройте чат** с персонажем, диалог которого вы хотите превратить в память.
2. Откройте меню **Extensions** в верхней панели.
3. Найдите панель **Lorebook Generator** в боковой панели расширений.
4. **Настройте параметры:**
   - **Название лорбука** — автоматически заполняется как `ИмяПерсонажа_Lore`; можно изменить.
   - **Начать с** — индекс первого сообщения (по умолчанию: `0`, начало чата).
   - **Закончить на** — индекс последнего сообщения (оставьте пустым для обработки всего чата).
   - **Scan Depth** — глубина сканирования в метаданных лорбука (по умолчанию: `10`).
5. Нажмите **«Создать и Сохранить»**.
6. ✅ **Готово!**
   - Обновите страницу.
   - Зайдите в меню **World Info** (иконка Глобуса 🌐).
   - Ваш лорбук будет там — подключите его к чату или персонажу.

---

## 🔌 Интеграция с SillyTavern

### Используемые API-эндпоинты

| Эндпоинт                | Метод  | Назначение                               |
| ----------------------- | ------ | ---------------------------------------- |
| `/api/worldinfo/create` | `POST` | Создаёт новый файл World Info на сервере |
| `/api/worldinfo/edit`   | `POST` | Записывает данные лорбука в файл         |

### API расширений

Расширение использует `getContext()` из API расширений SillyTavern для доступа к:

- `context.chat` — полная история сообщений чата
- `context.chatId` — идентификатор активного чата
- `context.characterId` / `context.characters` — данные персонажа для авто-именования
- `context.user_name` — отображаемое имя пользователя
- `context.loadWorldInfo()` — принудительное обновление панели World Info

### Требования к векторному поиску

> [!IMPORTANT]
> Для эффективной работы лорбука **необходимо** настроить **Vectorization Source** в настройках SillyTavern. Для этого нужен один из вариантов:
>
> - **SillyTavern Extras** API с провайдером векторизации, или
> - **Локальная модель эмбеддингов** (например, через `sentence-transformers`)
>
> Без векторизации записи с пустым массивом `key[]` не будут срабатывать по ключевым словам.

---

## ⚙️ Параметры настройки

| Параметр             | По умолчанию        | Описание                                                                           |
| -------------------- | ------------------- | ---------------------------------------------------------------------------------- |
| **Название лорбука** | `ИмяПерсонажа_Lore` | Имя файла. Спецсимволы автоматически заменяются на `_`.                            |
| **Начать с**         | `0`                 | Индекс первого обрабатываемого сообщения. Позволяет пропустить вступление.         |
| **Закончить на**     | _(пусто)_           | Индекс последнего сообщения. Пустое поле = весь чат. Полезно для сегментации арок. |
| **Scan Depth**       | `10`                | Значение `scan_depth` в метаданных лорбука. Определяет глубину поиска SillyTavern. |

---

## 💡 Советы и лучшие практики

### Оптимизация качества лорбука

1. **Сегментируйте длинные чаты по аркам.** Если чат содержит 500+ сообщений с несколькими сюжетными арками, создавайте отдельные лорбуки для каждой арки через диапазон Начать/Закончить (напр., 0–150, 151–300). Это улучшает релевантность векторного поиска.

2. **Почистите чат перед генерацией.** Удалите или перелистните поломанные/повторяющиеся сообщения. Расширение чистит код и HTML, но не может исправить бессвязный текст.

3. **Комбинируйте с ручными записями.** После генерации просмотрите лорбук в World Info и добавьте приоритетные ручные записи с ключевыми словами для критически важного лора (имена персонажей, локации, важные события).

### Оптимизация векторного поиска

4. **Используйте качественную модель эмбеддингов.** Качество семантического поиска полностью зависит от провайдера векторизации. Модели вроде `all-MiniLM-L6-v2` дают хороший баланс скорости и качества.

5. **Настраивайте глубину сканирования осознанно.** Большая `scan_depth` заставляет SillyTavern искать совпадения глубже в разговоре, но увеличивает время обработки. Значение `10` подходит для большинства случаев.

### Интеграция в рабочий процесс

6. **Используйте вместе с [Janitor Exporter](https://github.com/itsfantomas/janitor-exporter).** Если нужно извлечь метаданные чатов из JanitorAI для массового экспорта перед обработкой этим инструментом, Janitor Exporter поможет подготовить данные.

7. **Подключайте лорбуки стратегически.** Лорбук можно подключить глобально (к персонажу) или локально (к конкретному чату). Для продолжающихся историй локальное подключение предотвращает «загрязнение» между разными сюжетными ветками.

---

## 🔧 Решение проблем

| Проблема                                    | Решение                                                                                                                               |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Ошибка «Откройте чат!»**                  | Убедитесь, что у вас открыт активный чат с персонажем перед генерацией.                                                               |
| **Ошибка «Введите имя лорбука!»**           | Введите имя в поле ввода или раскройте панель для автозаполнения.                                                                     |
| **Сохранение не сработало, скачался .json** | Настройки безопасности сервера могут блокировать создание файлов. Импортируйте скачанный `.json` через World Info → «Import content». |
| **Записи лорбука не срабатывают в чате**    | Убедитесь, что настроен источник векторизации. Без него записи с пустым `key[]` не будут триггериться.                                |
| **В сгенерированном контенте мусор**        | Расширение фильтрует код, HTML и скрипты. Если мусор остался, проверьте исходные сообщения на необычное форматирование.               |

---

## 🤝 Участие в разработке

Вклад приветствуется! Проект лицензирован под AGPL-3.0, поэтому любые распространяемые модификации также должны быть открытыми.

1. Форкните репозиторий
2. Создайте ветку для фичи (`git checkout -b feature/my-feature`)
3. Зафиксируйте изменения (`git commit -m 'Add feature'`)
4. Отправьте ветку (`git push origin feature/my-feature`)
5. Откройте Pull Request

---

## 📄 Лицензия

Проект лицензирован под **GNU Affero General Public License v3.0** — подробности в файле [LICENSE](LICENSE).

---

<p align="center"><i>Created with ❤️ for the SillyTavern community</i></p>

