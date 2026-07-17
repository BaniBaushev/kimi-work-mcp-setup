# Как подключить MCP-сервер к Kimi Work

> Рабочий рецепт, добытый настройкой локального MCP-сервера. Главное:
> **mcp.json для этого клиента не работает** — MCP-серверы подключаются
> только как ПЛАГИН.

## Краткий алгоритм

1. **Создать манифест плагина** `kimi.plugin.json`:

```json
{
  "$schema": "https://kimi.com/schemas/kimi.plugin.schema.json",
  "name": "<plugin-id>",
  "version": "0.1.0",
  "description": "...",
  "interface": { "displayName": "...", "category": "DEVELOPER" },
  "mcpServers": {
    "<server-name>": {
      "command": "node",
      "args": ["<ABS_PATH_TO>/server.js"],
      "env": { "VAR": "value" }
    }
  }
}
```

   Удалённый сервер вместо command/args: `"mcpServers": { "<name>": { "url": "https://..." } }`.

2. **Разложить по двум местам** (обе копии обязательны):

| Назначение | Путь |
|---|---|
| Источник | `%APPDATA%\kimi-desktop\daimon-share\daimon\plugin-packages\<plugin-id>\kimi.plugin.json` |
| Установленная копия | `%APPDATA%\kimi-desktop\daimon-share\daimon\runtime\kimi-code\home\plugins\managed\<plugin-id>\kimi.plugin.json` |

3. **Зарегистрировать в реестре**
   `%APPDATA%\kimi-desktop\daimon-share\daimon\runtime\kimi-code\home\plugins\installed.json`
   — добавить объект в массив `plugins`:

```json
{
  "id": "<plugin-id>",
  "root": "...\\plugins\\managed\\<plugin-id>",
  "source": "local-path",
  "enabled": true,
  "installedAt": "<ISO-8601 timestamp>",
  "updatedAt": "<ISO-8601 timestamp>",
  "originalSource": "...\\plugin-packages\\<plugin-id>"
}
```

   Перед правкой — сделать бэкап `installed.json.bak`.

4. **Полностью перезапустить Kimi** (MCP поднимаются только при старте).

5. **Проверка:** в `%APPDATA%\kimi-desktop\logs\main.log` строка
   `local plugins parsed: N/N entries` (N увеличилось на 1, в списке есть
   `<plugin-id>`). Инструменты в сессии видны в `<tools_added>` и вызываются
   после `select_tools` как `mcp__plugin-<plugin-id>_<server-name>__<tool>`.

## Диагностика

- **Сервер проверять отдельно от клиента** — stdio-рукопожатие
  (initialize + tools/list) простым smoke-скриптом. Сначала убеждаемся,
  что сервер жив, потом чиним клиента.
- Плагин не появился → смотреть main.log: нет строки про `<plugin-id>` —
  не увидел папку/реестр; есть ошибка парсинга — формат манифеста.
- Загрузчик читает ТОЛЬКО реестр `installed.json`, а не голую папку в
  plugin-packages (проверено: без записи в реестре плагин игнорируется).
- **Плагина нет на странице Plugins в интерфейсе — это норма.** UI берёт
  список с облачного gateway (`PluginService/ListInstalledPlugins`), где
  только плагины из официального каталога. Локальный рантайм читает
  `installed.json` напрямую (`local plugins parsed: N/N`), поэтому
  инструменты работают, хотя карточки в UI нет. Ориентир для проверки —
  main.log и наличие инструментов в сессии, а не страница Plugins.
- После обновлений приложения проверять, не затёрты ли plugin-packages /
  installed.json. Мастер-копии манифестов хранить в workspace.

## Тупики, в которые заходили (не повторять)

- `mcp.json` в домашней папке пользователя и в `runtime\kimi-code\home\` —
  Kimi Work их игнорирует (это файлы для других клиентов: Kimi CLI /
  Claude Desktop).
- Кавычки внутри строки пути в args (`"\"<path>\""`) — node ищет файл
  с кавычками в имени и молча падает.

## Пример: локальный stdio-сервер на Node.js

- Сервер: `node <ABS_PATH_TO>/dist/cli.js`
- env: переменные подключения к хост-приложению, например
  `HOST=127.0.0.1`, `PORT=<port>` (если сервер — мост к внешней программе,
  она должна быть запущена и слушать указанный порт).
- Для вызовов инструментов хост-приложение должно быть открыто.
