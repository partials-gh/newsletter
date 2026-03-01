# Partials · Newsletter

> The official newsletter for **[Partials](https://partials-gh.github.com/home)** — updates, bug fixes, new projects, and announcements delivered through a live, self-hosted feed.

**---

## Overview

This is a single-file newsletter platform built for the Partials team. It lives as one `newsletter.html` file backed by a [Supabase](https://supabase.com) database, with no framework, no build step, and no server required. Drop it anywhere and it works.

Readers can browse issues, filter by category, and react to posts. Admins log in with a rotating monthly access code to publish, manage, and delete issues.

---

## Features

- **Live updates** — new issues appear for all readers within 60 seconds, no refresh needed
- **Markdown composer** — write issues in a custom markdown dialect (headings, bold, italic, underline, strikethrough, copiable code chips, links, blockquotes, bullets, and more)
- **Four built-in categories** — Bug Fix, Update, New Project, Other — each with its own color
- **Thumbs up / down reactions** — per-session voting with accurate live counters
- **Monthly rotating access code** — auto-generated on the 1st of every month via `pg_cron`, stored in Supabase. Fallback master code always available
- **Code embed** — a companion `code-embed.html` page for displaying and regenerating the current access code (designed to be embedded in Google Sites or similar)
- **Zero dependencies** — no npm, no bundler, one HTML file

---

## Project Structure

```
newsletter.html      — main reader + admin dashboard
code-embed.html      — access code display widget (for embedding)
README.md            — this file
```

---

## Setup

### 1. Supabase

Create a free project at [supabase.com](https://supabase.com) and run the following SQL in the **SQL Editor**:

```sql
-- Newsletters
CREATE TABLE newsletters (
  id         uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  title      text NOT NULL,
  author     text,
  category   text,
  kicker     text,
  body       text,
  created_at timestamptz DEFAULT now()
);
ALTER TABLE newsletters ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read"  ON newsletters FOR SELECT USING (true);
CREATE POLICY "Anon insert"  ON newsletters FOR INSERT WITH CHECK (true);
CREATE POLICY "Anon delete"  ON newsletters FOR DELETE USING (true);

-- Categories
CREATE TABLE categories (
  id         uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  name       text NOT NULL UNIQUE,
  created_at timestamptz DEFAULT now()
);
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read"  ON categories FOR SELECT USING (true);
CREATE POLICY "Anon insert"  ON categories FOR INSERT WITH CHECK (true);
CREATE POLICY "Anon delete"  ON categories FOR DELETE USING (true);

-- Access codes
CREATE TABLE access_codes (
  id         uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  code       text NOT NULL,
  month_key  text NOT NULL UNIQUE,
  created_at timestamptz DEFAULT now()
);
ALTER TABLE access_codes ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Anon read"    ON access_codes FOR SELECT USING (true);
CREATE POLICY "Anon insert"  ON access_codes FOR INSERT WITH CHECK (true);
CREATE POLICY "Anon update"  ON access_codes FOR UPDATE USING (true) WITH CHECK (true);

-- Reactions
CREATE TABLE reactions (
  id            uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  newsletter_id uuid NOT NULL REFERENCES newsletters(id) ON DELETE CASCADE,
  emoji         text NOT NULL,
  session_id    text NOT NULL,
  created_at    timestamptz DEFAULT now(),
  UNIQUE(newsletter_id, emoji, session_id)
);
ALTER TABLE reactions ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public read"  ON reactions FOR SELECT USING (true);
CREATE POLICY "Anon insert"  ON reactions FOR INSERT WITH CHECK (true);
CREATE POLICY "Anon delete"  ON reactions FOR DELETE USING (true);
```

Then enable **Realtime** on all four tables: Database → Replication → toggle ON for `newsletters`, `categories`, `access_codes`, `reactions`.

### 2. Auto-rotating access code

Enable `pg_cron` and schedule monthly code generation:

```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;

CREATE OR REPLACE FUNCTION generate_monthly_code()
RETURNS void AS $$
DECLARE
  mk text;
  new_code text;
  chars text := 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  i int;
BEGIN
  mk := to_char(now(), 'YYYY-MM');
  IF NOT EXISTS (SELECT 1 FROM access_codes WHERE month_key = mk) THEN
    new_code := '';
    FOR i IN 1..6 LOOP
      new_code := new_code || substr(chars, floor(random() * length(chars) + 1)::int, 1);
    END LOOP;
    INSERT INTO access_codes (code, month_key) VALUES (new_code, mk);
  END IF;
END;
$$ LANGUAGE plpgsql;

SELECT cron.schedule('monthly-access-code', '0 0 1 * *', 'SELECT generate_monthly_code();');

-- Generate this month's code immediately
SELECT generate_monthly_code();
```

### 3. Configure credentials

Open `newsletter.html` and `code-embed.html` in a text editor. Find and fill in:

```js
var SUPABASE_URL      = 'https://your-project.supabase.co';
var SUPABASE_ANON_KEY = 'your-anon-key';
```

---

## Categories

Four categories are built in and always available:

| Category | Color |
|---|---|
| Bug Fix | Red |
| Update | Blue |
| New Project | Purple |
| Other | Yellow |

Custom categories can be added from the admin panel under **Categories**.

---

## Tech Stack

- Vanilla HTML / CSS / JS — no framework
- [Supabase](https://supabase.com) — database, auth, realtime, `pg_cron`
- [Fraunces](https://fonts.google.com/specimen/Fraunces) + [DM Mono](https://fonts.google.com/specimen/DM+Mono) — typography
- [Netlify Drop](https://app.netlify.com/drop) — hosting

---**

*Part of the [Partials](https://partials-gh.github.com/home) project.*
