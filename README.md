# pycrawler
## 描述:
> refact project zbb-crawler, and make it easy to package
## Release info:
major.min.build -- build号码偶数为稳定版（对应poi-crawler-crane-crawl-frame分支master_sh），奇数为测试版(对应poi-crawler-crane-crawl-frame分支test)
#### v1.7.0
>   - 对爬虫日志收集做出修改，修改后的字段见https://wiki.sankuai.com/pages/viewpage.action?pageId=916585733
>       + mes_log未做变动
>       + req_log减少字段site
>       + task_log只保留suc，fail，persist
>       + task_validate_log 新增日志，字段为成功task的重试次数和访问类型
>       + 删除原来的appid，更名为jobid，为一轮爬取任务的唯一id，由用户配置，可以通过acaleph获得
>       + 对于每个真正执行的进程，日志会有一个proc自动生成，规则见wiki
>   - 配置文件中appid不再有效，需配置jobid, 不配监控acaleph将无法区分
>   - 配置文件不再提供logfile指定日志名或默认sitename做日志名，使用自动生成的proc作日志名，便于管理
>   - 配置文件中的sitename将仅作为存储db的目录名称，不再具有其他作用
#### v1.6.6
>   - 使jobid的参数优先级高于文件配置
#### v1.6.4
>   - 在爬虫真正开始爬取时更新代理资源
#### v1.6.2
>   - 设置checkpoint最小值
>   - 拉长ResultWriter创建新db的周期，checkpoint的倍数
>   - 优化写数据库的次数
#### v1.6.0
>   - 增加依赖库lxml
#### v1.5.8
>   - 增加校验跳过逻辑，当去重模块完整时跳过恢复步骤，缩短重启爬虫时间
#### v1.5.6
>   - 增加GeneralTaskHandlerA新特性：设置no_store_names=[xxx,xxx],可以使对应名字的链接回调不保存下载信息
#### v1.5.4
>   - 更改默认统计指标
#### v1.5.2
>   - 修复底层库bug，增加对chunked response可能引发异常的捕获
#### v1.5.0
>   - 重构dump去重模块逻辑，简化设计
#### v1.4.8
>   - 增加监控爬虫失败率逻辑，一旦超过指标，爬虫将自动退出.设置方式和老框架一致，在业务taskHandler类中设置属性
>       + warning_fail_count 该属性为半小时内失败上限，超过该值发送报警信息并关闭爬虫，默认值1000
>       + warning_good_per_bad 该属性为半小时内成功请求比失败请求比率，超过该值发送报警信息并关闭爬虫，默认值 0.1
>   - 增加监控日志
#### v1.4.6
>   - 增加检测如果是mac os, 自动添加log_local=true选项，方便开发测试
>   - 修复bug：多个代理配置时获取有误
#### v1.4.5
>   - 增加定时器信号方式安全中断程序（发送中断信号以后，程序会等待当前线程池中task完成，然后写数据库，然后正常退出）
>   - 使用方式：
>>      首先 import signal. 然后在程序的任意位置（比如taskhandler的__init__,比如run.py等）signal.alarm(seconds).
#### v1.4.4
>   - 修改报警发送方式:改为post请求
#### v1.4.2
>   - 修复同步问题bug: 保证dump文件完整性
>   - dump去重模块和边加载初始任务边运行爬虫配置详见v1.4.0
#### v1.4.0
>   - 增加新特性: 边加载初始任务，边启动爬虫
>>  通过程序TaskHandler中设定属性 seed_load_step=xxx 来控制加载多少初始任务数量便启动爬取任务。
>>  框架默认会有一个配置，小技巧是当给出值大于初始化总量时就是像框架之前全部初始化之后才开始爬取
>   - 增加新特性: 定时dump skip_task,seed_task,dedup module(bloomfilter or set)
>>  该特性其实是为支持上述功能所提供的。
>>  + 说明1：框架使用者在提供seed_task方法时应保证每次执行产生task是有序的。如果不能保证，请将seed_load_step设为大于初始化数量的值，一次性加载避免错乱
>>  + 说明2：当设置dedup=False时，只有初始化任务信息会被记录，不会产生dedup模块，skip_task也将不起作用
>>  + 说明3：当设置dedup=True时，用户可以给出dump相关配置。(参数依次为目录，名称，间隔分钟):
    "dump_conf":{
        "dump_dir":"/opt/logs",
        "dump_name":"filter",
        "dump_interval": 5
    }
    dump_interval单位为分钟，为保证一致性问题真正dump间隔时间会取其与checkpoint_interval的最大值，并可能有一定延迟
>>  + 说明4：该特性会产生counter和dedup(如上配置/opt/logs/filter).所以若想重启爬虫请保证这两个文件的完整（重启继续爬仅需删除run.pid文件），若该文件被删除将导致爬虫重复爬取等问题。
#### v1.3.4
>   - 日志req log中proxy字段只记录ip，不记录额外信息
#### v1.3.2
>   - 更新报警通知机制（使用公司提供的系统，不再使用51ping邮件,新版坑爹的不支持附件，而且显示蛋疼）
>   - 配置复用email_conf (为了兼容已在运行的系统)，示例如下：send_type默认值为mail(支持dx,sms,mail,phone)，to_list为mis号，如果填邮件全称会使用前缀
>>  "email_conf": {
        "send_type": "mail,dx",
        "to_list": ["hukai07"]
    },
#### v1.3.1
>   - 增加dump去重模块功能。(解决重启时加载慢的问题，但是不能完全保证精度，损失时间依据为配置参数)
>>  配置runconfig.json示例(参数依次为目录，名称，间隔分钟):
    "dump_conf":{
        "dump_dir":".",
        "dump_name":"filter",
        "dump_interval": 5
    }
#### v1.2.4
>   - task log增加down_count字段（https://wiki.sankuai.com/pages/viewpage.action?pageId=916585733）
>   - nupp代理picker逻辑增加默认配置：默认添加src字段，用于log统计. 默认添加score:60,100字段，如有特殊需求自行修改代理url
#### v1.2.3
>   - 将原来下载异常的报警由邮件转为warning日志，避免无效报警，也方便将指标型报警从爬虫框架中剥离后续处理更灵活
#### old version
>   - log 相关配置
>>  "loglevel": "info",          日志级别
    "log_local": false,          是否打日志到本地运行目录,线上务必不配或者false,特殊需求除外，否则收集不到日志
    "logfile": "amap_idxxx.log", 日志名称
    "appid": "amap_idxxx",       日志中记录的appid
>   - 代理相关配置
>>  "proxy": {
            "type":"nupp", 新代理api获取方式
            "conf":{
                代理api配置选项（根据需求），可以配多个链接
                "proxy_urls":["http://cricetidae.sankuai.com/api/v1/crawler/proxies?params={\"fields\":\"score\",\"limit\":3000,\"timerange\":7200,\"score\":\"80,100\"}"],
                "pp_type": "v2",
                "count":800, 拉取后总使用数量
				"pp_conf": {
				    "hosts":[]
				},
				"refresh_interval":100, 拉取代理间隔
				"update_hosts" : true   是否更新代理，false为替换
			}
        },

## 目标:
- 精简代码，把原先没用到的代码移除
- 梳理代码结构，解决依赖
    > [对于结构的理解](http://pythonguidecn.readthedocs.io/zh/latest/writing/structure.html)
- 建立打包所需的文件

## 目录结构的解释
- README.md 是总体介绍
- requirements.txt 所有项目依赖的python包都应该被测试包括进来
- 目录doc放置项目相关文档
- 目录commons，之前zbb_commons项目，应该整理成内部使用的包？
- app 对外开放的程序入口们
- 其他独立py文件可以被当作工具一样被其他程序使用？
- license Todo：目前还没有
- test 单元测试模块
- setup.py

## 具体改动：
+ 移动zbb_commons进项目代码，并改名为commons（放在一起便于维护，减少外部依赖）
+ 新增setup.py,用于python wheel打包
    > - 增加clean command 操作
+ pid 文件管理
    > - 统一存储为run.pid, 不再提供用户传参模式，避免误操作
    > - 只有在正常结束时才删除run.pid文件，异常退出不再删除
+ 工具类程序改变
    > - 更改app.convert_json_result， 使用方式不变，适用新的存储结构
    > - 删除app.harvester, 之后hive存储交给kafka那边的程序处理
+ wheel包
    - [下载地址](http://code.dianpingoa.com/yuhan.yang/poi-crawler-python-crane-demo/tree/master/dist)
### 存储结构变化
> - 根据新标准改变字段存储和压缩方式，store( gzip( json ( {..., b64(content), ... }) ) )
> - 根据新标准改变存储结果目录结构， ${work}/data/${host}/${type}/some.db|some.closed.tag
> - 之前pickle之后数据拼在一起存储，改为每条记录独立存储
### 部署流程影响的改变
> - 所有的配置参数调整为run.config里读取，当然目前还是兼容老版本传参方式的（方便调试和兼容性）
> - 传参数方式的参数名可以用-或_分割，如checkpoint-interval，但是在config文件里的应该用_分割，所以是checkpoint_interval
> -

### 关于参数字段的变化
+ pid_file: 不再支持，统一用文件run.pid
+ data_root:不再有result目录
+ sitename: 新增字段，指定爬取网站名称
+ logfile: 不再生成到指定地址，只能指定名字，统一到线上环境规定的目录
+ log_local:新增字段，true or false,为true时会存放到当前运行目录，仅用于调试
+ 在downloader_conf中增加seesion开关字段，eg. "downloader_conf": {"use_session": false}, 默认开启，开启的好处是不用反复握手提高效率，必要时共享token等，关闭可解决内存增长等问题
+ downloader_conf中threads字段2倍（由10缩减为2）为active_task_max，active_task_max也可在爬虫程序中指定

+ 新增代理类型nupp，原upp目前不受影响，等待服务关停后将不再可用
    > upp配置例子如下， "proxy": {"type":"upp", "conf":{"proxy_urls":["http://poi-crawler-export-service01.gq:4080/invoke.json?url=http://service.dianping.com/poiService/HttpProxyExportService_1.0.0&method=queryCrawlerProxyInfo&parameterTypes=java.lang.String&parameters=5dfe6b30fac7788c&parameterTypes=long&parameters=900&parameterTypes=int&parameters=80&parameterTypes=int&parameters=800&parameterTypes=java.lang.String&parameters=SITEDIGGER"],"pp_type": "v2",}}
    > nupp配置例子如下, "proxy": {"type":"nupp","conf":{"proxy_urls":["http://cricetidae.sankuai.com/api/v1/crawler/proxies?params={\"fields\":\"addtime\",\"limit\":1000,\"timerange\":7200}"],"pp_type": "v2",}}
+ 新增appid字段，给定则使用该值作为日志appid字段，没有则用机器名

### 日志新格式
+ 使用新的日志记录方式, 参考https://wiki.sankuai.com/pages/viewpage.action?pageId=916585733
+ 日志结构变为三个目录，参见上方wiki

### 新增特性
+ 增加爬虫结束爬取之后运行自定义操作
    > 具体方式为在爬虫逻辑里调用runner的bind_finish_operation方法绑定自定义函数和参数
    > 实例：def do_somthing(self, a, b):
       # do something
        log.info('%d, %d', a, b)

    > def initialize(self, runner):
        runner.bind_task_function('get_menu', self.get_menu)
        runner.initialize_tasks(self.seed_tasks_id_file())
        runner.bind_finish_operation(self.do_somthing, 1, 2)
