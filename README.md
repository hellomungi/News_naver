# News_naver

네이버 뉴스 페이지 크롤링을 위한 python 파일 입니다. <br>
최종 결과물은 python > DB > vue.js 를 통해서 아래와 같이 구성됩니다.<br>
<br>


<img src="https://github.com/hellomungi/News_naver/blob/main/news_naver.JPG" />


```
import pandas as pd
import requests

from bs4 import BeautifulSoup
from sqlalchemy import create_engine

def news_naver(url):
    ## Naver 뉴스페이지 가져오기
    html = requests.get(url)
    soup = BeautifulSoup(html.text, "lxml")
    
    ## html news list 찾기
    news_list1 = soup.find('ul', {'class':'realtimeNewsList'})
    news_list1 = str(news_list1)
    
    ## 편하게 찾기위해서 <dd 를 <mkkm으로 치환해줌
    news_list1 = news_list1.replace('<dd', '<mkkm').replace('<dt', '<mkkm').replace('</dd', '</mkkm').replace('</dt', '</mkkm')
    news_list1 = BeautifulSoup(news_list1, "lxml")
    sub_list = news_list1.find_all('mkkm', {'class':'articleSubject'})
    sum_list = news_list1.find_all('mkkm', {'class':'articleSummary'})
    
    ## Dataframe 생성
    news_df = pd.DataFrame(columns={'subject', 'summary', 'url', 'time'})

    news_num = 0
    for sub, sum in zip(sub_list, sum_list):
        ## 뉴스제목 , 뉴스내용, url 찾고 Dataframe에 저장
        subject = sub.text.replace("\n", "").replace("\t","")
        url = "https://finance.naver.com/" + sub.find('a').attrs['href']
        summary = sum.text.replace("\n", "").replace("\t","")
        time = sum.find('span', {'class':'wdate'}).text
        #print("\nsubject : ", subject, "\nsummary : ", summary, "\nurl : ", url, "\ntime : ", time)
        news_df.loc[news_num, 'subject'] = subject
        news_df.loc[news_num, 'summary'] = summary
        news_df.loc[news_num, 'url'] = url
        news_df.loc[news_num, 'time'] = pd.to_datetime(time)
        news_num = news_num + 1

    print(news_df)
    return news_df



if __name__ == '__main__':
    save_df = pd.DataFrame()

    for i in range(1, 4): ## 1~4페이지 까지 찾기위해서 for문 사용
        url = "https://finance.naver.com/news/news_list.naver?mode=LSS3D&section_id=101&section_id2=258&section_id3=401&page=" + str(i)
        news_df = news_naver(url)
	## news_df를 save_df 에 추가해줌
        save_df = save_df.append(news_df)
	
    ## 추가한 뉴스의 index 초기화
    save_df = save_df.reset_index(drop=True)
    engine = create_engine("mysql+pymysql://root:" + "PASSWORD" + "@localhost:3306/DBNAME?charset=utf8", encoding='utf-8')
    save_df.to_sql(name='TABLENAME', con=engine, if_exists='replace', index=False)```
