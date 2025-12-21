# SoftPrompt-IR: Von impliziter Absicht zu expliziter Struktur

Dieser Abschnitt vergleicht die ursprüngliche Anweisung (Natural Language) mit der strukturierten Darstellung (SoftPrompt-IR) und erläutert die Vorteile dieser Methode.

## 1. Vergleich der Anweisungsstile

### ❌ Vorher (Natürliche Sprache, Hoher Rauschpegel)

Die Anweisungen im ursprünglichen Stil waren wie folgt formuliert:

*   Bitte vermeiden Sie blumige Sprache.
*   Versuchen Sie, keine Klischees zu verwenden.
*   Übererklären Sie Dinge nicht.
*   Seien Sie klar und prägnant.
*   Konzentrieren Sie sich auf den Hauptpunkt.

### Probleme des vorherigen Stils

Dieser Stil führte zu folgenden Problemen:

*   **Repetition:** Wiederholungen in der Formulierung.
*   **Ambiguous Priority:** Die Priorität der Anweisungen war nicht klar definiert.
*   **Conflicting Signals:** Mögliche widersprüchliche Signale für das Modell.
*   **Weight is implicit:** Die Gewichtung der einzelnen Anweisungen war implizit.

### ✅ Nachher (SoftPrompt-IR)

Die gleiche Absicht wird durch eine strukturierte Darstellung ausgedrückt:

```
!~> AVOID_FLOWERY_STYLE
~>  AVOID_CLICHES
~>  LIMIT_EXPLANATION
```

## 2. Analyse der Änderungen

### Was hat sich geändert?

Die Transformation bewirkt folgende Verbesserungen:

*   **Same intent:** Die ursprüngliche Absicht bleibt erhalten.
*   **Explicit weighting:** Die Gewichtung wird explizit durch die Struktur (z.B. das `!~>` Präfix) dargestellt.
*   **No prose to interpret:** Es gibt keinen interpretierbaren Fließtext mehr.
*   **Lower entropy before decoding:** Die Mehrdeutigkeit wird reduziert, bevor die eigentliche Generierung beginnt.

### Warum ist dies nicht „nur Formatierung“?

Der Unterschied liegt in der Art der vermittelten Information:

*   **BOLD TEXT** >> **emphasis is implicit** (Betonung ist implizit)
*   **STRUCTURED TAG** >> **intent is explicit** (Absicht ist explizit)

## 3. Funktionsweise von SoftPrompt-IR

SoftPrompt-IR stützt sich **nicht** auf:

*   Spezielle Syntax
*   Versteckte Semantik
*   Neues Modelltraining (Model Retraining)

Es basiert stattdessen auf:

1.  **Consistency** (Konsistenz)
2.  **Structure** (Struktur)
3.  **Patterns already learned** (Bereits gelernte Muster, z.B. in Konfigurationen, Regeln oder Flags)

---

## One-Sentence Summary

**SoftPrompt-IR reduziert die Ambiguität vor dem Sampling, indem es implizite Absicht in explizite Struktur umwandelt.**