# 05 — Escalado de dificultad

Fuente principal: `lib/domain/use_cases/level_generator.dart` + timer en `GameViewModel`.

La campaña **no** usa un “difficulty enum”. Cada nivel `N` deriva parámetros con fórmulas deterministas (`Random(N)` como seed).

---

## Resumen: qué sube con el nivel

| Palanca | Cómo escala | Efecto en dificultad |
|---------|-------------|----------------------|
| **Nº de colores** | +1 color cada 3 niveles (máx. 16) | Más piezas que ordenar; espacio de estados crece |
| **Tubos vacíos** | 1 → 2 → 3 según colores | Más buffer ayuda un poco, pero no compensa el crecimiento |
| **Capacidad del tubo** | 4 normal; **5** cada 5 niveles; **6** cada 10 | Tubos más altos = más capas mezcladas |
| **Fragmentación del scramble** | Pares (1–3) → mixto (4–6) → unidades (7+) | Más fragmentos = más movimientos y planning |
| **Timer** | Desde nivel **≥ 4** (si está activado) | Presión de tiempo; duración crece con colores |
| **Meta de estrellas** | `~ colorCount * 4 ± 2` (ficticia) | Más colores → “par” más alto |

---

## 1. Cantidad de colores

```dart
// _baseColors = 3, _maxColors = 16
colorCount = (3 + (level - 1) ~/ 3).clamp(3, 16)
```

| Niveles | Colores |
|---------|---------|
| 1–3 | 3 |
| 4–6 | 4 |
| 7–9 | 5 |
| 10–12 | 6 |
| 13–15 | 7 |
| 16–18 | 8 |
| 19–21 | 9 |
| 22–24 | 10 |
| 25–27 | 11 |
| 28–30 | 12 |
| 31–33 | 13 |
| 34–36 | 14 |
| 37–39 | 15 |
| **≥ 40** | **16** (techo) |

Tras el nivel 40 la dificultad por colores **se estanca**; siguen variando capacidad (múltiplos de 5/10), seed y scramble.

---

## 2. Tubos vacíos y total de tubos

```dart
vacantes:
  colorCount ≤ 4  → 1
  colorCount ≤ 8  → 2
  colorCount ≤ 12 → 2
  colorCount > 12 → 3

tubos = colorCount + vacantes
```

Ejemplos:

| Nivel | Colores | Vacíos | Tubos totales |
|-------|---------|--------|---------------|
| 1 | 3 | 1 | 4 |
| 5 | 4 | 1 | 5 |
| 10 | 6 | 2 | 8 |
| 28 | 12 | 2 | 14 |
| 40 | 16 | 3 | 19 |

Más vacíos dan más maniobra, pero el tablero se satura de tubos (UI + complejidad visual).

---

## 3. Capacidad (altura del tubo)

```dart
si level % 10 == 0 → 6
si no, si level % 5 == 0 → 5
si no → 4
```

| Capacidad | Niveles (ejemplos) |
|-----------|--------------------|
| 4 | 1,2,3,4,6,7,8,9,11,… |
| 5 | 5, 15, 25, 35, … (múltiplos de 5 no de 10) |
| 6 | 10, 20, 30, 40, … |

En esos “hitos” cada color aporta más unidades de agua (`capacity` capas), así que el puzzle es más denso.

---

## 4. Fragmentación al generar el puzzle (muy importante)

Antes de barajar, cada color se parte en **chunks**:

| Condición | Chunks por color | Sensación |
|-----------|------------------|-----------|
| Nivel **1–3** y capacity 4 | Dos bloques de **2** | Tutorial / fácil: bloques ya agrupados |
| Nivel **4–6** y capacity 4 | 50% bloques de 2, 50% **4 singles** | Transición |
| Nivel **≥ 7** (o capacity ≠ 4) | `capacity` singles (1+1+…) | Máxima mezcla: cada gota separada |

Luego se `shuffle` de chunks, se aplastan a una secuencia y se rellenan solo los primeros `colorCount` tubos (los vacíos quedan al final).

**Conclusión:** el salto duro no es solo “más colores”, sino pasar de bloques dobles a **todo unitario** a partir del nivel 7.

---

## 5. Timer (presión temporal)

Si el setting de timer está ON:

- Campaña: timer desde nivel **≥ 4**
- Duración: `round((30 + colorCount * 15) * 1.5)` segundos

| Colores | Segundos aprox. |
|---------|-----------------|
| 3 | 112 |
| 4 | 135 |
| 6 | 180 |
| 9 | 247 |
| 12 | 315 |
| 16 | 405 |

El timer **crece** con los colores (más tiempo absoluto), pero el puzzle también crece; no es un “menos segundos por nivel” lineal.

---

## 6. Seed y reproducibilidad

`Random(levelNumber)` → el nivel N es **siempre el mismo puzzle** (colores elegidos, scramble, etc.), salvo que el bucle `while (!solvable)` reintente con más shuffles del mismo RNG (avanza el estado del `Random`). En la práctica el resultado queda fijo para un N dado una vez generado de forma estable… ojo: cada `shuffle`/`nextBool` consume RNG, así que el número de reintentos por fallos de DFS **sí puede** cambiar el puzzle entre versiones del algoritmo, pero no entre jugadores con el mismo código.

---

## 7. Modo aleatorio (no es campaña)

Mapeo fijo en `loadRandomLevel` (si no pasas color/capacity a mano):

| Dificultad UI | Colores | Capacidad | Timer |
|---------------|---------|-----------|-------|
| Easy | 3 | 4 | no |
| Medium | 6 | 4 | sí |
| Hard | 9 | 5 | sí |
| Super Hard | 12 | 5 | sí |
| Super Duper Hard | 16 | 6 | sí |

Equivalencia aproximada con campaña:

- Easy ≈ niveles 1–3
- Medium ≈ entorno nivel 10–12
- Hard ≈ nivel ~22 + tubos altos
- Super Hard ≈ nivel ~28
- Super Duper Hard ≈ techo (nivel ≥ 40) con capacity 6

Además, en random el scramble usa `levelNumber: -1` → **siempre** fragmentación máxima (singles), sin la fase tutorial de pares.

---

## 8. Curva mental (campaña)

```text
Nv 1–3   : 3 colores, bloques de 2, sin timer → onboarding
Nv 4–6   : +1 color, scramble mixto, TIMER ON
Nv 7–9   : 5 colores, scramble 100% unitario → primer “golpe”
Nv 10    : capacity 6 (pico de densidad)
…        : +1 color / 3 niveles
Nv 40+   : 16 colores, solo varían seed / capacity en hitos 5/10
```

---

## 9. Lo que NO escala

- Complejidad del solver de hints (mismo algoritmo; solo más estados).
- Velocidad de animación / reglas de pour.
- `optimalMoves` real (sigue siendo fórmula, no BFS óptimo).
- Después de 16 colores: no hay más “capas” de dificultad tipadas, solo variación procedural.
