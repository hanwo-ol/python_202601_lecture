# 할리스 매장정보 크롤러


``` python 
import requests
from bs4 import BeautifulSoup
import pandas as pd

def hollys_store_crawler():
    result = []
    print("데이터 수집을 시작합니다...")

    for page in range(1, 47):
        # 할리스 매장 정보 URL (pageNo 파라미터 변경)
        url = f'https://www.hollys.co.kr/store/korea/korStore2.do?pageNo={page}'
        
        try:
            response = requests.get(url)
            soup = BeautifulSoup(response.text, 'html.parser')
            
            # 매장 정보가 담긴 테이블의 tbody 찾기
            tbody = soup.find('tbody')
            trs = tbody.find_all('tr')
            
            for tr in trs:
                tds = tr.find_all('td')
                
                # 검색 결과가 없는 경우 예외 처리
                if len(tds) <= 1:
                    break
                
                region = tds[0].text.strip()      # 지역
                name = tds[1].text.strip()        # 매장명
                address = tds[3].text.strip()     # 주소
                phone = tds[5].text.strip()       # 전화번호
                
                result.append({
                    '매장명': name,
                    '지역': region,
                    '주소': address,
                    '전화번호': phone
                })
            
            print(f"[{page}/46] 페이지 완료")
            
        except Exception as e:
            print(f"{page}페이지에서 오류 발생: {e}")

    # 데이터프레임 변환 및 CSV 저장
    df = pd.DataFrame(result)
    
    # 인코딩은 엑셀에서 바로 열 수 있도록 'utf-8-sig' 사용
    filename = 'hollys_branches.csv'
    df.to_csv(filename, index=False, encoding='utf-8-sig')
    
    print("-" * 30)
    print(f"수집 완료! 총 {len(df)}개의 매장 정보가 '{filename}'에 저장되었습니다.")

if __name__ == "__main__":
    hollys_store_crawler()
```
