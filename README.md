# Enfyra documentation sources

Documentation is organized by language code. English is the canonical source in `en/`; a translation belongs at the identical relative path in its language directory, for example `vi/server/README.md` for `en/server/README.md`.

Run `yarn docs:check` from `enfyra-landing-page` to inspect translation coverage. Run `yarn docs:sync` to synchronize the English source and any present translations to the landing API.

Documentation is licensed under the MIT License (see [LICENSE](./LICENSE)). The Enfyra core (`server/` and `kernel/`) is licensed under the [Elastic License 2.0](https://www.elastic.co/licensing/elastic-license); other packages (app, SDKs, MCP server, cloud) are MIT licensed.
