# Solución nivel 30 (desde estado inicial)

Generado con `LevelGenerator` + `LevelSolver` (2026-09-04).

**Parámetros:** 12 colores, 14 tubos (2 vacíos), capacity **6**.

**Importante:** reinicia el nivel antes de seguir esta secuencia. Si ya moviste agua, no aplica.

Tubos vacíos iniciales: **12** y **13**.

Índices de color (paleta `AppColors.waterColors`): C0 rojo, C1 azul, C3 amarillo, C4 naranja, C5 morado, C7 cyan, C8 lila, C9 coral, C10 índigo, C12 marrón, C13 rojo oscuro, C15 oliva.

## Movimientos (tubo origen → destino), 67 pasos

1. 0→12  2. 0→13  3. 3→12  4. 6→0  5. 7→0
6. 4→7  7. 8→12  8. 13→8  9. 0→13  10. 1→0
11. 9→1  12. 9→3  13. 11→9  14. 11→6  15. 0→11
16. 12→0  17. 6→12  18. 4→6  19. 4→12  20. 9→4
21. 1→9  22. 5→9  23. 1→5  24. 7→1  25. 7→13
26. 11→7  27. 11→0  28. 6→11  29. 6→4  30. 2→6
31. 10→2  32. 10→1  33. 5→10  34. 5→11  35. 10→5
36. 6→10  37. 9→6  38. 5→9  39. 1→5  40. 2→1
41. 2→12  42. 2→6  43. 13→2  44. 0→13  45. 3→1
46. 4→0  47. 7→4  48. 3→7  49. 3→6  50. 2→3
51. 2→10  52. 1→2  53. 1→13  54. 4→1  55. 0→4
56. 7→0  57. 7→10  58. 8→0  59. 8→7  60. 11→7
61. 11→8  62. 10→11  63. 8→11  64. 8→0  65. 1→8
66. 10→2  67. 10→12

## Tips de estrategia (sin lista completa)

1. Usa los dos vacíos (12/13) solo para liberar tops que bloquean; no vuelques tubos mono-color a vacío.
2. Prioriza completar tubos (llenar un color al tope) antes que esparcir.
3. Si te trabas: Settings → Hint Helper ON y pide hint (resalta origen/destino del siguiente buen movimiento desde tu estado actual).
