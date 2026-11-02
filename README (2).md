# Predicción de entregas tardías · Olist

**Data Analytics ML Challenge 2026-2** · Universidad Central · Ingeniería de Sistemas
Profesor: Elias Buitrago Bolivar
Autor: David Santiago Pedraza Ochoa (Santi) · **Estudiante ID 25** (semilla `1025`)

---

## ¿De qué trata?

Con el [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) quiero predecir si un pedido va a llegar tarde.

Cada fila es un pedido y la variable objetivo `is_late` vale **1** cuando la entrega real ocurre después de la fecha estimada, y **0** si llega a tiempo.

---

## Laboratorio 1 (Semana 12): modelos supervisados y métricas

**Qué hice**

- Tomé una muestra individual de **35.000 pedidos** y la dividí así:

  | Partición | Pedidos | Uso |
  |---|---|---|
  | Entrenamiento | 24.500 | Ajustar los modelos |
  | Validación | 5.250 | Comparar modelos y métricas |
  | Prueba | 5.250 | Reservada para la semana 16 |

- Usé **28 variables predictoras** (24 numéricas y 4 categóricas), cuidando que no haya fuga de información.
- Comparé un modelo de referencia (`DummyClassifier`) contra una **regresión logística** con `class_weight='balanced'`.
- Analicé errores por mes y por estado.
- Calculé el **costo empresarial** de los errores: R$ 8 por falso positivo y R$ 35 por falso negativo.
- Cerré con un ejemplo guiado de regresión (Ridge).

### Resultados en validación (regresión logística, `balanced`)

| Métrica | Valor |
|---|---|
| Accuracy | 0.657 |
| Precision | 0.119 |
| Recall | 0.632 |
| F1 | 0.200 |
| ROC-AUC | 0.699 |
| Average Precision | 0.147 |
| Costo de errores | R$ 17.945 |
| MAE regresión (Ridge) | 5.16 días |

**Cómo leerlo:** el modelo detecta cerca del 63 % de las entregas tardías (recall), pero con muchas falsas alarmas (precision baja), algo esperable porque las entregas tardías son la clase minoritaria. La interpretación de negocio y la propuesta de mejora para la semana 13 están en la **sección 6** del cuaderno.

---

## Estructura del repositorio

```
.
├── notebooks/
│   └── S12_ML_Challenge.ipynb        # cuaderno ejecutado, con actividades e interpretación
├── reports/
│   └── S12_resultados_ID_25.json     # reporte generado por el cuaderno
└── README.md
```

---

## Cómo reproducirlo

1. Abre `notebooks/S12_ML_Challenge.ipynb` en Google Colab o Jupyter.
2. Verifica que `ID_ESTUDIANTE = 25`.
3. Reinicia y ejecuta todas las celdas en orden.
4. Al terminar, el cuaderno escribe el reporte en `reports/S12_resultados_ID_25.json`.

Los CSV de Olist se leen directamente desde una copia pública con revisión fija (`fa0cc6d1aff713d3c8b9a1e633b56889881848bf`), así que no necesitas descargarlos.

**Dependencias:** Python 3, `pandas`, `numpy` y `scikit-learn`.

---

## Sobre los datos

Los datos originales **no** se publican en este repositorio. La misma muestra, las mismas variables y las mismas particiones se mantienen entre las semanas 12 a 16, para poder comparar los modelos de forma justa.
