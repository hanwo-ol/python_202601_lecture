# 다중 페이지 중첩 데이터 크롤링 및 고급 데이터 분석

## 1. 중첩 구조 데이터 수집 (Nested Data Crawling)
단순한 텍스트 추출을 넘어, 한 항목 내에 여러 개의 태그(Tag)가 포함된 중첩 구조 데이터를 수집합니다. 대상 사이트는 `quotes.toscrape.com`이며, 페이지네이션을 통해 명언, 저자, 태그 리스트를 동시에 추출합니다.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

all_quotes = []

# 1페이지부터 5페이지까지 순회
for page in range(1, 6):
    url = f"https://quotes.toscrape.com/page/{page}/"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    quote_elements = soup.select('div.quote')
    
    for element in quote_elements:
        text = element.select_one('span.text').text
        author = element.select_one('small.author').text
        # 태그는 한 명언에 여러 개가 존재하므로 리스트로 수집
        tags = [tag.text for tag in element.select('div.tags > a.tag')]
        
        all_quotes.append({
            'Quote': text,
            'Author': author,
            'Tags': tags,
            'Length': len(text) # 명언 길이 계산
        })

df = pd.DataFrame(all_quotes)
```

## 2. 데이터프레임 재구조화: Explode
`Tags` 열은 리스트 형태의 데이터를 가지고 있습니다. 이를 분석하기 위해 리스트의 각 요소를 개별 행으로 분리하는 `explode` 메소드를 사용합니다.

```python
# Tags 열의 리스트 요소를 행으로 확장
df_exploded = df.explode('Tags')

# 확장 후 인덱스가 중복되므로 재설정
df_exploded = df_exploded.reset_index(drop=True)

print(df_exploded.head(10))
```

## 3. 저자별 및 태그별 통계 분석
데이터가 확장된 상태에서 저자별 태그 사용 빈도와 명언 길이에 대한 통계를 산출합니다.

### 3.1 저자별 명언 수 및 평균 길이 계산
```python
author_stats = df.groupby('Author').agg({
    'Quote': 'count',
    'Length': 'mean'
}).rename(columns={'Quote': 'Quote_Count', 'Length': 'Avg_Length'})

# 명언 수가 많은 순으로 정렬
author_stats = author_stats.sort_values(by='Quote_Count', ascending=False)
print(author_stats.head())
```

### 3.2 태그 빈도 분석
```python
# 가장 많이 등장한 태그 상위 10개
top_tags = df_exploded['Tags'].value_counts().head(10)
print(top_tags)
```

## 4. 피벗 테이블(Pivot Table)을 이용한 교차 분석
저자와 태그 간의 관계를 파악하기 위해 피벗 테이블을 생성합니다. 특정 저자가 어떤 주제(태그)를 주로 다루었는지 수치화합니다.

```python
# 상위 5명의 저자 데이터만 필터링
top_authors = df['Author'].value_counts().head(5).index
df_top = df_exploded[df_exploded['Author'].isin(top_authors)]

# 저자(행)와 태그(열) 간의 빈도수 피벗 테이블 생성
pivot_table = df_top.pivot_table(
    index='Author', 
    columns='Tags', 
    values='Quote', 
    aggfunc='count', 
    fill_value=0
)

print(pivot_table)
```

## 5. 데이터 필터링 및 텍스트 처리 심화
특정 조건과 문자열 메소드를 결합하여 정밀한 데이터를 추출합니다.

### 5.1 다중 조건 필터링
```python
# 명언 길이가 100자 이상이면서 'life' 태그를 포함하는 데이터 추출
# df_exploded는 태그별로 분리되어 있으므로 이를 활용
complex_filter = df_exploded[
    (df_exploded['Length'] >= 100) & 
    (df_exploded['Tags'] == 'life')
]
```

### 5.2 저자 이름 정규화 (문자열 처리)
저자 이름의 공백을 제거하거나 대소문자를 통일하여 데이터 일관성을 확보합니다.

```python
# 저자 이름을 모두 대문자로 변경하고 성(Last Name)만 추출하는 함수 적용
df['Author_Upper'] = df['Author'].str.upper()
df['Author_LastName'] = df['Author'].apply(lambda x: x.split()[-1])

print(df[['Author', 'Author_LastName']].head())
```

## 6. 결측치 탐지 및 데이터 무결성 확인
크롤링 과정에서 발생할 수 있는 빈 값이나 오류 데이터를 처리합니다.

```python
# 태그가 없는(빈 리스트) 데이터 확인
# explode를 수행하면 빈 리스트는 NaN이 됨
missing_tags = df_exploded[df_exploded['Tags'].isnull()]

# 결측치가 있는 행 제거
df_final = df_exploded.dropna(subset=['Tags'])

# 중복된 명언 제거 (데이터 무결성 유지)
df_final = df_final.drop_duplicates(subset=['Quote', 'Tags'])
```

## 7. 요약: 고급 분석 메소드
- `explode()`: 리스트 형태의 열을 여러 행으로 전개
- `reset_index(drop=True)`: 기존 인덱스를 버리고 새 인덱스 할당
- `pivot_table()`: 엑셀의 피벗 테이블과 유사한 다차원 집계 수행
- `isin()`: 리스트 내 포함 여부를 확인하여 필터링
- `apply(lambda)`: 사용자 정의 함수를 열 전체에 적용
- `value_counts()`: 고유값의 빈도수 계산
- `agg()`: 여러 개의 집계 함수를 동시에 적용 (count, mean, sum 등)
