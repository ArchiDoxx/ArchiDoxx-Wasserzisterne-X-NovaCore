# Aktionsplan: Empfehlungssystem wasserzisterne.de

**Stand:** 2026-05-20  
**Team:** 2 Personen, ~15–20 Std/Woche  
**Zieldomain:** `empfehlung.wasserzisterne.de`  
**Ablösung von:** TELLSCALE (KIWUS CONSULTING)

---

## 1. Was wird gebaut

Ein eigenentwickeltes Empfehlungssystem, das Bestandskunden zu Multiplikatoren macht. Bestandskunden erhalten nach einem Kauf eine automatisierte E-Mail, erstellen über eine Landing Page ihren persönlichen Empfehlungslink und teilen diesen per WhatsApp (Mobil) oder E-Mail (Desktop). Neukunden, die über diesen Link kommen, landen auf einer personalisierten Infopage. Wenn ein Neukunde kauft, erhält der Promotor einen 25 € Amazon-Gutschein.

Das System ersetzt die kostenpflichtige TELLSCALE-Lösung durch eine maßgeschneiderte Eigenentwicklung mit direkter KlickTipp-Integration und wasserzisterne-eigenem Branding.

---

## 2. User Flows (vollständig)

### Flow A — Bestandskunde Mobil (Primär-Flow)

```
E-Mail-Kampagne (Exchange / KlickTipp)
  → Bestandskunde öffnet E-Mail auf Handy
  → QR-Code scannen
  → NPS-Gate: "Wie wahrscheinlich empfiehlst du uns?" (0–10)
    ├── Score 9–10 → C1 Mobile Info-Seite ("Jetzt empfehlen")
    │     → D1 Formular: Vorname, Nachname, E-Mail, Telefon
    │     → System generiert persönlichen Token-Link (/29034)
    │     → WhatsApp-Share: vorgefertigter Text + Link (NUR WhatsApp)
    └── Score 0–8  → Feedback-Flow
          → Kategorisierung: Preis, Erreichbarkeit, Qualität, Info, Kreativität
          → Freitextfeld anonym
          → Danke-Seite
```

### Flow B — Bestandskunde Desktop (Sekundär-Flow)

```
E-Mail-Kampagne
  → Bestandskunde klickt Link im Desktop-Browser
  → NPS-Gate (gleiche Logik)
    ├── Score 9–10 → C2 Desktop Info-Seite
    │     → D2 Formular: gleiche Felder + separate E-Mail-Eingabe für Empfehlung
    │     → Übergabe an KlickTipp (automatisch)
    │     → E-Mail-Share
    └── Score 0–8 → Feedback-Flow (identisch)
```

### Flow C — Neukunde (Empfänger des Links)

```
Öffnet WhatsApp-Link / E-Mail-Link (empfehlung.wasserzisterne.de/[token])
  → Cookie-Consent (DSGVO)
  → Personalisierte Info-Seite: "Empfehlung von [Promotor-Vorname]"
    + 60s Video
    + CTA: "Beratungstermin buchen"
  → Redirect → etermin.net/wasserzisterne (existiert bereits)
```

### Flow D — Admin (Backend)

```
Internes Admin-Panel (passwortgeschützt)
  → Promotoren-Übersicht: Liste, Suche, CSV-Import
  → Pro Promotor: Empfehlungslinks, Klicks, Empfehlungen, Kunden
  → Empfehlungen-Liste: Status (Neu / Kontaktiert / Gekauft / Gutschein versendet)
  → Lookup: E-Mail / Name / Telefon → Wurde diese Person empfohlen?
  → Manuelle Status-Pflege (Gutschein-Versand)
```

---

## 3. Feature-Scope

### IN SCOPE (wird gebaut)

| # | Feature | Priorität |
|---|---------|-----------|
| 1 | NPS-Gate (0–10 Bewertung + Routing-Logik) | P0 |
| 2 | C1 Mobile Bestandskunden-Infoseite | P0 |
| 3 | D1 Mobile Link-Generator + WhatsApp-Share | P0 |
| 4 | E1 Neukunden-Landingpage (Video + CTA) | P0 |
| 5 | Tracking-System (eindeutige Token-Links, Klick-Logging) | P0 |
| 6 | C2 Desktop Bestandskunden-Infoseite | P1 |
| 7 | D2 Desktop Link-Generator + KlickTipp-Übergabe | P1 |
| 8 | Feedback-Flow (anonym, kategorisiert, Freitext) | P1 |
| 9 | Admin-Panel: Promotoren-Verwaltung (CRUD + CSV-Import) | P1 |
| 10 | Admin-Panel: Empfehlungs-Tracking + Status | P1 |
| 11 | Admin-Panel: Lookup-Funktion | P1 |
| 12 | Cookie-Consent (DSGVO) | P0 |
| 13 | wasserzisterne-Branding (Farben, Fonts, Logo) | P0 |

### OUT OF SCOPE (explizit ausgeschlossen)

- Template-Konfigurator für Neukundenpage — fixe Seite reicht
- Template-Konfigurator für Promotor-Seite — fix
- Automatische Gutschein-Auszahlung — manuell via Amazon-Gutscheincode
- Eigenes E-Mail-Versandsystem — Exchange / KlickTipp übernimmt das
- Bewertungsplattform-Integration (Google, Trustpilot) — nicht im Scope

---

## 4. Grobes Datenbankschema

```
promotors
  id, vorname, nachname, email, telefon, created_at, last_activity_at

referral_links
  id, promotor_id → promotors,
  token (eindeutig, z.B. "29034"),
  link_clicks_count, created_at

referrals  (= eingehende Neukunden-Leads)
  id, referral_link_id → referral_links, promotor_id,
  neukunde_vorname, neukunde_nachname, neukunde_email, neukunde_telefon,
  status ENUM(neu | kontaktiert | gekauft | gutschein_versendet),
  created_at, updated_at

nps_responses
  id, promotor_email, score (0–10),
  feedback_categories TEXT[],
  feedback_text TEXT,
  created_at

tracking_events
  id, referral_link_id, event_type ENUM(link_click | whatsapp_share | page_view),
  user_agent, ip_hash (DSGVO-konform gehashed), created_at
```

---

## 5. Tech-Stack-Empfehlung

| Schicht | Technologie | Begründung |
|---------|------------|------------|
| Frontend | Next.js (App Router) | SSR, mobil-optimiert, ein Repo für alle Seiten |
| Backend / API | Next.js API Routes | Kein separater Server nötig, geringer Overhead |
| Datenbank | PostgreSQL via Supabase | Schneller Start, Auth eingebaut, kein eigener DB-Server |
| Hosting | Vercel | Zero-Config-Deploy, passt zu Next.js |
| WhatsApp-Share | Browser Web Share API | Nativ, kein Drittdienst, kostenlos |
| E-Mail / KlickTipp | KlickTipp REST API | Webhook bei Empfehlung → Tag setzen → Automations-Flow |
| Admin-Auth | NextAuth.js (Credentials) | Einfaches Login ohne externe Auth-Dienste |

---

## 6. Phasenplan mit Stundenschätzungen

> Basis: 15–20 Std/Woche für das 2-Mann-Team gesamt

### Phase 1 — Fundament & Setup (Woche 1)
**~15 Stunden**

| Aufgabe | Std |
|---------|-----|
| Domain empfehlung.wasserzisterne.de konfigurieren (DNS, SSL) | 2 |
| Repo-Setup: Next.js, Supabase, Vercel-Deployment-Pipeline | 3 |
| Datenbank-Schema + Migrations schreiben | 3 |
| Design-System: wasserzisterne-Farben, Fonts, Logo, Komponenten-Basis | 4 |
| KlickTipp-API-Dokumentation studieren + Testanbindung | 3 |

### Phase 2 — Core Mobile Flow (Woche 2–3)
**~30 Stunden**

| Aufgabe | Std |
|---------|-----|
| NPS-Gate Seite + Routing-Logik (9/10 vs. 0–8) | 4 |
| C1 Mobile Bestandskunden-Infoseite | 3 |
| D1 Formular + Token-Link-Generierung (Backend-Logik) | 6 |
| WhatsApp Web Share Integration + vorgefertigter Text | 4 |
| Tracking-System: Token-Erkennung, Klick-Logging, Neukunden-Erfassung | 7 |
| E1 Neukunden-Landingpage + Video-Embed + CTA | 6 |

### Phase 3 — Desktop Flow + Feedback (Woche 4)
**~20 Stunden**

| Aufgabe | Std |
|---------|-----|
| C2 Desktop Bestandskunden-Infoseite (Responsive-Redesign) | 4 |
| D2 Desktop Link-Generator + E-Mail-Eingabe-Maske | 4 |
| KlickTipp-API-Integration: automatische Tag-Übergabe bei Empfehlung | 6 |
| Feedback-Flow: NPS 0–8 → Kategorisierung + Freitext (anonym) | 5 |
| Cookie-Consent DSGVO-konform | 2 |

### Phase 4 — Admin-Panel (Woche 5)
**~20 Stunden**

| Aufgabe | Std |
|---------|-----|
| Auth: einfaches Admin-Login (NextAuth Credentials) | 2 |
| Promotoren-Übersicht + CRUD + CSV-Import | 5 |
| Empfehlungen-Liste + Status-Verwaltung | 5 |
| Lookup-Funktion: E-Mail / Name / Tel eingeben → Treffer? | 3 |
| Dashboard: Kennzahlen (Klicks, Empfehlungen, Conversions, NPS) | 3 |
| Admin-Benachrichtigung bei neuer Empfehlung (E-Mail-Trigger) | 2 |

### Phase 5 — Finish, Testing, Launch (Woche 6)
**~15 Stunden**

| Aufgabe | Std |
|---------|-----|
| Mobile-Optimierung + Cross-Browser-Tests | 4 |
| End-to-End-Test aller Flows (Mobil + Desktop + Admin) | 5 |
| Bugfixes | 3 |
| Go-Live: DNS-Umschaltung, Monitoring | 2 |
| Übergabe + Mini-Doku für Admin-Panel | 1 |

### Gesamtübersicht

| Phase | Zeitraum | Stunden |
|-------|----------|---------|
| 1 — Setup | Woche 1 | ~15 h |
| 2 — Core Mobile Flow | Woche 2–3 | ~30 h |
| 3 — Desktop + Feedback | Woche 4 | ~20 h |
| 4 — Admin-Panel | Woche 5 | ~20 h |
| 5 — Finish & Launch | Woche 6 | ~15 h |
| **GESAMT** | **~6 Wochen** | **~100 Stunden** |

---

## 7. Offene Fragen — Pflicht-Klärung im Kundengespräch

### Kritisch (blockieren den Start)

1. **KlickTipp-API-Zugang**: Welchen Plan hat wasserzisterne.de? API-Key vorhanden?
2. **Initiale E-Mail-Kampagne**: Exchange oder KlickTipp? Wer konfiguriert den Automations-Trigger?
 Klicktipp wird die hauptkampagne werden 
3. **Bestandskunden-Daten**: CSV-Export aus dem Shop? Welche Felder? Wie viele Datensätze ca.?
  Würden wir aufbauen können gerade mit hinblick auf das zukünftige CRM System sollten wir da eine gute Lösung finden. Woo-Commerce? kann man ebenfalls zur Datensammeln für die Datenbank liefern.  
  Momeentan werden alle daten in safedesk gepseichert aber die kann man dann in der api ebenfalls zum Daten ziehen zu nutzen.
4. **Video**: Das 60s-Erklärvideo — bereits vorhanden? Wo gehostet (YouTube, Vimeo, eigener Server)?
- Video exisitert bereits hosting muss geklärt werden, dann wahrscheinlich Vimeo.

### Wichtig (Klärung in Woche 1)

5. **Gutschein-Workflow**: Wer löst den 25€-Gutschein aus? Knopf im Admin-Panel, oder reicht manuelle E-Mail?
.- wurde bisher über streamendes? gemanaged und dort dann automatisch geld aufladen kann und dann wird amazon gutschein gekauft und versendet. man muss hier dann also sehen wie die genaue aorkflow der Kundenentlohnung zu bekommen. 
6. **Hosting-Präferenz**: Vercel (empfohlen) oder eigener Server? (Beeinflusst Setup-Aufwand)
- eigener server präferiert kann man per plugin dann mit wordpress verknüpft werden.  // weiterleitung und
7. **TELLSCALE-Übergang**: Parallel betreiben bis Go-Live oder sofort ablösen?
- bereits quittiert
8. **Kaufbestätigung**: Wie wird geprüft, dass ein empfohlener Neukunde gekauft hat? Manuell im Admin eintragen oder Shop-System-Verknüpfung?
- Auftragsbestätigung? oder beim Rechnungsversand, dann muss das aber auch über Klicktip abgewickelt werden. 


! wie ist verknüpfung der Wordpress und anderen Proprietären Seiten und der Empfehlungspages, da dass backend über eigenen server laufen muss. 
! Arbeitsmittel? brauche mindestens etwas für Vibecoding tools / chatbot experimenten / Datenbank aufsetzen mit datenbank diensten. 

### Nice to know

9. **Anzahl Bestandskunden**: Größenordnung (100 / 1.000 / 10.000+)?
10. **DSGVO**: Bestehende Datenschutzerklärung anpassbar?
- wird über tellscale geregelt und wir machen für System eigene vereinbarung beachte dabei eben opt out und cookie Richtlinie.
11. **Branding-Assets**: Logo in SVG vorhanden? Exakte Hex-Farbcodes?
- Bekommen wir im Template zugeliefert

---

## 8. Risiken & Hinweise

| Risiko | Einschätzung | Maßnahme |
|--------|-------------|----------|
| KlickTipp-API-Limitierungen (Plan-abhängig) | Mittel | In Phase 1 direkt testen; Fallback: SMTP-Webhook |
| WhatsApp Web Share nur auf Mobilgeräten | Bekannt — by design | Desktop nutzt E-Mail; kein Problem |
| DSGVO: Speicherung Neukunden-Daten ohne Einwilligung | Mittel | Cookie-Consent + Datenschutz-Hinweis auf Neukunden-Page |
| Video-Hosting Bandbreite bei Selbst-Hosting | Mittel | YouTube/Vimeo empfehlen, kein Selbst-Hosting für Video |
| TELLSCALE-Datenmigration | Gering | Neues System startet frisch; Bestands-Promotoren via CSV-Import |

---

## 9. Nächste Schritte nach dem Kundengespräch

- [ ] Offene Fragen (Abschnitt 7) vom Kunden beantworten lassen
- [ ] KlickTipp API-Key + Testzugang einrichten lassen
 bekommen wir
- [ ] Branding-Assets liefern lassen (Logo SVG, Farbcodes, Video-URL)
 bekommen wir
- [ ] Bestandskunden-Export als CSV anfordern
 muss ich noch klären wird aber in klicktipp sein
- [ ] Domain-Zugang für DNS-Eintrag empfehlung.wasserzisterne.de klären
 machen wir neu
- [ ] Kick-off Phase 1 terminieren


Notitzen
- weiterleitung und QR-code klären whatsapp business?
- 2 pages vorhanden. 1. hello page & intro bla bla, sollte eher benefit enthalten nicht bla bla. dann nach empfehlen button kommt 2. ist dann Formular und Weiterleitungslink.
- Wird per Massenemail herausgeschickt, präferenz liegt bei weiterempfehlung klar auf Whatsapp. bei pc browser erkennung muss es dann email sein. 
- kontekate genereieren und mehrere bekommen sit ein main fokus ob über whatsapp link oder im andern bezug dazu. was geht mit business whatsapp? kann man Kontakte von chris verlinken direkt herstellen oder mit bots erstkontakt herstellen mit verlinkten? 
- Gitlab backend existiert bsiher nicht . N8N server besteht momentan. kann man gitlab mir auf den hetzner server packen?
- er will die whatsapp-weiterleiten-nachricht auf folie 6 überarbeiten vor allem den zu langen link der dann eine schöne kleine verison werden soll.
- Wie ist verknüpfung der Wordpress und anderen Proprietären Seiten und der Empfehlungspages, da dass backend über eigenen server laufen muss. 

- Die jetzt buchen seite wie kann man leute am besten dazu bringen nach "beratung buchen" zum kontakt zu bekommen?
  ggf automatisierte erkennung  von interessierten bei klicken des buttons ---- dann wird frage bogen ausgefüllt und intern wird info darüber wird antweder gespeichert oder als alert geballert.
- Agenten der ab dort ggf übernimmt? 
- Sie haben auf jeden fall schon schon über klick tipp krasse datenanalyse über die customer funktion dort. 
- Mindestbestellwert muss enthalten sein oder an produkte koppeln, das muss dann auch im wording enthalten sein beim empfehlungssystem.
- was ist deren Hauptserver? kann man darauf dann den Gitlabserver legen?
- Optimerung sollte am besten Möglichkeit für direktes feedbacm zurätzlich zu den kategorien haben., kann mit eigener Datenbank verbunden werden, die wir aufbauen wollen. 









[● Ja, das geht — und ist sogar extrem einfach umzusetzen. WhatsApp hat eine
  native Click-to-Chat URL:

  https://wa.me/4917282087204?text=Hey%20Christian%2C%20ich%20wurde%20empfoh
  len!

  Klick drauf → WhatsApp öffnet sich direkt mit Christians Nummer und dem
  vorausgefüllten Text im Eingabefeld. Der Neukunde muss nur noch auf Senden
   drücken. Funktioniert auf Mobil (WhatsApp App) und Desktop (WhatsApp
  Web). Kein API-Zugang, keine Integration, kein Backend — nur ein Link.

  Das Schlaue daran: Weil wir den Promotor-Token serverseitig kennen, können
   wir den Text dynamisch befüllen:

  Hey Christian, ich wurde von [Promotor-Vorname] empfohlen
  und möchte mehr über eure Wasserbehälter erfahren!

  Christian sieht sofort von wem der Lead kommt, ohne nachfragen zu müssen.

  Zwei sinnvolle Varianten für die E1 Neukunden-Landingpage:

  Option A — WhatsApp als primären CTA:
  - "Schreib mir direkt" → wa.me-Link mit vorausgefülltem Text
  - Gut wenn Christian den Erstkontakt persönlich bevorzugt

  Option B — Beide CTAs nebeneinander:
  - Primär: "Beratungstermin buchen" → etermin.net
  - Sekundär: "Oder schreib mir kurz auf WhatsApp" → wa.me-Link
  - Gibt dem Neukunden die Wahl, erhöht Conversion insgesamt

  Meine Empfehlung: Option B. Manche Leute buchen lieber direkt, andere
  wollen erst kurz schreiben. Zwei Einstiegspunkte fangen beide Typen ab.

  Technischer Aufwand: ~1h — ist eine Zeile Code auf der E1-Seite, die den
  wa.me-Link mit dem dynamischen Text baut.]