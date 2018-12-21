# 2018-12-21 학습내용

200 -> 응답코드

``` python
# -*- coding: utf-8 -*-
from flask import Flask,request
import requests
import json
import time
import os
from bs4 import BeautifulSoup as bs


app = Flask(__name__)

TELEGRAM_TOKEN = os.getenv('TELEGRAM_TOKEN')
TELEGRAM_URL = 'https://api.hphk.io/telegram'

@app.route('/{}'.format(TELEGRAM_TOKEN), methods=['POST'])
def telegram():
    #텔레그램으로부터 요청이 들어올 경우, 해당 요청을 처리하는 용도
    req = request.get_json()
    print(req)
    chat_id = req["message"]["from"]["id"]
    if(req["message"]["text"] == "안녕"):
        msg = "첫만남에는 존댓말을 써야죠!"
    elif(req["message"]["text"] == "안녕하세요"):
        msg = "인사 잘하시네유 ㅎㅎ"
    elif(req["message"]["text"] == "환율"):
        # 어떤 사이트든 ok 가장많은 환율정보를 끌어오시는 분께
        #url 설정
        url = "http://m.exchange.daum.net/mobile/exchange/exchangeMain.daum"
                
        #요청을 보내고 반응을 받는다.
        res = requests.get(url).text
        res2 = bs(res, 'html.parser')
        tmp = res2.select('#asiaBody > table > tbody')[0]
        tmp2 = tmp.select('.name')
        #나라 이름
        Country_name = []
        for i in tmp2:
          Country_name.append(i.text)
        
        #매매기준율
        tmp2 = tmp.select('.idx')
        exc_rate = [] #str형 주의!!
        for i in tmp2:
          exc_rate.append(i.text)
        
        #전일대비
        tmp2 = tmp.select('.idx_fluc cUp')
        idx_fluc_cUp = []
        for i in range(len(Country_name)):
          num = 4*i+2
          tmp2 = tmp.select('td')[num]
          idx_fluc_cUp.append(tmp2.text)
    
        #등락률
        rate_fluc_cUp = []
        for i in range(len(Country_name)):
          num = 4*i+3
          tmp2 = tmp.select('td')[num]
          rate_fluc_cUp.append(tmp2.text)
                
        dataList = []
        for i in range(len(Country_name)):
          data = {
            "Country_name": Country_name[i],
            "exc_rate": exc_rate[i],
            "idx_fluc_cUp": idx_fluc_cUp[i],
            "rate_fluc_cUp": rate_fluc_cUp[i]
          }
          dataList.append(data)
          msg = "환율정보에 대해 알려드리겠습니다."
          subMsg = ""
          for item in dataList:
            subMsg = subMsg + item["Country_name"] + "의 환율(단위) 은/는 {} 이고 전일대비 {} 변경됬고 등락률은 {} 입니다.".format(item["exc_rate"], item["idx_fluc_cUp"], item["rate_fluc_cUp"])
            subMsg = subMsg + "\n"
    ## join 기능
          msg = msg + subMsg
    url = 'https://api.hphk.io/telegram/bot{}/sendMessage'.format(TELEGRAM_TOKEN)
    requests.get(url, params = {"chat_id": req['message']['from']['id'], "text": msg})
    return '', 200
    
@app.route('/set_webhook')
def set_webhook():
    url = TELEGRAM_URL + '/bot' + TELEGRAM_TOKEN + '/setWebhook'
    params = {
        'url' : 'https://proj-a23822.c9users.io/{}'.format(TELEGRAM_TOKEN)
    }
    res = requests.get(url, params = params).text
    return res

## split
```

26/27일 자기주도학습

오전애 특강 점심먹고 자기주도학습 29장의 ppt 제작 주제는 자기가 가장 좋아하는것

27일 



``` python
# -*- coding: utf-8 -*-
from flask import Flask,request
import requests
import json
import time
import os
from bs4 import BeautifulSoup as bs


app = Flask(__name__)

TELEGRAM_TOKEN = os.getenv('TELEGRAM_TOKEN')
TELEGRAM_URL = 'https://api.hphk.io/telegram'

def get_total_info():
    url = "http://www.seoul-escape.com/reservation/change_date/"
    params = {
        "current_date" : time.strftime("%Y/%m/%d")
        
    }
    
    res = requests.get(url, params = params).text
    doc = json.loads(res)
    cafe_code = {
        "강남1호점" : 3,
        "홍대1호점" : 1,
        "부산 서면점" : 5,
        "인청 부평점" : 4,
        "강남2호점" : 11,
        "홍대2호점" : 10
    }
    total = {}
    #기본틀
    game_room_list = doc["gameRoomList"]
    for cafe in cafe_code:
        total[cafe] = []
        for room in game_room_list:
            if(cafe_code[cafe] == room["branch_id"]):
                total[cafe].append({"title": room["room_name"], "info": []})
    
    print(total)
    
    #앞에서 만든 틀에 데이터 집어넣기
    book_list = doc["bookList"]
    
    for cafe in total:
        for book in book_list:
            if(cafe == book["branch"]):
                for theme in total[cafe]:
                    if(theme["title"] == book["room"]):
                        if(book["booked"]):
                            booked = "예약완료"
                        else:
                            booked = "예약가능"
                        theme["info"].append("{} - {}".format(book["hour"], booked))
                        
    return total
    
def seoul_escape_list():
    total = get_total_info()
    return total.keys()
def seoul_escape_info(cd):
    total = get_total_info()
    cafe = total[cd]
    tmp = []
    for theme in cafe:
        tmp.append("{}\n {}".format(theme["title"], '\n'.join(theme["info"])))
    return tmp

print('\n'.join(seoul_escape_info("홍대1호점")))

cafelist = {
    "전체" : -1,
    "부천점": 15 ,
    "안양점": 13 ,
    "대구동성로2호점": 14 , 
    "대구동성로점": 9 ,
    "대구동성로점": 9 ,
    "궁동직영점": 1 ,
    "은행직영점": 2 ,
    "홍대상수점": 20 ,
    "강남점": 16 ,
    "건대점": 10 ,
    "홍대점": 11 ,
    "신촌점": 6 ,
    "잠실점": 21 ,
    "부평점": 17 ,
    "익산점": 12 ,
    "전주고사점": 8 , 
    "천안신부점": 18 ,
    "천안점": 3 ,
    "천안두정점": 7 ,
    "청주점": 4 
}

def master_key_info(cd):
    url = "http://www.master-key.co.kr/booking/booking_list_new"
    params = {
        'date' : time.strftime("%Y-%m-%d"),
        'store' : cd,
        'room' : ''
    }
    
    res = requests.post(url, params).text
    doc = bs(res, 'html.parser')
    ul = doc.select('.reserve .escape_view')
    theme_list = []
    for li in ul:
        title = li.select('p')[0].text
        info = ''
        for col in li.select('.col'):
            info = info + '{} - {} '.format(col.select_one('.time').text, col.select_one('.state').text)
    
        theme = {
            'title' : title,
            'info' : info
        }
        theme_list.append(theme)
    return theme_list

def master_key_list():
    url = "http://www.master-key.co.kr/home/office"
    
    res = requests.get(url).text
    #print(res)
    
    document = bs(res, 'html.parser')
    lis = document.select('.escape_list .escape_view')
    cafe_list = []
    #lis를 순회하면서 각종 정보를 추출한다.
    for li in lis:
        title = li.select_one('p').text
        if(title.endswith('NEW')):
            title = title[:-3]
        tel = li.select('dd')[1].text
        address = li.select('dd')[0].text
        link = 'http://www.master-key.co.kr' + li.select_one('a')["href"]
        
        cafe = {
            'title': title,
            'tel': tel,
            'address': address,
            'link': link
            
        }
        cafe_list.append(cafe)
    return cafe_list

@app.route('/{}'.format(TELEGRAM_TOKEN), methods=['POST'])
def telegram():
    #텔레그램으로부터 요청이 들어올 경우, 해당 요청을 처리하는 용도
    req = request.get_json()
    print(req)
    #print(chat_id)
    chat_id = req["message"]["from"]["id"]
    
    if(req["message"]["text"] == "안녕"):
        msg = "첫만남에는 존댓말을 써야죠!"
    elif(req["message"]["text"] == "안녕하세요"):
        msg = "인사 잘하시네유 ㅎㅎ"
    elif(req["message"]["text"] == "환율"):
        # 어떤 사이트든 ok 가장많은 환율정보를 끌어오시는 분께
        #url 설정
        url = "http://m.exchange.daum.net/mobile/exchange/exchangeMain.daum"
                
        #요청을 보내고 반응을 받는다.
        res = requests.get(url).text
        res2 = bs(res, 'html.parser')
        tmp = res2.select('#asiaBody > table > tbody')[0]
        tmp2 = tmp.select('.name')
        #나라 이름
        Country_name = []
        for i in tmp2:
          Country_name.append(i.text)
        
        #매매기준율
        tmp2 = tmp.select('.idx')
        exc_rate = [] #str형 주의!!
        for i in tmp2:
          exc_rate.append(i.text)
        
        #전일대비
        tmp2 = tmp.select('.idx_fluc cUp')
        idx_fluc_cUp = []
        for i in range(len(Country_name)):
          num = 4*i+2
          tmp2 = tmp.select('td')[num]
          idx_fluc_cUp.append(tmp2.text)
    
        #등락률
        rate_fluc_cUp = []
        for i in range(len(Country_name)):
          num = 4*i+3
          tmp2 = tmp.select('td')[num]
          rate_fluc_cUp.append(tmp2.text)
                
        dataList = []
        for i in range(len(Country_name)):
          data = {
            "Country_name": Country_name[i],
            "exc_rate": exc_rate[i],
            "idx_fluc_cUp": idx_fluc_cUp[i],
            "rate_fluc_cUp": rate_fluc_cUp[i]
          }
          dataList.append(data)
          msg = "환율정보에 대해 알려드리겠습니다."
          subMsg = ""
          for item in dataList:
            subMsg = subMsg + item["Country_name"] + "의 환율(단위) 은/는 {} 이고 전일대비 {} 변경됬고 등락률은 {} 입니다.".format(item["exc_rate"], item["idx_fluc_cUp"], item["rate_fluc_cUp"])
            subMsg = subMsg + "\n"
            
          msg = msg + subMsg
    # 마스터키 전체
    # 마스터키 ****점
    elif(req["message"]["text"].startswith('마스터키')):
        txt = req["message"]["text"]
        cafe_name = txt.split(' ')[1]
        cd = cafelist[cafe_name]
        if(cd > 0):
          data = master_key_info(cd)
        else:
          data = master_key_list()
        msg = []
        for d in data:
          msg.append('\n'.join(d.values()))
        msg = '\n'.join(msg)
        
    elif(req["message"]["text"].startswith("서이룸")):
        txt = req["message"]["text"]
        cafe_name = txt.split(' ')
        if(len(cafe_name)>2):
          cafe_name = ' '.join(cafe_name[1:3])
        else:
           cafe_name = cafe_name[-1]
           if(cafe_name == "전체"):
             data = seoul_escape_list()
           else:
             data = seoul_escape_info(cafe_name)
        msg = []
        for d in data:
          msg.append('\n'.join(d.values()))
        msg = '\n'.join(data) 
             
    else:
      msg = "설정되지 않은 명령어입니다."
    url = 'https://api.hphk.io/telegram/bot{}/sendMessage'.format(TELEGRAM_TOKEN)
    requests.get(url, params = {"chat_id": req['message']['from']['id'], "text": msg})
    return '', 200
    
@app.route('/set_webhook')
def set_webhook():
    url = TELEGRAM_URL + '/bot' + TELEGRAM_TOKEN + '/setWebhook'
    params = {
        'url' : 'https://proj-a23822.c9users.io/{}'.format(TELEGRAM_TOKEN)
    }
    res = requests.get(url, params = params).text
    return res
    


## 서이룸쪽 에러남 다시 한번 해볼것
```

