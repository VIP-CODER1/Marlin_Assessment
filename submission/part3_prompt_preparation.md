# Part 3: Prompt Preparation

## 3.1.1 Repository Context

The repository is beets, a Python music library manager for people who care about tagging, sorting, and correcting large collections of audio files. Its core job is to keep music metadata clean and searchable while giving users tools for import, lookup, and library maintenance. The web plugin extends that functionality by exposing a browser-accessible API, so users can inspect and sometimes modify their library remotely.

The intended users are individual music collectors, power users who automate library management, and people who want a local web interface on top of a command-line music organizer. The domain is media metadata management rather than playback. That means the repository cares about library safety, stable query behavior, and predictable handling of items and albums more than about UI-heavy workflows. The web plugin sits at the boundary between internal models and external clients, so changes there have to balance convenience with protection against accidental destructive actions.

## 3.1.2 Pull Request Description

This PR adds a read-only mode to the web plugin and makes that mode the default. Before the change, the web routes that supported DELETE and PATCH could perform library updates whenever those endpoints were called. After the change, the web server only allows read operations unless the configuration explicitly disables readonly protection. The practical difference is that browsing and searching still work as before, but write operations now fail with a method-not-allowed response unless the user opts in.

The change is needed because the web plugin exposes a powerful interface to library data, and the default should favor safety. In a desktop or home-server setup, users may enable the web server for convenience without expecting it to be writable. The old behavior made accidental changes easier. The new behavior reduces that risk while preserving backward compatibility for users who really do want remote editing. The previous behavior was permissive by default; the new behavior is restrictive by default but configurable.

## 3.1.3 Acceptance Criteria

✓ When `readonly` is left at its default value, DELETE requests to item and album endpoints should return HTTP 405.

✓ When `readonly` is left at its default value, PATCH requests to writable item endpoints should return HTTP 405.

✓ When `readonly` is set to `no`, DELETE requests should continue to remove the targeted item or album successfully.

✓ When `readonly` is set to `no`, PATCH requests should continue to update item metadata successfully.

✓ GET requests for items, albums, and query endpoints should continue to work regardless of the `readonly` setting.

✓ The web plugin configuration documentation should clearly describe the new option and its default value.

## 3.1.4 Edge Cases

✓ Endpoints that accept multiple IDs or query strings should still reject destructive requests when `readonly` is enabled.

✓ A configuration file that omits `readonly` entirely should still behave as read-only, not as writable.

✓ Error handling should remain stable for unsupported operations, such as DELETE against list-all endpoints.

## 3.1.5 Initial Prompt

Implement the web plugin read-only behavior in beets so that destructive web API calls are disabled by default and only enabled when the user explicitly turns them off in configuration.

Use the existing web plugin architecture and keep the change local to the plugin, its documentation, and its tests. The expected behavior is simple: GET requests should continue to work as they do now, but DELETE and PATCH requests must return HTTP 405 whenever `readonly` is enabled or omitted. When `readonly` is set to `no`, the existing mutation behavior should remain intact. Keep the configuration default safe. Make sure the web app receives the configuration value during startup, and verify that both item and album routes obey the flag.

Update the web plugin docs to explain the new option, its default, and the fact that write endpoints are blocked unless the user opts in. Add a changelog note that explains the user-visible change. Add or update tests so they cover both read-only and writable configurations, including item-by-id, item-by-query, album-by-id, and album-by-query cases where applicable.

Pay attention to the following edge cases: list-all DELETE behavior should remain invalid if it already is, query-based endpoints should not bypass the read-only check, and the default configuration path should be safe even if the user never sets the new option. Keep the implementation simple and consistent with the plugin’s existing response handling.

Relevant source areas to review while implementing:

- [beetsplug/web/__init__.py](https://github.com/beetbox/beets/blob/master/beetsplug/web/__init__.py#L116-L131)
- [beetsplug/web/__init__.py](https://github.com/beetbox/beets/blob/master/beetsplug/web/__init__.py#L162-L187)
- [docs/plugins/web.rst](https://github.com/beetbox/beets/blob/master/docs/plugins/web.rst#L66-L73)
- [test/test_web.py](https://github.com/beetbox/beets/blob/master/test/test_web.py#L313-L670)

## Integrity Declaration

I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.
