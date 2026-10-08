# CIB seven 2.3.0 Webclient Plugin Demo

A ready-to-build CIB seven Webclient frontend plugin using Vue 3 + Vite and packaged as a JAR.

The plugin adds **My Plugin** to the CIB seven process-instance tabs and shows the current process-instance context. The button calls:

GET /engine-rest/process-instance/{processInstanceId}

The Axios instance is provided by `@cibseven/plugin-runtime`, so Webclient authentication/session handling is reused.

## Architecture

```text
Browser
  |
  | http://localhost:8080/webapp
  v
CIB seven 2.3.0
  |
  +-- Webclient
  |     |
  |     +-- My Plugin (JAR)
  |           |
  |           +-- Vue 3 component
  |           +-- @cibseven/plugin-runtime (provided by Webclient)
  |           +-- configured Axios
  |
  +-- Engine REST
```

## Important

Do NOT run:

```bash
npm install @cibseven/plugin-runtime
```

CIB seven supplies this runtime. The plugin build externalizes:

- vue
- axios
- bootstrap
- @cibseven/plugin-runtime

This prevents duplicate Vue/runtime instances.

## Prerequisites

For local frontend build:

- Node.js 22
- npm
- JDK 21+ if packaging the JAR locally
- Docker Desktop / Docker Engine
- Docker Compose

## Option A - One-command Docker build

This is the easiest route because Node, npm and JDK are installed inside the Docker build stages.

```bash
docker compose build --no-cache
docker compose up -d
```

Then check:

```bash
docker ps
docker logs cibseven-plugin-demo
```

Open:

```text
http://localhost:8080/webapp
```

Log in, open a process instance and look for:

```text
My Plugin
```

## Option B - Local Vue build + local JAR

Install dependencies:

```bash
npm install
```

Build:

```bash
npm run build
```

Create JAR:

```bash
npm run package:jar
```

Expected output:

```text
dist/
└── META-INF/
    └── cibseven-plugins/
        └── my-plugin/
            ├── plugin.json
            ├── index.js
            ├── styles.css
            └── translations_en.json
```

Then:

```text
my-plugin.jar
└── META-INF/
    └── cibseven-plugins/
        └── my-plugin/
            ├── plugin.json
            ├── index.js
            ├── styles.css
            └── translations_en.json
```

## NPM commands

```bash
npm run dev
npm run build
npm run package:jar
npm run clean
npm run docker:build
npm run docker:up
npm run docker:down
```

## Plugin registration

The plugin exports:

```javascript
export function register({ id, registerPlugin }) {
  registerPlugin('process-instance-tab', MyPlugin, {
    id: 'my-plugin-tab',
    text: `plugins.${id}.title`
  })
}
```

The component receives:

```javascript
instance
process
tenantId
```

and uses:

```javascript
props.instance.id
```

for the current process-instance ID.

## Plugin enablement

`configuration/default.yml` contains:

```yaml
cibseven:
  webclient:
    plugins:
      enabled: true
```

A restart is required after installing/changing the plugin JAR or manifest.

## Troubleshooting

### Plugin is not visible

Check:

```bash
docker logs cibseven-plugin-demo
```

Look for frontend-plugin discovery messages.

Then verify the JAR:

```bash
docker exec -it cibseven-plugin-demo sh
ls -l /configuration/userlib/my-plugin.jar
```

Inspect:

```bash
jar tf /configuration/userlib/my-plugin.jar
```

Expected:

```text
META-INF/cibseven-plugins/my-plugin/plugin.json
META-INF/cibseven-plugins/my-plugin/index.js
META-INF/cibseven-plugins/my-plugin/styles.css
META-INF/cibseven-plugins/my-plugin/translations_en.json
```

### Check plugin configuration

```bash
docker exec -it cibseven-plugin-demo cat /configuration/default.yml
```

It must contain:

```yaml
cibseven:
  webclient:
    plugins:
      enabled: true
```

### 401 / 403

The plugin uses the Webclient-provided Axios instance. A 401/403 normally means the logged-in user/session does not have the required access to the Engine REST endpoint.

### 404

Check the browser Network tab and confirm:

```text
/engine-rest/process-instance/{id}
```

Also verify that the process instance still exists.

### @cibseven/plugin-runtime not found

Do not install it from npm. It is provided by the CIB seven Webclient and is externalized from the bundle.

### Duplicate Vue/runtime

Verify `vite.config.js` contains:

```javascript
external: [
  'vue',
  'axios',
  'bootstrap',
  '@cibseven/plugin-runtime'
]
```

### Wrong plugin API

The manifest uses:

```json
"apiVersion": ["2.3"]
```

The plugin architecture was introduced for the CIB seven 2.3 webclient line.

## Files

```text
cibseven-plugin-2.3.0/
├── src/
│   ├── index.js
│   ├── MyPlugin.vue
│   ├── components/
│   └── services/
│       └── cibseven-api.js
├── public/
│   ├── plugin.json
│   ├── styles.css
│   └── translations_en.json
├── configuration/
│   └── default.yml
├── scripts/
│   ├── clean.mjs
│   └── package-jar.mjs
├── Dockerfile
├── docker-compose.yml
├── vite.config.js
├── package.json
├── .dockerignore
├── .gitignore
└── README.md
```

## Version note

The Docker build is intentionally pinned to:

```text
cibseven/cibseven:2.3.0
```

If your registry uses a different published tag, change only:

```yaml
CIBSEVEN_VERSION: "2.3.0"
```

in `docker-compose.yml`.

Do not mix a 2.2 plugin architecture with the 2.3 Webclient plugin API.
"# cibseven-plugin-2.3.0" 
