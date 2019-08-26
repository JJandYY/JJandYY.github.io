---
layout: post
title: "pyinstaller打包项目为.exe"
data: 2019-7-26 20:50:00 +0800
author: "YY"
tags:
    - Django
---

#### 1.安装pyinstaller

```python
pip install pyinstaller
```

#### 2.制作项目的.spec文件

  进入django项目所在路径，运行

```python
pyi-makespec -D manage.py
```

   在路径下，会生成生成一个.spec文件

#### 3.以文本的方式打开.spec文件

spec文件格式如下。具体spec的使用，可以查看官网
https://pyinstaller.readthedocs.io/en/stable/spec-files.html

<img src="/img/timelogger/1-pyinstaller生成spec.PNG" class="img-responsive">

#### 4.不修改.spec文件，直接运行以下语句

```python
pyinstaller manage.spec
```

- 现在能够打包成功，但是在manage.exe所在路径下，在cmd中运行manage.exe runserver，会发现以下错误：ImportError: No module named admin



- 这种错误的原因是 django.contrib.admin在django项目中是隐式导入的，所以pyinstaller打包时，并不能识别这种库或者模块，导致打包出来的.exe中并不包括这样隐式导入的库。
- 如果碰到这样的错误，只需要将这个库添加到.spec文件中的hiddenimports中即可。在接下来打包django项目缺少很多这样的隐式库，所以我.spec文件中一并修改了，修改如下：(如果缺少什么，直接在hiddenimports中加就可以了)

<img src="/img/timelogger/2-pyinstaller生成spec.PNG" class="img-responsive">

注：这里有个坑，特别需要关注

- 如果打包好的.exe运行后，报错ImportError: No module named apps，当你在hiddenimports中加入了'django.contrib.admin.apps'，结果还是报同样的错误。这是因为添加的apps模块不完整，要在hiddenimports中加入以下全部的apps模块：(这些apps在django项目中的settings.py文件中可以全部找到，照着添加就可以了，另处还有context_processors模块，middleware模块也需要注意)

- 'django.contrib.admin.apps','django.contrib.auth.apps','django.contrib.contenttypes.apps',
- 'django.contrib.sessions.apps', 'django.contrib.messages.apps', 'django.contrib.staticfiles.apps',

#### 5.以上hiddenimports弄好后，运行后会出现以下的错误

TemplateDoesNotExist  这个是因为没有找到templates文件.

可以根据错误提示将templates文件添加至对应的路径下，刷新即可。其中front是我工程下一个放所有前端东西的文件，templates是用来放html的一个文件夹。(所以具体的添加要根据错误提示是在哪里找不到就添加至哪里)

#### 6.在第五步后，可以发现页面已经出来，但是发现页面没有css和js了

这是因为Pyinstaller 能找到templates(html files文件)，但不能找到css和js文件。
我的具体操作是在django项目的settirngs.py文件中加入

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'front')
```

- 
  其中front是我的文件夹，static_root是我在front下创建的一个空子文件，用来收集工程中所有的静态文件。

- 在django项目路径下执行manage.py collectstatic会自动地将STATICFILES_DIRS列出的目录以及各个App下的static子目录的所有文件复制到STATIC_ROOT。因为复制过程可能会覆盖掉原来的文件，所以，一定不能把我们辛苦做出来静态文件放这边！
- 然后来到urls.py文件下，加入下面的一句话，加入的同时要导入static库

```python
from django.conf.urls import static

urlpatterns += static.static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

这句话的意思就是将STATIC_ROOT目录的静态文件复制一份到网页 STATIC_URL路径下

最后我们还需要将STATIC_ROOT中的静态文件打包到.exe中。这一步是在.spec文件中的datas中加入下面一个元组

```python
datas=[(r'F:\work\master\ruidun_system\ruidun_system\front', r'front'), (r'F:\work\master\ruidun_system\ruidun_system\media', r'media')]
```

考虑到第5步，再这里我也直接将templates文件打包到了对应的文件。所以第五步就不用自己再复制templates文件到指定的文件夹了。
最后.spec文件看起来如下：

<img src="/img/timelogger/3-pyinstaller生成spec.PNG" class="img-responsive">

一切准备好后，执行下面语句就OK

```python
pyinstaller manage.spec
```

------

**如果项目中有使用APScheduler定时任务，打包时请做以下更改：**

未更改前：

```python
# 调度方法为 sendledarea，触发器选择 interval(间隔性)，间隔时长为 60 秒
from apscheduler.schedulers.background import BackgroundScheduler

# 创建后台执行的 schedulers
scheduler = BackgroundScheduler()

scheduler.add_job(self.sendledarea, 'interval', seconds=60, args=(request, ledprogramme_id), id=pk)

# 启动调度任务
scheduler.start()
```

更改后：

```python
# 为打包成.exe需要修改，不然无法使用定时任务
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.triggers.interval import IntervalTrigger

# 创建后台执行的 schedulers
scheduler = BackgroundScheduler()

# 修改代码
interval = IntervalTrigger(seconds=60)
scheduler.add_job(self.sendledarea, interval, seconds=60, args=(request, ledprogramme_id), id=pk)

# 启动调度任务
scheduler.start()
```

