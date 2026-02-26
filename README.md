<div align="center">

# 🔐 SafeAuth

**Advanced Account Protection Plugin for Minecraft**

[![Version](https://img.shields.io/badge/version-1.0.0-blue?style=for-the-badge)](https://github.com)
[![Paper](https://img.shields.io/badge/Paper-1.20.1+-green?style=for-the-badge)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-17+-orange?style=for-the-badge)](https://adoptium.net)
[![License](https://img.shields.io/badge/license-MIT-purple?style=for-the-badge)](LICENSE)

*A powerful, security-hardened authentication plugin with bcrypt hashing, session management, bot protection, and proxy support.*

[Features](#-features) • [Installation](#-installation) • [Commands](#-commands) • [Configuration](#-configuration) • [Security](#-security)

</div>

---

## ✨ Features

| Feature | Description |
|--------|-------------|
| 🔑 **Password Auth** | bcrypt-hashed passwords with configurable cost factor |
| 🎫 **Session System** | Auto-login on rejoin — no password needed within session window |
| 🤖 **Bot Captcha** | Unicode homoglyph + Zero-Width Character captcha, AI/OCR-resistant |
| 🌍 **Auth World** | Dedicated login lobby with automatic location save & restore |
| 👑 **Admin 2FA** | Second authentication factor for OPs and staff members |
| 🔗 **Proxy Support** | Velocity modern forwarding & BungeeCord with HMAC-signed messages |
| 🛡️ **Brute-Force Protection** | IP-based fail counter with automatic timed block |
| ⚡ **Rate Limiting** | Sliding-window limits on login, register, and join events |
| 📋 **Audit Log** | Full event log to file for security monitoring |
| 🎨 **Rich Effects** | Titles, boss bar, particles, sounds, blindness, freeze |
| 📊 **PlaceholderAPI** | Placeholders for scoreboards and other plugins |
| ⚖️ **LuckPerms** | Temporary group assignment until authenticated |

---

## 📋 Requirements

- **Paper** 1.20.1+ (or any fork)
- **Java** 17 or higher
- **Optional:** Multiverse-Core *(for auth-world)*
- **Optional:** PlaceholderAPI
- **Optional:** LuckPerms

---

## 🚀 Installation

1. Download `SafeAuth-1.0.0.jar` and place it in your `/plugins/` folder
2. Start the server — `config.yml` and `messages.yml` will be generated automatically
3. Configure the plugin (see [Configuration](#-configuration))
4. Restart the server or run `/safeauth reload`
5. Set your admin password: `/safeauth setadminpass <password>`

### Auth World Setup *(optional)*
```
1. /mv create auth_world normal        ← create the world via Multiverse
2. Set auth-world.enabled: true        ← in config.yml
3. Set spawn coordinates               ← auth-world.spawn.x/y/z
4. Restart the server
```
> ⚠️ Multiverse-Core must load **before** SafeAuth. Add it to `depend` or `softdepend` if needed.

---

## 🔄 How It Works

```
Player joins
    │
    ├─► IP blocked?          → Kicked instantly (pre-login)
    ├─► Join flood from IP?  → Kicked (bot protection)
    ├─► Valid session?       → Auto-authenticated ✓
    │
    ├─► Restrictions applied (invisible, frozen, no chat/commands)
    ├─► Auth-world enabled?  → Location saved → Teleport to lobby
    ├─► Captcha enabled?     → Must solve captcha first
    │
    ├─► New player → /register    Existing player → /login
    │
    ├─► Wrong password → fail counter++
    │       Account lockout after N attempts
    │       IP block after N IP-level attempts
    │
    ├─► Correct + OP/staff → /adminlogin required (2FA)
    │
    └─► Authenticated ✓
            Restrictions removed
            Teleported back to saved location
            Session token created (HMAC-SHA256, UUID+IP bound)
```

---

## 💬 Commands

### Player Commands

| Command | Description |
|---------|-------------|
| `/login <password>` | Log in to your account |
| `/l <password>` | Alias for `/login` |
| `/register <password> <password>` | Create a new account |
| `/reg <password> <password>` | Alias for `/register` |
| `/adminlogin <password>` | Second-factor auth for OPs and staff |
| `/changepassword <old> <new>` | Change your password |
| `/logout` | Log out and invalidate your session |

### Admin Commands
*Requires `safeauth.admin` permission*

| Command | Description |
|---------|-------------|
| `/safeauth reload` | Reload config and messages |
| `/safeauth info <player>` | View player account details |
| `/safeauth resetpass <player> <password>` | Reset a player's password |
| `/safeauth unregister <player>` | Delete a player's account |
| `/safeauth forcelogin <player>` | Force-authenticate a player |
| `/safeauth setadminpass <password>` | Set the global admin (2FA) password |
| `/safeauth kick` | Kick all unauthenticated players |

> 🔒 All commands containing a password are **masked in server logs** via a Log4j2 filter. Passwords never appear in console output or log files.

---

## 🔑 Permissions

| Permission | Description |
|------------|-------------|
| `safeauth.admin` | Access to all admin commands |
| `safeauth.staff` | Triggers 2FA requirement on login |
| `safeauth.bypass` | Skip authentication entirely |
| `safeauth.logout` | Allow use of `/logout` |

---

## ⚙️ Configuration

Below is a full reference for every option in `config.yml`.

---

### 🔧 `general`

```yaml
general:
  language: ru
  prefix: <gradient:#FF6B6B:#FFE66D>SafeAuth</gradient> <dark_gray>»</dark_gray>
  notify-admins: true
  verbose-logging: false
  debug: false
```

| Key | Default | Description |
|-----|---------|-------------|
| `language` | `ru` | Interface language: `ru` or `en`. Affects default messages.yml values. |
| `prefix` | gradient text | Chat prefix before all plugin messages. Supports [MiniMessage](https://docs.advntr.dev/minimessage/format.html) format. |
| `notify-admins` | `true` | Send security alerts (brute-force, IP change, admin login) to players with `safeauth.admin`. |
| `verbose-logging` | `false` | Log every auth event to console. Useful for debugging — keep `false` in production. |
| `debug` | `false` | Extra internal debug output. Development use only. |

---

### 🗄️ `database`

```yaml
database:
  type: sqlite
  sqlite:
    file: safeauth.db
  mysql:
    host: localhost
    port: 3306
    database: safeauth
    username: root
    password: password
    pool-size: 10
    connection-timeout: 30000
    max-lifetime: 1800000
```

| Key | Default | Description |
|-----|---------|-------------|
| `type` | `sqlite` | Storage backend: `sqlite` (single server) or `mysql` (multi-server proxy). |
| `sqlite.file` | `safeauth.db` | SQLite file path relative to the plugin folder. |
| `mysql.host/port` | `localhost:3306` | MySQL server address. |
| `mysql.database` | `safeauth` | Database name. Tables are created automatically. |
| `mysql.pool-size` | `10` | Connection pool size. 5–10 is enough for most servers. |
| `mysql.connection-timeout` | `30000` | Ms to wait before a connection attempt fails (30 sec). |
| `mysql.max-lifetime` | `1800000` | Max connection lifetime in ms (30 min). Must be less than MySQL `wait_timeout`. |

> ⚠️ **Multi-server proxy setup:** Always use `type: mysql` so all backend servers share the same player data.

---

### 🔗 `proxy`

```yaml
proxy:
  enabled: false
  mode: velocity
  trust-proxy-auth: false
  shared-secret: "CHANGE_ME_USE_A_LONG_RANDOM_STRING_HERE"
  trusted-proxy-addresses:
    - "127.0.0.1"
    - "::1"
```

| Key | Default | Description |
|-----|---------|-------------|
| `enabled` | `false` | Enable proxy mode. Set `true` if behind Velocity or BungeeCord. |
| `mode` | `velocity` | Proxy type: `velocity` (recommended, auto IP forwarding) or `bungee`. |
| `trust-proxy-auth` | `false` | If `true`, a `ForceAuth` message from the proxy bypasses authentication. Only enable on a dedicated auth-server setup. |
| `shared-secret` | placeholder | HMAC-SHA256 secret for signing proxy→backend plugin messages. Must match on all backend servers and the companion plugin. **Change this!** |
| `trusted-proxy-addresses` | `127.0.0.1, ::1` | Only plugin channel messages from these IPs are accepted. All others are rejected and logged. |

> 🔐 **Security:** Every proxy message must carry a valid HMAC-SHA256 signature computed with `shared-secret`. Invalid signatures and messages from untrusted IPs are rejected and written to the audit log.

**Velocity setup:**
```
1. config.toml → player-info-forwarding-mode = "modern"
2. paper-global.yml → proxies.velocity.enabled: true + secret
3. Set mode: velocity in SafeAuth config
```

**BungeeCord setup:**
```
1. BungeeCord config.yml → ip_forward: true
2. spigot.yml → settings.bungeecord: true
3. Install SafeAuth-Proxy.jar on the BungeeCord proxy
4. Set mode: bungee in SafeAuth config
```

---

### 🤖 `captcha`

```yaml
captcha:
  enabled: false
  length: 6
  max-attempts: 3
  separator-mode: zwc
  custom-separators: []
```

| Key | Default | Description |
|-----|---------|-------------|
| `enabled` | `false` | Show a captcha challenge on join before `/login` or `/register`. Recommended `true` on public servers. |
| `length` | `6` | Captcha code length (4–10). Recommended: 5–6. |
| `max-attempts` | `3` | Wrong attempts before kick. `0` = unlimited (not recommended). |
| `separator-mode` | `zwc` | Characters inserted between letters to defeat OCR/AI solvers. See table below. |
| `custom-separators` | `[]` | Strings inserted between characters when mode is `custom`. |

**Separator modes:**

| Mode | Description |
|------|-------------|
| `none` | No separators — only visual homoglyph substitution |
| `zwc` | Invisible Zero-Width Unicode characters *(default)*. If you see squares in your client, switch to `none`. |
| `custom` | Your own strings from `custom-separators`, rotated between characters |

**Custom separator examples:**
```yaml
custom-separators: [" "]         # A B C D E
custom-separators: [" · "]       # A · B · C · D
custom-separators: [" | ", " "]  # A | B C | D
```

> Players **never** need to type separators — they are stripped automatically before verification.

---

### 🔒 `auth`

```yaml
auth:
  timeout: 60
  max-attempts: 5
  lockout-time: 300
  session-duration: 3600
  session-by-ip: true
  premium-skip: false
  min-password-length: 6
  max-password-length: 32
  require-number: false
  require-uppercase: false
  require-special: false
  bcrypt-rounds: 12
```

| Key | Default | Description |
|-----|---------|-------------|
| `timeout` | `60` | Seconds to authenticate before kick. Warning sent 10s before. `0` = disabled. |
| `max-attempts` | `5` | Failed `/login` attempts before account lockout. |
| `lockout-time` | `300` | Seconds the account is locked after too many failures (5 min). |
| `session-duration` | `3600` | Session lifetime in seconds (1 hour). `0` = disable sessions. |
| `session-by-ip` | `true` | Bind session to player's IP. IP change = session invalidated. **Strongly recommended.** |
| `premium-skip` | `false` | Allow licensed players to skip authentication. |
| `min-password-length` | `6` | Minimum password length. |
| `max-password-length` | `32` | Maximum password length. |
| `require-number` | `false` | Password must contain at least one digit. |
| `require-uppercase` | `false` | Password must contain at least one uppercase letter. |
| `require-special` | `false` | Password must contain at least one special character. |
| `bcrypt-rounds` | `12` | bcrypt cost factor (10–14). Higher = more secure, slower. **Do not go below 10.** |

---

### 👑 `admin-auth`

```yaml
admin-auth:
  enabled: true
  staff-permission: safeauth.staff
  admin-password-hash: ""
  require-personal-first: true
  notify-on-admin-login: true
```

| Key | Default | Description |
|-----|---------|-------------|
| `enabled` | `true` | Enable 2FA for OPs and staff. After `/login`, they must also run `/adminlogin`. |
| `staff-permission` | `safeauth.staff` | Permission that triggers 2FA (in addition to OP status). |
| `admin-password-hash` | *(empty)* | bcrypt hash of the global admin password. **Never set plain text here.** Use `/safeauth setadminpass <password>` instead. |
| `require-personal-first` | `true` | Player must complete `/login` before `/adminlogin`. Keep `true`. |
| `notify-on-admin-login` | `true` | Notify online admins when a staff member completes 2FA. |

---

### 🚫 `restrictions`

```yaml
restrictions:
  block-movement: true
  block-commands: true
  allowed-commands:
    - login
    - register
    - l
    - reg
    - adminlogin
  block-chat: true
  block-inventory: true
  block-item-pickup: true
  block-item-drop: true
  block-damage-in: true
  block-damage-out: true
  block-interaction: true
  block-portal: true
  invisible-until-auth: true
  freeze-player: true
```

| Key | Default | Description |
|-----|---------|-------------|
| `block-movement` | `true` | Prevent walking, jumping, position changes. |
| `block-commands` | `true` | Block all commands except those in `allowed-commands`. |
| `allowed-commands` | *auth commands* | Commands (without `/`) permitted before authentication. |
| `block-chat` | `true` | Prevent sending chat messages. |
| `block-inventory` | `true` | Prevent opening or modifying inventory. |
| `block-item-pickup/drop` | `true` | Prevent picking up or dropping items. |
| `block-damage-in/out` | `true` | Prevent taking or dealing damage. |
| `block-interaction` | `true` | Prevent clicking blocks, buttons, chests, entities, etc. |
| `block-portal` | `true` | Prevent entering portals. |
| `invisible-until-auth` | `true` | Hide unauthenticated player from all others. |
| `freeze-player` | `true` | Completely immobilize the player (no gravity, no knockback). |

---

### 🌍 `auth-world`

```yaml
auth-world:
  enabled: false
  world-name: auth_world
  spawn:
    x: 0.5
    y: 64.0
    z: 0.5
    yaw: 0.0
    pitch: 0.0
  restore-location: true
  teleport-after-auth: true
  teleport-to-spawn-if-no-location: true
```

| Key | Default | Description |
|-----|---------|-------------|
| `enabled` | `false` | Enable the auth-world feature. Requires Multiverse-Core. |
| `world-name` | `auth_world` | Name of the login lobby world. Must be created and loaded by Multiverse **before** SafeAuth starts. |
| `spawn.x/y/z/yaw/pitch` | `0.5, 64, 0.5` | Spawn point inside the auth world. Set to a safe, enclosed area. |
| `restore-location` | `true` | Save player's location before teleport and restore it after login. |
| `teleport-after-auth` | `true` | Teleport back to saved location after authentication. |
| `teleport-to-spawn-if-no-location` | `true` | Send player to main world spawn if no saved location exists (first join). |

**How location saving works:**
```
1. Player joins          → Location captured synchronously (real position)
2. Teleport to auth world → Location saved to database immediately
3. Player authenticates  → Teleported back to saved position
4. Database cleared      → Next join will capture fresh position
```
> 🔁 If the server crashes while a player is in the auth world, their position is safe in the database and will be restored on next login.

---

### 🎨 `effects`

```yaml
effects:
  join-title:
    enabled: true
    title: <red><bold>⚠ AUTHORIZATION</bold></red>
    subtitle-login: '<yellow>Type: <white>/login <password>'
    subtitle-register: '<yellow>Type: <white>/register <password> <password>'
    fade-in: 20
    stay: 200
    fade-out: 20
  success-title:
    enabled: true
    title: <gradient:#00FF87:#60EFFF>✔ WELCOME</gradient>
    subtitle: <gray>Authentication successful
  bossbar:
    enabled: true
    color: RED
    style: SOLID
    text: <red>⚠ Enter your password!
  particles:
    enabled: true
    type: FLAME
    count: 3
    interval: 10
  sounds:
    on-join:     { enabled: true, sound: BLOCK_NOTE_BLOCK_BASS, volume: 1.0, pitch: 0.5 }
    on-success:  { enabled: true, sound: ENTITY_PLAYER_LEVELUP, volume: 1.0, pitch: 1.2 }
    on-fail:     { enabled: true, sound: ENTITY_VILLAGER_NO, volume: 1.0, pitch: 0.8 }
    on-register: { enabled: true, sound: UI_TOAST_CHALLENGE_COMPLETE, volume: 1.0, pitch: 1.0 }
  gui:
    enabled: true
    title: 🔐 Authorization
  blindness:
    enabled: true
  slowness:
    enabled: true
    level: 5
```

| Key | Description |
|-----|-------------|
| `join-title` | Title shown while player needs to authenticate. Supports [MiniMessage](https://docs.advntr.dev/minimessage/format.html). `fade-in/stay/fade-out` in ticks (20 = 1s). |
| `success-title` | Title shown after successful authentication. |
| `bossbar` | Boss bar at top of screen. `color`: RED/BLUE/GREEN/YELLOW/PURPLE/PINK/WHITE. `style`: SOLID/SEGMENTED_6/10/12/20. |
| `particles` | Particles around unauthenticated player. `type`: any [Bukkit Particle](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/Particle.html) name. |
| `sounds` | Sounds on join/success/fail/register. `volume`: 0.0–1.0, `pitch`: 0.5–2.0. |
| `gui.enabled` | Show chest GUI for password entry instead of typing a command. |
| `blindness` | Apply blindness effect while unauthenticated. |
| `slowness.level` | Slowness strength 1–5 (5 = unable to move). |

---

### 🛡️ `security`

```yaml
security:
  ip-bruteforce-protection: true
  max-ip-attempts: 10
  ip-block-duration: 600
  ip-change-action: NOTIFY
  log-to-file: true
  log-file: safeauth-audit.log
  alert-on-suspicious: true
  rate-limit:
    max-commands-per-window: 3
    command-window-ms: 5000
    max-joins-per-window: 5
    join-window-ms: 10000
    max-registers-per-window: 3
    register-window-ms: 60000
```

| Key | Default | Description |
|-----|---------|-------------|
| `ip-bruteforce-protection` | `true` | Track failed attempts per IP and block offenders. |
| `max-ip-attempts` | `10` | Failed attempts from one IP before block. |
| `ip-block-duration` | `600` | Seconds the IP stays blocked (10 min). |
| `ip-change-action` | `NOTIFY` | Action when player joins from a new IP: `NOTIFY` / `KICK` / `IGNORE`. |
| `log-to-file` | `true` | Write all auth events to audit log. |
| `log-file` | `safeauth-audit.log` | Audit log filename (relative to plugin folder). |
| `alert-on-suspicious` | `true` | Broadcast to admins when an IP is blocked. |
| `rate-limit.max-commands-per-window` | `3` | Max `/login` or `/adminlogin` commands per player per window. |
| `rate-limit.command-window-ms` | `5000` | Command rate-limit window duration (5 sec). |
| `rate-limit.max-joins-per-window` | `5` | Max connections from one IP per window. Protects against bot floods. |
| `rate-limit.join-window-ms` | `10000` | Join flood window duration (10 sec). |
| `rate-limit.max-registers-per-window` | `3` | Max registrations from one IP per window. |
| `rate-limit.register-window-ms` | `60000` | Registration flood window duration (60 sec). |

---

### 🔌 `integrations`

```yaml
integrations:
  placeholderapi:
    enabled: true
  luckperms:
    enabled: true
    unauthenticated-group: ''
    restore-after-auth: true
```

| Key | Description |
|-----|-------------|
| `placeholderapi.enabled` | Register PlaceholderAPI placeholders. Requires PlaceholderAPI plugin. |
| `luckperms.enabled` | Enable LuckPerms integration. |
| `luckperms.unauthenticated-group` | Group to assign before authentication. Leave empty to disable. |
| `luckperms.restore-after-auth` | Remove the temporary group after the player logs in. |

---

### 🧹 `maintenance`

```yaml
maintenance:
  cleanup-days: 0
  cleanup-on-startup: false
```

| Key | Default | Description |
|-----|---------|-------------|
| `cleanup-days` | `0` | Delete accounts unused for this many days. `0` = disabled. |
| `cleanup-on-startup` | `false` | Run cleanup check on server start. |

---

## 🔐 Security

### Architecture — Layers of Protection

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1 — Network                                       │
│  IP whitelist for proxy messages · Join flood protection │
├─────────────────────────────────────────────────────────┤
│  Layer 2 — Authentication                                │
│  bcrypt passwords · HMAC session tokens · Admin 2FA     │
├─────────────────────────────────────────────────────────┤
│  Layer 3 — Rate Limiting                                 │
│  Login · Register · Join — sliding window per IP/UUID   │
├─────────────────────────────────────────────────────────┤
│  Layer 4 — Monitoring                                    │
│  Audit log · Admin alerts · IP block notifications      │
└─────────────────────────────────────────────────────────┘
```

### Security Features Detail

**🔑 bcrypt Password Hashing**
Passwords are never stored in plain text. bcrypt with configurable rounds (10–14) makes brute-forcing stolen hashes computationally prohibitive.

**🎫 HMAC-SHA256 Session Tokens**
Session tokens are cryptographically signed with a per-runtime 256-bit secret key (regenerated on each server start). Tokens are bound to both the player UUID and IP address simultaneously. A changed IP immediately invalidates the session.

**📡 HMAC-Signed Proxy Messages**
All plugin channel messages from the proxy (ForwardIP, ForceAuth) require a valid HMAC-SHA256 signature. Messages from IPs not in `trusted-proxy-addresses` or with an invalid signature are rejected and written to the audit log. Each message includes a nonce with 60-second TTL to prevent replay attacks.

**📝 Log4j2 Password Masking**
A filter installed directly in the Log4j2 root logger intercepts log entries containing auth commands **before they are written anywhere** — console, file, or any appender. Password arguments are replaced with `*****` in all output. This addresses the fact that Paper logs commands through Log4j2 before any Bukkit event fires.

**🧱 IP Brute-Force Protection**
Failed login attempts are tracked per IP address. After exceeding `max-ip-attempts`, the IP is blocked for `ip-block-duration` seconds. The counter is **not reset on a successful login** — this closes the counter-reset exploit where an attacker uses their own account to clear the block.

**⚡ Rate Limiting**
Three independent sliding-window rate limiters:
- **Command** — limits `/login` spam per player
- **Join** — limits connection floods per IP
- **Register** — limits account creation floods per IP

**🤖 Unicode Captcha**
Captcha codes use visually identical Unicode homoglyphs (Cyrillic А instead of Latin A, Greek Η instead of H, etc.) and optional invisible Zero-Width Characters between letters. This defeats OCR-based and neural-network-based automated solving while remaining readable to human players.

---

## ✅ Quick Setup Checklist

```
[ ] Place JAR in /plugins/ and start the server
[ ] Set database type (sqlite for single server, mysql for proxy network)
[ ] Run: /safeauth setadminpass <strong_password>
[ ] Configure proxy section if running behind Velocity/BungeeCord
    [ ] Set a strong proxy.shared-secret (min 32 characters)
    [ ] Set proxy.trusted-proxy-addresses to your proxy's IP
[ ] Enable captcha on public servers: captcha.enabled: true
[ ] Create auth_world with Multiverse if you want a login lobby
[ ] Adjust password policy (min-password-length, require-number, etc.)
[ ] Test: join → register → /logout → rejoin → verify session works
```

---

## 📁 File Structure

```
plugins/SafeAuth/
├── config.yml          ← Main configuration
├── messages.yml        ← All plugin messages (MiniMessage format)
├── safeauth.db         ← SQLite database (if using sqlite)
└── safeauth-audit.log  ← Audit log (if log-to-file: true)
```

---

<div align="center">

**SafeAuth Documentation** | Version 1.0.0 | Author: Wolfox

</div>
