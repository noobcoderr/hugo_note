## apiDoc

基于nodejs自动生成RESTful Api文档的工具。

### 使用方法

####下载apidoc

`npm install apidoc -g`

####在定义接口的地方书写如下格式的注释代码(此为示例，具体可看看接口情况)

```go
/**
 * @api {post} /geocoder/v1/renderReverse.json  逆地理编码
 * @apiGroup translate
 * @apiPermission translate-manage
 * @apiParam {String} access_id          子账号ID
 * @apiParam {String} algorithm="sha256"    签名算法
 * @apiParam {String} secret_type="session" 签名密钥类型
 * @apiParam {Number} ts                时间戳
 * @apiParam {String} sign             签名内容
 * @apiParam {Float}   lat                纬度值
 * @apiParam {Float}   lng                经度值
 * @apiParam {Number}  max_distance      最大半径 【可选】 单位千米,默认1千米
 * @apiSuccessExample Success-Response:
 *     HTTP/1.1 200 OK
 *     {
 *       "code": 0,
 *       "result": {
 *          "address_component": {
 *             "country":"中国",       //国家
 *             "province":"北京市",     //省名
 *             "city":"北京市",        //城市名
 *             "district":"东城区",     //区县名
 *             "town":"",          //乡镇名
 *             "street":"中华路",       //街道名（行政区划中的街道层级）
 *             "street_number":"10号", //街道门牌号
 *             "adcode":"110101",    //行政区划代码 adcode映射表
 *             "country_code":0,     //国家代码
 *             "direction":"北",      //相对当前坐标点的方向，当有门牌号的时候返回数据
 *             "distance":"100"      //相对当前坐标点的距离，当有门牌号的时候返回数据
 *          },
 *          "cache": "true", //是否从缓存中取得的
 *          "formatted_address":""
 *     }
 *     }
 * @apiErrorExample {json} Error-Response:
 *     HTTP/1.1 200 OK
 *     {
 *     	 "code":201
 *       "error": "UserNotFound"
 *     }
 */
```

#### 配置apidoc.json

```json
{
  "name": "example",
  "version": "0.1.0",
  "description": "apiDoc basic example",
  "apidoc": {
    "title": "Custom apiDoc browser title",
    "url" : "https://api.github.com/v1"
  }
}
```

各字段可自定义

####运行命令、生成api文档

`apidoc -i /定义接口文件路径 -o /生成api静态文档存放路径`

#### 参数说明

* -i 需要生成接口的文件的路径，只需要写到其文件夹的路径，不必指向具体文件
* -o 生成的静态文件存放的位置

#### 注释里的注解说明apiDoc-PARAMS

####@api

`@api {method} path [title]`

| 参数   | 描述                              |
| ------ | --------------------------------- |
| method | http请求的方法，包括GET、POST..等 |
| path   | 接口的路径                        |
| title  | 出现在生成的静态文档的标题里      |

示例:

```go
/**
* @api {post} /deocoder/v1/renderReverse.json 逆地理编码
**/
```

其他api注解请移步



[apiDoc官方文档](http://apidocjs.com/#params)