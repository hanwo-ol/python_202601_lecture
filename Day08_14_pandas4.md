# 고급 데이터 분석을 위한 웹 크롤링 및 Pandas 심화 활용

## 1. 복합 구조 데이터 수집 및 전처리
단순한 텍스트 추출을 넘어, 테이블 구조 내의 수치 데이터와 속성 데이터를 동시에 수집합니다. 대상은 'Scrape This Site'의 하키 팀 데이터(페이지네이션 포함)를 활용하여 수치형 데이터 분석 환경을 구축합니다.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np

all_teams = []

# 1페이지부터 5페이지까지 수집
for page in range(1, 6):
    url = f"https://www.scrapethissite.com/pages/forms/?page_num={page}"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # 테이블의 각 행(tr) 선택 (헤더 제외)
    rows = soup.select('tr.team')
    
    for row in rows:
        all_teams.append({
            'Team': row.select_one('.name').text.strip(),
            'Year': row.select_one('.year').text.strip(),
            'Wins': row.select_one('.wins').text.strip(),
            'Losses': row.select_one('.losses').text.strip(),
            'OT_Losses': row.select_one('.ot-losses').text.strip() if row.select_one('.ot-losses') else "0",
            'Win_Pct': row.select_one('.pct').text.strip(),
            'GF': row.select_one('.gf').text.strip(), # Goals For
            'GA': row.select_one('.ga').text.strip()  # Goals Against
        })

df = pd.DataFrame(all_teams)
```

## 2. 데이터 타입 최적화 및 수치 변환
크롤링된 데이터는 모두 문자열(object) 타입입니다. 이를 연산 가능한 수치형으로 변환하고, 결측치를 처리합니다.

```python
# 수치형 변환 (errors='coerce'는 변환 불가 데이터를 NaN으로 처리)
cols_to_fix = ['Year', 'Wins', 'Losses', 'OT_Losses', 'Win_Pct', 'GF', 'GA']
for col in cols_to_fix:
    df[col] = pd.to_numeric(df[col], errors='coerce')

# 결측치 확인 및 처리
df['OT_Losses'] = df['OT_Losses'].fillna(0)
df = df.dropna()

# 메모리 최적화: 범주형 데이터 변환
df['Team'] = df['Team'].astype('category')
```

## 3. 고급 분석: Groupby와 Transform
`transform` 메소드를 사용하면 그룹별 통계량을 원래 데이터프레임의 행 길이에 맞춰 반환할 수 있습니다. 이를 통해 각 팀이 해당 연도 평균 대비 얼마나 성과를 냈는지 계산합니다.

```python
# 연도별 평균 승리 수 계산 후 원본 데이터에 열로 추가
df['Year_Avg_Wins'] = df.groupby('Year')['Wins'].transform('mean')

# 평균 대비 승리 편차 계산
df['Win_Deviation'] = df['Wins'] - df['Year_Avg_Wins']

# 연도별 최대 득점(GF) 팀 필터링 (idxmax 활용)
idx = df.groupby('Year')['GF'].idxmax()
top_scoring_teams = df.loc[idx, ['Year', 'Team', 'GF']]
```

## 4. 멀티 인덱스(Multi-Index) 설정 및 활용
데이터를 연도와 팀명으로 계층적 인덱스를 설정하여 다차원 분석을 수행합니다.

```python
# 연도와 팀을 인덱스로 설정 및 정렬
df_multi = df.set_index(['Year', 'Team']).sort_index()

# 1990년 데이터만 추출
data_1990 = df_multi.loc[1990]

# 특정 연도 범위와 특정 팀의 데이터 추출 (Cross-section)
# 1990~1992년 사이의 모든 팀 데이터
data_range = df_multi.loc[1990:1992]

# 특정 팀(예: 'Chicago Blackhawks')의 연도별 기록만 추출
chicago_data = df_multi.xs('Chicago Blackhawks', level='Team')
```

## 5. 데이터 재구조화: Pivot Table 및 Melt
데이터의 형태를 분석 목적에 맞게 가로 또는 세로로 재배치합니다.

### 5.1 Pivot Table을 이용한 연도별 승률 요약
```python
# 연도별(행), 팀별(열) 승률(Win_Pct) 테이블 생성
pivot_wins = df.pivot_table(
    index='Year', 
    columns='Team', 
    values='Win_Pct', 
    aggfunc='mean'
)
```

### 5.2 Melt를 이용한 Wide-to-Long 변환
득점(GF)과 실점(GA)을 하나의 열로 통합하여 비교 분석하기 쉬운 형태로 변환합니다.

```python
df_melted = df.melt(
    id_vars=['Year', 'Team'], 
    value_vars=['GF', 'GA'], 
    var_name='Goal_Type', 
    value_name='Count'
)
```

## 6. 복합 조건 필터링 및 Query 메소드
`query` 메소드를 사용하면 SQL과 유사한 문법으로 복잡한 조건을 간결하게 표현할 수 있습니다.

```python
# 승률이 0.5 이상이고 실점(GA)이 200 미만인 데이터 필터링
filtered_df = df.query("Win_Pct >= 0.5 and GA < 200")

# 변수를 활용한 쿼리
target_year = 1995
filtered_df_var = df.query("Year == @target_year")
```

## 7. 사용자 정의 함수 적용 (Apply & Lambda)
행 단위의 복잡한 로직을 처리하기 위해 `apply`를 활용합니다.

```python
def get_performance_grade(row):
    if row['Win_Pct'] >= 0.6:
        return 'Elite'
    elif row['Win_Pct'] >= 0.5:
        return 'Good'
    else:
        return 'Below_Average'

# axis=1을 설정하여 행 전체를 함수로 전달
df['Grade'] = df.apply(get_performance_grade, axis=1)
```

## 8. 요약: 고급 분석 메소드 명세
- `pd.to_numeric(errors='coerce')`: 비정형 문자열의 수치 변환 및 예외 처리
- `groupby().transform()`: 그룹 통계량을 원본 배열 크기에 맞춰 분산
- `set_index(['A', 'B'])`: 계층적 인덱스(Multi-Index) 생성
- `xs(key, level)`: 멀티 인덱스에서 특정 레벨의 데이터 추출
- `pivot_table()`: 다차원 집계 및 데이터 재구조화
- `melt()`: 열 데이터를 행 데이터로 압축(Unpivoting)
- `query()`: 문자열 기반의 동적 데이터 필터링
- `apply(axis=1)`: 행 단위 복합 로직 연산 수행
