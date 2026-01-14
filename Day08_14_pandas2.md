# 웹 크롤링 데이터 기반 데이터프레임 구축 및 분석 예제

## 1. 다중 페이지 웹 크롤링 (requests, bs4)
학습을 위해 크롤링 허용 사이트인 'Books to Scrape'를 대상으로 도서 정보를 수집합니다. 이 사이트는 페이지네이션 구조로 되어 있어 여러 페이지의 데이터를 반복문을 통해 수집하기 적합합니다.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np

# 데이터를 저장할 리스트
book_data = []

# 1페이지부터 3페이지까지 크롤링 (페이지네이션 처리)
for page in range(1, 4):
    url = f"http://books.toscrape.com/catalogue/page-{page}.html"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # 도서 정보가 담긴 컨테이너 선택
    books = soup.select('article.product_pod')
    
    for book in books:
        title = book.select_one('h3 > a')['title']
        price = book.select_one('p.price_color').text
        rating = book.select_one('p.star-rating')['class'][1] # 클래스명에서 별점 추출
        availability = book.select_one('p.instock.availability').text.strip()
        
        book_data.append({
            'Title': title,
            'Price': price,
            'Rating': rating,
            'Availability': availability
        })

# 데이터프레임 생성
df = pd.DataFrame(book_data)
print(df.head())
```

## 2. 데이터 정제 (Data Cleaning)
수집된 데이터는 문자열 형태이거나 불필요한 기호를 포함하고 있습니다. 분석을 위해 수치형 데이터로 변환합니다.

### 2.1 가격 데이터 수치화
```python
# '£' 기호 제거 및 float 타입 변환
df['Price'] = df['Price'].str.replace('£', '').astype(float)
```

### 2.2 별점 데이터 수치화
문자열로 된 별점(One, Two, ...)을 정수형(1, 2, ...)으로 매핑합니다.

```python
rating_map = {
    'One': 1,
    'Two': 2,
    'Three': 3,
    'Four': 4,
    'Five': 5
}
df['Rating'] = df['Rating'].map(rating_map)
```

### 2.3 재고 상태 데이터 정제
```python
# 'In stock' 문자열 포함 여부에 따라 불리언 값으로 변환
df['In_Stock'] = df['Availability'].str.contains('In stock')
# 기존 Availability 열 삭제
df = df.drop('Availability', axis=1)
```

## 3. 데이터프레임 심화 분석
정제된 데이터를 바탕으로 Pandas의 다양한 메소드를 활용하여 분석을 수행합니다.

### 3.1 기술 통계 및 정렬
```python
# 수치 데이터 요약
print(df.describe())

# 가격이 가장 높은 도서 상위 5개 추출
expensive_books = df.sort_values(by='Price', ascending=False).head(5)
print(expensive_books[['Title', 'Price']])
```

### 3.2 그룹화 분석 (Groupby)
별점에 따른 평균 가격을 계산하여 데이터 간의 상관관계를 파악합니다.

```python
# Rating별 평균 가격 및 도서 수 계산
rating_analysis = df.groupby('Rating')['Price'].agg(['mean', 'count', 'std'])
print(rating_analysis)
```

### 3.3 문자열 필터링 및 조건 검색
도서 제목에 특정 단어가 포함된 데이터를 추출합니다.

```python
# 제목에 'The'가 포함된 도서 중 별점이 4점 이상인 데이터
filtered_books = df[
    (df['Title'].str.contains('The', case=False)) & 
    (df['Rating'] >= 4)
]
print(filtered_books)
```

## 4. 데이터 변환 및 파생 변수 생성
분석 효율을 높이기 위해 기존 데이터를 가공합니다.

```python
# 가격 구간 설정 (Price_Grade)
# 0-20: Low, 20-40: Medium, 40+: High
bins = [0, 20, 40, np.inf]
labels = ['Low', 'Medium', 'High']
df['Price_Grade'] = pd.cut(df['Price'], bins=bins, labels=labels)

# 등급별 도서 수 확인
print(df['Price_Grade'].value_counts())
```

## 5. 최종 데이터 저장
정제 및 분석이 완료된 데이터프레임을 CSV 파일로 저장합니다.

```python
# 인덱스를 제외하고 저장
df.to_csv('cleaned_books_data.csv', index=False, encoding='utf-8-sig')
```

## 요약: 주요 사용 메소드
- `pd.DataFrame(list)`: 리스트 데이터를 데이터프레임으로 변환
- `str.replace()`: 문자열 내 특정 패턴 치환
- `astype()`: 데이터 타입 변경
- `map()`: 딕셔너리를 이용한 값 매핑
- `str.contains()`: 특정 문자열 포함 여부 확인 (불리언 인덱싱에 활용)
- `sort_values()`: 특정 열 기준 정렬
- `groupby().agg()`: 그룹화 및 다중 집계 연산
- `pd.cut()`: 연속형 수치를 범주형 데이터로 변환
- `to_csv()`: 데이터프레임을 CSV 파일로 출력
