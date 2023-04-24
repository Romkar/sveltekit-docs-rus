# Создание статических сайтов
---

Чтобы использовать SvelteKit в качестве генератора статических сайтов 'static site generator' (SSG), используйте [`adapter-static`](https://github.com/sveltejs/kit/tree/master/packages/adapter-static).

Это приведет к пререндеру всего сайта как коллекции статических файлов. Если вы хотите пререндерить только некоторые страницы, вам нужно использовать другой адаптер вместе с [опцией `prerender`](/20-core-concepts/40-page-options?id=пререндер).

## Использование

Установите с помощью `npm i -D @sveltejs/adapter-static`, затем добавьте адаптер в ваш файл `svelte.config.js`:

**```svelte.config.js```**
```js
import adapter from '@sveltejs/adapter-static';

export default {
	kit: {
		adapter: adapter({
			// Показаны параметры по умолчанию. На некоторых платформах
			// эти параметры устанавливаются автоматически — см. ниже
			pages: 'build',
			assets: 'build',
			fallback: null,
			precompress: false,
			strict: true
		})
	}
};
```

...и добавьте опцию [`prerender`](/20-core-concepts/40-page-options?id=пререндер) в ваш корневой макет:

**```src/routes/+layout.js```**
```js
// Параметр может быть 'false', если вы используете резервную страницу (например, в режиме SPA).
export const prerender = true;
```

> Вы должны убедиться, что опция SvelteKit [`trailingSlash`](/20-core-concepts/40-page-options?id=trailingslash) установлена соответствующим образом для вашего окружения. Если ваш хост не выводит `/a.html` при получении запроса на `/a`, то вам нужно установить `trailingSlash: 'always'` для вывода `/a/index.html` вместо `/a.html`.

## Поддержка нулевой конфигурации

Некоторые платформы имеют поддержку нулевой конфигурации (в будущем появится еще больше):

- [Vercel](https://vercel.com)

На этих платформах следует опустить опции адаптера, чтобы `adapter-static` мог обеспечить оптимальную конфигурацию:

**```svelte.config.js```**
```diff
export default {
	kit: {
-		adapter: adapter({...}),
+		adapter: adapter(),
		}
	}
};
```

## Параметры

### pages

Каталог для записи пререндеренных страниц. По умолчанию это `build`.

### assets

Каталог для записи статических ресурсов (содержимое каталога `static`, плюс клиентские JS и CSS, сгенерированные SvelteKit). Обычно это должно быть то же самое, что и `pages`, и по умолчанию оно будет равно значению `pages`, но в редких случаях вам может понадобиться выводить страницы и ресурсы в отдельные места.

### fallback

Укажите резервную страницу для [SPA-режима](/25-build-and-deploy/55-single-page-apps), например, `index.html` или `200.html` или `404.html`.

### precompress

Если значение параметра `true`, предварительно сжимает файлы с помощью brotli и gzip. В результате будут созданы файлы `.br` и `.gz`.

### strict

По умолчанию `adapter-static` проверяет, что либо все страницы и конечные точки (если они есть) вашего приложения были пререндерены, либо у вас установлена опция `fallback`. Эта проверка существует для того, чтобы вы случайно не опубликовали приложение, некоторые части которого недоступны, потому что они не содержатся в конечном выводе. Если вы знаете, что это нормально (например, когда определенная страница существует только условно), вы можете установить `strict` в `false`, чтобы отключить эту проверку.

## GitHub Pages

При сборке для GitHub Pages убедитесь, что [`paths.base`](/50-reference/10-configuration?id=paths) соответствует имени вашего репозитория, поскольку сайт будет обслуживаться из <https://ваше-имя.github.io/название-вашего-репозитария>, а не из корня.

Вам придется запретить предоставляемому GitHub Jekyll управлять вашим сайтом, поместив пустой файл `.nojekyll` в папку `static`.

Конфигурация для GitHub Pages может выглядеть следующим образом:

**```svelte.config.js```**
```js
import adapter from '@sveltejs/adapter-static';

const dev = process.argv.includes('dev');

/** @type {import('@sveltejs/kit').Config} */
const config = {
	kit: {
		adapter: adapter(),
		paths: {
			base: dev ? '' : process.env.BASE_PATH,
		}
	}
};
```

Вы можете использовать действия GitHub для автоматического развертывания вашего сайта на GitHub Pages при внесении изменений. Вот пример рабочего процесса:

**```.github/workflows/deploy.yml```**
```yaml
name: Развертывание на страницах GitHub

on:
  push:
    branches: 'main'

jobs:
  build_site:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      # Если вы используете pnpm, добавьте этот шаг, а затем измените команды и ключ кэша ниже, чтобы использовать `pnpm`.
      # - name: Установка pnpm
      #   uses: pnpm/action-setup@v2
      #   with:
      #     version: 8

      - name: Установка Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm

      - name: Установка зависимостей
        run: npm install

      - name: build
        env:
          BASE_PATH: '/название-вашего-репозитария'
        run: |
          npm run build
          touch build/.nojekyll

      - name: Загрузка Artifacts
        uses: actions/upload-pages-artifact@v1
        with:
          # это должно соответствовать параметру `pages` в ваших опциях статического адаптера
          path: 'build/'

  deploy:
    needs: build_site
    runs-on: ubuntu-latest

    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    
    steps:
      - name: Развёртывание
        id: deployment
        uses: actions/deploy-pages@v1
```
