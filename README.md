# XWDE-BoardSync

## 1 简介

同步工具主要是从信源库向`（新、老）两个版本`的采集库同步信源。可以按照`业务businessID、通道mediaID、信蔟sourceclusterID、信源boardID、信源列表boardList文件`五种方式进行选择，并可以选择`time_create信源创建时间`于某时间段以后的信源进行同步。

分为`本地运行版`和`API / request版`。本地运行版是可以直接运行jar包并挂载筛选条件的独立程序(基于Mybatis)。API/request版分为API端和request端：API端采用SSM框架，支持权限校验；request端在运行jar包时挂载筛选条件。一共6个jar包。

本地运行版和服务请求版，均需要在运行前` 配置好(application.properties或jdbc*.properties) `的数据库连接配置文件。

支持分页同步、记录并锁定批量同步的时间、异常情况分类统计和输入参数容错。


## 2 开发环境说明

* **Jdk版本 1.7**

* **Mysql版本5**

* Maven版本 4.0.0（Maven-compiler版本1.9、单机运行版Maven-shade版本2.4.1、API版 springboot版本 2.0.1.RELEASE）

* Mybatis版本3.2.6


## 3 使用说明

工程目录中一共有四个文件夹。

hebing_mybatisbase文件夹为本地运行版。带OLD标志的是同步到老采集库时需要使用的程序，不带OLD的程序是同步到新采集库时需要使用的程序。这两个文件夹，使用方式一样。所以3.1以hebing_mybatis中的版本为例。

剩下的一对文件夹中，api_request_OLD是同步到老采集库时需要使用的程序，而api_request是同步到新采集库时需要使用的工具。这两个文件夹，使用方式一样。所以3.2以api_request中的版本为例。

### 3.1 本地运行版使用说明
#### 3.1.1 修改mysql数据库连接信息(信源库和采集库)
首先需要配置好数据库连接信息，即jdbc.properties文件中的相关信息。改动部分在下方划线标注。

注意，修改好之后，不能修改文件名并将jdbc.properties置于mybatisbase.jar的同一目录中。

driver=com.mysql.jdbc.Driver	

\#\#不用改、上面是Mysql5的驱动

url=jdbc:mysql://~~10.61.1.\*\*:3306/wde\*\*~~?characterEncoding=utf8&useSSL=true&serverTimezone=Hongkong&allowMultiQueries=true		

\#\#修改中划线部分的**ip地址**和**数据库名**

username=\*\*\*\*\*\*\*\*\*\*\*

\#\#修改信源数据库用户名

password=\*\*\*\*\*\*\*\*\*\*\*

\#\#修改信源数据库用户名密码

`上面是信源库的数据库配置信息、下面是采集库的数据库配置信息`

driver2=com.mysql.jdbc.Driver 

url2=jdbc:mysql://~~10.61.1.28:3306/wde_monitor_wm~~?characterEncoding=utf8&useSSL=true&serverTimezone=Hongkong&allowMultiQueries=true   	

username2=		\#\#导入到的采集库用户名

password2=		\#\#导入到的采集库密码


#### 3.1.2 挂载参数运行mybatisbase.jar文件
例如： java -jar mybatisbase.jar -bs 2 -m 1 -d 2012-12-12 10:10:10 -size 1000000 -page 1

程序识别参数时具有一定容错性。这里updatetime的时间格式就是这样输入数字加分隔符，日期和时间之间用空格隔开即可。并且如果挂在了两个或多个可选模式的参数，程序会按照都符合这些挂载参数的最小粒度信源进行同步，不都符合则报错提示。

另外，由于加入了分页机制，如果想`一次性导入大批量的信源`，建议将size参数设为比较大的值，如例子中-size 1000000即可。

`-boardsfile 参数的使用`：只需将jar打包文件和××××.txt放在同一目录，然后挂载参数加上 -boardsfile或-f xxxx.txt即可，xxxx.txt中的信源格式为每一行一个boardid即可，换行分隔，无需其他格式。

#####参数说明：

|挂载参数控制符 |说明|
|:-------|:-------|
|-businessid -getbs -bs	|必须预先设定这次同步使用的业务businessid值，例如：-bs 2|
|-updatetime -getd -d	|可选是否设定updatetime，默认为lastupdatetime.lock文件中记录的时间值，格式yyyy-MM-dd%HH:mm:ss，空格分开日期与时间|
|-mediaid -getm -m	|可选模式1：根据mediaid选择一批满足条件的board|
|-sourceclusterid -gets -s 	|可选模式2：根据sourceclusterid选择一批满足条件的board|
|-boardid -getb -b	|可选模式3：根据boardid选择`一条`满足条件的board|
|-boardsfile -getf -f	|可选模式4：根据boardsfile.txt中的boardid进行批量同步|
|-size	|可选择设定分页每页信息条数，默认值50 |
|-page	|可选择分页页码，默认为第1页，支持异步分页导入|
|-h -help -geth	|帮助信息|





注意：建议只传入一个可选模式id值，也可以同时输入多个模式id值，同步程序会甄别最小粒度的有效id。

#### 3.1.3 查看反馈信息
反馈信息包括最后的统计结果打印输出、log日志和unsuccess_hebing.txt文件记录。打印输出中有详细的错误信息和纠正提示，unsuccess.txt文件中记录的是多条board信源一批同步时其中不成功的board信源情况。

### 3.2 API/request版使用说明
API_request或API_request_OLD文件夹下，包含API端和request端两个文件夹。

#### 3.2.1 修改mysql数据库连接信息(导出来的信源库)
进入API端文件夹，修改application.properties文件中信源库的数据库配置信息。例如下图

注意，修改好之后，不能修改文件名并将application.properties置于api.jar的同一目录中。

#### 3.2.2  开启API端服务
例如：java -jar api.jar

这时如果用浏览器访问8080端口的相关服务接口，填入账号和密码后，也可以查询到相关的信源。
例如：访问接口http://localhost:8080/tongbu?updatetime=2000-12-12+10:10:10&businessid=1
可以得到业务ID为1的若干条信源，json格式。

#### 3.2.3 修改mysql数据库连接信息(导入到的采集库)
jdbc_request.properties文件中采集库的数据库配置信息，如下图所示。

注意，修改好之后，不能修改文件名并将jdbc_request.properties置于request.jar的同一目录中。向老版本库同步的对应文件名是 jdbc_request_old.properties和request_old.jar。

#### 3.2.4 挂载参数发起request端请求
挂载参数，运行request端文件夹中的request.jar文件。与本地运行版相同，多出的参数是，必须`输入账号和密码`两个参数，以完成权限校验。还有API服务端`ip地址`的参数-ip需要确认。

例如：java -jar mybatisbase.jar -bs 1 -b 10602302 -username zhouyt -password 1234* -ip 10.61.1.37

参数说明：

|挂载参数控制符 |说明|
|:------|:------|
|-businessid -getbs -bs	|必须预先设定这次同步使用的业务businessid值，例如：-bs 2|
|-updatetime -getd -d	|可选是否设定updatetime，默认为lastupdatetime.lock文件中记录的时间值，格式yyyy-MM-dd%HH:mm:ss，空格分开日期与时间|
|-mediaid -getm -m	|可选模式1：根据mediaid选择一批满足条件的board|
|-sourceclusterid -gets -s 	|可选模式2：根据sourceclusterid选择一批满足条件的board|
|-boardid -getb -b	|可选模式3：根据boardid选择一条满足条件的board|
|-boardsfile -getf -f	|可选模式4：根据boardsfile.txt中的boardid进行批量同步|
|-size	|可选择设定分页每页信息条数，默认值50 |
|-page	|可选择分页页码，默认为第1页，支持异步分页导入|
|-h -help -geth	|帮助信息|
|**-password -pwd -p** |**账号密码** |
|**-ip -i -ipaddress** |**API服务的IP地址，默认值为127.0.0.1，例如 -ip 10.61.1.37** |
	

#### 3.2.5 查看反馈信息
这一版在request端和服务api端都可以看到程序比较准确的报错信息，反馈错误信息包括最后的统计结果打印输出（API端和request端都有）、log日志和request端的unsuccess.txt文件记录。打印输出中有详细的错误信息和纠正提示，unsuccess.txt文件中记录的是多条board信源一批同步时其中不成功的board信源情况。


## 4 维护说明

### 4.1 lastupdatetime.lock文件
按BusinessID值同步并且不设置updatetime时间时，单击运行版和服务请求版的request端会记录这次同步发生的时间，写入lastupdatetime.lock文件中。下一次所有`不挂载updatetime参数的同步请求`都会从这个日期之后进行同步。

如果同步时挂载了updatetime参数，那么lastupdatetime.lock文件中的时间记录值不会改变也不会被使用。

所以如果多次同步发现信源库中找不到符合条件的信源的情况出现时，有可能是lastupdatetime.lock文件中记录的时间超前，可以用vim检查这个文件中的时间值，初始的updatetime值是1971-01-01 00:00:00，检查发现问题修改时注意保持相同的时间格式。

### 4.2 常见的几类错误包括
* 数据库连接类（配置文件中信息）
* 配置文件、记录文件找不到类（文件的文件名或位置不对）
* JSON、XML解析类（extra_config字段含有除Configjson中	含有未出现过的新词）
* 非空属性类（采集库中要求非空的属性值，在封装前是一个空值）
* 找不到对应ID类（信源库中没有找到信源）

在`unsuccess.txt`中可以查看多条信源一起同步时，出错信源的信息。

所有统计的错误信息都是为了使用人员根据报错信息，进行相关修正后再次同步成功。

### 4.3 log说明
工具记录的log分为temp.log 、transfer_record.log、 transfer_record_历史日期.log

其中temp为上一次运行的输出，transfer_record.log为今天所有的输出记录日志，transfer_record_历史日期.log为历史记录中以天为单位的存储记录日志。这些日志便于查看错误和记录过程。

程序根目录下的unsuccess.txt文件仅为上一次运行时的报错记录信源。

### 4.4 运行程序时查看帮助
* 本地运行版：
挂载参数 -h 可以输出帮助信息，以便随时查看。

* Api/request版：
api端访问8080端口/h 接口，返回帮助信息。例如：访问http://127.0.0.1:8080/h

Request端同样挂在参数-h 可以输出帮助信息，以便随时查看。


## 5 开发人员历史记录
| 时间 | 角色 | 姓名 | 邮箱 |
| - | - | - | - |
| 2018.9-2018.10 | 开发者 | 周映彤 | zhouyingtong17s@ict.ac.cn |
| 2019.1-2019.1 | 开发者 | 周映彤 | zhouyingtong17s@ict.ac.cn |
| - | - | - | - |


## 发现问题

有任何bug或问题，联系邮箱zhouyingtong17s@ict.ac.cn 或者在 gitlab: xwde-boardsync 上提issue

## TODO List

- [ ]


## 附录： 信源同步时字段间的对应关系
信源库和老版本采集库中各个表之间属性间的对应关系：
```
select board.id as board_id,
board.jhi_key as board_jhikey,
board.name as board_name,
board.config as board_config,
board.add_time as board_addtime,
board.js as board_js,
board.proxy as board_proxy,
board.importance as board_importance,
source_cluster.name as sourcecluster_name,
source_cluster.id as sourcecluster_id,
media.id as media_id,
board_class.name as boardclass_name,
batch.business_id as batch_businessid
from board
left join source_cluster on board.source_cluster_id=source_cluster.id
left join media on source_cluster.media_id=media.id
left join batch_board on board.id=batch_board.board_id
left join batch on batch.id=batch_board.batch_id
left join board_class_map bcm on board.id = bcm.board_id
left join board_class on bcm.board_class_id = board_class.id
where board.status='FINISHED'
and media.id= #{mediaid}
and batch.business_id =#{businessid}
and DATE_FORMAT(board.update_Time,'%Y-%m-%d %H:%i:%s')>=#{updatetimeStr}
ORDER BY board.update_Time
```

以及 config字段的封装对应关系：
```
crawler.setBusiness_type(String.valueOf(businessid).concat("|").concat(businessname));
crawler.setDescription("");
crawler.setTime_modify(new Timestamp(System.currentTimeMillis()));
if(crawler.getImportance()!=null) priority = crawler.getImportance();
else {priority = crawler.getPriority(); if (priority > 10 || priority < 1) priority = 10;}
crawler.setPriority(priority);
String configtemp = crawler.getConfig();
Configjson configjson = objectMapper.readValue(configtemp, Configjson.class);
Config config = new Config();
config.setApp_id(businessid);
boardID=crawler.getSource_id();
config.setBoard_id(boardID);
config.setSite_id(crawler.getSource_cluster());
config.setChannel_id(crawler.getSource_media());
config.setBoard_name(crawler.getSource_name());
config.setSite_name(crawler.getSource_cluster_name());
config.setEntry_url(crawler.getSource_url());
config.setCharacter_set(configjson.getCharset());
if (configjson.getEntry_type() == null) config.setEntry_type(0);
else config.setEntry_type(Integer.parseInt(configjson.getEntry_type()));
if (configjson.getCurrentRegex() == null) config.setCurrent_news_regex(null);
else config.setCurrent_news_regex(configjson.getCurrent_news_regex());
if (configjson.getGather_depth() == null) config.setGather_depth(1);
else config.setGather_depth(Integer.parseInt(configjson.getGather_depth()));
if (configjson.getEvidence_degree() == null) config.setEvidence_degree(3);
else config.setEvidence_degree(Integer.parseInt(configjson.getEvidence_degree()));
if (configjson.getIs_homepage() == null) config.setIs_homepage(0);
else config.setIs_homepage(Integer.parseInt(configjson.getIs_homepage()));
if (configjson.getSub_board_list() == null) config.setSub_board_list(null);
else config.setSub_board_list(configjson.getSub_board_list());
if (configjson.getPost_enabled() == null) config.setPost_enabled(0);
else config.setPost_enabled(Integer.parseInt(configjson.getPost_enabled()));
config.setImportance_degree(crawler.getPriority());
config.setHttp_interval(configjson.getHttpInterval());
config.setDocurl_regex_yes(configjson.getDocurlRegexYes());
config.setDocurl_regex_no(configjson.getDocurlRegexNo());
config.setDocurl_regex_page(configjson.getDocurlRegexPage());
config.setImgurl_regex_no(configjson.getImgurlRegexNo());
config.setImgurl_regex_yes(configjson.getImgurlRegexYes());
config.setBoardurl_page_regex(configjson.getCurrentRegex());
config.setBoardurl_max_pages(configjson.getBoardUrlMaxPages());
config.setRefresh_period(configjson.getRefreshPeriod());configtemp = configjson.getExtractorConfig();
if (configtemp == null|| configtemp.equals(""))config.setExtractor_config("");
else config.setExtractor_config(split_extractorConfig(configjson.getExtractorConfig()));
config.setJs_enabled(crawler.getJs());
config.setGfw_enabled(crawler.getProxy());
config.setBoard_class_tag(crawler.getBoard_class_tag());
xmlMapper.enable(SerializationFeature.INDENT_OUTPUT);
String strConfig = xmlMapper.writeValueAsString(config).replace("</Config>", "</board>\n").replace("<Config>", "<board>");
crawler.setConfig(strConfig);
```
