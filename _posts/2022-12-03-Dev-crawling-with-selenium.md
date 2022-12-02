---
layout: single
title: "Selenium으로 크롤링하기"
categories:
  - Dev
tags:
  - Python
  - Selenium
  - Crawling
  - Chrome driver
---

```python
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support.ui import WebDriverWait

import time

path='C:/chromedriver.exe'
url='https://3d.nicovideo.jp/search?category=all&download_filter=all&limit=28&order=1&perfect_match=1&sort=created_at&usable_animation=&word=VTuber&word_type=tag&work_type=all&page='

driver=webdriver.Chrome(path)
driver.get(url+'1')
driver.implicitly_wait(5)

visited=[]

for pagenum in range(8, 12):
    driver.get(url+str(pagenum))

    items=driver.find_elements(By.CLASS_NAME, 'work-box-link')
    listlen=len(items)
    for ln in range(listlen):
        ilist=driver.find_elements(By.CLASS_NAME, 'work-box-link')
        ilist[ln].click()

        #name=driver.find_element(By.CLASS_NAME, 'work-info-title')
        #visited.append(name.get_attribute('text'))
        dllink=driver.find_element(By.ID, 'download-link')
        dllink.click()
        dl=driver.find_element(By.ID, 'js-download')
        dl.click()
        driver.back()
```

- `driver.implicitly_wait(sec)` : 웹 페이지가 로드될 때까지 최대 `sec`초 기다림(`sec`초 지나면 그냥 실행)
  - `time.sleep(sec)`으로 직접 로드될 때까지 기다려도 됨
- `driver.back()`으로 돌아와도 이전에 검색했던 리스트를 사용할 수 없음  
  ![selenium crawling](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/selenium-crawling.png)
  - session은 같은 요소를 선택할 경우 같지만, element의 값이 달라짐(인스턴스가 달라지는 것과 비슷함)
