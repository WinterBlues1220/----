# 進階程式設計--Python爬蟲

**從亞洲大學資工系全職教師頁面抓取「姓名」與「研究領域」並存成 CSV / SQLite**

---

## 專案說明

 **來源網頁**：  
  https://csie.asia.edu.tw/zh_tw/TeacherIntroduction/Full_time_faculty  
 **抓取欄位**：  
  - 姓名 (`<span class="member-data-value-name">`)  
  - 研究領域 (`<span class="member-data-value-7">`)
 **功能**:
- 自動爬取教師資料
- 儲存為 UTF-8 CSV 檔案
- 可擴充存入 SQLite 資料庫
 **加分項目**:
- [x] Github repository ✅
- [x] Selenium ✅
- [ ] Scrapy
- [x] SQLite ✅
---

## 學習歷程

1. **環境與工具**  
   - 安裝 Windows 上的 pip，安裝 Selenium、BeautifulSoup4、Pandas  
   - 選用並啟動 Jupyter Notebook 作為開發環境  

2. **HTML 結構解析**  
   - 用瀏覽器開發者工具定位：  
     - 姓名 `.member-data-value-name`  
     - 研究領域 `.member-data-value-7`  

3. **爬蟲實作**  
   - Selenium + BeautifulSoup
   - 實作中發現問題，增加額外條件保證順利執行:
     - 加入 `WebDriverWait` 動態等待，確保 JS 完全載入  
     - 防呆判斷避免 `NoneType` 錯誤  

4. **資料儲存**  
   - 輸出成 UTF‑8 CSV  
   - 寫入單檔式 SQLite 資料庫，方便後續 SQL 查詢  

---

## 最終程式碼
```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import pandas as pd
import sqlite3
options = Options()
options.add_argument('--headless')
driver = webdriver.Chrome(service=Service(), options=options)

url = "https://csie.asia.edu.tw/zh_tw/TeacherIntroduction/Full_time_faculty"
driver.get(url)

WebDriverWait(driver, 10).until(
    EC.presence_of_all_elements_located((By.CLASS_NAME, "i-member-profile-data-wrap"))
)

html = driver.page_source
driver.quit()

soup = BeautifulSoup(html, "html.parser")
profiles = soup.find_all("div", class_="i-member-profile-data-wrap")

rows = []
for profile in profiles:
    name = profile.find("span", class_="member-data-value-name")
    expertise = profile.find("span", class_="member-data-value-7")

    rows.append({
        "name": name.text.strip() if name else "",
        "expertise": expertise.text.strip() if expertise else ""
    })

df = pd.DataFrame(rows)
print(df.head()) 

conn = sqlite3.connect("professors.db")
c = conn.cursor()

c.execute('''
CREATE TABLE IF NOT EXISTS professors (
    name TEXT,
    expertise TEXT
)
''')

for row in rows:
    c.execute("INSERT INTO professors (name, expertise) VALUES (?, ?)", 
              (row["name"], row["expertise"]))

conn.commit()
conn.close()
df.to_csv("亞大教師專長.csv", index=False, encoding="utf-8-sig")
```
