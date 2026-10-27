# Piano di sviluppo — Agenda

Web app personale (PWA) per gestire impegni con importanza/urgenza, matrice di Eisenhower,
calendario Google e sincronizzazione privata su Google Drive. Sito statico su GitHub Pages.

- Repository: `marcodezan/agenda` → URL finale **https://marcodezan.github.io/agenda/**
- Origine OAuth autorizzata richiesta: `https://marcodezan.github.io` (coincide ✔) e `http://localhost:5173`.

## Decisioni prese

| Tema | Scelta |
| --- | --- |
| Stack | Vite 8 + React 19 + TypeScript 6 (strict) |
| Dipendenze runtime | `date-fns` (date, locale it), `@dnd-kit/core` (drag & drop accessibile), `lucide-react` (icone, tree-shaken), `@fontsource-variable/inter` (font self-hosted → funziona offline) |
| PWA | `vite-plugin-pwa` (manifest + service worker Workbox, precache dell'app shell) |
| Stato | store minimale su `useSyncExternalStore` (niente Redux/Zustand) |
| Stili | CSS puro con design token (variabili CSS), tema chiaro/scuro |
| Cifratura | WebCrypto: PBKDF2-SHA256 (310.000 iterazioni, salt casuale 16 byte) → chiave AES-GCM 256 non estraibile |
| Cache locale | IndexedDB (wrapper interno minimale), blob cifrato AES-GCM con IV nuovo a ogni scrittura |
| Archivio remoto | file `agenda-data.json` in `appDataFolder` di Google Drive |
| Merge | last-writer-wins per singolo elemento (`updatedAt`), eliminazioni come tombstone |
| Auth Google | Google Identity Services, token model; rinnovo con `prompt: 'none'`; se fallisce → banner "Riconnetti Google" |
| Urgenza automatica | 4 = scaduto o entro 24h · 3 = entro 3 giorni · 2 = entro 7 giorni · 1 = oltre / senza scadenza. "Urgente" ≥ 3, "Importante" = importanza ≥ 3 |
| Default | accento indaco, auto-blocco 10 min, settimana da lunedì, evento calendario di 60 min, categorie colorate |

## Struttura

```
.github/workflows/deploy.yml   build + test + deploy Pages (push su main)
public/                        icone PWA, favicon
src/
  config.ts                    Client ID, scope, costanti
  domain/                      logica pura e testata
    types.ts  defaults.ts  urgency.ts  quadrant.ts  quickAdd.ts
    recurrence.ts  migrations.ts  merge.ts  dates.ts  markdown.tsx
  services/
    crypto.ts  storage.ts  vault.ts  googleAuth.ts  googleDrive.ts
    googleCalendar.ts  sync.ts  notifications.ts
  state/                       store applicativo, sessione, toast
  hooks/                       scorciatoie, inattività, media query…
  components/                  UI condivisa (TaskEditor, QuickAdd, CommandPalette, …)
  views/                       Oggi, Matrice, Lista, Calendario, Completati, Impostazioni
  styles/                      token e stili
```

## Fasi (un commit per fase)

1. **Scaffold**: Vite/React/TS strict, ESLint, Vitest, PWA, workflow Pages, `base: '/agenda/'`, porta 5173 strictPort.
2. **Dominio**: tipi versionati, migrazioni, urgenza, quadranti, parser aggiunta rapida, ricorrenze, merge + test.
3. **Sicurezza e storage**: crypto (PBKDF2 + AES-GCM), IndexedDB cifrato, schermata PIN (setup, sblocco, backoff, cambia PIN, PIN dimenticato), auto-blocco + test.
4. **Google**: GIS token model con rinnovo silenzioso e banner di riconnessione, Drive appData, motore di sync con stato, Calendar (lettura, crea/aggiorna/rimuovi evento solo su azione esplicita).
5. **UI e viste**: design system, layout con sidebar, Oggi, Matrice (dnd), Lista, Calendario settimana/mese (dnd), Completati, editor impegno, impostazioni.
6. **Esperienza**: aggiunta rapida (`N`), command palette (`Ctrl+K`), scorciatoie (`?`), toast con Annulla, notifiche, tema, export/import backup.
7. **Docs e rilascio**: README in italiano, verifica completa (build, lint, test, prova flussi con browser headless, offline), push su `main`, verifica deploy.
