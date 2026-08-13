# awow-acore-rudb

**Ambershard World of Warcraft: AzerothCore Russian Database Localization**
Порт и адаптация базы данных русской локализации (на базе наработок YTDB) для эмулятора AzerothCore.

Этот репозиторий является автономным модулем. Вы можете использовать его как в составе серверного окружения Ambershard (`awow-acore-env`, через git submodule), так и отдельно — для любого другого сервера AzerothCore.

---

## Что локализуется

Модуль добавляет `ruRU`-переводы в следующие таблицы `world`-базы:

| Файл | Таблица | Что переводит |
|---|---|---|
| `acore-rudb-01_achievement_reward_locale.sql` | `achievement_reward_locale` | Письма-награды за достижения |
| `acore-rudb-02_gossip_menu_option_locale.sql` | `gossip_menu_option_locale` | Реплики в меню диалогов NPC |
| `acore-rudb-03_npc_text_locale.sql` | `npc_text_locale` | Тексты диалогов NPC |
| `acore-rudb-04_quest_offer_reward_locale.sql` | `quest_offer_reward_locale` | Текст завершения квеста |
| `acore-rudb-05_quest_request_items_locale.sql` | `quest_request_items_locale` | Текст сдачи предметов по квесту |
| `acore-rudb-06_quest_template_locale.sql` | `quest_template_locale` | Названия и описание квестов |
| `acore-rudb-07_gameobject_template_locale.sql` | `gameobject_template_locale` | Названия игровых объектов |
| `acore-rudb-08_page_text_locale.sql` | `page_text_locale` | Тексты книг, писем и прочих свитков |

> ⚠️ На данный момент локализация не включает creature_template.name, item_template.name и т.п. — такие поля требуют других механизмов перевода (например, замены строк через DBC-файлы клиента) и пока не входят в этот репозиторий. Перевод постепенно расширяется — актуальный охват см. в таблице выше.

Каждая запись привязана к `entry` соответствующей базовой таблицы (`quest_template`, `npc_text` и т.д.) без реального внешнего ключа. Если контент вашей `world`-базы заметно отличается от той версии, под которую готовился перевод (другие ID квестов/NPC после кастомных правок или устаревшего дампа), часть строк применится «в никуда» — без ошибок, но и без видимого эффекта.

---

## Требования

- AzerothCore (актуальная master-ветка, автообновлятор БД включён)
- Клиент WoW 3.3.5a с установленной локалью `ruRU` (сам SQL не подменяет язык клиента — сервер должен быть настроен на выдачу `ruRU`-строк, а клиент — понимать кириллицу в шрифтах)
- MySQL/MariaDB с базой `world` в кодировке `utf8`/`utf8mb4`

---

## Вариант 1. Через окружение Ambershard (git submodule) — рекомендуется

Если вы разворачиваете сервер на основе `awow-acore-env`, модуль уже подключён как submodule:

```
[submodule "data/sql/custom/db_world/awow-acore-rudb"]
    path = data/sql/custom/db_world/awow-acore-rudb
    url = https://github.com/Ambershard/awow-acore-rudb.git
```

1. При первом клонировании окружения подтяните сабмодули:
   ```bash
   git submodule update --init --recursive
   ```
2. Чтобы обновить локализацию до последней версии:
   ```bash
   git submodule update --remote data/sql/custom/db_world/awow-acore-rudb
   ```
3. Запустите или перезапустите сервер. Ядро AzerothCore рекурсивно найдёт все `*.sql` внутри `data/sql/custom/db_world/`, применит новые (ещё не отмеченные в таблице `updates`) файлы к базе `world` и запишет их в лог применённых обновлений.

> 📁 Автообновлятор ищет файлы строго в `data/sql/custom/db_world/`, `data/sql/custom/db_auth/` и `data/sql/custom/db_characters/` (с префиксом `db_`). Путь `data/sql/custom/world/` (без `db_`) ядро не сканирует.

> 🗑️ В директорию `db_world` можно спокойно клонировать весь репозиторий как есть — `README.md`, `LICENSE`, `.gitignore`, `.git/` и т.п. апдейтер игнорирует, обрабатываются только файлы с расширением `.sql`.

---

## Вариант 2. Автономная установка (без submodule)

Если вы используете этот репозиторий отдельно от `awow-acore-env`:

1. Склонируйте репозиторий в директорию вашего сервера:
   ```bash
   git clone https://github.com/Ambershard/awow-acore-rudb.git [Корень сервера]/data/sql/custom/db_world/awow-acore-rudb
   ```
   Итоговый путь к файлам должен выглядеть так: `data/sql/custom/db_world/awow-acore-rudb/*.sql`
2. Запустите или перезапустите worldserver — ядро автоматически обнаружит новые файлы и применит их.

---

## Вариант 3. Ручной импорт (через CLI / консоль)

Если хотите применить локализацию вручную, напрямую в базу данных:

> 📝 Переменные для замены:
> * `{username}` — имя пользователя вашей базы данных MySQL/MariaDB (например, `root`).
> * `{world}` — точное имя вашей базы данных игрового мира (например, `acore_world`).
>
> ⚠️ Обязательно указывайте `--default-character-set=utf8mb4`, иначе кириллица в текстах может побиться при импорте.

```bash
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-01_achievement_reward_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-02_gossip_menu_option_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-03_npc_text_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-04_quest_offer_reward_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-05_quest_request_items_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-06_quest_template_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-07_gameobject_template_locale.sql
mysql -u{username} -p --default-character-set=utf8mb4 {world} < acore-rudb-08_page_text_locale.sql
```

При ручном импорте обновления через автообновлятор ядра работать не будут отдельно от этого — если вы позже подключите тот же модуль как submodule/через `custom/`, ядро попытается применить те же файлы повторно. Ошибки это не вызовет (данные используют `REPLACE INTO`), но стоит иметь это в виду при переходе с ручного варианта на автоматический.

---

## Обновление и бэкапы

- **Рекомендуется** сделать дамп базы `world` перед первым применением модуля, особенно при ручном импорте.
- Не переименовывайте уже применённые `.sql`-файлы в репозитории — автообновлятор AzerothCore отслеживает применённые обновления по имени файла, и переименование приведёт к повторному (хоть и безопасному благодаря `REPLACE INTO`) применению.
- Если у вас уже установлен другой модуль перевода (например, отдельный старый порт YTDB), возможны конфликты данных — не устанавливайте два модуля локализации одновременно без проверки пересечений.

---

## Лицензия

Этот модуль локализации является производным трудом от проекта YTDB и распространяется под свободной лицензией **GNU General Public License v2 (GPL-2.0)**.

Вы можете свободно модифицировать и распространять эти SQL-скрипты при условии, что ваши изменения также будут открыты и опубликованы под лицензией GPL v2. Подробности см. в файле [LICENSE](LICENSE).

---

## Поддержка

Если вы нашли ошибку в переводе или проблему с установкой — сообщите об этом в issues репозитория.

---

*Проект Ambershard, 2026.*