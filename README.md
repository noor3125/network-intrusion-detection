# ML Network Intrusion Detector

Scikit-learn models that classify network connections as **normal / attack** (binary) and by **attack family** (normal, DoS, Probe, R2L, U2R) on the public [NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html) dataset.

## Quick start

```bash
git clone https://github.com/<your-username>/network-intrusion-detector.git
cd network-intrusion-detector
pip install -r requirements.txt
```

**Notebook** (primary):
```bash
jupyter lab network_intrusion_detector.ipynb   # Kernel -> Restart & Run All (~90 s)
```

**CLI:**
```bash
cd src
python train.py                                  # downloads data, trains, writes ../models and ../reports
python predict.py ../data/KDDTest+.txt           # binary prediction + attack probability
python predict.py ../data/KDDTest+.txt --multiclass
```
`predict.py` expects a headerless CSV in NSL-KDD format (first 41 columns are features) and requires `train.py` to have been run first.

## Pipeline

- Preprocessing: `StandardScaler` on 38 numeric features, `OneHotEncoder` on `protocol_type`, `service`, `flag` (unknown categories ignored)
- Models: LogisticRegression, RandomForest, HistGradientBoosting; the best binary model by F1 is reused for multiclass
- Train: `KDDTrain+` (125,973 rows). Test: `KDDTest+` (22,544 rows, includes 17 attack types unseen in training)

## Results (KDDTest+, binary)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| LogisticRegression | 0.7547 | 0.9170 | 0.6257 | 0.7439 | 0.7912 |
| RandomForest | 0.7685 | 0.9678 | 0.6137 | 0.7511 | 0.9595 |
| **HistGradientBoosting** | **0.8082** | 0.9696 | 0.6845 | **0.8025** | 0.9602 |

5-fold CV F1 on `KDDTrain+` alone is 0.9989. The gap to `KDDTest+` reflects distribution shift (novel attacks), not leakage.

Multiclass (HistGradientBoosting): accuracy 0.7883. Per-class F1: DoS 0.895, normal 0.820, Probe 0.689, R2L 0.329, U2R 0.091.

| Binary confusion matrix | Multiclass confusion matrix | Top features |
|---|---|---|
| ![](reports/cm_binary.png) | ![](reports/cm_multiclass.png) | ![](reports/feature_importance.png) |

## Limitations

- R2L and U2R recall is low: both are rare in training and largely novel in the test set.
- NSL-KDD is built from 1990s-era traffic. Treat it as a benchmark, not a production signal.

## Next steps

- Cost-sensitive tuning or SMOTE for R2L/U2R
- IsolationForest hybrid for unseen attacks
- Evaluate on UNSW-NB15 and CIC-IDS2017

## Repository layout

```
network_intrusion_detector.ipynb   # end-to-end notebook with saved outputs
src/data.py                        # loading + label mapping
src/train.py                       # train, evaluate, save models and reports
src/predict.py                     # CLI inference
reports/                           # confusion matrices, feature importance, metrics.json
```

## Citation

Tavallaee, M., Bagheri, E., Lu, W., Ghorbani, A. "A Detailed Analysis of the KDD CUP 99 Data Set." *IEEE CISDA*, 2009.

## License

MIT
