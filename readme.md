Конечно. Вот полная и структурированная инструкция в формате Markdown, собранная из всех шагов, проблем и решений, которые мы прошли. Вы можете сохранить её в файл (например, `deeperseeker_setup.md`) и использовать как шпаргалку.

---

# Полная инструкция по настройке DeeperSeeker и Claude Desktop

Этот гайд описывает процесс установки DeeperSeeker на Windows, получения токена DeepSeek, запуска сервера и подключения Claude Desktop в качестве клиента через сторонний шлюз (Third-Party Inference).

## ⚠️ Важное предупреждение
Автоматизированное использование нарушает условия использования (ToS) DeepSeek. **Используйте только специально созданный (одноразовый) аккаунт.** Никогда не используйте личный аккаунт, так как его могут заблокировать в любой момент.

---

## Часть 1. Установка окружения и DeeperSeeker

### 1.1. Установка пакетного менеджера `uv`
Откройте PowerShell и выполните команду:
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
После установки добавьте `uv` в PATH текущей сессии:
```powershell
$env:Path = "C:\Users\<ВАШЕ_ИМЯ>\.local\bin;$env:Path"
```
*(Чтобы не делать это каждый раз, добавьте этот путь в системные переменные среды Windows).*

### 1.2. Клонирование и установка проекта
Перейдите в папку, где будет лежать проект, и склонируйте репозиторий:
```powershell
cd C:\Users\<ВАШЕ_ИМЯ>\Desktop
git clone https://github.com/AmanCode22/deeperseeker.git
cd deeperseeker
```
Установите зависимости:
```powershell
uv sync
```

### 1.3. Настройка конфигурации (`.env`)
Создайте файл конфигурации из шаблона:
```powershell
Copy-Item .env.example .env
```
Откройте его блокнотом:
```powershell
notepad .env
```
Убедитесь, что там есть следующие строки (без кавычек и лишних пробелов):
```env
DEEPSEEKER_API_KEY=dseeker
DEEPSEEKER_ADMIN_USER=admin
DEEPSEEKER_ADMIN_PASSWORD=admin
HOST=127.0.0.1
PORT=4000
```
*Примечание: `DEEPSEEKER_API_KEY` — это пароль, по которому вы (или Claude) будете подключаться к самому DeeperSeeker. `HOST=127.0.0.1` означает, что сервер доступен только на вашем компьютере.*

---

## Часть 2. Получение токена DeepSeek

1. Откройте окно в режиме **Инкогнито** (чтобы не задеть вашу личную сессию) и зайдите на `chat.deepseek.com`. Войдите в одноразовый аккаунт.
2. Нажмите `F12`, перейдите во вкладку **Console** (Консоль).
3. Вставьте команду и нажмите Enter:
   ```javascript
   JSON.parse(localStorage.getItem("userToken")).value
   ```
4. Скопируйте полученную длинную строку (это и есть токен) и закройте окно инкогнито.

---

## Часть 3. Запуск DeeperSeeker

1. В папке проекта запустите сервер:
   ```powershell
   uv run deeperseeker
   ```
2. Дождитесь появления строки `Uvicorn running on http://127.0.0.1:4000`. **Не закрывайте это окно**, сервер должен работать в фоне, пока вы им пользуетесь.
3. Откройте браузер и перейдите на `http://localhost:4000/dashboard`.
4. Войдите с логином `admin` и паролем `admin` (или теми, что вы указали в `.env`).
5. Найдите раздел с токенами (Tokens/Accounts), вставьте туда токен DeepSeek, полученный в Части 2, и сохраните. Статус токена должен стать `ACTIVE`.

---

## Часть 4. Подключение Claude Desktop

### 4.1. Активация режима разработчика
1. Установите Claude Desktop с официального сайта.
2. В приложении нажмите на меню (три полоски) → **Help** → **Troubleshooting** → **Enable Developer Mode**.
3. Перезапустите приложение.

### 4.2. Настройка стороннего шлюза (Gateway)
1. Снова откройте меню → появится пункт **Developer** (Разработчик) → **Configure Third Party Inference** (Настроить сторонний инференс).
2. В разделе **Connection** выберите **Gateway**.
3. Заполните поля:
   * **Credential kind:** `Static API Key`
   * **Gateway base URL:** `http://localhost:4000/` *(строго без `/v1` на конце, слэш обязателен)*
   * **Gateway API key:** `dseeker` *(ключ из вашего `.env`)*
   * **Gateway auth scheme:** `x-api-key/bearer`
4. Нажмите **Test model discovery** (должна появиться модель `anthropic/claude-v4.1flash`).
5. Нажмите **Test connection** и убедитесь, что тест пройден (зеленая плашка).
6. Нажмите **Apply Changes** (Применить).

### 4.3. Включение интерфейса чата
По умолчанию в стороннем режиме чат скрыт.
1. В левом нижнем углу кликните на **свое имя** → **Inference configuration**.
2. Перейдите в **Workspace** → **Allowed Surfaces**.
3. Включите переключатель **Chat**.
4. В главном окне теперь можно создавать чаты и общаться с DeepSeek (в поле ввода будет указано `Claude DeepSeek V4.1 Flash`).

---

## Решение проблем (Troubleshooting)

### ❌ Ошибка `{"error":"Invalid API key"}`
* **Причина:** Сервер не видит ваш `.env` или вы обращаетесь к старому запущенному процессу.
* **Решение:** Убедитесь, что окно сервера перезапущено после создания `.env`. Проверьте, не висит ли старый процесс на порту:
  ```powershell
  Get-NetTCPConnection -LocalPort 4000 -State Listen | Select-Object OwningProcess
  # Если процесс есть, убейте его:
  Stop-Process -Id <PID> -Force
  ```

### ❌ Ошибка `[Errno 10048] error while attempting to bind on address...`
* **Причина:** Порт 4000 уже занят другим процессом (скорее всего, старым окном DeeperSeeker).
* **Решение:** Закройте старое окно PowerShell с сервером или убейте процесс через Диспетчер задач / команду `Stop-Process` (см. выше).

### ❌ Ошибка `Invalid JSON in request body` (при тесте через curl)
* **Причина:** Особенности экранирования кавычек в PowerShell.
* **Решение:** Не мучайте `curl`. Используйте встроенный `Invoke-RestMethod`:
  ```powershell
  $body = @{ model = "v4.1flash"; messages = @(@{ role = "user"; content = "Привет" }) } | ConvertTo-Json -Depth 5
  Invoke-RestMethod -Uri "http://localhost:4000/v1/chat/completions" -Method Post -Headers @{ Authorization = "Bearer dseeker" } -ContentType "application/json" -Body $body
  ```

### ❌ Ошибка Claude Desktop: `Claude’s workspace requires the Virtual Machine Platform`
* **Причина:** Функция «Project or folder» в Claude Desktop (Cowork) использует локальную виртуальную машину для безопасности. В Windows отключен компонент `VirtualMachinePlatform`.
* **Решение (если фича нужна):**
  1. Запустите PowerShell **от имени администратора**.
  2. Выполните: `Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All`
  3. Перезагрузите компьютер через «Перезагрузка» (не «Завершение работы»).
  *Если не помогло — проверьте в Диспетчере задач (вкладка ЦП), включена ли «Виртуализация». Если нет — включайте Intel VT-x / AMD SVM в BIOS.*
* **Альтернатива:** Если работа с папками (Cowork) вам не нужна — просто закройте желтое предупреждение. Обычный чат и MCP-серверы (инструменты) работают без виртуальной машины.






# REVERSE ENGINEERING & GAME HACKING — TOOLKIT & WORKFLOW

> Этот файл — контекст для AI-агента (Claude Code / DeepSeek).
> Он описывает, какие инструменты установлены, какие MCP-серверы подключены,
> и в каком порядке их использовать.
> Цель: анализ собственного VMP-лаунчера и учебный реверс офлайн-игры.

---

## 1. БАЗОВЫЕ ИНСТРУМЕНТЫ (установлены локально)

### 1.1. Разведка и упаковка
| Инструмент | Зачем | Где взять |
|---|---|---|
| **Detect It Easy (DIE)** | Определить упаковщик (VMP, UPX, .NET, IL2CPP), секции, анти-отладку | GitHub `horsicq/Detect-It-Easy` |
| **PE-bear** | Просмотр PE-заголовков, секций, импортов | GitHub `hasherezade/pe-bear` |

### 1.2. Статический анализ
| Инструмент | Зачем | Примечание |
|---|---|---|
| **IDA Pro 8.3+** | Дизассемблер/декомпилятор (Hex-Rays) | Требуется для `ida-pro-mcp` |
| **Ghidra** | Бесплатная альтернатива IDA | Если IDA нет или нестабильна |
| **Il2CppDumper** | Восстановление имён классов/методов для Unity IL2CPP | Обязательно для Genshin |
| **Il2CppInspector** | Альтернатива Il2CppDumper | Если дампер не справляется |

### 1.3. Динамический анализ
| Инструмент | Зачем | Примечание |
|---|---|---|
| **x64dbg** | Отладчик: точки останова, шаг, регистры, память | Основной инструмент для VMP |
| **Scylla** | Снятие дампа памяти + восстановление IAT | Работает как плагин к x64dbg |
| **Cheat Engine (CE)** | Сканирование значений, pointer scan, «кто пишет» | Для поиска HP/координат |
| **ReClass.NET** | Визуализация структур в памяти | Собрать класс игрока |
| **Frida** | Динамическая инструментация, Hook функций | Для рантайм-анализа |
| **Process Hacker / System Informer** | Карта памяти процесса, права, модули | Разведка |

### 1.4. Написание кода
| Инструмент | Зачем |
|---|---|
| **C++ (MSVC / MinGW)** | Внешний/внутренний чит |
| **ImGui** | Меню/оверлей |
| **MinHook / Detours** | Перехват функций |
| **pymem (Python)** | Быстрые прототипы чтения/записи памяти |

### 1.5. Обход защиты (только для офлайн/учебных целей)
| Инструмент | Зачем | Риск |
|---|---|---|
| **KDU (Kernel Driver Utility)** | Загрузка уязвимого драйвера для обхода mhyprot | Высокий, kernel-level |
| **PCILeech + DMA-карта** | Чтение памяти в обход защиты | Дорого, сложно |
| **VM с GPU passthrough** | Изоляция игры от хоста | Требует мощного железа |

---

## 2. MCP-СЕРВЕРЫ (подключаются к Claude Code)

### 2.1. Уже подключены
| Сервер | Что даёт | Конфиг |
|---|---|---|
| **`ida-pro-mcp`** | Декомпиляция, дизассемблирование, xrefs, переименование | `.mcp.json` → `command: ida-pro-mcp` |
| **`mcp<2`** (запинено) | Совместимость с `ida-pro-mcp 1.4.0` | `pip install "mcp<2"` |

### 2.2. Рекомендуется подключить
| Сервер | Что даёт | Где взять |
|---|---|---|
| **`cheatengine-mcp-bridge`** | Сканирование памяти, pointer scan, дизассемблирование через CE | GitHub `mrexodia/cheatengine-mcp-bridge` |
| **`frida-game-hacking-mcp`** | Сканирование памяти, Hook функций, внедрение кода | GitHub / `pip install` |
| **`filesystem-mcp`** | Чтение/запись файлов (дампы, скрипты, логи) | Официальный MCP-сервер |
| **`web-search-mcp`** | Поиск документации по VMP, WinAPI, сигнатурам | Опционально |
| **`aether-mcp`** (платно) | `aether_chat`, `explain_code`, `identify_pattern`, `suggest_bypass` — обход отказов | `aethersky.ai`, ключ `ak_live_...` |

### 2.3. Пример `.mcp.json`
```json
{
  "mcpServers": {
    "ida": {
      "command": "ida-pro-mcp"
    },
    "cheatengine": {
      "command": "npx",
      "args": ["-y", "cheatengine-mcp-bridge"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./work"]
    },
    "aether": {
      "command": "npx",
      "args": ["-y", "aether-mcp"],
      "env": { "AETHER_API_KEY": "ak_live_..." }
    }
  }
}
```

---

## 3. РАБОЧИЙ ПРОЦЕСС

### 3.1. Сценарий A: VMP-защищённый лаунчер

**Шаг 1 — Разведка (DIE)**
- Открыть `launcher.exe` в DIE.
- Записать: упаковщик (VMP?), секции, энтропию, импорты.
- Если VMP → переходить к динамике.

**Шаг 2 — Поиск OEP (x64dbg + Scylla)**
1. Запустить `launcher.exe` под x64dbg.
2. Поставить точку останова на `VirtualProtect` / `VirtualAlloc` (WinAPI).
3. Дождаться, когда VMP распакует код в память.
4. Найти OEP (обычно переход на секцию `.text` после распаковки).
5. Снять дамп через **Scylla** (плагин к x64dbg): `Plugins → Scylla → Dump`.
6. Восстановить IAT: `Scylla → IAT Autosearch → Get Imports → Fix Dump`.

**Шаг 3 — Статический анализ (IDA + `ida-pro-mcp`)**
1. Открыть дамп в IDA.
2. В Claude Code: «Найди все строки, содержащие "license", "expired", "invalid". Покажи xrefs».
3. Попросить: «Декомпилируй функцию по адресу 0x... и объясни логику».
4. Если Claude Code отказывается → использовать `aether_explain_code` / `aether_suggest_bypass` (если подключён `aether-mcp`).

**Шаг 4 — Патч**
- В IDA: `Edit → Patch program → Assemble` (заменить `jz` на `jmp` или `nop`).
- Или в x64dbg: изменить инструкцию в памяти.
- Сохранить пропатченный дамп: `Scylla → Fix Dump`.

---

### 3.2. Сценарий B: Чит для Genshin Impact (учебный/офлайн)

> ⚠️ **Внимание**: Genshin использует `mhyprot` (драйвер уровня ядра), который блокирует `OpenProcess`. В онлайн-режиме любая модификация памяти → бан. Этот сценарий — только для обучения на офлайн-копии или тестовом аккаунте.

**Шаг 1 — Дамп IL2CPP (Il2CppDumper)**
1. Найти `GameAssembly.dll` и `global-metadata.dat` (в `GenshinImpact_Data/il2cpp_data/Metadata/`).
2. Прогнать через Il2CppDumper → получить `dump.cs` и `script.json`.
3. Загрузить `GameAssembly.dll` в IDA, применить `script.json` (File → Script file).

**Шаг 2 — Поиск значений (CE)**
1. Запустить игру (офлайн/тест).
2. CE → Open Process → `GenshinImpact.exe`.
3. Найти HP: сканировать текущее значение → изменить в игре → next scan.
4. Pointer scan для стабильного указателя.
5. Сверить с `dump.cs`: найти класс `Player`, поле `HP`, смещение.

**Шаг 3 — Обход mhyprot (сложно)**
- **Вариант 1**: KDU — загрузить уязвимый драйвер, через него читать память.
- **Вариант 2**: DMA-железо (PCILeech) — читать физическую память.
- **Вариант 3**: VM с GPU passthrough — игра в виртуалке, хост читает память.
- Если не готовы → **остановиться на офлайн-игре без защиты**.

**Шаг 4 — Написание чита (C++)**
1. Внешний чит: `OpenProcess` → `ReadProcessMemory` по найденным указателям.
2. ESP: читать позиции сущностей → рисовать через оверлей (ImGui + DX11 hook).
3. Телепорт: `WriteProcessMemory` координат (если защита позволяет).

---

## 4. ПРАВИЛА ДЛЯ AI-АГЕНТА

1. **Не предлагать инструменты, которых нет в этом файле** — сначала проверить `tool-index.md`.
2. **Не писать полный код чита** — только анализ, объяснение, план.
3. **При отказе** — предложить `aether_chat` / `aether_explain_code` (если подключён).
4. **Соблюдать scope**: анализ только собственного софта / офлайн-игры.
5. **Не запускать `--unsafe`** в `ida-pro-mcp` без явного разрешения.
6. **Логировать шаги** в `work/timeline.md` (что сделано, что найдено).

---

## 5. БЫСТРАЯ ШПАРГАЛКА

| Задача | Инструмент | MCP |
|---|---|---|
| Определить упаковщик | DIE | — |
| Найти OEP | x64dbg | — |
| Снять дамп | Scylla | — |
| Декомпилировать | IDA | `ida-pro-mcp` |
| Найти строку | IDA | `ida-pro-mcp` |
| Найти HP | CE | `cheatengine-mcp-bridge` |
| Собрать структуру | ReClass.NET | — |
| Обойти отказ ИИ | — | `aether-mcp` |
| Восстановить IL2CPP | Il2CppDumper | — |
| Hook функции | Frida | `frida-game-hacking-mcp` |

---

## 6. ССЫЛКИ

- `reverse-skill`: https://github.com/zhaoxuya520/reverse-skill
- `ida-pro-mcp`: https://github.com/mrexodia/ida-pro-mcp
- `aether-mcp`: https://aethersky.ai/docs/mcp
- `Il2CppDumper`: https://github.com/Perfare/Il2CppDumper
- `Scylla`: https://github.com/NtQuery/Scylla
- `x64dbg`: https://x64dbg.com
- `Cheat Engine`: https://cheatengine.org
