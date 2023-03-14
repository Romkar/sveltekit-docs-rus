---
title: Node servers
---

To generate a standalone Node server, use [`adapter-node`](https://github.com/sveltejs/kit/tree/master/packages/adapter-node).

## Usage

Install with `npm i -D @sveltejs/adapter-node`, then add the adapter to your `svelte.config.js`:

```js
// @errors: 2307
/// file: svelte.config.js
import adapter from '@sveltejs/adapter-node';

export default {
	kit: {
		adapter: adapter()
	}
};
```

## Deploying

First, build your app with `npm run build`. This will create the production server in the output directory specified in the adapter options, defaulting to `build`.

You will need the output directory, the project's `package.json`, and the production dependencies in `node_modules` to run the application. Production dependencies can be generated with `npm ci --prod` (you can skip this step if your app doesn't have any dependencies). You can then start your app with this command:

```bash
node build
```

Development dependencies will be bundled into your app using [Rollup](https://rollupjs.org). To control whether a given package is bundled or externalised, place it in `devDependencies` or `dependencies` respectively in your `package.json`.

## Environment variables

In `dev` and `preview`, SvelteKit will read environment variables from your `.env` file (or `.env.local`, or `.env.[mode]`, [as determined by Vite](https://vitejs.dev/guide/env-and-mode.html#env-files).)

In production, `.env` files are _not_ automatically loaded. To do so, install `dotenv` in your project...

```bash
npm install dotenv
```

...and invoke it before running the built app:

```diff
-node build
+node -r dotenv/config build
```

### `PORT` и `HOST`

By default, the server will accept connections on `0.0.0.0` using port 3000. These can be customised with the `PORT` and `HOST` environment variables:

```
HOST=127.0.0.1 PORT=4000 node build
```

### `ORIGIN`, `PROTOCOL_HEADER` и `HOST_HEADER`

HTTP doesn't give SvelteKit a reliable way to know the URL that is currently being requested. The simplest way to tell SvelteKit where the app is being served is to set the `ORIGIN` environment variable:

```
ORIGIN=https://my.site node build
```

With this, a request for the `/stuff` pathname will correctly resolve to `https://my.site/stuff`. Alternatively, you can specify headers that tell SvelteKit about the request protocol and host, from which it can construct the origin URL:

```
PROTOCOL_HEADER=x-forwarded-proto HOST_HEADER=x-forwarded-host node build
```

> [`x-forwarded-proto`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto) and [`x-forwarded-host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host) are de facto standard headers that forward the original protocol and host if you're using a reverse proxy (think load balancers and CDNs). You should only set these variables if your server is behind a trusted reverse proxy; otherwise, it'd be possible for clients to spoof these headers.

If `adapter-node` can't correctly determine the URL of your deployment, you may experience this error when using [form actions](form-actions):

> Cross-site POST form submissions are forbidden

### `ADDRESS_HEADER` и `XFF_DEPTH`

The [RequestEvent](types#public-types-requestevent) object passed to hooks and endpoints includes an `event.getClientAddress()` function that returns the client's IP address. By default this is the connecting `remoteAddress`. If your server is behind one or more proxies (such as a load balancer), this value will contain the innermost proxy's IP address rather than the client's, so we need to specify an `ADDRESS_HEADER` to read the address from:

```
ADDRESS_HEADER=True-Client-IP node build
```

> Headers can easily be spoofed. As with `PROTOCOL_HEADER` and `HOST_HEADER`, you should [know what you're doing](https://adam-p.ca/blog/2022/03/x-forwarded-for/) before setting these.

If the `ADDRESS_HEADER` is `X-Forwarded-For`, the header value will contain a comma-separated list of IP addresses. The `XFF_DEPTH` environment variable should specify how many trusted proxies sit in front of your server. E.g. if there are three trusted proxies, proxy 3 will forward the addresses of the original connection and the first two proxies:

```
<client address>, <proxy 1 address>, <proxy 2 address>
```

Some guides will tell you to read the left-most address, but this leaves you [vulnerable to spoofing](https://adam-p.ca/blog/2022/03/x-forwarded-for/):

```
<spoofed address>, <client address>, <proxy 1 address>, <proxy 2 address>
```

We instead read from the _right_, accounting for the number of trusted proxies. In this case, we would use `XFF_DEPTH=3`.

> If you need to read the left-most address instead (and don't care about spoofing) — for example, to offer a geolocation service, where it's more important for the IP address to be _real_ than _trusted_, you can do so by inspecting the `x-forwarded-for` header within your app.

### `BODY_SIZE_LIMIT`

The maximum request body size to accept in bytes including while streaming. Defaults to 512kb. You can disable this option with a value of 0 and implement a custom check in [`handle`](hooks#server-hooks-handle) if you need something more advanced.

## Options

The adapter can be configured with various options:

```js
// @errors: 2307
/// file: svelte.config.js
import adapter from '@sveltejs/adapter-node';

export default {
	kit: {
		adapter: adapter({
			// default options are shown
			out: 'build',
			precompress: false,
			envPrefix: '',
			polyfill: true
		})
	}
};
```

### out

The directory to build the server to. It defaults to `build` — i.e. `node build` would start the server locally after it has been created.

### precompress

Enables precompressing using gzip and brotli for assets and prerendered pages. It defaults to `false`.

### envPrefix

If you need to change the name of the environment variables used to configure the deployment (for example, to deconflict with environment variables you don't control), you can specify a prefix:

```js
envPrefix: 'MY_CUSTOM_';
```

```sh
MY_CUSTOM_HOST=127.0.0.1 \
MY_CUSTOM_PORT=4000 \
MY_CUSTOM_ORIGIN=https://my.site \
node build
```

### polyfill

Controls whether your build will load polyfills for missing modules. It defaults to `true`, and should only be disabled when using Node 18.11 or greater.

## Custom server

The adapter creates two files in your build directory — `index.js` and `handler.js`. Running `index.js` — e.g. `node build`, if you use the default build directory — will start a server on the configured port.

Alternatively, you can import the `handler.js` file, which exports a handler suitable for use with [Express](https://github.com/expressjs/expressjs.com), [Connect](https://github.com/senchalabs/connect) or [Polka](https://github.com/lukeed/polka) (or even just the built-in [`http.createServer`](https://nodejs.org/dist/latest/docs/api/http.html#httpcreateserveroptions-requestlistener)) and set up your own server:

```js
// @errors: 2307 7006
/// file: my-server.js
import { handler } from './build/handler.js';
import express from 'express';

const app = express();

// add a route that lives separately from the SvelteKit app
app.get('/healthcheck', (req, res) => {
	res.end('ok');
});

// let SvelteKit handle everything else, including serving prerendered pages and static assets
app.use(handler);

app.listen(3000, () => {
	console.log('listening on port 3000');
});
```

## Troubleshooting

### Is there a hook for cleaning up before the server exits?

There's nothing built-in to SvelteKit for this, because such a cleanup hook depends highly on the execution environment you're on. For Node, you can use its built-in `process.on(..)` to implement a callback that runs before the server exits:

```js
// @errors: 2304 2580
function shutdownGracefully() {
	// anything you need to clean up manually goes in here
	db.shutdown();
}

process.on('SIGINT', shutdownGracefully);
process.on('SIGTERM', shutdownGracefully);
```
