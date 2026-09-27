# Subgraph Algorithmus

Der Subgraph Algorithmus – O(n³) Graphvergleich mittels injizierter Spaltensignaturen und zyklischer Rotationen.

## Überblick

Der **Subgraph Algorithmus** ist ein effizienter O(n³)-Algorithmus zum Vergleich zweier gerichteter Graphen G und G' anhand ihrer Adjazenzmatrizen. Der Algorithmus beantwortet die Frage:

> *Ist Graph G ein Teilgraph von Graph G'?*

Hierzu wird ein neuartiger Ansatz über **injizierte Spaltensignaturen** und **zyklische Rotationen** eingesetzt – ohne erschöpfende Permutationssuche (die O(n!) wäre).

**Kernaussage:** Enthält G' den Graphen G als Teilgraph, so wird G verworfen, da G' mehr strukturelle Information trägt.

## Algorithmus

### Schritt 1 – Spaltensignatur berechnen (O(n²))

Jeder Spaltenvektor der Adjazenzmatrix erhält eine eindeutige ganzzahlige Signatur über eine polynomiale Hash-Funktion:

```
sig(col) = Σ 2^i  für alle Zeilen i mit Wert 1
         + col_idx · 2^n
```

Der Gewichtungsterm `col_idx · 2^n` kodiert die Spaltenposition und garantiert Eindeutigkeit auch bei identischen Spaltenmustern.

**Beispiel (n = 4):**
```
Spalte 0: [1, 0, 1, 0]  →  2⁰ + 2² + 0·2⁴  =  5
Spalte 1: [1, 0, 1, 0]  →  2⁰ + 2² + 1·2⁴  = 21
```

### Schritt 2 – Zyklische Rotation (O(n))

Statt alle n! Permutationen zu prüfen, werden nur **n zyklische Rotationen** der Spalten von G' betrachtet:

```
Original:   [0, 1, 2, 3]
Rotation 1: [1, 2, 3, 0]
Rotation 2: [2, 3, 0, 1]
Rotation 3: [3, 0, 1, 2]
```

Die sequentielle Ordnung der Knoten bleibt so erhalten.

### Schritt 3 – LCS-basierter Signaturvergleich (O(n²))

Für jede Rotation wird geprüft, ob die Signatursequenz von G als **längste gemeinsame Teilsequenz (LCS)** in der Signatursequenz von G' enthalten ist (DP-Tabelle).

### Gesamtkomplexität

| Schritt | Komplexität |
|---------|-------------|
| Signaturberechnung G | O(n²) |
| Signaturberechnung G' | O(n²) |
| n Rotationen × O(n²) Vergleich | O(n³) |
| **Gesamt** | **O(n³)** |

## Rückgabewerte

| Ergebnis | Bedeutung |
|----------|-----------|
| `keep_B` | G ⊆ G' → G' enthält mehr Information, G wird verworfen |
| `keep_A` | G' ⊆ G → G enthält mehr Information, G' wird verworfen |
| `keep_both` | Kein Teilgraph-Verhältnis → beide werden behalten |
| `equal_keep_A/B` | Strukturell gleich → der kantenstärkere Graph wird behalten |

## Installation

```bash
git clone https://github.com/hjstephan86/subgraph.git
cd subgraph
pip install -e ".[dev]"
```

**Abhängigkeiten:**
- Python ≥ 3.8
- numpy ≥ 1.24.0
- pytest, pytest-cov (für Tests)

## Verwendung

```python
import numpy as np
from src.subgraph import Subgraph

algo = Subgraph()

# Graph G (4 Knoten)
A = np.array([
    [0, 1, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1],
    [0, 0, 0, 0]
])

# Graph G' mit zusätzlicher Kante
B = np.array([
    [0, 1, 1, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1],
    [0, 0, 0, 0]
])

decision, kept = algo.compare_graphs(A, B)
print(decision)   # → keep_B  (G ⊆ G')
print(kept)       # → Matrix B
```

### Komplexitätsanalyse

```python
info = algo.analyze_complexity(n=4)
for key, value in info.items():
    print(f"{key}: {value}")
```

## Tests

```bash
pytest
```

Der Testlauf erzeugt automatisch einen HTML-Coverage-Report unter `doc/coverage/index.html`.


## Wissenschaftlicher Hintergrund

Der Subgraph Algorithmus bildet das formale Fundament einer Reihe domänenübergreifender Arbeiten (Physik, Ingenieurwesen, Biologie, Informatik, Mathematik), die alle unter [`hjstephan86/science`](https://github.com/hjstephan86/science) katalogisiert sind.

Das vollständige Paper mit Beweisen, Pseudocode, Laufzeitanalyse und Anwendungsbeispielen ist unter [`science/subgraph.pdf`](science/subgraph.pdf) verfügbar.
