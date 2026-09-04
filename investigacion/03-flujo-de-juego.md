# 03 — Flujo de juego

## Arranque

1. `main()` inicializa Flutter binding.
2. `HiveService.init()` abre boxes y crea perfil default si hace falta.
3. `ProviderScope` overridea `hiveServiceProvider`.
4. `WaterSortApp` → tema dark fijo → `HomeView`.

## Menú principal (`HomeView` + `HomeViewModel`)

Carga:

- progreso del perfil activo
- lista de perfiles
- flags (timer, super hard, blur, instant pour, hints, sonido)
- estrellas y tema activo

Desde home se navega a:

- partida (continuar / nivel actual)
- selección de niveles
- modo aleatorio (diálogo de dificultad)
- how to play
- settings (temas, perfiles, toggles, código unlock)

## Generación de niveles (`LevelGenerator`)

> Detalle completo de la curva: **[05-escalado-dificultad.md](./05-escalado-dificultad.md)**.

### Campaña (`generate(levelNumber)`)

- Seed determinista: `Random(levelNumber)` → mismo nivel siempre.
- Colores: `3 + (level-1)//3`, clamp 3–16 (**+1 color cada 3 niveles**).
- Tubos vacíos: 1–3 según cantidad de colores.
- Capacidad: 4 normal; **5** cada 5 niveles; **6** cada 10.
- Fragmentación: niveles 1–3 bloques de 2; 4–6 mixto; **≥7 todo unitario** (más difícil).
- Construcción: trocea el agua en segmentos, baraja, reparte, y **valida solvabilidad** con DFS (límite 5000 estados). Si no es solvable, reintenta (`while (true)`).

### Modo random (`generateRandom`)

- Seed libre (timestamp o seed guardada).
- Dificultad mapea a `colorCount` + `capacity` en `GameViewModel.loadRandomLevel` (Easy 3/4 … Super Duper Hard 16/6).
- Con `levelNumber: -1` el scramble es siempre de singles (sin fase tutorial).

### “Optimal moves”

**No** se calcula con el solver real. Fórmula aproximada:

```text
baseMoves = colorCount * 4 + random(-2..2)
```

Se usa para estrellas al completar (ver abajo).

## Partida (`GameViewModel` + `WaterSortGame`)

### Interacción

1. Tap tubo origen (no vacío) → selección + haptic.
2. Tap destino válido → marca `pouringFrom/To`.
3. Flame anima el vertido (salvo instant pour) y llama `completePendingPour`.
4. Se actualizan tubos, historial (undo), se persiste estado parcial.
5. Si `isComplete` → cancela timer, guarda estrellas + progreso.

### Reglas de vertido

- Solo se mueve el bloque continuo del color del tope.
- Destino vacío o mismo color en el tope.
- Cantidad limitada por espacio libre.
- (En generator/solver) se evita mover un tubo monocolor a un vacío vacío “inútil” en la búsqueda; en gameplay el view model no aplica esa heurística de forma explícita al validar (solo `canReceive`).

### Timer

- Activo si setting `timer_enabled`.
- Campaña: desde nivel ≥ 4.
- Random: si dificultad ≠ Easy.
- Duración: `round((30 + colorCount * 15) * 1.5)` segundos.

### Hints

- `LevelSolver.solve` (DFS con heurística de score, hasta 150k estados).
- Cachea solución; si el jugador se desvía, invalida cache y re-resuelve.
- Marca `hintFromIndex` / `hintToIndex` para highlight en Flame.

### Estrellas

Al completar:

| Condición | Estrellas |
|-----------|-----------|
| moves < optimal | 3 |
| moves == optimal | 2 |
| moves > optimal | 1 |

Como `optimalMoves` es estimado, las 3 estrellas son alcanzables “por debajo” del óptimo ficticio, no necesariamente del óptimo real.

### Persistencia mid-game

Tras cada move se serializa a Hive:

- tubos (colores como `Color.value` int)
- historial de snapshots
- moveCount, timeLeft, flags de modo random

Al `loadLevel` / `loadRandomLevel` se restaura si coincide nivel/dificultad/seed.

## Rendering (Flame)

`WaterSortGame` (~1100 LOC) concentra:

- layout de tubos
- animación de pour
- partículas / ripples
- audio al completar tubo / verter
- modos visuales (super hard, blur, hints)

La UI Flutter (`game_view.dart`) envuelve el `GameWidget` con HUD (movimientos, timer, botones undo/hint/reset, diálogos win/timeout).
