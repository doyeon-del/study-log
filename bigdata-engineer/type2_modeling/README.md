# 작업형 2유형

분류/회귀 모델링 → 지정 형식 `result.csv` 제출. 실제 시험은 `train.csv`(target 포함)·`test.csv`(target 없음)가 주어지는데, 여기선 sklearn/seaborn 내장셋을 train/test로 쪼개 그 상황을 재현했다(교재 데이터는 올리지 않음).

| 노트북 | 묶음 | 평가지표 | 제출 |
| --- | --- | --- | --- |
| [`WORKFLOW.ipynb`](WORKFLOW.ipynb) | 표준 풀이 골격 (titanic, 완주 예시) | ROC-AUC | 확률 |
| `A_classification_binary.ipynb` | 이진 분류 | ROC-AUC · F1 | 확률 / 클래스 |
| `B_classification_multiclass.ipynb` | 다중 분류 | macro-F1 · weighted-F1 | 클래스 |
| `C_regression.ipynb` | 회귀 | RMSE · R² · RMSLE | 값 |

각 셀 주석(`# T2-xx`)이 문제 번호와 1:1 대응. 먼저 `WORKFLOW`로 흐름을 손에 익힌 뒤 A·B·C의 빈 셀을 직접 채운다.

## 표준 흐름

데이터 로드 → EDA → 결측치 → 인코딩 → (스케일링) → 검증 분할 → 학습/평가 → 예측 → `result.csv`

- **train에 한 전처리는 test에 똑같이 적용**한다(같은 통계값·같은 인코더).
- 제출 형식은 문제 지시대로: 컬럼명(보통 `pred`), 파일명(`result.csv`), `index=False`.

## 지표별 제출/주의

| 지표 | 제출 | 코드 |
| --- | --- | --- |
| ROC-AUC | 양성 **확률** | `model.predict_proba(test)[:, 1]` |
| F1 / 정확도 | **클래스** | `model.predict(test)` |
| macro / weighted F1 | 클래스 | `f1_score(y, p, average='macro'|'weighted')` |
| RMSE | 값 | `mean_squared_error(y, p) ** 0.5` |
| R² | 값 | `r2_score(y, p)` |
| RMSLE | 값(음수는 0으로) | `mean_squared_error(np.log1p(y), np.log1p(np.clip(p,0,None))) ** 0.5` |

## 사용 라이브러리

pandas, numpy, seaborn, scikit-learn (LightGBM은 있으면 선택적으로)
