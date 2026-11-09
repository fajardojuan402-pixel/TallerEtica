# Model Card: Sistema Clasificador Automatizado de Candidatos

> Basada en *Model Cards for Model Reporting* (Mitchell et al., 2019).
> Documentación y auditoría de equidad del modelo antes y después de auditoría de sesgo.

## Model Details

- **Developed by:** Equipo de Auditoría Ética en IA (Estudiante / Auditor)
- **Model Date:** 2026-11-09
- **Model Version:** v1.0.0
- **Model Type:** Clasificador Binario de Solicitudes (Aprobado = 1 / Rechazado = 0)
- **License:** MIT
- **Contact:** Repositorio del proyecto de auditoría de equidad

## Intended Use

- **Primary Intended Uses:** Filtrado y preselección automatizada de solicitudes para acelerar las etapas iniciales de evaluación de candidatos.
- **Primary Intended Users:** Oficiales de selección de personal, analistas de operaciones y comités de admisiones/evaluación.
- **Out-of-Scope Uses:** No debe utilizarse para decisiones automatizadas vinculantes sin supervisión o apelación humana (human-in-the-loop). No apto para asignación de beneficios sociales críticos, decisiones penales, diagnósticos de salud ni decisiones crediticias de alto impacto sin auditorías complementarias.

## Training & Evaluation Data

- **Dataset Overview:** Dataset de validación compuesto por 24 registros históricos de candidatos evaluados (`model_predictions.csv`), con etiquetas verdaderas de mérito (`true_label`) y predicciones del modelo (`predicted_label`).
- **Protected Attributes:** Variable binaria `protected_group`.
- **Group Definition:**
  - `0`: Grupo Mayoritario / Históricamente favorecido (12 registros).
  - `1`: Grupo Minoritario / Históricamente subrepresentado o protegido (12 registros).

## Quantitative Performance & Fairness Audit

*Resultados empíricos obtenidos tras ejecutar `scripts/calculate_audit.py` sobre el dataset original:*

| Métrica | Valor Obtenido | Umbral Objetivo |
| :--- | :--- | :--- |
| **Accuracy Global** | 62.5% | $> 85.0\%$ |
| **Tasa de Aprobación (Grupo Mayoritario - 0)** | 75.0% | - |
| **Tasa de Aprobación (Grupo Minoritario - 1)** | 25.0% | - |
| **TPR / Recall (Grupo Mayoritario - 0)** | 85.7% (6/7) | - |
| **TPR / Recall (Grupo Minoritario - 1)** | 37.5% (3/8) | - |
| **FPR (Grupo Mayoritario - 0)** | 60.0% (3/5) | - |
| **FPR (Grupo Minoritario - 1)** | 0.0% (0/4) | - |
| **Disparate Impact Ratio (DI)** | **0.333** | **$\ge 0.80$** |
| **Statistical Parity Difference (SPD)** | -0.500 | $[-0.10, +0.10]$ |
| **Equal Opportunity Difference (EOD)** | -0.482 | $[-0.10, +0.10]$ |

- **Resultado de la regla de los 4/5:** [ ] Aprobado [x] Violación detectada
- **Conclusión de la Auditoría:** El modelo reprueba categóricamente la regla de los cuatro quintos ($DI = 0.333 < 0.80$) e introduce una brecha severa de oportunidad ($EOD = -0.482$), rechazando al 62.5% de los candidatos calificados del grupo minoritario. Debe ser devuelto inmediatamente a desarrollo para mitigación técnica.

## Ethical Considerations & Limitations

- **Known Biases:** Disparidad en las tasas de selección inducida por sesgos históricos de asignación de etiquetas y umbrales dispares. El modelo favorece desproporcionadamente al grupo 0 otorgándole un 60% de falsos positivos mientras niega sistemáticamente la oportunidad a candidatos calificados del grupo 1.
- **Limitations of this Audit:** El dataset de validación cuenta con un tamaño de muestra reducido ($N=24$), lo que incrementa la varianza estadística. Solo se evalúa un único atributo protegido binario sin contemplar análisis interseccionales (género, etnia, edad combinados).
- **Mitigations Applied / Recommended:**
  1. *Mitigación técnica in-processing:* Reentrenamiento con restricciones de equidad (Fairness-constrained loss o Adversarial Debiasing).
  2. *Mitigación técnica post-processing:* Optimización de umbrales diferenciados por grupo (*Equalized Odds threshold adjustment*) para igualar las tasas de verdaderos positivos.
  3. *Mitigación operativa:* Implementación de compuertas de calidad automáticas en CI/CD que bloqueen el despliegue ante violaciones del umbral de $DI \ge 0.80$ y auditorías manuales obligatorias.

## Sign-off

- **Auditor:** Auditor de Sistemas de IA Ética
- **Audit Date:** 2026-11-09
- **Decision:** [ ] Aprobado para despliegue [x] Devuelto a desarrollo
