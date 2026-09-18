<p align="center">
  <img src="https://storage.googleapis.com/golden-wind/rocketredis/logo.png" width="200" alt="Rocket Redis logo" />
</p>

<h1 align="center">
  Rocket Redis
</h1>

<p align="center">
  A beautiful desktop GUI for managing Redis databases with ease.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License: MIT" />
  <img src="https://img.shields.io/badge/electron-11-blue" alt="Electron 11" />
  <img src="https://img.shields.io/badge/react-16-61dafb" alt="React 16" />
  <img src="https://img.shields.io/badge/platform-win%20%7C%20mac%20%7C%20linux-lightgrey" alt="Win | Mac | Linux" />
</p>

> 🚧 **Rocket Redis is under development** 🚧 — expect incomplete features and breaking changes.

## Features

- Manage multiple Redis connections (create, edit, test, delete)
- Browse databases and keys with virtualized lists and debounced search
- View values for `string`, `hash`, `list`, `set` and `zset` keys
- Persist connections locally with `electron-store`
- Custom frameless window, auto-updates via `electron-updater`, i18n support

Planned / not yet available: key CRUD, TTL management, CLI console, pub/sub, TLS/Cluster/Sentinel/SSH, key pagination with `SCAN`. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full status, risks and roadmap.

## Layout

UI design: https://www.figma.com/file/YRor24p0TmTxcjl5L86jGb/Rocket-Redis?node-id=1%3A2

![Rocket Redis](.github/layout.png)

## Prerequisites

- [Node.js](https://nodejs.org) (see `.nvmrc` if present, otherwise any recent LTS works for docs; the app pins Electron 11 / Node 12-era toolchain)
- [Yarn](https://yarnpkg.com) 1.x
- Optional: [Docker](https://www.docker.com) to run a local Redis via `docker-compose.yml`

## Quick start

```bash
git clone https://github.com/diego3g/rocketredis.git
cd rocketredis
yarn install
```

Start a local Redis (password `redis` on `6379`):

```bash
docker-compose up
```

Run the app in development mode (React dev server on `:4000` + Electron):

```bash
yarn dev
```

## Scripts

| Script         | What it does                                      |
| -------------- | ------------------------------------------------- |
| `yarn dev`     | React dev server + Electron (recommended for dev) |
| `yarn build`   | Production build (`dist/`)                        |
| `yarn package` | Build + package installers (portable Win, AppImage Linux, dmg Mac) |
| `yarn lint`    | ESLint over `src/` and `electron/`                |
| `yarn tsc`     | TypeScript check (`tsc --noEmit`)                 |
| `yarn test`    | Jest (`--passWithNoTests`, no tests yet)          |

## Tech stack

Electron 11 · React 16 · TypeScript 3.9 · Webpack 4 · styled-components · Recoil · ioredis · Unform + Yup · i18next · electron-store

Details in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Project structure

```
electron/main.ts        # main process: window, menu, auto-updater
src/App.tsx             # renderer entry: RecoilRoot > ThemeProvider > Screen
src/screen/             # ConnectionsList, KeyList, KeyContent, Header
src/services/           # Redis connection, connection CRUD, key/database loaders
src/atoms/ src/store/   # Recoil state + electron-store persistence
src/components/         # Button, Modal, Input, Toast, EmptyContent
webpack/                # electron + react builds
locales/en/             # i18n strings
```

## Contributing

Contributions are welcome. Fork the repository, create a branch, commit with [Conventional Commits](https://www.conventionalcommits.org) (commitlint + husky hooks apply), and open a Pull Request.

## License

[MIT](LICENSE)