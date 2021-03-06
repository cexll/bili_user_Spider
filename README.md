# 教你用Python爬取哔哩哔哩全站用户信息

## 运行
### 下载
```
git clone https://github.com/cexll/bili_user_Spider.git
```
### 运行环境
- Ubuntu
- Python 3.6
- VSCode
### 依赖
- requests
- pymongo
- mongodb

使用本脚本请先安装好MongoDB,并且安装好库
```
pip install -r requirements.txt
```
### 运行
运行前请进入脚本把cookie和mid改成自己的方便第一时间给数据库增加相应的数据
```
python3 spider.py
```

## 思路
*这是用思维导图画的一个脚本的大致运行情况*
![](img/daotu1.png)
**程序的大致思路是这样,通过mid进入用户主目录,然后获取用户个人信息并保存到数据库,然后获取用户的粉丝数量以及关注数量方便之后计算页数,下一步获取所有的关注用户mid以及粉丝用户mid保存到数据库,然后依次提取数据库mid进入用户主目录循环至结束**

### 运行过程
![](img/1.PNG)

### 数据来源
数据通过`bilibili`官方`api`获取除了获取`个人信息`必须要进入`用户目录`才能成功获取其他都可直接通过`API`获取到数据

## 分析代码
这里就上最主要的代码

运行函数,首选进入用户主页然后获取个人信息到数据库

然后获取粉丝数量以及关注数量

通过`关注数量 / 50` 得到页数, 这里做了如果结果`小于或等于1`那么就直接当1,不然`range(1, 1)`是无法运行的

```python
def run(mid):
    """
    运行函数
    """
    
    # 进入用户主页
    get_space(mid)

    # 获取关注数量和粉丝数量
    f, g = get_myinfo(mid)

    # 获取关注用户信息
    f_g_ps = 50
    f_g_pn = int(g / f_g_ps)+1
    if f_g_pn <= 1:
        get_followers(mid, 1, f_g_ps)
    else:
        for g_pn in range(1, f_g_pn):
            get_followings(mid, g_pn, f_g_ps)
    
    # 获取粉丝用户信息
    f_r_ps = 50
    f_r_pn = int(f / f_r_ps)+1
    print(f_r_pn)
    if f_r_pn <= 1:
        get_followers(mid, 1, f_r_ps)
    else:
        for r_pn in range(1, f_r_pn): 
            get_followers(mid, r_pn, f_r_ps)

    # 循环
    rep_run()
```

## 核心代码

这里`MIN`必须要先初始值,就设置了一个全局变量,进入`rep_run`,`MIN`变量加一,连接`list`数据库,查询数据库所有信息保存到`result`,`max`是从数据库提取出来的信息最大`count`,接下来判断`MIN`是否大于`max`,如果大于则说明数据库数据已经运行完了,直接结束脚本,如果不大于则继续循环
```python
def rep_run():
    """
    当上一个mid所有事情完成后进入此函数进行循环爬取下一个mid
    """
    global MIN
    # 每次运行此函数使MIN加一,不能大于max(数据库count)
    MIN += 1
    collection = db.list
    # 查询数据库所有数据保存到result
    if collection.find({'id': MIN}):
        ran = collection.find({'id': MIN})
        # 查询数据库有多少条
        count = collection.find({}).count()

        for x in ran:
            mid = x.get('mid')

        if MIN > count:
            print('程序即将停止运行,所有信息爬取完成')
            time.sleep(10)
            exit()
        else:
            run(mid)
    else:
        print('数据库没有该数据 id: {}'.format(MIN))

```

## 分析

*2018-9-12 现已有10W数据进行简单数据分析各位看官看看就好*

#### *性别分布情况*

![](/img/xb.png)

​    可以看到,还是有非常多的人不愿意透露性别的(.....)

#### *用户增长情况*

![](/img/zz.png)

这**1970**年都有大哥注册了,真是元老啊.....但是很少,大部分元老包括b站官方站长等也是2009年

    
