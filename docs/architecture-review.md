# Architecture Review

**Datum:** 2026-07-23
**Reviewer:** Claude Code (Chief Software Architect Rolle für dieses Projekt)
**Scope:** Vollständiges Repository zum Zeitpunkt dieses Reviews (Commit `ad7f74b` und Vorgänger)
**Änderungen während dieses Reviews:** Keine. Dieses Review ist eine reine Lese-Analyse — siehe [Methodik](#methodik) und [Während des Reviews vorgenommene Änderungen](#während-des-reviews-vorgenommene-änderungen).

---

## Methodik

Jede Aussage in diesem Dokument stützt sich auf eine der folgenden, tatsächlich ausgeführten Prüfungen — nicht auf Erinnerung oder Annahme:

- Direktes Lesen und `grep`-Durchsuchen von `index.html`, `vision.html`, `support.js`, `image-slot.js`, `_ds/industry-.../styles.css`.
- `ls -la` / Dateigrößen-Messung für alle Assets.
- `gh api` Abfragen des tatsächlichen GitHub-Pages-Zustands (Zertifikat, `custom_404`, Domain-Status).
- Historisches Wissen aus der tatsächlichen Entwicklung dieses Repositories in dieser Session (u. a. die Kontrast-Fixes, der Jekyll-`_ds/`-Bug, die Mobile-Sticky-Saga, der Hakenkreuz-Fund) — jeweils mit Verweis auf den entsprechenden Commit, wo relevant.
- Wo eine Aussage nicht durch eine dieser Prüfungen belegt werden konnte, ist sie **explizit als Annahme oder offene Frage gekennzeichnet** — nicht als Tatsache dargestellt. Diese Kennzeichnung erscheint durchgängig als „⚠️ Annahme" oder „❓ Offene Frage".

**Wichtigster Rahmen für die gesamte Bewertung:** Dieses Repository enthält aktuell **ausschließlich** eine statische Marketing-/Demo-Website (`index.html`, `vision.html`, das Designsystem in `_ds/`). Es existiert **kein** Backend, keine Datenbank, keine Authentifizierung, kein Build-System. Für die meisten der 21 nachfolgend geforderten Bereiche lautet der ehrliche „Aktueller Zustand" deshalb: *existiert nicht*. Das wird pro Bereich explizit ausgewiesen statt stillschweigend übersprungen — ein Bereich, der „nicht existiert", ist trotzdem bewertungsrelevant, weil er die Eignung als Produkt-Grundlage direkt beantwortet.

---

## Bewertungen (0–100)

Jede Bewertung nennt explizit, **worauf sie sich bezieht** — bei den meisten Dimensionen unterscheidet sich der Wert stark je nachdem, ob die aktuelle statische Seite oder die (nicht existierende) Produktplattform gemeint ist. Wo eine Dimension für die Plattform schlicht nicht bewertbar ist, steht das da, statt eine Zahl zu erfinden.

| Dimension | Score | Bewertungsbasis & Begründung |
|---|---:|---|
| **Wartbarkeit** | 58 / 100 | Bezieht sich auf den aktuellen Code. Designsystem ist sauber token-basiert und hat sich unter realer Änderungslast (mehrere Kontrast-Fixes in dieser Session) bewährt (+). Abgezogen für: keinerlei Tooling-Absicherung (kein Lint, kein Format-Check), starke Inline-Style-Duplizierung (dc-runtime-Format-bedingt), und den konkret gefundenen 5-fach duplizierten SVG-Diagramm-Block in `vision.html` (Bereich 14). |
| **Sicherheit** | 72 / 100 | Bezieht sich auf die aktuelle Seite (keine Nutzerdaten, keine Formulare, kein Server). Angriffsfläche ist strukturell klein (+). Abgezogen für zwei konkrete, verifizierte Funde: ungepinnte Lucide-CDN-Abhängigkeit ohne SRI (Bereich 3/18) und hotlinked Drittanbieter-Bilder (Bereich 6). Für die Produktplattform ist dieser Wert **nicht anwendbar** — dort existiert noch keine Angriffsfläche zu bewerten. |
| **Testbarkeit** | 45 / 100 | Bezieht sich auf die Eigenschaft des vorhandenen JS-Codes, nicht auf tatsächliche Testabdeckung (separat bewertet). Die Scroll-/Phasenlogik in `vision.html` ist als isolierbare, reine Funktion geschrieben (+) — mittelmäßiger Wert trotzdem, weil komplett ohne Testharness, Fixtures oder etablierte Konvention. |
| **Skalierbarkeit** | 90 / 100 (aktuelle Seite) / **nicht bewertbar** (Plattform) | GitHub Pages skaliert für eine statische Seite praktisch beliebig — echte Stärke, aber nahezu bedeutungslos als Aussage über die künftige Backend-Skalierbarkeit, zu der schlicht keine Information existiert. |
| **Performance** | 62 / 100 | Gestützt auf gemessene Lighthouse-Werte dieser Session (`vision.html`, Mobile-Emulation: Performance ≈ 69, Accessibility 100, SEO 100, Best Practices 96) und den in diesem Review neu gefundenen, zum Zeitpunkt der Lighthouse-Messung schon vorhandenen ~1,8 MB schweren, unoptimierten Bildern (Bereich 12) — der gemessene Wert berücksichtigt deren Einfluss vermutlich bereits, daher keine zusätzliche Abwertung, aber auch keine Aufwertung. |
| **Erweiterbarkeit** | 30 / 100 | Das dc-runtime-Format ist im eigenen Projekt bereits als Sackgasse für Handerweiterung dokumentiert (`docs/architecture.md`, „What NOT to do"). Die 5-fache Diagramm-Duplizierung (Bereich 14) ist ein direktes, beobachtbares Symptom fehlender Erweiterbarkeit. |
| **Developer Experience** | 50 / 100 | Sehr schnell für triviale Änderungen (kein Build-Schritt) (+), aber jede Verifikation ist manuell, nicht skriptgestützt, nicht reproduzierbar dokumentiert (Bereich 15). |
| **Architektur-Konsistenz** | 70 / 100 | Bezieht sich auf das, was an Architektur tatsächlich existiert (im Wesentlichen: das Designsystem) — dort hohe Konsistenz (85+), aber der Gesamtscope ist so klein, dass „Konsistenz" kaum getestet wurde. Zwei konkrete Abweichungen gefunden (`#fff` statt Token, Bereich 20). |
| **Produktionsreife** | 80 / 100 (als Marketing-/Demo-Seite) / **0** (als Produkt) | Die Seite ist live, funktioniert, erfüllt ihren tatsächlichen Zweck zuverlässig. Als Aussage über Produktreife ist der Wert **0**, explizit getrennt ausgewiesen, um Fehlinterpretation auszuschließen. |

---

## Bereich 1: Gesamtarchitektur

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Statische Zwei-Seiten-Website (`index.html`, `vision.html`), gerendert über einen vendorten, generierten Runtime (`support.js`, Header: „GENERATED from dc-runtime/src/*.ts — do not edit"), der ein `<x-dc>`-Custom-Element-Format zur Laufzeit gegen dynamisch von `unpkg.com` geladenes React rendert. Kein Applikations-Framework, kein Produkt-Backend. |
| **Stärken** | Extrem einfach, funktioniert zuverlässig, null Infrastruktur-Overhead, schnelle Iteration für Marketing-Inhalte. |
| **Schwächen** | `support.js` ist ein Artefakt des externen Design-Tools, nicht eine bewusst gewählte Anwendungsarchitektur — nicht für Handerweiterung gedacht. |
| **Risiken** | Wird das Muster naiv fortgesetzt (weitere handgeschriebene `.dc.html`-artige Seiten), entsteht ein nie bewusst gewähltes Schatten-Framework. |
| **Betroffene Dateien** | `index.html`, `vision.html`, `support.js`, `_ds/` |
| **Empfohlene Maßnahmen** | Aktuelle Seite strikt als Marketing-/Demo-Oberfläche behandeln; Produktplattform als eigenständiges, bewusst architektiertes Repository starten (siehe `docs/architecture.md`); keine weiteren `.dc.html`-Seiten von Hand bauen. |
| **Priorität** | P1 — muss vor Beginn der Produktentwicklung kommuniziert/entschieden sein, damit niemand versehentlich den dc-runtime als Plattform-Basis fortführt. |
| **Aufwand** | XS (Entscheidung/Kommunikation; der eigentliche Plattform-Aufbau ist unter Bereich 21 mit XL bewertet) |
| **Konsequenz ohne Maßnahme** | Schleichende Fortführung eines nie gewählten Frameworks als De-facto-Standard. |

## Bereich 2: Modulgrenzen

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Keine Code-Module existieren. Einzige modulartige Einheit: `_ds/industry-.../` als in sich geschlossener Ordner mit eigenem Manifest und Readme. |
| **Stärken** | Das Designsystem ist bereits sauber isoliert — ein echtes, gutes Beispiel für eine Grenze, die funktioniert. |
| **Schwächen** | Kein JS-Modulsystem (kein ESM, kein Bundler) — `support.js` (64 KB) und `image-slot.js` (63 KB) sind monolithische Global-Scope-Dateien. |
| **Risiken** | Für die aktuelle Seite keine. Für die Plattform: die in `docs/coding-standards.md` verlangten Modulgrenzen-Regeln existieren nur als Dokument, keine Lint-Durchsetzung ist konfiguriert (es gibt noch kein Lint-Setup überhaupt). |
| **Betroffene Dateien** | `support.js`, `image-slot.js` |
| **Empfohlene Maßnahmen** | Vendorte Dateien unverändert lassen; bei Plattform-Start Modulgrenzen-Lint-Regeln (z. B. `eslint-plugin-boundaries`) von Commit 1 an einrichten, nicht nachträglich. |
| **Priorität** | P2 (aktuell nicht dringend) → wird P0, sobald Backend-Arbeit beginnt. |
| **Aufwand** | S (initiale Lint-Konfiguration) |
| **Konsequenz ohne Maßnahme** | Modulgrenzen bleiben unverbindliche Konvention statt durchgesetzter Regel — genau das Erosionsmuster, vor dem `docs/constitution.md` warnt. |

## Bereich 3: Abhängigkeiten

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Kein `package.json`, keine npm-Abhängigkeiten. Vier Laufzeit-Abhängigkeiten, alle CDN-geladen (unpkg.com): React 18.3.1, ReactDOM 18.3.1, Babel-standalone 7.29.0 — alle drei **versionsgepinnt und SRI-abgesichert** (verifiziert: `REACT_SRI`, `REACT_DOM_SRI`, `BABEL_SRI` in `support.js`). Lucide dagegen als `lucide@latest` **ohne Versionspinning und ohne SRI-Hash**, identisch auf beiden Seiten. |
| **Stärken** | Abhängigkeitszahl minimal; das React/ReactDOM/Babel-Loading ist vorbildlich gemacht (gepinnt + integritätsgesichert). |
| **Schwächen** | Lucide ist der Ausreißer im selben File, mit demselben Muster verfügbar, aber nicht angewendet. |
| **Risiken** | (a) Ein breaking Lucide-Release bricht Icons in Produktion, unbemerkt, ohne CI, die es fangen würde; (b) fehlende SRI bedeutet: eine kompromittierte/manipulierte Antwort von unpkg.com für Lucide würde ungeprüft im Browser ausgeführt. |
| **Betroffene Dateien** | `index.html:15`, `vision.html:13` (`<script src="https://unpkg.com/lucide@latest">`) |
| **Empfohlene Maßnahmen** | Lucide auf exakte Version pinnen + SRI-Hash ergänzen, exakt nach dem Muster, das im selben `support.js` für React/ReactDOM/Babel bereits existiert. |
| **Priorität** | P1 — konkretes, aktuell live bestehendes Risiko, sehr günstig behebbar. |
| **Aufwand** | XS |
| **Konsequenz ohne Maßnahme** | Stilles Brechen bei einem Lucide-Release oder unentdeckte Supply-Chain-Kompromittierung. |

## Bereich 4: Datenmodell

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Existiert nicht im Code. Keine Datenbank, kein ORM. Die einzigen „Daten" sind hartkodierte JS-Arrays (`renderVals()`) mit Demo-Inhalten (Projekte, Events, Testimonials) direkt in der Rendering-Logik jeder Seite. Das *Ziel*-Domänenmodell ist in `docs/domain-model.md` spezifiziert — das ist Dokumentation, keine Implementierung. |
| **Stärken** | Für aktuellen Code nicht anwendbar. Das dokumentierte Zielmodell (12 Entitäten, Mermaid-ER-Diagramm) ist inhaltlich fundiert. |
| **Schwächen** | Die `renderVals()`-Arrays sind unstrukturierte Inline-Literale ohne Schema/Validierung — für eine Demo-Seite unproblematisch, würde als Muster für die Plattform `docs/product-principles.md` Prinzip 3 („strukturierte Daten vor Freitext") direkt verletzen. |
| **Risiken** | Hauptrisiko ist eine Verwechslung: „wir haben `domain-model.md` geschrieben" ≠ „das Datenmodell existiert". Es existiert nicht, in keiner Form. |
| **Betroffene Dateien** | `index.html`, `vision.html` (inline `renderVals()`) |
| **Empfohlene Maßnahmen** | Für aktuelle Seite: keine. Für Produkt: `domain-model.md` als Drizzle-Schema implementieren, beginnend mit `Account`/`Person`/`Organization`/`Membership` gemäß der im Engineering Report festgelegten Reihenfolge. |
| **Priorität** | P0 für das Produkt (alles andere hängt davon ab) — N/A für aktuelle Seite. |
| **Aufwand** | L (vollständiges Schema + Migrationen für das initiale Modul-Set) |
| **Konsequenz ohne Maßnahme** | Startlinie für sämtliche Produktarbeit verzögert sich; alles Nachgelagerte verzögert sich mit. |

## Bereich 5: Authentifizierung und Autorisierung

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Existiert nicht. Kein Login, keine Sessions, keine geschützten Routen. Einziges „Identitäts"-Konzept: GitHub-Push-Zugriff und Domain-Kontrolle bei Strato — beides außerhalb des Codes. |
| **Stärken** | N/A — für eine Demo-Seite ohne Nutzerdaten korrekt, dass nichts existiert. |
| **Schwächen** | N/A für aktuellen Scope. |
| **Risiken** | Kein Branch-Protection auf `main` (verifiziert in früherer Deployment-Arbeit: direkter Push deployt sofort). Bei fehlenden Nutzerdaten geringes Risiko — aber genau diese Gewohnheit darf nicht ins Produkt-Repository übernommen werden. |
| **Betroffene Dateien** | N/A (GitHub-Repository-Einstellung, keine Datei) |
| **Empfohlene Maßnahmen** | Branch-Protection + Pflicht-Review vor dem ersten PR im Produkt-Repository einrichten — nicht für die aktuelle Seite dringend. |
| **Priorität** | P1 für das künftige Produkt-Repository; P3 für aktuelle Seite (optionale Härtung). |
| **Aufwand** | XS |
| **Konsequenz ohne Maßnahme** | Das Produkt-Repository erbt das „jeder mit Push-Zugriff deployt direkt" -Muster — bei echten Nutzerdaten ein echtes Risiko. |

## Bereich 6: Sicherheit

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | HTTPS erzwungen (verifiziert: `https_enforced: true`, gültiges Zertifikat bis 2026-10-15). Keine Formulare, keine Server-seitige Verarbeitung — strukturell kleine Angriffsfläche. Konkrete Funde: (a) ungepinnte Lucide-CDN-Abhängigkeit (Bereich 3); (b) kein Content-Security-Policy-Header möglich (GitHub-Pages-Plattformgrenze, kein behebbarer Code-Fehler); (c) sieben Partner-Logo-Bilder werden direkt von einer externen, nicht zu diesem Projekt gehörenden WordPress-Seite (`smartest.network/wp-content/uploads/...`) eingebettet (Hotlinking), statt im eigenen `assets/`-Ordner zu liegen. |
| **Stärken** | SRI auf 3 von 4 CDN-Skripten; durchgesetztes HTTPS; keine Secrets im Repository (verifiziert); minimale serverseitige Angriffsfläche (keine). |
| **Schwächen** | Siehe Funde oben. |
| **Risiken** | Hotlinked Bilder können ohne Wissen dieses Projekts verschwinden oder sich ändern und die „Vertrauens-Logos"-Sektion unbemerkt brechen — keine CI würde das auffangen. |
| **Betroffene Dateien** | `index.html:93-99` (hotlinked Partner-Logos), `index.html`/`vision.html` (Lucide-CDN-Tag) |
| **Empfohlene Maßnahmen** | Partner-Logos herunterladen und in `assets/` selbst hosten; Lucide pinnen + SRI (Querverweis Bereich 3). |
| **Priorität** | P2 (Hotlinking-Fix — nicht dringend, aber günstig und entfernt eine unnötige externe Abhängigkeit). |
| **Aufwand** | XS (7 kleine Bilddateien herunterladen, `src`-Attribute anpassen) |
| **Konsequenz ohne Maßnahme** | Die Vertrauens-Logo-Sektion kann still brechen oder sich unbeabsichtigt ändern, ohne dass es automatisch auffällt. |

## Bereich 7: Datenschutz

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Umami-Analytics (self-hosted) — kein sichtbarer Cookie-/Consent-Banner auf dieser Domain (verifiziert: kein „cookie"/„consent"-String in beiden Seiten oder `support.js`). Die Footer-Links „Datenschutz" und „Impressum" verweisen auf **externe, nicht zu diesem Projekt gehörende Seiten einer anderen echten, live betriebenen Firma** (`smartest.network/629-2-2/` bzw. `/impressum-2-2/`). |
| **Stärken** | Self-hosted Analytics vermeidet das verbreitetste Datenschutzproblem (Versand an große US-Adtech-Plattformen). |
| **Schwächen** | Die Datenschutz-/Impressum-Verlinkung ist ein konkretes, reales Problem: ein Besucher, der „Impressum" klickt (in Deutschland/EU ein rechtlich bedeutsamer Link), landet auf der Rechtsseite eines fremden Unternehmens — nicht auf etwas, das diese Demo-Seite selbst beschreibt. |
| **Risiken** | Mögliche rechtliche Exposition, falls diese Domain als „echte", öffentlich erreichbare Seite mit Impressumspflicht (TMG/DDG) gilt — **❓ Offene Frage, nicht technisch von mir zu beurteilen, benötigt juristische Einschätzung**; Verwirrung von Besuchern, die auf die Rechtsseiten eines unbeteiligten Unternehmens geleitet werden. |
| **Offene Frage / Annahme** | ⚠️ Ob die aktuelle Umami-Konfiguration einen Cookie-Consent-Banner nach ePrivacy/TTDSG erfordert, hängt von der genauen Tracking-Methode ab (cookielose Hashing- vs. persistente Identifier-Variante). Ich habe die konkrete Umami-Instanz-Konfiguration nicht geprüft und behaupte hier keine Gewissheit in keine Richtung. |
| **Betroffene Dateien** | `index.html:355-356`, `vision.html:353-354` |
| **Empfohlene Maßnahmen** | Externe Datenschutz-/Impressum-Links durch echte, projekteigene Seiten ersetzen, oder bis dahin durch einen klar erkennbaren Platzhalter (im Einklang mit dem bereits vorhandenen „Demo-Website"-Banner) ersetzen — keinesfalls weiter auf die Rechtsseiten eines unbeteiligten Dritten verweisen. |
| **Priorität** | **P1** — konkreter, aktuell live bestehender Fund mit plausibler rechtlicher/Vertrauens-Exposition, kein reines Stil-Detail. |
| **Aufwand** | XS (zwei Links pro Seite austauschen) |
| **Konsequenz ohne Maßnahme** | Besucher können über den Betreiber der Seite getäuscht werden; bei Indexierung/Bewerbung über den internen Rahmen hinaus eine ungeklärte Compliance-Frage. |

## Bereich 8: Fehlerbehandlung

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Der dc-runtime (`support.js`) enthält eine eingebaute React-Error-Boundary (`componentDidCatch`, verifiziert), die Rendering-Fehler innerhalb eines `<x-dc>`-Components auffängt, statt die ganze Seite abstürzen zu lassen — geerbt vom vendorten Runtime, nicht selbst geschrieben. Kein projekteigenes Error-Handling. Kein eigenes 404-Layout (verifiziert: `custom_404: false`) — GitHub Pages liefert die generische Standard-404-Seite. |
| **Stärken** | Das geerbte Error-Boundary ist ein echtes, funktionierendes Sicherheitsnetz. |
| **Schwächen** | Kein projekteigenes Error-Handling; ungetestet, wie sich ein Ausfall einer externen Abhängigkeit (z. B. Lucide-CDN) tatsächlich auswirkt. |
| **Risiken** | ❓ Offene Frage: unklares Degradationsverhalten bei CDN-Ausfall — in diesem Review nicht verifiziert, weder bestätigt noch ausgeschlossen. |
| **Betroffene Dateien** | `support.js` (Error-Boundary), keine projekteigene Entsprechung |
| **Empfohlene Maßnahmen** | Eigene `404.html` im Repo-Root ergänzen (von GitHub Pages nativ unterstützt, praktisch kostenlos); einmalig gezielt testen (Playwright, CDN-Requests blockieren), wie sich ein Lucide-/React-CDN-Ausfall tatsächlich auswirkt. |
| **Priorität** | P2 (404-Seite) / P3 (Degradationstest). |
| **Aufwand** | XS / XS |
| **Konsequenz ohne Maßnahme** | Besucher mit Tippfehler in der URL sehen GitHubs generische 404 statt einer eigenen — kleine Politur-Lücke, kein funktionales Risiko. |

## Bereich 9: Logging und Observability

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Ausschließlich Umami-Seitenaufruf-Analytics. Kein Error-Tracking, kein Performance-Monitoring, kein Uptime-Monitoring. |
| **Stärken** | Zumindest grundlegende Traffic-Sichtbarkeit vorhanden (viele statische Seiten haben nicht einmal das). |
| **Schwächen** | Ein defektes CDN-Skript oder ein JS-Fehler wäre unsichtbar, außer jemand prüft manuell. |
| **Risiken** | Stilles Brechen — dieses Projekt hat bereits einen realen Vorfall dieser Art erlebt (der Jekyll-`_ds/`-Ausschluss-Bug, der die gesamte Seite ungestylt in Produktion zeigte) und hätte ihn ohne manuelle Live-Prüfung nicht bemerkt. Nichts automatisiert diese Prüfung heute. |
| **Betroffene Dateien** | `index.html`, `vision.html` (nur Umami-Tag) |
| **Empfohlene Maßnahmen** | Einfachen/kostenlosen Uptime-Check ergänzen (externer Monitor auf die Live-URL); vollständiges Error-Tracking (Sentry-artig) zurückstellen, bis clientseitige Logik komplex genug dafür ist. |
| **Priorität** | P2 (Uptime-Check) / P3 (Error-Tracking). |
| **Aufwand** | XS |
| **Konsequenz ohne Maßnahme** | Ausfälle oder Regressionen werden von Besuchern entdeckt, nicht vom Team — ist in der Historie dieses Projekts bereits fast passiert. |

## Bereich 10: Testbarkeit

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Null automatisierte Tests. Die Scroll-/Phasenlogik in `vision.html` (`_initScrollDrivenState`, Phasenerkennung über `getBoundingClientRect`) ist als vergleichsweise reine, isolierbare Funktion geschrieben (verifiziert durch Lesen des Codes). |
| **Stärken** | Der Code selbst ist nicht testfeindlich — kein tief verschachtelter globaler Zustand. Eine echte, wenn auch bescheidene Stärke angesichts fehlender Testinfrastruktur. |
| **Schwächen** | Kein Test-Runner, keine Assertion-Bibliothek, nichts Ausführbares selbst wenn ein Test geschrieben würde. |
| **Risiken** | Jede Verhaltensgarantie dieses Codes (Reveal-Animationen, korrekte Phasenerkennung, browserübergreifendes SVG-`transform-box`-Verhalten) ist ausschließlich durch einmalige manuelle Playwright-Verifikation und eine Commit-Message abgesichert — keinerlei Regressionsschutz. Ein künftiger Edit könnte jeden der drei bereits einmal gefundenen und behobenen Bugs (Fast-Scroll-Phasensprung, Sticky/Pane-Überlappung, Firefox-`transform-origin`) unbemerkt reaktivieren. |
| **Betroffene Dateien** | `vision.html` (inline `<script type="text/x-dc" data-dc-script">`) |
| **Empfohlene Maßnahmen** | Nicht Gegenstand dieses Reviews (kein Refactoring). Empfehlung für später: Scroll-/Phasenlogik in ein kleines, eigenständiges, getestetes JS-Modul auslösen, statt ein volles Test-Framework für die ganze dc-runtime-Seite aufzusetzen. |
| **Priorität** | P2 (wird P1, sobald `vision.html`s Scroll-Logik erneut angefasst wird). |
| **Aufwand** | S |
| **Konsequenz ohne Maßnahme** | Jede künftige Berührung der Scroll-/Animationslogik riskiert eine stille Reaktivierung eines der drei bereits behobenen, nicht offensichtlichen Bugs. |

## Bereich 11: Testabdeckung

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | 0 % — es existieren keine Tests, die eine Abdeckung haben könnten. |
| **Stärken** | N/A. |
| **Schwächen** | 0 % Abdeckung, keine CI, die sie überhaupt messen/berichten könnte. |
| **Risiken** | Siehe Bereich 10. |
| **Betroffene Dateien** | N/A (keine Testdateien vorhanden) |
| **Empfohlene Maßnahmen** | Wie Bereich 10 — beginnend beim Scroll-Logik-Modul; Coverage-Reporting einrichten, sobald ein Test-Runner existiert, damit die Zahl sichtbar und nachverfolgbar ist statt angenommen. |
| **Priorität** | P2 |
| **Aufwand** | XS inkrementell, sobald der Harness aus Bereich 10 existiert |
| **Konsequenz ohne Maßnahme** | Unmessbar, unverwaltet — das Projekt kann ohne dies nicht erkennen, ob sich Qualität verbessert oder verschlechtert. |

## Bereich 12: Performance

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Gemessene Lighthouse-Werte aus dieser Session (`vision.html`, Mobile-Emulation): Accessibility 100, SEO 100, Best Practices 96, **Performance ≈ 69**. Bekannte Ursachen damals: render-blockierender Google-Fonts-`@import`, mehrere zur Laufzeit geladene CDN-Skripte (React/ReactDOM/Babel/Lucide) statt gebündelt. **Neuer Fund dieses Reviews:** `assets/hero-network.png` (1.862.160 Bytes) und `assets/founder-photo.png` (1.765.315 Bytes) — beide **~1,8 MB, vollständig unkomprimiert**, kein modernes Format (WebP/AVIF), kein responsive `srcset`, für dekorative Bilder auf einer Marketing-Seite. |
| **Stärken** | SEO- und Accessibility-Werte exzellent — die Performance-Lücke betrifft spezifisch Ladegeschwindigkeit, nicht Markup-Qualität. |
| **Schwächen** | Zwei ~1,8-MB-Bilder auf einer Zwei-Seiten-Site sind ein echter, leicht behebbarer Performance-Kostenfaktor. |
| **Risiken** | Auf langsamen Mobilverbindungen (plausible Zielgruppe: breiter deutscher Mittelstand, nicht nur urbane High-Speed-Nutzer) können die beiden mehrere-Megabyte-Bilder mehrsekündige Ladezeiten für die Hero-Sektion bedeuten. |
| **Betroffene Dateien** | `assets/hero-network.png`, `assets/founder-photo.png`, `_ds/.../styles.css` (Google-Fonts-`@import`) |
| **Empfohlene Maßnahmen** | Beide Bilder komprimieren/verkleinern (voraussichtlich 80–90 % Größenreduktion ohne sichtbaren Qualitätsverlust bei den tatsächlich angezeigten Größen möglich) — eine sichere, nicht-refaktorierende Maßnahme; Google Fonts self-hosten als P2-Folgeschritt. |
| **Priorität** | **P1** (Bildoptimierung — real, aktuell live, hoher Impact, sehr geringes Risiko/Aufwand) / P3 (Font-Self-Hosting, CDN-vs-Bundle-Architekturfrage). |
| **Aufwand** | XS (Bildkompression) / M (Font-Self-Hosting) / L (Weg von Runtime-CDN-React — de facto „Next.js-Frontend bauen", bereits andernorts getrackt) |
| **Konsequenz ohne Maßnahme** | Weiterhin unnötig langsames Laden der Hero-Sektion, besonders auf Mobilgeräten — untergräbt direkt den „hochwertig wirken"-Anspruch aus `docs/ui-principles.md`. |

## Bereich 13: Skalierbarkeit

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Statische Seite auf GitHub Pages — skaliert praktisch beliebig über GitHubs CDN. |
| **Stärken** | Exzellent für den aktuellen Zweck — kein realistisches Traffic-Level bricht das. |
| **Schwächen** | N/A für aktuellen Scope. |
| **Risiken** | Keine für aktuelle Seite. Für das Produkt: **keine Information vorhanden** (kein Backend existiert) — bewusst als unbekannt/unbewertet ausgewiesen statt angenommen. |
| **Betroffene Dateien** | N/A |
| **Empfohlene Maßnahmen** | Keine jetzt nötig; erneut betrachten, sobald erste Backend-Module existieren und reale Lastannahmen getestet werden können. |
| **Priorität** | P3 (aktuell keine Sorge; für das Produkt zu früh für konkrete Planung). |
| **Aufwand** | N/A |
| **Konsequenz ohne Maßnahme** | Keine — vorzeitige Optimierung hier würde `docs/product-principles.md`s Einfachheitsprinzip verletzen. |

## Bereich 14: Wartbarkeit

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Designsystem gut organisiert und dokumentiert (`_ds/.../readme.md`), Tokens konsistent genutzt (verifiziert über mehrere Kontrast-Fixes in dieser Session). Gegengewicht: beide HTML-Seiten nutzen extrem viele lange Inline-`style`-Attribute statt CSS-Klassen (dc-runtime-Format-bedingt, kein Projekt-Entscheid) — `index.html` 36 KB, `vision.html` 32,5 KB Markup. |
| **Stärken** | Tokens sind echte Source-of-Truth, auch innerhalb von Inline-Styles konsistent referenziert — eine globale visuelle Änderung (z. B. der Akzentfarben-Kontrast-Fix dieser Session) erfordert weiterhin nur eine Änderung in `_ds/.../styles.css`. |
| **Schwächen** | Die zwei `#fff`-Hardcodes (Bereich 20) zeigen, dass die Token-Disziplin nicht 100 % lückenlos ist. Die Verbosität erschwert visuelles Scannen/Diffen; jeder der fünf Phasen-Diagramm-SVG-Blöcke in `vision.html` wiederholt nahezu identisches Markup fünfmal, ohne gemeinsame Abstraktion (dc-runtime hat kein Komponenten-/Partial-System). |
| **Risiken** | Die 5-fache Diagramm-Duplizierung ist ein **bereits real aufgetretenes** Fehlerrisiko: eine Änderung an den Knoten-Positionen erfordert Bearbeitung an bis zu 6 Stellen (geteilte Desktop-Rail plus 5 Mobile-Snapshots) — laut Projekt-Historie tatsächlich schon passiert, nicht hypothetisch. |
| **Betroffene Dateien** | `vision.html` (5× duplizierte Phasen-Diagramm-SVG-Blöcke), `index.html` |
| **Empfohlene Maßnahmen** | Keine für die aktuellen dc-runtime-Seiten empfohlen (das Format-Limit zu bekämpfen lohnt sich nicht für eine bald abzulösende Demo-Seite); dieser Duplizierungs-Schmerzpunkt ist selbst ein starkes, konkretes Argument für die Next.js-Migration (eine echte Komponente würde das Diagramm einmal parametrisieren statt es zu wiederholen). |
| **Priorität** | P3 für aktuelle Seite (akzeptierte, begrenzte Kosten). |
| **Aufwand** | N/A für aktuelle Seite |
| **Konsequenz ohne Maßnahme** | Fortgesetztes, begrenztes, kalkulierbares Duplizierungs-Fehlerrisiko auf den aktuellen Seiten. |

## Bereich 15: Developer Experience

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Kein Build-Schritt → sehr geringe Einstiegshürde. Aber: kein `package.json`, also kein skriptgestütztes `npm run dev`/`test`/`lint` — jede Session in der tatsächlichen Historie dieses Projekts startete manuell einen Python-HTTP-Server und startete ad hoc, nicht committete Playwright-Skripte aus einem Scratch-Verzeichnis. Kein Type-Checking, kein Linting (hätte z. B. den `#fff`-Fund in diesem Review automatisch verhindern können). |
| **Stärken** | Schnelle Iterationsschleife für die Art kleiner, visueller Änderungen, die diese Seite tatsächlich braucht. |
| **Schwächen** | Verifikationsaufwand ist vollständig manuell und nicht kodifiziert — die tatsächlich verwendeten Schritte (lokaler Server, Playwright über 3 Engines, Lighthouse) existieren nur als Wissen in vergangenen Sessions, nicht als lauffähiges Skript im Repository. |
| **Risiken** | Verifikationsqualität hängt vollständig davon ab, dass sich jemand erinnert, alles davon zu tun — der Prüfprozess selbst ist nicht reproduzierbar oder erzwingbar. |
| **Betroffene Dateien** | N/A (Abwesenheit von Tooling, keine konkrete Datei) |
| **Empfohlene Maßnahmen** | Minimales `package.json` mit Skripten, die das bereits bewährte Verifikationsmuster kapseln (`npm run serve`, ein eingecheckter Playwright-Smoke-Test über Konsolenfehler + grundlegende Lighthouse-Schwellenwerte über 3 Engines). Wandelt „stilles Wissen aus vergangenen Sessions" in einen committeten, lauffähigen Baustein um — eine Tooling-Ergänzung, kein Refactoring der Seite selbst. |
| **Priorität** | **P1** — günstig und entschärft jede künftige Änderung an dieser statischen Seite unabhängig vom Produkt-Zeitplan. |
| **Aufwand** | S |
| **Konsequenz ohne Maßnahme** | Verifikationsqualität bleibt davon abhängig, dass jemand denselben Ad-hoc-Playwright-Ansatz jedes Mal neu erinnert und reproduziert, ohne Konsistenzgarantie. |

## Bereich 16: Deployment

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | `git push` auf `main` = sofortiges Live-Deploy über GitHub Pages, verifiziert funktionierend, kein CI-Gate, keine Staging-Umgebung, kein Rollback außer `git revert`. DNS bei Strato, außerhalb des Repos/IaC (dokumentiert in `docs/deployment.md`, von hier aus nicht automatisierbar). Custom Domain live, HTTPS-Zertifikat gültig bis 2026-10-15 (verifiziert). |
| **Stärken** | Extrem einfach, hat sich in der gesamten Projekt-Historie zuverlässig bewährt, keine Infrastrukturkosten. |
| **Schwächen** | Keine Staging-Umgebung — jede Änderung wird erstmals in Produktion real getestet; nichts gated einen fehlerhaften Deploy automatisch. |
| **Risiken** | Ein fehlerhafter Push könnte die Live-Seite brechen, ohne dass etwas es vor echten Besuchern auffängt (bisher durch manuelle Live-Verifikationsdisziplin abgefedert — das ist eine Prozessgewohnheit, kein Sicherheitsnetz). |
| **Betroffene Dateien** | `.nojekyll`, `CNAME` |
| **Empfohlene Maßnahmen** | Für aktuelle Seite keine dringenden. Die eigentliche Deployment-Arbeit liegt vollständig im Aufbau der Ziel-Infrastruktur (OpenTofu/Ansible/Docker, `docs/deployment.md`), nicht in einem Fix der aktuellen Pipeline. |
| **Priorität** | P3 für aktuelle Seite; **P0**-Grundlage für das Produkt (Staging-Umgebung muss vor echten Nutzerdaten existieren). |
| **Aufwand** | N/A aktuell / L für Ziel-Infra-Aufbau |
| **Konsequenz ohne Maßnahme** | Akzeptables Risiko heute; nicht akzeptabel, unverändert ins Produkt übernommen (bereits in `security.md`/`deployment.md` festgehalten). |

## Bereich 17: Konfigurationsmanagement

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Existiert nicht — nichts ist konfigurierbar, die gesamte Seite ist statisch mit inline-Werten (bewusst so, über den Token-in-CSS-Ansatz). Keine Umgebungsvariablen, kein `.env`, keine Secrets. |
| **Stärken** | Nichts, was falsch konfiguriert werden könnte — eine echte, wenn auch minimale Stärke angesichts der Alternative (ein Ad-hoc-Konfigsystem für eine Zwei-Seiten-Site wäre reiner Overhead). |
| **Schwächen** | N/A für aktuellen Scope. |
| **Risiken** | Keine aktuell. Die Umami-Website-ID im Tracking-Skript ist ein hartkodierter, öffentlicher, nicht-geheimer Wert (bereits früher in diesem Projekt so behandelt — korrekt). |
| **Betroffene Dateien** | N/A |
| **Empfohlene Maßnahmen** | Für aktuelle Seite keine. Für Produkt: umgebungsvariablenbasierte Konfiguration (12-Factor-Stil) ab dem ersten Backend-Commit etablieren, mit committetem `.env.example` und gitignorter echter `.env`. |
| **Priorität** | **P0**-Grundlage für das Produkt (muss vor dem ersten echten Secret existieren); N/A für aktuelle Seite. |
| **Aufwand** | XS (Ersteinrichtung, Teil des Plattform-Skeletts) |
| **Konsequenz ohne Maßnahme** | Risiko, dass ein Secret unter Zeitdruck beim initialen Plattform-Aufbau versehentlich hartkodiert oder committet wird — ein sehr verbreiteter, vermeidbarer Fehler. |

## Bereich 18: Abhängigkeitsrisiken

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Genau 4 externe Laufzeit-Abhängigkeiten, alle CDN-geladen: React, ReactDOM, Babel-standalone (gepinnt + SRI-gesichert), Lucide (ungepinnt, ohne SRI). Google Fonts als 5. externe Laufzeit-Abhängigkeit (render-blockierend, kein Self-hosted-Fallback). |
| **Stärken** | Abhängigkeitszahl minimal — echte Stärke, kleine Angriffs-/Ausfallfläche per Konstruktion. |
| **Schwächen** | Single Point of Failure auf unpkg.com- und Google-Fonts-Verfügbarkeit für **das gesamte Seiten-Rendering** (nicht nur Icons) — falls unpkg.com nicht erreichbar ist, lädt React selbst nicht, und die ganze Seite rendert nicht. ❓ **Offene Frage**: Dieses konkrete Ausfallszenario wurde in diesem Review nicht getestet — weder bestätigt noch ausgeschlossen. |
| **Risiken** | Wie Bereich 3 (Lucide) plus das oben beschriebene, umfassendere Single-Point-of-Failure-Risiko, das die ganze Seite betrifft, nicht nur Icons. |
| **Betroffene Dateien** | `index.html`, `vision.html` (alle `<script src="https://...">` sowie der Google-Fonts-`@import`) |
| **Empfohlene Maßnahmen** | Wie Bereich 3 (Lucide-Pin+SRI); separat: gezielt testen (bisher nicht geschehen), was ein vollständiger unpkg.com-Ausfall tatsächlich bewirkt, und danach entscheiden, ob vollständiges Self-Hosting aller vier Skripte den Aufwand wert ist. |
| **Priorität** | P1 (Lucide-Fix) / P2 (Test + Entscheidung Self-Hosting). |
| **Aufwand** | XS / S |
| **Konsequenz ohne Maßnahme** | Unquantifizierte Single-Point-of-Failure-Exposition gegenüber der Verfügbarkeit eines Drittanbieters für die Kernfunktion der Seite — aktuell ungeprüft, wie schwerwiegend der tatsächliche Ausfall wäre. |

## Bereich 19: Barrierefreiheit

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | **Stärkster Bereich der gesamten Codebasis.** Lighthouse Accessibility 100 auf beiden Seiten (Stand dieser Session), erreicht durch reale, dokumentierte Fixes: systemischer Farbkontrast-Fehler bei `--color-accent`-Text (auf Designsystem-Ebene behoben), eine CSS-Spezifitäts-Falle, die Button-Text im Nav unbemerkt verdunkelte (gefunden und behoben), ein fehlendes `<main>`-Landmark auf `index.html` (gefunden und behoben). `prefers-reduced-motion`-Fallbacks für alle animierten Elemente vorhanden. |
| **Stärken** | Nachweislich rigoros, nicht nur behauptet — durch reale Bugfix-Historie belegt. |
| **Schwächen** | Verifikation ausschließlich automatisiert-tool-basiert (Lighthouse) — kein echter Assistive-Technology-Nutzertest (z. B. Screenreader-Nutzer, der die Seite tatsächlich navigiert), was automatisierte Tools strukturell nicht vollständig ersetzen können. |
| **Risiken** | Kein akutes Risiko; Hauptrisiko ist Score-Regression bei künftigen Änderungen ohne Re-Verifikation, da kein CI-Gate den 100er-Wert automatisch durchsetzt (Bezug zu Bereich 10/15 — kein Testharness macht dies zur geprüften Invariante statt zum einmalig verifizierten Zustand). |
| **Betroffene Dateien** | `_ds/.../styles.css` (Kontrast-Fixes), `index.html` (`<main>`-Landmark-Fix) |
| **Empfohlene Maßnahmen** | Lighthouse-Accessibility-Score-Check in CI verankern (sobald überhaupt CI existiert, Anknüpfung an Bereich 15); eine Runde echter Screenreader-Stichprobenprüfung (NVDA/VoiceOver) auf `vision.html`s Scroll-Story, da strukturell komplexeste Seite. |
| **Priorität** | P2 (CI-Durchsetzung) / P3 (echter AT-Test). |
| **Aufwand** | XS (CI-Check, sobald Harness existiert) / S (AT-Stichprobentest) |
| **Konsequenz ohne Maßnahme** | Der 100er-Wert ist eine Momentaufnahme, die bei der nächsten Content-Änderung unbemerkt erodieren kann — genau die Art Regression, die dieses Projekt bereits zweimal durch manuelle Sorgfalt gefangen und behoben hat, was nicht unbegrenzt skaliert. |

## Bereich 20: Konsistenz mit dem Designsystem

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Insgesamt hohe Konsistenz — jede Komponente auf beiden Seiten nutzt das Industry-Vokabular (`.blueprint`, `.tag`, `.btn-primary`, `var(--color-*)`). Zwei konkrete, verifizierte Abweichungen gefunden: (1) `color: #fff` zweimal hartkodiert (je einmal pro Seite, identische Instanz) im Demo-Banner-`<div>`, statt eines Tokens — das Designsystem-Readme verlangt explizit: „Never hard-code a hex... the tokens already carry" genau diesen Fall. (2) Die Partner-Logo-Sektion sitzt in einem `--color-accent-900`-gefüllten Band — das ist laut Readme eine **dokumentierte, zulässige Ausnahme** („the accent's own deep step may carry a full field where the deck's section dividers use it"), also **keine** Verletzung, sondern korrekte Anwendung. |
| **Stärken** | Die Token-Disziplin hat sich unter realer Last bewährt — mehrere Kontrast-/Accessibility-Fix-Runden in dieser Session griffen direkt ins Designsystem ein und propagierten konsistent auf beide Seiten, ohne Drift. |
| **Schwächen** | Die zwei `#fff`-Instanzen sind eine kleine, konkrete, leicht behebbare Verletzung der eigenen, expliziten Systemregel. |
| **Risiken** | Aktuell minimal (`#fff` und der Token für `--color-bg` sehen auf dem dunklen Banner-Hintergrund heute vermutlich identisch aus), aber: wird das Token-System je retuned (ein vom Readme selbst unterstützter Workflow: „edit the tokens... every page... reads from them"), würden diese zwei hartkodierten Stellen dem **nicht** folgen — genau die Garantie brechen, für die das Token-System existiert. |
| **Betroffene Dateien** | `index.html:22`, `vision.html:19` |
| **Empfohlene Maßnahmen** | `color: #fff` durch das passende Token ersetzen (voraussichtlich `var(--color-bg)`, analog zum Muster für Text-auf-Akzent an anderer Stelle im selben Stylesheet) — auf beiden Seiten, zwei triviale, sichere Zeilenänderungen. |
| **Priorität** | P2 (real, aber geringer aktueller Impact). |
| **Aufwand** | XS |
| **Konsequenz ohne Maßnahme** | Ein künftiges Token-Retuning aktualisiert diese zwei Stellen still nicht mit — eine kleine, aber vermeidbare Inkonsistenz, die der eigenen dokumentierten Systemregel widerspricht. |

## Bereich 21: Eignung für die geplante Produktentwicklung

| Aspekt | Inhalt |
|---|---|
| **Aktueller Zustand** | Die aktuelle Codebasis (statisches HTML/CSS/JS über dc-runtime) teilt **praktisch keine Technologie** mit dem Ziel-Stack (Next.js/NestJS/PostgreSQL/Drizzle/Keycloak, `docs/technology-stack.md`). Dies ist die Kernfrage, in die alle vorigen 20 Bereiche einfließen. |
| **Stärken** | Tatsächlich wiederverwendbar: (a) das Industry-Designsystem — Tokens/Komponentenvokabular direkt als CSS-Custom-Properties in das Next.js-Frontend portierbar, keine Wert-Neuerfindung nötig; (b) die Produkt-Erzählung/Texte (`vision.html`s Fünf-Stufen-Story, `index.html`s Positionierung) — verwendbarer Content, prägt die tatsächliche Produkt-UX-Sprache; (c) hart erarbeitetes, dokumentiertes Wissen über konkrete Cross-Browser-/Mobile-Fallstricke (Sticky-Positionierung + CSS-Grid-Containing-Block-Verhalten, SVG-`transform-box`-Browserunterschiede), bereits in `docs/ui-principles.md` festgehalten — wertvoll, obwohl der Code selbst nicht wiederverwendet wird. |
| **Schwächen** | Alles andere muss von null gebaut werden — der Rendering-Mechanismus, sämtliche Interaktivität, das komplette Backend, das komplette Domänenmodell, die Deployment-Pipeline, die Testinfrastruktur. Es gibt **keinen inkrementellen Migrationspfad** von den aktuellen Seiten zum Ziel-Frontend — es ist ein Neuaufbau der Präsentationsschicht, der zufällig Design-Tokens und Text wiederverwenden kann, keine Evolution der bestehenden HTML-Dateien. |
| **Risiken** | Das Hauptrisiko ist organisatorisch/Wahrnehmungsbedingt: „wir haben zwei gut aussehende Seiten live" mit „wir haben eine Grundlage für das Produkt" zu verwechseln. Die Funde dieses Reviews (kein geteilter Tech-Stack, kein Datenmodell, kein Auth, plus die konkreten neuen Funde zu Abhängigkeitsrisiken/Performance/Datenschutz) bestätigen und untermauern die ursprüngliche Einschätzung aus `docs/engineering-report.md` mit zusätzlicher, spezifischer Evidenz. |
| **Betroffene Dateien** | Gesamtes Repository (Scope-Aussage, kein Einzeldatei-Fund) |
| **Empfohlene Maßnahmen** | Exakt der in `docs/engineering-report.md` festgelegten Kurz-/Mittel-/Langfrist-Roadmap folgen: Produkt-Repository als sauberen, separaten Aufbau gegen `docs/architecture.md`s Zielarchitektur starten, dabei nur Design-Tokens und Text portieren — nicht den dc-runtime-Mechanismus oder eine Code-Datei. |
| **Priorität** | **P0** — dies ist der grundlegende strategische Befund, den das gesamte Review stützt. |
| **Aufwand** | XL (das ist buchstäblich „die Produktplattform bauen", kein Fix) |
| **Konsequenz ohne Maßnahme** | Würde man versuchen, das Produkt durch Erweiterung der aktuellen Seiten zu bauen, bedeutete das einen Kampf gegen das dc-runtime-Format für Dinge, für die es nie gedacht war (State-Management, Formulare, auth-geschützte Views, Datenabruf) — hohe, sich summierende Kosten, am Ende schlechter als ein sauberer Neuanfang, exakt wie im „What NOT to do"-Abschnitt von `docs/architecture.md` bereits festgehalten. |

---

## Priorisierte Maßnahmenübersicht

### Quick Wins (günstig, hoher Wert, sofort machbar — unabhängig vom Produkt-Zeitplan)

1. `color: #fff` → Design-Token ersetzen, beide Seiten (Bereich 20) — XS
2. Lucide-CDN-Skript pinnen + SRI-Hash ergänzen, beide Seiten (Bereich 3/18) — XS
3. `hero-network.png` und `founder-photo.png` komprimieren/optimieren (Bereich 12) — XS
4. Datenschutz-/Impressum-Links nicht länger auf fremde externe Firma verweisen lassen (Bereich 7) — XS
5. Partner-Logos selbst hosten statt Hotlinking (Bereich 6) — XS
6. Eigene `404.html` ergänzen (Bereich 8) — XS
7. `meta description` auch auf `index.html` ergänzen (aktuell nur auf `vision.html` vorhanden — Asymmetrie, Bereich 19/Konsistenz-Querschnitt) — XS

### Notwendige Grundlagen vor Phase 1

1. Produkt-Repository-Skelett (NestJS + Next.js + TypeScript strict) aufsetzen — `docs/architecture.md`
2. `package.json` + CI (Lint/Test) ab erstem Commit — `docs/coding-standards.md`, `docs/testing.md`
3. Docker Compose für lokale Entwicklung (Postgres/Valkey/Keycloak) — `docs/deployment.md`
4. Umgebungsvariablen-/Config-Muster (`.env.example`) etablieren (Bereich 17)
5. Branch-Protection + Pflicht-Review auf dem Produkt-Repository (Bereich 5)
6. ADR zur `TrustScore`-Berechnungslogik (offene Frage aus `domain-model.md`)
7. ADR zur DSGVO-Löschung-vs-Audit-Trail-Spannung (offene Frage aus `security.md`)

### Maßnahmen während Phase 1

1. `identity`- und `organizations`-Module zuerst implementieren (Reihenfolge aus `engineering-report.md`)
2. OpenTelemetry-Instrumentierung ab dem ersten Modul, nicht nachträglich
3. Unit-/Integrationstests parallel zu jedem entstehenden Modul
4. Staging-Umgebung auf Hetzner via OpenTofu/Ansible aufbauen

### Maßnahmen nach Phase 1

1. `skills`-, `verification`-, `relationships`-, `trust`-Module
2. Vollständiger Observability-Stack (Grafana/Loki/Prometheus-Dashboards)
3. PostHog-vs-Matomo-Entscheidung + Integration
4. Echter AT-(Screenreader-)Test auf der fertigen Produkt-UI

### Bewusst zurückgestellte Themen

1. Reichhaltigeres `Cooperation`-Entity (bereits in `domain-model.md` als offene Frage markiert)
2. `Event`-/`Content`-(Wissensbereich-)Entitäten — nicht modelliert, bewusst verschoben
3. Microservice-Extraktion aus dem modularen Monolithen — erst wenn ein Modul das konkret verdient (ADR-0001)
4. Vollständiges Self-Hosting aller CDN-Abhängigkeiten (React/Babel) — aktueller SRI-gepinnter Ansatz „gut genug" für jetzt
5. Empfehlungs-/Matching-„Intelligenz"-Stufe — bewusst zuletzt sequenziert gemäß `vision.md`s eigener Begründung

---

## Klassifikation der Funde

| Kategorie | Funde |
|---|---|
| **Tatsächliche Defekte** (real, aktuell live, unabhängig vom Produkt-Zeitplan behebbar) | `#fff` statt Token (Bereich 20); Lucide ohne Pin/SRI (Bereich 3/18); Datenschutz-/Impressum-Verlinkung auf fremde Firma (Bereich 7); unoptimierte ~1,8-MB-Bilder (Bereich 12); fehlende eigene 404-Seite (Bereich 8); fehlende `meta description` auf `index.html` (Bereich 19/Konsistenz) |
| **Technische Schulden** (bekannte, akzeptierte Abkürzungen, die Aufwand zur Behebung brauchen) | Null automatisierte Tests (Bereich 10/11); kein CI/CD für aktuelle Seite (Bereich 16); kein `package.json`/Tooling (Bereich 15); 5-fach duplizierter SVG-Diagramm-Block (Bereich 14); keine Branch-Protection (Bereich 5) |
| **Zukünftige Architekturentscheidungen** (noch nicht getroffen, bewusst offen) | `TrustScore`-Berechnungsalgorithmus; reichhaltigeres `Cooperation`-Modell; DSGVO-Löschung-vs-Audit-Trail-Auflösung; PostHog-vs-Matomo; vollständiges CDN-Self-Hosting vs. aktueller Ansatz; Modul-Extraktion aus dem Monolithen (falls/wann) |
| **Optionale Verbesserungen** (Nice-to-have, nicht erforderlich) | Echter Screenreader-AT-Testdurchlauf; Uptime-Monitoring; Font-Self-Hosting; CI-durchgesetzter Lighthouse-Floor |

---

## Während des Reviews vorgenommene Änderungen

**Keine.** Dieses Review bestand ausschließlich aus Lese-Operationen (Dateien lesen, `grep`, Dateigrößen prüfen, `gh api`-Abfragen). Es wurden keine Code-, Konfigurations- oder Dokumentationsänderungen am Repository vorgenommen — auch keine „kleinen, risikofreien" im Sinne der Aufgabenstellung, da keine für die Analyse oder Dokumentation selbst notwendig waren. Alle oben empfohlenen Maßnahmen (inkl. der Quick Wins) sind **Empfehlungen, keine bereits durchgeführten Änderungen** — sie warten auf explizite Freigabe.

---

## Abschließende Empfehlung

**Ist die bestehende Anwendung eine geeignete Grundlage?**
Für das Produkt (Backend, Datenmodell, Auth, Matching) — **nein**, es gibt keine Technologie-Überschneidung mit dem Zielstack, und keinen inkrementellen Migrationspfad (Bereich 21). Als Marketing-/Demo-Frontend und als Quelle wiederverwendbarer Design-Assets — **ja**, mit hoher Qualität, wie die Bewertungen in den Bereichen 12, 19 und 20 zeigen.

**Welche Probleme müssen vor Phase 1 behoben werden?**
Nicht die im aktuellen Code gefundenen Defekte (die sind unabhängig, niedrig-riskant, jederzeit behebbar, siehe Quick Wins) — sondern die unter „Notwendige Grundlagen vor Phase 1" gelisteten Punkte: Repository-Skelett, CI/Tooling ab Tag 1, Config-Management-Muster, Branch-Protection, sowie die beiden offenen ADR-Fragen (`TrustScore`-Berechnung, DSGVO-Löschung). Ohne diese beginnt Phase 1 mit genau den Lücken, die dieses Projekt in seiner bisherigen Historie bereits einmal (Testing/CI) schmerzhaft nachgeholt hat.

**Welche Teile können weiterverwendet werden?**
Das Industry-Designsystem (Tokens + Komponentenvokabular, direkt portierbar), die Produkt-Erzählung/Texte aus `vision.html`/`index.html`, und das dokumentierte Cross-Browser-/Mobile-Wissen in `docs/ui-principles.md`. Ebenso die im Projekt bereits gelebte Praxis strenger Live-Verifikation und Accessibility-Disziplin — als **Arbeitsweise**, nicht als Code.

**Welche Teile sollten ersetzt oder neu aufgebaut werden?**
Der dc-runtime-Rendering-Mechanismus, sämtliche Interaktivitäts-/Rendering-Logik, das komplette Backend/Datenmodell (100 % Neubau), und die Deployment-Pipeline für das Produkt (separat von der aktuellen statischen Seite, die unverändert für Marketing weiterlaufen kann).

**In welcher Reihenfolge sollte die technische Weiterentwicklung erfolgen?**
1. Quick Wins auf der aktuellen Seite (parallel möglich, keine Abhängigkeit zu Phase 1).
2. Grundlagen vor Phase 1 (Repository-Skelett, CI, Config, Branch-Protection, offene ADRs).
3. `identity`- und `organizations`-Module (Fundament für alles Weitere).
4. `skills`, `verification`, `relationships`, `trust` — in dieser Reihenfolge, da jedes auf dem vorigen aufbaut (`docs/domain-model.md`s Abhängigkeitsrichtung).
5. Observability-Vervollständigung, Produkt-Analytics-Entscheidung, Barrierefreiheits-AT-Test.
6. Zuletzt: die „Intelligenz"-Stufe (Matching/Empfehlungen) — bewusst zuletzt, weil sie auf verifizierten, strukturierten Daten aus allen vorigen Schritten aufbaut, nicht davor sinnvoll beginnbar ist.

Diese Reihenfolge ist identisch mit der bereits in `docs/engineering-report.md` festgelegten Roadmap — dieses Review bestätigt sie mit zusätzlicher, konkreter Evidenz, ändert sie nicht.
