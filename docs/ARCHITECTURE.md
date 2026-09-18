# RocketRedis — Documentação do Estado Atual

> GUI desktop para Redis, MVP em desenvolvimento (`README.md:11` `🚧 under development 🚧`).
> Autor: Diego Fernandes. Licença MIT.

## Como rodar

```bash
yarn install
yarn dev # react :4000 + electron
docker-compose up # redis:6.0 --requirepass redis 6379
```

Build/package:

```bash
yarn build
yarn package # electron-builder: portable Win, AppImage Linux, dmg Mac
```

## 1. Stack

| Camada | Versão pinada |
|---|---|
| App | `Electron 11.1.0 + electron-builder 22.8.0`, `electron-store 5.2.0`, `electron-updater 4.3.4` |
| Renderer | `React 16.13.1`, `TypeScript 3.9.7`, `Webpack 4.44`, `Babel 7.10` |
| UI/State | `styled-components 5.1.1`, `recoil 0.0.10`, `Context toast`, `react-virtual 2.2.1`, `react-resizable`, `unform 2.1.3 + yup 0.29` |
| Data | `ioredis 4.17.3`, `i18next 19.6` (só `locales/en/` — 8 json) |
| Qualidade | `ESLint 7.5`, `Jest 26 --passWithNoTests` (0 testes), `husky/commitlint/renovate` |

## 2. Estrutura

```
electron/main.ts — createWindow, createMenu, autoUpdater
src/App.tsx:18 — RecoilRoot > ThemeProvider > AppProvider > Screen
src/screen/index.tsx:11 — Header + ConnectionsList | KeyList + KeyContent
src/atoms/connections.ts:5 — IConnection{name,host,port,password}, IDatabase{name,index,keys}
src/store/connections.ts:4 — electron-store encryptionKey:'' schema connections[]
src/services/RedisConnection.ts:5 — let connection global + initializeConnection/terminate
src/services/connection/ — Create/Update/Delete/Test/LoadConnectionDatabases (INFO keyspace)
src/services/database/LoadKeysFromDatabase.ts:8 — connection.keys('*')
src/services/key/LoadKeyContent.ts:13 — TYPE + GET/HGETALL/LRANGE/SMEMBERS/ZRANGE → JSON.stringify
src/components/ — Button, Modal, Input, Toast, EmptyContent
src/screen/ — Header, ConnectionsList (+Connection, ConnectionFormModal, DeleteConnectionModal), KeyList (+SearchInput), KeyContent
src/hooks/ — useConfig, useWindowSize
src/utils/ — windowBoundsController, getValidationErrors
webpack/ — electron.webpack.js (electron-main dist/main.js), react.webpack.js (electron-renderer dist/renderer/)
i18n.ts, index.html, locales/en/, docker-compose.yml, build/icon.png
```

Total aproximado: `src/` com 52 arquivos `.ts/.tsx`.

## 3. Entradas chave

* Main: `electron/main.ts:21` `createWindow()` — frameless/transparent `min 1000x600`, `nodeIntegration:true + enableRemoteModule:true`, bounds persistidos, `loadURL localhost:4000` em dev ou `dist/renderer/index.html` em prod.
* Menu: `electron/main.ts:62` `createMenu()` — IPC `newConnection` (`CmdOrCtrl+N`), i18n `applicationMenu`.
* Renderer: `src/App.tsx:18` + `src/screen/index.tsx:11` layout 3 colunas.
* Persistência: `src/store/connections.ts` + `src/utils/windowBoundsController`.

## 4. Fluxos

1. **Conexão:** `ConnectionFormModal (unform+yup)` → `store/connections` → `initializeConnection(conn,db)` (`src/services/RedisConnection.ts:7`, `disconnect()` abrupto se já existe, `retryStrategy()->null`, `connectTimeout 3s`) → `LoadConnectionDatabases` via `INFO keyspace`.
2. **Keys:** seleção DB → `KEYS *` (`LoadKeysFromDatabase.ts:8`) → `KeyList` virtualizada + filtro client-side debounce 500ms.
3. **Conteúdo:** seleção key → `TYPE` → fetch total → `JSON.stringify` → `KeyContent` só-leitura.

State híbrido: `Recoil atoms/connections.ts` (`connectionsState, currentConnectionState, currentDatabaseState, currentKeyState`) + `Context toast.tsx` + `electron-store` + `hooks/useConfig/useWindowSize`. Sem backend próprio; renderer acessa Redis direto via `nodeIntegration`.

## 5. Escopo implementado vs. ausente

Faz: CRUD conexões, teste conexão, listar DBs/keys, visualizar string/hash/list/set/zset.

Não faz: criar/editar/deletar key, TTL, console CLI, pub/sub, slowlog/monitor, TLS/Cluster/Sentinel/SSH, import/export, multi-idioma, auto-update testado.

## 6. Riscos / dívidas confirmadas no código

1. `electron/main.ts:35-38` `nodeIntegration:true + enableRemoteModule:true` sem `contextIsolation/preload/sandbox` — risco RCE via XSS. `enableRemoteModule` removido no Electron 14+.
2. `src/services/database/LoadKeysFromDatabase.ts:8` `KEYS *` bloqueante, sem `SCAN`/paginação.
3. `src/services/key/LoadKeyContent.ts:19` `throw Key type not supported yet` (stream etc), `lrange 0 -1` / `smembers` carrega tudo em memória + `JSON.stringify`.
4. `src/store/connections.ts:5` `encryptionKey:''` — senha em texto puro.
5. Chamadas `.then()` sem `.catch` em `KeyList/KeyContent` — erro trava silencioso. `currentKey` não limpa ao trocar DB/key. Filtro `includes` case-sensitive.
6. Deps 2020: upgrade inevitável Electron 11→3x, Webpack 4→5, React 16→18/19, Recoil 0.0.10 experimental → Zustand/Jotai, TS 3.9→5.x. `i18n debug:true`, `saveMissing:true` gera `*.missing.json` em prod.

## 7. Próximos passos sugeridos (não executados — escopo atual é só documentar)

* Curto: trocar `KEYS` por `SCAN`, tratar erros/tipos, criptografar `electron-store`, testes mínimos.
* Médio: CRUD de keys + TTL + console, `preload` + `contextIsolation`, i18n pt-BR.
* Longo: migração Electron/React/Webpack/TS.
