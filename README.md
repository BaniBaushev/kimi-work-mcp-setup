# Kimi Work: подключение MCP-серверов

Пошаговое руководство по подключению произвольного MCP-сервера к десктопному клиенту **Kimi Work** (Windows), когда стандартные способы не срабатывают.

## Зачем этот репозиторий

У Kimi Work нет документированного способа добавить свой MCP-сервер: привычный `mcp.json` (как у Kimi CLI или Claude Desktop) клиент просто игнорирует. Методом проб и ошибок был найден рабочий путь — через механизм **плагинов**. Этот репозиторий фиксирует рецепт, чтобы не пришлось выводить его заново.

## Ключевые тезисы

- MCP-сервер подключается **только как плагин** с манифестом `kimi.plugin.json` (блок `mcpServers`).
- Манифест нужен **в двух копиях**: `plugin-packages\<id>\` и `plugins\managed\<id>\`.
- Обязательна **запись в реестре** `installed.json` — без неё загрузчик папку игнорирует.
- Плагины поднимаются **только при старте** приложения: после изменений — полный перезапуск.
- Плагина **не будет в списке Plugins** в интерфейсе — UI показывает только официальный каталог из облака. Проверять нужно по логу `main.log` (`local plugins parsed: N/N`) и по появлению инструментов `mcp__plugin-<id>_<server>__<tool>` в сессии.

Полный алгоритм, диагностика и разбор тупиков — в файле [kimi-work-mcp-setup.md](kimi-work-mcp-setup.md).

## Содержимое

| Файл | Описание |
|---|---|
| [kimi-work-mcp-setup.md](kimi-work-mcp-setup.md) | Основное руководство: алгоритм, примеры манифеста и записи реестра, диагностика, тупики |

## Для кого это

- Для тех, кто хочет подключить к Kimi Work **локальный** MCP-сервер (stdio, `command`/`args`) или **удалённый** (по URL).
- Для тех, у кого плагин уже подключён, но «не виден» — и непонятно, сломалось что-то или так задумано.

## Ограничения и дисклеймер

- Проверено на Windows-версии клиента (актуально на момент написания). Поведение может измениться в новых версиях — после обновлений приложения проверяйте, не затёрты ли `plugin-packages` и `installed.json`.
- Способ опирается на внутреннее устройство клиента, а не на публичный API. Используйте на свой страх и риск; перед правкой `installed.json` делайте бэкап.
- Все пути в руководстве обезличены: `%APPDATA%` — это `C:\Users\<USER>\AppData\Roaming`, `<plugin-id>` — идентификатор вашего плагина.

## Contributing

Нашли неточность, новый тупик или изменившееся поведение после обновления клиента? Issues и pull request'ы приветствуются.

## Лицензия

MIT

# Kimi Work: MCP Server Setup

Step-by-step guide for connecting an arbitrary MCP server to the **Kimi Work** desktop client (Windows) when standard methods don't work.

## Why this repository

Kimi Work has no documented way to add a custom MCP server: the familiar `mcp.json` (used by Kimi CLI or Claude Desktop) is simply ignored by the client. Through trial and error, a working path was found — via the **plugin** mechanism. This repository documents the recipe so you don't have to figure it out from scratch.

## Key takeaways

- MCP servers can only be connected **as a plugin** with a `kimi.plugin.json` manifest (the `mcpServers` block).
- The manifest needs to be in **two copies**: `plugin-packages\&lt;id&gt;\` and `plugins\managed\&lt;id&gt;\`.
- A **registry entry** in `installed.json` is mandatory — without it, the loader ignores the folder.
- Plugins are initialized **only at app startup**: after any changes, do a full restart.
- The plugin **won't appear in the Plugins list** in the UI — the interface only shows the official catalog from the cloud. Check via `main.log` (`local plugins parsed: N/N`) and by looking for `mcp__plugin-&lt;id&gt;_&lt;server&gt;__&lt;tool&gt;` tools in your session.

Full algorithm, diagnostics, and dead-end breakdown — in [kimi-work-mcp-setup.md](kimi-work-mcp-setup.md).

## Contents

| File | Description |
|---|---|
| [kimi-work-mcp-setup.md](kimi-work-mcp-setup.md) | Main guide: algorithm, manifest and registry examples, diagnostics, dead ends |

## Who is this for

- Anyone who wants to connect a **local** MCP server (stdio, `command`/`args`) or a **remote** one (by URL) to Kimi Work.
- Anyone who has a plugin connected but it's "not visible" — and it's unclear whether something is broken or it's working as intended.

## Limitations and disclaimer

- Tested on the Windows version of the client (current at the time of writing). Behavior may change in newer versions — after app updates, check whether `plugin-packages` and `installed.json` were overwritten.
- This method relies on the client's internal structure, not a public API. Use at your own risk; back up `installed.json` before editing.
- All paths in the guide are anonymized: `%APPDATA%` stands for `C:\Users\&lt;USER&gt;\AppData\Roaming`, `&lt;plugin-id&gt;` is your plugin's identifier.

## Contributing

Found an inaccuracy, a new dead end, or changed behavior after a client update? Issues and pull requests are welcome.

## License

MIT
