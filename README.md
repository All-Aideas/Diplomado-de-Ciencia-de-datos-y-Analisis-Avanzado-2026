# Proyecto Final Integrador - Priorización de contactos bancarios

> **Modelo de cátedra - Diplomado en Ciencia de Datos y Análisis Avanzado (UTN.BA)**

Un call center dispone de capacidad para contactar aproximadamente al **30 % de su cartera** por campaña. El proyecto construye un ranking de clientes según su propensión estimada a suscribir un plazo fijo, con el objetivo de concentrar las llamadas donde tienen mayor valor esperado.

## Resultado principal

Todos los números del informe salen de `proyecto_integrador.ipynb` y se guardan en `results/`.

| Métrica (test) | Sin priorización | Random Forest - período estable | Random Forest - período posterior |
|---|---:|---:|---:|
| Recall @ top 30 % | 30,4 % | **72,8 %** | 52,9 % |
| PR-AUC (piso = tasa de suscripción) | 0,117 | 0,485 (4,3× el piso) | 0,538 (1,7× el piso) |
| Tasa de suscripción del test | 11,3 % | 11,3 % | 30,8 % |

La meta fijada antes de entrenar era capturar al menos **70 % de las suscripciones con 30 % de las llamadas**. En distribución estable se cumple; en el período posterior no.

### Cómo se eligió el modelo

XGBoost y Random Forest quedan prácticamente empatados en validación cruzada: PR-AUC 0,467 vs. 0,466. La regla fijada **antes de mirar los tests** usa una validación temporal interna como desempate y deja a Random Forest con PR-AUC 0,113 frente a una tasa positiva de 11,1 %.

Esto debe interpretarse con cuidado: **Random Forest queda seleccionado como candidato provisional según la regla ex ante, no porque exista evidencia estadística fuerte de que sea superior**. En ese tramo temporal todos los modelos están cerca del piso, lo cual anticipa el principal riesgo del proyecto: inestabilidad temporal.

### Qué aprendimos del período posterior

En el test temporal la tasa de suscripción pasa de 6,4 % en entrenamiento a 30,8 % en test y el Recall @ 30 % de Random Forest cae a 52,9 %. La degradación se conserva y se reporta; no se cambia el test para obtener una métrica más alta.

SHAP y las ablaciones muestran que el modelo utiliza fuertemente variables macroeconómicas (`nr.employed`, `emp.var.rate`, `euribor3m`, entre otras). **Esto es consistente con una dependencia fuerte del contexto, pero no demuestra que el drift temporal sea causado únicamente por esas variables**: también pueden cambiar la campaña, la composición de clientes, el canal y la estacionalidad.

### Recomendación

No desplegar directamente. Reentrenar con datos actuales y realizar un piloto A/B con el mismo presupuesto de llamadas antes de una adopción operativa.

## Estructura del repositorio

```text
.
├── README.md
├── proyecto_integrador.ipynb      # notebook ejecutado de punta a punta
├── requirements.txt
├── data_log.md                    # registro de decisiones y límites
├── .gitignore
├── data/
│   └── README.md                  # fuente, licencia y descarga
├── results/                       # tablas y figuras usadas en el informe
│   └── README.md
└── docs/
    ├── Pre-entrega_Proyecto_Final_Integrador.pdf / .docx
    └── Informe_Final_Proyecto_Integrador.pdf / .docx
```

## Cómo ejecutar

### Google Colab
1. Abrir `proyecto_integrador.ipynb` en Colab.
2. Ejecutar **Entorno de ejecución → Ejecutar todas**.
3. El notebook instala `xgboost` y `shap` si faltan, descarga el dataset oficial de UCI y valida que sea exactamente `bank-additional-full.csv`.

### Local

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook proyecto_integrador.ipynb
```

Tiempo aproximado de ejecución completa: 5-6 minutos en una notebook de 4 núcleos. Las partes más costosas son la validación cruzada y las ablaciones.

## Flujo metodológico (CRISP-DM)

| Sección | Contenido | Salidas |
|---|---|---|
| 1. Negocio | KPI, benchmark y meta ex ante | - |
| 2. Datos | descarga validada, calidad, EDA de H1-H3, prevalencia temporal | `eda_*.csv`, `fig_eda.png` |
| 3. Preparación | exclusión de `duration`, duplicados, variables derivadas, pipelines | - |
| 4. Modelado | CV 5 folds, validación temporal interna y regla de selección ex ante | `cv_5folds.csv`, `validacion_temporal_interna.csv` |
| 5. Evaluación estable | ranking aleatorio + 3 modelos, curvas PR/ganancia, lift | `model_metrics.csv`, `lift_deciles.csv`, `fig_pr_ganancia.png` |
| 6. Evaluación temporal | entrenamiento con 80 % inicial y test en 20 % final | `temporal_metrics.csv` |
| 7. SHAP | importancia global agrupada y caso local real | `shap_global.csv`, `shap_local_case.csv`, `fig_shap_*.png` |
| 8. Profundización | segmentación, ablaciones y auditoría por grupos | `segmentacion.csv`, `ablaciones.csv`, `auditoria_grupos.csv` |
| 9. Negocio | escenario económico paramétrico | `escenario_economico.csv` |

## Decisiones metodológicas clave

- **`duration` se excluye**: se conoce después de la llamada; usarla sería fuga de información.
- **El problema es de ranking**: la métrica principal de negocio es Recall @ top 30 %. Accuracy no gobierna la decisión.
- **El clasificador de clase mayoritaria no es benchmark de ranking**: sirve para mostrar que una accuracy cercana a 89 % puede ser inútil. Como asigna el mismo score a todos, no se le reportan métricas `@30`.
- **Benchmark operativo**: ranking aleatorio, que captura aproximadamente 30 % de los positivos contactando 30 % de la base.
- **No se ponderaron clases en este experimento**: no porque la ponderación sea incorrecta, sino porque no resultaba necesaria para la comparación planteada y puede cambiar la escala de los scores. Si se quisiera interpretar el score como probabilidad absoluta, habría que evaluar calibración explícitamente.
- **`predict_proba` no garantiza calibración**: el proyecto usa el score principalmente para ordenar. Un valor 0,20 no se interpreta automáticamente como “20 % de probabilidad real” sin curva de calibración/Brier u otra evaluación apropiada.
- **SHAP**: para Random Forest, las contribuciones usadas aquí reconstruyen el output de `predict_proba`. El valor base es el **valor esperado del output del modelo**; en este caso es 11,26 %, casi igual a la prevalencia de desarrollo, pero no son conceptos idénticos por definición.
- **SHAP no prueba causalidad**: explica el comportamiento del modelo.
- **El test temporal se mantiene aunque dé peor**: esa degradación es uno de los resultados más importantes del proyecto.
- **Auditoría ética**: diferencias de prevalencia entre grupos no prueban ausencia de sesgo ni justifican automáticamente trato diferencial.

## Fuente de datos

**Fuente oficial:** https://archive.ics.uci.edu/dataset/222/bank+marketing  
**Descarga oficial:** https://archive.ics.uci.edu/static/public/222/bank%2Bmarketing.zip  
**DOI:** https://doi.org/10.24432/C5K306  
**Licencia:** CC BY 4.0  
**Archivo utilizado:** `bank-additional-full.csv` - 41.188 registros, 20 variables de entrada + `y`, orden cronológico mayo 2008-noviembre 2010.

Moro, S., Rita, P. y Cortez, P. (2014). *Bank Marketing* [Dataset]. UCI Machine Learning Repository.
