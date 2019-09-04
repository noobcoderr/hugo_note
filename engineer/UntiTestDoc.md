##需求

目前有一个需求，接入wearn渠道。

##需求解读

该需求里包含2大事件:

* 对点击事件回传进行处理 — HandleCallback( )

* 要对符合上报的数据进行匹配上报处理 — Report( )

##系统流程设计

##对点击事件回传进行处理 — HandleCallback( )

该流程接受来自渠道的http请求(携带了一些关键参数),我们需要对其进行一些处理后进行存库。处理包括:

* 解析携带的参数 — ParseParams( )
* 查询下载页 — QryDownloadUrl( )
* 转换muid — CreatMuid( )
* 打印点击事件bi日志 — BIReport( )
* 点击事件数据入库 — InsertClkData( )

##要对符合上报的数据进行匹配上报处理 — Report( )

该流程接受来自SDK的http请求(当游客进行注册、途游用户进行登录、消费时触发，并携带一系列用户信息)。我们对其进行处理后存库。

* 查询clientId — FindClientId( )
* 打印BI日志
* 传入消息队列处理
* 修改匹配参数 — FindAndModify( )
* 查询wearn渠道的signKey — QryWearnSignKey( )
* 组装好http请求后进行请求上报

## 表格设计

* 点击事件回传表格

  |  参数名  |  类型  | 是否必选 |
  | :------: | :----: | :------: |
  |  Params  | string |    是    |
  | DeviceId | string |    是    |
  |  UserId  | string |    是    |
  |  TaskId  |  int   |    是    |
  |    OS    |  int   |    是    |
  | Callback | string |    是    |

  

## 测试

* HandleCallback( )
  * ParseParams( )
    * 该函数接收一个字符串参数，将我们提供的params参数进行解析，返回解析好的填充到点击事件数据表的表结构体
    * 测试用例：空串""、
  * QryDownloadUrl( )
    * 该函数接收一个字符串参数client_id，根据提供的client_id查询mongo数据库中对应的downloadurl
    * 测试用例：空串""、

未完。。。











关于是否有必要分割一些操作为一个函数的标准我觉得是

* 是否对原始数据进行了一长串的复杂判断与处理。如果是，可以分割出来单独成为一个函数，可以被其他渠道调用
* 是否具有通用性，如果就单某一个渠道才会出现的情况，不建议分割。