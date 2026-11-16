# Predicción de entregas tardías · Olist

**Data Analytics ML Challenge 2026-2** · Universidad Central · Ingeniería de Sistemas
Profesor: Elias Buitrago Bolivar
Autor: David Santiago Pedraza Ochoa (Santi) · **Estudiante ID 25** (semilla `1025`)

---

## ¿De qué trata?

Con el [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) quiero predecir si un pedido va a llegar tarde.

Cada fila es un pedido y la variable objetivo `is_late` vale **1** cuando la entrega real ocurre después de la fecha estimada, y **0** si llega a tiempo. Solo 6,8 % de los pedidos llegan tarde, por eso la Accuracy sola engaña: un modelo que siempre dice «a tiempo» acierta el 93 % y no detecta ningún retraso.

Mi muestra (ID 25) tiene **35.000 pedidos**, con la huella `2bceba3982e4`, y se mantiene igual en todas las semanas:

| Partición | Pedidos | Uso |
|---|---|---|
| Entrenamiento | 24.500 | Ajustar los modelos |
| Validación | 5.250 | Comparar modelos y métricas |
| Prueba | 5.250 | Reservada para la semana 16; no la he usado para elegir nada |

---

## Laboratorio 1 (Semana 12): modelos supervisados y métricas

- Usé **28 variables predictoras** (24 numéricas y 4 categóricas), cuidando que no haya fuga de información.
- Comparé un modelo de referencia (`DummyClassifier`) contra una **regresión logística** con `class_weight='balanced'`.
- Analicé errores por mes y por estado y calculé el **costo empresarial** de los errores: R$ 8 por falso positivo y R$ 35 por falso negativo (supuesto didáctico).

| Métrica (validación) | Valor |
|---|---|
| Accuracy | 0.657 |
| Precision | 0.119 |
| Recall | 0.632 |
| F1 | 0.200 |
| ROC-AUC | 0.699 |
| Average Precision | 0.147 |
| Costo de errores | R$ 17.945 |

---

## Laboratorio 2 (Semana 14): PCA y evaluación de mejoras al modelo

**La pregunta:** ¿reducir variables con PCA o cambiar la regularización mejora el modelo de la semana 12? Para responderla mantuve la misma muestra, el mismo umbral (0,50) y `class_weight='balanced'`, y cambié una sola cosa a la vez.

**Qué hice**

- Apliqué **PCA** solo a las 24 variables numéricas (imputación por mediana y estandarización, ajustadas solo con entrenamiento). Con el umbral del 90 % de varianza quedaron **15 componentes** (90,60 %). Con 14 solo se llegaba a 87,44 %.
- Comparé cuatro regresiones logísticas: **A** sin PCA (C = 1), **B** con PCA (C = 1), **C** con PCA (C = 0,1) y **D** con PCA (C = 10).
- Calculé falsos positivos, falsos negativos, costo didáctico y el F1 en entrenamiento frente a validación para revisar si había sobreajuste.
- Apliqué la prueba **Kolmogorov-Smirnov** a `valor_flete` entre entrenamiento y validación.

### Resultados en validación

| Exp. | PCA | C | Accuracy | Precision | Recall | F1 | AP | FP | FN | Costo |
|---|---|---|---|---|---|---|---|---|---|---|
| A | No | 1 | 0.657 | 0.119 | 0.632 | 0.200 | 0.147 | 1.670 | 131 | R$ 17.945 |
| B | Sí | 1 | 0.645 | 0.110 | 0.598 | 0.186 | 0.128 | 1.720 | 143 | R$ 18.765 |
| C | Sí | 0,1 | 0.643 | 0.110 | 0.601 | 0.186 | 0.130 | 1.730 | 142 | R$ 18.810 |
| D | Sí | 10 | 0.644 | 0.110 | 0.601 | 0.186 | 0.128 | 1.727 | 142 | R$ 18.786 |

Referencias de costo: siempre «a tiempo» = R$ 12.460; alertar todos los pedidos = R$ 39.152.

### Lo que encontré en mi muestra

1. **PCA no ayudó.** Pasar de 24 a 15 variables numéricas bajó el Recall de 0.632 a 0.598 (12 retrasos más sin detectar) y subió el costo R$ 820. Como también cayeron AP y ROC-AUC, no fue solo un efecto del umbral.
2. **La regularización casi no importa.** Entre C = 0,1 y C = 10 la diferencia es de un pedido tardío. El F1 de entrenamiento y validación es casi igual (brecha menor a 0.015), así que no hay sobreajuste que corregir.
3. **KS sobre `valor_flete`:** D = 0.0147 y p = 0.307, por lo que no se rechaza H0 al 5 %. No demuestra que las distribuciones sean idénticas, y hay muchos valores repetidos (solo 4.593 distintos en 24.500 pedidos de entrenamiento), así que el valor p se lee con cautela.
4. **Elección:** entre las cuatro, **A** (mayor Accuracy, mayor Recall y menor costo). Pero con el umbral 0,50 y estos costos ningún modelo supera en costo a «nunca alertar»: la precisión de A (0.119) queda por debajo del punto de equilibrio 8 / 43 ≈ 0.186. Como sistema automático de alertas no se justifica; el siguiente paso sería otro experimento (umbral o variables nuevas) evaluado en validación.

Los costos son supuestos didácticos, no costos reales de Olist, y todo se midió en una sola partición de validación. La interpretación completa está en el cuaderno y en `reports/S14_resumen_ID_25.md`.

---

## Estructura del repositorio

```
notebooks/
  S12_ML_Challenge.ipynb              # semana 12, ejecutado con actividades e interpretación
  S14_PCA_Evaluacion_ID_25.ipynb      # semana 14: PCA, comparación A-D, costos, sobreajuste y KS
reports/
  S12_resultados_ID_25.json           # reporte de la semana 12
  S14_registro_experimentos_ID_25.csv # bitácora de los experimentos A-D
  S14_resultados_ID_25.json           # reporte completo de la semana 14
  S14_resumen_ID_25.md                # resumen del análisis
  S14_varianza_acumulada_ID_25.png    # curva de varianza explicada acumulada
  S14_costos_ID_25.png                # costo por experimento
  S14_ks_valor_flete_ID_25.png        # ECDF de valor_flete, entrenamiento y validación
README.md
```

## Cómo reproducirlo

1. Abre el cuaderno que quieras (`notebooks/S12_ML_Challenge.ipynb` o `notebooks/S14_PCA_Evaluacion_ID_25.ipynb`) en Google Colab o Jupyter.
2. Verifica que `ID_ESTUDIANTE = 25`.
3. Reinicia y ejecuta todas las celdas en orden. El cuaderno de la semana 14 comprueba que la huella de la muestra sea `2bceba3982e4`, igual que en la semana 12.
4. Cada cuaderno escribe sus reportes en `reports/`.

Los CSV de Olist se leen directamente desde una copia pública con revisión fija (`fa0cc6d1aff713d3c8b9a1e633b56889881848bf`), así que no necesitas descargarlos.

**Dependencias:** Python 3, `pandas`, `numpy`, `scikit-learn`, `scipy` y `matplotlib`.

---

## Sobre los datos

Los datos originales **no** se publican en este repositorio. La misma muestra, las mismas variables y las mismas particiones se mantienen entre las semanas 12 a 16, para poder comparar los modelos de forma justa.
