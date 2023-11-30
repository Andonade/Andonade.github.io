---
title: 用代码统计装潢 GitHub profile
date: 2023-11-30 17:09:24
tags:
    - plaything
    - Python
categories: 
    - 好玩的东西
---

{% note blue fa-check %}
阅读本文需要基本的 Python 语法基础
{% endnote %}

你可能见过如下的 Github profile，你可能也想要一个这样的能够统计自己代码数据的 profile，本文正是为此而生！

<center>
    <img src="https://i.imgs.ovh/2023/11/30/pzgDD.png" width="80%">
</center>

## WakaTime

在拥有这样的 profile 之前，我们需要一个能够统计代码数据的工具，这里我们使用 [WakaTime](https://wakatime.com/)，WakaTime 是一款功能非常强大的代码统计工具，它能够作为插件集成在大多数主流代码编辑器、甚至 Chrome, 终端这些地方来统计你的代码数据。我们首先在官网上注册一个账号，可以看到它有多个订阅计划，免费的个人计划即可保存 14 天的数据，这对于我们来说已经足够了。

然后我们来到 [Settings | Account](https://wakatime.com/settings/account)，找到 **API key**（如下图）并复制，这个 **API key** 非常重要，我们在获取数据的时候需要用到它。

<center>
    <img src="https://i.imgs.ovh/2023/11/30/pzJKA.png" width="90%">
</center>

然后我们可以在自己平常用的编辑器或 IDE 安装 WakaTime 插件，可以在 [plugins](https://wakatime.com/plugins) 查看安装方式，插件在安装后会要求你设置 **API key**，这一部分也就完成了。

## 爬取数据

这里我们使用 Python 的 requests 库来爬取数据，这里就不介绍安装过程了，首先我们查看一下 [WakaTime API Docs](https://wakatime.com/developers)，我们这里要用到的两个接口分别是 [Durations](https://wakatime.com/developers#durations) 和 [stats](https://wakatime.com/developers#stats)，但是在此之前，我们需要先完成鉴权，否则无法完成数据的获取。

### 鉴权

鉴权的过程很简单，你可以参考 [Authentication](https://wakatime.com/developers#authentication) 中的多种实现方式，这里我们使用 HTTP Basic Auth 包含在请求头的方式来完成鉴权。

```python
# example
import requests
import base64
API_Key = 'your api key'
headers = {'Authorization': 'Basic ' + base64.b64encode(API_Key.encode('utf-8')).decode('utf-8')}
response = requests.get('api url', headers=headers)
```

{% notel red fa-triangle-exclamation 注意 %}
这里请注意，***一定！一定！一定！*** 不要使用明文来存储你的 **API key**，这段代码只是一个例子，你应当使用环境变量以及 GitHub 的 Actions Secrets 来存储你的 **API key**，这样才能防止 **API key** 的泄露，相关内容我们之后介绍。
{% endnotel %}

### 获取数据

#### 爬取活动

我们首先爬取日常的代码活动数据，查看 API 文档后得知接口为 `https://wakatime.com/api/v1/users/current/durations` 且必须加入 date 参数，为了得到最近一周的数据，我们需要以前 7 天的日期为参数，如下

```python
# get activity data
import datetime
import json
date = [datetime.date.today() - datetime.timedelta(days=i) for i in range(1, 8)]
for i in range(6, -1, -1):
    res = requests.get('https://wakatime.com/api/v1/api/v1/users/current/durations', params={'date': date[i].strftime('%Y-%m-%d')}, headers=headers)
    data = json.loads(res.text)
    # more code
```

我们在 Postman 中查看接口返回的 json 文件，发现格式如下

```json
{
    "data": [
        {
            "time": <timestamp>,
            "project": <project name>,
            "duration": <seconds>,
            "color": null
        },
        {
            <more>
        }
    ]
}
```

不难发现，数据统计的方式为将一天从早到晚的所有活动按时间顺序排列，不同的 project 分开统计，要获取一天的活动，只需要

```python
active = [False] * 24
activities = data['data']
for activity in activities:
    start = datetime.datetime.fromtimestamp(activity['time'])
    end = datetime.datetime.fromtimestamp(activity['time'] + activity['duration'])
    for i in range(start.hour, end.hour + 1):
        active[i] = True
```

这样就可以得出一天中每个小时是否有活动，然后我们以此为根据画出分布图

```python
filled_char = '█'
empty_char = '░'
hour_str = '|'
for i in range(24):
    hour_str += filled_char if active[i] else empty_char
hour_str += '|'
```

这样就可以得到一天的活动分布图！

#### 爬取其他数据

其他数据使用接口 `https://wakatime.com/api/v1/users/current/stats/last_7_days` 获取，解析数据的方式遇上了相似，这里为了展现比例，使用了进度条来展示数据

```python
def convert_to_progress_bar(percentage: int, length: int = 25) -> str:
    filled_length = int(length * percentage / 100.0)
    empty_length = length - filled_length
    progress_bar = filled_char * filled_length + empty_char * empty_length
    progress_bar += f' {percentage}%'
    return progress_bar
```

数据的爬取就介绍到这里，完整代码可以在 [Andonade](https://github.com/Andonade/Andonade) 中查看，这里不再赘述。

## Github Action

将上面的代码 push 到 GitHub 上后，我们需要使用 GitHub 的 Workflow 来完成自动化的爬取和更新，首先我们需要在项目根目录下新建 `.github/workflows` 文件夹，然后在该文件夹下新建 `wakatime.yml` 文件，这个文件就是我们的 Workflow 文件，我们一步步将其完善。

首先是 Workflow 的名字和触发条件，我们需要定时运行任务，所以使用 `schedule` 来完成

```yaml
name: Wakatime README
on:
  schedule:
    - cron: '0 15 * * *'
```

然后是要完成的任务

```yaml
jobs:
  update-readme:
    name: Update this repo's README with stats from WakaTime
    runs-on: ubuntu-latest
    permissions: write-all
    env:
      TZ: Asia/Shanghai
    steps:
      # more code
```

我们需要运行的服务器有 Python 的环境和 requests 库，所以我们需要安装这些依赖

```yaml
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.10.12
          cache: 'pip'
      - name: Update pip
        run: python -m pip install --upgrade pip
      - name: Install dependencies
        run: pip install -r requirements.txt
```

然后我们再运行脚本来生成 README.md，注意这里要带上我们在 GitHub 上设置的 Actions secrets 且变量名不能出错

```yaml
      - name: Run script
        env:
          WAKATIME_API_KEY: ${{ secrets.WAKATIME_API_KEY }}
        run: python action.py
```

在脚本里，我们这样获取 **API key**

```python
import os
API_Key = os.environ['WAKATIME_API_KEY']
```

最后我们需要将生成的 README.md push 到 GitHub 上，这里我们使用 GitHub 上有人写好的轮子

```yaml
      - uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: Update README.md with the latest stats
          branch: main
          file_pattern: README.md
```

完整的 Workflow 文件同样在 [Andonade](https://github.com/Andonade/Andonade) 中。

最后来看效果，你也可以根据自己的喜好进行自己的定制！

<center>
    <img src="https://i.imgs.ovh/2023/11/30/pDDTX.png" width="90%">
    <img src="https://i.imgs.ovh/2023/11/30/pDTlU.png" width="90%">
</center>