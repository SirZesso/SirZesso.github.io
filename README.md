# Rezept-Links (#137)

Statische Webseite hinter den Links von "Rezept senden", ausgeliefert auf `https://hamsti.ch` (`RecipeLink.webHost`, `applinks:hamsti.ch` in `HAMSTi/HAMSTi.entitlements`).

- `.well-known/apple-app-site-association` — macht `https://hamsti.ch/r` zum Universal Link, der HAMSTi öffnet (Team `3PS8M5VAAE`). Muss per HTTPS, ohne Weiterleitung und als `application/json` kommen (`.htaccess` für Apache, `_headers` für Cloudflare Pages/Netlify).
- `index.html` — Landingpage (#143): Hamsti, Screenshots, Funktionen, Datenschutz-Hinweis, App-Store-Badge mit „Bald im App Store“. Bilder in `img/` (WebP, aus `AppStore/latest/screenshots/de` und den Ebenen von `HAMSTi App Icon.icon` verkleinert), `og-card.jpg` (1200 × 630) ist das Vorschaubild für Links auf die Startseite.
- `r/index.html` — für alle ohne HAMSTi: zeigt Emoji, Name und Absender aus dem Link und führt zum Download. Das Rezept steckt im Fragment (`#…`), das der Browser nie an den Server schickt. Die Seite speichert nichts und zeigt keine Zutaten oder Schritte.

## Einrichten

### GitHub Pages mit eigener Domain (aktuell)

Ausgeliefert vom **öffentlichen** Repo `SirZesso.github.io` (User-Site), Inhalt von `web/` im Root, Pages aus `main`. `.nojekyll` sorgt dafür, dass `.well-known/` ausgeliefert wird; GitHub Pages liefert die App-Link-Datei ohne Endung als `application/json`.

1. **Domain bei GitHub verifizieren** (schützt vor Übernahme der Domain): GitHub → Settings (Profil) → Pages → „Add a domain“ → `hamsti.ch`. GitHub zeigt einen TXT-Eintrag `_github-pages-challenge-SirZesso.hamsti.ch`; den im Hostpoint Control Panel unter Domains → `hamsti.ch` → DNS anlegen, dann „Verify“.
2. **DNS bei Hostpoint** (gleiche Seite):
   - `hamsti.ch` A → `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - `hamsti.ch` AAAA → `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`
   - `www.hamsti.ch` CNAME → `sirzesso.github.io.`
   - Bestehende A/AAAA/CNAME-Einträge für `@` und `www` (Parkseite von Hostpoint) vorher löschen. MX-Einträge für E-Mail bleiben.
3. **Domain im Repo eintragen:** `SirZesso.github.io` → Settings → Pages → Custom domain `hamsti.ch` → Save. Das legt die Datei `CNAME` (Inhalt `hamsti.ch`, wie `web/CNAME`) an. Wenn `main` dort geschützt ist, `CNAME` per PR hinzufügen.
4. Warten, bis der DNS-Check grün ist und GitHub das Zertifikat ausgestellt hat (Minuten bis ein paar Stunden), dann „Enforce HTTPS“ einschalten.
5. Prüfen: `curl -i https://hamsti.ch/.well-known/apple-app-site-association` liefert Status 200, `content-type: application/json` und keine Weiterleitung. Apple holt die Datei über sein CDN: `curl -i https://app-site-association.cdn-apple.com/a/v1/hamsti.ch` zeigt, was Geräte bekommen (kann bis zu einem Tag hinterherhinken).

Mit der eigenen Domain leitet `sirzesso.github.io` auf `hamsti.ch` weiter. Früher verschickte Links auf `sirzesso.github.io/r#…` landen dadurch auf der Seite unter `hamsti.ch` (das Fragment bleibt erhalten) und öffnen HAMSTi über „In HAMSTi öffnen“, nicht mehr direkt. `applinks:sirzesso.github.io` und `RecipeLink.acceptedWebHosts` bleiben trotzdem drin, das schadet nicht.

`APP_STORE_ID` in `r/index.html` durch die ID aus App Store Connect ersetzen, das Smart-App-Banner-Meta-Tag einkommentieren, das Badge verlinken und den Hinweis „Bald im App Store“ entfernen. `app-icon.png` ist aus `HAMSTi App Icon.icon` gerendert, `share-card.jpg` (1200 × 630) ist das Vorschaubild für iMessage und Co.; seine URL in `og:image` ist absolut (`https://hamsti.ch/share-card.jpg`). `app-store-badge.svg` ist das unveränderte Badge aus Apples Marketing-Ressourcen.

### Klassisches Webhosting (Hoststar, Hostpoint)

1. Im Hosting-Panel die Domain `hamsti.ch` aufschalten und ein SSL-Zertifikat (Let's Encrypt) aktivieren. Liegt die Domain bei einem anderen Anbieter, dort die DNS-Einträge (A/AAAA) oder Nameserver auf das Hosting zeigen lassen.
2. Den **Inhalt** von `web/` per FTP/SFTP ins Webverzeichnis der Domain laden: `.well-known/`, `r/`, `.htaccess`. Ordner und Dateien mit Punkt am Anfang sind in vielen FTP-Programmen versteckt, versteckte Dateien einblenden.
3. Prüfen wie oben. `.htaccess` setzt den Content-Type unter Apache.

Ein Umzug zu einem anderen Hoster ist unkritisch: Dateien kopieren, DNS umstellen. Die Links bleiben gültig, solange die Domain gleich bleibt.

### Alternativ: Cloudflare Pages oder Netlify

Projekt aus diesem Repo mit `web/` als Ausgabeordner, kein Build-Schritt; `_headers` setzt dort den Content-Type.

Apple lädt die Datei über sein CDN; nach einer Änderung kann es bis zu einem Tag dauern, bis Geräte sie neu holen. Die App muss nach dem Hinzufügen der Domain einmal neu installiert werden.
