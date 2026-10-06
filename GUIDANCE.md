# Next.js on Wodby

What this service adds to the Node.js service it is based on.

## Build and start

The service's Dockerfile copies the build context to `/usr/src/app`, runs `npm run build` and starts the container with `npm run start`. `package.json` must define both scripts, normally `next build` and `next start`.

- The Dockerfile does not install dependencies. The pipeline does it before the image is built (`wodby ci run -- npm ci`), and `node_modules` is copied with the code.
- `npm run build` runs while the image is built. Runtime variables of the service, including the link variables and secrets, are not available to it. Pages that need them must be rendered at request time, not prerendered during the build.
- The application must listen on port 3000, which `next start` does by default.

## Variables the service sets

| Variable | Meaning |
| --- | --- |
| `NODE_ENV` | `production` in every environment type, including `dev`. The Node.js service's `development` value for `dev` is replaced, because the container runs a production build. |
| `WORKSPACE_NODE_COMMAND` | `workspace-node next-start`, the start command of a development workspace. |

Do not set `NODE_ENV` to `development` for a deployed environment: `next start` serves the production build.

## Linked services

The service adds no link variables. The database, mail and Redis or Valkey variables of the Node.js service apply unchanged and are read by server-side code. They are not exposed to the browser.

## In a development workspace

- The application is started with `next dev --hostname "$HOST" --port "$PORT"`, called directly from `node_modules`, not through the `dev` script of `package.json`. Options in that script do not apply.
- With Next.js 16 or later `--webpack` is added: the development server uses Webpack, whose file polling works on the checkout's volume, instead of Turbopack.
- `NODE_ENV` is `development` here. A saved file is picked up by the development server without a restart.
- `/.next/` and `/next-env.d.ts` are kept out of Git status, in addition to the Node.js service's paths.
- The image build's `npm run build` does not run in a workspace.
