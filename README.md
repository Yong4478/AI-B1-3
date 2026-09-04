# 전기차 화재 정보 수집 및 정리 워크플로우

주요 목표: 전기차 화재에 대한 정보 부족을 해결함과 동시에 정보를 정리 및 시각화하여 유의미한 데이터를 추출해내는 것을 목표로 함.

주요 사용 AI 1: Gemini 3.1 pro

💡 추천 워크플로우 아키텍처 (개념도)

트리거(Trigger): 8시간마다 자동 실행 (스케줄러)

뉴스 검색(Search): 네이버 뉴스 API 또는 구글 알리미(RSS)로 '전기차 화재' 최신 기사 수집

AI 분석(AI Processing): ChatGPT API를 사용해 기사 본문에서 [장소, 기종, 원인, 인명피해, 피해금액] 추출

데이터베이스(DB): 구글 스프레드시트(Google Sheets)나 노션(Notion)에 표 형태로 자동 기록

통계 요약(Analysis): 누적된 데이터를 바탕으로 가장 많이 불난 기종과 주요 원인을 요약하여 슬랙(Slack)이나 이메일로 전송

🛠️ 방법 1: 노코드(No-Code) 자동화 툴 사용하기 (코딩 없이 만들기)
코딩에 익숙하지 않다면 Make.com이나 Zapier 같은 자동화 툴을 추천합니다.

Step 1: Make.com에서 Timer 모듈을 추가하고 8시간마다 실행되도록 설정합니다.
Step 2: RSS 모듈을 연결해 구글 뉴스에서 '전기차 화재'를 검색한 피드를 가져옵니다.
Step 3: OpenAI (ChatGPT) 모듈을 연결하고 프롬프트를 이렇게 작성합니다.
"다음 뉴스 기사를 읽고 아래 항목을 추출해서 JSON 형식으로 줘: 1. 화재 장소, 2. 차량 기종, 3. 화재 원인, 4. 인명 피해, 5. 피해 금액"

Step 4: Google Sheets 모듈을 연결해 ChatGPT가 뽑아준 데이터를 엑셀 표의 각 열(Column)에 맞게 추가합니다.
Step 5: 구글 스프레드시트의 '피벗 테이블' 기능을 사용하면, 어떤 기종이 가장 많이 탔는지, 원인이 무엇인지 자동으로 통계가 잡힙니다.

💻 방법 2: 파이썬(Python) 코드로 직접 만들기
코딩을 공부하고 계신다면, 파이썬을 이용해 직접 스크립트를 짜보는 것을 강력히 추천합니다!

아래는 전체적인 흐름을 보여주는 파이썬 코드 뼈대(Template) 입니다.
import schedule
import time
import pandas as pd
# 필요한 라이브러리: requests, bs4(크롤링), openai(AI 분석)

def job():
    print("🔄 8시간 주기 워크플로우 시작...")
    
    # 1. 뉴스 데이터 수집 (예: 네이버 뉴스 API 또는 크롤링)
    news_articles = get_latest_news("전기차 화재") 
    
    extracted_data = []
    
    # 2. AI로 정보 추출하기
    for article in news_articles:
        info = extract_info_with_chatgpt(article)
        # info는 딕셔너리 형태: {'장소': '...', '기종': '...', '원인': '...', '인명피해': '...', '피해금액': '...'}
        extracted_data.append(info)
        
    # 3. 표(DataFrame)로 만들고 저장하기
    df = pd.DataFrame(extracted_data)
    df.to_csv("ev_fire_records.csv", mode='a', index=False) # 엑셀 파일로 누적 저장
    print("✅ 표 업데이트 완료!\n", df)
    
    # 4. 통계 분석 및 요약
    analyze_data(df)

def analyze_data(df):
    print("📊 [현재까지의 화재 통계 요약]")
    # 가장 화재가 많이 발생한 기종 찾기
    most_frequent_model = df['기종'].value_counts().idxmax()
    print(f"🔥 가장 화재가 많이 발생한 기종: {most_frequent_model}")
    
    # 주요 화재 원인 요약
    common_causes = df['원인'].value_counts().head(3)
    print(f"🔍 주요 화재 원인 TOP 3:\n{common_causes}")

# 8시간마다 job 함수 실행
schedule.every(8).hours.do(job)

print("타이머가 시작되었습니다. 8시간마다 뉴스를 확인합니다.")
while True:
    schedule.run_pending()
    time.sleep(1)

    📝 코드 설명:
schedule 라이브러리를 사용해 8시간마다 코드가 자동으로 돌게 만들었습니다.
pandas 라이브러리를 사용하면 수집한 데이터를 엑셀(CSV) 형태의 표로 아주 쉽게 만들고, .value_counts()라는 함수 하나로 어떤 기ㅈ종이 가장 많은지 통계를 낼 수 있습니다.

       최종 액션:
뉴스 기사들로 수집된 모든 전기차 화재에 대한 정보 수집 및 정보 시각화 (화재 발생 시간, 장소, 차량 기종, 화재 원인, 인명 피해, 피해 금액, 등등) 

# 사용 툴 MAKE 확정
