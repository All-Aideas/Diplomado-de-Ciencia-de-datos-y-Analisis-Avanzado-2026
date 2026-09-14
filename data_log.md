# Data log - decisiones del proyecto

| # | Etapa | Decisión | Motivo | Evidencia / consecuencia |
|---|---|---|---|---|
| 1 | Datos | Usar solo `bank-additional-full.csv` | Variante de 41.188 filas con indicadores macro y orden temporal | El notebook valida el esquema; no se mezcla con `bank-full.csv` |
| 2 | Datos | Recorrer zips anidados en la descarga | El zip oficial de UCI contiene `bank.zip` y `bank-additional.zip` | La descarga funciona en un clon limpio o en Colab |
| 3 | Preparación | Excluir `duration` | Solo se conoce después de la llamada | Evita fuga de información |
| 4 | Preparación | Eliminar 12 duplicados exactos | Reportados antes de eliminar; no alteran materialmente la prevalencia | 41.176 filas |
| 5 | Preparación | Conservar `unknown` como categoría | 0 nulos explícitos; `default` tiene 20,9 % de `unknown` | Evita imputación arbitraria |
| 6 | Preparación | `pdays = 999` → `contactado_antes` + `dias_desde_contacto` | 999 es un centinela, no una cantidad de días | Evita interpretar 999 como valor numérico ordinario |
| 7 | Preparación | No recortar `campaign` | Los casos altos son registros válidos; se evalúa su efecto con modelos robustos | 869 casos con más de 10 contactos |
| 8 | Preparación | `month` como categoría | El archivo mezcla meses de campañas/años distintos | H2 se interpreta con cautela |
| 9 | Modelado | No ponderar clases en este experimento | El objetivo principal es ranking y la ponderación puede cambiar la escala del score | No se asume que `predict_proba` esté perfectamente calibrado |
| 10 | Validación | Holdout estratificado 80/20 + CV 5 folds sobre el 80 % | El test estable no participa en la selección | `cv_5folds.csv` |
| 11 | Validación | Validación temporal interna (60 % → 60-80 %) para desempatar | Busca robustez temporal sin tocar el último 20 % | `validacion_temporal_interna.csv` |
| 12 | Selección | Regla ex ante de empate y desempate | Evita elegir por el mejor número del test | RF queda como **candidato provisional**; su PR-AUC temporal interna 0,113 está casi en el piso 0,111, por lo que el desempate es débil |
| 13 | Evaluación | Mantener el test temporal aunque dé peor | Responde si el ranking generaliza hacia un período posterior | Recall @ 30 % temporal 52,9 % |
| 14 | Métricas | Recall @ 30 % y PR-AUC como principales | Clase minoritaria + cupo operativo | Accuracy se reporta solo como complementaria |
| 15 | Baselines | Clase mayoritaria solo como baseline de clasificación; ranking aleatorio como benchmark operativo | Un score constante no ordena clientes | No se reporta Recall/Precision/Lift @30 para clase mayoritaria |
| 16 | Interpretación | SHAP sobre test y agrupación de one-hot por variable original | Facilita lectura de negocio manteniendo aditividad | El valor base es el output esperado del modelo, no “la prevalencia” por definición |
| 17 | Hipótesis | Ablaciones H1, H2 y bloque macro sobre el mismo test | Evita comparaciones inválidas entre subconjuntos con distinta prevalencia | H1 respaldada; macro produce la mayor caída |
| 18 | Drift | Interpretar SHAP + ablación macro como dependencia, no como causalidad | El drift también puede venir de campaña, clientes, canal o estacionalidad | Se evita atribuir toda la degradación al contexto macro |
| 19 | Ética | Auditar selección y recall por edad y estado civil | Atributos sensibles | Diferencias de prevalencia no prueban ausencia de sesgo |
| 20 | Negocio | Escenario económico con recall medido, estable y temporal | No usar solo la meta teórica | Temporal: beneficio neto del primer año negativo con una campaña |
| 21 | Calibración | No interpretar el score como probabilidad absoluta sin validación adicional | `predict_proba` no implica calibración perfecta | Si el caso de uso necesita probabilidad absoluta, evaluar Brier/curva de calibración y calibrar si corresponde |
