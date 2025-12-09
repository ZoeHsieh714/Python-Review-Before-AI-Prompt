# 🐍 Python 快速複習 Day 5-7: 錯誤處理、套件管理與 AI 專案

> **學習時間:** Day 5 (40分鐘) + Day 6 (45分鐘) + Day 7 (45分鐘)  
> **目標:** 掌握異常處理、logging、虛擬環境、Pydantic 和專案結構

---

## 📅 Day 5: 錯誤處理與除錯 (40分鐘)

### 學習重點
- [ ] try-except 異常處理
- [ ] 自定義異常
- [ ] 日誌記錄
- [ ] 除錯技巧

### 實作練習

```python
# ============================================================
# 1. 基本異常處理
# ============================================================

def divide(a: float, b: float) -> float:
    """
    除法函數,示範異常處理
    
    try-except 結構:
    - try: 嘗試執行的程式碼
    - except: 捕捉特定錯誤並處理
    """
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        # 捕捉除以零的錯誤
        print("錯誤: 除數不能為零")
        return 0
    except TypeError:
        # 捕捉型別錯誤 (如傳入字串)
        print("錯誤: 參數必須是數字")
        return 0

# 測試
print(divide(10, 2))    # 5.0
print(divide(10, 0))    # 錯誤: 除數不能為零, 返回 0
print(divide("10", 2))  # 錯誤: 參數必須是數字, 返回 0


# ============================================================
# 2. 多重異常處理
# ============================================================

import json

def load_config(filename: str) -> dict:
    """
    載入配置檔案,示範多重異常處理
    
    按照從具體到一般的順序捕捉異常
    """
    try:
        with open(filename, "r") as f:
            return json.load(f)
    except FileNotFoundError:
        # 檔案不存在
        print(f"檔案不存在: {filename}")
        return {}
    except json.JSONDecodeError:
        # JSON 格式錯誤
        print(f"JSON 格式錯誤: {filename}")
        return {}
    except Exception as e:
        # 捕捉所有其他異常 (最後的保險)
        # e 是異常物件,包含錯誤訊息
        print(f"未預期的錯誤: {e}")
        return {}

# 💡 異常處理順序很重要:
# - 先捕捉具體的異常 (FileNotFoundError, JSONDecodeError)
# - 最後捕捉一般的異常 (Exception)
# - 如果順序反了,具體的異常永遠不會被捕捉到


# ============================================================
# 3. finally 子句 (清理資源)
# ============================================================

def process_file(filename: str):
    """
    finally 區塊無論是否發生異常都會執行
    常用於清理資源 (關閉檔案、釋放連線等)
    """
    file = None
    try:
        file = open(filename, "r")
        content = file.read()
        return content
    except FileNotFoundError:
        print("檔案不存在")
        return None
    finally:
        # 無論是否發生異常,都會執行
        if file:
            file.close()
            print("檔案已關閉")

# 執行流程:
# 1. 嘗試執行 try 區塊
# 2. 如果發生異常,執行對應的 except 區塊
# 3. 無論如何,最後執行 finally 區塊

# 💡 with 語句 vs try-finally:
# with open(...) as f:  # 推薦,自動處理關閉
#     ...
# 
# 等同於:
# f = open(...)
# try:
#     ...
# finally:
#     f.close()


# ============================================================
# 4. 自定義異常
# ============================================================

class APIError(Exception):
    """
    自定義異常基類
    
    繼承自 Exception,可以加入自己的屬性和方法
    """
    pass

class RateLimitError(APIError):
    """
    速率限制錯誤
    
    繼承自 APIError,形成異常階層
    """
    pass

class AuthenticationError(APIError):
    """認證錯誤"""
    pass

def call_api(endpoint: str):
    """模擬 API 呼叫"""
    if endpoint == "/rate-limited":
        raise RateLimitError("API 呼叫次數超過限制")
    elif endpoint == "/unauthorized":
        raise AuthenticationError("API Key 無效")
    return "Success"

# 使用自定義異常
try:
    call_api("/rate-limited")
except RateLimitError as e:
    print(f"速率限制: {e}")
    # 可以實作重試邏輯
except AuthenticationError as e:
    print(f"認證錯誤: {e}")
    # 可以提示使用者檢查 API Key
except APIError as e:
    print(f"API 錯誤: {e}")
    # 捕捉所有 API 相關錯誤

# 💡 為什麼使用自定義異常:
# 1. 語意清楚 - RateLimitError 比 Exception 更明確
# 2. 分類處理 - 可以針對不同錯誤採取不同策略
# 3. 異常階層 - 可以捕捉一類異常 (APIError)


# ============================================================
# 5. 日誌記錄 (比 print 更專業)
# ============================================================

import logging

# 設定日誌
logging.basicConfig(
    level=logging.INFO,  # 日誌級別: DEBUG < INFO < WARNING < ERROR < CRITICAL
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    # %(asctime)s - 時間
    # %(name)s - logger 名稱
    # %(levelname)s - 級別 (INFO, ERROR 等)
    # %(message)s - 訊息內容
)

logger = logging.getLogger(__name__)  # 取得 logger 物件

def chat_with_ai(message: str):
    """示範日誌記錄"""
    logger.info(f"收到訊息: {message}")  # INFO 級別
    try:
        # 模擬 API 呼叫
        response = f"回應: {message}"
        logger.info("API 呼叫成功")
        return response
    except Exception as e:
        logger.error(f"API 呼叫失敗: {e}")  # ERROR 級別
        raise

# 日誌級別說明:
# DEBUG - 詳細的除錯資訊
# INFO - 一般資訊 (程式正常運作)
# WARNING - 警告訊息 (可能有問題,但不影響運作)
# ERROR - 錯誤訊息 (功能失敗)
# CRITICAL - 嚴重錯誤 (程式可能崩潰)

# 💡 logging vs print:
# print:
# - 簡單,適合除錯
# - 無法控制輸出級別
# - 無法輸出到檔案
#
# logging:
# - 可以設定級別 (只顯示重要訊息)
# - 可以輸出到檔案
# - 可以加入時間戳記
# - 生產環境的標準做法


# ============================================================
# 6. 除錯技巧
# ============================================================

def debug_example():
    """常用的除錯技巧"""
    data = {"name": "張三", "age": 25}
    
    # 技巧 1: 使用 print 除錯
    print(f"Debug: data = {data}")
    
    # 技巧 2: 使用 type() 檢查型別
    print(f"Type: {type(data)}")  # <class 'dict'>
    
    # 技巧 3: 使用 dir() 查看可用方法
    # print(dir(data))  # 列出所有方法和屬性
    
    # 技巧 4: 使用 assert 驗證假設
    assert isinstance(data, dict), "data 必須是字典"
    assert "name" in data, "data 必須包含 name"
    # assert 條件為 False 時會引發 AssertionError
    
    # 技巧 5: 使用 pdb 除錯器 (進階)
    # import pdb; pdb.set_trace()  # 設定中斷點
    # 程式會在此暫停,可以互動式檢查變數
    
    # 技巧 6: 使用 try-except 捕捉並顯示詳細錯誤
    try:
        result = data["不存在的鍵"]
    except KeyError as e:
        print(f"KeyError: {e}")
        print(f"可用的鍵: {list(data.keys())}")

# 💡 除錯流程:
# 1. 重現問題
# 2. 加入 print 或 logging 觀察變數
# 3. 使用 assert 驗證假設
# 4. 使用 try-except 捕捉異常
# 5. 使用 pdb 進行互動式除錯 (複雜問題)
```

### 💡 Day 5 核心概念總結

1. **try-except**: 捕捉和處理異常
2. **finally**: 無論如何都會執行的清理程式碼
3. **自定義異常**: 建立有意義的異常階層
4. **logging**: 專業的日誌記錄,比 print 更強大
5. **assert**: 驗證程式假設,早期發現問題

### 檢查點
- [ ] 能正確使用 try-except 處理錯誤
- [ ] 知道何時使用 finally
- [ ] 能使用 logging 記錄日誌
- [ ] 掌握基本的除錯技巧

---

## 📅 Day 6: 常用套件與工具 (45分鐘)

### 學習重點
- [ ] pip 套件管理
- [ ] 虛擬環境
- [ ] Jupyter Notebook
- [ ] 常用第三方套件

### Bash 指令詳解

```bash
# ============================================================
# 1. pip 套件管理
# ============================================================

# 安裝套件
pip install openai           # 安裝最新版本
pip install python-dotenv    # 環境變數管理
pip install requests         # HTTP 請求

# 安裝特定版本
pip install openai==1.3.0    # 安裝 1.3.0 版本
pip install "openai>=1.0.0"  # 安裝 1.0.0 或更新版本

# 從 requirements.txt 安裝
pip install -r requirements.txt
# requirements.txt 包含所有依賴套件和版本

# 匯出已安裝套件
pip freeze > requirements.txt
# 將當前環境的所有套件和版本寫入檔案

# 升級套件
pip install --upgrade openai  # 升級到最新版本
pip install -U openai         # 同上,-U 是 --upgrade 的縮寫

# 查看已安裝套件
pip list                      # 列出所有套件
pip show openai               # 顯示特定套件的詳細資訊

# 卸載套件
pip uninstall openai          # 移除套件


# ============================================================
# 2. 虛擬環境 (重要!)
# ============================================================

# 為什麼需要虛擬環境:
# - 隔離專案依賴,避免版本衝突
# - 專案 A 需要 openai 1.0,專案 B 需要 openai 2.0
# - 每個專案有獨立的套件環境

# 建立虛擬環境
python -m venv venv
# venv 是模組名稱
# 第二個 venv 是目錄名稱 (可以自訂,如 myenv)

# 啟動虛擬環境 (Windows)
venv\Scripts\activate
# 啟動後,命令提示字元會顯示 (venv)

# 啟動虛擬環境 (Mac/Linux)
source venv/bin/activate

# 停用虛擬環境
deactivate

# 💡 虛擬環境最佳實踐:
# 1. 每個專案建立獨立的虛擬環境
# 2. 虛擬環境目錄 (venv/) 加入 .gitignore
# 3. 使用 requirements.txt 記錄依賴
# 4. 團隊成員用 pip install -r requirements.txt 安裝相同版本


# ============================================================
# 3. Jupyter Notebook
# ============================================================

# 安裝
pip install jupyter

# 啟動 Jupyter Notebook
jupyter notebook
# 會在瀏覽器中開啟 http://localhost:8888

# 安裝 kernel (讓 Jupyter 使用特定虛擬環境)
python -m ipykernel install --user --name=myenv
# myenv 是 kernel 名稱,會顯示在 Jupyter 的 kernel 選單中

# 列出已安裝的 kernel
jupyter kernelspec list

# 移除 kernel
jupyter kernelspec uninstall myenv

# 💡 Jupyter Notebook 優點:
# - 互動式執行程式碼
# - 即時查看結果
# - 適合資料分析和實驗
# - 可以加入 Markdown 說明
```

### Python 程式碼詳解

```python
# ============================================================
# 4. 常用第三方套件
# ============================================================

# requests - HTTP 請求 (呼叫 API 必備)
import requests

response = requests.get("https://api.example.com/data")
# GET 請求,取得資料

if response.status_code == 200:
    # 200 表示成功
    data = response.json()  # 將 JSON 回應轉為 dict
    print(data)
else:
    print(f"錯誤: {response.status_code}")

# POST 請求 (傳送資料)
response = requests.post(
    "https://api.example.com/data",
    json={"key": "value"},  # 自動轉為 JSON
    headers={"Authorization": "Bearer token"}
)

# 💡 requests 常用方法:
# requests.get() - 取得資料
# requests.post() - 傳送資料
# requests.put() - 更新資料
# requests.delete() - 刪除資料


# python-dotenv - 環境變數管理
from dotenv import load_dotenv
import os

load_dotenv()  # 載入 .env 檔案
api_key = os.getenv("API_KEY")


# datetime - 日期時間處理
from datetime import datetime, timedelta

now = datetime.now()  # 當前時間
print(now)  # 2024-11-26 14:30:00.123456

# 格式化輸出
formatted = now.strftime("%Y-%m-%d %H:%M:%S")
print(formatted)  # 2024-11-26 14:30:00

# 日期計算
tomorrow = now + timedelta(days=1)  # 明天
next_week = now + timedelta(weeks=1)  # 下週
one_hour_ago = now - timedelta(hours=1)  # 一小時前

# 💡 strftime 格式代碼:
# %Y - 年份 (4 位數)
# %m - 月份 (01-12)
# %d - 日期 (01-31)
# %H - 小時 (00-23)
# %M - 分鐘 (00-59)
# %S - 秒 (00-59)


# typing - 型別提示
from typing import List, Dict, Optional, Union, Any

def process_data(
    items: List[str],                    # 字串列表
    config: Optional[Dict[str, Any]] = None  # 可選的字典
) -> Union[str, None]:                   # 返回字串或 None
    """
    型別提示讓程式碼更易讀,IDE 也能提供更好的支援
    """
    if config is None:
        config = {}
    return "processed"

# 常用型別:
# List[T] - 列表,元素型別為 T
# Dict[K, V] - 字典,鍵型別 K,值型別 V
# Optional[T] - T 或 None
# Union[T1, T2] - T1 或 T2
# Any - 任意型別
# Tuple[T1, T2] - 固定長度的 tuple
# Callable[[Arg1, Arg2], Return] - 函數型別
```

### 💡 Day 6 核心概念總結

1. **pip**: Python 套件管理工具
2. **虛擬環境**: 隔離專案依賴,避免版本衝突
3. **requirements.txt**: 記錄專案依賴
4. **Jupyter Notebook**: 互動式開發環境
5. **requests**: HTTP 請求庫
6. **typing**: 型別提示,提高程式碼品質

### 檢查點
- [ ] 能建立和使用虛擬環境
- [ ] 熟悉 pip 基本指令
- [ ] 能啟動 Jupyter Notebook
- [ ] 了解 requirements.txt 的作用

---

## 📅 Day 7: AI 專案相關知識 (45分鐘)

### 學習重點
- [ ] API 呼叫基礎
- [ ] Pydantic 資料驗證
- [ ] 專案結構組織
- [ ] 配置管理

### 實作練習

```python
# ============================================================
# 1. API 呼叫基礎 (使用 requests)
# ============================================================

import requests
import json

def call_api(url: str, data: dict) -> dict:
    """
    呼叫 API 的標準模式
    
    Args:
        url: API 端點
        data: 要傳送的資料
        
    Returns:
        API 回應的 dict
    """
    # 設定 HTTP 標頭
    headers = {
        "Content-Type": "application/json",  # 告訴伺服器我們傳送 JSON
        "Authorization": "Bearer YOUR_API_KEY"  # 認證 token
    }
    
    try:
        response = requests.post(
            url,
            headers=headers,
            json=data,      # 自動將 dict 轉為 JSON
            timeout=30      # 30 秒逾時
        )
        
        # 檢查 HTTP 錯誤 (4xx, 5xx)
        response.raise_for_status()
        # 如果狀態碼不是 2xx,會引發 HTTPError
        
        return response.json()  # 將 JSON 回應轉為 dict
        
    except requests.exceptions.Timeout:
        print("請求逾時")
        return {}
    except requests.exceptions.HTTPError as e:
        print(f"HTTP 錯誤: {e}")
        return {}
    except requests.exceptions.RequestException as e:
        # 捕捉所有 requests 相關錯誤
        print(f"API 錯誤: {e}")
        return {}

# 💡 API 呼叫流程:
# 1. 準備資料和標頭
# 2. 發送請求
# 3. 檢查狀態碼
# 4. 解析回應
# 5. 處理錯誤


# ============================================================
# 2. Pydantic 資料驗證 (重要!)
# ============================================================

from pydantic import BaseModel, Field, validator
from typing import List

class Person(BaseModel):
    """
    Pydantic 模型定義資料結構和驗證規則
    
    好處:
    1. 自動型別檢查
    2. 資料驗證
    3. 生成 JSON Schema
    4. IDE 自動完成
    """
    name: str = Field(..., min_length=1, max_length=50)
    # ... 表示必填
    # min_length, max_length 限制字串長度
    
    age: int = Field(..., ge=0, le=150)
    # ge = greater than or equal (大於等於)
    # le = less than or equal (小於等於)
    
    email: str
    
    @validator('email')
    def validate_email(cls, v):
        """
        自定義驗證器
        
        cls: 類別本身
        v: 欄位值
        """
        if '@' not in v:
            raise ValueError('無效的 email')
        return v  # 必須返回值

# 使用 Pydantic 模型
try:
    person = Person(
        name="張三",
        age=25,
        email="test@example.com"
    )
    print(person.model_dump())  # 轉為 dict
    # {'name': '張三', 'age': 25, 'email': 'test@example.com'}
    
    print(person.model_dump_json())  # 轉為 JSON 字串
    
except Exception as e:
    print(f"驗證錯誤: {e}")

# 驗證失敗範例
try:
    invalid_person = Person(
        name="",  # ❌ 太短
        age=200,  # ❌ 超過範圍
        email="invalid"  # ❌ 沒有 @
    )
except Exception as e:
    print(f"驗證錯誤: {e}")

# 💡 Pydantic 在 AI 開發中的應用:
# 1. 定義 API 請求/回應格式
# 2. 驗證 LLM 輸出的結構化資料
# 3. 生成 JSON Schema 給 LLM 參考
# 4. 確保資料品質


# ============================================================
# 3. 專案結構組織
# ============================================================

"""
推薦的 AI 專案結構:

my_ai_project/
├── .env                    # 環境變數 (不提交到 Git)
├── .gitignore             # Git 忽略檔案
├── requirements.txt       # 套件依賴
├── README.md             # 專案說明
├── src/                  # 原始碼
│   ├── __init__.py       # 讓 src 成為 Python 套件
│   ├── main.py          # 主程式入口
│   ├── config.py        # 配置管理
│   ├── api/             # API 相關
│   │   ├── __init__.py
│   │   └── client.py    # API 客戶端
│   ├── models/          # 資料模型
│   │   ├── __init__.py
│   │   └── schemas.py   # Pydantic 模型
│   └── utils/           # 工具函數
│       ├── __init__.py
│       └── helpers.py
├── tests/               # 測試
│   ├── __init__.py
│   └── test_main.py
└── data/                # 資料
    ├── prompts/         # 提示詞模板
    └── outputs/         # 輸出結果

為什麼這樣組織:
1. 清楚的目錄結構,容易找到檔案
2. 分離關注點 (API、模型、工具)
3. 方便測試
4. 易於擴展
"""


# ============================================================
# 4. 配置管理範例
# ============================================================

# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    """
    使用 Pydantic Settings 管理配置
    
    自動從環境變數讀取設定
    """
    openai_api_key: str  # 從 OPENAI_API_KEY 環境變數讀取
    model: str = "gpt-4"  # 預設值
    temperature: float = 0.7
    max_tokens: int = 1000
    
    class Config:
        env_file = ".env"  # 從 .env 檔案讀取
        env_file_encoding = "utf-8"

# 建立全域設定物件
settings = Settings()

# 使用設定
print(f"使用模型: {settings.model}")
print(f"溫度: {settings.temperature}")

# 💡 為什麼使用 Pydantic Settings:
# 1. 型別檢查 - 確保配置值的型別正確
# 2. 預設值 - 不用每個都設定
# 3. 環境變數 - 自動讀取,不用手動 os.getenv()
# 4. 驗證 - 可以加入驗證規則


# ============================================================
# 5. 工具函數範例
# ============================================================

# utils.py
from typing import List, Dict

def format_messages(
    system_prompt: str,
    user_message: str
) -> List[Dict[str, str]]:
    """
    格式化訊息為 OpenAI API 格式
    
    Args:
        system_prompt: 系統提示
        user_message: 使用者訊息
        
    Returns:
        訊息列表
    """
    return [
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": user_message}
    ]

def count_tokens(text: str) -> int:
    """
    簡單的 token 計數 (實際應使用 tiktoken)
    
    這只是粗略估算,實際 token 數可能不同
    """
    return len(text.split())

def truncate_text(text: str, max_length: int = 100) -> str:
    """
    截斷文字
    
    Args:
        text: 原始文字
        max_length: 最大長度
        
    Returns:
        截斷後的文字
    """
    if len(text) <= max_length:
        return text
    return text[:max_length] + "..."

# 💡 工具函數設計原則:
# 1. 單一職責 - 每個函數只做一件事
# 2. 可重複使用 - 不要寫死特定值
# 3. 型別提示 - 讓使用者知道如何呼叫
# 4. 文檔字串 - 說明功能和參數


# ============================================================
# 6. 主程式範例
# ============================================================

# main.py
from config import settings
from utils import format_messages, count_tokens

def main():
    """
    主程式入口
    
    組織程式流程:
    1. 載入配置
    2. 初始化元件
    3. 執行主要邏輯
    4. 處理錯誤
    """
    print(f"使用模型: {settings.model}")
    print(f"溫度: {settings.temperature}")
    
    # 格式化訊息
    messages = format_messages(
        system_prompt="你是專業的程式設計助手",
        user_message="如何學習 Python?"
    )
    
    print(f"訊息數量: {len(messages)}")
    
    # 計算 token (粗略)
    total_tokens = sum(count_tokens(msg["content"]) for msg in messages)
    print(f"預估 token: {total_tokens}")

# Python 慣例: 只有直接執行此檔案時才執行 main()
# 如果被 import,不會執行
if __name__ == "__main__":
    main()

# 💡 if __name__ == "__main__" 的作用:
# 直接執行: python main.py
#   -> __name__ == "__main__" -> 執行 main()
#
# 被 import: from main import something
#   -> __name__ == "main" -> 不執行 main()
```

### 💡 Day 7 核心概念總結

1. **API 呼叫**: 標準的請求-回應模式
2. **Pydantic**: 資料驗證和型別檢查
3. **專案結構**: 清楚的目錄組織
4. **配置管理**: 使用 Pydantic Settings
5. **工具函數**: 可重複使用的輔助函數
6. **主程式**: if __name__ == "__main__" 模式

### 檢查點
- [ ] 理解 API 呼叫的基本流程
- [ ] 能使用 Pydantic 定義資料模型
- [ ] 知道如何組織專案結構
- [ ] 能撰寫配置和工具函數

---

## 🎓 Day 5-7 總複習

### 必須掌握的概念
1. **異常處理** - try-except-finally
2. **logging** - 專業的日誌記錄
3. **虛擬環境** - 隔離專案依賴
4. **pip** - 套件管理
5. **Pydantic** - 資料驗證
6. **專案結構** - 組織程式碼

### 實戰練習建議

```python
# 綜合練習: 建立完整的 AI 專案框架
from pydantic import BaseModel, Field
from pydantic_settings import BaseSettings
from pathlib import Path
import logging

# 1. 配置管理
class Settings(BaseSettings):
    openai_api_key: str
    model: str = "gpt-4"
    temperature: float = 0.7
    
    class Config:
        env_file = ".env"

# 2. 資料模型
class ChatMessage(BaseModel):
    role: str = Field(..., pattern="^(system|user|assistant)$")
    content: str = Field(..., min_length=1)

# 3. 日誌設定
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# 4. 主程式
def main():
    try:
        settings = Settings()
        logger.info(f"使用模型: {settings.model}")
        
        msg = ChatMessage(role="user", content="Hello")
        logger.info(f"建立訊息: {msg}")
        
    except Exception as e:
        logger.error(f"錯誤: {e}")
        raise

if __name__ == "__main__":
    main()
```

---

## 🎉 恭喜完成 Python 一週快速複習!

您現在已經掌握:

### 基礎能力
- ✅ Python 基礎語法
- ✅ 函數與模組
- ✅ 物件導向基礎
- ✅ 檔案與 JSON 處理

### 進階能力
- ✅ 異常處理與除錯
- ✅ 套件管理與虛擬環境
- ✅ Pydantic 資料驗證
- ✅ 專案結構組織

**現在您已經準備好開始《12週AI 提示工程全端修練計畫》了!** 🚀

---

**建議:** 如果某些概念還不夠熟悉,可以:
1. 重複閱讀該部分的註解
2. 實際執行程式碼觀察結果
3. 修改程式碼參數,觀察變化
4. 結合 12 週計畫的實際專案練習
