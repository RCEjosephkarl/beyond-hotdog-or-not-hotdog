# beyond-hotdog-or-not-hotdog

**Learning Team 9** : Cacao, Chavez, Libre, Zumel

---

## Presentation Deck Summary

### Overview

*Beyond Hotdog or Not Hotdog* is a deep learning reliability study that asks: **can food photos reliably estimate calories and macronutrients, and when should we trust that estimate?** Inspired by the "Not Hotdog" app from HBO's *Silicon Valley*, the project goes beyond simple food classification to test the practical limits of image-based nutrition prediction.

### Problem Statement

Food photos capture appearance, not the full nutrition story. Portion mass, oils, sauces, hidden ingredients, and cooking method are invisible to a camera. Similar-looking dishes can have wildly different calorie counts. The core challenge is not just predicting nutrition values, but knowing **when a prediction can be trusted**.

### Dataset — Nutrition5k

- 4,768 fully labeled dishes; **3,092 dishes** used for fair comparison (those with both overhead and side-angle photos)
- Labels per dish: calories (kcal), mass (g), fat (g), carbohydrates (g), protein (g)
- Split 70 / 15 / 15 by dish ID to prevent leakage → 2,164 train / 463 validation / 465 test dishes
- Image imbalance: 265,869 side-angle images vs. 3,242 overhead images; one of each view selected per dish
- Dataset skew: calories and mass have large mean–median gaps; leans toward U.S. cafeteria-style meals

### Prior Work

| Study | Contribution | Relevance |
|---|---|---|
| Myers et al. (2015), *Im2Calories* | Estimated nutrition from meal photos | Shows the task is feasible |
| Thames et al. (2021), *Nutrition5k* | Built the dataset (avg PMAE ~29.1%) | Provides dataset and baseline |
| Shao et al. (2022), *Swin-Nutrition* | Swin Transformer model (avg PMAE ~17.2%) | Stronger accuracy benchmark |

The team's contribution is not state-of-the-art accuracy, but a systematic **reliability test** under real-world degraded photo conditions.

### Evaluation Metrics

- **PMAE** (Percent Mean Absolute Error) — primary metric; comparable across targets with different units
- **MAE / RMSE** — average error and large-error penalty
- **R²** — how well the model captures variation across dishes
- Clean and **degraded-photo robustness** tests

### Model Families Tested (19 variants)

| Family | Best variant | Clean PMAE |
|---|---|---|
| Simple baselines | Median regression | 67.96 |
| Pure tabular (sklearn) | Extra Trees | — |
| CNN from scratch | Overhead single-view | 36.79 |
| **CNN feature extractor** | **Extra Trees on 320 CNN features** | **31.81** |
| Ensemble | Stacking Ridge | 37.51 |
| Multimodal (scratch CNN + tabular) | Overhead + side + 34 features | 39.79 |
| Pretrained EfficientNetV2B0 | Overhead only | 34.41 |

**Best clean-photo model:** `method_4_cnn_feature_extra_trees` — extracts 128 features from an overhead CNN, 128 from a side-angle CNN, and 64 from a multi-view CNN (320 features total), then feeds them into Extra Trees regression. R² = 0.71; 53% PMAE improvement over the median baseline.

### Robustness Test

The same held-out dishes were re-evaluated after synthetic degradation:

| Degradation | Description |
|---|---|
| Dark | Brightness factor 0.45 (dim/indoor lighting) |
| Blur | Average-pool blur, kernel 5 (handheld motion blur) |
| Compressed | JPEG quality 35 (messaging-app compression) |
| Downscaled | Scale 0.35 → resized back (low-res uploads) |

**Key finding — best accuracy ≠ best reliability:**

| Model | Clean PMAE | Mean degraded PMAE | % worse |
|---|---|---|---|
| CNN Feature + Extra Trees | 31.81 | 49.23 | +54.8% |
| Pretrained Overhead CNN | 34.41 | 37.67 | +9.5% |

The pretrained overhead CNN is the **safer deployment choice**: slightly less accurate on clean photos but far more stable under poor conditions.

### Reliability Framework

Estimates are presented with a three-tier reliability card:

| Error band | Label | User message |
|---|---|---|
| 0–10% | Good | Solid first-pass estimate |
| 10–35% | Typical | Useful, but confirm details |
| > 35% | Higher-error | Show warning and ask for user input |

### Conclusions

- Food photos carry real nutrition signal.
- The best clean-photo model is a CNN feature extractor + Extra Trees (PMAE 31.81).
- The best deployment model is the pretrained overhead CNN — balanced accuracy and robustness.
- Estimates should ship with reliability flags, not as final labels.
- *"Accuracy wins the test; reliability wins deployment."*

### Future Directions

**Product:** automatic photo-quality check to route images to the right model; reliability flags for end users; user confirmation for hidden details (oils, sauces, portions); wellness/coaching use only — not medical.

**Technical:** log-transform targets, filter extreme labels, add uncertainty quantification, test on real phone-captured photos and more diverse (non-U.S.) food images.

### References

- Myers et al. (2015). *Im2Calories*. ICCV.
- Thames et al. (2021). *Nutrition5k*. CVPR, pp. 8903–8911.
- Shao et al. (2022). *Swin-Nutrition*. *Foods*, 11(21), 3429.