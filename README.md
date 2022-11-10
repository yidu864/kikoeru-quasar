# Kikoeru (kikoeru-quasar)

A self-hosted web media player for listening to your DLsite voice works.

[![unstable build status](https://github.com/umonaca/kikoeru-quasar/actions/workflows/build-and-publish.yml/badge.svg)](https://github.com/umonaca/kikoeru-quasar/actions)

## Install the dependencies

because of node-sass, Nodejs 12 is recommended here

```bash
npm install
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)

```bash
npm run dev
```

### Build the app for production

If you prefer SPA:

```bash
quasar build
```

If you prefer PWA:

```
quasar build -m pwa
```

### Customize the configuration

See [Configuring quasar.conf.js](https://quasar.dev/quasar-cli/quasar-conf-js).
