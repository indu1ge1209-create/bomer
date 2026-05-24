# buy_pig_plan | 买猪计划
> #### 前言
> 我把项目的思路放在最后了,感兴趣的话直接拉到下面[看思路](https://github.com/aqiongbei/buy_pig_plan#思路分享).


- 下载
```sh
git clone https://github.com/aqiongbei/bomer.git
```

- 安装依赖

```sh
# 由于安装puppeteer时候需要下载chromium，而下载chromium需要外网，所以建议对puppeteer使用国内镜像下载，加上下面这句
npm config set puppeteer_download_host=https://npm.taobao.org/mirrors
npm install
```

- 配置
**使用前请先修改配置文件**，配置文件共有两个，都放在`/config`目录下。
    - `default.json`: 默认配置文件，所有可配置的内容都在这里列举了，`npm start`使用的就是这个配置文件
    - `debug.json`: debug模式的配置文件，在debug模式这里的配置会覆盖`default.json`中的配置

各个配置字段说明说明如下：
```js
{
    "target": {                    
        "phone": "",                
        "name": "",                
        "email": "",                
        "address": "",              
        "comment": "",             
    },
    "attack": {                    
        "times": 6,                 
        "time": "0 2 * * * *",      
        "web_type": "baidu_lxb",    
        "type_type": "call",        
        "interval": 600000,         
    },
    "chromium": {                   
        "slowMo": 100,              
        "timeout": 30000,          
        "devtools": false,         
    }
}
```

- 启动项目
```sh
# production模式，非立即执行，是定时执行的
npm start
# debug模式
npm run debug
```

- 网站源切换


#### 思路分享
整个项目的思路是这样的:
##### step 0 网站收集
从[百度离线宝(百度的一个营销平台,内含电话回拨系统,简单说就是可以通过网站打电话)](https://lxb.baidu.com/lxb/index.html)的一个不知道[为何存在的页面](http://lxbjs.baidu.com/cb/url/show?f=56&id=1)遍历id获取使用其服务的客户网站.然后打开他们客服的网站,根据下面的特征我们可以大致分成三种:
- 有电话回拨功能的(下图右边中部),这中网站我称之为`call`类型
![有电话回拨的](./images/call.png)
- 有留言功能的(下图左下角),这种网站我称之为`comment`类型
![有留言功能的](./images/comment.png)
- 没有以上两种任意一种功能的

前两种是我需要的网站类型,我会把他们保存在`json`文件中,供后面的流程使用.
这个步骤对应的脚本是`/utils/get_lxb_sources.js`,使用方法是
更改`/utils/get_lxb_sources.js`里面的
```js
// start_id end_id
await start(114000, 114001); // 更改这里的传参,第一个参数为开始id,第二个为结束id,这里建议start_id和end_id相差10000最好
```
然后在项目根目录跑一下这个脚本
```sh
node /utils/get_lxb_sources.js
```
脚本跑完之后会在`sources`目录的不同类型下产生形如:`baidu_shangqiao_50000.json`的文件.

接下来我们开始使用这些收集到的网站.

##### step 1 
step 0我们已经收集到了一些可用的网站,现在我们要使用这些网站了.
在主程序运行的时候,针对不同类型的网站我会采用不同的处理逻辑,但是大致的流程都一样,就拿`call`类型的网站:
- 在chromium中打开这个页面
- 等页面加载加载完成之后在页面内`.lxb-cb-input`对应的输入框中输入目标的手机号,然后点击`.lxb-cb-input-btn`元素触发电话回拨
- 1s中之后收集上个操作是否成功的反馈,方便后面统计使用

对于`comment`的网站,操作类似:
- 在chromium中打开这个页面
- 等页面加载加载完成之后在页面内找到一些必填的信息,把目标的信息填写进去,然后提交
- 1s中之后收集上个操作是否成功的反馈,方便后面统计使用


```sh
.
├── app.js
├── config
│   ├── debug.json
│   └── default.json
├── flow
│   ├── call
│   │   ├── baidu_lxb.js
│   │   └── baidu_shangqiao.js
│   ├── comment
│   │   └── baidu_shangqiao.js
│   └── flow.js
├── LICENSE
├── package.json
├── package-lock.json
├── README.md
├── sources
│   ├── call
│   │   └── baidu_lxb.json
│   ├── comment
│   │   └── baidu_shangqiao.json
│   └── sms
└── utils
    ├── get_lxb_sources.js
    └── util.js
```


