# Web Crawler on Booking.com
## Purpose
<ul>
<li>When we reserves a hotel on some good holidays, there're so many fantastic rooms or facilities we want to live, but we have no time or moods (surely have money) to click the page seperately.</li>
<p></p>
<img src="/ImgForIntro/booking.png" width="70%" height="70%"/>

##### Why not take the advantage of elegant Python codes to grab the information to an excel file?
</ul>
<hr>

## OverView result
##### Here is the output:
<img src="/ImgForIntro/crawler_excel.png" width="70%" height="70%"/>
<hr>

### Built With

<ul>
    <li><a href="https://jupyterlab.readthedocs.io/en/stable/">Jupyterlab 3.4.3</a></li>
    <li><a href="https://www.selenium.dev/">Selenium 4.2.0</a></li>
    <li><a href="https://chromedriver.chromium.org/downloads">ChromeDriver  102.0.5005.61</a></li>
</ul>
<hr>

### Code example

```Python
%%time
import selenium
import numpy as np
from selenium import webdriver
from selenium.common.exceptions import NoSuchElementException
from selenium.webdriver.common.keys import Keys
import time
import pandas as pd
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from datetime import datetime
from retrying import retry
from itertools import zip_longest
import warnings

warnings.filterwarnings('ignore')
chrome_options = Options()
chrome_options.page_load_strategy = 'eager'
browser=webdriver.Chrome(r"C:\chromedriver_win32\chromedriver.exe", options=chrome_options)
browser.get("https://www.booking.com/searchresults.zh-tw.html?label=booking-name-WheOB4fo0bvQXDeTzH5xHwS267725091255%3Apl%3Ata%3Ap1%3Ap22%2C563%2C000%3Aac%3Aap%3Aneg%3Afi%3Atikwd-51481728%3Alp1012810%3Ali%3Adec%3Adm%3Appccp%3DUmFuZG9tSVYkc2RlIyh9YfqnDqqG8nt1O4nYvDr1lms&sid=90ef178b84caf8bb0367e392dae66490&aid=376396&ss=%E8%8A%B1%E8%93%AE%E5%B8%82&ssne=%E8%8A%B1%E8%93%AE%E5%B8%82&ssne_untouched=%E8%8A%B1%E8%93%AE%E5%B8%82&lang=zh-tw&src=searchresults&dest_id=-2631690&dest_type=city&checkin=2022-07-30&checkout=2022-08-03&group_adults=2&no_rooms=1&group_children=0&sb_travel_purpose=leisure&nflt=price%3DTWD-min-2200-1%3Bfc%3D2%3Breview_score%3D90")
def get_booking():
    global total_hotel_data
    global hotel_url
    
    #個別旅館頁面
    browser.switch_to.window(browser.window_handles[-1])
    
    #最長等7秒
    @retry(wait_fixed=1, stop_max_attempt_number=10)
    def get_hotel_ele():
        h_ele=WebDriverWait(browser, 10).until(EC.visibility_of_element_located((By.ID, 'hp_hotel_name')))
        return h_ele
    
    hotel_name2=get_hotel_ele()
    hotel_name=hotel_name2.text
    
    #檢查是否找過此旅館
    if (len(hotel_url[hotel_url['旅館總名稱'].astype(str).str.contains(hotel_name)]))>0:
        browser.close()
        browser.switch_to.window(browser.window_handles[-1])  
        return
    
    #增加網址
    one_url=pd.DataFrame({'旅館總名稱':[hotel_name], '旅館網址':[browser.current_url]})
    hotel_url=hotel_url.append(one_url)
    
    @retry(wait_fixed=1, stop_max_attempt_number=10)
    def get_room_facility_ele():
        rooms=WebDriverWait(browser, 10).until(EC.presence_of_all_elements_located((By.CLASS_NAME, 'hprt-roomtype-icon-link')))
        facilities=WebDriverWait(browser, 10).until(EC.presence_of_all_elements_located((By.CLASS_NAME, 'hprt-facilities-block')))
        return rooms, facilities
    
    rooms=get_room_facility_ele()[0]
    facilities=get_room_facility_ele()[1]

    #個別旅館資訊
    room_types=[]
    facilities_total=[]
    room_price=[]
    
    @retry(wait_fixed=1, stop_max_attempt_number=10)
    def table_ele():
        table=WebDriverWait(browser, 10).until(EC.presence_of_all_elements_located((By.CLASS_NAME, 'js-rt-block-row')))
        return table
    
    table=table_ele()
    
    #table=browser.find_elements_by_class_name('js-rt-block-row')
    table_lenth=len(table)


    #撈個別旅館價格與設施
    @retry(wait_fixed=1, stop_max_attempt_number=10)
    def find_price(i,j):
        price_xpath='//*[@id="hprt-table"]/tbody/tr['+str(i) + ']/td[' + str(j) + ']/div/div[1]/div[1]/div[2]/div/span[1]'
        price=browser.find_element_by_xpath(price_xpath).text
        room_price.append(price)
        
    def room_facility(k):
        room_types.append(rooms[k].text)
        facilities_total.append(facilities[k].text)

    k=0; i=1; j=3
    find_price(i,j)     
    room_facility(k)
    while i < table_lenth:
        try:
            j=3
            find_price(i+1,j)
            i=i+1
            k=k+1
            room_facility(k)
        except NoSuchElementException:       
            j=2
            find_price(i+1,j)
            i=i+1
            room_facility(k)
        except Exception:
            print (hotel_name+'發生例外')

    #個別旅館形成DataFrame
    one_hotel=pd.DataFrame({'旅館總名稱':[hotel_name], '房間類型':[''], '總價格': [''], '總設施': ['']})
    one_hotel=one_hotel.merge(pd.DataFrame(zip(room_types, room_price, facilities_total), columns=['房間類型','總價格','總設施']), how='outer')
    one_hotel['旅館總名稱'].fillna(one_hotel.iloc[0,0], inplace=True)
    one_hotel=one_hotel.iloc[1:,:]
    total_hotel_data=total_hotel_data.append(one_hotel)

    #關閉個別旅館頁面
    browser.close()
    browser.switch_to.window(browser.window_handles[-1])
    
        
#Global變數    
current_page=0
next_page=True
#是否有下一頁的按紐
@retry(wait_fixed=1, stop_max_attempt_number=10)
def exist_next_page():
    global next_page
    try:
        next_page=browser.find_element_by_xpath('//*[@id="search_results_table"]/div/div/div/div/div[7]/div[2]/nav/div/div[3]/button').is_enabled()
    except NoSuchElementException:
        next_page=False

#定義DataFrame
total_hotel_data=pd.DataFrame()
hotel_url=pd.DataFrame({'旅館總名稱':[], '旅館網址':[]})

while(next_page==True):

    @retry(wait_fixed=1, stop_max_attempt_number=10)
    def hotel_check():
        hotels_click=WebDriverWait(browser, 10).until(EC.visibility_of_all_elements_located((By.CLASS_NAME, 'b2f0d6a80e'))) #每個旅館的察看客房供應情況按鈕,class=b2f0d6a80e,f52f7d2689
        hotels=WebDriverWait(browser, 10).until(EC.visibility_of_all_elements_located((By.CLASS_NAME, 'a23c043802'))) #旅館list    
        return hotels_click, hotels

    hotels_click=hotel_check()[0]
    hotels=hotel_check()[1]
    click_counts=5
    hotel_length=len(hotels)
    hotel_5=hotel_length//click_counts
    hotel_remainder=hotel_length % click_counts

    #總共幾次loop
    for q in range(0, hotel_5):
        for w in range(0, click_counts):
            hotels_click[click_counts*q+w].click()
        for t in range(0, click_counts):
            get_booking()

    #餘數loop
    if (hotel_remainder>0):
        for r in list(range(click_counts*hotel_5, hotel_length)):
            hotels_click[r].click()
        for w in range(0, hotel_remainder):
            get_booking()

    #判斷是否有下一頁
    exist_next_page()
    
    
    if (next_page==True):
        
        current_page=current_page+1
        print ('第'+str(current_page)+'頁已經爬完')
        @retry(wait_fixed=1, stop_max_attempt_number=10)
        def go_next_page():
            browser.find_element_by_class_name('f78c3700d2').click()  #前往下一頁
        go_next_page()
        time.sleep(2.5)
        browser.switch_to.window(browser.window_handles[-1])
        #WebDriverWait(browser, 7).until(EC.presence_of_element_located((By.CLASS_NAME, 'f78c3700d2'))).click() #前往下一頁
        
            
    else:
        current_page=current_page+1
        print ('已經爬完'+str(current_page)+'頁全部旅店')
        break
        


#處理DataFrame
hotel_url.drop_duplicates(subset='旅館總名稱', keep='first', inplace=True)
hotel_url.reset_index(drop=True, inplace=True)
total_hotel_data.reset_index(drop=True, inplace=True)
total_hotel_data.drop_duplicates(keep='first', inplace=True)
total_hotel_data['旅館名稱']=np.where(total_hotel_data['旅館總名稱'].str.contains('\n'), total_hotel_data['旅館總名稱'].str.split('\n').str[1], total_hotel_data['旅館總名稱'].str.split(' ').str[1])
total_hotel_data['旅館類型']=np.where(total_hotel_data['旅館總名稱'].str.contains('\n'), total_hotel_data['旅館總名稱'].str.split('\n').str[0], total_hotel_data['旅館總名稱'].str.split(' ').str[0])
total_hotel_data['坪數m²']=np.where(total_hotel_data['總設施'].str.extract('(\d+).m²').notna(), total_hotel_data['總設施'].str.extract('(\d+).m²').astype('Int32'),'')
total_hotel_data.loc[total_hotel_data['總設施'].str.contains('浴缸'), '浴缸']='V'
total_hotel_data['總設施']=total_hotel_data['總設施'].str.replace('\n',' ').str.replace('查看更多','')
total_hotel_data['價格']=total_hotel_data['總價格'].str.split(' ').str[1]
total_hotel_data['價格']=total_hotel_data['價格'].str.replace(',','').astype('int')
total_hotel_data=total_hotel_data[['旅館總名稱','旅館名稱','旅館類型','房間類型','價格','坪數m²','浴缸','總設施']]

with pd.ExcelWriter(r'C:\Jupyterlab\Output_Excel\Booking.xlsx', engine = 'xlsxwriter') as writer:
    total_hotel_data.to_excel(writer, '總表', index = False, freeze_panes = (1,1))
    hotel_url.to_excel(writer, '網址', index = False, freeze_panes = (1,1))
```

