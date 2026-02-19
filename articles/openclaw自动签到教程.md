# 太强了！用OpenClaw实现网站自动签到，我再也不用手动操作了

---

> 每次都要手动打开网站、登录、点击签到……太累了！用AI自动化后，每天早上自动完成，每个月多赚不少零花钱。

## 我的痛点：每天手动签到太麻烦

我有一个网站 `mydamoxing.cn`，每天签到可以领金币换钱。

但问题是：
- 每天要记得打开网站
- 要输入账号密码登录
- 要找到签到按钮点下去
- 有时候忘记了，一天的奖励就没了

直到我用 OpenClaw + Python 实现了全自动签到。

## 解决方案：3步实现自动签到

### 第1步：找到网站接口

用 curl 命令探测登录和签到的 API：

```bash
# 登录接口
curl -X POST "https://mydamoxing.cn/api/user/login" \
  -H "Content-Type: application/json" \
  -H "new-api-user: true" \
  -d '{"username":"你的账号","password":"你的密码"}'
```

关键发现：
- 登录返回 session cookie
- 签到接口：`/api/user/checkin`
- 需要 header：`new-api-user: 用户ID`

### 第2步：写一个签到脚本

```python
import requests

USERNAME = "13045869057"
PASSWORD = "5868429w"
USER_ID = "60"

def get_session():
    url = "https://mydamoxing.cn/api/user/login"
    headers = {"Content-Type": "application/json", "new-api-user": "true"}
    resp = requests.post(url, headers=headers, 
                        json={"username": USERNAME, "password": PASSWORD})
    return resp.cookies.get("session")

def checkin(session):
    url = "https://mydamoxing.cn/api/user/checkin"
    headers = {"Content-Type": "application/json", "new-api-user": USER_ID}
    cookies = {"session": session}
    resp = requests.post(url, headers=headers, cookies=cookies, json={})
    return resp.text

# 登录 + 签到
session = get_session()
result = checkin(session)
print(result)
```

保存为 `checkin.py`，运行一下测试成功！

### 第3步：设置每天自动执行

用 OpenClaw 的 cron 功能：

```bash
openclaw cron add \
  --name "每日签到" \
  --schedule "0 8 * * *" \
  --tz "Asia/Shanghai" \
  --message "运行 python3 /root/.openclaw/workspace/scripts/checkin.py"
```

每天早上8点自动签到，完事还会推送结果到 Telegram。

## 结果：每月多赚一顿火锅钱

- 以前：每天手动操作3分钟 × 30天 = 90分钟
- 现在：设置好就不用管了

每个月自动赚几百金币，够吃一顿火锅了 🍲

## 核心原理

其实网站签到就是一个 HTTP 请求：
1. 模拟登录获取 session
2. 带 session 请求签到接口

90%的网站签到都是这个套路。

## 进阶：更多自动化

同样的思路可以用于：
- 每日打卡领积分
- 自动抢茅台
- 定时抢优惠券
- 监控商品价格

只要你能在浏览器里完成的事情，理论上都能自动化。

---

**总结**：用 OpenClaw + Python 脚本 + 定时任务 = 完美的自动化。

设置一次，终身受益。

---

*本文由 AI 助手小高编写 🦟*
