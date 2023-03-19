# Маршрутизация
---

В основе SvelteKit лежит _маршрутизатор на базе файловой системы_. Маршруты вашего приложения - т.е. пути URL, к которым могут обращаться пользователи - определяются каталогами в вашей кодовой базе:

- `src/routes` корневой маршрут
- `src/routes/about` создает маршрут `/about`
- `src/routes/blog/[slug]` создает маршрут с _параметром_, `slug`, который может быть использован для динамической загрузки данных при запросе пользователем страницы типа `/blog/hello-world`.

> Вы можете изменить `src/routes` на другой каталог, отредактировав [конфигурацию проекта](/50-reference/10-configuration).

Каждый каталог маршрутов содержит один или несколько _файлов маршрутов_, которые можно идентифицировать по префиксу `+`.

## +page

### +page.svelte

Компонент `+page.svelte` определяет страницу вашего приложения. По умолчанию страницы отображаются как на сервере ([SSR](/60-appendix/30-glossary?id=ssr)) для первоначального запроса, так и в браузере ([CSR](/60-appendix/30-glossary?id=csr)) для последующей навигации.

**```src/routes/+page.svelte```**
```svelte
<h1>Здравствуйте и добро пожаловать на мой сайт!</h1>
<a href="/about">О моем сайте</a>
```

**```src/routes/about/+page.svelte```**
```svelte
<h1>Об этом сайте</h1>
<p>СДЕЛАТЬ...</p>
<a href="/">Главная</a>
```

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>

<h1>{data.title}</h1>
<div>{@html data.content}</div>
```
#### **TypeScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
</script>

<h1>{data.title}</h1>
<div>{@html data.content}</div>
```
<!-- tabs:end -->


> Обратите внимание, что SvelteKit использует элементы `<a>` для перехода между маршрутами, а не специфический для фреймворка компонент `<Link>`.

### +page.js

Часто перед отображением страницы требуется загрузить некоторые данные. Для этого мы добавляем модуль `+page.js` (или `+page.ts`, если вы разбираетесь в TypeScript), который экспортирует функцию `load`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.js```**
```js
import { error } from '@sveltejs/kit';

/** @type {import('./$types').PageLoad} */
export function load({ params }) {
	if (params.slug === 'hello-world') {
		return {
			title: 'Здравствуй мир!',
			content: 'Добро пожаловать в наш блог. Но чтобы вы поняли, откуда возникает...'
		};
	}

	throw error(404, 'Не найдено');
}
```

#### **TypeScript**
**```src/routes/blog/[slug]/+page.ts```**
```ts
import { error } from '@sveltejs/kit';
import type { PageLoad } from './$types';
 
export const load = (({ params }) => {
  if (params.slug === 'hello-world') {
    return {
      title: 'Здравствуй мир!',
      content: 'Добро пожаловать в наш блог. Но чтобы вы поняли, откуда возникает...'
    };
  }
 
  throw error(404, 'Не найдено');
}) satisfies PageLoad;
```
<!-- tabs:end -->

Эта функция работает вместе с `+page.svelte`, что означает, что она выполняется на сервере во время рендеринга на стороне сервера (SSR) и в браузере во время навигации на стороне клиента. Полную информацию об API см. в [`load`](/20-core-concepts/20-load).

Помимо `load`, `+page.js` может экспортировать значения, которые настраивают поведение страницы:

- `export const prerender = true` или `false` или `'auto'`
- `export const ssr = true` или `false`
- `export const csr = true` или `false`

Более подробную информацию о них можно найти в разделе [параметры страницы](/20-core-concepts/40-page-options).

### +page.server.js

Если ваша функция `load` может работать только на сервере - например, если она должна получать данные из базы данных или вам нужен доступ к приватным [переменным окружения](/50-reference/30-modules?id=envstaticprivate), таким как API ключи - тогда вы можете переименовать `+page.js` в `+page.server.js` и изменить тип `PageLoad` на `PageServerLoad`.

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare global {
	const getPostFromDatabase: (slug: string) => {
		title: string;
		content: string;
	}
}

export {};
```

**```src/routes/blog/[slug]/+page.server.js```**
```js
import { error } from '@sveltejs/kit';

/** @type {import('./$types').PageServerLoad} */
export async function load({ params }) {
	const post = await getPostFromDatabase(params.slug);

	if (post) {
		return post;
	}

	throw error(404, 'Не найдено');
}
```

#### **TypeScript**
**```ambient.d.ts```**
```ts
declare global {
	const getPostFromDatabase: (slug: string) => {
		title: string;
		content: string;
	}
}

export {};
```

**```src/routes/blog/[slug]/+page.server.ts```**
```ts
import { error } from '@sveltejs/kit';
import type { PageServerLoad } from './$types';
 
export const load = (async ({ params }) => {
  const post = await getPostFromDatabase(params.slug);
 
  if (post) {
    return post;
  }
 
  throw error(404, 'Не найдено');
}) satisfies PageServerLoad;
```
<!-- tabs:end -->

Во время навигации на стороне клиента SvelteKit будет загружать эти данные с сервера, что означает, что возвращаемое значение должно быть сериализуемым с помощью [devalue](https://github.com/rich-harris/devalue). Полную информацию об API смотрите в [`load`](/20-core-concepts/20-load).

Как и `+page.js`, `+page.server.js` может экспортировать [параметры страницы](/20-core-concepts/40-page-options) - `prerender`, `ssr` и `csr`.

Файл `+page.server.js` может также экспортировать _действия_. Если `load` позволяет считывать данные с сервера, то `actions` позволяет записывать данные _на_ сервер с помощью элемента `<form>`. Чтобы узнать, как их использовать, обратитесь к разделу [действия формы](/20-core-concepts/30-form-actions).

## +error

Если во время загрузки (`load`) произойдет ошибка, SvelteKit отобразит страницу ошибки по умолчанию. Вы можете настроить страницу ошибки для каждого маршрута, добавив файл `+error.svelte`:

**```src/routes/blog/[slug]/+error.svelte```**
```svelte
<script>
	import { page } from '$app/stores';
</script>

<h1>{$page.status}: {$page.error.message}</h1>
```

SvelteKit будет "подниматься по дереву" в поисках ближайшей границы ошибки - если файл выше не существует, он попробует `src/routes/blog/+error.svelte`, а затем `src/routes/+error.svelte`, прежде чем вывести страницу ошибки по умолчанию. Если _это_ не удается (или если ошибка была вызвана функцией `load` корневого `+layout`, который находится "над" корневым `+error`), SvelteKit отступит и отобразит статическую страницу ошибки, которую вы можете настроить, создав файл `src/error.html`.

Если ошибка возникает внутри функции `load` в `+layout(.server).js`, то ближайшей границей ошибки в дереве будет файл `+error.svelte` _над_ этим макетом (не рядом с ним).

Если маршрут не найден (404), будет использоваться `src/routes/+error.svelte` (или страница ошибки по умолчанию, если этот файл не существует).

> `+error.svelte` _не_ используется, когда ошибка возникает внутри [`handle`](/30-advanced/20-hooks?id=handle) или обработчика запроса [+server.js](/20-core-concepts/10-routing?id=server).

Подробнее об обработке ошибок можно прочитать [здесь](/30-advanced/25-errors).

## +layout

До сих пор мы рассматривали страницы как полностью самостоятельные компоненты - при навигации существующий компонент `+page.svelte` будет уничтожен, а на его месте появится новый.

Но во многих приложениях есть элементы, которые должны быть видны на _каждой_ странице, например, навигация верхнего уровня или нижний колонтитул. Вместо того чтобы повторять их в каждом `+page.svelte`, мы можем поместить их в _layouts_.

### +layout.svelte

Чтобы создать макет (layout), который будет применяться к каждой странице, создайте файл `src/routes/+layout.svelte`. Макет по умолчанию (тот, который использует SvelteKit, если вы не создали свой собственный) выглядит следующим образом...

```html
<slot></slot>
```

...но мы можем добавить любую разметку, стили и поведение, какие захотим. Единственное требование - чтобы компонент включал `<slot>` для содержимого страницы. Например, давайте добавим панель навигации:

**```src/routes/+layout.svelte```**
```html
<nav>
	<a href="/">Главная</a>
	<a href="/about">Об</a>
	<a href="/settings">Настройки</a>
</nav>

<slot></slot>
```

Если мы создадим страницы для `/`, `/about` и `/settings`...

**```src/routes/+page.svelte```**
```html
<h1>Главная</h1>
```

**```src/routes/about/+page.svelte```**
```html
<h1>Об</h1>
```

**```src/routes/settings/+page.svelte```**
```html
<h1>Настройки</h1>
```

...навигатор всегда будет виден, а при клике между тремя страницами будет заменяться только `<h1>`.

Макеты могут быть _вложенными_. Допустим, у нас не одна страница `/settings`, а вложенные страницы `/settings/profile` и `/settings/notifications` с общим подменю (реальный пример см. на [github.com/settings](https://github.com/settings)).

Мы можем создать макет, который будет применяться только к страницам ниже `/settings` (наследуя при этом корневой макет с навигатором верхнего уровня):

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/settings/+layout.svelte```**
```svelte
<script>
	/** @type {import('./$types').LayoutData} */
	export let data;
</script>

<h1>Настройки</h1>

<div class="submenu">
	{#each data.sections as section}
		<a href="/settings/{section.slug}">{section.title}</a>
	{/each}
</div>

<slot></slot>
```

#### **TypeScript**
**```src/routes/settings/+layout.svelte```**
```svelte
<script lang="ts">
  import type { LayoutData } from './$types';

  export let data: LayoutData;
</script>

<h1>Настройки</h1>

<div class="submenu">
  {#each data.sections as section}
    <a href="/settings/{section.slug}">{section.title}</a>
  {/each}
</div>

<slot></slot>
```
<!-- tabs:end -->

По умолчанию каждый макет наследует макет, расположенный над ним. Иногда это не то, что вам нужно - в этом случае вам может помочь [продвинутые макеты](/30-advanced/10-advanced-routing?id=Продвинутые-макеты).

### +layout.js

Подобно тому, как `+page.svelte` загружает данные из `+page.js`, ваш компонент `+layout.svelte` может получать данные из функции [`load`](/20-core-concepts/20-load) в `+layout.js`.

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/settings/+layout.js```**
```js
/** @type {import('./$types').LayoutLoad} */
export function load() {
	return {
		sections: [
			{ slug: 'profile', title: 'Профиль' },
			{ slug: 'notifications', title: 'Уведомления' }
		]
	};
}
```
#### **TypeScript**
**```src/routes/settings/+layout.ts```**
```ts
import type { LayoutLoad } from './$types';
 
export const load = (() => {
  return {
    sections: [
      { slug: 'profile', title: 'Профиль' },
      { slug: 'notifications', title: 'Уведомления' }
    ]
  };
}) satisfies LayoutLoad;
```
<!-- tabs:end -->


Если `+layout.js` экспортирует [параметры страницы](/20-core-concepts/40-page-options) - `prerender`, `ssr` и `csr` - они будут использоваться по умолчанию для дочерних страниц.

Данные, возвращаемые из функции `load` макета, также доступны всем его дочерним страницам:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/settings/profile/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;

	console.log(data.sections); // [{ slug: 'profile', title: 'Профиль' }, ...]
</script>
```
#### **TypeScript**
**```src/routes/settings/profile/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;

  console.log(data.sections); // [{ slug: 'profile', title: 'Профиль' }, ...]
</script>
```
<!-- tabs:end -->

> Часто данные макета остаются неизменными при переходе между страницами. SvelteKit будет интеллектуально перезапускать функции [`load`](/20-core-concepts/20-load), когда это необходимо.

### +layout.server.js

Чтобы запустить функцию `load` вашего макета на сервере, переместите ее в `+layout.server.js` и измените тип `LayoutLoad` на `LayoutServerLoad`.

Как и `+layout.js`, `+layout.server.js` может экспортировать [параметры страницы](/20-core-concepts/40-page-options) - `prerender`, `ssr` и `csr`.

## +server

Помимо страниц, вы можете определять маршруты с помощью файла `+server.js` (иногда называемого "маршрутом API" или "конечной точкой"), который дает вам полный контроль над ответом. Ваш файл `+server.js` (или `+server.ts`) экспортирует функции, соответствующие таким HTTP-глаголам, как `GET`, `POST`, `PATCH`, `PUT`, `DELETE` и `OPTIONS`, которые принимают аргумент `RequestEvent` и возвращают объект [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response).

Например, мы можем создать маршрут `/api/random-number` с обработчиком `GET`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/api/random-number/+server.js```**
```js
import { error } from '@sveltejs/kit';

/** @type {import('./$types').RequestHandler} */
export function GET({ url }) {
	const min = Number(url.searchParams.get('min') ?? '0');
	const max = Number(url.searchParams.get('max') ?? '1');

	const d = max - min;

	if (isNaN(d) || d < 0) {
		throw error(400, 'min и max должны быть числами, и min должно быть меньше max');
	}

	const random = min + Math.random() * d;

	return new Response(String(random));
}
```
#### **TypeScript**
**```src/routes/api/random-number/+server.ts```**
```ts
import { error } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
 
export const GET = (({ url }) => {
  const min = Number(url.searchParams.get('min') ?? '0');
  const max = Number(url.searchParams.get('max') ?? '1');
 
  const d = max - min;
 
  if (isNaN(d) || d < 0) {
    throw error(400, 'min и max должны быть числами, и min должно быть меньше max');
  }
 
  const random = min + Math.random() * d;
 
  return new Response(String(random));
}) satisfies RequestHandler;
```
<!-- tabs:end -->

Первым аргументом `Response` может быть [`ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream), что позволяет передавать большие объемы данных или создавать события, отправляемые на сервер (если только не развертывание на платформах, буферизирующих ответы, таких как AWS Lambda).

Для удобства вы можете использовать методы [`error`](/50-reference/30-modules?id=error), [`redirect`](/50-reference/30-modules?id=redirect) и [`json`](/50-reference/30-modules?id=json) из `@sveltejs/kit` (но это не обязательно).

Если произошла ошибка (либо `throw error(...)`, либо неожиданная ошибка), ответом будет JSON-представление ошибки или страница с ошибкой, которую можно настроить через `src/error.html`, в зависимости от заголовка `Accept`. Компонент [`+error.svelte`](/20-core-concepts/10-routing?id=error) в этом случае _не_ будет отображаться. Подробнее об обработке ошибок вы можете прочитать [здесь](/30-advanced/25-errors).

> При создании обработчика `OPTIONS` обратите внимание, что Vite будет инжектировать заголовки `Access-Control-Allow-Origin` и `Access-Control-Allow-Methods` - они не будут присутствовать в продакшене, если вы их не добавите.

### Получение данных

Экспортируя обработчики `POST`/`PUT`/`PATCH`/`DELETE`/`OPTIONS`, файлы `+server.js` можно использовать для создания полноценного API:

**```src/routes/add/+page.svelte```**
```svelte
<script>
	let a = 0;
	let b = 0;
	let total = 0;

	async function add() {
		const response = await fetch('/api/add', {
			method: 'POST',
			body: JSON.stringify({ a, b }),
			headers: {
				'content-type': 'application/json'
			}
		});

		total = await response.json();
	}
</script>

<input type="number" bind:value={a}> +
<input type="number" bind:value={b}> =
{total}

<button on:click={add}>Рассчитать</button>
```

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/api/add/+server.js```**
```js
import { json } from '@sveltejs/kit';

/** @type {import('./$types').RequestHandler} */
export async function POST({ request }) {
	const { a, b } = await request.json();
	return json(a + b);
}
```
#### **TypeScript**
**```src/routes/api/add/+server.ts```**
```ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
 
export const POST = (async ({ request }) => {
  const { a, b } = await request.json();
  return json(a + b);
}) satisfies RequestHandler;
```
<!-- tabs:end -->

> В целом, [действия формы](/20-core-concepts/30-form-actions) - это лучший способ отправки данных из браузера на сервер.

### Соглашение по контенту

Файлы `+server.js` могут быть размещены в том же каталоге, что и файлы `+page`, что позволяет одному и тому же маршруту быть либо страницей, либо конечной точкой API. Для определения маршрута SvelteKit применяет следующие правила:

- `PUT`/`PATCH`/`DELETE`/`OPTIONS` запросы всегда обрабатываются `+server.js`, поскольку они не относятся к страницам
- Запросы `GET`/`POST` рассматриваются как запросы страниц, если в заголовке `accept` приоритетом является `text/html` (другими словами, это запрос страницы браузера), в противном случае они обрабатываются `+server.js`.

## $types

Во всех примерах выше мы импортировали типы из файла `$types.d.ts`. Это файл, который SvelteKit создает для вас в скрытом каталоге, если вы используете TypeScript (или JavaScript с аннотациями типов JSDoc), чтобы обеспечить безопасность типов при работе с корневыми файлами.

Например, аннотация `export let data` с `PageData` (или `LayoutData`, для файла `+layout.svelte`) говорит TypeScript, что тип `data` - это то, что было возвращено из `load`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;
</script>
```
#### **TypeScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData } from './$types';

  export let data: PageData;
</script>
```
<!-- tabs:end -->

В свою очередь, аннотирование функции `load` с помощью `PageLoad`, `PageServerLoad`, `LayoutLoad` или `LayoutServerLoad` (для `+page.js`, `+page.server.js`, `+layout.js` и `+layout.server.js` соответственно) гарантирует, что `params` и возвращаемое значение будут правильно типизированы.

## Другие файлы

Любые другие файлы внутри каталога маршрута игнорируются SvelteKit. Это означает, что вы можете размещать компоненты и полезные модули в тех маршрутах, которые в них нуждаются.

Если компоненты и модули нужны нескольким маршрутам, то лучше поместить их в [`$lib`](/50-reference/30-modules?id=lib).

## Дальнейшее чтение

- [Учебник: Маршрутизация](https://learn.svelte.dev/tutorial/pages)
- [Учебник: Маршруты API](https://learn.svelte.dev/tutorial/get-handlers)
- [Документация: Расширенная маршрутизация](/30-advanced/10-advanced-routing)
