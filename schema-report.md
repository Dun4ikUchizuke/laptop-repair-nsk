# Часть 3. Микроразметка Schema.org

Проект: https://remont-noutbukov-nsk.ru/ · дата: 20.11.2026.

## 1. Выбранные типы и сущности страницы

| Сущность страницы | Тип Schema.org | Где размещено | Зачем |
|---|---|---|---|
| Сервисный центр | `LocalBusiness` + `ProfessionalService` | JSON-LD `@graph`, Microdata на `<body>` | Название, телефон, адрес, гео, часы, цена, рейтинг — база для локальной выдачи и knowledge panel |
| Услуги | `OfferCatalog` → `Service` → `Offer` | JSON-LD и Microdata в `#services` | Показать 7 услуг, провайдера, город и минимальную цену |
| Адрес | `PostalAddress` | JSON-LD и Microdata в `#contacts` | Корректный NAP для локального SEO |
| Координаты | `GeoCoordinates` | JSON-LD и Microdata | Привязка к карте, локальный ранжирующий сигнал |
| Часы работы | `OpeningHoursSpecification` | JSON-LD и Microdata (`openingHours`) | Показать график, снизить нецелевые звонки |
| Отзывы и рейтинг | `AggregateRating`, `Review`, `Rating` | JSON-LD и Microdata в `#reviews` | Социальное доказательство; данные должны быть реальными |
| Сайт и страница | `WebSite`, `WebPage` | JSON-LD | Связка сайта, страницы и организации через `@id` |
| Хлебные крошки | `BreadcrumbList` | JSON-LD | Навигационная цепочка в сниппете |
| Вопросы и ответы | `FAQPage` → `Question` → `Answer` | JSON-LD и Microdata в `#faq` | Понимание вопроса-ответа; rich result не гарантирован |
| Изображение | `ImageObject` | JSON-LD (`primaryImageOfPage`) | Главное изображение страницы |

`ComputerRepair` в Schema.org нет, поэтому выбран корректный подтип `ProfessionalService` (наследник `LocalBusiness`); специализация раскрыта через `knowsAbout`, `serviceType` и `description`. Для товарного магазина подошёл бы `ComputerStore`, но здесь продаётся услуга, а не товар.

## 2. Microdata: что внедрено

- `<body itemscope itemtype="https://schema.org/LocalBusiness">` — корневая сущность.
- `itemprop="name"` на видимом названии, `telephone` на телефоне, `email` на адресе почты, `address` → `PostalAddress` с `postalCode`, `addressLocality`, `streetAddress`.
- `geo` → `GeoCoordinates` с `latitude`/`longitude`, `openingHours`, `priceRange`, `areaServed` (в JSON-LD).
- `#services` → `OfferCatalog`, каждая карточка → `Service` с `name`, `description`, `serviceType`, вложенным `Offer` (`price`, `priceCurrency`).
- `#reviews` → `aggregateRating` (`ratingValue`, `reviewCount`, `bestRating`) и три `Review` с `author`, `datePublished`, `reviewBody`, `reviewRating`.
- `#faq` → `FAQPage`, пять пар `Question` → `acceptedAnswer` → `Answer` → `text`.

Microdata привязана к видимому контенту: если текст на странице меняется, разметку нужно править в том же месте. Это плюс для синхронности и минус для сопровождения.

## 3. JSON-LD: что внедрено

В конце `<body>` добавлен один `<script type="application/ld+json">` с `@graph` из пяти узлов:

1. `LocalBusiness` + `ProfessionalService` (`@id .../#business`) — контакты, адрес, гео, часы, `areaServed`, `sameAs`, `knowsAbout`, `aggregateRating`, 3 `Review`, вложенный `OfferCatalog` из 7 `Service` с `Offer` и `PriceSpecification.minPrice`.
2. `WebSite` (`#website`) — издатель, язык.
3. `WebPage` (`#webpage`) — title, description, `isPartOf`, `about`, `primaryImageOfPage`, `breadcrumb`.
4. `BreadcrumbList` (`#breadcrumb`) — Главная → Ремонт ноутбуков в Новосибирске.
5. `FAQPage` (`#faq`) — 5 вопросов и ответов, совпадающих с видимым FAQ.

Связка через `@id` (`provider`, `publisher`, `about`, `isPartOf`, `breadcrumb`) позволяет поисковику собрать единый граф, а не набор разрозненных объектов. Цены переданы через `priceSpecification.minPrice`, потому что на сайте указано «от»: так честнее, чем фиксированный `price`, который не соответствует реальной смете.

## 4. Проверка валидности

Инструмент: **Schema Markup Validator** — https://validator.schema.org (официальный валидатор Schema.org, тот же движок используется в Google Rich Results Test).

Команда:

```bash
curl -s -X POST 'https://validator.schema.org/validate' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data-urlencode html@index.html
```

Результат (фрагмент ответа, служебный префикс `)]}'` опущен):

```json
{
  "isRendered": false,
  "numObjects": 4,
  "errors": [],
  "totalNumErrors": 0,
  "totalNumWarnings": 0
}
```

**Статус: валидно.** Ошибок — 0, предупреждений — 0. Отдельно `index.html` проверен в W3C Nu HTML Checker: 0 сообщений (ошибок и предупреждений). JSON-LD дополнительно распарсен локально (`json.loads`) — синтаксис корректен.

Исправленные по ходу проверки замечания:

- `aria-label` на `<span>` со звёздами рейтинга — добавлен `role="img"`, иначе W3C выдавал ошибку ARIA.
- `itemprop="email"` и `itemprop="telephone"` стояли на `<a href="mailto:...">`/`tel:...`; микроформат подставлял URL вместо текста, валидатор Schema.org выдавал `TYPE_CONVERSION_FAILED`. Значения перенесены внутрь `<span itemprop="...">`, ошибка ушла.

## 5. Какие бонусы для сниппета ожидаем

- **Локальная выдача и карты.** `LocalBusiness` + NAP + `geo` + часы помогают Яндексу и Google связать сайт с организацией, повышают шансы на локальный блок и корректную карточку.
- **Понимание услуг и цен.** `OfferCatalog`/`Service`/`Offer` дают поисковику структурированный список услуг и диапазон цен — полезно для тематического соответствия страницы.
- **Хлебные крошки.** `BreadcrumbList` может отобразиться в сниппете вместо голого URL.
- **FAQ и отзывы — не гарантированы.** С 2023 года Google показывает FAQ rich results в основном государственным и медицинским сайтам, а с 2019-го не показывает звёзды для self-serving отзывов `LocalBusiness`. Разметка всё равно помогает понять контент и может использоваться Яндексом, но обещать расширенный сниппет со звёздами нельзя. Это ограничение поисковиков, а не ошибка разметки.

## 6. Ограничения разметки

- Телефон, адрес, реквизиты, отзывы и рейтинг — демонстрационные. Для реального сайта нужны подлинные NAP и отзывы; накрученный `aggregateRating` нарушает правила поисковиков.
- Цена указана как `minPrice`, потому что итог зависит от модели и диагностики. Точную цену в разметке без привязки к модели ставить нельзя.
- `areaServed` ограничен городом и областью; районы перечислены только в тексте. Полигоны обслуживания (`GeoShape`) не заданы — для выездного сервиса это возможное развитие.
- `hasMap` ведёт на общий раздел 2ГИС, а не на конкретную карточку: демо-адрес. Перед публикацией заменить на реальную карточку организации.
- Нет исключений в графике (`specialOpeningHoursSpecification`) для праздников и нет `sameAs` на реальные соцпрофили.
- Microdata и JSON-LD описывают одну и ту же организацию и **дублируют данные**. Для продакшена нужно оставить один формат — JSON-LD; дублирование усложняет поддержку и может дать расхождения при обновлении. В учебной работе показаны оба формата по требованию задания.

## 7. Microdata и JSON-LD: сравнение

| Критерий | Microdata | JSON-LD |
|---|---|---|
| Где живёт | Внутри видимого HTML (`itemscope`, `itemtype`, `itemprop`) | Отдельный `<script type="application/ld+json">` |
| Читаемость | Разметка смешана с вёрсткой, сложно читать на большой странице | Чистый JSON, видно структуру графа |
| Связывание сущностей | Через вложенность и `itemref`, ограниченно | Через `@id` и ссылки — удобно для сложных графов |
| Поддержка | Google, Яндекс, Bing | Google (рекомендует), Яндекс, Bing |
| Риск рассинхрона | Низкий: разметка на тех же элементах, что и текст | Средний: нужно следить, чтобы JSON совпадал с контентом |
| Дублирование | Легко случайно продублировать при копировании блоков | Один блок на страницу, дубли видны сразу |
| Итог | Хорош для простых карточек и устаревших CMS | **Рекомендуемый формат** для этого проекта |

Оба подхода поддерживаются основными поисковиками. Для страницы с услугами, ценами, отзывами и FAQ JSON-LD выигрывает: он не ломает вёрстку, масштабируется через `@graph` и проще в сопровождении. Microdata оставлена как демонстрация второго подхода и как fallback, но в продакшене её следует убрать, чтобы не дублировать сущности.
