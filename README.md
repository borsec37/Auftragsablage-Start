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
5. Ist die Anwendung im Vordergrund, **schließt sich der Tab von selbst**.

Die aufgerufene Adresse wird nicht an die Anwendung weitergereicht. Für dieses Schema
gibt es nur die eine Aktion; eine Webseite kann der Anwendung also nichts unterschieben.

## Wie die Seite Erfolg und Fehlschlag auseinanderhält

Ob eine Anwendung wirklich gestartet ist, kann eine Webseite nicht abfragen. Sie muss sich
an das halten, was beim Start tatsächlich geschieht:

1. Die Seite ruft `auftragsablage://start` auf.
2. Der Browser fragt nach → **die Seite verliert den Fokus**.
3. Der Benutzer bestätigt → **der Fokus kommt sofort zurück**.
4. `cmd.exe`, PowerShell, Auspacken, Abgleich mit dem Ablageordner, Webserver starten →
   **das dauert Sekunden bis Minuten**.
5. Die Anwendung öffnet ihren eigenen Browsertab.

Schritt 3 ist die Falle: Der zurückkehrende Fokus sieht aus wie ein Abbruch, ist aber das
Gegenteil. Der Fokus taugt darum nicht als Erfolgsmerkmal – wohl aber **die Nachfrage aus
Schritt 2**, denn die kommt nur, wenn Windows das Schema kennt.

| Beobachtung                                        | Deutung                            | Was geschieht                          |
|----------------------------------------------------|------------------------------------|----------------------------------------|
| Keine Nachfrage innerhalb 3,5 s                     | Schema unbekannt                   | Anleitung zur Einrichtung               |
| Nachfrage kam, Tab wird **verdeckt**                | Anwendung hat ihren Tab geöffnet   | Tab schließt sich                       |
| Nachfrage kam, Fokus **12 s am Stück** weg          | Anwendung in einem anderen Fenster | Tab schließt sich                       |
| Nachfrage kam, nach 45 s immer noch nichts          | abgebrochen *oder* sehr langsam    | Anleitung, beide Möglichkeiten benannt  |

Nach neun Sekunden sagt die Seite von sich aus, dass der erste Start länger dauert –
ohne dieses Wort wirkt die Stille wie ein Fehlschlag.

Ob abgebrochen oder nur langsam, lässt sich von außen nicht unterscheiden. Die Seite rät
deshalb nicht, sondern nennt beides.

### Das Schließen hat eine Grenze

Ein Browser schließt per Skript nur Fenster, die er selbst geöffnet hat **oder die noch
keinen eigenen Verlauf angesammelt haben**. Praktisch heißt das:

* **Link öffnet einen neuen Tab** → der Tab schließt sich von selbst. ✅
* **Link öffnet im selben Tab** → das Schließen wird abgelehnt. Die Seite sieht das an
  `history.length` **vorher** und bietet dann gar keinen Knopf an, sondern die Anweisung
  „Diesen Tab können Sie jetzt schließen – am schnellsten mit Strg + W“. Ein Knopf, der
  nachweislich nichts tut, ist schlechter als ein Satz, der weiterhilft.

Nachgemessen: Weder `window.close()` noch der ältere Umweg `window.open('','_self').close()`
kommt an dieser Regel vorbei.

**Empfehlung:** Den Link in SharePoint so eintragen, dass er in einem neuen Tab öffnet.
Der Aufruf von `auftragsablage://start` legt selbst keinen Verlaufseintrag an – das wurde
nachgemessen –, ein frisch geöffneter Tab bleibt also schließbar.

### Wenn der Standardbrowser ein anderer ist

Die Anwendung öffnet ihre Oberfläche über `Start-Process` – also im **Standardbrowser**.
Ist das ein anderer als der, in dem der SharePoint-Link angeklickt wurde, wird der Tab
dieser Seite nie verdeckt. Dann greift die Frist über den Fokus (12 s), und die Seite
schließt sich trotzdem.

## Auftragsablage.cmd zum Herunterladen anbieten

Wer die Anwendung noch gar nicht hat, sieht in der Anleitung nur den Verweis „Sie finden
die Datei in SharePoint neben diesem Link“. Daraus wird ein Knopf, sobald in `index.html`
ganz oben im Skript eine Adresse hinterlegt ist:

```js
var DOWNLOAD = '';   // leer = nur der allgemeine Verweis
```

Aus SharePoint führen zwei Wege zu einer solchen Adresse:

**1. Freigabelink mit `&download=1`** – Link der Datei kopieren („Link kopieren“) und den
Zusatz anhängen. Ohne ihn öffnet sich erst die SharePoint-Ansicht statt des Downloads:

```
https://<tenant>.sharepoint.com/:u:/s/<Site>/<Kennung>?e=<...>&download=1
```

**2. Downloadadresse der Bibliothek** – unabhängig von einem Freigabelink, dafür muss der
Bibliothekspfad stimmen:

```
https://<tenant>.sharepoint.com/sites/<Site>/_layouts/15/download.aspx?SourceUrl=/sites/<Site>/<Bibliothek>/Auftragsablage.cmd
```

Beides setzt voraus, dass der Benutzer an SharePoint angemeldet ist – im Haus ist er das
üblicherweise. Wer keinen Zugriff hat, bekommt die Datei auch nicht; das ist so gewollt und
der Grund, warum hier eine interne Adresse stehen darf, obwohl die Seite öffentlich ist.

Der Knopf trägt bewusst **kein** `download`-Attribut: Bei einer fremden Adresse ignorieren
Browser es. Was den Download auslöst, ist die Adresse selbst.

### Wenn der Download-Link 404 liefert, obwohl die Datei da ist

Kein Widerspruch – SharePoint antwortet auf einen **ungültigen oder veränderten**
Freigabelink absichtlich mit 404 statt mit „Zugriff verweigert“, damit niemand durch
Ausprobieren herausfinden kann, welche Dateien überhaupt existieren.

Ein normaler Freigabelink endet auf `&e=<Token>` – dieses Token ist kryptografisch an
genau die Zeichenkette gebunden, mit der SharePoint es erzeugt hat. Wird danach noch
etwas angehängt (etwa `&download=1`), passt die Signatur nicht mehr, und SharePoint
liefert 404, obwohl die Datei unverändert existiert.

**Deshalb:** Den kopierten Freigabelink unverändert einsetzen, nichts anhängen. Für
eine `.cmd`-Datei gibt es ohnehin keine Web-Vorschau – SharePoint bietet meist von
selbst den Download an, auch ohne `download=1` oder `&web=1`.

Bricht das trotzdem ab, ist die **`download.aspx`-Variante** die robustere Wahl, weil
sie ohne signiertes Token auskommt – dafür muss der Bibliothekspfad exakt stimmen:

```
https://sgxs.sharepoint.com/sites/mts0344/_layouts/15/download.aspx?SourceUrl=/sites/mts0344/Auftraege/_Anwendung/Auftragsablage.cmd
```

Den Pfad am besten nicht selbst tippen, sondern in SharePoint bis zur Datei
durchklicken und aus der Adresszeile kopieren – sonst genau derselbe Fehler, nur mit
umgekehrter Ursache: ein Tippfehler statt eines kaputten Tokens, wieder 404.

### Nachsehen, was ein neuer Kollege liest

Die Anleitung bekommt nur zu Gesicht, wer die Anwendung **nicht** hat – wer sie hat, sieht
sie nie. Zum Nachschauen hängt man einen Parameter an die Adresse; dabei wird nichts
gestartet, und ein deutliches Band oben sagt, dass es eine Probeansicht ist:

| Adresse            | Zeigt                                                   |
|--------------------|---------------------------------------------------------|
| `?probe=fehlt`     | Anleitung zur Einrichtung – **mit dem Download-Knopf**    |
| `?probe=stumm`     | Anleitung, wenn sich die Anwendung nicht meldet           |
| `?probe=erfolg`    | die Erfolgsanzeige                                        |

Also zum Beispiel:

```
https://borsec37.github.io/Auftragsablage-Start/?probe=fehlt
```

Der Startknopf bleibt dabei bedienbar – ein Klick startet dann wirklich.

**Ganz echt nachstellen** geht auch, denn die Registrierung repariert sich bei jedem Start
von selbst:

```bat
reg export HKCU\Software\Classes\auftragsablage "%USERPROFILE%\Desktop\auftragsablage.reg"
reg delete HKCU\Software\Classes\auftragsablage /f
```

Danach verhält sich der Rechner wie einer ohne Auftragsablage. Zurück kommt der Eintrag
durch einen Doppelklick auf `Auftragsablage.cmd` – oder über die gesicherte `.reg`-Datei.

### Warum der Explorer der bessere Weg bleibt

Eine aus dem Browser heruntergeladene `.cmd` bekommt von Windows die Herkunftsmarkierung
„aus dem Internet“. Beim Ausführen meldet sich dann SmartScreen mit „Der Computer wurde
durch Windows geschützt“, und der Benutzer muss über *Weitere Informationen* → *Trotzdem
ausführen* gehen. Manche Firmenrichtlinien unterbinden den Download ausführbarer Dateien
ganz.

Dieselbe Datei aus dem **synchronisierten Ordner im Explorer** trägt diese Markierung
nicht – dort genügt ein Doppelklick. Deshalb nennt die Seite diesen Weg zuerst und den
Download als Rückfall. Ob der Download in eurem Tenant überhaupt durchgeht, lässt sich nur
vor Ort ausprobieren.

## Veraltete Fassungen

Dafür braucht die Seite nichts zu tun: `Auftragsablage.cmd` sieht bei **jedem** Start
selbst nach, ob unter `<Ablageordner>\_Anwendung\` eine neuere Fassung liegt, und
übernimmt sie, bevor die Anwendung öffnet – auch beim Start über diesen Link. Wer klickt,
hat danach den aktuellen Stand.

Nötig ist nur, die neue Fassung dort abzulegen; im Anwendungs-Repository erledigt das
`tools/Veroeffentliche-Einzeldatei.ps1`.

Eine Webseite könnte das ohnehin nicht prüfen: Sie sieht den Rechner nicht, und den
lokalen Webserver der Anwendung darf sie nicht befragen – der Port wechselt, es braucht
ein Sitzungsmerkmal, und Browser unterbinden Anfragen von öffentlichen Seiten an
`127.0.0.1`.

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
