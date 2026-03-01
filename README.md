# Partials · Newsletter

> The official newsletter for **[Partials](https://partials-gh.github.com/home)** — updates, bug fixes, new projects, and announcements delivered through a live, self-hosted feed.

**---

## Overview

This is a single-file newsletter platform built for the Partials team. It lives as one `newsletter.html` file backed by a [Supabase](https://supabase.com) database, with no framework, no build step, and no server required. Drop it anywhere and it works.

Readers can browse issues, filter by category, and react to posts. Admins log in with a rotating monthly access code to publish, manage, and delete issues.

---

## Features

- **Live updates** — new issues appear for all readers within 60 seconds, no refresh needed
- **Thumbs up / down reactions** — per-session voting with accurate live counters

---

## Categories

Four categories are built in and always available to browse:

| Category | Color |
|---|---|
| Bug Fix | Red |
| Update | Blue |
| New Project | Purple |
| Other | Yellow |

---

## Tech Stack

- Vanilla HTML / CSS / JS — no framework
- [Supabase](https://supabase.com) — database, auth, realtime, `pg_cron`
- [Fraunces](https://fonts.google.com/specimen/Fraunces) + [DM Mono](https://fonts.google.com/specimen/DM+Mono) — typography
- [Netlify Drop](https://app.netlify.com/drop) — hosting

---**

*Part of the [Partials](https://partials-gh.github.com/home) project.*
