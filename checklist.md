# Чек-лист «Проверка внутренней оптимизации»

Проект: https://remont-noutbukov-nsk.ru/ · дата проверки: 20.11.2026.

- [x] **1. Валидность HTML5.** `index.html` проверен через W3C Nu HTML Checker (`https://validator.w3.org/nu/?out=json`): 0 ошибок и 0 предупреждений. Один `<h1>`, иерархия `h2 → h3` без пропусков, семантические теги `header`, `nav`, `main`, `section`, `article`, `figure`, `address`, `footer`.
- [x] **2. Метаданные.** `<title>` — 54 символа, `meta description` — 159, `meta keywords` — 10 фраз / 238 символов, canonical, robots (`index, follow`), Open Graph и Twitter Card. Один h1, ключи без переспама.
- [x] **3. robots.txt.** Файл лежит в корне проекта (`/robots.txt`). Синтаксис директив корректен: `User-agent`, `Allow`, `Disallow`, `Clean-param`, `Sitemap`. Основные страницы открыты, `/admin/`, `/temp/`, `/cache/` и URL с UTM/yclid/gclid закрыты; указан `Sitemap: https://remont-noutbukov-nsk.ru/sitemap.xml`. Три пункта проверки: (1) файл доступен по `example.com/robots.txt`; (2) нет опечаток в директивах; (3) политика индексации соответствует — основные разделы открыты, технические закрыты.
- [x] **4. sitemap.xml.** 7 URL: главная, услуги, цены, гарантии, контакты, бренды, срочный ремонт. Каждый URL с `lastmod`, `changefreq`, `priority`. Валидность XML проверена `xmllint --noout sitemap.xml` — ошибок нет. Для онлайн-проверки: `https://www.xml-sitemaps.com/validate-xml-sitemap.html`.
- [x] **5. Оптимизация изображений.** `hero-remont-noutbukov-novosibirsk.jpg` — 1200×675, 52 КБ, JPEG q82; `diagnostika-noutbuka.jpg` — 800×600, 30 КБ, JPEG q82. Формат JPG 75–85%, progressive, optimize. У обоих описательный `alt` с ключами без спама, заданы `width`/`height` (нет CLS), hero — `fetchpriority="high"` + preload, изображение ниже сгиба — `loading="lazy" decoding="async"`. `logo.svg` и `favicon.svg` — векторные, 1 КБ.
- [x] **6. Адаптивность и относительные размеры.** CSS использует `rem`, `em`, `clamp()`, `minmax()`, grid/flex; брейкпоинты `60em` и `40em`; изображения масштабируются через `aspect-ratio` и `object-fit`. Проверено в headless Firefox при 1440 и 390 px: горизонтального скролла нет, навигация и таблица цен адаптируются.
- [x] **7. Доступность.** Skip-link, видимый `:focus-visible`, `aria-label` у навигации и рейтинга, `aria-hidden` у декоративных элементов, подписи `label` у полей, `scope` у таблицы, `details/summary` работают с клавиатуры, контраст текста и кнопок проверен, есть `prefers-reduced-motion`.
- [x] **8. Производительность.** Один CSS-файл 15,9 КБ без внешних шрифтов и библиотек, нет блокирующего JS (единственный скрипт — JSON-LD), hero preload, lazy-loading ниже сгиба, системный стек шрифтов, статичная вёрстка без layout shift.
- [x] **9. Микроразметка Schema.org.** Microdata + JSON-LD: LocalBusiness/ProfessionalService, OfferCatalog, Service, Offer, PostalAddress, GeoCoordinates, OpeningHoursSpecification, AggregateRating, Review, WebSite, WebPage, BreadcrumbList, FAQPage. Проверка `validator.schema.org`: 0 ошибок, 0 предупреждений. Детали — в `schema-report.md`.
- [x] **10. Внутренняя перелинковка и интенты.** Все якорные ссылки ведут на существующие `id` (проверено скриптом: 0 битых якорей), 7 ссылок в футере соответствуют sitemap, у каждой секции один коммерческий интент, информационные запросы вынесены в FAQ, а не в коммерческие блоки.

## Технические проверки (команды)

| Проверка | Команда/инструмент | Результат |
|---|---|---|
| HTML5 | `validator.w3.org/nu` | 0 errors, 0 warnings |
| XML sitemap | `xmllint --noout sitemap.xml` | valid |
| JSON-LD синтаксис | `python3 -c "json.loads(...)"` | valid, 5 узлов `@graph` |
| Schema.org | `validator.schema.org` | 0 errors, 0 warnings |
| Якоря | regex-проверка `href="#..."` vs `id` | 0 битых |
| Изображения | PIL + `ls -l` | 2 JPG, 52 КБ и 30 КБ |
| Адаптивность | Firefox headless 1440×12000 и 390×900 | ок |

## Ограничения демо-проекта

- Телефон, адрес, реквизиты, отзывы и рейтинг — демонстрационные. Перед публикацией заменить на реальные; отзывы и `aggregateRating` должны быть подлинными.
- Форма заявки не отправляет данные (нет бэкенда) — помечена как демонстрационная.
- В sitemap указаны 7 запланированных страниц. В рамках работы реализована главная; остальные разделы создаются на следующем этапе по этой же структуре. Ссылки в футере ведут на будущие URL, поэтому до запуска разделов нужен либо их релиз, либо временный редирект.
- Частотность в `semantics.csv` — учебная оценка; перед публикацией перепроверить в Яндекс Вордстате по региону «Новосибирская область».
