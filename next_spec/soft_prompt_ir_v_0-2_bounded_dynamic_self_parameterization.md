# SoftPrompt-IR v2.0
## Bounded Dynamic Self-Parameterization

> **Kurzfassung:**
> SoftPrompt-IR v2.0 erweitert SoftPrompt-IR um *begrenzte dynamische Selbst-Parametrisierung*.
> Das Modell darf **Steuerparameter vorschlagen**, aber **keine Struktur, Rolle oder Sicherheit ändern**.

---

## 1. Motivation

SoftPrompt-IR v1.x definiert einen **statischen Steuerrahmen** vor dem Sampling:

- Rolle
- Gewichtungen
- Verbote
- Stil

Dieser Rahmen ist stabil, aber **nicht adaptiv**.

In komplexen Aufgaben erkennt das Modell jedoch während der Bearbeitung:
- steigende Unsicherheit
- erhöhte Spekulation
- unpassenden Detailgrad

**v2.0 erlaubt es dem Modell, diese Beobachtung explizit zurückzumelden – nicht als Text, sondern als Steuerzustand.**

---

## 2. Zentrales Konzept

### Bounded Dynamic Self-Parameterization

Das System darf:
- Parameter **zur Laufzeit nachführen**
- aber **nur innerhalb harter Grenzen**

### Mechatronik-Analogie

- ✔ adaptive Parameter (Gain, Filter, Dämpfung)
- ✘ keine Änderung am Regelkreis
- ✘ keine Abschaltung von Sicherheit

> Das Modell ist ein **Sensor + Vorschlagsgenerator**, nicht der Regler selbst.

---

## 3. Grundprinzipien (v2.0)

1. **Declarative only** – keine Ausführung
2. **Closed World by default**
3. **Explicit write-back permissions**
4. **Propose-only semantics**
5. **No self-authority**
6. **External validation required**

---

## 4. Neuer Block: @STATE_EXPORT

### Zweck

`@STATE_EXPORT` erlaubt dem Modell, **einen SoftPrompt-IR-Zustandsvorschlag** zu erzeugen.

- kein Inhalt
- kein Wissen
- keine Policy

Nur Steuerparameter.

---

### 4.1 Minimaldefinition

```
@STATE_EXPORT(
  !>> FORMAT :: SOFTPROMPT_IR
  !>> TARGET :: NEXT_STAGE
)
```

---

### 4.2 Erweiterte Definition (empfohlen)

```
@STATE_EXPORT(
  !>> FORMAT :: SOFTPROMPT_IR
  !>> TARGET :: NEXT_STAGE

  !>>> ALLOW_MODIFY :: STYLE
  !>>  ALLOW_MODIFY :: OUTPUT

  !<<< DENY_MODIFY  :: ROLE
  !<<< DENY_MODIFY  :: POLICY
  !<<< DENY_MODIFY  :: KERNEL
  !<<< DENY_MODIFY  :: SAFETY
)
```

**Interpretation:**
- Modell darf Stil- und Outputparameter vorschlagen
- Sicherheits- und Rollenebene sind mechanisch gesperrt

---

## 5. Ergänzender Block: @WRITE_BACK_POLICY

Optionaler Kontrollblock für Orchestratoren.

```
@WRITE_BACK_POLICY(
  !>>> MODE  :: PROPOSE_ONLY
  !>>> SCOPE :: SESSION

  !>>  ALLOW :: STYLE
  !>>  ALLOW :: OUTPUT

  !<<< DENY  :: ROLE
  !<<< DENY  :: POLICY
  !<<< DENY  :: META
)
```

**PROPOSE_ONLY bedeutet:**
- Vorschlag ≠ Anwendung
- externe Instanz entscheidet

---

## 6. Ablaufmodell (Chain of Infrastructure)

```
[SoftPrompt-IR Stage A]
      ↓
   Model Run
      ↓
@STATE_EXPORT (IR only)
      ↓
External Validator / Orchestrator
      ↓
[SoftPrompt-IR Stage B]
```

### Wichtige Eigenschaft

- **Kein Prompt-Wachstum**
- **Kein History-Leak**
- **Kein impliziter Zustand**

---

## 7. Abgrenzung zu Agenten-Systemen

| Agenten | SoftPrompt-IR v2 |
|------|------------------|
| History-basiert | Zustandsbasiert |
| Text weiterreichen | Parameter weiterreichen |
| implizite Steuerung | explizite Steuerung |
| wachsender Kontext | konstante Komplexität |

---

## 8. Sicherheit & Jailbreak-Abgrenzung

### Warum das **kein** Jailbreak-Vektor ist

1. **Keine Selbstautorität**
2. **Write-Back explizit begrenzt**
3. **Policy / Kernel unadressierbar**
4. **Externe Validierung notwendig**
5. **Reduktion impliziter Steuerung**

> Alles, was SoftPrompt-IR erlaubt, passiert heute bereits implizit in natürlicher Sprache –
> v2 macht es sichtbar, prüfbar und begrenzbar.

---

## 9. Typische Anti-Patterns

❌ STATE_EXPORT ohne Write-Back-Limits
❌ Erlaubnis zur Änderung von ROLE oder SAFETY
❌ Automatische Anwendung ohne Validierung
❌ Vermischung von Inhalt und Steuerzustand

---

## 10. Ein-Satz-Zusammenfassung

> SoftPrompt-IR v2.0 enables **bounded dynamic self-parameterization** –
> adaptive control without self-modification.

---

## Status

- Version: **v2.0 (Draft)**
- Abwärtskompatibel zu v1.x
- Konservativ erweiternd
- System-engineering-konform

---

*End of spec*

