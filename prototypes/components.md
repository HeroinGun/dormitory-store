# ОбщМаркет — Каталог UI-компонентов

Справочник переиспользуемых компонентов. Основан на макетах гостевых экранов, но применим ко всем ролям.

**Как пользоваться:**

- Перед созданием нового экрана — просмотрите этот файл. Скорее всего 80% нужных элементов уже описаны.
- Копируйте HTML-снипет, подставляйте данные через Blade (`{{ $variable }}`).
- Не создавайте новые CSS-классы, если можно использовать существующие.
- Если действительно нужен новый компонент — добавьте его в этот файл перед тем, как использовать на экране.

**Структура раздела:**

1. Дизайн-токены
2. Типографика
3. Layout-примитивы (container, site-header)
4. Атомы (button, tag, badge)
5. Молекулы (form field, notice, author-card)
6. Органайзеры (card, filters, gallery)
7. Шаблоны страниц (feed, listing, auth, message)

---

## 1. Дизайн-токены

Все значения определены в `styles.css` как CSS-переменные. **Никогда не используйте hex-коды напрямую** — только через переменные. Это позволит при необходимости сменить всю палитру одной правкой.

### Цвета

| Переменная | Значение | Когда использовать |
|---|---|---|
| `--color-blue-700` | #1e4a8a | Основные CTA, ссылки, акценты |
| `--color-blue-800` | #13367a | Hover-состояние синих элементов, тёмный блок на auth-страницах |
| `--color-blue-900` | #0f2a5c | Самые тёмные тексты (заголовок ленты) |
| `--color-blue-500` | #4775c2 | Декоративные акценты в заголовках |
| `--color-blue-100` | #dde6f5 | Аватары, иконки-кружки, thumbnails-подложки |
| `--color-blue-50`  | #eef3fb | Подложки бейджей, focus-ring у инпутов |
| `--color-paper` | #fbfaf6 | Карточки, поля формы фон |
| `--color-paper-2` | #f4f2eb | Фон страницы, secondary-поверхности |
| `--color-white` | #ffffff | Чистые карточки, site-header |
| `--color-ink` | #1a1a1a | Основной текст |
| `--color-ink-soft` | #555 | Описания, подзаголовки |
| `--color-ink-fade` | #888 | Метаданные, плейсхолдеры |
| `--color-rule` | #e4e0d5 | Границы карточек, разделители |
| `--color-rule-strong` | #c9c3b3 | Границы инпутов |
| `--color-red`, `--color-red-bg` | — | Ошибки, опасные действия, блокировка |
| `--color-amber`, `--color-amber-bg` | — | Предупреждения, pending-состояния |
| `--color-green`, `--color-green-bg` | — | Успех, active-статус |

### Шрифты

| Переменная | Семейство | Использование |
|---|---|---|
| `--font-display` | Fraunces (serif) | Заголовки страниц, цены, логотип |
| `--font-body` | Inter (sans-serif) | UI, кнопки, формы, основной текст |
| `--font-mono` | JetBrains Mono | Коды, метаданные, даты, URL, метки категорий |

### Размеры

Используйте шкалу `--space-1` (4px) ... `--space-12` (48px). Для радиусов — `--radius` (3px) для инпутов, `--radius-lg` (6px) для карточек.

---

## 2. Типографика

### Заголовок страницы

```html
<h1 class="page-feed__title">
    Объявления <em>студентов общежития</em>
</h1>
```

`<em>` внутри заголовков используется не как курсив, а как **цветовой акцент** — задаётся через CSS. Не ставьте `<em>` ради семантического ударения — только для визуального выделения части заголовка.

### Заголовок раздела внутри контента

```html
<h3 class="listing-description__heading">Описание</h3>
```

Моноширинный, маленький, в верхнем регистре, с разрядкой. Используется как «eyebrow» — ориентир в длинной странице.

### Eyebrow (надзаголовок)

Утилита `.u-eyebrow` для стилизации одного span или div. Например:

```html
<div class="u-eyebrow">Регистрация</div>
<h1 class="auth-form__heading">Создать аккаунт</h1>
```

---

## 3. Layout-примитивы

### Site header (общая шапка)

**Файл:** `resources/views/partials/header.blade.php`

```html
<header class="site-header">
    <div class="site-header__inner">
        <a href="{{ route('feed') }}" class="site-logo">
            Общ<span class="site-logo__dot">.</span>Маркет
        </a>

        <form action="{{ route('feed') }}" method="GET" class="site-search">
            <span class="site-search__icon">⌕</span>
            <input type="search" name="q" placeholder="Поиск по объявлениям..." value="{{ request('q') }}">
        </form>

        <div class="site-actions">
            @guest
                <a href="{{ route('login') }}">Войти</a>
                <a href="{{ route('register') }}" class="btn btn--primary">Регистрация</a>
            @else
                {{-- Для авторизованных: колокольчик, аватар, dropdown --}}
                {{-- Будет описано при проработке экранов авторизованных --}}
            @endguest
        </div>
    </div>
</header>
```

**Правила:**

- Всегда используйте `@guest / @else` для переключения правой части — это единственное место, где шапка различается для ролей.
- Поиск — форма с `method="GET"`, работает без JS. Параметр `q` в query string.
- Логотип ведёт только на `/` (маршрут `feed`), независимо от состояния авторизации.

### Containers

Три варианта ширины:

```html
<div class="container">...</div>         <!-- 1200px max — дефолт для страниц -->
<div class="container-wide">...</div>    <!-- 1280px max — лента, широкие экраны -->
<div class="container-narrow">...</div>  <!-- 720px max — формы, тексты -->
```

---

## 4. Атомы

### Кнопки

```html
<!-- Основная кнопка (primary CTA на экране) -->
<button type="submit" class="btn btn--primary">Войти</button>
<a href="{{ route('register') }}" class="btn btn--primary">Регистрация</a>

<!-- Вторичная (ghost) -->
<button type="button" class="btn btn--ghost">Отмена</button>

<!-- Опасное действие (ghost + красный) — жалоба, удаление -->
<button class="btn btn--danger-ghost">⚑</button>

<!-- На всю ширину -->
<button class="btn btn--primary btn--block">Зарегистрироваться</button>
```

**Правила:**

- На одном экране только **одна** `--primary` кнопка. Если их больше — пересмотрите иерархию.
- Иконки внутри кнопки — через `<span>` или inline `<svg>` с `gap`, который уже задан.
- Текст кнопок — **глагол в повелительном наклонении** (Войти, Отправить, Подтвердить). Не «Вход», не «Отправка».

### Tag / Category

```html
<span class="tag">Учёба · Продам</span>
<span class="tag">Электроника</span>
```

Используется на странице объявления для отображения категории и типа. Моноширинный шрифт, верхний регистр.

### Card badge (бейдж на фото карточки)

```html
<span class="card__badge">Продам</span>
```

Располагается absolutely внутри `.card__photo`. Значения: «Продам», «Аренда», «Услуга». Маппинг из БД:

| `listings.listing_type` | Текст бейджа |
|---|---|
| `sell` | Продам |
| `rent` | Аренда |
| `service_offer` | Услуга |

---

## 5. Молекулы

### Form field (поле формы с лейблом и подсказкой)

```html
<div class="field">
    <label for="email" class="field__label">Корпоративный email</label>
    <input id="email" name="email" type="email" placeholder="ivanov@university.edu" value="{{ old('email') }}">
    <span class="field__hint">Регистрация возможна только с email вашего учебного заведения</span>
</div>
```

С ошибкой валидации:

```html
<div class="field">
    <label for="email" class="field__label">Email</label>
    <input id="email" name="email" type="email" value="{{ old('email') }}">
    @error('email')
        <span class="field__hint field__hint--error">{{ $message }}</span>
    @enderror
</div>
```

Лейбл в одну строку со ссылкой (например, «Забыли пароль?»):

```html
<label for="password" class="field__label field__label--row">
    <span>Пароль</span>
    <a href="{{ route('password.reset') }}">Забыли?</a>
</label>
```

### Notice / Callout

Четыре варианта: info, warn, danger, success.

```html
<div class="notice notice--info">
    <span class="notice__icon">ⓘ</span>
    <span>Чтобы написать продавцу, войдите в систему или зарегистрируйтесь.</span>
</div>

<div class="notice notice--warn">
    <span class="notice__icon">⚠</span>
    <span>Это объявление истекло. Подтвердите актуальность в течение 48 часов.</span>
</div>

<div class="notice notice--danger">
    <span class="notice__icon">⊘</span>
    <span>Объявление было заблокировано модератором.</span>
</div>

<div class="notice notice--success">
    <span class="notice__icon">✓</span>
    <span>Объявление успешно опубликовано, активно до 29.04.2026.</span>
</div>
```

**Правила:**

- `notice--danger` — для пост-событий (заблокировано, отклонено), не для предупреждений.
- `notice--warn` — для предупреждений, требующих действия (истёк срок, email не верифицирован).
- На экране одновременно **максимум один** notice сверху основного контента.

### Author card

```html
<div class="author-card">
    <div class="author-card__avatar">{{ mb_strtoupper(mb_substr($listing->author->full_name, 0, 2)) }}</div>
    <div class="author-card__info">
        <div class="author-card__name">{{ $listing->author->full_name }}</div>
        <div class="author-card__meta">
            НА ОБЩМАРКЕТЕ С {{ $listing->author->created_at->isoFormat('MMMM YYYY') }} ·
            {{ $listing->author->activeListings()->count() }} АКТИВНЫХ ОБЪЯВЛЕНИЙ
        </div>
    </div>
</div>
```

В MVP аватар — инициалы из ФИО (первые 2 буквы). Когда появится реальная загрузка аватарки, заменить на `<img>`.

---

## 6. Органайзеры

### Card (карточка объявления)

```html
<a href="{{ route('listing.show', $listing->id) }}" class="card">
    <div class="card__photo">
        <span class="card__badge">{{ $typeLabel[$listing->listing_type] }}</span>
        <img src="{{ $listing->photos->first()->thumbUrl() }}" alt="{{ $listing->title }}">
    </div>
    <div class="card__body">
        <div class="card__title">{{ $listing->title }}</div>
        @if($listing->price)
            <div class="card__price">{{ number_format($listing->price, 0, '.', ' ') }} ₽</div>
        @else
            <div class="card__price card__price--free">Договорная</div>
        @endif
        <div class="card__meta">
            <span>{{ $listing->category->name }}</span>
            <span>{{ $listing->created_at->diffForHumans() }}</span>
        </div>
    </div>
</a>
```

**Правила:**

- `.card` **всегда** — это `<a>`, не `<div>`. Это улучшает доступность и позволяет открывать в новой вкладке через Cmd+click.
- Если карточка используется на экране «Мои объявления» с кнопками действий — возможно, стоит добавить отдельный модификатор `.card--owner` с дополнительной панелью внизу. Добавим при проработке этого экрана.

### Filters (сайдбар фильтров)

```html
<aside class="filters">
    <h3 class="filters__title">Фильтры</h3>

    <form method="GET" action="{{ route('feed') }}">
        <div class="filters__group">
            <span class="filters__legend">Категория</span>
            @foreach($categories as $category)
                <label class="filters__option">
                    <input type="checkbox" name="categories[]" value="{{ $category->id }}"
                           {{ in_array($category->id, $selectedCategories) ? 'checked' : '' }}>
                    {{ $category->name }}
                    <span class="filters__option-count">{{ $category->listings_count }}</span>
                </label>
            @endforeach
        </div>

        <div class="filters__group">
            <span class="filters__legend">Тип</span>
            <label class="filters__option">
                <input type="radio" name="type" value="" {{ !request('type') ? 'checked' : '' }}>
                Все
            </label>
            <label class="filters__option">
                <input type="radio" name="type" value="sell" {{ request('type') === 'sell' ? 'checked' : '' }}>
                Продам
            </label>
            {{-- и так далее --}}
        </div>

        <div class="filters__group">
            <span class="filters__legend">Цена, ₽</span>
            <div class="filters__price-range">
                <input type="number" name="price_from" placeholder="От" value="{{ request('price_from') }}">
                <input type="number" name="price_to" placeholder="До" value="{{ request('price_to') }}">
            </div>
        </div>

        <button type="submit" class="btn btn--primary btn--block">Применить</button>
        <a href="{{ route('feed') }}" class="filters__reset">Сбросить фильтры</a>
    </form>
</aside>
```

**Важно:** в MVP фильтрация выполняется через submit формы (GET-запрос). Когда появится Alpine.js для инкрементальной фильтрации без перезагрузки — добавим `@change` или debounced submit.

На мобильных (≤900px) фильтры схлопываются в кнопку, которая открывает drawer-модалку. Разметка та же, стиль модалки будет описан позже.

### Gallery (галерея фото на странице объявления)

```html
<div class="gallery">
    <div class="gallery__hero">
        <img src="{{ $currentPhoto->fullUrl() }}" alt="{{ $listing->title }}">
    </div>
    <div class="gallery__thumbs">
        @foreach($listing->photos as $index => $photo)
            <div class="gallery__thumb {{ $loop->first ? 'gallery__thumb--active' : '' }}"
                 x-on:click="setActive({{ $index }})">
                <img src="{{ $photo->thumbUrl() }}" alt="">
            </div>
        @endforeach
    </div>
</div>
```

Переключение hero-фото при клике на thumb — через Alpine.js (`x-data`, `x-on:click`). Пример Alpine-контроллера — в `routes.md`, раздел «AJAX/Alpine».

---

## 7. Шаблоны страниц

### Feed page

**Маршрут:** `GET /` (для гостя и авторизованного одинаково).
**Файл:** `resources/views/feed.blade.php`

```html
@extends('layouts.app')

@section('content')
    <div class="page-feed__hero">
        <div class="page-feed__hero-inner">
            <h1 class="page-feed__title">
                Объявления <em>студентов общежития</em>
            </h1>
            <div class="page-feed__stats">
                <strong>{{ $totalCount }}</strong> активных объявлений
            </div>
        </div>
    </div>

    <div class="page-feed__body">
        @include('partials.filters', ['categories' => $categories])

        <main>
            <div class="page-feed__toolbar">
                <div class="page-feed__toolbar-count">
                    Показано <strong>{{ $listings->count() }}</strong> из <strong>{{ $totalCount }}</strong>
                </div>
                <div class="page-feed__toolbar-sort">
                    <label for="sort">Сортировать:</label>
                    <select id="sort" name="sort" onchange="this.form.submit()">
                        <option value="newest">Новые сначала</option>
                        <option value="oldest">Старые сначала</option>
                        <option value="cheap">Сначала дешёвые</option>
                        <option value="expensive">Сначала дорогие</option>
                    </select>
                </div>
            </div>

            <div class="page-feed__grid">
                @foreach($listings as $listing)
                    @include('partials.listing-card', ['listing' => $listing])
                @endforeach
            </div>

            @if($hasMore)
                <div class="page-feed__load-more">
                    <button type="button" class="btn btn--ghost"
                            x-data="loadMore({{ $nextPage }})" x-on:click="load()">
                        Показать ещё 12
                    </button>
                </div>
            @endif
        </main>
    </div>
@endsection
```

### Listing detail page

**Маршрут:** `GET /listings/{id}`.
**Файл:** `resources/views/listings/show.blade.php`

Ключевое отличие гостя от авторизованного — блок actions. Используйте `@guest / @else` как описано выше в `site-header`.

```html
<div class="listing-info__actions">
    @guest
        <a href="{{ route('login', ['return' => url()->current()]) }}" class="btn btn--primary">
            ✉ Написать автору
        </a>
        <a href="{{ route('login', ['return' => url()->current()]) }}" class="btn btn--danger-ghost">
            ⚑
        </a>
    @else
        @if($listing->author_id !== auth()->id())
            <a href="{{ route('chat.openFor', $listing->id) }}" class="btn btn--primary">
                ✉ Написать автору
            </a>
            <button type="button" class="btn btn--danger-ghost"
                    x-on:click="$dispatch('open-complaint-modal', {listingId: {{ $listing->id }}})">
                ⚑
            </button>
        @else
            <span class="notice notice--info">Это ваше объявление</span>
        @endif
    @endguest
</div>

@guest
    <div class="notice notice--info">
        <span class="notice__icon">ⓘ</span>
        <span>Чтобы написать продавцу или сообщить о нарушении, войдите в систему.</span>
    </div>
@endguest
```

### Auth pages (login, register)

Используют split-layout `.page-auth`. Левая часть (`.auth-decor`) — декоративный брендинг, правая (`.auth-form`) — сама форма. См. снипеты в `styles.css`.

### Message pages (/register/sent, /verify/*, /blocked)

Узкая центрированная карточка. Единственное отличие между экранами — иконка, цвет иконки и заголовок:

```html
<div class="page-message">
    <div class="message-card">
        <div class="message-card__icon message-card__icon--info">
            <svg>...</svg>
        </div>
        <h1 class="message-card__heading">Проверьте почту</h1>
        <p class="message-card__body">
            Мы отправили письмо на <strong>{{ $email }}</strong>.
        </p>

        {{-- Форма повторной отправки или другие действия --}}

        <div class="message-card__footer">
            Не пришло письмо? Подождите пару минут.
        </div>
    </div>
</div>
```

| Экран | Иконка | Модификатор иконки |
|---|---|---|
| `/register/sent` | письмо | `--info` |
| `/verify/success` | галочка | `--success` |
| `/verify/expired` | восклицательный знак | `--warn` |
| `/blocked` | запрещающий знак | `--danger` |

---

## Дополнение: когда добавлять новый компонент

Если вы хотите добавить что-то новое, сначала задайте вопросы:

1. **Встречается ли этот элемент на 3+ экранах?** Если да — это компонент. Если нет — достаточно inline-стилей на конкретной странице.
2. **Можно ли это собрать из существующих компонентов?** Например, модалка жалобы = `.notice` + `.field` + `.btn`.
3. **Новое ли это визуально?** Если это просто вариация (другой цвет, размер) — добавьте модификатор `.block--variant` к существующему, а не новый блок.

Только если ответили «да» на 1 и «нет» на 2-3 — добавляйте новый компонент. И сразу задокументируйте его в этом файле.
