# Адаптеры
---

Прежде чем развернуть приложение SvelteKit, необходимо _адаптировать_ его для цели развёртывания. Адаптеры - это небольшие плагины, которые принимают собранное приложение в качестве входных данных и генерируют выходные данные для развёртывания.

Официальные адаптеры существуют для различных платформ - они документированы на следующих страницах:

- [`@sveltejs/adapter-cloudflare`](/25-build-and-deploy/60-adapter-cloudflare) для Cloudflare Pages
- [`@sveltejs/adapter-cloudflare-workers`](/25-build-and-deploy/70-adapter-cloudflare-workers) для Cloudflare Workers
- [`@sveltejs/adapter-netlify`](/25-build-and-deploy/80-adapter-netlify) для Netlify
- [`@sveltejs/adapter-node`](/25-build-and-deploy/40-adapter-node) для серверов Node
- [`@sveltejs/adapter-static`](/25-build-and-deploy/50-adapter-static) для генерации статических сайтов "Static Site Generation" (SSG)
- [`@sveltejs/adapter-vercel`](/25-build-and-deploy/90-adapter-vercel) для Vercel

Для других платформ существуют дополнительные [адаптеры, предоставляемые сообществом](https://sveltesociety.dev/components#adapters).

## Использование адаптеров

Ваш адаптер указан в `svelte.config.js`:

**```ambient.d.ts```**
```ts
declare module 'svelte-adapter-foo' {
	const adapter: (opts: any) => import('@sveltejs/kit').Adapter;
	export default adapter;
}
```
**```svelte.config.js```**
```js
import adapter from 'svelte-adapter-foo';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	kit: {
		adapter: adapter({
			// параметры адаптера здесь
		})
	}
};

export default config;
```

## Платформо-специфичный контекст

Некоторые адаптеры могут иметь доступ к дополнительной информации о запросе. Например, Cloudflare Workers может получить доступ к объекту `env`, содержащему пространства имен KV и т. д. Его можно передать в `RequestEvent`, используемый в [hooks](/30-advanced/20-hooks) и [server routes](/20-core-concepts/10-routing?id=server) в качестве свойства `platform` - обратитесь к документации каждого адаптера, чтобы узнать больше.

