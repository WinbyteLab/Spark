## Этап 3. Установка и настройка Hermes Agent на Geekom A8 Max (Windows)

### 3.1. Установка WSL2

Откройте **PowerShell от имени администратора** и выполните:
```powershell
wsl --install
```
После перезагрузки установите дистрибутив Ubuntu из Microsoft Store, если он не установился автоматически. WSL2 — официально рекомендуемый путь для Hermes Agent на Windows.

### 3.2. Установка Hermes Agent внутри WSL2

Откройте терминал Ubuntu (WSL2) и выполните:
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

После завершения перезагрузите оболочку:
```bash
source ~/.bashrc   # или source ~/.zshrc
```


### 3.3. Настройка подключения к DGX Spark

Запустите мастер настройки:
```bash
hermes model
```

В мастере:
1. При выборе провайдера укажите **«Custom endpoint»** (Пользовательская конечная точка).
2. **API base URL**: `http://<IP_адрес_DGX_Spark>:8000/v1`
   - **Обязательно** с `/v1` на конце.
   - IP-адрес — тот, который вы узнали на этапе 1.5.
3. **API key**: Для локального vLLM без `--api-key` можно оставить пустым или ввести заглушку (например, `not-needed`).
4. **Model name**: Выберите из списка, который Hermes получит с эндпоинта. Если список пуст — проверьте доступность API (шаг 2.5). Ожидаемый идентификатор: `Qwen/Qwen3.8-27B` (или тот, который вы обслуживаете).

**Альтернатива — неинтерактивная настройка:**
```bash
hermes config set model.provider custom
hermes config set model.base_url http://<IP_адрес_DGX_Spark>:8000/v1
hermes config set model.default Qwen/Qwen3.8-27B
```


### 3.4. Проверка связки

Выполните тестовый запрос:
```bash
hermes -z "Reply exactly MODEL_OK"
```

Если агент отвечает осмысленно — связка работает. Если нет, проверьте по порядку:
- Запущен ли vLLM на DGX Spark (`curl http://localhost:8000/v1/models` на самом Spark).
- Доступен ли порт 8000 с Geekom (`curl http://<IP_DGX_Spark>:8000/v1/models`).
- Не блокирует ли брандмауэр DGX Spark входящие соединения.
- Правильно ли указан URL с `/v1`.

### 3.5. Настройка инструментов (опционально)

Если установка была «Blank Slate» (без инструментов), включите их:
```bash
hermes tools
```
Используйте **пробел** для переключения и **Enter** для подтверждения. Доступны веб-поиск, браузер, терминал, файлы, выполнение кода и другие.

### 3.6. Управление и обслуживание

| Действие | Команда |
|---|---|
| **Переключить модель** | `hermes model` (повторно выбрать Custom endpoint) |
| **Обновить Hermes** | `hermes update` |
| **Просмотреть конфигурацию** | `cat ~/.hermes/config.yaml` |
| **Задать переменные окружения** | Редактировать `~/.hermes/.env` (`OPENAI_BASE_URL`, `OPENAI_API_KEY`) |
| **Удалить с сохранением данных** | `hermes uninstall` |

---

## Ключевые предупреждения (сводка)

1. **Не прерывайте установку обновлений при First-Time Setup** — это может повредить систему.
2. **Подключайте периферию до подачи питания** — DGX Spark стартует автоматически.
3. **Используйте только проверенные рецепты vLLM** — произвольные комбинации параметров могут не работать.
4. **Не забывайте `/v1` в URL** при настройке Hermes — без него API не будет распознан.
5. **`nvidia-smi` на DGX Spark не показывает память** — это не ошибка, а следствие унифицированной архитектуры.
6. **Пароля по умолчанию нет** — учётная запись создаётся вами при первой настройке.

---

## Источники

- NVIDIA DGX Spark First Boot: `docs.nvidia.com/dgx/dgx-spark/first-boot.html`
- vLLM Recipe Qwen3.8-27B DGX Spark: `recipes.vllm.ai/Qwen/Qwen3.8-27B?hardware=dgx_spark_gb10`
- NVIDIA Build: Serve LLMs with vLLM: `build.nvidia.com/station/vllm/choose-recipe`
- Hermes Agent Quickstart: `hermes-agent.nousresearch.com/docs/getting-started/quickstart`
- Hermes Agent LLM Providers: `hermes-agent.nousresearch.com/docs/integrations/providers`
- NVIDIA DGX Spark Playbooks (Hermes Agent): `github.com/NVIDIA/dgx-spark-playbooks`
- DGX Spark / GB10 FAQ (NVIDIA Developer Forums): `forums.developer.nvidia.cn/t/dgx-spark-gb10/27801`
