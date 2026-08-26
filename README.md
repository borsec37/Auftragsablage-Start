# Auftragsablage-Start

Eine einzelne Webseite, die die **Auftragsablage WKT Air Solutions** auf dem eigenen
Rechner öffnet. Sie ist der Umweg, den SharePoint nötig macht: Dort lässt sich kein
Programm hinterlegen und kein Link auf ein eigenes Schema eintragen – ein Link auf eine
gewöhnliche `https://`-Seite dagegen schon. Diese Seite ruft dann `auftragsablage://start`
auf, und Windows startet die Anwendung.

Die Seite lädt nichts herunter und überträgt nichts. Sie besteht aus einer einzigen
HTML-Datei mit eingebettetem CSS und JavaScript – in einer Firmenumgebung ist nicht
gesagt, dass eine Seite Schriften oder Skripte von anderen Adressen nachladen darf.

## Inhalt

| Datei                   | Zweck                                                    |
|-------------------------|----------------------------------------------------------|
| `index.html`            | die Startseite, alles darin enthalten                     |
| `logo-weisstechnik.png` | Bildmarke im Kopf, wie in der Anwendung                    |
| `.nojekyll`             | GitHub Pages soll die Dateien unverändert ausliefern       |

## Einmalig einrichten: GitHub Pages einschalten

1. In diesem Repository auf **Settings** → in der linken Spalte **Pages**.
2. Unter *Build and deployment* → *Source*: **Deploy from a branch**.
3. *Branch*: **main**, Ordner **/ (root)** → **Save**.
4. Nach ein bis zwei Minuten ist die Seite erreichbar unter:

   ```
   https://borsec37.github.io/Auftragsablage-Start/
   ```

Das Repository ist öffentlich – nötig, damit GitHub Pages ohne kostenpflichtigen Plan
funktioniert. Auf der Seite steht nichts Vertrauliches: kein Pfad, kein Name, keine
Adresse aus dem Haus. Der Link öffnet ausschließlich eine Anwendung, die auf dem Rechner
des Benutzers ohnehin schon liegt.

## In SharePoint hinterlegen

Die Adresse von oben als gewöhnlichen Hyperlink eintragen – zum Beispiel:

* **Schnellstart / Menü links:** *Bearbeiten* → *Link hinzufügen* → Adresse einfügen,
  Anzeigename etwa „Auftragsablage starten“.
* **Auf einer Seite:** Webpart *Link* oder *Schnellzugriff*, dieselbe Adresse.
* **In der Dokumentbibliothek:** *Neu* → *Link*, dieselbe Adresse.

Die Auftragsablage.cmd am besten daneben legen: Wer die Anwendung noch nie gestartet hat,
braucht sie einmal – danach kennt Windows den Link.

## Was beim Klick geschieht

1. Der Benutzer klickt in SharePoint auf den Link, die Seite öffnet sich im Browser.
2. Die Seite ruft `auftragsablage://start` auf.
3. Der Browser fragt beim ersten Mal nach („Auftragsablage öffnen?“).
4. Windows führt den Befehl aus, der beim Einrichten unter
   `HKEY_CURRENT_USER\Software\Classes\auftragsablage` hinterlegt wurde – das ist die
   `Auftragsablage.cmd`. Sie sieht erst nach einer neueren Fassung und startet dann.

Die aufgerufene Adresse wird nicht an die Anwendung weitergereicht. Für dieses Schema
gibt es nur die eine Aktion; eine Webseite kann der Anwendung also nichts unterschieben.

Meldet sich nichts, blendet die Seite nach wenigen Sekunden eine kurze Anleitung ein.
Ob der Start wirklich fehlgeschlagen ist, kann eine Webseite nicht sicher feststellen –
deshalb steht dort eine Frage und keine Fehlermeldung.

## Grenzen

* **Nur Windows.** Das Schema wird beim ersten Start der Auftragsablage eingetragen.
* **Nicht eingebettet.** Aus einem in SharePoint eingebetteten Rahmen heraus lässt sich
  kein Programm starten. Die Seite erkennt das und bietet stattdessen an, sich in einem
  eigenen Tab zu öffnen.
* **Erst nach dem ersten Start.** Auf einem frischen Rechner muss `Auftragsablage.cmd`
  einmal von Hand ausgeführt werden.

## Ändern

`index.html` bearbeiten, committen, pushen – GitHub Pages veröffentlicht den neuen Stand
nach kurzer Zeit von selbst. Zum Ausprobieren genügt ein lokaler Webserver, etwa
`python3 -m http.server`; über `file://` verhalten sich Browser bei eigenen Schemata anders.
