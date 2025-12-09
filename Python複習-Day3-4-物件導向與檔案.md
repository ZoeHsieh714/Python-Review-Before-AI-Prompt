# 🐍 Python 快速複習 Day 3-4: 物件導向與檔案處理

> **學習時間:** Day 3 (40分鐘) + Day 4 (40分鐘)  
> **目標:** 掌握類別定義、@dataclass、檔案處理、JSON 和環境變數管理

---

## 📅 Day 3: 物件導向基礎 (40分鐘)

### 學習重點
- [ ] 類別定義
- [ ] 初始化方法 __init__
- [ ] 實例方法與屬性
- [ ] 簡單的類別設計

### 實作練習

```python
# ============================================================
# 1. 基本類別定義
# ============================================================

class AIAssistant:
    """
    AI 助手類別
    
    類別 (Class) 是物件的藍圖,定義了物件的屬性和方法
    """
    
    def __init__(self, model: str = "gpt-4"):
        """
        初始化方法 (Constructor)
        
        __init__ 是特殊方法,在建立物件時自動呼叫
        self 代表物件本身,類似其他語言的 this
        
        Args:
            model: 模型名稱,預設為 gpt-4
        """
        # 實例屬性 (Instance Attributes) - 每個物件獨立擁有
        self.model = model                    # 儲存模型名稱
        self.conversation_history = []        # 儲存對話歷史
    
    def chat(self, message: str) -> str:
        """
        對話方法 (Instance Method)
        
        實例方法的第一個參數必須是 self
        可以存取和修改實例屬性
        
        Args:
            message: 使用者訊息
            
        Returns:
            AI 的回應
        """
        # 將訊息加入歷史記錄
        self.conversation_history.append(message)
        return f"AI 回應: {message}"
    
    def get_history(self) -> list:
        """取得歷史記錄"""
        return self.conversation_history
    
    def clear_history(self):
        """清除歷史記錄"""
        self.conversation_history = []

# 使用類別 - 建立物件 (Object/Instance)
assistant = AIAssistant(model="gpt-4")  # 呼叫 __init__
response = assistant.chat("Hello")       # 呼叫 chat 方法
history = assistant.get_history()        # 取得歷史

# 建立多個獨立的物件
assistant1 = AIAssistant(model="gpt-4")
assistant2 = AIAssistant(model="gpt-3.5-turbo")
# assistant1 和 assistant2 有各自獨立的 conversation_history

# 💡 為什麼使用類別:
# 1. 組織相關的資料和功能
# 2. 封裝 (Encapsulation) - 隱藏內部實作細節
# 3. 重複使用 - 可以建立多個物件


# ============================================================
# 2. 類別屬性 vs 實例屬性
# ============================================================

class ChatConfig:
    # 類別屬性 (Class Attribute) - 所有實例共享
    # 定義在 __init__ 外面
    default_model = "gpt-4"
    total_instances = 0  # 追蹤建立了多少個物件
    
    def __init__(self, temperature: float):
        # 實例屬性 (Instance Attribute) - 每個實例獨立
        # 定義在 __init__ 裡面,使用 self.
        self.temperature = temperature
        
        # 修改類別屬性
        ChatConfig.total_instances += 1

# 存取類別屬性
print(ChatConfig.default_model)  # "gpt-4" - 不需要建立物件

# 建立實例
config1 = ChatConfig(temperature=0.7)
config2 = ChatConfig(temperature=0.5)

# 實例屬性 - 各自獨立
print(config1.temperature)  # 0.7
print(config2.temperature)  # 0.5

# 類別屬性 - 所有實例共享
print(config1.default_model)  # "gpt-4"
print(config2.default_model)  # "gpt-4"
print(ChatConfig.total_instances)  # 2

# 💡 使用時機:
# - 類別屬性: 所有實例共享的常數或設定
# - 實例屬性: 每個物件獨立的資料


# ============================================================
# 3. 屬性裝飾器 @property
# ============================================================

class TokenCounter:
    def __init__(self):
        # 私有屬性 (慣例用 _ 開頭,表示不應直接存取)
        self._total_tokens = 0
    
    @property
    def total_tokens(self):
        """
        @property 裝飾器讓方法可以像屬性一樣存取
        
        好處:
        1. 可以加入驗證邏輯
        2. 可以計算衍生值
        3. 可以設為只讀
        """
        return self._total_tokens
    
    def add_tokens(self, count: int):
        """增加 token 數量"""
        if count < 0:
            raise ValueError("Token 數量不能為負數")
        self._total_tokens += count

# 使用
counter = TokenCounter()
counter.add_tokens(100)

# 使用 @property - 像存取屬性一樣,但實際上是呼叫方法
print(counter.total_tokens)  # 100 - 不需要加 ()

# 無法直接修改 (因為沒有定義 setter)
# counter.total_tokens = 200  # ❌ 會報錯

# 💡 實際應用: 建立只讀屬性,防止外部直接修改內部狀態
# 例如: API 呼叫的總 token 數,只能透過 add_tokens() 增加


# ============================================================
# 4. 實用的資料類別 (Python 3.7+)
# ============================================================

from dataclasses import dataclass

@dataclass
class Message:
    """
    @dataclass 自動生成:
    - __init__ 方法
    - __repr__ 方法 (print 時的顯示)
    - __eq__ 方法 (比較相等)
    
    非常適合用來定義資料結構
    """
    role: str                    # 必填欄位
    content: str                 # 必填欄位
    timestamp: str = None        # 選填欄位 (有預設值)

# 建立物件 - 不需要手動寫 __init__
msg = Message(role="user", content="Hello")

# 自動生成的 __repr__ - 顯示清楚
print(msg)  
# 輸出: Message(role='user', content='Hello', timestamp=None)

# 存取屬性
print(msg.role)      # "user"
print(msg.content)   # "Hello"

# 比較物件
msg1 = Message(role="user", content="Hello")
msg2 = Message(role="user", content="Hello")
print(msg1 == msg2)  # True - 內容相同就相等

# 💡 使用時機:
# - 定義 API 請求/回應的資料結構
# - 儲存配置資訊
# - 任何主要用來儲存資料的類別

# 傳統寫法 vs @dataclass 對比:
# 
# 傳統寫法:
# class Message:
#     def __init__(self, role, content, timestamp=None):
#         self.role = role
#         self.content = content
#         self.timestamp = timestamp
#     
#     def __repr__(self):
#         return f"Message(role={self.role}, ...)"
#     
#     def __eq__(self, other):
#         return (self.role == other.role and ...)
#
# @dataclass 寫法: 只需要 3 行!


# ============================================================
# 實際應用範例: AI 對話管理器
# ============================================================

@dataclass
class ChatMessage:
    """對話訊息"""
    role: str        # "system", "user", "assistant"
    content: str     # 訊息內容

class ConversationManager:
    """對話管理器 - 整合前面學到的概念"""
    
    # 類別屬性 - 支援的角色
    VALID_ROLES = ["system", "user", "assistant"]
    
    def __init__(self, system_prompt: str = "你是助手"):
        # 實例屬性
        self._messages = []  # 私有屬性
        self._token_count = 0
        
        # 初始化時加入系統提示
        self.add_message("system", system_prompt)
    
    def add_message(self, role: str, content: str):
        """新增訊息"""
        if role not in self.VALID_ROLES:
            raise ValueError(f"角色必須是 {self.VALID_ROLES} 之一")
        
        msg = ChatMessage(role=role, content=content)
        self._messages.append(msg)
        self._token_count += len(content.split())  # 簡單估算
    
    @property
    def messages(self):
        """取得所有訊息 (只讀)"""
        return self._messages.copy()  # 返回副本,防止外部修改
    
    @property
    def token_count(self):
        """取得 token 總數 (只讀)"""
        return self._token_count
    
    def clear(self):
        """清除所有訊息"""
        self._messages = []
        self._token_count = 0

# 使用範例
manager = ConversationManager(system_prompt="你是專業的程式設計助手")
manager.add_message("user", "如何學習 Python?")
manager.add_message("assistant", "建議從基礎語法開始...")

print(f"訊息數量: {len(manager.messages)}")
print(f"Token 總數: {manager.token_count}")
```

### 💡 Day 3 核心概念總結

1. **類別 (Class)**: 物件的藍圖,組織相關的資料和功能
2. **__init__**: 初始化方法,建立物件時自動呼叫
3. **self**: 代表物件本身,存取實例屬性和方法
4. **類別屬性 vs 實例屬性**: 共享 vs 獨立
5. **@property**: 將方法變成只讀屬性,加入驗證邏輯
6. **@dataclass**: 快速定義資料結構,自動生成常用方法

### 檢查點
- [ ] 能定義簡單的類別
- [ ] 理解 __init__ 的作用
- [ ] 知道如何使用 @dataclass
- [ ] 能區分類別屬性和實例屬性

---

## 📅 Day 4: 檔案處理與 JSON (40分鐘)

### 學習重點
- [ ] 檔案讀寫
- [ ] JSON 處理
- [ ] 環境變數管理
- [ ] 路徑操作

### 實作練習

```python
# ============================================================
# 1. 檔案讀寫
# ============================================================

# with 語句 (Context Manager) - 自動處理檔案關閉
# 即使發生錯誤,也會確保檔案被正確關閉

# 寫入檔案
with open("output.txt", "w", encoding="utf-8") as f:
    # "w" = write mode (寫入模式,會覆蓋原有內容)
    # encoding="utf-8" 確保中文正確顯示
    # f 是檔案物件
    f.write("Hello, World!\n")  # \n 是換行符號
    f.write("第二行\n")
# 離開 with 區塊後,檔案自動關閉

# 讀取整個檔案
with open("output.txt", "r", encoding="utf-8") as f:
    # "r" = read mode (讀取模式)
    content = f.read()  # 讀取全部內容為一個字串
    print(content)

# 逐行讀取 (適合大檔案,節省記憶體)
with open("output.txt", "r", encoding="utf-8") as f:
    for line in f:  # f 是可迭代物件,每次迴圈讀一行
        print(line.strip())  # strip() 移除行尾的換行符號

# 其他檔案模式:
# "a" = append mode (附加模式,不會覆蓋,在檔案末尾新增)
# "r+" = read and write (讀寫模式)
# "b" = binary mode (二進位模式,如 "rb", "wb")

# 💡 為什麼使用 with:
# 不用 with 的寫法:
# f = open("file.txt", "r")
# content = f.read()
# f.close()  # 容易忘記關閉,或發生錯誤時無法關閉
#
# 用 with 的寫法:
# with open("file.txt", "r") as f:
#     content = f.read()
# # 自動關閉,更安全!


# ============================================================
# 2. JSON 處理 (API 回應常用)
# ============================================================

import json

# Python 字典 -> JSON 字串 (序列化 Serialization)
data = {
    "name": "張三",
    "age": 25,
    "skills": ["Python", "AI"]
}

json_string = json.dumps(
    data, 
    ensure_ascii=False,  # 保留中文,不轉成 \uxxxx
    indent=2             # 縮排 2 格,讓 JSON 更易讀
)
print(json_string)
# 輸出:
# {
#   "name": "張三",
#   "age": 25,
#   "skills": [
#     "Python",
#     "AI"
#   ]
# }

# JSON 字串 -> Python 字典 (反序列化 Deserialization)
parsed_data = json.loads(json_string)
print(parsed_data["name"])  # "張三"
print(type(parsed_data))    # <class 'dict'>

# 寫入 JSON 檔案
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
    # dump() 直接寫入檔案
    # dumps() 返回字串 (s = string)

# 讀取 JSON 檔案
with open("data.json", "r", encoding="utf-8") as f:
    loaded_data = json.load(f)  # load() 從檔案讀取
    # loads() 從字串解析 (s = string)

# 💡 JSON vs Python 型別對應:
# JSON          Python
# object    ->  dict
# array     ->  list
# string    ->  str
# number    ->  int/float
# true      ->  True
# false     ->  False
# null      ->  None

# 💡 實際應用: OpenAI API 回應
# response_json = response.choices[0].message.content
# data = json.loads(response_json)  # 將 JSON 字串轉為 dict


# ============================================================
# 3. 環境變數管理 (API Key 安全存儲)
# ============================================================

import os
from dotenv import load_dotenv

# 載入 .env 檔案
# .env 檔案內容範例:
# OPENAI_API_KEY=sk-xxxxxxxxxxxxx
# DATABASE_URL=postgresql://localhost/mydb
load_dotenv()  # 讀取專案根目錄的 .env 檔案

# 讀取環境變數
api_key = os.getenv("OPENAI_API_KEY")
# os.getenv(key, default) - 如果環境變數不存在,返回 default

# 檢查是否存在
if not api_key:
    raise ValueError("請設定 OPENAI_API_KEY 環境變數")

# 其他環境變數操作:
# os.environ["KEY"] = "value"  # 設定環境變數
# os.environ.get("KEY")        # 取得環境變數 (同 getenv)
# "KEY" in os.environ          # 檢查是否存在

# 💡 為什麼使用 .env:
# 1. 安全性 - API Key 不會被提交到 Git
# 2. 方便性 - 不同環境 (開發/測試/生產) 使用不同設定
# 3. 最佳實踐 - 敏感資訊不應寫死在程式碼中

# .gitignore 應該包含:
# .env
# *.env


# ============================================================
# 4. 路徑操作 (推薦使用 pathlib)
# ============================================================

from pathlib import Path

# 建立路徑物件
# __file__ 是當前檔案的路徑
project_dir = Path(__file__).parent  # 取得父目錄
# 或者使用當前工作目錄
project_dir = Path.cwd()  # current working directory

# 路徑拼接 - 使用 / 運算子 (跨平台相容)
data_dir = project_dir / "data"           # project_dir/data
config_file = data_dir / "config.json"    # project_dir/data/config.json

# 傳統寫法 (不推薦):
# data_dir = os.path.join(project_dir, "data")  # 較繁瑣

# 建立目錄
data_dir.mkdir(exist_ok=True)
# exist_ok=True - 如果目錄已存在,不會報錯
# parents=True - 同時建立父目錄 (如 mkdir -p)

# 檢查檔案/目錄是否存在
if config_file.exists():
    print("Config file exists")

if config_file.is_file():
    print("這是檔案")

if data_dir.is_dir():
    print("這是目錄")

# 列出目錄中的檔案
for file in data_dir.glob("*.json"):
    # glob() 支援萬用字元
    # *.json - 所有 .json 檔案
    # **/*.py - 所有子目錄中的 .py 檔案
    print(file.name)       # 檔案名稱 (不含路徑)
    print(file.stem)       # 檔案名稱 (不含副檔名)
    print(file.suffix)     # 副檔名 (.json)
    print(file.absolute()) # 絕對路徑

# 讀寫檔案 (pathlib 整合)
config_file.write_text('{"key": "value"}', encoding="utf-8")
content = config_file.read_text(encoding="utf-8")

# 💡 pathlib vs os.path:
# pathlib (推薦):
# - 物件導向,更直觀
# - 使用 / 拼接路徑
# - 跨平台相容
#
# os.path (舊式):
# - 函數式,較繁瑣
# - 使用 os.path.join()
# - 需要記住很多函數名稱
```

### 💡 Day 4 核心概念總結

1. **with 語句**: 自動管理資源,確保檔案正確關閉
2. **json.dumps/loads**: 字串序列化/反序列化
3. **json.dump/load**: 檔案序列化/反序列化
4. **python-dotenv**: 安全管理 API Key 等敏感資訊
5. **pathlib.Path**: 現代化的路徑操作,使用 / 拼接路徑
6. **encoding="utf-8"**: 處理中文時必須指定編碼

### 檢查點
- [ ] 能使用 with 語句處理檔案
- [ ] 熟悉 JSON 的序列化與反序列化
- [ ] 知道如何使用 .env 管理 API Key
- [ ] 能使用 pathlib 操作路徑

---

## 🎯 Day 3-4 總複習

### 必須掌握的概念
1. **類別定義** - class, __init__, self
2. **@dataclass** - 快速定義資料結構
3. **@property** - 只讀屬性
4. **with 語句** - 自動資源管理
5. **JSON 處理** - dumps/loads, dump/load
6. **環境變數** - python-dotenv, os.getenv()
7. **pathlib** - 現代化路徑操作

### 實戰練習建議

```python
# 練習: 建立配置管理器
from dataclasses import dataclass
from pathlib import Path
import json
from dotenv import load_dotenv
import os

@dataclass
class APIConfig:
    """API 配置"""
    api_key: str
    model: str = "gpt-4"
    temperature: float = 0.7

class ConfigManager:
    """配置管理器 - 整合 Day 3-4 知識"""
    
    def __init__(self, config_dir: str = "config"):
        self.config_dir = Path(config_dir)
        self.config_dir.mkdir(exist_ok=True)
        load_dotenv()
    
    def save_config(self, config: APIConfig):
        """儲存配置為 JSON"""
        file_path = self.config_dir / "api_config.json"
        with open(file_path, "w", encoding="utf-8") as f:
            json.dump(config.__dict__, f, ensure_ascii=False, indent=2)
    
    @property
    def api_key(self) -> str:
        """從環境變數取得 API Key"""
        key = os.getenv("OPENAI_API_KEY")
        if not key:
            raise ValueError("請設定 OPENAI_API_KEY")
        return key

# 使用
manager = ConfigManager()
config = APIConfig(api_key=manager.api_key, temperature=0.5)
manager.save_config(config)
```

**準備好了嗎?** 繼續前往 Day 5-7! 🚀
