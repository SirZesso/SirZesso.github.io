# Rezept-Links (#137)

Statische Webseite hinter den Links von "Rezept senden", ausgeliefert auf `https://hamsti.ch` (`RecipeLink.webHost`, `applinks:hamsti.ch` in `HAMSTi/HAMSTi.entitlements`).

- `.well-known/apple-app-site-association` — macht `https://hamsti.ch/r` zum Universal Link, der HAMSTi öffnet (Team `3PS8M5VAAE`). Muss per HTTPS, ohne Weiterleitung und als `application/json` kommen (`.htaccess` für Apache, `_headers` für Cloudflare Pages/Netlify).
- `r/index.html` — für alle ohne HAMSTi: zeigt Emoji, Name und Absender aus dem Link und führt zum Download. Das Rezept steckt im Fragment (`#…`), das der Browser nie an den Server schickt. Die Seite speichert nichts und zeigt keine Zutaten oder Schritte.

## Einrichten

### Klassisches Webhosting (Hoststar, Hostpoint)

1. Im Hosting-Panel die Domain `hamsti.ch` aufschalten und ein SSL-Zertifikat (Let's Encrypt) aktivieren. Liegt die Domain bei einem anderen Anbieter, dort die DNS-Einträge (A/AAAA) oder Nameserver auf das Hosting zeigen lassen.
2. Den **Inhalt** von `web/` per FTP/SFTP ins Webverzeichnis der Domain laden: `.well-known/`, `r/`, `.htaccess`. Ordner und Dateien mit Punkt am Anfang sind in vielen FTP-Programmen versteckt, versteckte Dateien einblenden.
3. Prüfen: `curl -i https://hamsti.ch/.well-known/apple-app-site-association` liefert Status 200, `content-type: application/json` und keine Weiterleitung. `.htaccess` setzt den Typ unter Apache.
4. `APP_STORE_ID` in `r/index.html` durch die ID aus App Store Connect ersetzen, das Smart-App-Banner-Meta-Tag einkommentieren, das Badge verlinken und den Hinweis „Bald im App Store“ entfernen. `app-icon.png` ist aus `HAMSTi App Icon.icon` gerendert, `app-store-badge.svg` ist das unveränderte Badge aus Apples Marketing-Ressourcen.

Ein Umzug zu einem anderen Hoster ist unkritisch: Dateien kopieren, DNS umstellen. Die Links bleiben gültig, solange die Domain gleich bleibt.

### Vorübergehend: GitHub Pages (ohne eigene Domain)

Universal Links brauchen die App-Link-Datei im Root der Domain, deshalb ein **öffentliches** Repo namens `SirZesso.github.io` (User-Site, Adresse `https://sirzesso.github.io`). Inhalt von `web/` in den Root, **ohne** `CNAME`, Pages aus `main` aktivieren. `.nojekyll` sorgt dafür, dass `.well-known/` ausgeliefert wird. Die App erzeugt solange Links auf `sirzesso.github.io` (`RecipeLink.webHost`) und öffnet Links beider Domains.

Sobald `hamsti.ch` steht: `CNAME` hinzufügen (bei GitHub Pages) oder auf Hoststar/Hostpoint hochladen, `RecipeLink.webHost` auf `hamsti.ch` stellen. Die User-Site sollte danach weiterlaufen, damit früher verschickte Links gültig bleiben.

### Alternativ: Cloudflare Pages oder Netlify

Projekt aus diesem Repo mit `web/` als Ausgabeordner, kein Build-Schritt; `_headers` setzt dort den Content-Type.

Apple lädt die Datei über sein CDN; nach einer Änderung kann es bis zu einem Tag dauern, bis Geräte sie neu holen. Die App muss nach dem Hinzufügen der Domain einmal neu installiert werden.
