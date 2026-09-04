# Diseño — Tips limitados por nivel

Estado: **aprobado e implementado** (2026-09-04)

## Objetivo

Tips contextuales del tablero (solver / highlight origen→destino), no tips de tutorial del juego.

## Reglas

| Caso | Comportamiento |
|------|----------------|
| Campaña 1–9 | Sin botón de tip |
| Campaña ≥ 10 | 3 tips por nivel |
| Modo random | 3 tips por partida |
| Reset nivel | Contador vuelve a 3 (si el modo aplica tips) |
| Tip OK | Consume 1; resalta siguiente movimiento del solver |
| Tip fallido (sin solución) | No consume |
| Undo | No recupera tips |
| Setting “Hint Helper” | Ignorado; la feature se rige por nivel/contador |

## UI

- Botón tip solo si `hintsAllowed` (campaña ≥10 o random).
- Badge con restantes (3→0); en 0 deshabilitado.

## Archivos

- `game_view_model.dart` — estado + lógica
- `game_view.dart` — botón + contador

## Enfoque

Extender `showHint()` existente (enfoque 1).
