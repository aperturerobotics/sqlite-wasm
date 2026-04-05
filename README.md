# @aptre/sqlite-wasm

[![NPM Version](https://img.shields.io/npm/v/@aptre/sqlite-wasm)](https://www.npmjs.com/package/@aptre/sqlite-wasm)

Aperture Robotics fork of
[@sqlite.org/sqlite-wasm](https://github.com/sqlite/sqlite-wasm) with a
bare-bones build for minimal binary size.

## Related Projects

- [SQLite Wasm](https://sqlite.org/wasm/doc/trunk/index.md) - Upstream SQLite
  Wasm
- [@sqlite.org/sqlite-wasm](https://github.com/sqlite/sqlite-wasm) - Upstream
  npm package (full-featured build)
- [SQLite](https://github.com/sqlite/sqlite) - Upstream SQLite source

## What Changed

This fork builds SQLite Wasm with the `wasm-bare-bones` flag, which disables
features we do not use to reduce binary size:

**Disabled features:**

- FTS5 (full-text search)
- RTREE (spatial indexing)
- JSON functions
- WAL (write-ahead logging)
- Session extension
- DBPAGE/DBSTAT virtual tables
- Preupdate hooks
- Authorization callbacks
- Incremental blob I/O
- Introspection pragmas
- Progress callbacks
- GET_TABLE convenience function

**Kept features:**

- Core SQL engine
- OPFS VFS (origin private file system)
- OPFS SAH Pool VFS (sync access handle)
- Math functions
- KV VFS
- URI filenames
- API armor (bounds checking)

The build is automatically updated when a new upstream SQLite version is
released. A daily GitHub Actions workflow checks for new `version-*` tags in the
upstream SQLite repository.

## Installation

```bash
npm install @aptre/sqlite-wasm
```

## Usage

Same API as `@sqlite.org/sqlite-wasm`. See the
[upstream documentation](https://github.com/sqlite/sqlite-wasm#usage) for full
usage examples.

```ts
import sqlite3InitModule from '@aptre/sqlite-wasm';

const sqlite3 = await sqlite3InitModule();
const db = new sqlite3.oo1.DB('/mydb.sqlite3', 'ct');
```

### In a worker (with OPFS):

> **Note:** OPFS requires these headers on your server:
>
> `Cross-Origin-Opener-Policy: same-origin`
>
> `Cross-Origin-Embedder-Policy: require-corp`

```ts
import sqlite3InitModule from '@aptre/sqlite-wasm';

const sqlite3 = await sqlite3InitModule();
const db =
  'opfs' in sqlite3
    ? new sqlite3.oo1.OpfsDb('/mydb.sqlite3')
    : new sqlite3.oo1.DB('/mydb.sqlite3', 'ct');
```

## Building locally

1. Build the Docker image:

   ```bash
   docker build -t sqlite-wasm-builder:env .
   ```

2. Run the build (bare-bones by default):

   ```bash
   docker run --rm \
     -e SQLITE_REF="master" \
     -e SQLITE_BARE_BONES=1 \
     -e HOST_UID="$(id -u)" \
     -e HOST_GID="$(id -g)" \
     -v "$(pwd)/out":/out \
     -v "$(pwd)/src/bin":/src/bin \
     sqlite-wasm-builder:env build
   ```

   Set `SQLITE_BARE_BONES=0` for a full-featured build.

## Running tests

```bash
npm install
npx playwright install chromium --with-deps --no-shell
npm test
```

## Deploying a new version

Automated: the `update-sqlite` workflow runs daily. When a new upstream SQLite
`version-*` tag appears, it rebuilds, tests, and publishes automatically.

Manual:

```bash
npm run release          # bumps patch, commits, tags
npm run release:publish  # pushes commit + tag (triggers GH Actions npm publish)
```

## License

Apache 2.0.
