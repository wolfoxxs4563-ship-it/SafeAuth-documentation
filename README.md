<div align="center">

```
 ░██████╗░█████╗░███████╗███████╗░█████╗░██╗░░░██╗████████╗██╗░░██╗
 ██╔════╝██╔══██╗██╔════╝██╔════╝██╔══██╗██║░░░██║╚══██╔══╝██║░░██║
 ╚█████╗░███████║█████╗░░█████╗░░███████║██║░░░██║░░░██║░░░███████║
 ░╚═══██╗██╔══██║██╔══╝░░██╔══╝░░██╔══██║██║░░░██║░░░██║░░░██╔══██║
 ██████╔╝██║░░██║██║░░░░░███████╗██║░░██║╚██████╔╝░░░██║░░░██║░░██║
 ╚═════╝░╚═╝░░╚═╝╚═╝░░░░░╚══════╝╚═╝░░╚═╝░╚═════╝░░░╚═╝░░░╚═╝░░╚═╝
```

**Professional account protection for Minecraft servers**

![Version](https://img.shields.io/badge/version-2.0.0-6366f1?style=for-the-badge&logo=github)
![Paper](https://img.shields.io/badge/Paper-1.20+-22c55e?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-17+-f59e0b?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-ec4899?style=for-the-badge)

</div>

---

## 📋 Table of Contents

- [Features](#-features)
- [Requirements & Installation](#-requirements--installation)
- [Commands](#-commands)
- [Permissions](#-permissions)
- [Configuration Reference](#-configuration-reference)
  - [general](#general)
  - [database](#database)
  - [proxy](#proxy)
  - [auth](#auth)
  - [admin-auth](#admin-auth)
  - [two-factor](#two-factor)
  - [captcha](#captcha)
  - [restrictions](#restrictions)
  - [auth-world](#auth-world)
  - [effects](#effects)
  - [security](#security)
  - [integrations](#integrations)
  - [maintenance](#maintenance)
  - [features](#features-1)
- [messages.yml](#messagesyml)
- [PlaceholderAPI Variables](#placeholderapi-variables)
- [FAQ](#faq)

---

## ✨ Features

<table>
<tr>
<td width="50%">

**🔐 Authentication**
- Password-based login & registration (BCrypt hashing)
- IP-bound sessions — automatic re-login without a password
- Per-player 2FA with any TOTP app (Google Authenticator, Aegis, Authy…)
- Admin double-authentication for OPs and staff
- `/logout` invalidates session and kicks the player

</td>
<td width="50%">

**🛡️ Security**
- Brute-force lockout after N failed attempts
- IP-based rate limiting for joins, commands, registrations
- IP-change detection with configurable action (notify / kick)
- Full security audit log to file
- HMAC-signed proxy messages (Velocity & BungeeCord)

</td>
</tr>
<tr>
<td width="50%">

**🌍 Auth World**
- Separate lobby world for unauthenticated players
- Player's real location saved to DB before teleport, restored after login
- Permanent 3×3 barrier floor at spawn (for void worlds)
- Always teleports to configured spawn coords on every join

</td>
<td width="50%">

**🎲 Captcha**
- **Text** — obfuscated code with Unicode homoglyphs & zero-width chars
- **Action** — physical in-game actions bots cannot script: jump, crouch, look direction, or combo
- Configurable timeouts, attempts, visual feedback

</td>
</tr>
</table>

---

## 📦 Requirements & Installation

### Requirements

| Component | Version |
|-----------|---------|
| Paper (or Spigot) | **1.20+** |
| Java | **17+** |
| Multiverse-Core | Only needed for Auth World |
| PlaceholderAPI | Optional — for PAPI variables |

> ⚠️ **Paper is strongly recommended.** Action captcha (jump detection) uses Paper's `PlayerJumpEvent` which is not available on Spigot. On Spigot, jump captcha will not work.

### Installation Steps

```
1.  Drop SafeAuth.jar into your plugins/ folder
2.  Start the server — config.yml and messages.yml are generated automatically
3.  Edit plugins/SafeAuth/config.yml to your needs
4.  Restart or run /safeauth reload
```

**Auth World quick setup:**
```
1.  Install Multiverse-Core
2.  Run:  /mv create auth_world normal
3.  In config.yml set:  auth-world.enabled: true
4.  Set spawn coordinates under auth-world.spawn
5.  Restart the server
```

---

## 🎮 Commands

### Player Commands

| Command | Aliases | Description |
|---------|---------|-------------|
| `/register <password> <confirm>` | `/reg` | Create an account |
| `/login <password>` | `/l`, `/log` | Log in to your account |
| `/logout` | — | Log out and invalidate your session (you will be kicked) |
| `/changepassword <old> <new>` | `/cp`, `/changepass` | Change your password |
| `/2fa setup` | `/totp setup` | Begin 2FA setup — get your secret key |
| `/2fa confirm <code>` | `/totp confirm` | Confirm and activate 2FA |
| `/2fa code <code>` | — | Enter your one-time code after `/login` |
| `/2fa disable <password>` | — | Disable 2FA (requires current password) |
| `/2fa status` | — | Show whether 2FA is active on your account |

### Admin Commands

| Command | Permission | Description |
|---------|-----------|-------------|
| `/safeauth reload` | `safeauth.admin` | Reload config.yml and messages.yml without restart |
| `/safeauth forcelogin <player>` | `safeauth.admin` | Force-authenticate a player |
| `/safeauth unregister <player>` | `safeauth.admin` | Delete an account (player can register again) |
| `/safeauth setpassword <player> <pass>` | `safeauth.admin` | Set a new password for a player |
| `/safeauth setadminpass <password>` | `safeauth.admin` | Set the global admin second-factor password |
| `/safeauth disable2fa <player>` | `safeauth.admin` | Reset a player's 2FA (lost phone recovery) |
| `/safeauth info <player>` | `safeauth.admin` | Show account details |
| `/safeauth stats` | `safeauth.admin` | Show server-wide auth statistics |
| `/safeauth kick <player>` | `safeauth.admin` | Kick an unauthenticated player |
| `/adminlogin <password>` | `safeauth.staff` or OP | Enter the second authentication factor |

---

## 🔑 Permissions

| Permission | Default | Description |
|-----------|:-------:|-------------|
| `safeauth.admin` | OP | Access to all `/safeauth` admin commands |
| `safeauth.bypass` | `false` | Skip authentication entirely on join |
| `safeauth.staff` | `false` | Requires double-authentication via `/adminlogin` |

---

## ⚙️ Configuration Reference

File: `plugins/SafeAuth/config.yml`

All text values support **MiniMessage** formatting:

```
Colors:    <red>  <green>  <gold>  <aqua>  <#FF6B6B>
Gradient:  <gradient:#FF6B6B:#FFE66D>text</gradient>
Style:     <bold>  <italic>  <underlined>
Click:     <click:run_command:/login>text</click>
Hover:     <hover:show_text:'info'>text</hover>
Full docs: https://docs.advntr.dev/minimessage/format.html
```

---

### `general`

```yaml
general:
  language: en
  prefix: "<gradient:#FF6B6B:#FFE66D><bold>SafeAuth</bold></gradient> <dark_gray>»</dark_gray>"
  notify-admins: true
  verbose-logging: false
  debug: false
```

| Key | Values | Default | Description |
|-----|--------|:-------:|-------------|
| `language` | `en` / `ru` | `en` | Language for all messages. Switches the active messages.yml section. |
| `prefix` | MiniMessage string | gradient text | Prefix prepended to every plugin message in chat. |
| `notify-admins` | `true` / `false` | `true` | Send suspicious event alerts to online players with `safeauth.admin`. |
| `verbose-logging` | `true` / `false` | `false` | Log every auth event to console. Useful for debugging, noisy in production. |
| `debug` | `true` / `false` | `false` | Enable full debug output to console **and** `plugins/SafeAuth/debug.log`. For developers only — generates large log files. |

---

### `database`

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
    password: ""
    pool-size: 10
    connection-timeout: 30000
    max-lifetime: 1800000
```

| Key | Description |
|-----|-------------|
| `type` | `sqlite` — single file, zero configuration, ideal for standalone servers. `mysql` — external database, **required** for networks with multiple backend servers behind a proxy. |
| `sqlite.file` | Filename of the SQLite database inside `plugins/SafeAuth/`. |
| `mysql.host` | MySQL server hostname or IP address. |
| `mysql.port` | MySQL port. Default is `3306`. |
| `mysql.database` | Name of the MySQL database (must exist before starting). |
| `mysql.username` / `password` | MySQL credentials. |
| `mysql.pool-size` | HikariCP connection pool size. `5`–`10` is appropriate for most servers. |
| `mysql.connection-timeout` | Milliseconds to wait when acquiring a connection before throwing an error. |
| `mysql.max-lifetime` | Maximum lifetime of a connection in the pool (ms). Must be less than MySQL's `wait_timeout` (default 28800000 ms = 8h). Recommended: `1800000` (30 min). |

> **Multi-server networks:** all backend servers must share one MySQL database. SQLite is a local file and cannot be shared across servers.

---

### `proxy`

```yaml
proxy:
  enabled: false
  mode: velocity
  trust-proxy-auth: false
  channel: safeauth:main
  shared-secret: "CHANGE_ME_USE_A_LONG_RANDOM_STRING_HERE"
  trusted-proxy-addresses:
    - "127.0.0.1"
    - "::1"
```

| Key | Description |
|-----|-------------|
| `enabled` | Set `true` when running behind Velocity or BungeeCord. Enables IP-forwarding support. |
| `mode` | `velocity` (recommended, uses Modern Forwarding) or `bungee`. |
| `trust-proxy-auth` | If `true`, the proxy can send a `ForceAuth` message to bypass authentication entirely. **Only enable on a dedicated auth-proxy network.** Leave `false` for standard setups. |
| `shared-secret` | HMAC-SHA256 signing key for messages sent from the proxy to this backend. Must match on every backend server and the companion proxy plugin. Leave `"CHANGE_ME…"` and the plugin will auto-generate a secure random key on first launch. |
| `trusted-proxy-addresses` | List of IP addresses from which proxy messages are accepted. Any message arriving from an unlisted IP is rejected. If your proxy runs on the same machine, `127.0.0.1` is sufficient. |

**Velocity setup checklist:**
1. In `velocity.toml` set `player-info-forwarding-mode = "MODERN"`
2. On each backend set `proxy.enabled: true` and `proxy.mode: velocity`
3. Ensure `paper-global.yml` → `proxies.velocity.enabled: true` with the matching forwarding secret

---

### `auth`

```yaml
auth:
  timeout: 60
  registration-enabled: true
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
|-----|:-------:|-------------|
| `timeout` | `60` | Seconds a player has to authenticate before being kicked. `0` disables the timeout. |
| `registration-enabled` | `true` | Allow new players to create accounts. Set `false` for login-only servers (e.g. transferred player base, whitelist-only). Unregistered players are kicked with a configurable message. |
| `max-attempts` | `5` | Number of wrong password attempts before the account is temporarily locked. |
| `lockout-time` | `300` | Seconds the account stays locked after hitting `max-attempts`. |
| `session-duration` | `3600` | How long (in seconds) a successful login session lasts. While the session is valid, the player logs in automatically on join without entering a password. `0` = sessions disabled (password required every join). |
| `session-by-ip` | `true` | Bind the session to the player's IP address. If the player joins from a different IP the session is invalidated and they must log in again. Recommended to keep `true`. |
| `premium-skip` | `false` | If `true`, players with a valid Mojang account (online-mode) skip authentication automatically. Only works on hybrid servers (e.g. AuthMe/Geyser setups). |
| `min-password-length` | `6` | Minimum number of characters required for a password. |
| `max-password-length` | `32` | Maximum number of characters allowed. |
| `require-number` | `false` | Password must contain at least one digit (0–9). |
| `require-uppercase` | `false` | Password must contain at least one uppercase letter (A–Z). |
| `require-special` | `false` | Password must contain at least one special character (`!@#$%^&*` etc.). |
| `bcrypt-rounds` | `12` | BCrypt hashing cost factor. Higher = more CPU time per hash = harder to brute-force offline. `12` ≈ 250–400 ms per hash. `14` ≈ 1–1.5 s. Do not exceed `14` on busy servers (simultaneous logins will cause lag spikes). |

---

### `admin-auth`

Double-authentication for operators and staff. After passing normal `/login`, staff must also enter a shared admin password via `/adminlogin`.

```yaml
admin-auth:
  enabled: true
  staff-permission: safeauth.staff
  admin-password-hash: ""
  require-personal-first: true
  notify-on-admin-login: true
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `enabled` | `true` | Enable the second-factor system for OPs and staff. |
| `staff-permission` | `safeauth.staff` | Permission node that marks a player as staff and requires `/adminlogin`. OPs are **always** included regardless of this permission. |
| `admin-password-hash` | `""` | BCrypt hash of the global admin password. **Never enter a plain-text password here.** Use `/safeauth setadminpass <password>` to set it — the plugin hashes and saves it automatically. |
| `require-personal-first` | `true` | Staff must complete their personal `/login` before `/adminlogin` is accepted. Recommended. |
| `notify-on-admin-login` | `true` | Broadcast a message to all online admins when a staff member successfully completes double-auth. |

---

### `two-factor`

TOTP-based two-factor authentication compatible with any standard authenticator app.

```yaml
two-factor:
  totp:
    enabled: true
    server-secret: "CHANGE_ME_WILL_BE_AUTO_GENERATED"
    require-for-all: false
    require-for-permission: ""
    code-timeout: 120
    remind-on-login: true
    remind-interval: 5
  visual:
    title:
      enabled: true
      fade-in:  10
      stay:     200
      fade-out: 10
    action-bar:
      enabled: true
    chat-message:
      enabled: true
    sound:
      enabled: true
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `totp.enabled` | `true` | Enable the 2FA system server-wide. Individual players still opt in unless `require-for-all` is set. |
| `totp.server-secret` | auto | AES encryption key for TOTP secrets stored in the database. Leave as `"CHANGE_ME…"` — the plugin generates a secure key automatically on first launch and writes it back to config. **Never change this after players have set up 2FA** — it will break all existing 2FA links. |
| `totp.require-for-all` | `false` | `true` = every player must set up 2FA to be able to log in. Players without 2FA configured will be halted at the 2FA-setup prompt after entering their password. |
| `totp.require-for-permission` | `""` | Require 2FA only for players holding a specific permission node. Example: `"safeauth.staff"`. Leave `""` to not enforce 2FA by permission. |
| `totp.code-timeout` | `120` | Seconds the player has to enter their one-time code after `/login`. |
| `totp.remind-on-login` | `true` | Show a reminder message to players who haven't set up 2FA yet. |
| `totp.remind-interval` | `5` | Show the reminder once every N logins. `0` = show on every single login. |
| `visual.*` | — | Toggle and time the title, action bar, chat message, and sound shown when 2FA is required. `fade-in`, `stay`, `fade-out` are in **ticks** (20 ticks = 1 second). |

**Player 2FA setup flow:**
```
/login <password>        → authenticate with password first
/2fa setup               → receive the 16-character secret key
  (open authenticator app → add account → enter the key)
/2fa confirm <6 digits>  → activate 2FA

On subsequent logins:
/login <password>
/2fa code <6 digits>     → enter current code from the app
```

**Compatible apps:** Google Authenticator, Aegis (recommended, open-source), Authy, Microsoft Authenticator, Bitwarden, 1Password, and any standard TOTP application.

---

### `captcha`

Captcha is presented **before** `/login` or `/register`. It protects against automated bot attacks.

```yaml
captcha:
  enabled: false
  type: text
```

#### Text Captcha (`type: text`)

```yaml
  length: 5
  max-attempts: 3
  obfuscation:
    homoglyph-chance: 65
    separator-mode: zwc
    custom-separators: []
  charset: "ACDEFGHJKLMNPQRTUVWXY3478"
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `length` | `5` | Number of characters in the generated code. Range: 4–10. Recommended: 5–6. |
| `max-attempts` | `3` | Wrong attempts allowed before the player is kicked. `0` = unlimited. |
| `obfuscation.homoglyph-chance` | `65` | Percentage chance (0–100) that each character is replaced with a visually identical character from a different Unicode script. This makes OCR and neural-network recognition harder. Players type what they see — the plugin automatically strips decorators before comparing. |
| `obfuscation.separator-mode` | `zwc` | What to insert between each code character. `none` = nothing (homoglyphs only, clean look). `zwc` = invisible Zero-Width Characters (default — recommended; makes copy-paste and OCR fail). `custom` = use the strings listed in `custom-separators`, cycling through them. |
| `obfuscation.custom-separators` | `[]` | Only used when `separator-mode: custom`. Example: `[" · ", " | "]` produces `A · B | C · D`. |
| `charset` | see config | Characters used to generate codes. The default excludes visually ambiguous characters: `I`/`1`/`l`, `O`/`0`, `S`/`5`, `Z`/`2`. |

#### Action Captcha (`type: action`)

```yaml
  action:
    type: RANDOM
    count-max: 3
    timeout: 30
    kick-on-timeout: true
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `action.type` | `RANDOM` | Which physical action to require. See table below. |
| `action.count-max` | `3` | Upper bound for the random repetition count. The actual number required is chosen randomly between 1 and this value on each captcha issue. |
| `action.timeout` | `30` | Seconds the player has to complete the action. |
| `action.kick-on-timeout` | `true` | `true` = kick if time runs out. `false` = issue a new captcha task instead. |

**Action types:**

| Value | Action | How to solve | Bot resistance |
|-------|--------|-------------|:--------------:|
| `JUMP` | Press Space N times | Tap the jump key | ★★★☆ |
| `SNEAK` | Press Shift N times | Tap the crouch key | ★★★☆ |
| `LOOK` | Face a compass direction | Rotate the camera | ★★★★ |
| `COMBO` | Jump then crouch | Both in sequence | ★★★★★ |
| `RANDOM` | Different every join | — | ★★★★★ |

> **Recommendation:** use `type: RANDOM` or `type: COMBO` for the strongest bot protection.

#### Captcha Visual Settings

```yaml
  visual:
    title:
      enabled: true
      fade-in:  10
      stay:     300
      fade-out: 10
    action-bar:
      enabled: true
      update-interval: 10
    bossbar:
      enabled: true
      color-text:   YELLOW
      color-action: GREEN
      style: PROGRESS
      animated: true
    chat-message:
      enabled: true
    effects:
      blindness: true
      nausea:    false
      particles:
        enabled: true
        type:     ENCHANTMENT_TABLE
        count:    5
        interval: 15
    sounds:
      on-captcha-show: { enabled: true, sound: BLOCK_NOTE_BLOCK_PLING, volume: 1.0, pitch: 0.8 }
      on-wrong:        { enabled: true, sound: ENTITY_VILLAGER_NO,      volume: 1.0, pitch: 0.8 }
      on-success:      { enabled: true, sound: ENTITY_EXPERIENCE_ORB_PICKUP, volume: 1.0, pitch: 1.2 }
```

| Key | Description |
|-----|-------------|
| `title.*` | Full-screen title shown during captcha. `fade-in`, `stay`, `fade-out` are in ticks (20 = 1 second). |
| `action-bar.*` | Text above the hotbar showing the captcha code or instruction. `update-interval` controls how often (in ticks) it is refreshed. |
| `bossbar.*` | Boss bar at the top of the screen. `color-text` = bar color for text captcha. `color-action` = color for action captcha. `style` = `PROGRESS` or `NOTCHED_6 / _10 / _12 / _20`. `animated: true` drains the bar over time to show remaining seconds. |
| `chat-message` | Whether to also print the code/instruction in the chat window. |
| `effects.blindness` | Apply Blindness potion effect during captcha. |
| `effects.nausea` | Apply Nausea (screen wobble) during captcha. |
| `effects.particles` | Spawn particles around the player. Particle type names: `ENCHANTMENT_TABLE`, `FLAME`, `HEART`, `SPELL_INSTANT`, etc. |
| `sounds.*` | Play a sound when captcha appears, on a wrong answer, and on success. Sound names use Bukkit's `Sound` enum (e.g. `ENTITY_PLAYER_LEVELUP`). |

---

### `restrictions`

Applied immediately when an unauthenticated player joins. All restrictions are automatically removed when the player successfully authenticates.

```yaml
restrictions:
  block-movement:    true
  block-commands:    true
  allowed-commands:
    - login
    - register
    - l
    - reg
    - log
    - adminlogin
    - al
    - 2fa
  block-chat:        true
  block-inventory:   true
  block-item-pickup: true
  block-item-drop:   true
  block-damage-in:   true
  block-damage-out:  true
  block-interaction: true
  block-portal:      true
  teleport-to-spawn: false
  invisible-until-auth: true
  freeze-player: true
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `block-movement` | `true` | Cancel any positional movement (walking, falling). Head rotation (camera look) is always allowed. |
| `block-commands` | `true` | Cancel all commands not listed in `allowed-commands`. |
| `allowed-commands` | see above | Commands (without the `/`) that players may use before authenticating. Add any additional commands your login flow requires. |
| `block-chat` | `true` | Cancel chat messages. Players still receive messages sent by the server. |
| `block-inventory` | `true` | Prevent the player from opening or clicking their inventory. |
| `block-item-pickup` | `true` | Prevent the player from picking up items from the ground. |
| `block-item-drop` | `true` | Prevent the player from dropping items. |
| `block-damage-in` | `true` | Unauthenticated player cannot take damage (fall, mobs, players). |
| `block-damage-out` | `true` | Unauthenticated player cannot deal damage. |
| `block-interaction` | `true` | Prevent interacting with blocks (doors, chests, buttons, signs). |
| `block-portal` | `true` | Prevent entering Nether or End portals. |
| `teleport-to-spawn` | `false` | Teleport the player to the world's spawn point on join (when Auth World is disabled). |
| `invisible-until-auth` | `true` | Hide the unauthenticated player from all other players. They are also hidden from tab-list until authenticated. |
| `freeze-player` | `true` | Completely freeze the player in place (no gravity, no knockback). Combined with `block-movement`, the player cannot move at all. |

---

### `auth-world`

A dedicated world players are teleported to on every join. After successful login they are returned to their original location.

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
  return-mode: saved
  return-world: world
  return-x:     0.5
  return-y:     64.0
  return-z:     0.5
  return-yaw:   0.0
  return-pitch: 0.0
  clear-inventory-during-auth: false
  spawn-barrier-enabled: false
  spawn-barrier-material: BARRIER
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `enabled` | `false` | Enable the Auth World system. Requires Multiverse-Core. |
| `world-name` | `auth_world` | Name of the Multiverse world to use. The world **must** be loaded by Multiverse before SafeAuth initializes. |
| `spawn.x/y/z` | `0.5 / 64 / 0.5` | Exact coordinates of the spawn point inside the auth world. The player is teleported here on **every** join, regardless of where they logged out. |
| `spawn.yaw` | `0.0` | Horizontal rotation at spawn (`0` = south, `90` = west, `180` = north, `270` = east). |
| `spawn.pitch` | `0.0` | Vertical rotation at spawn (`0` = straight ahead, `-90` = looking up). |
| `restore-location` | `true` | Save the player's real-world location before teleporting to the auth world, and restore it after successful login. |
| `teleport-after-auth` | `true` | Teleport the player back to their saved location after authenticating. Disable only if you handle teleportation yourself. |
| `teleport-to-spawn-if-no-location` | `true` | If a player has no saved location (e.g. first join), send them to the main world's spawn instead of dropping them at `0,0,0`. |
| `return-mode` | `saved` | Destination after authentication. `saved` = exact world + coordinates from before the auth-world teleport. `config` = fixed coordinates defined by `return-world/x/y/z` below. |
| `return-world` | `world` | Used only when `return-mode: config`. Target world name. |
| `return-x/y/z` | `0.5/64/0.5` | Used only when `return-mode: config`. Target coordinates. |
| `return-yaw/pitch` | `0.0` | Used only when `return-mode: config`. Target facing direction. |
| `clear-inventory-during-auth` | `false` | `true` = hide the player's inventory while they are in the auth world (restored after login). `false` = player keeps their items visible. Recommended to leave `false`. |
| `spawn-barrier-enabled` | `false` | Place a permanent 3×3 grid of barrier blocks at `spawnY - 1`. Prevents the player from falling in void worlds. The barrier is **permanent** (never automatically removed). |
| `spawn-barrier-material` | `BARRIER` | Block material used for the floor. `BARRIER` is invisible to players and cannot be broken in Survival. You can use any valid Bukkit `Material` name (e.g. `GLASS`, `BEDROCK`). |

---

### `effects`

Visual and audio effects shown to unauthenticated players. All message text is configured in `messages.yml`.

```yaml
effects:
  join-title:
    enabled: true
    fade-in:  10
    stay:     300
    fade-out: 10
  bossbar:
    enabled:  true
    color:    RED
    style:    PROGRESS
    animated: true
  action-bar:
    enabled: true
    update-interval: 20
  particles:
    enabled:  true
    type:     FLAME
    count:    3
    interval: 10
    offset-x: 0.3
    offset-y: 0.3
    offset-z: 0.3
    speed:    0.0
  blindness: true
  slowness:
    enabled: true
    level:   5
  nausea: false
  success-title:
    enabled:  true
    fade-in:  5
    stay:     60
    fade-out: 20
  success-particles:
    enabled:  true
    type:     TOTEM
    count:    40
    offset-x: 0.5
    offset-y: 0.8
    offset-z: 0.5
    speed:    0.1
  sounds:
    on-join:     { enabled: true, sound: BLOCK_NOTE_BLOCK_BASS,       volume: 1.0, pitch: 0.5 }
    on-success:  { enabled: true, sound: ENTITY_PLAYER_LEVELUP,       volume: 1.0, pitch: 1.2 }
    on-fail:     { enabled: true, sound: ENTITY_VILLAGER_NO,          volume: 1.0, pitch: 0.8 }
    on-register: { enabled: true, sound: UI_TOAST_CHALLENGE_COMPLETE, volume: 1.0, pitch: 1.0 }
```

| Key | Description |
|-----|-------------|
| `join-title.*` | Title shown on screen immediately when an unauthenticated player joins. `fade-in`, `stay`, `fade-out` in ticks. |
| `bossbar.color` | Color of the timeout countdown bar. Options: `PINK`, `BLUE`, `RED`, `GREEN`, `YELLOW`, `PURPLE`, `WHITE`. |
| `bossbar.style` | Bar style. `PROGRESS` = solid bar. `NOTCHED_6/_10/_12/_20` = segmented. |
| `bossbar.animated` | If `true`, the bar visually drains over the `auth.timeout` duration. |
| `action-bar.update-interval` | How often (ticks) the action bar text is refreshed. |
| `particles.*` | Particles spawned around the player while unauthenticated. Particle names are Bukkit `Particle` enum values. `interval` = ticks between spawns. |
| `blindness` | Apply Blindness effect (dark screen) while unauthenticated. |
| `slowness.level` | Slowness potion level (1–6). `5` = nearly unable to walk. |
| `nausea` | Apply Nausea (screen wobble) while unauthenticated. |
| `success-title.*` | Title shown after successful login. |
| `success-particles.*` | Particle burst played at the player's location on login success. |
| `sounds.*` | Sounds for join, login success, login failure, and registration. `sound` accepts Bukkit `Sound` enum names. |

---

### `security`

```yaml
security:
  ip-bruteforce-protection: true
  max-ip-attempts: 10
  ip-block-duration: 600
  ip-change-action: NOTIFY
  log-to-file: true
  log-file: safeauth-audit.log
  alert-on-suspicious: true
  block-proxy: false
  rate-limit:
    max-commands-per-window: 3
    command-window-ms: 5000
    max-joins-per-window: 5
    join-window-ms: 10000
    max-registers-per-window: 3
    register-window-ms: 60000
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `ip-bruteforce-protection` | `true` | Track failed login attempts by IP and block the IP after too many failures. |
| `max-ip-attempts` | `10` | Number of failed attempts from one IP before that IP is blocked at the connection level (rejected in `AsyncPlayerPreLoginEvent`). |
| `ip-block-duration` | `600` | How long (seconds) a blocked IP stays blocked. |
| `ip-change-action` | `NOTIFY` | Action taken when a player logs in from a different IP than their last session. `NOTIFY` = send an alert to online admins. `KICK` = kick the player. `IGNORE` = do nothing. |
| `log-to-file` | `true` | Write security events (logins, failures, lockouts, IP blocks) to a log file. |
| `log-file` | `safeauth-audit.log` | Filename of the security audit log inside `plugins/SafeAuth/`. |
| `alert-on-suspicious` | `true` | Notify online admins in-game when an IP is blocked or a suspicious event occurs. |
| `block-proxy` | `false` | Basic heuristic check to block connections from known proxy/VPN IP ranges. |
| `rate-limit.max-commands-per-window` | `3` | Maximum `/login` or `/adminlogin` commands a single player may send within `command-window-ms`. Excess commands are silently dropped. |
| `rate-limit.command-window-ms` | `5000` | Sliding time window in milliseconds for the per-player command rate limit. |
| `rate-limit.max-joins-per-window` | `5` | Maximum connections from a single IP within `join-window-ms`. Excess connections are kicked immediately (bot flood protection). |
| `rate-limit.join-window-ms` | `10000` | Time window for join flood detection (ms). |
| `rate-limit.max-registers-per-window` | `3` | Maximum new account registrations from one IP within `register-window-ms`. |
| `rate-limit.register-window-ms` | `60000` | Time window for registration flood detection (ms). |

---

### `integrations`

```yaml
integrations:
  placeholderapi:
    enabled: true
  discordsrv:
    enabled: false
    notify-channel: security-log
  luckperms:
    enabled: true
    unauthenticated-group: ''
    restore-after-auth: true
```

| Key | Description |
|-----|-------------|
| `placeholderapi.enabled` | Register PlaceholderAPI variables. Requires PlaceholderAPI installed. |
| `discordsrv.enabled` | Send security alerts to a Discord channel via DiscordSRV. Requires DiscordSRV installed. |
| `discordsrv.notify-channel` | DiscordSRV channel name to send alerts to. |
| `luckperms.enabled` | Enable LuckPerms integration. |
| `luckperms.unauthenticated-group` | If set, temporarily assigns this LuckPerms group to unauthenticated players (removed on login). Leave `''` to disable. |
| `luckperms.restore-after-auth` | Restore the original LuckPerms group after authentication. Keep `true`. |

---

### `maintenance`

```yaml
maintenance:
  cleanup-days: 0
  cleanup-on-startup: false
```

| Key | Default | Description |
|-----|:-------:|-------------|
| `cleanup-days` | `0` | Automatically delete accounts that haven't logged in for this many days. `0` = disabled. **Be careful** — this permanently removes data. |
| `cleanup-on-startup` | `false` | Run the cleanup task when the server starts (instead of waiting for the scheduled interval). |

---

### `features`

#### Login Streak

Tracks how many consecutive days a player has logged in. Milestones (7, 14, 30, 60, 100, every 100 after) show a special message.

```yaml
features:
  login-streak:
    enabled: true
    color-tiers:
      - { min-days: 0,   color: "<white>"           }
      - { min-days: 3,   color: "<yellow>"          }
      - { min-days: 7,   color: "<gold>"            }
      - { min-days: 14,  color: "<green>"           }
      - { min-days: 30,  color: "<aqua>"            }
      - { min-days: 60,  color: "<light_purple>"    }
      - { min-days: 100, color: "<gradient:#FFD700:#FF6B35>" }
      - { min-days: 365, color: "<gradient:#FF0000:#FF6B35:#FFD700>" }
    color-suffix: "</gradient>"
    icon: "🔥 "
```

| Key | Description |
|-----|-------------|
| `enabled` | Show login streak messages and expose PAPI variables. |
| `color-tiers` | List of milestone thresholds and their MiniMessage color tags. Tiers are evaluated from highest `min-days` downward — the first matching tier is used. You may add, remove, or edit tiers freely. |
| `color-suffix` | Text appended after the streak number. Needed to close gradient tags. Use `""` for plain colors. |
| `icon` | Prefix icon shown in the `%safeauth_streak_colored%` placeholder. Use `""` to disable. |

#### Other Features

```yaml
  password-strength:
    enabled: true
  stats:
    enabled: true
```

| Key | Description |
|-----|-------------|
| `password-strength.enabled` | Show a strength indicator (Weak / Fair / Strong / Very Strong) when a player registers or changes their password. |
| `stats.enabled` | Enable the `/safeauth stats` admin command. Displays registered accounts, online/authenticated ratio, 2FA adoption rate, and locked accounts. |

---

## 📝 messages.yml

All in-game text lives in `plugins/SafeAuth/messages.yml`. Supports full **MiniMessage** formatting including colors, gradients, hover tooltips, and clickable text.

```yaml
# Example entries
login-success:  "<green>✔ Welcome back, <bold>{player}</bold>!"
login-required: "<red>⚠ Please type <yellow>/login <password></yellow> to authenticate."
register-required: "<aqua>✦ New here? Type <yellow>/register <password> <password></yellow>."
```

**Available placeholders in messages:**

| Placeholder | Available in |
|-------------|-------------|
| `{player}` | Most messages |
| `{time}` | Timeout and lockout messages |
| `{attempts}` | Captcha and failed login messages |
| `{code}` | Text captcha messages |
| `{streak}` / `{days}` | Login streak messages |

To change language, set `general.language: ru` (or `en`) in config.yml. This switches which translation file is used.

---

## 📊 PlaceholderAPI Variables

Requires PlaceholderAPI installed and `integrations.placeholderapi.enabled: true`.

| Placeholder | Type | Description |
|-------------|------|-------------|
| `%safeauth_is_logged_in%` | `true` / `false` | Whether the player is currently authenticated |
| `%safeauth_is_registered%` | `true` / `false` | Whether the player has a registered account |
| `%safeauth_2fa_enabled%` | `true` / `false` | Whether the player has 2FA active |
| `%safeauth_login_streak%` | Number | Consecutive days the player has logged in |
| `%safeauth_streak_colored%` | Formatted string | Streak number with tier color and icon (uses `color-tiers` config) |
| `%safeauth_total_logins%` | Number | Total number of times the player has logged in |
| `%safeauth_last_login%` | Date string | Date and time of the player's last login |
| `%safeauth_login_streak_color%` | MiniMessage color tag | Raw color tag for the current streak tier |

**Usage examples (TAB plugin, scoreboard, etc.):**
```
%safeauth_streak_colored% day streak
Authenticated: %safeauth_is_logged_in%
```

---

## ❓ FAQ

**A player forgot their password — how do I reset it?**
```
/safeauth setpassword <player> <newpassword>
```

**A player lost their phone and can't use 2FA — how do I reset it?**
```
/safeauth disable2fa <player>
```

**I want to completely delete a player's account so they can register again:**
```
/safeauth unregister <player>
```

**How do I set the admin second-factor password?**
```
/safeauth setadminpass <password>
```
The password is hashed and stored automatically. Never paste a plain-text password into config.yml.

**Auth World isn't working / players are not teleported:**
1. Ensure Multiverse-Core is installed and has loaded the world (`/mv list`).
2. Check that `auth-world.enabled: true` and `auth-world.world-name` exactly matches the Multiverse world name (case-sensitive).
3. Verify the spawn coordinates point to a valid location inside the world.
4. Check server logs on startup for SafeAuth warning messages about the world being null.

**Players are getting kicked for "flying" in the auth world:**
- Enable `auth-world.spawn-barrier-enabled: true` to place a solid floor under the spawn point.
- The barrier is placed before the player teleports in, so they land on it immediately.

**Action captcha isn't counting jumps or crouches:**
- SafeAuth uses Paper's `PlayerJumpEvent`. This event **requires Paper** — it does not exist on Spigot. If you are on Spigot, switch to Paper.
- Crouching uses `PlayerToggleSneakEvent` which works on both Paper and Spigot.

**Session expired but player still had to re-enter their password before timeout:**
- Check `auth.session-by-ip: true`. If the player's IP changed (VPN, mobile network switch), the session is intentionally invalidated.
- Reduce `auth.session-duration` if you want shorter sessions, or set to `0` to disable sessions entirely.

**Players with a valid Mojang account should skip authentication:**
- Set `auth.premium-skip: true`. This works only on hybrid servers where the server can verify Mojang accounts.

---

## 📁 File Structure

```
plugins/SafeAuth/
├── config.yml            ← main configuration (this document)
├── messages.yml          ← all player-facing text (EN + RU)
├── safeauth-audit.log    ← security event log
├── debug.log             ← debug log (only when debug: true)
└── safeauth.db           ← SQLite database (when database.type: sqlite)
```

---

<div align="center">

**SafeAuth v2.0** — Reliable account protection, zero compromise

*Paper 1.20+ · SQLite & MySQL · PlaceholderAPI · Velocity & BungeeCord*

</div>
