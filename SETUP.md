# Sam Prep — Einrichtung

## 1. Dateien ins Repo
Diese drei Dateien nach `annadiack/sam_prep`, Branch `master`:

- `index.html`
- `script.js`
- `config.js`

`linkify.js`, `bg.mp4`, `bg-landing.jpg`, `bg-inner.jpg` und `icon.png` liegen schon drin
und bleiben unverändert. **`supabase-setup.sql` und `einkaufsliste-app*` können weg** —
die stammen aus einem alten Stand und das Schema ist inzwischen anders.

## 2. Im Supabase-Dashboard (Projekt `sam_prep`)

**Edge Functions → Secrets**
`ANTHROPIC_API_KEY` = dein Anthropic-Key. Ohne ihn geht kein Rezept-Import,
keine Kalorienschätzung und kein Ernährungs-Chat.

**Authentication → URL Configuration**
- Site URL: `https://annadiack.github.io/sam_prep/`
- Redirect URLs: dieselbe Adresse zusätzlich eintragen

Ohne diesen Eintrag landen die Magic Links auf einer Fehlerseite.

**Authentication → Providers → Email**
Email-Provider aktiviert lassen. Magic Link ist damit an.
Der kostenlose Mailversand von Supabase ist auf wenige Mails pro Stunde begrenzt.
Für mehr Testnutzer brauchst du unter *Authentication → Emails → SMTP Settings*
einen eigenen Versand (Resend, Postmark, Brevo o. ä.).

## 3. GitHub Pages
Erst **nachdem** Schritt 1 und 2 erledigt sind:
Settings → Pages → Branch `master`, Ordner `/ (root)`.

## Was in der Datenbank steht

8 Tabellen, jede mit `user_id` und einer RLS-Regel `user_id = auth.uid()`:
`shopping_lists`, `categories`, `shopping_items`, `recipes`,
`calendar_entries`, `foods`, `food_log`, `nutrition_goals`.

Ohne Login ist nichts lesbar (die Rechte für die Rolle `anon` sind entzogen).
Mit Login sieht jedes Konto ausschließlich eigene Zeilen.

Jedes neue Konto bekommt automatisch:
- eine leere Liste „Einkaufsliste"
- die Kategorien Obst & Gemüse, Gläser & Konserven, Kräuter & Gewürze,
  Getreide, Milchprodukte, TK, Getränke, Sonstiges
- Standard-Nährwertziele (2000 kcal, 100 g Protein, 250 g KH, 70 g Fett)

Storage-Bucket `recipe-thumbs` (öffentlich) hält die Rezept-Vorschaubilder,
weil die TikTok-Original-URLs nach wenigen Stunden ablaufen.

## Edge Functions

Alle neun laufen mit `verify_jwt: true` — nur angemeldete Konten können sie
aufrufen. Das Frontend schickt dafür das Session-Token mit.

`fetch-link`, `extract-ingredients`, `analyze-food`, `nutrition-chat`,
`estimate-calories`, `estimate-costs`, `suggest-recipes`, `sort-item`, `sort-items`

## Was gegenüber HomeBase fehlt

Absichtlich entfernt: Prospekte, Angebotssuche, Kassenzettel-Scan,
Preis-Historie, Lager, Notizen, Standort, Backlog.
Die Kostenschätzung ist noch da, schätzt aber ohne gelernte Preise.

## Instagram

Automatischer Import ist nicht möglich — Instagram liefert ohne Login keine
Beschreibung aus (getestet mit Browser- und Crawler-Kennung, beides Login-Wall).
Stattdessen: im Import-Fenster einen **Screenshot des Beitrags** hochladen,
Claude liest Titel, Zutaten und Schritte aus dem Bild.
