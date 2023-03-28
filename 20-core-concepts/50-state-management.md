# Управление состоянием
---

Если вы привыкли создавать приложения только для клиентов, управление состояниями в приложении, которое охватывает сервер и клиента, может показаться пугающим. В этом разделе приведены советы по предотвращению некоторых распространенных проблем.

## Избегайте общего состояния на сервере

Браузеры являются _состоятельными_ — состояние хранится в памяти по мере того, как пользователь взаимодействует с приложением. Серверы, с другой стороны, являются _бессостоятельными_ — содержание ответа полностью определяется содержанием запроса.

Концептуально это так. В реальности серверы часто являются долгоживущими и используются совместно несколькими пользователями. По этой причине важно не хранить данные в общих переменных. Например, рассмотрим следующий код:

<!-- tabs:start -->
#### **JavaScript**
**```+page.server.js```**
```js
let user;

/** @type {import('./$types').PageServerLoad} */
export function load() {
	return { user };
}

/** @type {import('./$types').Actions} */
export const actions = {
	default: async ({ request }) => {
		const data = await request.formData();

		// 🚫 НИКОГДА НЕ ДЕЛАЙТЕ ТАК!!!!111
		user = {
			name: data.get('name'),
			embarrassingSecret: data.get('secret')
		};
	}
}
```
#### **TypeScript**
**```+page.server.ts```**
```ts
import type { PageServerLoad, Actions } from './$types';
let user;
 
export const load = (() => {
	return { user };
}) satisfies PageServerLoad;
 
export const actions = {
	default: async ({ request }) => {
		const data = await request.formData();

		// 🚫 НИКОГДА НЕ ДЕЛАЙТЕ ТАК!!!!111
		user = {
			name: data.get('name'),
			embarrassingSecret: data.get('secret')
		};
	}
} satisfies Actions
```
<!-- tabs:end -->

Переменная `user` является общей для всех, кто подключается к этому серверу. Если Агата сообщила постыдный секрет, а Глеб посетил страницу после нее, Глеб будет знать [секрет](https://www.youtube.com/watch?v=v4s68ZhZAPA) Агаты. Кроме того, когда Агата вернется на сайт через день, сервер может перезагрузиться, что приведет к потере ее данных.

Вместо этого следует _аутентифицировать_ пользователя с помощью [`cookies`](/20-core-concepts/20-load?id=Файлы-cookie-и-заголовки) и сохранить данные в базе данных.

## Отсутствие побочных эффектов при загрузке

По этой же причине ваши функции `load` должны быть _чистыми_ — никаких побочных эффектов (кроме, может быть, случайного `console.log(...)`). Например, у вас может возникнуть соблазн записать в хранилище внутри функции `load`, чтобы потом использовать значение хранилища в своих компонентах:

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare module '$lib/user' {
	export const user: { set: (value: any) => void };
}
```
**```+page.js```**
```js
import { user } from '$lib/user';

/** @type {import('./$types').PageLoad} */
export async function load({ fetch }) {
	const response = await fetch('/api/user');

	// 🚫 НИКОГДА НЕ ДЕЛАЙТЕ ТАК!!!!111
	user.set(await response.json());
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare module '$lib/user' {
	export const user: { set: (value: any) => void };
}
```
**```+page.ts```**
```ts
import { user } from '$lib/user';
import type { PageLoad } from './$types';
 
export const load = (async ({ fetch }) => {
	const response = await fetch('/api/user');

	// 🚫 НИКОГДА НЕ ДЕЛАЙТЕ ТАК!!!!111
	user.set(await response.json());
}) satisfies PageLoad;
```
<!-- tabs:end -->

Как и в предыдущем примере, это помещает информацию одного пользователя в место, которое является общим для _всех_ пользователей. Вместо этого просто верните данные...

**```+page.js```**
```diff
export async function load({ fetch }) {
	const response = await fetch('/api/user');

+	return {
+		user: await response.json()
+	};
}
```

...и передайте их компонентам, которым они нужны, или используйте [`$page.data`](/20-core-concepts/20-load?id=pagedata).

Если вы не используете SSR, то нет риска случайного раскрытия данных одного пользователя другому. Но вы все равно должны избегать побочных эффектов в функциях `load` - без них ваше приложение будет гораздо проще понимать.

## Использование хранилищ с контекстом

Вам может быть интересно, как мы можем использовать `$page.data` и другие [app stores](/50-reference/30-modules?id=appstores), если мы не можем использовать наши собственные хранилища. Ответ заключается в том, что хранилища приложений на сервере используют [контекстный API Svelte](https://learn.svelte.dev/tutorial/context-api) - хранилище прикрепляется к дереву компонентов с помощью `setContext`, а когда вы подписываетесь, вы получаете его с помощью `getContext`. Мы можем сделать то же самое с нашими собственными хранилищами:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/+layout.svelte```**
```svelte
<script>
	import { setContext } from 'svelte';
	import { writable } from 'svelte/store';

	/** @type {import('./$types').LayoutData} */
	export let data;

	// Создайте хранилище и обновляйте его по мере необходимости...
	const user = writable();
	$: user.set(data.user);

	// ...и добавьте его в контекст, чтобы дочерние компоненты могли получить к нему доступ
	setContext('user', user);
</script>
```
#### **TypeScript**
**```src/routes/+layout.svelte```**
```svelte
<script lang="ts">
	import { setContext } from 'svelte';
	import { writable } from 'svelte/store';

	import type { LayoutData } from './$types';

	export let data: LayoutData;
	// Создайте хранилище и обновляйте его по мере необходимости...
	const user = writable();
	$: user.set(data.user);
	// ...и добавьте его в контекст, чтобы дочерние компоненты могли получить к нему доступ
	setContext('user', user);
</script>
```
<!-- tabs:end -->

**```src/routes/user/+page.svelte```**
```svelte
<script>
	import { getContext } from 'svelte';

	// Получение хранилища пользователя из контекста
	const user = getContext('user');
</script>

<p>Добро пожаловать {$user.name}</p>
```

Если вы не используете SSR (и можете гарантировать, что вам не понадобится использовать SSR в будущем), то вы можете безопасно хранить состояние в общем модуле, не используя контекстный API.

## Состояние компонента сохраняется

Когда вы перемещаетесь по своему приложению, SvelteKit повторно использует существующие компоненты макета и страницы. Например, если у вас есть такой маршрут...

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;

	// ЭТОТ КОД СОДЕРЖИТ ОШИБКИ!
	const wordCount = data.content.split(' ').length;
	const estimatedReadingTime = wordCount / 250;
</script>

<header>
	<h1>{data.title}</h1>
	<p>Время чтения: {Math.round(estimatedReadingTime)} минут</p>
</header>

<div>{@html data.content}</div>
```
#### **TypeScript**
**```src/routes/blog/[slug]/+page.svelte```**
```svelte
<script lang="ts">
	import type { PageData } from './$types';

	export let data: PageData;
	// ЭТОТ КОД СОДЕРЖИТ ОШИБКИ!
	const wordCount = data.content.split(' ').length;
	const estimatedReadingTime = wordCount / 250;
</script>

<header>
	<h1>{data.title}</h1>
	<p>Время чтения: {Math.round(estimatedReadingTime)} минут</p>
</header>

<div>{@html data.content}</div>
```
<!-- tabs:end -->

...то переход от `/blog/my-short-post` к `/blog/my-long-post` не приведет к уничтожению и воссозданию компонента. Реквизит `data` (и, соответственно, `data.title` и `data.content`) изменится, но поскольку код не выполняется повторно, `estimatedReadingTime` не будет пересчитан.

Вместо этого нам нужно сделать значение [_реактивным_](https://learn.svelte.dev/tutorial/reactive-assignments):

**```src/routes/blog/[slug]/+page.svelte```**
```diff
 <script>
	/** @type {import('./$types').PageData} */
	export let data;

+	$: wordCount = data.content.split(' ').length;
+	$: estimatedReadingTime = wordCount / 250;
 </script>
```

Подобное повторное использование компонентов означает, что такие вещи, как состояние прокрутки боковой панели, сохраняются, и вы можете легко анимировать изменяющиеся значения. Однако, если вам необходимо полностью уничтожить и снова монтировать компонент при навигации, вы можете использовать этот шаблон:

```svelte
{#key $page.url.pathname}
	<BlogPost title={data.title} content={data.title} />
{/key}
```

## Хранение состояния в URL

Если у вас есть состояние, которое должно пережить перезагрузку и/или повлиять на SSR, такое как фильтры или правила сортировки в таблице, то параметры поиска URL (например, `?sort=price&order=ascending`) являются хорошим местом для их размещения. Вы можете поместить их в атрибуты `<a href="...">` или `<form action="...">`, или задать их программно через `goto('?key=value')`. Доступ к ним можно получить внутри функций `load` через параметр `url`, а внутри компонентов - через `$page.url.searchParams`.

## Хранение эфемерного состояния в снэпшотах

Некоторые состояния пользовательского интерфейса, такие как "открыт ли аккордеон?", являются одноразовыми — если пользователь перейдет на другую страницу или обновит ее, не имеет значения, если состояние будет потеряно. В некоторых случаях вы _хотите_, чтобы данные сохранялись, если пользователь перейдет на другую страницу и вернется, но хранить состояние в URL или в базе данных было бы излишним. Для этого SvelteKit предоставляет [снэпшоты](/30-advanced/65-snapshots), которые позволяют связать состояние компонента с записью истории.