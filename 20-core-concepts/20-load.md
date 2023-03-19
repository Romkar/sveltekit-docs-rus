# Загрузка данных
---

Прежде чем компонент [`+page.svelte`](/20-core-concepts/10-routing?id=pagesvelte) (и содержащие его компоненты [`+layout.svelte`](/20-core-concepts/10-routing?id=layoutsvelte)) будет отрисован, нам часто необходимо получить некоторые данные. Это делается путем определения функций `load`.

## Данные страницы

A `+page.svelte` file can have a sibling `+page.js` (or `+page.ts`) that exports a `load` function, the return value of which is available to the page via the `data` prop:

Файл `+page.svelte` может иметь дочерний `+page.js` (или `+page.ts`), экспортирующий функцию `load`, возвращаемое значение которой доступно странице через параметр `data`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export function load({ params }) {
	return {
		post: {
			title: `Заголовок для ${params.slug} находится здесь`,
			content: `Содержание для ${params.slug} находится здесь`
		}
	};
}
```

#### **TypeScript**
**```src/routes/blog/[slug]/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (({ params }) => {
  return {
    post: {
      title: `Заголовок для ${params.slug} находится здесь`,
	  content: `Содержание для ${params.slug} находится здесь`
    }
  };
}) satisfies PageLoad;
```
<!-- tabs:end -->


<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>
```

#### **TypeScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
</script>

<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>
```
<!-- tabs:end -->

Благодаря сгенерированному модулю `$types` мы получаем полную безопасность типов.

Функция `load` в файле `+page.js` выполняется как на сервере, так и в браузере. Если ваша функция `load` должна _всегда_ работать на сервере (например, потому что она использует частные переменные окружения или обращается к базе данных), то она должна быть помещена в файл `+page.server.js`.

Более реалистичная версия функции `load` вашего блога, которая работает только на сервере и получает данные из базы данных, может выглядеть следующим образом:

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getPost(slug: string): Promise<{ title: string, content: string }>
}
```
**```src/routes/blog/[slug]/+page.server.js```**
```js
import * as db from '$lib/server/database';
 
/** @type {import('./$types').PageServerLoad} */
export async function load({ params }) {
  return {
    post: await db.getPost(params.slug)
  };
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getPost(slug: string): Promise<{ title: string, content: string }>
}
```
**```src/routes/blog/[slug]/+page.server.ts```**
```ts
import * as db from '$lib/server/database';
import type { PageServerLoad } from './$types';
 
export const load = (async ({ params }) => {
  return {
    post: await db.getPost(params.slug)
  };
}) satisfies PageServerLoad;
```
<!-- tabs:end -->

Обратите внимание, что тип изменился с `PageLoad` на `PageServerLoad`, поскольку серверные функции `load` могут обращаться к дополнительным аргументам. Чтобы понять, когда использовать `+page.js`, а когда `+page.server.js`, смотрите [универсальный против серверного](/20-core-concepts/20-load?id=Универсальный-против-серверного).

## Данные макета

Ваши файлы `+layout.svelte` также могут загружать данные через `+layout.js` или `+layout.server.js`.

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getPostSummaries(): Promise<Array<{ title: string, slug: string }>>
}
```
**```src/routes/blog/[slug]/+layout.server.js```**
```js
import * as db from '$lib/server/database';

/** @type {import('./$types').LayoutServerLoad} */
export async function load() {
	return {
		posts: await db.getPostSummaries()
	};
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getPostSummaries(): Promise<Array<{ title: string, slug: string }>>
}
```
**```src/routes/blog/[slug]/+layout.server.ts```**
```ts
import * as db from '$lib/server/database';
import type { LayoutServerLoad } from './$types';
 
export const load = (async () => {
  return {
    posts: await db.getPostSummaries()
  };
}) satisfies LayoutServerLoad;
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+layout.svelte```**
```svelte
<script>
	/** @type {import('./$types').LayoutData} */
	export let data;
</script>

<main>
	<!-- +page.svelte отображается здесь <slot> -->
	<slot />
</main>

<aside>
	<h2>Больше постов</h2>
	<ul>
		{#each data.posts as post}
			<li>
				<a href="/blog/{post.slug}">
					{post.title}
				</a>
			</li>
		{/each}
	</ul>
</aside>
```
#### **TypeScript**
**```src/routes/blog/[slug]/+layout.svelte```**
```svelte
<script lang="ts">
  import type { LayoutData } from './$types';

  export let data: LayoutData;
</script>

<main>
	<!-- +page.svelte отображается здесь <slot> -->
	<slot />
</main>

<aside>
	<h2>Больше постов</h2>
	<ul>
		{#each data.posts as post}
			<li>
				<a href="/blog/{post.slug}">
					{post.title}
				</a>
			</li>
		{/each}
	</ul>
</aside>
```
<!-- tabs:end -->

Данные, возвращаемые функциями `load` макета, доступны дочерним компонентам `+layout.svelte` и компоненту `+page.svelte`, а также макету, к которому он "принадлежит".

**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script>
+	import { page } from '$app/stores';

	/** @type {import('./$types').PageData} */
	export let data;

+	// мы можем получить доступ к `data.posts` потому что он возвращается из
+	// функции родительского макета `load`
+	$: index = data.posts.findIndex(post => post.slug === $page.params.slug);
+	$: next = data.posts[index - 1];
</script>

<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>

+{#if next}
+	<p>Следующая публикация: <a href="/blog/{next.slug}">{next.title}</a></p>
+{/if}
```

> Если несколько функций `load` возвращают данные с одним и тем же ключом, то "побеждает" последняя - результат того, что функция макета `load` возвращает `{ a: 1, b: 2 }` и страницы `load`, возвращающей `{ b: 3, c: 4 }` будет `{ a: 1, b: 3, c: 4 }`.

## $page.data

Компонент `+page.svelte` и каждый компонент `+layout.svelte` над ним имеют доступ к своим собственным данным плюс ко всем данным своих родителей.

В некоторых случаях нам может понадобиться обратное - родительскому макету может понадобиться доступ к данным страницы или дочернего макета. Например, корневой макет может захотеть получить доступ к свойству `title`, возвращаемому из функции `load` в `+page.js` или `+page.server.js`. Это можно сделать с помощью `$page.data`:

```svelte
/// file: src/routes/+layout.svelte
<script>
	import { page } from '$app/stores';
</script>

<svelte:head>
	<title>{$page.data.title}</title>
</svelte:head>
```

Информация о типе для `$page.data` предоставляется [`App.PageData`](/50-reference/40-types?id=pagedata).

## Универсальный против серверного

Как мы уже видели, существует два типа функции `load`:

* файлы `+page.js` и `+layout.js` экспортируют _универсальные_ `load` функции, которые выполняются как на сервере, так и в браузере
* файлы `+page.server.js` и `+layout.server.js` экспортируют _серверные_ функции `load`, которые работают только на стороне сервера.

Концептуально это одно и то же, но есть несколько важных различий, о которых следует знать.

### Когда какая функция загрузки выполняется?

Серверная функция `load` _всегда_ выполняются на сервере.

По умолчанию универсальные функции `load` запускаются на сервере во время SSR, когда пользователь впервые посещает вашу страницу. Затем они будут снова запущены во время гидратации, повторно используя любые ответы от [fetch-запросов](/20-core-concepts/20-load?id=Выполнение-fetch-запросов). Все последующие вызовы универсальных функций `load` происходят в браузере. Вы можете настроить поведение через [параметры страницы](/20-core-concepts/40-page-options). Если вы отключите [рендеринг на стороне сервера (SSR)](/20-core-concepts/40-page-options?id=ssr), вы получите SPA, а универсальные функции `load` будут _всегда_ выполняться на клиенте.

Функция `load` вызывается во время выполнения, если только вы не [пререндерите](/20-core-concepts/40-page-options?id=пререндер) страницу - в этом случае она вызывается во время сборки.

### Input

Both universal and server `load` functions have access to properties describing the request (`params`, `route` and `url`) and various functions (`fetch`, `setHeaders`, `parent` and `depends`). These are described in the following sections.

Server `load` functions are called with a `ServerLoadEvent`, which inherits `clientAddress`, `cookies`, `locals`, `platform` and `request` from `RequestEvent`.

Universal `load` functions are called with a `LoadEvent`, which has a `data` property. If you have `load` functions in both `+page.js` and `+page.server.js` (or `+layout.js` and `+layout.server.js`), the return value of the server `load` function is the `data` property of the universal `load` function's argument.

### Output

A universal `load` function can return an object containing any values, including things like custom classes and component constructors.

A server `load` function must return data that can be serialized with [devalue](https://github.com/rich-harris/devalue) — anything that can be represented as JSON plus things like `BigInt`, `Date`, `Map`, `Set` and `RegExp`, or repeated/cyclical references — so that it can be transported over the network. Your data can include [promises](#streaming-with-promises), in which case it will be streamed to browsers.

### When to use which

Server `load` functions are convenient when you need to access data directly from a database or filesystem, or need to use private environment variables.

Universal `load` functions are useful when you need to `fetch` data from an external API and don't need private credentials, since SvelteKit can get the data directly from the API rather than going via your server. They are also useful when you need to return something that can't be serialized, such as a Svelte component constructor.

In rare cases, you might need to use both together — for example, you might need to return an instance of a custom class that was initialised with data from your server.

## Using URL data

Often the `load` function depends on the URL in one way or another. For this, the `load` function provides you with `url`, `route` and `params`.

### url

An instance of [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL), containing properties like the `origin`, `hostname`, `pathname` and `searchParams` (which contains the parsed query string as a [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) object). `url.hash` cannot be accessed during `load`, since it is unavailable on the server.

> In some environments this is derived from request headers during server-side rendering. If you're using [adapter-node](adapter-node), for example, you may need to configure the adapter in order for the URL to be correct.

### route

Contains the name of the current route directory, relative to `src/routes`:

```js
/// file: src/routes/a/[b]/[...c]/+page.js
/** @type {import('./$types').PageLoad} */
export function load({ route }) {
	console.log(route.id); // '/a/[b]/[...c]'
}
```

### params

`params` is derived from `url.pathname` and `route.id`.

Given a `route.id` of `/a/[b]/[...c]` and a `url.pathname` of `/a/x/y/z`, the `params` object would look like this:

```json
{
	"b": "x",
	"c": "y/z"
}
```

## Выполнение fetch запросов

To get data from an external API or a `+server.js` handler, you can use the provided `fetch` function, which behaves identically to the [native `fetch` web API](https://developer.mozilla.org/en-US/docs/Web/API/fetch) with a few additional features:

- it can be used to make credentialed requests on the server, as it inherits the `cookie` and `authorization` headers for the page request
- it can make relative requests on the server (ordinarily, `fetch` requires a URL with an origin when used in a server context)
- internal requests (e.g. for `+server.js` routes) go direct to the handler function when running on the server, without the overhead of an HTTP call
- during server-side rendering, the response will be captured and inlined into the rendered HTML by hooking into the `text` and `json` methods of the `Response` object. Note that headers will _not_ be serialized, unless explicitly included via [`filterSerializedResponseHeaders`](hooks#server-hooks-handle). Then, during hydration, the response will be read from the HTML, guaranteeing consistency and preventing an additional network request - if you got a warning in your browser console when using the browser `fetch` instead of the `load` `fetch`, this is why.

```js
/// file: src/routes/items/[id]/+page.js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, params }) {
	const res = await fetch(`/api/items/${params.id}`);
	const item = await res.json();

	return { item };
}
```

> Cookies will only be passed through if the target host is the same as the SvelteKit application or a more specific subdomain of it.

## Cookies and headers

A server `load` function can get and set [`cookies`](types#public-types-cookies).

```js
/// file: src/routes/+layout.server.js
// @filename: ambient.d.ts
declare module '$lib/server/database' {
	export function getUser(sessionid: string | undefined): Promise<{ name: string, avatar: string }>
}

// @filename: index.js
// ---cut---
import * as db from '$lib/server/database';

/** @type {import('./$types').LayoutServerLoad} */
export async function load({ cookies }) {
	const sessionid = cookies.get('sessionid');

	return {
		user: await db.getUser(sessionid)
	};
}
```

> When setting cookies, be aware of the `path` property. By default, the `path` of a cookie is the current pathname. If you for example set a cookie at page `admin/user`, the cookie will only be available within the `admin` pages by default. In most cases you likely want to set `path` to `'/'` to make the cookie available throughout your app.

Both server and universal `load` functions have access to a `setHeaders` function that, when running on the server, can set headers for the response. (When running in the browser, `setHeaders` has no effect.) This is useful if you want the page to be cached, for example:

```js
// @errors: 2322 1360
/// file: src/routes/products/+page.js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, setHeaders }) {
	const url = `https://cms.example.com/products.json`;
	const response = await fetch(url);

	// cache the page for the same length of time
	// as the underlying data
	setHeaders({
		age: response.headers.get('age'),
		'cache-control': response.headers.get('cache-control')
	});

	return response.json();
}
```

Setting the same header multiple times (even in separate `load` functions) is an error — you can only set a given header once. You cannot add a `set-cookie` header with `setHeaders` — use `cookies.set(name, value, options)` instead.

## Using parent data

Occasionally it's useful for a `load` function to access data from a parent `load` function, which can be done with `await parent()`:

```js
/// file: src/routes/+layout.js
/** @type {import('./$types').LayoutLoad} */
export function load() {
	return { a: 1 };
}
```

```js
/// file: src/routes/abc/+layout.js
/** @type {import('./$types').LayoutLoad} */
export async function load({ parent }) {
	const { a } = await parent();
	return { b: a + 1 };
}
```

```js
/// file: src/routes/abc/+page.js
/** @type {import('./$types').PageLoad} */
export async function load({ parent }) {
	const { a, b } = await parent();
	return { c: a + b };
}
```

```svelte
/// file: src/routes/abc/+page.svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<!-- renders `1 + 2 = 3` -->
<p>{data.a} + {data.b} = {data.c}</p>
```

> Notice that the `load` function in `+page.js` receives the merged data from both layout `load` functions, not just the immediate parent.

Inside `+page.server.js` and `+layout.server.js`, `parent` returns data from parent `+layout.server.js` files.

In `+page.js` or `+layout.js` it will return data from parent `+layout.js` files. However, a missing `+layout.js` is treated as a `({ data }) => data` function, meaning that it will also return data from parent `+layout.server.js` files that are not 'shadowed' by a `+layout.js` file

Take care not to introduce waterfalls when using `await parent()`. Here, for example, `getData(params)` does not depend on the result of calling `parent()`, so we should call it first to avoid a delayed render.

```diff
/// file: +page.js
/** @type {import('./$types').PageLoad} */
export async function load({ params, parent }) {
-	const parentData = await parent();
	const data = await getData(params);
+	const parentData = await parent();

	return {
		...data
		meta: { ...parentData.meta, ...data.meta }
	};
}
```

## Errors

If an error is thrown during `load`, the nearest [`+error.svelte`](routing#error) will be rendered. For _expected_ errors, use the `error` helper from `@sveltejs/kit` to specify the HTTP status code and an optional message:

```js
/// file: src/routes/admin/+layout.server.js
// @filename: ambient.d.ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
			isAdmin: boolean;
		}
	}
}

// @filename: index.js
// ---cut---
import { error } from '@sveltejs/kit';

/** @type {import('./$types').LayoutServerLoad} */
export function load({ locals }) {
	if (!locals.user) {
		throw error(401, 'not logged in');
	}

	if (!locals.user.isAdmin) {
		throw error(403, 'not an admin');
	}
}
```

If an _unexpected_ error is thrown, SvelteKit will invoke [`handleError`](hooks#shared-hooks-handleerror) and treat it as a 500 Internal Error.

## Redirects

To redirect users, use the `redirect` helper from `@sveltejs/kit` to specify the location to which they should be redirected alongside a `3xx` status code.

```js
/// file: src/routes/user/+layout.server.js
// @filename: ambient.d.ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
		}
	}
}

// @filename: index.js
// ---cut---
import { redirect } from '@sveltejs/kit';

/** @type {import('./$types').LayoutServerLoad} */
export function load({ locals }) {
	if (!locals.user) {
		throw redirect(307, '/login');
	}
}
```

> Make sure you're not catching the thrown redirect, which would prevent SvelteKit from handling it.

In the browser, you can also navigate programmatically outside of a `load` function using [`goto`](modules#$app-navigation-goto) from [`$app.navigation`](modules#$app-navigation).

## Streaming with promises

Promises at the _top level_ of the returned object will be awaited, making it easy to return multiple promises without creating a waterfall. When using a server `load`, _nested_ promises will be streamed to the browser as they resolve. This is useful if you have slow, non-essential data, since you can start rendering the page before all the data is available:

```js
/// file: src/routes/+page.server.js
/** @type {import('./$types').PageServerLoad} */
export function load() {
	return {
		one: Promise.resolve(1),
		two: Promise.resolve(2),
		streamed: {
			three: new Promise((fulfil) => {
				setTimeout(() => {
					fulfil(3)
				}, 1000);
			})
		}
	};
}
```

This is useful for creating skeleton loading states, for example:

```svelte
/// file: src/routes/+page.svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<p>
	one: {data.one}
</p>
<p>
	two: {data.two}
</p>
<p>
	three:
	{#await data.streamed.three}
		Loading...
	{:then value}
		{value}
	{:catch error}
		{error.message}
	{/await}
</p>
```

On platforms that do not support streaming, such as AWS Lambda, responses will be buffered. This means the page will only render once all promises resolve.

> Streaming data will only work when JavaScript is enabled. You should avoid returning nested promises from a universal `load` function if the page is server rendered, as these are _not_ streamed — instead, the promise is recreated when the function re-runs in the browser.

## Parallel loading

When rendering (or navigating to) a page, SvelteKit runs all `load` functions concurrently, avoiding a waterfall of requests. During client-side navigation, the result of calling multiple server `load` functions are grouped into a single response. Once all `load` functions have returned, the page is rendered.

## Rerunning load functions

SvelteKit tracks the dependencies of each `load` function to avoid re-running it unnecessarily during navigation.

For example, given a pair of `load` functions like these...

```js
/// file: src/routes/blog/[slug]/+page.server.js
// @filename: ambient.d.ts
declare module '$lib/server/database' {
	export function getPost(slug: string): Promise<{ title: string, content: string }>
}

// @filename: index.js
// ---cut---
import * as db from '$lib/server/database';

/** @type {import('./$types').PageServerLoad} */
export async function load({ params }) {
	return {
		post: await db.getPost(params.slug)
	};
}
```

```js
/// file: src/routes/blog/[slug]/+layout.server.js
// @filename: ambient.d.ts
declare module '$lib/server/database' {
	export function getPostSummaries(): Promise<Array<{ title: string, slug: string }>>
}

// @filename: index.js
// ---cut---
import * as db from '$lib/server/database';

/** @type {import('./$types').LayoutServerLoad} */
export async function load() {
	return {
		posts: await db.getPostSummaries()
	};
}
```

...the one in `+page.server.js` will re-run if we navigate from `/blog/trying-the-raw-meat-diet` to `/blog/i-regret-my-choices` because `params.slug` has changed. The one in `+layout.server.js` will not, because the data is still valid. In other words, we won't call `db.getPostSummaries()` a second time.

A `load` function that calls `await parent()` will also re-run if a parent `load` function is re-run.

Dependency tracking does not apply _after_ the `load` function has returned — for example, accessing `params.x` inside a nested [promise](#streaming-with-promises) will not cause the function to re-run when `params.x` changes. (Don't worry, you'll get a warning in development if you accidentally do this.) Instead, access the parameter in the main body of your `load` function.

### Manual invalidation

You can also re-run `load` functions that apply to the current page using [`invalidate(url)`](modules#$app-navigation-invalidate), which re-runs all `load` functions that depend on `url`, and [`invalidateAll()`](modules#$app-navigation-invalidateall), which re-runs every `load` function.

A `load` function depends on `url` if it calls `fetch(url)` or `depends(url)`. Note that `url` can be a custom identifier that starts with `[a-z]:`:

```js
/// file: src/routes/random-number/+page.js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, depends }) {
	// load reruns when `invalidate('https://api.example.com/random-number')` is called...
	const response = await fetch('https://api.example.com/random-number');

	// ...or when `invalidate('app:random')` is called
	depends('app:random');

	return {
		number: await response.json()
	};
}
```

```svelte
/// file: src/routes/random-number/+page.svelte
<script>
	import { invalidate, invalidateAll } from '$app/navigation';

	/** @type {import('./$types').PageData} */
	export let data;

	function rerunLoadFunction() {
		// any of these will cause the `load` function to re-run
		invalidate('app:random');
		invalidate('https://api.example.com/random-number');
		invalidate(url => url.href.includes('random-number'));
		invalidateAll();
	}
</script>

<p>random number: {data.number}</p>
<button on:click={rerunLoadFunction}>Update random number</button>
```

To summarize, a `load` function will re-run in the following situations:

- It references a property of `params` whose value has changed
- It references a property of `url` (such as `url.pathname` or `url.search`) whose value has changed
- It calls `await parent()` and a parent `load` function re-ran
- It declared a dependency on a specific URL via [`fetch`](#making-fetch-requests) or [`depends`](types#public-types-loadevent), and that URL was marked invalid with [`invalidate(url)`](modules#$app-navigation-invalidate)
- All active `load` functions were forcibly re-run with [`invalidateAll()`](modules#$app-navigation-invalidateall)

`params` and `url` can change in response to a `<a href="..">` link click, a [`<form>` interaction](form-actions#get-vs-post), a [`goto`](modules#$app-navigation-goto) invocation, or a [`redirect`](modules#sveltejs-kit-redirect).

Note that re-running a `load` function will update the `data` prop inside the corresponding `+layout.svelte` or `+page.svelte`; it does _not_ cause the component to be recreated. As a result, internal state is preserved. If this isn't what you want, you can reset whatever you need to reset inside an [`afterNavigate`](modules#$app-navigation-afternavigate) callback, and/or wrap your component in a [`{#key ...}`](https://svelte.dev/docs#template-syntax-key) block.

## Further reading

- [Tutorial: Loading data](https://learn.svelte.dev/tutorial/page-data)
- [Tutorial: Errors and redirects](https://learn.svelte.dev/tutorial/error-basics)
- [Tutorial: Advanced loading](https://learn.svelte.dev/tutorial/await-parent)
