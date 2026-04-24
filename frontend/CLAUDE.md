# CLAUDE.md — Frontend

React 17 single-page app served by nginx in production and webpack-dev-server in development.

## Start

```bash
yarn install
yarn start        # webpack-dev-server on http://localhost:8080 with hot reload
```

Production build:
```bash
yarn build        # outputs to dist/
```

## Key files

| File | Purpose |
|------|---------|
| `src/` | React source code |
| `webpack.config.js` | Webpack + dev-server config |
| `Dockerfile` | Multi-stage build: webpack → nginx |
| `chart/` | Helm chart for Kubernetes deployment |

## Okteto dev environment

```bash
# From repo root
export OKTETO_SHARED_NAMESPACE="movies-shared"
okteto up -f okteto.frontend.yaml
```

The Okteto manifest syncs the source and runs `yarn start` inside the cluster, so changes hot-reload against the shared backend services.

## Dependencies

No backing services — the frontend talks to the API Gateway (`/api/*`) which is either the shared instance or your personal one depending on the `baggage: okteto-divert=<namespace>` header.

## Tech

- React 17, React Router 5
- Webpack 5 + Babel
- react-hot-loader for HMR
