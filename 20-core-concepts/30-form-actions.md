# Действия формы
---

Файл `+page.server.js` может экспортировать _действия_, которые позволяют вам отправлять `POST` данные на сервер с помощью элемента `<form>`.

При использовании `<form>`, JavaScript на стороне клиента необязателен, но вы можете легко _прогрессивно улучшить_ взаимодействие с формой с помощью JavaScript для обеспечения наилучшего пользовательского опыта.

## Действия по умолчанию

В простейшем случае страница объявляет действие `default`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/login/+page.server.js```**
```js
/** @type {import('./$types').Actions} */
export const actions = {
	default: async (event) => {
		// СДЕЛАТЬ: вход пользователя в систему
	}
};
```
#### **TypeScript**
**```src/routes/login/+page.server.ts```**
```ts
import type { Actions } from './$types';
 
export const actions = {
	default: async (event) => {
		// СДЕЛАТЬ: вход пользователя в систему
	}
} satisfies Actions;
```
<!-- tabs:end -->

Чтобы вызвать это действие со страницы `/login`, просто добавьте `<form>` - JavaScript не нужен:

**```src/routes/login/+page.svelte```**
```svelte
<form method="POST">
	<label>
		Email
		<input name="email" type="email">
	</label>
	<label>
		Password
		<input name="password" type="password">
	</label>
	<button>Log in</button>
</form>
```

Если кто-то нажмет на кнопку, браузер отправит данные формы через `POST` запрос на сервер, запустив действие по умолчанию.

> Действия всегда используют `POST` запросы, так как `GET` запросы никогда не должны иметь побочных эффектов.

Мы также можем вызвать действие с других страниц (например, если есть виджет входа в панели навигации в корневом макете), добавив атрибут `action`, указывающий на страницу:

**```src/routes/+layout.svelte```**
```html
<form method="POST" action="/login">
	<!-- содержимое -->
</form>
```

## Именованные действия

Вместо одного действия `default`, страница может иметь столько именованных действий, сколько ей необходимо:

**```src/routes/login/+page.server.js```**
```diff
/** @type {import('./$types').Actions} */
export const actions = {
-	default: async (event) => {
+	login: async (event) => {
		// СДЕЛАТЬ: вход пользователя в систему
	},
+	register: async (event) => {
+		// СДЕЛАТЬ: регистрацию пользователя
+	}
};
```

Чтобы вызвать именованное действие, добавьте параметр запроса с именем, с префиксным символом `/`:

**```src/routes/login/+page.svelte```**
```svelte
<form method="POST" action="?/register">
```

**```src/routes/+layout.svelte```**
```svelte
<form method="POST" action="/login?/register">
```

Наряду с атрибутом `action`, мы можем использовать атрибут `formaction` на кнопке, чтобы те же данные формы передавались методом `POST` в действие, отличное от родительского `<form>`:

**```src/routes/login/+page.svelte```**
```diff
-<form method="POST">
+<form method="POST" action="?/login">
	<label>
		Электронная почта
		<input name="email" type="email">
	</label>
	<label>
		Пароль
		<input name="password" type="password">
	</label>
	<button>Войти</button>
+	<button formaction="?/register">Register</button>
 </form>
```

> Мы не можем иметь действия по умолчанию рядом с именованными действиями, потому что если вы передадите методом POST на именованное действие без перенаправления, параметр запроса сохраняется в URL, что означает, что следующий POST по умолчанию будет проходить через предыдущее именованное действие.

## Анатомия действия

Каждое действие получает объект `RequestEvent`, позволяя вам читать данные с помощью `request.formData()`. После обработки запроса (например, регистрации пользователя путем установки cookie), действие может ответить данными, которые будут доступны через свойство `form` на соответствующей странице и через `$page.form` в масштабах всего приложения до следующего обновления.

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/login/+page.server.js```**
```js
/** @type {import('./$types').PageServerLoad} */
export async function load({ cookies }) {
	const user = await db.getUserFromSession(cookies.get('sessionid'));
	return { user };
}

/** @type {import('./$types').Actions} */
export const actions = {
	login: async ({ cookies, request }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

		const user = await db.getUser(email);
		cookies.set('sessionid', await db.createSession(user));

		return { success: true };
	},
	register: async (event) => {
		// СДЕЛАТЬ: регистрацию пользователя
	}
};
```
#### **TypeScript**
**```src/routes/login/+page.server.ts```**
```ts
import type { PageServerLoad, Actions } from './$types';
 
export const load = (async ({ cookies }) => {
  const user = await db.getUserFromSession(cookies.get('sessionid'));
  return { user };
}) satisfies PageServerLoad;
 
export const actions = {
	login: async ({ cookies, request }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

		const user = await db.getUser(email);
		cookies.set('sessionid', await db.createSession(user));

		return { success: true };
	},
	register: async (event) => {
		// СДЕЛАТЬ: регистрацию пользователя
	}
} satisfies Actions;
```
<!-- tabs:end -->


<!-- tabs:start -->
#### **JavaScript**
**```src/routes/login/+page.svelte```**
```svelte
<script>
	/** @type {import('./$types').PageData} */
	export let data;

	/** @type {import('./$types').ActionData} */
	export let form;
</script>

{#if form?.success}
	<!-- это сообщение является эфемерным; оно существует, потому что страница была отображена в
		 в ответ на отправку формы. оно исчезнет, если пользователь перезагрузит страницу. -->
	<p>Успешный вход в систему! С возвращением, {data.user.name}</p>
{/if}
```
#### **TypeScript**
**```src/routes/login/+page.svelte```**
```svelte
<script lang="ts">
  import type { PageData, ActionData } from './$types';

  export let data: PageData;

  export let form: ActionData;
</script>

{#if form?.success}
	<!-- это сообщение является эфемерным; оно существует, потому что страница была отображена в
		 в ответ на отправку формы. оно исчезнет, если пользователь перезагрузит страницу. -->
	<p>Успешный вход в систему! С возвращением, {data.user.name}</p>
{/if}
```
<!-- tabs:end -->

### Ошибки валидации

Если запрос не может быть обработан из-за недопустимых данных, вы можете вернуть ошибки валидации вместе с ранее отправленными значениями формы обратно пользователю, чтобы он мог повторить попытку. Функция `fail` позволяет вернуть код состояния HTTP (обычно 400 или 422, в случае ошибок валидации) вместе с данными. Код состояния доступен через `$page.status`, а данные - через `form`:

**```src/routes/login/+page.server.js```**
```diff
+import { fail } from '@sveltejs/kit';

/** @type {import('./$types').Actions} */
export const actions = {
	login: async ({ cookies, request }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

+		if (!email) {
+			return fail(400, { email, missing: true });
+		}

		const user = await db.getUser(email);

+		if (!user || user.password !== hash(password)) {
+			return fail(400, { email, incorrect: true });
+		}

		cookies.set('sessionid', await db.createSession(user));

		return { success: true };
	},
	register: async (event) => {
		// СДЕЛАТЬ: регистрацию пользователя
	}
};
```

> Обратите внимание, что в качестве меры предосторожности мы возвращаем на страницу только электронное письмо, а не пароль.

**```src/routes/login/+page.svelte```**
```diff
 <form method="POST" action="?/login">
+	{#if form?.missing}<p class="error">The email field is required</p>{/if}
+	{#if form?.incorrect}<p class="error">Invalid credentials!</p>{/if}
	<label>
		Email
-		<input name="email" type="email">
+		<input name="email" type="email" value={form?.email ?? ''}>
	</label>
	<label>
		Пароль
		<input name="password" type="password">
	</label>
	<button>Войти</button>
	<button formaction="?/register">Регистрация</button>
 </form>
```

Возвращаемые данные должны быть сериализуемы как JSON. В остальном структура зависит только от вас. Например, если у вас на странице несколько форм, вы можете различать, на какую `<form>` ссылаются возвращаемые `form` данные с помощью свойства `id` или аналогичного.

### Перенаправления

Перенаправления (и ошибки) работают точно так же, как и в функции [`load`](/20-core-concepts/20-load?id=Перенаправления):

**```src/routes/login/+page.server.js```**
```diff
+import { fail, redirect } from '@sveltejs/kit';

/** @type {import('./$types').Actions} */
export const actions = {
+	login: async ({ cookies, request, url }) => {
		const data = await request.formData();
		const email = data.get('email');
		const password = data.get('password');

		const user = await db.getUser(email);
		if (!user) {
			return fail(400, { email, missing: true });
		}

		if (user.password !== hash(password)) {
			return fail(400, { email, incorrect: true });
		}

		cookies.set('sessionid', await db.createSession(user));

+		if (url.searchParams.has('redirectTo')) {
+			throw redirect(303, url.searchParams.get('redirectTo'));
+		}

		return { success: true };
	},
	register: async (event) => {
		// СДЕЛАТЬ: регистрацию пользователя
	}
};
```

## Загрузка данных

После выполнения действия страница будет перезагружена (если не произойдет перенаправления или непредвиденной ошибки), а возвращаемое значение действия будет доступно странице в качестве реквизита `form`. Это означает, что функции `load` вашей страницы будут выполняться после завершения действия.

Обратите внимание, что функция `handle` выполняется до вызова действия и не запускается повторно перед функциями `load`. Это означает, что если, например, вы используете `handle` для заполнения `event.locals` на основе cookie, вы должны обновить `event.locals` при установке или удалении cookie в действии:

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user: {
			name: string;
		} | null
	}
}
```
**```global.d.ts```**
```ts
declare global {
	function getUser(sessionid: string | undefined): {
		name: string;
	};
}

export {};
```
**```src/hooks.server.js```**
```js
/** @type {import('@sveltejs/kit').Handle} */
export async function handle({ event, resolve }) {
	event.locals.user = await getUser(event.cookies.get('sessionid'));
	return resolve(event);
}
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user: {
			name: string;
		} | null
	}
}
```
**```global.d.ts```**
```ts
declare global {
	function getUser(sessionid: string | undefined): {
		name: string;
	};
}

export {};
```
**```src/hooks.server.ts```**
```ts
import type { Handle } from '@sveltejs/kit';
 
export const handle = (async ({ event, resolve }) => {
  event.locals.user = await getUser(event.cookies.get('sessionid'));
  return resolve(event);
}) satisfies Handle;
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **JavaScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user: {
			name: string;
		} | null
	}
}
```
**```src/routes/account/+page.server.js```**
```js
/** @type {import('./$types').PageServerLoad} */
export function load(event) {
	return {
		user: event.locals.user
	};
}

/** @type {import('./$types').Actions} */
export const actions = {
	logout: async (event) => {
		event.cookies.delete('sessionid');
		event.locals.user = null;
	}
};
```
#### **TypeScript**
**```ambient.d.ts```**
```ts
declare namespace App {
	interface Locals {
		user: {
			name: string;
		} | null
	}
}
```
**```src/routes/account/+page.server.ts```**
```ts
import type { PageServerLoad, Actions } from './$types';
 
export const load = ((event) => {
  return {
    user: event.locals.user
  };
}) satisfies PageServerLoad;
 
export const actions = {
  logout: async (event) => {
    event.cookies.delete('sessionid');
    event.locals.user = null;
  }
} satisfies Actions;
```
<!-- tabs:end -->

## Прогрессивное улучшение

В предыдущих разделах мы создали действие `/login`, которое [работает без JavaScript на стороне клиента](https://kryogenix.org/code/browser/everyonehasjs.html) - ни одного `fetch` в поле зрения. Это замечательно, но когда JavaScript _есть_, мы можем постепенно улучшать взаимодействие с формой, чтобы обеспечить лучший пользовательский опыт.

### use:enhance

Самый простой способ постепенного улучшения формы - добавить действие `use:enhance`:

**```src/routes/login/+page.svelte```**
```diff
 <script>
+	import { enhance } from '$app/forms';

	/** @type {import('./$types').ActionData} */
	export let form;
 </script>

+<form method="POST" use:enhance>
```

> Да, немного смущает, что действие `enhance` и `<form action>` оба называются "action". В этой документации много действий. Извините.

Без аргумента `use:enhance` будет эмулировать нативное поведение браузера, только без перезагрузки всей страницы. Это будет:

- обновление свойства `form`, `$page.form` и `$page.status` при успешном или недействительном ответе, но только если действие происходит на той же странице, с которой вы отправляете запрос. Так, например, если ваша форма выглядит как `<form action="/somewhere/else" ..>`, `form` и `$page` _не_ будут обновлены. Это происходит потому, что в случае с обычной формой вы будете перенаправлены на страницу, на которой происходит действие. Если вы хотите, чтобы они обновлялись в любом случае, используйте [`applyAction`](/20-core-concepts/30-form-actions?id=applyaction)
- сброс элемента `<form>` и аннулирование всех данных с помощью `invalidateAll` при успешном ответе
- вызов `goto` при перенаправлении ответа
- вывод ближайшей границы `+error` при возникновении ошибки
- [сброс фокуса](/40-best-practices/10-accessibility?id=Управление-фокусом) на соответствующем элементе

Чтобы настроить поведение, вы можете предоставить `SubmitFunction`, которая запускается непосредственно перед отправкой формы и (опционально) возвращает колбэк, который запускается вместе с `ActionResult`. Обратите внимание, что если вы возвращаете колбэк, то упомянутое выше поведение по умолчанию не срабатывает. Чтобы вернуть его, вызовите `update`.

```html
<form
	method="POST"
	use:enhance={({ form, data, action, cancel, submitter }) => {
		// `form` - это элемент `<form>`
		// `data` - это объект `FormData`
		// `action` - это URL, на который отправляется форма
		// `cancel()` предотвратит отправку
		// `submitter` - это `HTMLElement`, который вызвал отправку формы

		return async ({ result, update }) => {
			// `result` - это объект `ActionResult`
			// `update` - это функция, которая запускает логику, которая была бы запущена, если бы этот колбэк не был установлен
		};
	}}
>
```

Вы можете использовать эти функции для показа и скрытия пользовательского интерфейса загрузки и так далее.

### applyAction

Если вы предоставляете собственные колбэки, вам может понадобиться воспроизвести часть поведения `use:enhance` по умолчанию, например, показать ближайшую границу `+error`. В большинстве случаев достаточно вызвать `update`, переданный колбэку. Если вам нужно больше настроек, вы можете сделать это с помощью `applyAction`:

**```src/routes/login/+page.svelte```**
```diff
 <script>
+	import { enhance, applyAction } from '$app/forms';

	/** @type {import('./$types').ActionData} */
	export let form;
 </script>

 <form
	method="POST"
	use:enhance={({ form, data, action, cancel, submitter }) => {
		// `form` - это элемент `<form>`
		// `data` - это объект `FormData`
		// `action` - это URL, на который отправляется форма
		// `cancel()` предотвратит отправку
		// `submitter` - это `HTMLElement`, который вызвал отправку формы

		return async ({ result }) => {
			// `result` - это объект `ActionResult`
+			if (result.type === 'error') {
+				await applyAction(result);
+			}
		};
	}}
>
```

Поведение `applyAction(result)` зависит от `result.type`:

- `success`, `failure` — устанавливает `$page.status` в `result.status` и обновляет `form` и `$page.form` в `result.data` (независимо от того, откуда вы отправляете, в отличие от `update` из `enhance`)
- `redirect` — вызывает `goto(result.location)`.
- `error` — отображает ближайшую границу `+error` с `result.error`.

Во всех случаях, [фокус будет сброшен](/40-best-practices/10-accessibility?id=Управление-фокусом).

### Пользовательский прослушиватель событий

Мы также можем реализовать прогрессивное улучшение самостоятельно, без `use:enhance`, с помощью обычного слушателя событий на `<form>`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/login/+page.svelte```**
```svelte
<script>
	import { invalidateAll, goto } from '$app/navigation';
	import { applyAction, deserialize } from '$app/forms';

	/** @type {import('./$types').ActionData} */
	export let form;

	/** @type {any} */
	let error;

	async function handleSubmit(event) {
		const data = new FormData(this);

		const response = await fetch(this.action, {
			method: 'POST',
			body: data
		});

		/** @type {import('@sveltejs/kit').ActionResult} */
		const result = deserialize(await response.text());

		if (result.type === 'success') {
			// повторно запустить все функции `load` после успешного обновления
			await invalidateAll();
		}

		applyAction(result);
	}
</script>

<form method="POST" on:submit|preventDefault={handleSubmit}>
	<!-- сожержимое -->
</form>
```
#### **TypeScript**
**```src/routes/login/+page.svelte```**
```svelte
<script lang="ts">
	import { invalidateAll, goto } from '$app/navigation';
	import { applyAction, deserialize } from '$app/forms';

	import type { ActionData } from './$types';
	import type { ActionResult } from '@sveltejs/kit';

	export let form: ActionData;

	let error: any;

	async function handleSubmit(event) {
		const data = new FormData(this);

		const response = await fetch(this.action, {
			method: 'POST',
			body: data
		});

		const result: ActionResult = deserialize(await response.text());

		if (result.type === 'success') {
			// повторно запустить все функции `load` после успешного обновления
			await invalidateAll();
		}

		applyAction(result);
	}
</script>

<form method="POST" on:submit|preventDefault={handleSubmit}>
	<!-- сожержимое -->
</form>
```
<!-- tabs:end -->

Обратите внимание, что вам необходимо `десериализовать` ответ перед его дальнейшей обработкой с помощью соответствующего метода из `$app/forms`. `JSON.parse()` недостаточно, потому что действия формы - как и функции `load` - также поддерживают возврат объектов `Date` или `BigInt`.

Если у вас есть `+server.js` рядом с вашим `+page.server.js`, запросы `fetch` будут направляться туда по умолчанию. Чтобы направить `POST` на действие в `+page.server.js`, используйте пользовательский заголовок `x-sveltekit-action`:

```diff
const response = await fetch(this.action, {
	method: 'POST',
	body: data,
+	headers: {
+		'x-sveltekit-action': 'true'
+	}
});
```

## Альтернативы

Действия формы являются предпочтительным способом отправки данных на сервер, поскольку они могут быть постепенно усовершенствованы, но вы также можете использовать файлы [`+server.js`](/20-core-concepts/10-routing?id=server) для представления (например) JSON API. Вот как может выглядеть такое взаимодействие:

**```send-message/+page.svelte```**
```svelte
<script>
	function rerun() {
		fetch('/api/ci', {
			method: 'POST'
		});
	}
</script>

<button on:click={rerun}>Повторный запуск CI</button>
```

<!-- tabs:start -->
#### **JavaScript**
**```api/ci/+server.js```**
```js
/** @type {import('./$types').RequestHandler} */
export function POST() {
	// что-то сделать
}
```
#### **TypeScript**
**```api/ci/+server.ts```**
```ts
import type { RequestHandler } from './$types';
 
export const POST = (() => {
	// что-то сделать
}) satisfies RequestHandler;
```
<!-- tabs:end -->

## GET против POST

Как мы видели, для вызова действия формы необходимо использовать `method="POST"`.

Некоторые формы не нуждаются в отправке данных методом `POST` на сервер — например, поисковые вводы. Для них вы можете использовать `method="GET"` (или, эквивалентно, вообще не использовать `method`), и SvelteKit будет обращаться с ними как с элементами `<a>`, используя маршрутизатор на стороне клиента вместо полной навигации по странице:

```html
<form action="/search">
	<label>
		Поиск
		<input name="q">
	</label>
</form>
```

Отправка этой формы приведет к переходу по адресу `/search?q=...` и вызовет вашу функцию загрузки, но не вызовет действие. Как и для элементов `<a>`, вы можете установить атрибуты [`data-sveltekit-reload`](/30-advanced/30-link-options?id=data-sveltekit-reload), [`data-sveltekit-replacestate`](/30-advanced/30-link-options?id=data-sveltekit-replacestate), [`data-sveltekit-keepfocus`](/30-advanced/30-link-options?id=data-sveltekit-keepfocus) и [`data-sveltekit-noscroll`](/30-advanced/30-link-options?id=data-sveltekit-noscroll) на `<form>` для управления поведением маршрутизатора.

## Дальнейшее чтение

- [Учебник: Формы](https://learn.svelte.dev/tutorial/the-form-element)