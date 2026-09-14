# Resultados

Todos los archivos de esta carpeta se generan al ejecutar `proyecto_integrador.ipynb` y son los que usa el informe final.

| Archivo | Contenido |
|---|---|
| `eda_calidad.csv` | Nulos y categorías `unknown` por variable |
| `eda_h1_poutcome.csv`, `eda_h2_mes.csv`, `eda_h3_campaign.csv` | Evidencia descriptiva de las hipótesis |
| `eda_prevalencia_por_tramo.csv`, `fig_eda.png` | Prevalencia por tramo temporal y gráficos de EDA |
| `cv_5folds.csv` | Validación cruzada de 5 folds |
| `validacion_temporal_interna.csv` | Desempate temporal; evidencia débil y candidato provisional |
| `baseline_mayoritaria.csv` | Baseline de clasificación: PR-AUC, ROC-AUC y accuracy de predecir siempre “no”; no contiene métricas @30 |
| `model_metrics.csv` | Test estable: ranking aleatorio y 3 modelos |
| `lift_deciles.csv`, `fig_pr_ganancia.png` | Lift por deciles, curvas PR y de ganancia |
| `temporal_metrics.csv` | Test temporal (80 % inicial → 20 % final) |
| `shap_global.csv`, `fig_shap_global.png`, `shap_summary_dummies.png` | SHAP global |
| `shap_local_case.csv`, `fig_shap_local.png` | SHAP local de un cliente real |
| `segmentacion.csv` | Modelo global vs. modelos por segmento |
| `ablaciones.csv` | Ablaciones de H1, H2 y contexto macro |
| `auditoria_grupos.csv` | Auditoría por edad y estado civil |
| `escenario_economico.csv` | Escenario económico con recall medido |
| `resumen.csv` | Resumen de los números principales |
