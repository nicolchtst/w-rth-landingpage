# VOICO × Würth – Landingpage Werkstatt & Autohaus

Statische Single-Page-Landingpage (kein Build-Step) für die Kooperation **VOICO × Würth**.
Zielgruppe: Werkstätten, Autohäuser, Karosserie & Lack, Reifen-/Teiledienst.
Traffic: Direktlink vom Würth-Außendienst (kein Paid Ads nötig, funktioniert aber genauso).

## Dateien

| Datei | Zweck |
|---|---|
| `index.html` | Komplette Landingpage inkl. Anfrage-Funnel (Overlay) |
| `danke.html` | Danke-Seite – **hier feuert das Meta-`Lead`-Event** |
| `vercel.json` | cleanUrls, Rewrites für die Funnel-URLs, Cache- & Security-Header |
| `assets/fonts/` | Montserrat (self-hosted, DSGVO – kein Google-CDN) |
| `assets/logos/` | Würth, Kia, BERESA, auto dm, MainzlGroup, Telekom-T |
| `assets/voico-logo.svg`, `assets/favicon.svg` | Logo & Favicon |

## Deploy (GitHub → Vercel)

1. Neues GitHub-Repo anlegen, **Inhalt dieses Ordners direkt ins Repo-Root** legen
   (nicht in einen Unterordner – sonst 404 auf `/`).
2. Vercel → *Add New Project* → Repo wählen → **Framework Preset: Other** → Deploy.
3. Domain: z. B. `wuerth.voico.ai` (CNAME auf Vercel).
4. Fertig – kein Build-Command, kein Output-Directory.

### URLs

| URL | Inhalt |
|---|---|
| `/` | Landingpage |
| `/anfrage/betrieb`, `/anfrage/standorte`, `/anfrage/anrufe`, `/anfrage/problem`, `/anfrage/heute`, `/anfrage/dms`, `/anfrage/wuerth`, `/anfrage/rolle`, `/anfrage/start`, `/anfrage/erreichbarkeit`, `/anfrage/kontakt` | je ein Funnel-Step (eigene URL → Absprung-Tracking) |
| `/danke` | Danke-Seite, `noindex`, Lead-Event |

Jeder Step ist auch direkt verlinkbar (Reload/Deeplink funktioniert, Browser-Zurück ebenfalls).
`/?anfrage=1` oder `/#anfrage` öffnet den Funnel direkt.

## Vor dem Livegang anpassen

1. **Make-Webhook** – in `index.html` oben:
   ```js
   window.VOICO_WEBHOOK = "";   // ← Webhook-URL eintragen
   ```
   Leer = Testmodus: Der Lead wird nur in der Browser-Konsole ausgegeben
   (`window.__lastLead`), die Weiterleitung auf `/danke` passiert trotzdem.
2. **Tracking-IDs** – Pixel/GA4/Ads sind von voico.ai übernommen.
   Für Stape/Server-GTM: `window.VV_SGTM_URL = "https://sgtm.voico.ai"` setzen.
3. **Preis** – steht an drei Stellen im Text (`0,22 €`) und einmal im Rechner
   (`var PRICE = 0.22;` im Page-Script).

## Außendienst-Zuordnung

Hängt der Würth-Außendienst sein Kürzel an den Link, landet es im Lead:

```
https://wuerth.voico.ai/?ad=MS-4711
```

→ Feld `aussendienst = "MS-4711"`, `quelle = "wuerth-aussendienst"`.
Das Kürzel überlebt den ganzen Funnel (sessionStorage), auch wenn zwischendurch
neu geladen wird. UTM-Parameter und `fbclid`/`gclid` werden zusätzlich übernommen.

## Qualifizierung (Scoring)

- Gewichtete Punkte je Antwort (`s`), Gewichtung je Frage (`weight`), Knock-outs (`ko`).
- **Fit A** ab 72 %, **Fit B** ab 52 %, sonst **C**. K.O. (= „Erst mal nur informieren") → immer C.
- Jede Anfrage wird gesendet – auch C. Die Bewertung steht in den Feldern
  `fit`, `fit_label`, `score`, `qualifiziert`, `ko_gruende`.
- Schrauben an `STEPS` / `THRESHOLDS` ganz oben im Funnel-Script.

## Lead-Felder (Make → Lead Table)

Flache Felder, form-urlencoded – in der Lead Table müssen Felder mit **exakt diesen Namen**
existieren, sonst werden sie nicht angezeigt:

```
zeitpunkt, datum, status, angebot, partner, aussendienst,
fit, fit_label, score, qualifiziert, ko_gruende,
firma, vorname, nachname, name, telefon, telefon_e164, email, ort, kundennummer, nachricht,
erreichbarkeit, betriebsart, standorte, anrufe_pro_tag, probleme, heute, dms,
wuerth_kunde, rolle, start, zusammenfassung,
quelle, utm_source, utm_medium, utm_campaign, utm_content, utm_term, fbclid, gclid,
cta, funnel_dauer_sek, bot_verdacht, geraet, seite, event_id
```

`event_id` ist die Dedupe-ID für Meta (Browser-Pixel ↔ CAPI/Stape) – im Make-Szenario
als `event_id` an die Conversions-API weitergeben.

## Tracking-Events

| Event | Wann |
|---|---|
| `PageView`, `ViewContent` | Seitenaufruf (nach Consent) |
| `FunnelStart` | erster Klick auf einen CTA |
| `FunnelStep` + `PageView` | je Funnel-Step (mit `step`, `step_nr`) |
| `Lead` (+ `QualifiedLead` bei A/B) | **nur** auf `/danke`, nur nach echtem Absenden, nur einmal |

Alles lädt erst nach Einwilligung (Consent-Banner). Ohne Marketing-Consent feuert kein Pixel.

## Inhaltliche Hinweise

- Preis- und Konditionsangaben: **0,22 €/Min., keine Grundgebühr, keine Mindestlaufzeit.**
- Der Rechner ist eine transparente Beispielrechnung (Anrufe × Dauer × Öffnungstage × 0,22 €) –
  es werden keine Ersparnisse behauptet.
- Rollen in der Kooperation stehen explizit auf der Seite: **Würth = Vertrieb/Ansprechpartner,
  VOICO = Technik und Vertragspartner.** Über Würth selbst werden keine Zahlen behauptet.
- Die Screenshots im Abschnitt „Hinter den Kulissen" werden von `www.voico.ai/platform/…`
  geladen und blenden sich automatisch aus, falls sie nicht erreichbar sind.
  Besser: die drei PNGs nach `assets/img/` legen und die Pfade ersetzen.
