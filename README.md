

|![icon](_media/favicon.png)|
|--|
![logo](_media/sveltekit.webp)

# Русская документация SvelteKit

(доступна также [русская документация Svelte](https://romkar.github.io/svelte-docs-rus/))

| БЫСТРО 🚀 | ВЕСЕЛО 🎉 | ГИБКО 🎛️|
|--|--|--|
| [SvelteKit](https://kit.svelte.dev/) работает на [Svelte](https://svelte.dev/) и [Vite](https://vitejs.dev/), скорость заложена в каждую щель: быстрая настройка, быстрая разработка, быстрая сборка, быстрая загрузка страниц, быстрая навигация. Мы уже говорили, что это быстро? | Больше не нужно тратить дни на выяснение конфигурации bundler, маршрутизации, SSR, CSP, TypeScript, настроек развертывания и прочих скучных вещей. Пишите код с радостью. | SPA? MPA? ССР? SSG? Проверьте. SvelteKit дает вам инструменты для достижения успеха, что бы вы ни создавали. И он работает везде, где есть JavaScript. |



## Фичи? У нас они есть

Смешивайте и сочетайте **пререндеренные** страницы для максимальной производительности с **динамическим рендерингом на стороне сервера (SSR)** для максимальной гибкости. Превратите свое приложение в **PWA** с клиентским рендерингом с помощью одной строки кода, для всего приложения или только одной страницы. Используйте доступную **маршрутизацию на стороне клиента (CSR)** с автоматической **предварительной загрузкой** для удобной, мгновенной навигации, которая не перезагружает всю страницу (и аналитику, и прочий мусор). Защитите своих пользователей с помощью автоматической **защиты CSRF** и простой в использовании конфигурации **политики безопасности контента (CSP)**. Храните свои секреты при себе с помощью расширенной обработки **переменных окружения**. Изящно и **безопасно** обрабатывайте ошибки. Загружайте данные **непосредственно из базы данных** и соединяйте бэкэнд с фронт-эндом с помощью **типо-безопасной** загрузки данных и встроенных **действий с формами**, которые работают как с JavaScript, так и без него. **Сочетать** с другими фреймворками маршрутизации на стороне клиента на одной странице. Добавляйте сервис воркеры для поддержки работы в **офлайне**. Генерируйте **AMP-совместимые** страницы, если это действительно необходимо. Создавайте сложные пользовательские интерфейсы с необычайно мощными **маршрутами на основе файловой системы**. Вложенные макеты? Да. Изучайте **веб-стандарты**, которые работают в разных средах. Интегрируйтесь с **Tailwind**, **Playwright**, **Vitest**, **Storybook** и, в общем, с чем угодно. Создавайте **библиотеки** и приложения. Развертывание в **любом месте** с помощью адаптеров.

SvelteKit - это фреймворк, который **растет вместе с вами**, что бы вы в итоге ни создали.


## Развёртывание в любом месте

Экспортируйте статические HTML-файлы. Запустите свой собственный сервер Node. Развёртывание кода на краю света. Если платформа работает на JavaScript, она работает на SvelteKit - в некоторых случаях с нулевой конфигурацией.

Хотите попробовать развернуть код на другой платформе? Поменяйте адаптер одной строкой кода.


---
**Переведено:** `обновлено 31.03.2023`

- [x] Начало работы
  - [x] [Введение](10-getting-started/10-introduction.md)
  - [x] [Создание проекта](10-getting-started/20-creating-a-project.md)
  - [x] [Структура проекта](10-getting-started/30-project-structure.md)
  - [x] [Веб-стандарты](10-getting-started/40-web-standards.md)
- [x] Основные понятия
  - [x] [Маршрутизация](20-core-concepts/10-routing.md)
  - [x] [Загрузка данных](20-core-concepts/20-load.md)
  - [x] [Действия формы](20-core-concepts/30-form-actions.md)
  - [x] [Параметры страницы](20-core-concepts/40-page-options.md)
  - [x] [Управление состоянием](20-core-concepts/50-state-management.md)
- [ ] Сборка и развёртывание
  - [ ] [Создание вашего приложения](25-build-and-deploy/10-building-your-app.md)
  - [ ] [Адаптеры](25-build-and-deploy/20-adapters.md)
  - [ ] [Развёртывание с нулевой конфигурацией](25-build-and-deploy/30-adapter-auto.md)
  - [ ] [Серверы Node](25-build-and-deploy/40-adapter-node.md)
  - [ ] [Создание статических сайтов](25-build-and-deploy/50-adapter-static.md)
  - [ ] [Страницы Cloudflare](25-build-and-deploy/60-adapter-cloudflare.md)
  - [ ] [Cloudflare обработчики](25-build-and-deploy/70-adapter-cloudflare-workers.md)
  - [ ] [Netlify](25-build-and-deploy/80-adapter-netlify.md)
  - [ ] [Vercel](25-build-and-deploy/90-adapter-vercel.md)
  - [ ] [Создание адаптеров](25-build-and-deploy/99-writing-adapters.md)
- [ ] Расширенные функции
  - [ ] [Расширенная маршрутизация](30-advanced/10-advanced-routing.md)
  - [ ] [Хуки](30-advanced/20-hooks.md)
  - [ ] [Ошибки](30-advanced/25-errors.md)
  - [ ] [Параметры ссылок](30-advanced/30-link-options.md)
  - [ ] [Сервис-воркеры](30-advanced/40-service-workers.md)
  - [ ] [Модули только для сервера](30-advanced/50-server-only-modules.md)
  - [ ] [Работа с ресурсами](30-advanced/60-assets.md)
  - [ ] [Снэпшоты](30-advanced/65-snapshots.md)
  - [ ] [Упаковка](30-advanced/70-packaging.md)
- [ ] Лучшие практики
  - [ ] [Доступность (a11y)](40-best-practices/10-accessibility.md)
  - [ ] [SEO](40-best-practices/20-seo.md)
- [ ] Ссылки
  - [ ] [Конфигурация](50-reference/10-configuration.md)
  - [ ] [Интерфейс командной строки CLI](50-reference/20-cli.md)
  - [ ] [Модули](50-reference/30-modules.md)
  - [ ] [Типы](50-reference/40-types.md)
- [ ] Приложение
  - [ ] [Интеграции](60-appendix/05-integrations.md)
  - [ ] [Миграция с Sapper](60-appendix/10-migrating.md)
  - [ ] [Дополнительные ссылки](60-appendix/20-additional-resources.md)
  - [x] [Глоссарий](60-appendix/30-glossary.md)
- [ ] FAQ
  - [ ] [FAQ](FAQ.md)
---



---
|![icon_docsify](_media/icon_docsify.svg)|
|--|
|[Работает на docsify](https://docsify.js.org)|