<div align="center">

<img src="https://img.shields.io/badge/SafeAuth-2.0-blueviolet?style=for-the-badge&logo=minecraft" alt="SafeAuth"/>

# 🔐 SafeAuth

**Advanced Account Protection Plugin for Minecraft Servers**

[![Version](https://img.shields.io/badge/version-2.0-blue?style=for-the-badge)](https://github.com)
[![Paper](https://img.shields.io/badge/Paper-1.20.1+-green?style=for-the-badge)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)](https://adoptium.net)
[![License](https://img.shields.io/badge/license-MIT-purple?style=for-the-badge)](LICENSE)

*Security-hardened authentication with bcrypt, TOTP 2FA, action captcha, login streaks and full PlaceholderAPI support.*

**[Features](#-features) • [Installation](#-installation) • [Commands](#-commands) • [Configuration](#-configuration) • [Placeholders](#-placeholderapi) • [Security](#-security)**

</div>

---

## ✨ Features

| | Feature | Description |
|--|---------|-------------|
| 🔑 | **bcrypt Passwords** | Industry-standard hashing with configurable cost factor (10–14) |
| 🎫 | **Session System** | Auto-login on rejoin — no password needed within session window |
| 📱 | **TOTP 2FA** | Google Authenticator / Aegis / Authy — RFC 6238, 6-digit codes |
| 🤖 | **Text Captcha** | Unicode homoglyph + Zero-Width Characters — AI/OCR-resistant |
| 🎮 | **Action Captcha** | Jump / Crouch / Look / Drop — bots cannot execute physical actions |
| 🔥 | **Login Streak** | Daily login counter with color tiers and PlaceholderAPI support |
| 🔒 | **Password Strength** | Live feedback (Weak → Very Strong) shown on registration |
| 📊 | **Server Stats** | `/safeauth stats` — accounts, 2FA adoption, locked count |
| 🌍 | **Auth World** | Dedicated login lobby with auto location save & restore |
| 👑 | **Staff 2FA Enforcement** | Require 2FA for specific permission groups only |
| 🔗 | **Proxy Support** | Velocity modern forwarding + BungeeCord with HMAC signatures |
| 🛡️ | **Brute-Force Protection** | IP-based fail counter with automatic timed block |
| ⚡ | **Rate Limiting** | Sliding-window limits on login, register, and join events |
| 🚫 | **Command Blocker** | 28 always-blocked commands (`/plugins`, `/op`, `/reload`, etc.) |
| 📋 | **Audit Log** | Full security event log to file |
| 🎨 | **Rich Effects** | Titles, boss bar, action bar, particles, sounds — all configurable |
| 📊 | **PlaceholderAPI** | 10 placeholders including colored streak |
| ⚖️ | **LuckPerms** | Temporary group assignment until authenticated |

---

## 📋 Requirements

| Requirement | Version |
|-------------|---------|
| Paper / Purpur / Folia | 1.20.1+ |
| Java | 17+ |
| Multiverse-Core | Optional — for auth world |
| PlaceholderAPI | Optional — for placeholders |
| LuckPerms | Optional — for group management |

---

## 🚀 Installation

```
1. Download SafeAuth-2.0.jar → put it in /plugins/
2. Start the server → config.yml and messages.yml are generated
3. Edit config.yml (database, language, features)
4. Restart or /safeauth reload
5. Set admin password: /safeauth setadminpass <password>
```

### Auth World Setup *(optional, requires Multiverse-Core)*
```
1. /mv create auth_world normal
2. Set auth-world.enabled: true in config.yml
3. Set auth-world.spawn.x/y/z coordinates
4. Restart the server
```
> ⚠️ Multiverse-Core must load **before** SafeAuth.

---

## 🔄 How It Works

```
Player joins
    │
    ├─► IP blocked?          → Kicked instantly
    ├─► Join flood from IP?  → Kicked (bot protection)
    ├─► Valid session?       → Auto-authenticated ✓
    │
    ├─► Frozen, invisible, movement/chat/commands blocked
    ├─► Auth world enabled?  → Teleport to lobby
    ├─► Captcha enabled?     → Must solve captcha first
    │       ├─ type: text    → Type obfuscated code in chat
    │       └─ type: action  → Jump / Crouch / Look / Drop item
    │
    ├─► /login <password>
    │       ├─► Staff member? → /adminlogin <admin-password>
    │       └─► 2FA enabled?  → /2fa code <6 digits>
    │
    └─► Authenticated ✓
            ├─ Auth world → Teleport back to saved location
            ├─ Login streak updated
            └─ LuckPerms group restored
```

---

## 💬 Commands

### Player Commands

| Command | Description |
|---------|-------------|
| `/register <pass> <pass>` | Create an account |
| `/login <password>` | Log in |
| `/logout` | Log out (clears session) |
| `/changepassword <old> <new>` | Change password |
| `/adminlogin <password>` | Second factor for staff |
| `/2fa setup` | Start TOTP 2FA setup |
| `/2fa confirm <code>` | Confirm setup with 6-digit code |
| `/2fa code <code>` | Enter code during login |
| `/2fa disable <code>` | Disable 2FA (requires working code) |
| `/2fa status` | Check your 2FA status |

### Admin Commands *(permission: `safeauth.admin`)*

| Command | Description |
|---------|-------------|
| `/safeauth reload` | Reload config.yml and messages.yml |
| `/safeauth stats` | Show server statistics panel |
| `/safeauth help` | Show all admin commands |
| `/safeauth info <player>` | View player account details |
| `/safeauth resetpass <player>` | Reset a player's password |
| `/safeauth unregister <player>` | Delete a player account |
| `/safeauth forcelogin <player>` | Force-authenticate a player |
| `/safeauth setadminpass <pass>` | Set the global admin password |
| `/safeauth kick` | Kick all unauthenticated players |

---

## 📱 TOTP Two-Factor Authentication (2FA)

2FA adds a second layer of protection — even if a password leaks (stream, demo recording, other server), the attacker cannot log in without the player's phone.

### How players set it up

```
1. /2fa setup
   → SafeAuth shows a 16-character secret key and an otpauth:// link

2. Open Google Authenticator → tap + → Enter a setup key
   Enter any account name and paste the 16-character key
   (or open any online QR generator and paste the otpauth:// link)

3. /2fa confirm 123456
   → Confirm with the 6-digit code from the app

4. Done! On next login: /login <password> → /2fa code 123456
```

### Server-secret key

The `two-factor.totp.server-secret` in `config.yml` encrypts player secrets in the database. **If you leave the default value, it is generated automatically on first start.** You don't need to do anything.

> ⛔ Never change `server-secret` after players have linked 2FA — it will break all existing bindings.

### Enforcement modes

| Config | Behavior |
|--------|----------|
| `require-for-all: false` + empty `require-for-permission` | Fully optional |
| `require-for-permission: safeauth.staff` | Only staff must have 2FA |
| `require-for-all: true` | Everyone must set up 2FA before they can play |

---

## 🎮 Action Captcha

Unlike text captcha, action captcha **cannot be solved by bots** — it requires physically controlling the Minecraft character.

| Type | What the player does |
|------|---------------------|
| `JUMP` | Press Space N times |
| `SNEAK` | Hold Shift N times |
| `LOOK` | Turn camera towards a direction |
| `ITEM_DROP` | Press Q to drop a given item |
| `COMBO` | Jump N times, then crouch |
| `RANDOM` | Random type each login *(recommended)* |

```yaml
captcha:
  enabled: true
  type: action
  action:
    type: RANDOM        # JUMP / SNEAK / LOOK / ITEM_DROP / COMBO / RANDOM
    count-max: 3        # random 1..N repetitions
    timeout: 30         # seconds to complete
    kick-on-timeout: true
```

---

## 🔥 Login Streak

Players earn a consecutive-day login streak. The streak number changes color based on configurable tiers.

```yaml
features:
  login-streak:
    enabled: true
    color-tiers:
      - min-days: 0     color: "<white>"
      - min-days: 3     color: "<yellow>"
      - min-days: 7     color: "<gold>"
      - min-days: 14    color: "<green>"
      - min-days: 30    color: "<aqua>"
      - min-days: 60    color: "<light_purple>"
      - min-days: 100   color: "<gradient:#FFD700:#FF6B35>"
      - min-days: 365   color: "<gradient:#FF0000:#FF6B35:#FFD700>"
    color-suffix: "</gradient>"
    icon: "🔥 "
```

| Placeholder | Output |
|-------------|--------|
| `%safeauth_streak%` | Raw number: `14` |
| `%safeauth_streak_colored%` | Colored with icon: `🔥 <gold>14</gold>` |

---

## 📊 PlaceholderAPI

Requires [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/).

| Placeholder | Returns | Example |
|-------------|---------|---------|
| `%safeauth_authenticated%` | `true` / `false` | `true` |
| `%safeauth_registered%` | `true` / `false` | `true` |
| `%safeauth_has_2fa%` | `true` / `false` | `false` |
| `%safeauth_is_admin_authed%` | `true` / `false` | `true` |
| `%safeauth_last_login%` | Formatted date | `26.02.2026 19:45` |
| `%safeauth_registered_date%` | Formatted date | `01.01.2026 10:00` |
| `%safeauth_failed_attempts%` | Number | `2` |
| `%safeauth_total_logins%` | Number | `347` |
| `%safeauth_streak%` | Number | `14` |
| `%safeauth_streak_colored%` | Colored string | `🔥 <gold>14</gold>` |

---

## ⚙️ Configuration Overview

`config.yml` contains only **display parameters** (enabled, colors, timings, sounds).  
All **text content** is in `messages.yml`.

### Key sections

| Section | Purpose |
|---------|---------|
| `general` | Language (`en` / `ru`), prefix, logging |
| `database` | SQLite (default) or MySQL |
| `proxy` | Velocity / BungeeCord + HMAC secret |
| `auth` | Timeout, max attempts, bcrypt rounds, password requirements |
| `two-factor` | TOTP enable, server-secret, enforcement, visual settings |
| `captcha` | Type, obfuscation, action settings, visual settings |
| `restrictions` | What is blocked before authentication |
| `auth-world` | Auth lobby world with Multiverse |
| `effects` | Titles, boss bar, particles, sounds, potion effects |
| `security` | IP brute-force, rate limits, audit log |
| `features` | Login streak tiers, password strength, stats |

### Language

```yaml
general:
  language: en    # en or ru
```

All message text is in `messages.yml` under `en:` and `ru:` blocks. You can edit any message freely.

---

## 🔒 Security

### Threat model

| Threat | Mitigation |
|--------|-----------|
| Password leak (stream / demo) | TOTP 2FA — phone required |
| Brute-force login | IP block after N fails, account lockout |
| Bot floods | Rate limiting on join, register, login |
| OCR / AI chat captcha bypass | Action captcha (physical input required) |
| `/plugins` reconnaissance | 28 always-blocked commands, silent drop |
| DB dump → password recovery | bcrypt (cost 12 default) + TOTP server-secret encryption |
| Fake proxy messages | HMAC-signed proxy channel |
| Replay attack on 2FA | Used-code tracking for 90 seconds |

### bcrypt cost

```yaml
auth:
  bcrypt-rounds: 12   # 10 = fast dev, 12 = recommended, 14 = high security
```

### Always-blocked commands

These commands are **always** blocked for unauthenticated players regardless of `allowed-commands`, and respond with silence (no output to avoid fingerprinting):

`/plugins` `/pl` `/version` `/ver` `/tps` `/lag` `/timings` `/reload` `/rl` `/stop` `/op` `/deop` `/give` `/gamemode` `/tp` `/teleport` `/whitelist` `/banlist` `/help` and their namespaced variants (`bukkit:*`, `paper:*`, `spigot:*`)

---

## 🗄️ Database

### SQLite *(default)*
```yaml
database:
  type: sqlite
  sqlite:
    file: safeauth.db
```

### MySQL
```yaml
database:
  type: mysql
  mysql:
    host: localhost
    port: 3306
    database: safeauth
    username: root
    password: ""
    pool-size: 10
```

> If you run multiple servers behind a proxy, you **must** use MySQL so all backends share the same account data.

---

## 📝 Messages

All player-facing text is in `messages.yml`. Both `en` and `ru` are included. Switch with `general.language`.

```yaml
# messages.yml — example customization
en:
  login-success: "<gradient:#00FF87:#60EFFF>✔ Welcome back, {player}!</gradient>"
  login-streak: "<gold>🔥 Day {streak} — keep the streak alive!"
```

Supports full **MiniMessage** format: gradients, hover, click events, etc.

---

## 📜 Permissions

| Permission | Default | Description |
|------------|---------|-------------|
| `safeauth.admin` | OP | Full access to admin commands |
| `safeauth.bypass` | false | Skip authentication entirely |
| `safeauth.staff` | false | Requires admin 2FA + optionally TOTP 2FA |

---

## 🔄 Upgrading from v1.x

SafeAuth v2.0 automatically migrates the database on first launch:
- Adds columns: `login_streak`, `total_logins`, `last_streak_day`
- Existing player data is preserved

No manual SQL needed.

---

## 🐛 Troubleshooting

**Players see squares in captcha code**
→ Set `captcha.obfuscation.separator-mode: none`

**2FA codes rejected**
→ Make sure the server clock is synced (`timedatectl status`). TOTP requires accurate time.

**Auth world not working**
→ Multiverse-Core must be in `softdepend` and load before SafeAuth. Check startup log order.

**`server-secret` warning in console**
→ This is normal on first start. The key was generated and saved to `config.yml` automatically.

---

<br>

---

<div align="center">

# 🔐 SafeAuth — Документация

**Плагин защиты аккаунтов для Minecraft серверов**

*Версия 2.0 • Безопасность • Двухфакторка • Стрик входов • Капча*

</div>

---

## ✨ Возможности

| | Функция | Описание |
|--|---------|----------|
| 🔑 | **Пароли bcrypt** | Хэширование с настраиваемым коэффициентом (10–14) |
| 🎫 | **Сессии** | Автовход при повторном подключении в пределах окна сессии |
| 📱 | **TOTP 2FA** | Google Authenticator / Aegis / Authy — стандарт RFC 6238 |
| 🤖 | **Текстовая капча** | Омоглифы + Zero-Width символы — защита от OCR и нейросетей |
| 🎮 | **Экшн-капча** | Прыжок / Присед / Поворот / Дроп — бот не выполнит действие |
| 🔥 | **Стрик входов** | Счётчик дней подряд с цветными тирами и PlaceholderAPI |
| 🔒 | **Сила пароля** | Фидбек при регистрации: Слабый → Очень сильный |
| 📊 | **Статистика** | `/safeauth stats` — аккаунты, 2FA, заблокированные |
| 🌍 | **Мир авторизации** | Лобби с сохранением и восстановлением позиции |
| 👑 | **Принуждение к 2FA** | Обязательная 2FA для конкретных групп прав |
| 🔗 | **Прокси** | Velocity + BungeeCord с HMAC-подписью |
| 🛡️ | **Защита от брутфорса** | Блокировка IP + блокировка аккаунта |
| ⚡ | **Rate Limiting** | Скользящие окна для входа, регистрации, подключений |
| 🚫 | **Блокировка команд** | 28 постоянно заблокированных команд (`/plugins`, `/op`…) |
| 📋 | **Аудит-лог** | Полный лог событий безопасности в файл |
| 🎨 | **Визуал** | Тайтлы, боссбар, частицы, звуки — всё настраивается |
| 📊 | **PlaceholderAPI** | 10 плейсхолдеров включая цветной стрик |

---

## 📋 Требования

| Зависимость | Версия |
|-------------|--------|
| Paper / Purpur / Folia | 1.20.1+ |
| Java | 17+ |
| Multiverse-Core | Опционально — для мира авторизации |
| PlaceholderAPI | Опционально |
| LuckPerms | Опционально |

---

## 🚀 Установка

```
1. Скачай SafeAuth-2.0.jar → положи в /plugins/
2. Запусти сервер → config.yml и messages.yml создадутся автоматически
3. Настрой config.yml (база данных, язык, функции)
4. Перезапусти сервер или выполни /safeauth reload
5. Установи пароль администратора: /safeauth setadminpass <пароль>
```

### Настройка мира авторизации *(опционально, требует Multiverse-Core)*
```
1. /mv create auth_world normal
2. Установи auth-world.enabled: true в config.yml
3. Укажи координаты спавна: auth-world.spawn.x/y/z
4. Перезапусти сервер
```
> ⚠️ Multiverse-Core должен загружаться **до** SafeAuth.

---

## 🔄 Как это работает

```
Игрок заходит
    │
    ├─► IP заблокирован?       → Кик мгновенно
    ├─► Флуд с IP?             → Кик (защита от ботов)
    ├─► Активная сессия?       → Автовход ✓
    │
    ├─► Заморожен, невидим, чат/команды/движение заблокированы
    ├─► Мир авторизации?       → Телепорт в лобби
    ├─► Капча включена?        → Нужно решить капчу
    │       ├─ тип: text       → Ввести код в чат
    │       └─ тип: action     → Прыжок / Присед / Поворот / Дроп
    │
    ├─► /login <пароль>
    │       ├─► Стафф?         → /adminlogin <пароль>
    │       └─► 2FA включена?  → /2fa code <6 цифр>
    │
    └─► Авторизован ✓
            ├─ Мир авторизации → Телепорт обратно
            ├─ Стрик обновляется
            └─ Группа LuckPerms восстанавливается
```

---

## 💬 Команды

### Команды игроков

| Команда | Описание |
|---------|----------|
| `/register <пароль> <пароль>` | Создать аккаунт |
| `/login <пароль>` | Войти |
| `/logout` | Выйти (сбрасывает сессию) |
| `/changepassword <старый> <новый>` | Изменить пароль |
| `/adminlogin <пароль>` | Второй фактор для стаффа |
| `/2fa setup` | Начать настройку TOTP 2FA |
| `/2fa confirm <код>` | Подтвердить настройку |
| `/2fa code <код>` | Ввести код при входе |
| `/2fa disable <код>` | Отключить 2FA |
| `/2fa status` | Проверить статус 2FA |

### Команды администратора *(право: `safeauth.admin`)*

| Команда | Описание |
|---------|----------|
| `/safeauth reload` | Перезагрузить конфиг и сообщения |
| `/safeauth stats` | Показать статистику сервера |
| `/safeauth help` | Показать все команды |
| `/safeauth info <игрок>` | Информация об аккаунте |
| `/safeauth resetpass <игрок>` | Сбросить пароль игрока |
| `/safeauth unregister <игрок>` | Удалить аккаунт игрока |
| `/safeauth forcelogin <игрок>` | Принудительная авторизация |
| `/safeauth setadminpass <пароль>` | Установить пароль администратора |
| `/safeauth kick` | Кикнуть всех неавторизованных |

---

## 📱 TOTP Двухфакторная аутентификация

2FA защищает аккаунт даже если пароль утёк (стрим, демо, другой сервер) — без телефона войти невозможно.

### Как игрок подключает 2FA

```
1. /2fa setup
   → SafeAuth показывает 16-символьный секрет и ссылку otpauth://

2. Открой Google Authenticator → + → Ввести ключ вручную
   Введи любое имя аккаунта и 16-символьный ключ
   (или открой онлайн QR-генератор и вставь ссылку otpauth://)

3. /2fa confirm 123456
   → Подтверди 6-значным кодом из приложения

4. Готово! При следующем входе: /login <пароль> → /2fa code 123456
```

### Серверный ключ шифрования

`two-factor.totp.server-secret` в `config.yml` шифрует TOTP-секреты игроков в базе данных.  
**Если оставить значение по умолчанию — ключ сгенерируется автоматически при первом запуске.**

> ⛔ Никогда не меняй `server-secret` после того как игроки привязали 2FA — это сломает все их привязки.

### Режимы принуждения к 2FA

| Настройка | Поведение |
|-----------|----------|
| `require-for-all: false` + пустой `require-for-permission` | Полностью добровольная |
| `require-for-permission: safeauth.staff` | Только стафф обязан |
| `require-for-all: true` | Все обязаны настроить 2FA |

---

## 🎮 Экшн-капча

В отличие от текстовой, экшн-капча **не может быть решена ботами** — требует физического управления персонажем.

| Тип | Что делает игрок |
|-----|-----------------|
| `JUMP` | Нажать Пробел N раз |
| `SNEAK` | Зажать Shift N раз |
| `LOOK` | Повернуть камеру в нужную сторону |
| `ITEM_DROP` | Нажать Q чтобы выбросить предмет |
| `COMBO` | Прыжок N раз, потом присесть |
| `RANDOM` | Случайный тип при каждом входе *(рекомендуется)* |

```yaml
captcha:
  enabled: true
  type: action
  action:
    type: RANDOM        # JUMP / SNEAK / LOOK / ITEM_DROP / COMBO / RANDOM
    count-max: 3        # случайно 1..N повторений
    timeout: 30         # секунд на выполнение
    kick-on-timeout: true
```

---

## 🔥 Стрик входов

Игроки накапливают стрик последовательных дней входа. Число стрика меняет цвет по настраиваемым тирам.

```yaml
features:
  login-streak:
    enabled: true
    color-tiers:
      - min-days: 0     color: "<white>"
      - min-days: 3     color: "<yellow>"
      - min-days: 7     color: "<gold>"
      - min-days: 14    color: "<green>"
      - min-days: 30    color: "<aqua>"
      - min-days: 60    color: "<light_purple>"
      - min-days: 100   color: "<gradient:#FFD700:#FF6B35>"
      - min-days: 365   color: "<gradient:#FF0000:#FF6B35:#FFD700>"
    color-suffix: "</gradient>"
    icon: "🔥 "
```

| Плейсхолдер | Результат |
|-------------|-----------|
| `%safeauth_streak%` | Число: `14` |
| `%safeauth_streak_colored%` | С цветом и иконкой: `🔥 <gold>14</gold>` |

---

## 📊 PlaceholderAPI

Требует [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/).

| Плейсхолдер | Возвращает | Пример |
|-------------|-----------|--------|
| `%safeauth_authenticated%` | `true` / `false` | `true` |
| `%safeauth_registered%` | `true` / `false` | `true` |
| `%safeauth_has_2fa%` | `true` / `false` | `false` |
| `%safeauth_is_admin_authed%` | `true` / `false` | `true` |
| `%safeauth_last_login%` | Дата и время | `26.02.2026 19:45` |
| `%safeauth_registered_date%` | Дата и время | `01.01.2026 10:00` |
| `%safeauth_failed_attempts%` | Число | `2` |
| `%safeauth_total_logins%` | Число | `347` |
| `%safeauth_streak%` | Число | `14` |
| `%safeauth_streak_colored%` | Цветная строка | `🔥 <gold>14</gold>` |

---

## ⚙️ Обзор конфигурации

`config.yml` содержит только **параметры отображения** (enabled, цвета, тайминги, звуки).  
Весь **текст** — в `messages.yml`.

### Основные секции

| Секция | Назначение |
|--------|-----------|
| `general` | Язык (`en` / `ru`), префикс, логирование |
| `database` | SQLite (по умолчанию) или MySQL |
| `proxy` | Velocity / BungeeCord + HMAC-секрет |
| `auth` | Таймаут, попытки, bcrypt, требования к паролю |
| `two-factor` | TOTP, серверный ключ, принуждение, визуал |
| `captcha` | Тип, обфускация, экшн-настройки, визуал |
| `restrictions` | Что заблокировано до авторизации |
| `auth-world` | Лобби с Multiverse |
| `effects` | Тайтлы, боссбар, частицы, звуки, эффекты зелий |
| `security` | IP брутфорс, rate limits, аудит-лог |
| `features` | Стрик тиры, сила пароля, статистика |

### Язык

```yaml
general:
  language: en    # en или ru
```

Все тексты в `messages.yml` в блоках `en:` и `ru:`. Можно редактировать любое сообщение.

---

## 🔒 Безопасность

### Модель угроз

| Угроза | Защита |
|--------|--------|
| Утечка пароля (стрим / демо) | TOTP 2FA — нужен телефон |
| Брутфорс | Блокировка IP + блокировка аккаунта |
| Боты / флуд | Rate limiting на вход, регистрацию, подключения |
| OCR / AI обход текстовой капчи | Экшн-капча (нужно физически управлять персонажем) |
| Разведка через `/plugins` | 28 всегда заблокированных команд, тихий дроп |
| Дамп БД → восстановление паролей | bcrypt (стоимость 12) + шифрование TOTP server-secret |
| Подделка прокси-сообщений | HMAC-подписанный канал |
| Replay-атака на 2FA | Использованные коды запоминаются на 90 секунд |

---

## 🗄️ База данных

### SQLite *(по умолчанию)*
```yaml
database:
  type: sqlite
```

### MySQL
```yaml
database:
  type: mysql
  mysql:
    host: localhost
    port: 3306
    database: safeauth
    username: root
    password: ""
    pool-size: 10
```

> Если используешь несколько серверов за прокси — **обязательно** MySQL, иначе у каждого сервера будет своя БД.

---

## 📝 Сообщения

Все тексты для игроков в `messages.yml`. Есть `en:` и `ru:` блоки. Переключение через `general.language`.

```yaml
ru:
  login-success: "<gradient:#00FF87:#60EFFF>✔ Добро пожаловать, {player}!</gradient>"
  login-streak: "<gold>🔥 День {streak} — стрик жив!"
```

Поддерживается полный **MiniMessage**: градиенты, hover, click-события и т.д.

---

## 📜 Права

| Право | По умолчанию | Описание |
|-------|-------------|----------|
| `safeauth.admin` | OP | Полный доступ к командам администратора |
| `safeauth.bypass` | false | Пропустить авторизацию полностью |
| `safeauth.staff` | false | Требует второй пароль + опционально TOTP |

---

## 🔄 Обновление с v1.x

SafeAuth v2.0 автоматически мигрирует базу данных при первом запуске:
- Добавляет колонки: `login_streak`, `total_logins`, `last_streak_day`
- Данные существующих игроков сохраняются

SQL-запросы руками не нужны.

---

## 🐛 Решение проблем

**Игроки видят квадратики в коде капчи**  
→ Установи `captcha.obfuscation.separator-mode: none`

**Коды 2FA не принимаются**  
→ Проверь синхронизацию времени на сервере (`timedatectl status`). TOTP требует точного времени.

**Мир авторизации не работает**  
→ Multiverse-Core должен загружаться до SafeAuth. Проверь порядок в логе запуска.

**Предупреждение о `server-secret` в консоли**  
→ Это нормально при первом запуске. Ключ сгенерирован и сохранён в `config.yml` автоматически.

---

<div align="center">

*SafeAuth v2.0 • Made with ❤️*

</div>
