# Веб-стандарты
---

Во всей этой документации вы увидите ссылки на стандартные [Web API](https://developer.mozilla.org/en-US/docs/Web/API), на основе которых построен SvelteKit. Вместо того чтобы изобретать колесо, мы _используем платформу_, что означает, что ваши существующие навыки веб-разработки применимы к SvelteKit. И наоборот, время, потраченное на изучение SvelteKit, поможет вам стать лучшим веб-разработчиком в других областях.

Эти API доступны во всех современных браузерах и во многих небраузерных средах, таких как Cloudflare Workers, Deno и Vercel Edge Functions. В процессе разработки и в [адаптерах](/25-build-and-deploy/20-adapters?id=Адаптеры) для сред на базе Node (включая AWS Lambda) они становятся доступны с помощью полифиллов, где это необходимо (на данный момент, то есть - Node быстро добавляет поддержку большего количества веб-стандартов).

В частности, вы освоите следующее:

## Fetch APIs

SvelteKit использует [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) для получения данных из сети. Он доступен в разделе [хуки](/30-advanced/20-hooks) и [серверная маршрутизация](/20-core-concepts/10-routing?id=server), а также в браузере.

> Специальная версия `fetch` доступна в функциях [`load`](/20-core-concepts/20-load), [серверные хуки](/30-advanced/20-hooks?id=server-hooks) и [API маршрутизации](/20-core-concepts/10-routing?id=server) для вызова конечных точек непосредственно во время рендеринга на стороне сервера, без вызова HTTP, с сохранением учетных данных. (Для выполнения мандатных запросов в коде на стороне сервера вне `load` необходимо явно передать заголовки `cookie` и/или `authorization`). Это также позволяет вам делать относительные запросы, в то время как для серверной `fetch` обычно требуется полный URL.

Помимо самого `fetch`, [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) включает следующие интерфейсы:

### Request

Экземпляр [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) доступен в разделе [хуки](/30-advanced/20-hooks) и [серверная маршрутизация](/20-core-concepts/10-routing?id=server) как `event.request`. Он содержит такие полезные методы, как `request.json()` и `request.formData()` для получения данных, которые были отправлены в конечную точку.

### Response

Экземпляр [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) возвращается из `await fetch(...)` и обработчиков в файлах `+server.js`. По сути, приложение SvelteKit - это машина для превращения `Request` в `Response`.

### Headers

Интерфейс [`Headers`](https://developer.mozilla.org/en-US/docs/Web/API/Headers) позволяет вам читать входящие `request.headers` и устанавливать исходящие `response.headers`:

<!-- tabs:start -->
#### **JavaScript**
**```src/routes/what-is-my-user-agent/+server.js```**
```js
import { json } from '@sveltejs/kit';

/** @type {import('./$types').RequestHandler} */
export function GET(event) {
	// записать в лог все заголовки
	console.log(...event.request.headers);

	return json({
		// получить определенный заголовок
		userAgent: event.request.headers.get('user-agent')
	});
}
```
#### **TypeScript**
**```src/routes/what-is-my-user-agent/+server.ts```**
```ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
 
export const GET = ((event) => {
  // записать в лог все заголовки
  console.log(...event.request.headers);
 
  return json({
    // получить определенный заголовок
    userAgent: event.request.headers.get('user-agent')
  });
}) satisfies RequestHandler;
```
<!-- tabs:end -->

## FormData

Когда вы имеете дело с отправкой форм на основе HTML, вы будете работать с объектами [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData).


<!-- tabs:start -->
#### **JavaScript**
**```src/routes/hello/+server.js```**
```js
import { json } from '@sveltejs/kit';

/** @type {import('./$types').RequestHandler} */
export async function POST(event) {
	const body = await event.request.formData();

	// записать в лог все поля
	console.log([...body]);

	return json({
		// получить значение определенного поля
		name: body.get('name') ?? 'world'
	});
}
```
#### **TypeScript**
**```src/routes/hello/+server.ts```**
```ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
 
export const POST = (async (event) => {
  const body = await event.request.formData();
 
  // записать в лог все поля
  console.log([...body]);
 
  return json({
    // получить значение определенного поля
    name: body.get('name') ?? 'world'
  });
}) satisfies RequestHandler;
```
<!-- tabs:end -->


## Stream APIs

В большинстве случаев ваши конечные точки будут возвращать полные данные, как в примере `userAgent` выше. Иногда вам может понадобиться вернуть ответ, который слишком велик, чтобы поместиться в памяти за один раз, или доставляется частями, и для этого платформа предоставляет [streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API) - [ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream), [WritableStream](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream) и [TransformStream](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream).

## URL APIs

URL представлены интерфейсом [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL), который включает такие полезные свойства, как `origin` и `pathname` (и, в браузере, `hash`). Этот интерфейс проявляется в различных местах - `event.url` в [хуки](/30-advanced/20-hooks) и [серверные маршруты](/20-core-concepts/10-routing?id=server), [`$page.url`](/50-reference/30-modules?id=appstores) в [pages](/20-core-concepts/10-routing?id=page), `from` и `to` в [`beforeNavigate` и `afterNavigate`](/50-reference/30-modules?id=appnavigation) и так далее.

### URLSearchParams

Где бы вы ни встретили URL, вы можете получить доступ к параметрам запроса через `url.searchParams`, который является экземпляром [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams):

**```ambient.d.ts```**
```ts
declare global {
	const url: URL;
}

export {};
```

**```index.js```**
```js
const foo = url.searchParams.get('foo');
```

## Web Crypto

[Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) доступен через `crypto` глобально. Он используется для внутренних заголовков [Content Security Policy](/50-reference/10-configuration?id=csp), но вы также можете использовать его для таких вещей, как генерация UUID:

```js
const uuid = crypto.randomUUID();
```
