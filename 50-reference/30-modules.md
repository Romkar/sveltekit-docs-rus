# Модули
---

SvelteKit делает ряд модулей доступными для вашего приложения.

> MODULES

---
## $app/navigation


---
## $app/stores

`import { getStores, navigating, page, updated } from '$app/stores';`

Хранилища на сервере являются контекстными - они добавляются в [context](https://svelte.dev/tutorial/context-api) вашего корневого компонента. Это означает, что страница является уникальной для каждого запроса, а не общей для нескольких запросов, обрабатываемых одним и тем же сервером одновременно.

Из-за этого вы должны подписаться на хранилище во время инициализации компонента (что происходит автоматически, если вы ссылаетесь на значение хранилища, например, как `$page`, в компоненте), прежде чем вы сможете их использовать.

В браузере нам не нужно беспокоиться об этом, и доступ к хранилищам можно получить из любого места. Код, который будет выполняться только в браузере, может ссылаться (или подписываться) на любое из этих хранилищ в любое время.

### getStores

A function that returns all of the contextual stores. On the server, this must be called during component initialization. Only use this if you need to defer store subscription until after the component has mounted, for some reason.

function getStores(): {
  navigating: typeof navigating;
  page: typeof page;
  updated: typeof updated;
};

### navigating

A readable store. When navigating starts, its value is a
Navigation
object with from, to, type and (if type === 'popstate') delta properties. When navigating finishes, its value reverts to null.
On the server, this store can only be subscribed to during component initialization. In the browser, it can be subscribed to at any time.
const navigating: Readable<Navigation | null>;

### page

A readable store whose value contains page data.
On the server, this store can only be subscribed to during component initialization. In the browser, it can be subscribed to at any time.
const page: Readable<Page>;

### updated

A readable store whose initial value is false. If version.pollInterval is a non-zero value, SvelteKit will poll for new versions of the app and update the store value to true when it detects one. updated.check() will force an immediate check, regardless of polling.
On the server, this store can only be subscribed to during component initialization. In the browser, it can be subscribed to at any time.
const updated: Readable<boolean> & { check(): Promise<boolean> };

---
## $lib

Это простой псевдоним `src/lib` или любого другого каталога, указанного в качестве `config.kit.files.lib`. Он позволяет получить доступ к общим компонентам и модулям утилит без такой ../../../../../../ бессмыслицы.

### `$lib/server`

A subdirectory of $lib. SvelteKit will prevent you from importing any modules in $lib/server into client-side code. See server-only modules.


## @sveltejs/kit/vite

`import { sveltekit } from '@sveltejs/kit/vite';`