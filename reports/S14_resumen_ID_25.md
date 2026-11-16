# Resumen S14 · PCA y evaluación de mejoras · ID 25

Muestra: 35,000 pedidos (huella `2bceba3982e4`), validación de 5,250 pedidos
(356 tardíos). Umbral 0.50, `class_weight='balanced'`. La prueba final no se usó.

## PCA
24 variables numéricas -> 15 componentes
(90.60 % de varianza acumulada). El clasificador recibe
140 columnas con PCA frente a 149 sin PCA.

## Comparación en validación
| Exp. | PCA | C | Accuracy | Precision | Recall | F1 | AP | FP | FN | Costo (8·FP+35·FN) |
|---|---|---|---|---|---|---|---|---|---|---|
| A | No | 1.0 | 0.657 | 0.119 | 0.632 | 0.200 | 0.147 | 1,670 | 131 | R$ 17,945 |
| B | Sí | 1.0 | 0.645 | 0.110 | 0.598 | 0.186 | 0.128 | 1,720 | 143 | R$ 18,765 |
| C | Sí | 0.1 | 0.643 | 0.110 | 0.601 | 0.186 | 0.130 | 1,730 | 142 | R$ 18,810 |
| D | Sí | 10.0 | 0.644 | 0.110 | 0.601 | 0.186 | 0.128 | 1,727 | 142 | R$ 18,786 |

Referencia "siempre a tiempo": Accuracy 0.932, Recall 0, costo R$ 12,460.
Referencia "alertar todos": Recall 1, costo R$ 39,152.

## Sobreajuste (F1 entrenamiento frente a validación)
A: 0.211 y 0.200 | B: 0.200 y 0.186 | C: 0.200 y 0.186 | D: 0.200 y 0.186.
Brecha máxima: 0.014; no hay señal de sobreajuste, por eso cambiar C casi no tiene efecto.

## Conclusión
- Modelo elegido entre A-D: **A** (menor costo y mejor desempeño en todas las métricas).
- PCA (B frente a A) empeoró Recall, F1, AP y costo; cambiar C (C y D frente a B) casi no modificó nada.
- Ninguna configuración supera en costo a la referencia trivial con los costos supuestos
  (la precisión de A es 0.119 y el punto de equilibrio es 0.186); no se recomienda operar con umbral 0.50.
- KS sobre `valor_flete` (entrenamiento frente a validación): D = 0.0147, p = 0.307 -> No rechazar H0 al 5 %.
  No demuestra que las distribuciones sean idénticas; hay muchos empates en los importes.
- Limitaciones: costos supuestos, una sola partición de validación, sin pruebas de comparación entre modelos.
