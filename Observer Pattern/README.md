# Observer Pattern

Observer is a behavioral design pattern that allows some objects to notify other objects about changes in their state.

當多個 Class 都需要接收同一種資料的變化時，就適合使用 Observer Pattern

此篇做法以此[教學範例](https://www.youtube.com/watch?v=oNalXg67XEE&ab_channel=ArjanCodes)為主([範例原始碼](https://github.com/ArjanCodes/betterpython/tree/main/4%20-%20observer%20pattern))
並參考其他文章概念輔助說明

而各模式都可能因情境而有稍微變化以及優缺點，因此需看狀況取捨

如有需要其他範例在最底下有相關連結

---

- [Observer Pattern](#observer-pattern)
  - [以註冊新帳號為例](#以註冊新帳號為例)
    - [原版流程](#原版流程)
    - [Observer Pattern](#observer-pattern-1)
      - [建立event.py](#建立eventpy)
      - [建立listener](#建立listener)
      - [初始化連動關係](#初始化連動關係)
      - [呼叫對應主題](#呼叫對應主題)

---

## 以註冊新帳號為例
### 原版流程
```python
# api/user.py

def register_new_user(name: str, password: str, email: str):
    # create an entry in the database
    user = create_user(name, password, email)

    # post a Slack message to sales department
    post_slack_message("sales",
                       f"{user.name} has registered with email address {user.email}. Please spam this person incessantly.")

    # send a welcome email
    send_email(user.name, user.email,
               "Welcome",
               f"Thanks for registering, {user.name}!\nRegards, The DevNotes team")

    # write server log
    log(f"User registered with email address {user.email}")

```

如需要修改`register_new_user`的slack, email, log的訊息<br>
到`api/user.py`就可以看到所有slack, email, log並修改訊息內容，看似方便<br>
但對於user來說，不應該負責訊息內容

且如要停用slack、換其他通訊軟體、修改email語句格式等<br>
就需跨檔案找所有用到的地方

### Observer Pattern

#### 建立event.py

```python
# api_v2\event.py
subscribers = dict()

# 建立依賴關係
# 如 subscribe["user_registered"] = ["發送slack訊息", "寄email", "log紀錄"]
def subscribe(event_type: str, fn):
    if not event_type in subscribers:
        subscribers[event_type] = []
    subscribers[event_type].append(fn)

# 啟動事件，執行所有關係動作
def post_event(event_type: str, data):
    if not event_type in subscribers:
        return
    for fn in subscribers[event_type]:
        fn(data)

```

#### 建立listener

```python
# api_v2\slack_listener.py
from lib.slack import post_slack_message
from .event import subscribe

# 定義各種動作的訊息
def handle_user_registered_event(user):
    post_slack_message("sales",
                       f"{user.name} has registered with email address {user.email}. Please spam this person incessantly.")


def handle_user_upgrade_plan_event(user):
    post_slack_message("sales",
                       f"{user.name} has upgraded their plan.")

# 與對應主題綁定
def setup_slack_event_handlers():
    subscribe("user_registered", handle_user_registered_event)
    subscribe("user_upgrade_plan", handle_user_upgrade_plan_event)
```

email, log 也同樣定義各動作與綁定主題

#### 初始化連動關係

```python
# 1_observer_after.py

setup_slack_event_handlers()
setup_log_event_handlers()
setup_email_event_handlers()

# register a new user
register_new_user("Arjan", "BestPasswordEva", "hi@arjanegges.com")

```

#### 呼叫對應主題

```python
# api_v2\user.py

def register_new_user(name: str, password: str, email: str):
    # create an entry in the database
    user = create_user(name, password, email)

    # post an event
    # 後續就會執行與user_registered綁定連動的slack, email, log動作
    post_event("user_registered", user)

```

這時要替換slack訊息內容或email格式等，就往負責的listener尋找<br>
如需暫停使用slack也只要將初始化連動關係那一行註解即可

---

更多範例與說明可參考
1. [觀察者模式 (Observer Pattern)](https://notfalse.net/10/observer-pattern)
2. [Design Pattern | 只要你想知道，我就告訴你 - 觀察者模式（ Observer Pattern ） feat. TypeScript](https://medium.com/enjoy-life-enjoy-coding/design-pattern-%E5%8F%AA%E8%A6%81%E4%BD%A0%E6%83%B3%E7%9F%A5%E9%81%93-%E6%88%91%E5%B0%B1%E5%91%8A%E8%A8%B4%E4%BD%A0-%E8%A7%80%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F-observer-pattern-feat-typescript-8c15dcb21622)
3. [Observer in Python](https://refactoring.guru/design-patterns/observer/python/example)