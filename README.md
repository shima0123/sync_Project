# XWDE-BoardSync

## 1 简介

同步工具主要是从信源库向（新、老）两个版本的采集库同步信源。可以按照业务businessID、通道mediaID、信蔟sourceclusterID、信源boardID、信源列表boardList文件五种方式进行选择，并可以选择time_create信源创建时间于某时间段以后的信源进行同步。

分为本地运行版和API / request版。本地运行版是可以直接运行jar包并挂载筛选条件的独立程序(基于Mybatis)。API/request版分为API端和request端：API端采用SSM框架，支持权限校验；request端在运行jar包时挂载筛选条件。一共6个jar包。

本地运行版和服务请求版，均需要在运行前配置好(application.properties或jdbc*.properties)的数据库连接配置文件)。

支持分页同步、记录并锁定批量同步的时间、异常情况分类统计和输入参数容错。


## 2 开发环境说明

* Jdk版本 1.7

* Mysql版本5

* Maven版本 4.0.0（Maven-compiler版本1.9、单机运行版Maven-shade版本2.4.1、API版 springboot版本 2.0.1.RELEASE）

* Mybatis版本3.2.6


## 3 使用说明

工程目录中一共有四个文件夹。

hebing_mybatisbase文件夹为本地运行版。带OLD标志的是同步到老采集库时需要使用的程序，不带OLD的程序是同步到新采集库时需要使用的程序。这两个文件夹，使用方式一样。所以3.1以hebing_mybatis中的版本为例。

剩下的一对文件夹中，api_request_OLD是同步到老采集库时需要使用的程序，而api_request是同步到新采集库时需要使用的工具。这两个文件夹，使用方式一样。所以3.2以api_request中的版本为例。

### 3.1 本地运行版使用说明
#### 3.1.1 修改mysql数据库连接信息(信源库和采集库)
首先需要配置好数据库连接信息，即jdbc.properties文件中的相关信息。改动部分在下图中粗体，下划线标注。

注意，修改好之后，不能修改文件名并将jdbc.properties置于mybatisbase.jar的同一目录中。


#### 3.1.2 挂载参数运行mybatisbase.jar文件
例如： java -jar mybatisbase.jar -bs 2 -m 1 -d 2012-12-12 10:10:10 -size 1000000 -page 1

程序识别参数时具有一定容错性。这里updatetime的时间格式就是这样输入数字加分隔符，日期和时间之间用空格隔开即可。并且如果挂在了两个或多个可选模式的参数，程序会按照都符合这些挂载参数的最小粒度信源进行同步，不都符合则报错提示。

另外，由于加入了分页机制，如果想一次性导入大批量的信源，建议将size参数设为比较大的值，如例子中-size 1000000即可。

-boardsfile 参数的使用：只需将jar打包文件和××××.txt放在同一目录，然后挂载参数加上 -boardsfile或-f xxxx.txt即可，xxxx.txt中的信源格式为每一行一个boardid即可，换行分隔，无需其他格式。

参数说明：






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
例如：访问接口http://localhost:8080/tongbu?updatetime=2000-12-12 10:10:10&businessid=1
可以得到业务ID为1的若干条信源，json格式。

#### 3.2.3 修改mysql数据库连接信息(导入到的采集库)
jdbc_request.properties文件中采集库的数据库配置信息，如下图所示。

注意，修改好之后，不能修改文件名并将jdbc_request.properties置于request.jar的同一目录中。向老版本库同步的对应文件名是 jdbc_request_old.properties和request_old.jar。

#### 3.2.4 挂载参数发起request端请求
挂载参数，运行request端文件夹中的request.jar文件。与本地运行版相同，多出的参数是，必须输入账号和密码两个参数，以完成权限校验。还有API服务端ip地址的参数-ip需要确认。

例如：java -jar mybatisbase.jar -bs 1 -b 10602302 -username zhouyt -password 1234** -ip 10.61.1.37

参数说明：




#### 3.2.5 查看反馈信息
这一版在request端和服务api端都可以看到程序比较准确的报错信息，反馈错误信息包括最后的统计结果打印输出（API端和request端都有）、log日志和request端的unsuccess.txt文件记录。打印输出中有详细的错误信息和纠正提示，unsuccess.txt文件中记录的是多条board信源一批同步时其中不成功的board信源情况。


## 4 维护说明

### 4.1 lastupdatetime.lock文件
按BusinessID值同步并且不设置updatetime时间时，单击运行版和服务请求版的request端会记录这次同步发生的时间，写入lastupdatetime.lock文件中。下一次所有不挂载updatetime参数的同步请求都会从这个日期之后进行同步。

如果同步时挂载了updatetime参数，那么lastupdatetime.lock文件中的时间记录值不会改变也不会被使用。

所以如果多次同步发现信源库中找不到符合条件的信源的情况出现时，有可能是lastupdatetime.lock文件中记录的时间超前，可以用vim检查这个文件中的时间值，初始的updatetime值是1971-01-01 00:00:00，检查发现问题修改时注意保持相同的时间格式。

### 4.2 常见的几类错误包括
* 数据库连接类（配置文件中信息）
* 配置文件、记录文件找不到类（文件的文件名或位置不对）
* JSON、XML解析类（extra_config字段含有除Configjson中	含有未出现过的新词）
* 非空属性类（采集库中要求非空的属性值，在封装前是一个空值）
* 找不到对应ID类（信源库中没有找到信源）

在unsuccess.txt中可以查看多条信源一起同步时，出错信源的信息。

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
| - | - | - | - |
| 2019.1-2019.1 | 开发者 | 周映彤 | zhouyingtong17s@ict.ac.cn |
| - | - | - | - |

## 发现问题

有任何bug或问题，联系邮箱zhouyingtong17s@ict.ac.cn 或者在 gitlab: xwde-boardsync 上提issue

## TODO List

- [ ]
