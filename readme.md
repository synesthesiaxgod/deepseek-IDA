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
