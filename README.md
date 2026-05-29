# Lidl Tronic Solarstromspeicher – tuya-local Integration

> **Gerät:** Lidl TRONIC Solarstromspeicher 2,2 kWh  
> **Baugleich:** Marstek Saturn B2500  
> **Tuya Produkt-ID:** `fpi7g58s9hoxlvng` | **Kategorie:** `bxsdy`  
> **Protokoll:** Tuya 3.3 (lokal)  
> **Integration:** [make-all/tuya-local](https://github.com/make-all/tuya-local)  
> **Status:** ✅ Funktioniert – alle Basiswerte bestätigt

---

## Inhalt

- [Voraussetzungen](#voraussetzungen)
- [Installation](#installation)
- [Entitäten](#entitäten)
- [DP-Referenz](#dp-referenz)
- [base64-DP-Strukturen](#base64-dp-strukturen)
- [Bekannte Einschränkungen](#bekannte-einschränkungen)
- [Template-Sensoren](#template-sensoren)
- [Automationen](#automationen)
- [Credits](#credits)

---

## Voraussetzungen

- Home Assistant mit [tuya-local](https://github.com/make-all/tuya-local) (via HACS)
- Lokale IP-Adresse des Speichers (am besten per DHCP-Reservierung fix vergeben)
- **Local Key** des Geräts (aus Tuya IoT Platform oder per tinytuya-Scan)
- Protokollversion: **3.3**

---

## Installation

1. Datei `lidl_tronic_speicher_tuyalocal.yaml` kopieren nach:
   ```
   /config/custom_components/tuya_local/devices/
   ```

2. Home Assistant neu starten.

3. In HA: **Einstellungen → Geräte & Dienste → tuya-local → Gerät hinzufügen**  
   → „Manuell" wählen → IP-Adresse + Local Key + Protokoll **3.3** eingeben.

4. Das Gerät wird als **„Lidl Tronic Solarstromspeicher"** erkannt und sollte eine hohe Übereinstimmungsqualität (>90%) zeigen.

> **Hinweis:** Die base64-DPs (3, 33, 101, 106, 114) erscheinen nicht im initialen  
> Status-Response des Geräts – sie werden nur asynchron gepusht. Sie sind daher  
> als `optional: true` markiert und beeinflussen die Erkennungsqualität nicht.

---

## Entitäten

Nach der Einrichtung erstellt tuya-local folgende Entitäten:

### Sensoren (Hauptwerte)
| Entität | DP | Beschreibung |
|---|---|---|
| `sensor.batteriestand` | 1 | Ladestand in % |
| `sensor.lade_entladestatus` | 109 | `charge` / `discharge` / `standby` |
| `sensor.ladeleistung` | 3 | Lade-/Entladeleistung in W |
| `sensor.ladestrom` | 3 | Lade-/Entladestrom in A (÷10) |
| `sensor.akkuspannung` | 3 | Akkuspannung in V (÷10) |
| `sensor.temperatur` | 10 | Akkutemperatur in °C |
| `sensor.restlaufzeit` | 2 | Verbleibende Zeit in Minuten |

### PV-Eingang (DP 101, nur tagsüber)
| Entität | Bytes | Beschreibung |
|---|---|---|
| `sensor.pv1_leistung` | 5–6 | PV-Eingang 1 Leistung W |
| `sensor.pv2_leistung` | 11–12 | PV-Eingang 2 Leistung W |
| `sensor.pv_ertrag_gesamt` | DP 37 | PV-Ertrag gesamt kWh (÷100) |

### DC-Ausgänge (DP 33, asynchron)
| Entität | Bytes | Beschreibung |
|---|---|---|
| `sensor.dc_out1_leistung` | 5–6 | Ausgang OUT1 Leistung W |
| `sensor.dc_out2_leistung` | 11–12 | Ausgang OUT2 Leistung W |

### Energie-Zähler (Lifetime)
| Entität | DP | Beschreibung |
|---|---|---|
| `sensor.batterie_geladen_gesamt` | 102 | Batterie geladen gesamt kWh (÷100) |
| `sensor.batterie_entladen_gesamt` | 103 | Batterie entladen gesamt kWh (÷100) |
| `sensor.verbrauch_gesamt` | 104 | Gesamtverbrauch kWh (÷100) |

### Steuerung
| Entität | DP | Beschreibung |
|---|---|---|
| `select.betriebsmodus` | 105 | Laden+Entladen / Laden zuerst |
| `select.temperatureinheit` | 24 | Celsius / Fahrenheit |
| `number.wr_ausgangsleistung` | 115 | WR-Maximalleistung in W (0–800, entspricht „Inverter Configuration → Power") |
| `number.entladeleistung_slot_1` | 106 | Entladeleistung Slot 1 in W (0 oder 80–800, Schritt 10) |
| `number.entladetiefe` | 107 | Entladetiefe (DoD) in % (1–100) |
| `button.daten_aktualisieren` | 111 | Erzwingt sofortigen DP-Push vom Gerät |
| `button.entladeleistung_reset` | 106 | Schreibt Standard-Blob auf DP 106 (Initialisierung nach Neustart) |

### Diagnose
| Entität | DP | Beschreibung |
|---|---|---|
| `sensor.fehlercode` | 4 | Fehlercode (0 = kein Fehler) |
| `sensor.anzahl_akkupacks` | 113 | Anzahl angeschlossener Akkupacks |
| `sensor.dc_out1_spannung` | 33 | Ausgang OUT1 Spannung V (÷10) |
| `sensor.dc_out1_strom` | 33 | Ausgang OUT1 Strom A (÷100) |
| `sensor.dc_out2_spannung` | 33 | Ausgang OUT2 Spannung V (÷10) |
| `sensor.dc_out2_strom` | 33 | Ausgang OUT2 Strom A (÷10) |
| `sensor.pv1_spannung` | 101 | PV-Eingang 1 Spannung V (÷10) |
| `sensor.pv1_strom` | 101 | PV-Eingang 1 Strom A (÷10) |
| `sensor.pv2_spannung` | 101 | PV-Eingang 2 Spannung V (÷10) |
| `sensor.pv2_strom` | 101 | PV-Eingang 2 Strom A (÷100) |
| `sensor.dc_message_raw` | 33 | DC-Nachricht als base64 |
| `sensor.abschaltschwelle` | 108 | Intern berechneter Standby-Schwellwert in W (readonly) |
| `sensor.wr_max_leistung` | 114 | Maximale WR-Leistung in W (aus Inverter-Konfiguration) |
| `sensor.wr_konfiguration_raw` | 114 | WR-Konfiguration als base64 |

---

## DP-Referenz

### Integer / String DPs (im Status-Response enthalten)

| DP | Name (Tuya intern) | Typ | Einheit | Skala | Bestätigt |
|---|---|---|---|---|---|
| 1 | `battery_percentage` | integer | % | ×1 | ✅ |
| 2 | `remain_time` | integer | min | ×1 | ✅ |
| 4 | `fault` | integer | – | ×1 | ✅ |
| 10 | `temp_current` | integer | °C | ×1 | ✅ |
| 24 | `temp_set_enum` | string | – | – | ⓘ Spec |
| 37 | `reverse_energy_total` | integer | kWh | ÷100 | ✅ |
| 102 | `batt_char_total` | integer | kWh | ÷100 | ✅ |
| 103 | `batt_dischar_total` | integer | kWh | ÷100 | ✅ |
| 104 | `electric_total` | integer | kWh | ÷100 | ✅ |
| 105 | `charge_mode` | string | – | – | ✅ |
| 107 | `discharge_limit` | integer | % | ×1 | ✅ |
| 108 | `batt_on_threshold` | integer | W | ×1 | ⚠️ unbestätigt |
| 109 | `charge_flag` | string | – | – | ✅ |
| 111 | `force_reflesh` | boolean | – | – | ✅ |
| 113 | `pack_number` | integer | – | ×1 | ✅ |
| 115 | `invt_power` | integer | W | ×1 | ✅ |

**DP 24 Werte:** `c` (Celsius) / `f` (Fahrenheit)  
**DP 105 Werte:** `charge_discharge` / `charge_first`  
**DP 109 Werte:** `charge` / `discharge` / `standby`

### base64 DPs (nur asynchron gepusht, nicht im initialen Status-Response)

| DP | Name (Tuya intern) | Bytes | Bestätigt |
|---|---|---|---|
| 3 | `battery_parameters` | 6 | ✅ |
| 33 | `dc_message` | 13 | ✅ |
| 101 | `pv_dc_data` | 13 | ✅ (nur tagsüber) |
| 106 | `discharge_mode` | 36 | ✅ |
| 114 | `invt_id` | 8 | ✅ |

---

## base64-DP-Strukturen

### DP 3 – `battery_parameters` (6 Bytes = 3× uint16 big-endian)

```
Byte 0–1:  Akkuspannung            uint16 BE  ÷10 → V
Byte 2–3:  Lade-/Entladestrom      uint16 BE  ÷10 → A
Byte 4–5:  Lade-/Entladeleistung   uint16 BE  ×1  → W
```

Beispiel: `AdwALwDj` → `01DC002F00E3`
- `01DC` = 476 → 47,6 V
- `002F` = 47 → 4,7 A
- `00E3` = 227 → 227 W

### DP 33 – `dc_message` (13 Bytes)

```
Byte 0:     Flag (0=normal, 3=Entladefenster aktiv)
Byte 1–2:   OUT1 Spannung  uint16 BE  ÷10  → V
Byte 3–4:   OUT1 Strom     uint16 BE  ÷100 → A
Byte 5–6:   OUT1 Leistung  uint16 BE  ×1   → W
Byte 7–8:   OUT2 Spannung  uint16 BE  ÷10  → V
Byte 9–10:  OUT2 Strom     uint16 BE  ÷10  → A
Byte 11–12: OUT2 Leistung  uint16 BE  ×1   → W
```

### DP 101 – `pv_dc_data` (13 Bytes, nur bei aktiver PV vorhanden)

```
Byte 0:     Flag/Version
Byte 1–2:   PV1 Spannung  uint16 BE  ÷10  → V
Byte 3–4:   PV1 Strom     uint16 BE  ÷10  → A
Byte 5–6:   PV1 Leistung  uint16 BE  ×1   → W
Byte 7–8:   PV2 Spannung  uint16 BE  ÷10  → V
Byte 9–10:  PV2 Strom     uint16 BE  ÷100 → A
Byte 11–12: PV2 Leistung  uint16 BE  ×1   → W
```

### DP 106 – `discharge_mode` (36 Bytes = 1 Byte Header + 5× 7 Byte Slots)

```
Byte 0: Header (immer 0x00)

Pro Slot (Offset = 1 + Slot-Index × 7):
  Byte +0:   Aktiv (0x01 = ja, 0x00 = nein)
  Byte +1:   Startstunde  (0–23)
  Byte +2:   Startminute  (0–59)
  Byte +3:   Endstunde    (0–23)
  Byte +4:   Endminute    (0–59)
  Byte +5–6: Leistung     uint16 BE → W
```

Beispiel: Slot 1 aktiv, 20:30–23:59, 320 W  
→ `00 01 14 1E 17 3B 01 40 00 00 00 00 00 00 00 ...`

> **Hinweis:** Die YAML-Datei schreibt **nur die Leistung von Slot 1** (Bytes 6–7).  
> tuya-local liest den aktuellen Blob, ändert die Ziel-Bytes und schreibt ihn zurück –  
> alle anderen Zeitfenster bleiben erhalten.  
> Gültige Werte: **0W** (Einspeisung stopp) oder **80–700W** – Werte zwischen 1–79W werden vom Gerät ignoriert.

### DP 114 – `invt_id` (8 Bytes = 4× uint16 big-endian)

```
Byte 0–1: WR-Modellcode  (Hoymiles: 1000er-Reihe, Deye: 2000er-Reihe)
Byte 2–3: 360 (konstant)
Byte 4–5: 30  (Mindestleistung, konstant)
Byte 6–7: Max. WR-Leistung in W
```

Bekannte Modellcodes:

| Code | Modell | Max. Leistung |
|---|---|---|
| 1004 | Hoymiles HM-600 | 600 W |
| 1005 | Hoymiles HM-700 | 700 W |
| 1006 | Hoymiles HM-800 | 800 W |
| 1012 | Hoymiles HM-1500 | 1500 W |
| 1019 | Hoymiles HM-2250 | 2250 W |
| 1028 | Hoymiles HMT-2250-6T | 2250 W |
| 2001 | Deye SUN600G3 | 600 W |

> **Tipp:** Bytes 6–7 werden als `sensor.wr_max_leistung` extrahiert und können in HA-Template-Entities als dynamisches Maximum für `number.wr_ausgangsleistung` und `number.entladeleistung_slot_1` genutzt werden.

---

## Bekannte Einschränkungen

- **DP 106 (Entladeplan) – ⚠️ Flash-Verschleiß:** Die Werte in DP 106 werden vermutlich im Flash-Speicher des Geräts abgelegt. Flash hat eine begrenzte Anzahl an Schreibzyklen – häufige automatische Schreibvorgänge (z. B. durch eine Nulleinspeisung-Automation im Sekundentakt) können den Speicher langfristig schädigen. **Empfehlung von [@Hofyyy](https://github.com/Hofyyy):** Statt den Speicher aktiv zu regeln, die Entladeleistung auf einen festen Maximalwert setzen und die Regelung dem Wechselrichter überlassen – der nimmt sich nur so viel, wie er benötigt: `Speicher → 600 W (Dauerleistung) → Wechselrichter (gesteuert)`. Nur Slot-1-Leistung wird über die YAML gesteuert. Für vollständige Zeitfenster-Konfiguration (Slot 1–5) ist ein Node-RED/AppDaemon-Script notwendig. Minimum der Entity: 80 W (Werte 1–79 W werden vom Gerät ignoriert; 0 W stoppt die Einspeisung, ist über die Entity aber bewusst nicht einstellbar – dafür Betriebsmodus „Laden zuerst" verwenden).
- **DP 115 (WR Ausgangsleistung):** Entspricht „Inverter Configuration → Power" in der App. Verhält sich als reine **Obergrenze**: Senken cappt DP 106 Slot 1 mit; Senken auf ≤200W cappt zusätzlich die Abschaltschwelle (DP 108). Erhöhen hat keinen Effekt auf Slot 1 oder DP 108. **Nicht geeignet für Nulleinspeisung** – dafür DP 106 Slot 1 verwenden. Gerät akzeptiert maximal das WR-Modell-Maximum (aus DP 114).
- **DP 101 (PV-Daten):** Taucht nur bei aktiver PV-Produktion auf. Nachts bleiben diese Sensoren leer.
- **DP 33 (DC-Ausgang):** Werte kommen nur als asynchrone Pushes, nicht auf Anfrage.
- **DP 108 (Abschaltschwelle):** Tuya-intern als `batt_on_threshold` dokumentiert. Vermutlich intern berechneter Schwellwert, ab dem das Gerät aus dem Standby wechselt (z. B. bei vorhandener PV-Leistung). Als readonly Sensor behandelt – theoretisch schreibbar, Effekt nicht bestätigt.
- **Temperatur-Skala:** DP 10 liefert den Rohwert direkt in °C (kein Teiler). Tuya-Cloud-Spec bestätigt `scale:0`.

---

## Template-Sensoren

Die Datei [`ha_templates.yaml`](ha_templates.yaml) enthält zwei Template-Sensoren für das **HA Energie-Dashboard**.

**Installation:**
```yaml
# configuration.yaml
template: !include ha_templates.yaml
```
Anschließend HA neu starten. Den Gerätenamen ggf. unter `variables: device:` anpassen.

---

### Ladeleistung netto (vorzeichenbehaftet)

tuya-local liefert `sensor.ladeleistung` immer als positiven Absolutwert. Dieser Sensor kombiniert ihn mit `sensor.lade_entladestatus` zum korrekten Vorzeichen für das Energie-Dashboard:

| Status | Wert |
|---|---|
| `discharge` | positiv (z. B. +320 W) |
| `charge` | negativ (z. B. −180 W) |
| `standby` | 0 W |

**Energie-Dashboard:** Einstellungen → Energie → Heimspeicher → `sensor.ladeleistung_netto`

---

### Solarleistung netto

Summiert PV1 + PV2 aus DP 101. Tagsüber aktiv, nachts `unavailable` (DP 101 wird vom Gerät nur bei aktiver PV-Produktion gepusht).

**Energie-Dashboard:** Einstellungen → Energie → Solaranlage → `sensor.solarleistung_netto`

> Inspiriert von [Hofyyy/lidl-tronic-solarspeicher](https://github.com/Hofyyy/lidl-tronic-solarspeicher)

---

## Automationen

Die Datei [`ha_automations.yaml`](ha_automations.yaml) enthält eine dokumentierte Beispiel-Automation.

### Tronic – Refresh & Monitoring

Löst zwei Probleme gleichzeitig:

**1. Sensor-Refresh (minütlich)**  
Drückt `button.daten_aktualisieren` (DP 111 force_refresh) – das Gerät pushed daraufhin sofort alle aktuellen DPs. Ohne diesen Trigger bleiben base64-DPs wie DP 3, 33 und 101 nach einer Pause veraltet.

**2. DP 106 Initialisierung nach Neustart / WLAN-Reconnect**  
DP 106 (Entladeleistung Slot 1) wird vom Gerät **nie** von sich aus gepusht. Nach jedem Verbindungsaufbau steht `sensor.entladeleistung_slot_1` auf `unknown` – tuya-locals Masking schlägt dann fehl:
```
Cannot mask unknown current value
```
Die Automation erkennt diesen Zustand und drückt nach 5 Sekunden `button.entladeleistung_reset`, der den Standard-Blob (alle 5 Slots, 80 W) direkt auf DP 106 schreibt – ohne Masking.

**Import:** Automation in HA öffnen → Drei-Punkte-Menü → „Als YAML bearbeiten" → Inhalt aus `ha_automations.yaml` einfügen (ohne den Kopfkommentar, `id:` weglassen).

---

## Verwandte Projekte

- [Hofyyy/lidl-tronic-solarspeicher](https://github.com/Hofyyy/lidl-tronic-solarspeicher) – Reverse-Engineering, tinytuya-Diagnostics, DP-Dokumentation
- [make-all/tuya-local Issue #5164](https://github.com/make-all/tuya-local/issues/5164) – Offizielle Aufnahme ins tuya-local-Projekt (offen)
- [make-all/tuya-local](https://github.com/make-all/tuya-local) – Die verwendete HA-Integration

---

## Credits

- **Hofyyy** – Reverse-Engineering der base64-DP-Strukturen, tinytuya-Diagnostics, erste DP-Dokumentation
- Dieses Repo ergänzt die Arbeit von Hofyyy um eine funktionierende tuya-local YAML-Konfiguration mit:
  - Korrektem Schema-Format (`entities` statt `primary_entity`/`secondary_entities`)
  - Korrekte uint16-Masken für alle base64-DPs
  - `optional: true` für alle base64-DPs (nötig für korrekte Geräte-Erkennung)
  - Bestätigte Einheiten und Skalierungen (kWh ÷100, Temperatur ×1)
  - Vollständige PV-Sensoren (DP 101: Leistung, Spannung, Strom für PV1+PV2)
  - DP 24 Temperatureinheit-Selector
  - DP 115 als schreibbarer Number-Entity ohne künstliche Begrenzung
  - Button-Entity für DP 111 (force_refresh)
  - Button-Entity für DP 106 (Initialisierungs-Reset nach Neustart/Reconnect)
  - Template-Sensoren für das HA Energie-Dashboard (`ha_templates.yaml`)
  - Beispiel-Automation für Sensor-Refresh und DP 106 Initialisierung (`ha_automations.yaml`)

---

*Getestet mit: tuya-local 2025.x, Home Assistant 2026.x, Protokoll Tuya 3.3*
