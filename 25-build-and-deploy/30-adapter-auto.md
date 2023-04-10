# Развёртывание с нулевой конфигурацией
---

Когда вы создаете новый проект SvelteKit с помощью `npm create svelte@latest`, он по умолчанию устанавливает [`adapter-auto`](https://github.com/sveltejs/kit/tree/master/packages/adapter-auto). Этот адаптер автоматически устанавливается и использует правильный адаптер для поддерживаемых сред при развертывании:

- [`@sveltejs/adapter-cloudflare`](/25-build-and-deploy/60-adapter-cloudflare) для [Cloudflare Pages](https://developers.cloudflare.com/pages/)
- [`@sveltejs/adapter-netlify`](/25-build-and-deploy/80-adapter-netlify) для [Netlify](https://netlify.com/)
- [`@sveltejs/adapter-vercel`](/25-build-and-deploy/90-adapter-vercel) для [Vercel](https://vercel.com/)
- [`svelte-adapter-azure-swa`](https://github.com/geoffrich/svelte-adapter-azure-swa) для [Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/)

Рекомендуется установить соответствующий адаптер в `devDependencies`, как только вы определились с целевым окружением, поскольку это добавит адаптер в ваш lockfile и немного улучшит время установки при непрерывной интеграции (CI - Continuous Integration).

## Конфигурация для конкретного окружения

Чтобы добавить опции конфигурации, такие как `{ edge: true }` в [`adapter-vercel`](/25-build-and-deploy/90-adapter-vercel) и [`adapter-netlify`](/25-build-and-deploy/80-adapter-netlify), вы должны установить базовый адаптер - `adapter-auto` не принимает никаких опций.

## Добавление адаптеров сообщества

Вы можете добавить поддержку нулевой конфигурации для дополнительных адаптеров, отредактировав файл [adapters.js](https://github.com/sveltejs/kit/blob/master/packages/adapter-auto/adapters.js) и открыв запрос на изменение кода (pull request).