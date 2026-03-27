# README DEV

## Ziel

Dieses Dokument beschreibt den aktuellen Entwicklungsstand und die lokalen Schritte, um das Projekt reproduzierbar zu starten.

## Projektkontext

MVP-Flow:

1. Barcode scannen/eingeben.
2. Produktdaten von Open Food Facts abrufen.
3. Produkt in Odoo anlegen oder aktualisieren.
4. Optional Bestand auf Lagerort buchen.

## Lokales Setup (Windows)

### 1. In Odoo-Verzeichnis wechseln

```powershell
cd C:\playground\ERP\odoo
```

### 2. Virtual Environment erstellen (falls noch nicht vorhanden)

```powershell
py -3.12 -m venv .venv
```

### 3. Python-Abhaengigkeiten installieren

```powershell
.\.venv\Scripts\python.exe -m pip install --upgrade pip setuptools wheel
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Zusatzpaket (wegen Modulabhaengigkeit `account_peppol`):

```powershell
.\.venv\Scripts\python.exe -m pip install phonenumbers
```

## Odoo-Konfiguration

Datei: `odoo/odoo.conf`

Wichtige Werte:

```ini
[options]
admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = openpg
db_password = openpgpwd
http_interface = 127.0.0.1
http_port = 8069
addons_path = C:\playground\ERP\odoo\addons
```

## Datenbank

Entwicklungsdatenbank:

- Name: `erp_dev`
- DB User: `openpg`
- DB Passwort: `openpgpwd`

Initialisierung (nur beim ersten Mal):

```powershell
cd C:\playground\ERP\odoo
.\.venv\Scripts\python.exe odoo-bin -c odoo.conf -d erp_dev -i base --stop-after-init
```

## Odoo starten

### Empfohlen auf Port 8070 (um Konflikte mit installiertem Odoo-Dienst zu vermeiden)

```powershell
cd C:\playground\ERP\odoo
.\.venv\Scripts\python.exe odoo-bin -c odoo.conf -d erp_dev --http-port=8070
```

Browser:

- http://127.0.0.1:8070

## Custom Addon: barcode_open_food_facts

Pfad:

- `odoo/addons/barcode_open_food_facts`

Inhalt:

1. Wizard zum Barcode-Import (`barcode.off.import.wizard`).
2. Open-Food-Facts Abruf mit API-Fallback (`v2` -> `v0`).
3. User-Agent Header fuer OFF-Anfragen.
4. Barcode-Variantenlogik (12/13-stellig mit/ohne fuehrende 0).
5. Produktanlage/Update in Odoo.
6. Optionale Bestandsbuchung via `stock.quant._update_available_quantity`.

Modul aktualisieren:

```powershell
cd C:\playground\ERP\odoo
.\.venv\Scripts\python.exe odoo-bin -c odoo.conf -d erp_dev --http-port=8070 -u barcode_open_food_facts
```

## Bekannte Stolperfallen

1. Pfadfehler bei venv:
   - Richtig: `.\.venv\Scripts\python.exe`
   - Falsch: `..venv\Scripts\python.exe`

2. Parallel laufender Odoo-Dienst auf Port 8069:
   - Kann alte Program-Files Instanz ausliefern.
   - Fuer Entwicklung Port 8070 verwenden.

3. PostgreSQL-Warnung in Odoo 19:
   - Aktuell lokal PostgreSQL 12.x.
   - Odoo 19 meldet Mindestversion 13.
   - Lokal laeuft es, fuer produktive Nutzung Upgrade auf PG 13+ einplanen.

4. OFF-Barcode nicht vorhanden:
   - Dann kommt "No product found...".
   - Das ist ein fachlicher Fall, kein Systemfehler.

## Schnelltest

1. Odoo auf 8070 starten.
2. Modul `barcode_open_food_facts` aktualisieren.
3. App-Menue -> "Barcode OFF" -> "Import Product".
4. Test-Barcodes probieren (falls in OFF vorhanden):
   - `3017620422003`
   - `737628064502`

## Nächste Entwicklungsschritte

1. Fallback implementieren: Produkt minimal anlegen, wenn OFF nichts findet.
2. Mehr Felder mappen (Marke, Gewicht, Bild, Herkunft).
3. Logging und Benutzer-Meldungen verbessern.
4. Tests fuer Wizard-Flow ergaenzen.
