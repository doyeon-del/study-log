# 실기 치트시트

검색 불가 환경 대비. 막히면 `help(객체)`, `dir(객체)`, `print(객체.__doc__)`.

## 작업 1유형 — 전처리 / 기술통계

```python
import pandas as pd
import numpy as np

df = pd.read_csv('data.csv')

df.head(); df.info(); df.describe(); df.shape
df.isnull().sum()                 # 결측치 개수
df['col'].value_counts()          # 범주 빈도
df['col'].nunique()               # 고유값 수

# 결측 처리
df['col'].fillna(df['col'].median(), inplace=True)
df = df.dropna(subset=['col'])

# 정렬 / 상위 N
df.sort_values('col', ascending=False).head(10)

# 조건 필터 + 집계
df[df['col'] > df['col'].mean()]['target'].sum()
df.groupby('cat')['val'].agg(['mean', 'sum', 'count'])

# 이상치 (IQR)
q1, q3 = df['col'].quantile(0.25), df['col'].quantile(0.75)
iqr = q3 - q1
out = df[(df['col'] < q1 - 1.5*iqr) | (df['col'] > q3 + 1.5*iqr)]

# 표준화 / 정규화 직접
df['z'] = (df['col'] - df['col'].mean()) / df['col'].std()
```

## 작업 2유형 — 모델링 (분류/회귀)

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.metrics import roc_auc_score, f1_score, mean_squared_error

# 1) 분리
X = train.drop(columns=['target', 'ID'])
y = train['target']
test_id = test['ID']
X_test = test.drop(columns=['ID'])

# 2) 인코딩 (범주형)
for c in X.select_dtypes('object').columns:
    le = LabelEncoder()
    X[c] = le.fit_transform(X[c])
    X_test[c] = le.transform(X_test[c])   # 동일 인코더 재사용 주의

# 3) 검증셋 분리
X_tr, X_val, y_tr, y_val = train_test_split(X, y, test_size=0.2, random_state=42)

# 4) 학습 + 평가
model = RandomForestClassifier(random_state=42)
model.fit(X_tr, y_tr)
pred_val = model.predict_proba(X_val)[:, 1]      # 확률 (AUC용)
print(roc_auc_score(y_val, pred_val))

# 5) 제출 (★ 형식·컬럼명 문제 지시 그대로!)
pred = model.predict_proba(X_test)[:, 1]
submit = pd.DataFrame({'ID': test_id, 'target': pred})
submit.to_csv('result.csv', index=False)         # index=False 필수
```

> 체크: 제출 컬럼명/파일명은 **문제 지시문 그대로**. 분류=`predict_proba`(확률) vs `predict`(라벨) 헷갈리지 말 것. 회귀는 `mean_squared_error(y, p) ** 0.5`로 RMSE.

## 작업 3유형 — 통계 검정 / 회귀분석

```python
from scipy import stats
import statsmodels.api as sm
from statsmodels.formula.api import ols, logit

# t-검정 (단일/독립/대응)
stats.ttest_1samp(a, popmean=0)
stats.ttest_ind(a, b, equal_var=True)
stats.ttest_rel(before, after)

# 정규성 / 등분산
stats.shapiro(a)
stats.levene(a, b)

# 카이제곱 (독립성)
table = pd.crosstab(df['A'], df['B'])
chi2, p, dof, exp = stats.chi2_contingency(table)

# 상관
stats.pearsonr(df['x'], df['y'])

# 분산분석 (ANOVA)
model = ols('y ~ C(group)', data=df).fit()
sm.stats.anova_lm(model, typ=2)

# 다중 회귀 — 계수/p값/R²
model = ols('y ~ x1 + x2', data=df).fit()
print(model.summary())
model.params; model.pvalues; model.rsquared

# 로지스틱 회귀 — 오즈비
m = logit('y ~ x1 + x2', data=df).fit()
np.exp(m.params)
```

> 결과 추출: `round(값, 소수자리)`로 문제 요구 자리수 맞추기. p값 해석은 유의수준(보통 0.05) 기준.
