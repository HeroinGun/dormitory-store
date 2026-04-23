# ОбщМаркет — Карта маршрутов Laravel

Полный справочник HTTP-маршрутов приложения. Все маршруты определяются в `routes/web.php` (HTTP-рендер Blade) и используют сессионную аутентификацию с CSRF.

**Зачем этот документ:** чтобы при создании Blade-шаблонов и контроллеров сразу было понятно, какой экран вызывает какой маршрут, какой метод использовать, какие middleware применить, и на какое требование SRS это опирается.

**Структура:**

1. Условные обозначения
2. Middleware и группы
3. Публичные маршруты (гость + все роли)
4. Маршруты студента (`auth`)
5. Маршруты модератора (`auth + role:moderator`)
6. JSON / AJAX-эндпоинты (для Alpine.js и Reverb)
7. Routes file skeleton (готовый шаблон `routes/web.php`)

---

## 1. Условные обозначения

| Символ | Значение |
|---|---|
| `GET` | Отображение страницы, без побочных эффектов |
| `POST` | Создание ресурса (например, нового объявления) |
| `PATCH` / `PUT` | Обновление существующего ресурса |
| `DELETE` | Soft-удаление (`status=closed` и подобное) |
| **SRS** | Ссылка на пункт спецификации, который реализует маршрут |
| **Mw** | Требуемый middleware |
| **JSON** | Возвращает JSON, а не HTML (для AJAX) |

Все маршруты, изменяющие состояние (POST/PATCH/PUT/DELETE), автоматически защищены CSRF — используйте `@csrf` в формах.

---

## 2. Middleware и группы

| Middleware | Что делает |
|---|---|
| `guest` | Редиректит авторизованных на `/`. Применяется к `/login`, `/register`. |
| `auth` | Редиректит неавторизованных на `/login?return=...`. |
| `verified` | Блокирует вход до подтверждения email. Применяется вместе с `auth`. |
| `role:moderator` | Пропускает только `users.role = moderator`. Кастомный middleware. |
| `check.blocked` | Проверяет `users.is_blocked`. Если true — разлогинивает и редиректит на `/blocked`. Применяется глобально для авторизованных. |
| `throttle:verify-resend` | Rate limit для AUTH-07: 1/мин, 5/сутки. Laravel's RateLimiter. |

---

## 3. Публичные маршруты

Доступны всем, включая гостя. В коде не требуют middleware `auth`.

| Метод | URL | Route name | Контроллер@метод | SRS | Комментарий |
|---|---|---|---|---|---|
| `GET` | `/` | `feed` | `FeedController@index` | LIST-04, LIST-05, LIST-06, LIST-07 | Лента. Принимает query params: `q`, `categories[]`, `type`, `price_from`, `price_to`, `sort`, `page`. |
| `GET` | `/listings/{listing}` | `listing.show` | `ListingController@show` | LIST-04 | Страница объявления. Если `status !== active` — 404 (кроме случаев для автора или модератора — они увидят и другие статусы). |
| `GET` | `/register` | `register` | `AuthController@showRegister` | AUTH-01 | Форма регистрации. Mw: `guest`. |
| `POST` | `/register` | `register.store` | `AuthController@register` | AUTH-01, AUTH-02 | Создаёт `users`, `email_verifications`, dispatch SendVerificationEmail job. Редирект на `/register/sent`. Mw: `guest`. |
| `GET` | `/register/sent` | `register.sent` | `AuthController@sent` | AUTH-02 | «Проверьте почту». Email передаётся через session flash. |
| `POST` | `/register/resend` | `register.resend` | `AuthController@resend` | AUTH-07 | Повторная отправка письма. Mw: `throttle:verify-resend`. Редирект назад с toast. |
| `GET` | `/verify` | `verify.handle` | `AuthController@verify` | AUTH-02 | Обработчик ссылки из письма. Принимает `?token=...`. Внутри редиректит на `/verify/success` или `/verify/expired`. Сам HTML не рендерит. |
| `GET` | `/verify/success` | `verify.success` | `AuthController@verifySuccess` | AUTH-02 | «Email подтверждён». |
| `GET` | `/verify/expired` | `verify.expired` | `AuthController@verifyExpired` | AUTH-07 | «Ссылка устарела» + форма повторной отправки. |
| `POST` | `/verify/resend` | `verify.resend` | `AuthController@resend` | AUTH-07 | Тот же метод, что и `register.resend`. Можно оставить один маршрут с двумя алиасами. |
| `GET` | `/login` | `login` | `AuthController@showLogin` | AUTH-03 | Форма входа. Mw: `guest`. |
| `POST` | `/login` | `login.attempt` | `AuthController@login` | AUTH-03 | Проверка email + password + `is_blocked`. Если is_blocked — редирект на `/blocked`. Mw: `guest`. |
| `GET` | `/blocked` | `blocked` | `AuthController@blocked` | AUTH-03, MOD-06 | «Аккаунт заблокирован». Единственный доступный экран для забаненного. |
| `POST` | `/logout` | `logout` | `AuthController@logout` | AUTH-04 | Инвалидирует сессию + удаляет ключ в Redis. Mw: `auth`. |

---

## 4. Маршруты студента

Требуют `auth` + `verified` + `check.blocked`. Модератор имеет доступ ко всему этому плюс собственные маршруты.

### Объявления

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `GET` | `/my-listings` | `my-listings` | `MyListingController@index` | PROF-01 |
| `GET` | `/listings/new` | `listing.create` | `ListingController@create` | LIST-01 |
| `POST` | `/listings` | `listing.store` | `ListingController@store` | LIST-01, LIST-08, NFR-08 |
| `GET` | `/listings/{listing}/edit` | `listing.edit` | `ListingController@edit` | LIST-02 |
| `PATCH` | `/listings/{listing}` | `listing.update` | `ListingController@update` | LIST-02, NFR-05 |
| `DELETE` | `/listings/{listing}` | `listing.destroy` | `ListingController@destroy` | LIST-03 |
| `POST` | `/listings/{listing}/confirm` | `listing.confirm` | `ListingController@confirm` | 3.3 (pending→active) |
| `POST` | `/listings/{listing}/republish` | `listing.republish` | `ListingController@republish` | 3.3 (inactive→active) |

**Важно о LIST-02:** контроллер `update` принимает только `description` и `price`. Используйте `FormRequest::validated()`, который возвращает whitelist, и дополнительно защититесь триггером БД `BEFORE UPDATE listings`.

### Чат

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `GET` | `/chats` | `chat.index` | `ChatController@index` | CHAT-06 |
| `GET` | `/chats/{chat}` | `chat.show` | `ChatController@show` | CHAT-05 |
| `POST` | `/chats/open-for/{listing}` | `chat.openFor` | `ChatController@openFor` | CHAT-01 |
| `POST` | `/chats/{chat}/messages` | `message.send` | `MessageController@store` | CHAT-04 |

**openFor:** на странице объявления кнопка «Написать» ведёт сюда. Контроллер находит или создаёт чат по UNIQUE `(listing_id, buyer_id)`, редиректит на `/chats/{chat}`. Это решает CHAT-01.

**Отправка сообщения:** в MVP — обычный form submit с редиректом. Для realtime (CHAT-04) параллельно нужен WebSocket-канал через Reverb — см. раздел 6.

### Профиль и настройки

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `GET` | `/profile` | `profile` | `ProfileController@index` | PROF-01 |
| `GET` | `/profile/settings` | `profile.settings` | `ProfileController@settings` | PROF-02, PROF-03, PROF-04 |
| `PATCH` | `/profile` | `profile.update` | `ProfileController@update` | PROF-02 |
| `PATCH` | `/profile/password` | `profile.password` | `ProfileController@updatePassword` | AUTH-05, PROF-03 |
| `DELETE` | `/profile` | `profile.destroy` | `ProfileController@destroy` | AUTH-06, PROF-04 |

### Жалобы (как репортёр)

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `POST` | `/complaints` | `complaint.store` | `ComplaintController@store` | MOD-01, MOD-02 |

Принимает `listing_id` и `reason`. UNIQUE `(reporter_id, listing_id)` — если уже жаловался, возвращает 409.

### Уведомления

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `GET` | `/notifications` | `notifications` | `NotificationController@index` | 3.6 |
| `POST` | `/notifications/{id}/read` | `notifications.read` | `NotificationController@markRead` | 3.6 |
| `POST` | `/notifications/read-all` | `notifications.readAll` | `NotificationController@markAllRead` | 3.6 |

Dropdown-список в шапке подгружается через отдельный JSON-эндпоинт (см. раздел 6).

---

## 5. Маршруты модератора

Требуют дополнительно `role:moderator`.

| Метод | URL | Route name | Контроллер@метод | SRS |
|---|---|---|---|---|
| `GET` | `/moderation/complaints` | `moderation.complaints` | `ModerationController@index` | MOD-04 |
| `GET` | `/moderation/complaints/{complaint}` | `moderation.show` | `ModerationController@show` | MOD-04 |
| `POST` | `/moderation/complaints/{complaint}/block` | `moderation.block` | `ModerationController@block` | MOD-04, MOD-06, NFR-08 |
| `POST` | `/moderation/complaints/{complaint}/dismiss` | `moderation.dismiss` | `ModerationController@dismiss` | MOD-04 |
| `POST` | `/moderation/complaints/{complaint}/restore` | `moderation.restore` | `ModerationController@restore` | MOD-05, MOD-07, NFR-08 |
| `POST` | `/listings/{listing}/block-directly` | `moderation.blockDirect` | `ModerationController@blockDirect` | MOD-03 |

**Транзакционность (NFR-08):** методы `block`, `restore`, `blockDirect` должны выполняться в `DB::transaction(function () { ... })`. Внутри транзакции — обновление `listings`, `complaints`, `users.blocked_count`, инвалидация сессий в Redis (для автобана).

**Автобан:** в методе `block` после инкремента `blocked_count` проверяйте `>= 5`. Если да — вызовите `app(AuthService::class)->autoBan($user)`, который установит `is_blocked = true` и удалит ключи сессий в Redis.

**Автоснятие (MOD-07):** в методе `restore` после декремента проверяйте `blocked_count < 5 AND is_blocked = true`. Если да — снимайте бан. Поскольку ручных банов нет (SRS 2.0), дополнительных проверок не требуется.

---

## 6. JSON / AJAX-эндпоинты

Эти маршруты вызываются через `fetch()` из Alpine.js или через Laravel Echo (Reverb). Они живут в `routes/web.php` (чтобы пользоваться сессионной аутентификацией и CSRF), но возвращают JSON вместо HTML.

**Префикс:** `/api/*`. Это визуальная конвенция — всё в одном monolith-приложении, отдельного `routes/api.php` в MVP не нужно.

| Метод | URL | Возвращает | Используется на | SRS |
|---|---|---|---|---|
| `GET` | `/api/listings` | JSON с массивом объявлений + `hasMore` флаг | «Показать ещё» на `/` | LIST-05..07 |
| `POST` | `/api/listings/photos` | `{url, thumb_url}` | Загрузка фото в `/listings/new` до сабмита | LIST-08 |
| `GET` | `/api/notifications` | JSON с последними 10 уведомлениями | Dropdown в хедере | 3.6 |
| `GET` | `/api/notifications/unread-count` | `{count: N}` | Бейдж на колокольчике | 3.6 |

**WebSocket-каналы (Laravel Reverb):**

| Канал | Событие | Тип | Доставляется | Реагирует |
|---|---|---|---|---|
| `private-chat.{chat_id}` | `MessageSent` | Private | Обоим участникам чата | Обновляет список сообщений без перезагрузки |
| `private-user.{user_id}` | `NotificationCreated` | Private | Получателю | Инкрементирует счётчик колокольчика |

**Примеры использования Alpine.js в шаблонах:**

```html
<!-- Показать ещё на ленте -->
<button x-data="{
    page: {{ $nextPage }},
    loading: false,
    async load() {
        this.loading = true;
        const res = await fetch(`/api/listings?page=${this.page}&${new URLSearchParams(window.location.search)}`);
        const data = await res.json();
        data.listings.forEach(l => {
            document.querySelector('.page-feed__grid').insertAdjacentHTML('beforeend', l.html);
        });
        this.page++;
        if (!data.hasMore) this.$el.remove();
        this.loading = false;
    }
}" x-on:click="load()" :disabled="loading" class="btn btn--ghost">
    <span x-show="!loading">Показать ещё</span>
    <span x-show="loading">Загрузка...</span>
</button>
```

```html
<!-- Подписка на уведомления и колокольчик -->
<div x-data="{
    count: 0,
    async init() {
        const res = await fetch('/api/notifications/unread-count');
        this.count = (await res.json()).count;

        Echo.private(`user.{{ auth()->id() }}`)
            .listen('NotificationCreated', () => this.count++);
    }
}">
    <span class="badge" x-show="count > 0" x-text="count"></span>
</div>
```

---

## 7. Готовый скелет `routes/web.php`

```php
<?php

use App\Http\Controllers\AuthController;
use App\Http\Controllers\FeedController;
use App\Http\Controllers\ListingController;
use App\Http\Controllers\ChatController;
use App\Http\Controllers\MessageController;
use App\Http\Controllers\ProfileController;
use App\Http\Controllers\NotificationController;
use App\Http\Controllers\ComplaintController;
use App\Http\Controllers\ModerationController;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Публичные маршруты
|--------------------------------------------------------------------------
*/
Route::get('/', [FeedController::class, 'index'])->name('feed');
Route::get('/listings/{listing}', [ListingController::class, 'show'])->name('listing.show');

// Auth (только для гостя)
Route::middleware('guest')->group(function () {
    Route::get('/register', [AuthController::class, 'showRegister'])->name('register');
    Route::post('/register', [AuthController::class, 'register'])->name('register.store');
    Route::get('/login', [AuthController::class, 'showLogin'])->name('login');
    Route::post('/login', [AuthController::class, 'login'])->name('login.attempt');
});

// Email verification (для всех)
Route::get('/register/sent', [AuthController::class, 'sent'])->name('register.sent');
Route::post('/register/resend', [AuthController::class, 'resend'])
    ->middleware('throttle:verify-resend')->name('register.resend');
Route::get('/verify', [AuthController::class, 'verify'])->name('verify.handle');
Route::get('/verify/success', [AuthController::class, 'verifySuccess'])->name('verify.success');
Route::get('/verify/expired', [AuthController::class, 'verifyExpired'])->name('verify.expired');
Route::get('/blocked', [AuthController::class, 'blocked'])->name('blocked');

/*
|--------------------------------------------------------------------------
| Маршруты студента
|--------------------------------------------------------------------------
*/
Route::middleware(['auth', 'verified', 'check.blocked'])->group(function () {
    Route::post('/logout', [AuthController::class, 'logout'])->name('logout');

    // Listings
    Route::get('/my-listings', [ListingController::class, 'myListings'])->name('my-listings');
    Route::get('/listings/new', [ListingController::class, 'create'])->name('listing.create');
    Route::post('/listings', [ListingController::class, 'store'])->name('listing.store');
    Route::get('/listings/{listing}/edit', [ListingController::class, 'edit'])->name('listing.edit');
    Route::patch('/listings/{listing}', [ListingController::class, 'update'])->name('listing.update');
    Route::delete('/listings/{listing}', [ListingController::class, 'destroy'])->name('listing.destroy');
    Route::post('/listings/{listing}/confirm', [ListingController::class, 'confirm'])->name('listing.confirm');
    Route::post('/listings/{listing}/republish', [ListingController::class, 'republish'])->name('listing.republish');

    // Chat
    Route::get('/chats', [ChatController::class, 'index'])->name('chat.index');
    Route::get('/chats/{chat}', [ChatController::class, 'show'])->name('chat.show');
    Route::post('/chats/open-for/{listing}', [ChatController::class, 'openFor'])->name('chat.openFor');
    Route::post('/chats/{chat}/messages', [MessageController::class, 'store'])->name('message.send');

    // Profile
    Route::get('/profile', [ProfileController::class, 'index'])->name('profile');
    Route::get('/profile/settings', [ProfileController::class, 'settings'])->name('profile.settings');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::patch('/profile/password', [ProfileController::class, 'updatePassword'])->name('profile.password');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');

    // Notifications
    Route::get('/notifications', [NotificationController::class, 'index'])->name('notifications');
    Route::post('/notifications/{id}/read', [NotificationController::class, 'markRead'])->name('notifications.read');
    Route::post('/notifications/read-all', [NotificationController::class, 'markAllRead'])->name('notifications.readAll');

    // Complaints (as reporter)
    Route::post('/complaints', [ComplaintController::class, 'store'])->name('complaint.store');

    // JSON / AJAX endpoints
    Route::prefix('api')->group(function () {
        Route::get('/listings', [FeedController::class, 'ajaxList']);
        Route::post('/listings/photos', [ListingController::class, 'uploadPhoto']);
        Route::get('/notifications', [NotificationController::class, 'dropdown']);
        Route::get('/notifications/unread-count', [NotificationController::class, 'unreadCount']);
    });
});

/*
|--------------------------------------------------------------------------
| Маршруты модератора
|--------------------------------------------------------------------------
*/
Route::middleware(['auth', 'verified', 'check.blocked', 'role:moderator'])
    ->prefix('moderation')->group(function () {
        Route::get('/complaints', [ModerationController::class, 'index'])->name('moderation.complaints');
        Route::get('/complaints/{complaint}', [ModerationController::class, 'show'])->name('moderation.show');
        Route::post('/complaints/{complaint}/block', [ModerationController::class, 'block'])->name('moderation.block');
        Route::post('/complaints/{complaint}/dismiss', [ModerationController::class, 'dismiss'])->name('moderation.dismiss');
        Route::post('/complaints/{complaint}/restore', [ModerationController::class, 'restore'])->name('moderation.restore');
});

Route::middleware(['auth', 'verified', 'check.blocked', 'role:moderator'])
    ->post('/listings/{listing}/block-directly', [ModerationController::class, 'blockDirect'])
    ->name('moderation.blockDirect');
```

---

## Рекомендации по реализации

**FormRequest-классы:** создайте отдельный FormRequest для каждого действия с валидацией (`StoreListingRequest`, `UpdateListingRequest`, `StoreComplaintRequest` и т.д.). Это закрывает NFR-05 и даёт чистую валидацию без загромождения контроллеров.

**Jobs:** асинхронная отправка email (верификация, повторная верификация) — через Laravel Queue + Redis. Класс `SendVerificationEmail implements ShouldQueue`.

**Events:** при отправке сообщения в чате и при создании уведомления — dispatch event `MessageSent` / `NotificationCreated`, который broadcast через Reverb (`implements ShouldBroadcast`).

**Scheduler (`app/Console/Kernel.php`):** настройте три команды:
- `listings:check-expiry` — каждые 5 минут, обрабатывает `CheckListingExpiry`.
- `listings:deactivate-unconfirmed` — каждые 5 минут, обрабатывает `DeactivateUnconfirmed`.
- Автобан не в scheduler, он синхронный в рамках транзакции `block`.

**RateLimiter (`app/Providers/RouteServiceProvider.php`):**

```php
RateLimiter::for('verify-resend', function (Request $request) {
    return [
        Limit::perMinute(1)->by($request->input('email') ?? $request->ip()),
        Limit::perDay(5)->by($request->input('email') ?? $request->ip()),
    ];
});
```

---

## Что НЕ включено в этот документ

- Формат ответов JSON-эндпоинтов — это можно уточнить при реализации конкретного экрана.
- Формат событий WebSocket — аналогично, можно уточнить при работе с чатом.
- Шаблоны email-писем — отдельная тема.

Когда приступите к реализации экранов авторизованных пользователей или модератора — дайте знать, и я дополню этот документ прикладными примерами (контроллер, FormRequest, Blade-шаблон) для конкретного экрана.
