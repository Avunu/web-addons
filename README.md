# Avunu web addons

An IBM Carbon Design System theme for the Odoo backend, in light and dark, with per-company brand colours. It is an Odoo 18.0 module built and maintained by [Avunu LLC](https://avunu.net) for our own client work and published for anyone to use. The repository extends [OCA/web](https://github.com/OCA/web): the theme builds on that repository's `web_responsive` and `web_dark_mode` rather than duplicating them.

## Modules

| Module | What it does | Licence |
| --- | --- | --- |
| [`web_theme_carbon`](web_theme_carbon/README.md) | IBM Carbon Design System theme for the Odoo backend, light (g10) and dark (g100), with per-company brand colours | AGPL-3.0-or-later |

Each module's README explains why it exists and how it works.

## Branches

`develop` is the default branch and the one pull requests go to. Its code targets Odoo 18.0, the series named in `.odoo-series`. `18.0` follows `develop` automatically (a workflow fast-forwards it after every push) and is the branch to install or pin. When a newer Odoo series is maintained, it gets its own branch (`19.0`, …) and changes flow up to it from `develop`.

## Install

Requires Odoo 18.0 and a PostgreSQL database. Installing a module pulls in the modules it depends on (see below), and those must be on your `addons_path` first.

With a plain Odoo checkout, clone this repository on its `18.0` branch and add it to the addons path:

```sh
git clone --branch 18.0 https://github.com/Avunu/web-addons.git
odoo-bin --addons-path=odoo/addons,/path/to/web-addons,<your other addons paths> \
  -d <database> -i web_theme_carbon --stop-after-init
```

With [odoo-nix](https://github.com/Avunu/odoo-nix), add the repository from the dev shell. `odoo module add` accepts an `owner/repo` and a branch, adds it as a submodule, and the addons path is regenerated from the folders present (run `direnv reload` afterwards):

```sh
odoo module add Avunu/web-addons 18.0
```

OCA dependencies can be added by module name, which also adds the OCA repository that holds them: `odoo module add web_responsive web_dark_mode`.

## Dependencies

Each module's `__manifest__.py` is the source of truth. Besides the Odoo core module `web`, `web_theme_carbon` depends on the following. The OCA repository listed is the 18.0 branch; clone it, along with whatever its modules need in turn (their own manifests list it).

| Module | Needs from OCA or others | Python packages |
| --- | --- | --- |
| `web_theme_carbon` | `web_responsive` and `web_dark_mode` ([OCA/web](https://github.com/OCA/web)) | none |

Both OCA modules exclude `web_enterprise` in their manifests, so Odoo will not install the theme in a database that has Odoo Enterprise's `web_enterprise`.

`web_theme_carbon` also has a build-time npm toolchain for regenerating its Carbon tokens and IBM Plex fonts; it is not needed to run the theme, because the generated files are committed. See its README.

## Development

The repository carries its own development environment, built by [odoo-nix](https://github.com/Avunu/odoo-nix) in addons mode. OCB and the repositories these modules depend on are flake inputs pinned in `flake.lock`, and the Python environment is locked in `nix/`. You need Nix with flakes, and optionally direnv.

```sh
git clone https://github.com/Avunu/web-addons.git && cd web-addons
direnv allow                  # or: nix develop --no-pure-eval
devenv up                     # PostgreSQL, Odoo and Mailpit
odoo db provision             # a database with every module here installed
odoo test web_theme_carbon
```

After `nix flake update`, or when a module gains a `depends` or a Python requirement, run `odoo project relock` and commit `flake.lock` and `nix/`. A new dependency on another repository also needs its input and an `externalSources` entry in `flake.nix`. See odoo-nix's [A repository of addons](https://github.com/Avunu/odoo-nix/blob/main/docs/scaffolding/addons-repository.md).

## Tests

Odoo runs a module's tests when you install or update it with `--test-enable`. Select a module's tests with a `--test-tags` filter and use a throwaway database:

```sh
odoo-bin -c odoo.conf -d <testdb> -i web_theme_carbon \
  --test-enable --test-tags /web_theme_carbon --stop-after-init
```

`web_theme_carbon` has automated tests for its brand colours and its cascade. Notes from its README:

- The cascade tests check for conflicts in the compiled asset bundles, which a static scan of the SCSS cannot decide; the README runs them with `odoo -u web_theme_carbon --test-enable --stop-after-init`.
- `npm run check` in the module directory (after `npm install --ignore-scripts`) checks that the generated assets are current: it regenerates them in memory, fails if the committed output is stale, and runs the drift audit in strict mode.

## Licence

Every module is licensed under the [GNU Affero General Public License v3.0 or later](LICENSE), as its `__manifest__.py` also states (`AGPL-3`, Odoo's identifier for version 3).

`web_theme_carbon` vendors the IBM Plex Sans and IBM Plex Mono fonts (SIL Open Font License 1.1) under `static/fonts/`. They keep their own licence, whose texts sit next to the font files. Its Carbon token files, under `static/lib/carbon/` and `static/src/scss/generated/`, are generated at build time from IBM's Carbon packages (Apache-2.0).

## Provenance

`web_theme_carbon` moved here, with its history, from [Avunu/avunu-odoo-addons](https://github.com/Avunu/avunu-odoo-addons) in November 2026, when Avunu split that repository into one repository per OCA area.
