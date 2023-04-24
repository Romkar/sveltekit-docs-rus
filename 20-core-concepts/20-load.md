# Загрузка данных
---

Прежде чем компонент [`+page.svelte`](/20-core-concepts/10-routing?id=pagesvelte) (и содержащие его компоненты [`+layout.svelte`](/20-core-concepts/10-routing?id=layoutsvelte)) будет отрисован, нам часто необходимо получить некоторые данные. Это делается путем определения функций `load`.

## Данные страницы

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
```diff
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

**```src/routes/+layout.svelte```**
```svelte
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

* файлы `+page.js` и `+layout.js` экспортируют _универсальные_ функции `load`, которые выполняются как на сервере, так и в браузере
* файлы `+page.server.js` и `+layout.server.js` экспортируют _серверные_ функции `load`, которые работают только на стороне сервера.

Концептуально это одно и то же, но есть несколько важных различий, о которых следует знать.

### Когда какая функция загрузки выполняется?

Серверная функция `load` _всегда_ выполняется на сервере.

По умолчанию универсальные функции `load` запускаются на сервере во время SSR, когда пользователь впервые посещает вашу страницу. Затем они будут снова запущены во время гидратации, повторно используя любые ответы от [fetch-запросов](/20-core-concepts/20-load?id=Выполнение-fetch-запросов). Все последующие вызовы универсальных функций `load` происходят в браузере. Вы можете настроить поведение через [параметры страницы](/20-core-concepts/40-page-options). Если вы отключите [рендеринг на стороне сервера (SSR)](/20-core-concepts/40-page-options?id=ssr), вы получите SPA, а универсальные функции `load` будут _всегда_ выполняться на клиенте.

Функция `load` вызывается во время выполнения, если только вы не [пререндерите](/20-core-concepts/40-page-options?id=пререндер) страницу - в этом случае она вызывается во время сборки.

### Ввод

Как универсальные, так и серверные функции `load` имеют доступ к свойствам, описывающим запрос (`params`, `route` и `url`) и различные функции (`fetch`, `setHeaders`, `parent` и `depends`). Они описаны в следующих разделах.

Серверные функции `load` вызываются с помощью `ServerLoadEvent`, которое наследует `clientAddress`, `cookies`, `locals`, `platform` и `request` от `RequestEvent`.

Универсальные функции `load` вызываются с помощью `LoadEvent`, которое имеет свойство `data`. Если у вас есть функции `load` и в `+page.js`, и в `+page.server.js` (или `+layout.js` и `+layout.server.js`), возвращаемое значение функции `load` сервера является свойством `data` аргумента универсальной функции `load`.

### Вывод

Универсальная функция `load` может возвращать объект, содержащий любые значения, включая такие вещи, как пользовательские классы и конструкторы компонентов.

Функция `load` сервера должна возвращать данные, которые могут быть сериализованы с помощью [devalue](https://github.com/rich-harris/devalue) - все, что может быть представлено как JSON, плюс такие вещи, как `BigInt`, `Date`, `Map`, `Set` и `RegExp`, или повторяющиеся/циклические ссылки - чтобы их можно было передавать по сети. Ваши данные могут включать [промисы](/20-core-concepts/20-load?id=Потоковая-передача-с-промисами), в этом случае они будут передаваться в браузеры.

### Когда что использовать

Серверные функции `load` удобны, когда вам нужно получить доступ к данным непосредственно из базы данных или файловой системы, или необходимо использовать приватные переменные окружения.

Универсальные функции `load` полезны, когда вам нужно получить (`fetch`) данные из внешнего API и не нужны приватные учетные данные, поскольку SvelteKit может получить данные непосредственно из API, а не через ваш сервер. Они также полезны, когда вам нужно вернуть что-то, что не может быть сериализовано, например, конструктор компонента Svelte.

В редких случаях вам может понадобиться использовать оба метода вместе - например, вам может понадобиться вернуть экземпляр пользовательского класса, который был инициализирован данными с вашего сервера.

## Использование данных URL

Часто функция `load` тем или иным образом зависит от URL. Для этого функция `load` предоставляет вам `url`, `route` и `params`.

### url

Экземпляр [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL), содержащий такие свойства, как `origin`, `hostname`, `pathname` и `searchParams` (который содержит разобранную строку запроса как объект [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)). К `url.hash` нельзя получить доступ во время `загрузки`, поскольку он недоступен на сервере.

> В некоторых средах это значение извлекается из заголовков запроса во время рендеринга на стороне сервера. Если вы, например, используете [adapter-node](/25-build-and-deploy/40-adapter-node), вам может потребоваться настроить адаптер, чтобы URL был корректным.

### route

Содержит имя текущего каталога маршрутов, относительно `src/routes`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/a/[b]/[...c]/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export function load({ route }) {
	console.log(route.id); // '/a/[b]/[...c]'
}
```
#### **TypeScript**
**```src/routes/a/[b]/[...c]/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (({ route }) => {
  console.log(route.id); // '/a/[b]/[...c]'
}) satisfies PageLoad;
```
<!-- tabs:end -->

### params

`params` является производным от `url.pathname` и `route.id`.

Учитывая `route.id` из `/a/[b]/[...c]` и `url.pathname` из `/a/x/y/z`, объект `params` будет выглядеть следующим образом:

```json
{
	"b": "x",
	"c": "y/z"
}
```

## Выполнение fetch запросов
  
Для получения данных из внешнего API или обработчика `+server.js` можно использовать предоставляемую функцию `fetch`, которая ведет себя идентично [нативный `fetch` веб-API](https://developer.mozilla.org/en-US/docs/Web/API/fetch) с несколькими дополнительными возможностями:

- она может использоваться для выполнения авторизованных запросов на сервере, поскольку наследует заголовки `cookie` и `authorization` для запроса страницы
- она может выполнять относительные запросы на сервере (обычно `fetch` требует URL с оригиналом при использовании в контексте сервера)
- внутренние запросы (например, для маршрутов `+server.js`) идут прямо к функции-обработчику при работе на сервере, без накладных расходов на HTTP-вызов
- во время рендеринга на стороне сервера ответ будет перехвачен и вставлен в отрисованный HTML с помощью методов `text` и `json` объекта `Response`. Обратите внимание, что заголовки _не_ будут сериализованы, если они явно не включены через [`filterSerializedResponseHeaders`](/30-advanced/20-hooks?id=handle). Затем, во время гидратации, ответ будет считан из HTML, гарантируя согласованность и предотвращая дополнительный сетевой запрос - если вы получили предупреждение в консоли браузера при использовании `fetch` браузера вместо `load` `fetch`, то вот почему.

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/items/[id]/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, params }) {
	const res = await fetch(`/api/items/${params.id}`);
	const item = await res.json();

	return { item };
}
```
#### **TypeScript**
**```src/routes/items/[id]/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (async ({ fetch, params }) => {
  const res = await fetch(`/api/items/${params.id}`);
  const item = await res.json();
 
  return { item };
}) satisfies PageLoad;
```
<!-- tabs:end -->

> Cookies будут передаваться только в том случае, если целевой хост совпадает с приложением SvelteKit или является его более конкретным поддоменом.

## Файлы cookie и заголовки

Серверная функция `load` может получать и устанавливать [`cookies`](/50-reference/40-types?id=cookies).

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getUser(sessionid: string | undefined): Promise<{ name: string, avatar: string }>
}
```
**```src/routes/+layout.server.js```**
```js
import * as db from '$lib/server/database';

/** @type {import('./$types').LayoutServerLoad} */
export async function load({ cookies }) {
	const sessionid = cookies.get('sessionid');

	return {
		user: await db.getUser(sessionid)
	};
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare module '$lib/server/database' {
	export function getUser(sessionid: string | undefined): Promise<{ name: string, avatar: string }>
}
```
**```src/routes/+layout.server.ts```**
```ts
import * as db from '$lib/server/database';
import type { LayoutServerLoad } from './$types';
 
export const load = (async ({ cookies }) => {
  const sessionid = cookies.get('sessionid');
 
  return {
    user: await db.getUser(sessionid)
  };
}) satisfies LayoutServerLoad;
```
<!-- tabs:end -->

> При установке cookie-файлов обратите внимание на свойство `path`. По умолчанию `path` cookie - это текущее имя пути. Если вы, например, установите cookie на странице `admin/user`, то по умолчанию cookie будет доступен только на страницах `admin`. В большинстве случаев вы, вероятно, захотите установить `path` в `'/'`, чтобы cookie был доступен во всем вашем приложении.

Как серверные, так и универсальные функции `load` имеют доступ к функции `setHeaders`, которая при запуске на сервере может устанавливать заголовки для ответа. (При выполнении в браузере функция `setHeaders` не имеет эффекта). Это полезно, если вы хотите, чтобы страница была кэширована, например:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/products/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, setHeaders }) {
	const url = `https://cms.example.com/products.json`;
	const response = await fetch(url);

	// кэшировать страницу в течение того же времени.
	// как и базовые данные
	setHeaders({
		age: response.headers.get('age'),
		'cache-control': response.headers.get('cache-control')
	});

	return response.json();
}
```
#### **TypeScript**
**```src/routes/products/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (async ({ fetch, setHeaders }) => {
  const url = `https://cms.example.com/products.json`;
  const response = await fetch(url);
 
	// кэшировать страницу в течение того же времени.
	// как и базовые данные
  setHeaders({
    age: response.headers.get('age'),
    'cache-control': response.headers.get('cache-control')
  });
 
  return response.json();
}) satisfies PageLoad;
```
<!-- tabs:end -->

Установка одного и того же заголовка несколько раз (даже в отдельных функциях `load`) является ошибкой - вы можете установить данный заголовок только один раз. Вы не можете добавить заголовок `set-cookie` с помощью `setHeaders` - вместо этого используйте `cookies.set(name, value, options)`.

## Использование родительских данных

Иногда полезно, чтобы функция `load` получила доступ к данным из родительской функции `load`, что можно сделать с помощью `await parent()`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/+layout.js```**
```js
/** @type {import('./$types').LayoutLoad} */
export function load() {
	return { a: 1 };
}
```
#### **TypeScript**
**```src/routes/+layout.ts```**
```ts
import type { LayoutLoad } from './$types';
 
export const load = (() => {
  return { a: 1 };
}) satisfies LayoutLoad;
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/abc/+layout.js```**
```js
/** @type {import('./$types').LayoutLoad} */
export async function load({ parent }) {
	const { a } = await parent();
	return { b: a + 1 };
}
```
#### **TypeScript**
**```src/routes/abc/+layout.ts```**
```ts
import type { LayoutLoad } from './$types';
 
export const load = (async ({ parent }) => {
  const { a } = await parent();
  return { b: a + 1 };
}) satisfies LayoutLoad;
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/abc/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export async function load({ parent }) {
	const { a, b } = await parent();
	return { c: a + b };
}
```
#### **TypeScript**
**```src/routes/abc/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (async ({ parent }) => {
  const { a, b } = await parent();
  return { c: a + b };
}) satisfies PageLoad;
```
<!-- tabs:end -->


<!-- tabs:start -->
#### **JavaScript**
**```src/routes/abc/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<!-- отображает `1 + 2 = 3` -->
<p>{data.a} + {data.b} = {data.c}</p>
```
#### **TypeScript**
**```src/routes/abc/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
</script>
<!-- отображает `1 + 2 = 3` -->
<p>{data.a} + {data.b} = {data.c}</p>
```
<!-- tabs:end -->


> Обратите внимание, что функция `load` в `+page.js` получает объединенные данные от обеих функций `load` макета, а не только от непосредственного родителя.

Внутри `+page.server.js` и `+layout.server.js`, `parent` возвращает данные из родительских файлов `+layout.server.js`.

В `+page.js` или `+layout.js` он будет возвращать данные из родительских файлов `+layout.js`. Однако отсутствующий `+layout.js` рассматривается как функция `({ data }) => data`, что означает, что она также вернет данные из родительских файлов `+layout.server.js`, которые не "затенены" файлом `+layout.js`.

Будьте осторожны, чтобы не создавать водопады при использовании `await parent()`. Здесь, например, `getData(params)` не зависит от результата вызова `parent()`, поэтому мы должны вызвать его первым, чтобы избежать задержки рендеринга.

**```+page.js```**
```diff
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

## Ошибки

Если во время загрузки (`load`) произошла ошибка, будет выведен ближайший маршрут [`+error.svelte`](/20-core-concepts/10-routing?id=error). Для _ожидаемых_ ошибок используйте помощник `error` из `@sveltejs/kit`, чтобы указать код состояния HTTP и необязательное сообщение:

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
			isAdmin: boolean;
		}
	}
}
```
**```src/routes/admin/+layout.server.js```**
```js
import { error } from '@sveltejs/kit';

/** @type {import('./$types').LayoutServerLoad} */
export function load({ locals }) {
	if (!locals.user) {
		throw error(401, 'вход не выполнен');
	}

	if (!locals.user.isAdmin) {
		throw error(403, 'не администратор');
	}
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
			isAdmin: boolean;
		}
	}
}
```
**```src/routes/admin/+layout.server.ts```**
```ts
import { error } from '@sveltejs/kit';
import type { LayoutServerLoad } from './$types';
 
export const load = (({ locals }) => {
  if (!locals.user) {
    throw error(401, 'вход не выполнен');
  }
 
  if (!locals.user.isAdmin) {
    throw error(403, 'не администратор');
  }
}) satisfies LayoutServerLoad;
```
<!-- tabs:end -->

Если возникла _неожиданная_ ошибка, SvelteKit вызовет [`handleError`](/30-advanced/20-hooks?id=handleerror) и обработает ее как внутреннюю ошибку 500.


## Перенаправления

Чтобы перенаправить пользователей, используйте помощник `redirect` из `@sveltejs/kit`, чтобы указать место, на которое они должны быть перенаправлены, вместе с кодом статуса `3xx`.

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
		}
	}
}
```
**```src/routes/user/+layout.server.js```**
```js
import { redirect } from '@sveltejs/kit';

/** @type {import('./$types').LayoutServerLoad} */
export function load({ locals }) {
	if (!locals.user) {
		throw redirect(307, '/login');
	}
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user?: {
			name: string;
		}
	}
}
```
**```src/routes/user/+layout.server.ts```**
```ts
import { redirect } from '@sveltejs/kit';
import type { LayoutServerLoad } from './$types';
 
export const load = (({ locals }) => {
  if (!locals.user) {
    throw redirect(307, '/login');
  }
}) satisfies LayoutServerLoad;
```
<!-- tabs:end -->

> Убедитесь, что вы не перехватываете брошенное перенаправление, что не позволит SvelteKit обработать его.

В браузере вы также можете осуществлять навигацию программно вне функции `load`, используя [`goto`](/50-reference/30-modules?id=goto) из [`$app.navigation`](/50-reference/30-modules?id=appnavigation).

## Потоковая передача с промисами

Промисы на _верхнем уровне_ возвращаемого объекта будут ожидаться, что упрощает возврат нескольких промисов без создания водопада. При использовании серверной загрузки (`load`), _вложенные_ промисы будут передаваться в браузер по мере их выполнения. Это полезно, если у вас есть медленные, несущественные данные, так как вы можете начать рендеринг страницы до того, как все данные будут доступны:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/+page.server.js```**
```js
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
#### **TypeScript**
**```src/routes/+page.server.ts```**
```ts
import type { PageServerLoad } from './$types';
 
export const load = (() => {
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
}) satisfies PageServerLoad;
```
<!-- tabs:end -->

Это полезно, например, для создания каркасных состояний загрузки:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<p>
	один: {data.one}
</p>
<p>
	два: {data.two}
</p>
<p>
	три:
	{#await data.streamed.three}
		Загрузка...
	{:then value}
		{value}
	{:catch error}
		{error.message}
	{/await}
</p>
```
#### **TypeScript**
**```src/routes/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
</script>

<p>
	один: {data.one}
</p>
<p>
	два: {data.two}
</p>
<p>
	три:
	{#await data.streamed.three}
		Загрузка...
	{:then value}
		{value}
	{:catch error}
		{error.message}
	{/await}
</p>
```
<!-- tabs:end -->

На платформах, не поддерживающих потоковую передачу, таких как AWS Lambda, ответы будут буферизироваться. Это означает, что страница будет отображаться только после выполнения всех промисов.

> Потоковая передача данных будет работать только при включенном JavaScript. Вам следует избегать возврата вложенных промисов из универсальной функции `load`, если страница рендерится на сервере, поскольку они _не_ передаются потоком - вместо этого промис создается заново при повторном запуске функции в браузере.

## Параллельная загрузка

При рендеринге (или навигации по странице) SvelteKit запускает все функции `load` одновременно, избегая водопада запросов. Во время навигации на стороне клиента результаты вызова нескольких серверных функций `load` группируются в один ответ. Как только все функции `load` вернутся, страница будет отрисована.

## Повторное выполнение функций загрузки

SvelteKit отслеживает зависимости каждой функции `load`, чтобы избежать ее повторного запуска без необходимости во время навигации.

Например, если есть пара таких функций `load` как эти...

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

...та, что в `+page.server.js`, будет запущена заново, если мы перейдем от `/blog/trying-the-raw-meat-diet` к `/blog/i-regret-my-choices`, потому что `params.slug` изменился. А в `+layout.server.js` этого не произойдет, потому что данные все еще действительны. Другими словами, мы не будем вызывать `db.getPostSummaries()` во второй раз.

Функция `load`, которая вызывает `await parent()`, также будет повторно запущена, если родительская функция `load` будет повторно запущена.

Отслеживание зависимостей не применяется _после_ возвращения функции `load` - например, обращение к `params.x` внутри вложенного [промиса](/20-core-concepts/20-load?id=Потоковая-передача-с-промисами) не приведет к повторному запуску функции при изменении `params.x`. (Не волнуйтесь, вы получите предупреждение в процессе разработки, если случайно сделаете это). Вместо этого обратитесь к параметру в основном теле вашей функции `load`.

### Ручная инвалидация

Вы также можете повторно запустить функции `load`, которые применяются к текущей странице, используя функцию [`invalidate(url)`](/50-reference/30-modules?id=invalidate), которая повторно запускает все функции `load`, зависящие от `url`, и функцию [`invalidateAll()`](/50-reference/30-modules?id=invalidateall), которая повторно запускает каждую функцию `load`.

Функция `load` зависит от `url`, если она вызывает `fetch(url)` или `depends(url)`. Обратите внимание, что `url` может быть пользовательским идентификатором, начинающимся с `[a-z]:`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/random-number/+page.js```**
```js
/** @type {import('./$types').PageLoad} */
export async function load({ fetch, depends }) {
	// загрузка перезапускается при вызове `invalidate('https://api.example.com/random-number')`...
	const response = await fetch('https://api.example.com/random-number');

	// ...или когда вызывается `invalidate('app:random')`
	depends('app:random');

	return {
		number: await response.json()
	};
}
```
#### **TypeScript**
**```src/routes/random-number/+page.ts```**
```ts
import type { PageLoad } from './$types';
 
export const load = (async ({ fetch, depends }) => {
	// загрузка перезапускается при вызове `invalidate('https://api.example.com/random-number')`...
	const response = await fetch('https://api.example.com/random-number');

	// ...или когда вызывается `invalidate('app:random')`
	depends('app:random');

	return {
	number: await response.json()
	};
}) satisfies PageLoad;
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/random-number/+page.svelte```**
```svelte
<script>
	import { invalidate, invalidateAll } from '$app/navigation';

	/** @type {import('./$types').PageData} */
	export let data;

	function rerunLoadFunction() {
		// любое из них приведет к повторному запуску функции `load`.
		invalidate('app:random');
		invalidate('https://api.example.com/random-number');
		invalidate(url => url.href.includes('random-number'));
		invalidateAll();
	}
</script>

<p>случайное число: {data.number}</p>
<button on:click={rerunLoadFunction}>Обновить случайное число</button>
```
#### **TypeScript**
**```src/routes/random-number/+page.svelte```**
```svelte
<script lang="ts">
	import { invalidate, invalidateAll } from '$app/navigation';

	import type { PageData } from './$types';

	export let data: PageData;

	function rerunLoadFunction() {
		// любое из них приведет к повторному запуску функции `load`.
		invalidate('app:random');
		invalidate('https://api.example.com/random-number');
		invalidate(url => url.href.includes('random-number'));
		invalidateAll();
	}
</script>

<p>случайное число: {data.number}</p>
<button on:click={rerunLoadFunction}>Обновить случайное число</button>
```
<!-- tabs:end -->

Подводя итог, можно сказать, что функция `load` будет повторно запущена в следующих ситуациях:

- Она ссылается на свойство `params`, значение которого изменилось
- Она ссылается на свойство `url` (например, `url.pathname` или `url.search`), значение которого изменилось
- Вызывается `await parent()` и повторно запускается родительская функция `load`
- Объявлена зависимость от определенного URL через [`fetch`](/20-core-concepts/20-load?id=Выполнение-fetch-запросов) или [`depends`](/50-reference/40-types?id=loadevent), и этот URL был помечен недействительным с помощью [`invalidate(url)`](/50-reference/30-modules?id=invalidate)
- Все активные функции `load` были принудительно перезапущены с помощью [`invalidateAll()`](/50-reference/30-modules?id=invalidateall)

`params` и `url` могут изменяться в ответ на клик по ссылке `<a href="...">`, взаимодействие [`<form>`](/20-core-concepts/30-form-actions?id=get-против-post), вызов [`goto`](/50-reference/30-modules?id=goto) или [`redirect`](/50-reference/30-modules?id=redirect).

Обратите внимание, что повторный запуск функции `load` обновляет реквизит `data` внутри соответствующего `+layout.svelte` или `+page.svelte`; это _не_ приводит к повторному созданию компонента. В результате внутреннее состояние сохраняется. Если это не то, что вам нужно, вы можете сбросить все, что вам нужно сбросить, внутри обратного вызова [`afterNavigate`](/50-reference/30-modules?id=afternavigate), и/или обернуть ваш компонент в блок [`{#key ...}`](https://romkar.github.io/svelte-docs-rus/#/docs/03-template-syntax?id=key-).

## Дальнейшее чтение

- [Учебник: Загрузка данных](https://learn.svelte.dev/tutorial/page-data)
- [Учебник: Ошибки и переадресации](https://learn.svelte.dev/tutorial/error-basics)
- [Учебник: Расширенная загрузка](https://learn.svelte.dev/tutorial/await-parent)
