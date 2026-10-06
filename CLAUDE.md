# Medienpass Thomaeum – Projektkontext für Claude Code

## Was ist dieses Projekt?

Eine digitale Lernplattform für den **Medienpass** des Gymnasiums Thomaeum Kempen (NRW).
Schülerinnen und Schüler der Klassen 5–10 erwerben Module zu digitaler Kompetenz.
Lehrkräfte verwalten Fortschritte über eine separate Konsole.

Entwickler/Admin: Thomas Jansen (thomas.jansen@thomaeum.de), Digitalisierungsbeauftragter

---

## Dateien im Repo

| Datei | Zweck |
|---|---|
| `index.html` | **Schüler-App** – Wissensspeicher, Quiz, Fortschrittsanzeige |
| `teacher_app.html` | **Lehrer-App** – SharePoint-Anbindung, Modul-/Quiz-Freigaben, Schülerübersicht |

**Wichtige Regel:** Jede inhaltliche oder strukturelle Änderung muss IMMER in BEIDEN Dateien parallel umgesetzt werden. Nie nur eine Datei ändern.

---

## Technologie-Stack

Beide Dateien sind **Single-HTML-Dateien ohne Build-Step** (kein npm, kein Webpack).

- **React 18** via UMD/CDN (`unpkg.com/react@18/umd/react.production.min.js`)
- **ReactDOM 18** via UMD/CDN
- **Babel Standalone** für JSX im Browser (`unpkg.com/@babel/standalone/babel.min.js`)
- **Tailwind CSS** via CDN (`cdn.tailwindcss.com`)
- **Lucide Icons** via CDN (`unpkg.com/lucide@latest`)
- **MSAL.js 2.32.2** für Microsoft-Authentifizierung (`alcdn.msauth.net/browser/2.32.2/...`)

Alles ist inline in einer einzigen `.html`-Datei. Kein externer Build-Prozess nötig.

---

## Microsoft-Zugangsdaten (MS_CONFIG)

```javascript
const MS_CONFIG = {
    tenantId: "09c4bd58-1b8c-410d-930e-816015e61028",
    clientId: "7d277b19-93c1-4df7-8f1b-d38749b3d440",
    siteId: "thomaeumde.sharepoint.com,9ef17c4d-b57b-482e-9168-3cc3bc43bc3d,d600d53b-cfd0-45f7-96a5-f1ac20222f8f",
    listId: "08da02d9-5b67-4938-9d7e-9e9dee97fa2e",
    // Nur in teacher_app.html:
    teacherGroupId: "2bbe9733-7137-4d54-a788-e988c3d56cd9"
};
```

Diese Werte sind **unveränderlich** – nie anfassen.

---

## Deployment

- **Azure Static Web Apps**: `https://red-sand-056025e10.2.azurestaticapps.net`
- Deployment läuft automatisch via GitHub Actions bei Push auf `main`
- Kein `staticwebapp.config.json` oder ähnliches nötig

---

## Datenstruktur: Schüler-App (`index.html`)

### ALL_MODULES (13 Module, identisch in beiden Dateien)

```javascript
const ALL_MODULES = [
    { id: 1,  title: "Word-Führerschein und Dateimanagement", stage: "5-6",  category: "skills",    icon: "file-text" },
    { id: 2,  title: "Umgang mit PowerPoint",                 stage: "5-6",  category: "skills",    icon: "presentation" },
    { id: 3,  title: "Digitale Verantwortung",                stage: "5-6",  category: "reflexion", icon: "heart-handshake" },
    { id: 4,  title: "Suchmaschinen",                         stage: "5-6",  category: "reflexion", icon: "search" },
    { id: 5,  title: "Sichere Passwörter",                    stage: "5-6",  category: "reflexion", icon: "key" },
    { id: 6,  title: "Digitale Kunst & Design",               stage: "7-8",  category: "skills",    icon: "palette" },
    { id: 7,  title: "Tabellenkalkulation",                   stage: "7-8",  category: "skills",    icon: "table" },
    { id: 8,  title: "Audio & Podcasts",                      stage: "9-10", category: "skills",    icon: "mic" },
    { id: 9,  title: "Digitale Quellenarbeit",                stage: "7-8",  category: "reflexion", icon: "search-check" },
    { id: 10, title: "Digitale Selbstbestimmung",             stage: "7-8",  category: "reflexion", icon: "shield-check" },
    { id: 11, title: "KI-Produktivität",                      stage: "9-10", category: "skills",    icon: "cpu" },
    { id: 12, title: "Digitale Mündigkeit",                   stage: "9-10", category: "reflexion", icon: "brain" },
    { id: 13, title: "Datenökonomie",                         stage: "9-10", category: "reflexion", icon: "database" }
];
```

### MODULE_CONTENT_DATA (nur in index.html)

Großes Datenobjekt mit Inhalt pro Modul. Struktur pro Modul:

```javascript
MODULE_CONTENT_DATA[n] = {
    sections: [
        {
            title: "Abschnittstitel",
            intro: "Einleitungstext",
            blocks: [
                { type: 'standard'|'warning'|'info'|'tip'|'success', title: '...', text: '...' }
            ]
        }
    ],
    missions: ["Mission 1", "Mission 2", ...]  // 5–10 Missions pro Modul
}
```

**Text-Formatierung in `text`-Feldern:**
- `**fett**` → wird zu `<strong>` gerendert
- `` `code` `` → wird zu `<code>` gerendert
- `\n\n` → Absatz

**Kritisch – Apostroph-Escaping:**
Alle Strings verwenden einfache Anführungszeichen. Apostrophe in Texten MÜSSEN escaped werden:
```javascript
// FALSCH (bricht JavaScript):
{ text: 'So geht\'s nicht' }  // → 'So geht'  ← String endet hier!

// RICHTIG:
{ text: 'So geht\'s richtig' }  // Backslash-Escape
// ODER:
{ text: "So geht's richtig" }   // Doppelte Anführungszeichen außen
```
Gedankenstriche als `–` (–) und Pfeile als `→` (→) in einfach-gequoteten Strings.

### QUIZ_DATA (nur in index.html)

15 Fragen pro Modul. Format:
```javascript
QUIZ_DATA[n] = [
    { q: "Fragetext", a: ["Antwort A", "Antwort B", "Antwort C", "Antwort D"], c: [korrekterIndex] },
    // Mehrfachantwort möglich: c: [0, 2]
]
```

**Regel für korrekte Antwortpositionen:** Die korrekten Antworten MÜSSEN gleichmäßig über alle Positionen (0, 1, 2, 3) verteilt sein – nicht alle auf derselben Position!

**Quiz-Logik:**
- 15 Fragen, davon 10 werden zufällig gezogen
- Bestehen: ≥ 8/10 richtig (80%)
- 3 Fehlversuche → 5 Minuten Sperre (via `localStorage`)
- Quiz muss von Lehrkraft freigeschaltet werden (SharePoint-Feld `Quiz_Freigaben`)

---

## SharePoint-Felder (Schüler-Liste)

| Feld | Inhalt |
|---|---|
| `Title` | E-Mail-Adresse |
| `Vorname` | Vorname |
| `Nachname` | Nachname |
| `Klasse` | z.B. "5a", "10c" |
| `Erledigte_Module` | Komma-getrennte Modul-IDs, z.B. `"1, 3, 5"` |
| `Quiz_Freigaben` | Komma-getrennte Modul-IDs der freigeschalteten Quizze |
| `Modul_Daten` | JSON-String `{"1": {"datum": "12.05.2026", "lehrer": "Max Müller"}}` |

---

## Stehende Entwicklungsregeln

1. **Umlaute verwenden** – immer: ä, ö, ü, Ä, Ö, Ü, ß
2. **Immer beide Dateien ausgeben** – bei jeder Änderung `index.html` UND `teacher_app.html`
3. **Wissensspeicher nur in Schülerapp** – `MODULE_CONTENT_DATA` und `QUIZ_DATA` gibt es nur in `index.html`, nicht in `teacher_app.html`
4. **MS_CONFIG nie ändern**
5. **Antwortverteilung bei Quiz prüfen** – korrekte Antworten gleichmäßig auf Positionen 0–3 verteilen
6. **Apostroph-Escaping beachten** – Apostrophe in einfach-gequoteten Strings escapen (`\'`)
7. **Keine externen Build-Tools** – alles bleibt eine Single-HTML-Datei

---

## Komponenten-Übersicht: Schüler-App

- `App` – Hauptkomponente, MSAL-Auth, Modulübersicht mit Stufen-Tabs (5-6 / 7-8 / 9-10)
- `QuizModal` – Phasen: `start` → `active` → `passed`/`failed`/`locked`
- `RenderWissensspeicher` – Rendert `MODULE_CONTENT_DATA` mit farbigen Block-Typen
- `Icon` – Wrapper für Lucide-Icons

## Komponenten-Übersicht: Lehrer-App

- `App` – Hauptkomponente, MSAL-Auth, Schülerliste mit Filtern
- `StudentPreview` – Detailansicht eines Schülers mit Modul-/Quiz-Toggle
- `Icon` – Wrapper für Lucide-Icons

---

## Häufige Fehlerquellen

| Problem | Ursache | Fix |
|---|---|---|
| Weiße Seite nach Deployment | Syntaxfehler in JS (meist Apostroph in String) | Apostrophe escapen: `\'` |
| Quiz nicht sichtbar | `Quiz_Freigaben`-Feld in SharePoint nicht gesetzt | Lehrer-App → Quiz freischalten |
| Modul zeigt falsches Datum | `Modul_Daten`-JSON malformed | JSON.parse-Try-Catch prüfen |
| MSAL-Popup-Fehler | Popup-Blocker aktiv | App fällt auf `loginRedirect` zurück |
