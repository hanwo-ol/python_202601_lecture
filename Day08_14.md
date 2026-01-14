판다스 라이브러리
---

# Pandas 실무 기초: 데이터프레임

## 1. 데이터프레임(DataFrame)의 구조 이해
데이터프레임은 행(Row)과 열(Column)로 구성된 2차원 자료구조입니다. 각 열은 서로 다른 데이터 타입을 가질 수 있으며, 이는 NumPy의 ndarray와 차별화되는 특징입니다.

```python
import pandas as pd
import numpy as np

# 샘플 데이터 생성
data = {
    '이름': ['김철수', '이영희', '박민수', '최지우', '정다은'],
    '국어': [80, 90, 75, 100, 95],
    '영어': [85, 95, 80, 90, 85],
    '수학': [90, 85, 70, 95, 100],
    '합격여부': [True, True, False, True, True]
}

df = pd.DataFrame(data)

# 데이터프레임 정보 확인 메소드
print(df.head(3))      # 상위 3개 행 출력
print(df.info())       # 데이터프레임 요약 정보(인덱스, 컬럼명, 데이터 타입 등)
print(df.describe())   # 수치형 데이터의 기술 통계량(평균, 표준편차 등)
print(df.shape)        # 행과 열의 개수 반환
```

## 2. 데이터 선택 및 추출 (Indexing & Slicing)
데이터프레임에서 특정 행이나 열을 선택하는 방식은 크게 레이블 기반의 `loc`와 정수 위치 기반의 `iloc`로 나뉩니다.

### 2.1 열(Column) 선택
```python
# 단일 열 선택 (Series 반환)
name_column = df['이름']

# 여러 열 선택 (DataFrame 반환)
score_columns = df[['국어', '영어', '수학']]
```

### 2.2 loc (레이블 기반 선택)
`loc[행_레이블, 열_레이블]` 형식을 사용합니다.
```python
# 0번 인덱스 행의 '이름' 값
print(df.loc[0, '이름'])

# 0~2번 인덱스 행의 '이름'과 '국어' 열
print(df.loc[0:2, ['이름', '국어']])
```

### 2.3 iloc (정수 위치 기반 선택)
`iloc[행_인덱스, 열_인덱스]` 형식을 사용합니다.
```python
# 0번째 행, 0번째 열 값
print(df.iloc[0, 0])

# 0~2번째 행, 0~2번째 열 (슬라이싱 시 마지막 인덱스 제외)
print(df.iloc[0:3, 0:3])
```

## 3. 조건부 필터링 (Boolean Indexing)
특정 조건을 만족하는 데이터만 추출하는 방법입니다.

```python
# 국어 점수가 90점 이상인 데이터
high_korean = df[df['국어'] >= 90]

# 국어 점수가 80점 이상이고 수학 점수가 90점 이상인 데이터 (AND 조건)
filtered_df = df[(df['국어'] >= 80) & (df['수학'] >= 90)]

# 영어 점수가 95점이거나 수학 점수가 100점인 데이터 (OR 조건)
filtered_df2 = df[(df['영어'] == 95) | (df['수학'] == 100)]

# isin() 메소드를 활용한 필터링
name_list = ['김철수', '최지우']
filtered_df3 = df[df['이름'].isin(name_list)]
```

## 4. 데이터 조작 및 변형
기존 데이터를 수정하거나 새로운 열을 생성, 삭제하는 방법입니다.

### 4.1 열 생성 및 수정
```python
# 새로운 열 추가 (총점 계산)
df['총점'] = df['국어'] + df['영어'] + df['수학']

# 기존 열 수정
df['합격여부'] = df['총점'] >= 250
```

### 4.2 데이터 정렬
```python
# '총점' 기준 내림차순 정렬
df_sorted = df.sort_values(by='총점', ascending=False)

# 인덱스 기준 정렬
df_index_sorted = df.sort_index(ascending=True)
```

### 4.3 열 및 행 삭제
```python
# 특정 열 삭제 (axis=1)
df_dropped = df.drop('합격여부', axis=1)

# 특정 행 삭제 (axis=0)
df_row_dropped = df.drop(0, axis=0)
```

## 5. 결측치(Missing Value) 처리
데이터에 값이 없는 경우(NaN)를 처리하는 필수 메소드입니다.

```python
# 샘플 결측치 데이터 생성
df.loc[0, '영어'] = np.nan
df.loc[2, '수학'] = np.nan

# 결측치 확인
print(df.isnull().sum())

# 결측치가 포함된 행 삭제
df_cleaned = df.dropna()

# 결측치를 특정 값으로 채우기 (평균값으로 채우기)
mean_math = df['수학'].mean()
df['수학'] = df['수학'].fillna(mean_math)
```

## 6. 데이터 집계 및 그룹화 (Groupby)
특정 기준에 따라 데이터를 그룹으로 나누고 통계량을 계산합니다.

```python
# 데이터프레임 확장
df['반'] = ['1반', '1반', '2반', '2반', '1반']

# '반'별 국어, 영어, 수학 점수의 평균 계산
group_mean = df.groupby('반')[['국어', '영어', '수학']].mean()

# 여러 개의 집계 함수 동시 적용
group_agg = df.groupby('반')['총점'].agg(['mean', 'sum', 'max', 'count'])
```

## 7. 데이터프레임 병합 (Merge & Concat)
여러 개의 데이터프레임을 하나로 합치는 방법입니다.

### 7.1 concat (단순 연결)
```python
df1 = pd.DataFrame({'A': ['A0', 'A1'], 'B': ['B0', 'B1']}, index=[0, 1])
df2 = pd.DataFrame({'A': ['A2', 'A3'], 'B': ['B2', 'B3']}, index=[2, 3])

# 위아래로 연결
result_v = pd.concat([df1, df2], axis=0)

# 좌우로 연결
result_h = pd.concat([df1, df2.reset_index(drop=True)], axis=1)
```

### 7.2 merge (공통 키 기준 병합)
```python
left = pd.DataFrame({'ID': [1, 2, 3], '이름': ['김철수', '이영희', '박민수']})
right = pd.DataFrame({'ID': [1, 2, 4], '수학': [90, 85, 70]})

# ID 열을 기준으로 내부 조인 (Inner Join)
merge_inner = pd.merge(left, right, on='ID', how='inner')

# ID 열을 기준으로 왼쪽 외부 조인 (Left Outer Join)
merge_left = pd.merge(left, right, on='ID', how='left')
```

## 8. 데이터 타입 변환 및 문자열 처리
```python
# 데이터 타입 변환 (astype)
df['국어'] = df['국어'].astype(float)

# 문자열 처리 (str 접근자 사용)
# '이름' 열에서 '김'으로 시작하는 데이터 필터링
kim_family = df[df['이름'].str.startswith('김')]

# 문자열 치환
df['반'] = df['반'].str.replace('반', 'Class')
```
