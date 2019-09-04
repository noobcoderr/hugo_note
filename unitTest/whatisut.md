

##单元测试-MOCK

[通俗理解mock](https://jingwei.link/2018/06/23/golang-testcase-best-practices.html)

单元测试，不应该依赖dao层的执行逻辑是否正确，否则就是集成测试，需要假设dao层的行为是什么样子，然后再看本层的逻辑是否正确。

公司开发人员的代码质量往往不是很高，尤其是对代码的拆分和逻辑的抽象还处于懵懂阶段。要对这类代码写单测，即使是工作了3，4年的高级码农也是一个挑战，对新人来说几乎是不可能完成的任务。这也让很多开发人员有了**写单元测试很难**的感觉。所以，**写单元测试的难易程度跟代码的质量关系最大，并且是决定性的**。项目里无论用了哪个测试框架都不能解决代码本身难以测试的问题。

## 单元测试

单元测试的重点在于发现程序设计或实现的逻辑错误，使问题及早暴露，便于问题的定位解决

定义单元测试概念的关键点就是，它能够脱离运行时的依赖项或合作者，这在 Go 语言中是通过接口来实现的，Go 语言中接口是隐含的，而不是一种强制措施。意味着实际的类并不需要知道接口的存在。

## 性能测试

性能测试的重点在于发现程序设计上的一些问题，让线上的程序能够在高并发的情况下还能保持稳定

##功能测试 TestXxxx(t *testing.T)

功能测试偏向于测试源码文件中的具体功能，属于UT的范畴，比如具体的函数功能。如muid转换函数的测试就属于功能测试



## 测试用例的组织

将多个测试用例放在列表中，这叫Table Driven,在定义输入的时候就定义好了结果输出，便于测试用例的组织，以及将来的添加，删除，更改维护。

```go
func TestSqrt(t *testing.T) {
  var cases = [] struct{
    in int
    out int
    err error
  }{
    { 4,2,nil},
    {16,4,nil},
    }

    for _, testcase := range cases {
        actualOut := Sqrt(testcase.in)
        if actualOut != testcase.out {
            t.Errorf("case (%v) expect (%v) but got (%v)", testcase, testcase.out, actualOut)
        }
    }
}
```



## 测试Main函数

来在运行测试用例之前做一些必要的setup以及之后的tearDown操作。

## 伪造模式

单元测试中比较头痛的依赖问题。也就是伪造模式，经典的伪造模式有桩对象(stub),模拟对象(mock)和伪对象(fake)。

##gomock

gomock模拟对象的方式是让用户声明一个接口，然后使用gomock提供的mockgen工具生成mock对象代码。要模拟(mock)被测试代码的依赖对象时候，即可使用mock出来的对象来模拟和记录依赖对象的各种行为：比如最常用的返回值，调用次数等等。

1、定义想要模拟的依赖项的interface

```go
type MockTest interface {
   B()    int
}

func A() int {
   return 18 + B()
}

func B() int {
   return rand.Intn(10)
}
```

2、使用 mockgen 命令对所需 mock 的 interface 生成 mock 文件

mockgen -source=/相对路径/依赖interface所在文件.go -destination=/mock出的文件存放的位置 -package=指定mock文件的包名

-source：设置需要模拟（mock）的接口文件

-destination：设置 mock 文件输出的地方，若不设置则打印到标准输出中

-package：设置 mock 文件的包名，若不设置则为 mock_ 前缀加上文件名

3、编写单元测试的逻辑，在测试中使用 mock

##使用 mock 隔离第三方依赖或者复杂调用

​	很多时候，测试环境不具备 routine 执行的必要条件。比如查询 consul 里的 KV，即使准备了测试consul，也要先往里面塞测试数据，十分麻烦。又比如查询 AWS S3 的文件列表，每个开发人员一个测试 bucket 太混乱，大家用同一个测试 bucket 更混乱。必须找个方式伪造 consul client 和 AWS S3 client。通过伪造 consul client 查询 KV 的方法，免去连接 consul， 直接返回预设的结果。

​	首先考虑一下怎样伪造 client。假设 client 被定义为 var client *SomeClient。当 SomeClient 是 type SomeClient struct{...} 时，我们永远没法在 test 环境修改 client 的行为。当是 type SomeClient interface{...} 时，我们可以在测试代码中实现一个符合 SomeClient interface 的 struct，用这个 struct 的实例替换原来的 client。



##golang单元测试之httptest

场景：广告点击数据经过匹配归因后，要上报到广告平台，即需要发送http请求，并通过广告平台返回的数据来判断成功与否，我们的目的是测试HandlePush的逻辑是否正确。

```go
func (this *AiClk) HandlePush(data mgotools.CallbackDataBean, extra map[string]string, logger *utils.Logger) {

   //激活BI日志打印
   biData := &tools.BiData{
      CloudId:   utils.String2Int(data.CloudId),
      LogType:   tools.LogType,
      EventTime: time.Now().Unix(),
      EventId:   tools.ActiveId,
      UserId:    utils.String2Int(extra["user_id"]),
      GameId:    0,
      AppId:     utils.String2Int(data.AppId),
      ClientId:  data.ClientId,
      IpAddr:    0,
      DevId:     data.MuId,
      BindId:    "0",
      PhoneType: utils.GetPhoneType(extra["phone_type"]),
      Province:  0,
      EventParams: tools.BiEventParams{
         PromoteChannel: data.AdsType,
         ActivityName:   data.ActName,
         AdsId:          "0",
         AdvertiserId:   data.Extra["uid"],
         CreativeId:     data.Extra["cid"],
         UnitId:         data.Extra["unit"],
         PlanId:         data.Extra["plan"],
      },
   }
   err := tools.BIReportData(biData)
   if err != nil {
      logger.Errorf("qtt BI Register Log Error >>> %+v, %+v, %+v", err.Error(), data, extra)
   }
  
   //修改状态
   rptdao := &mgotools.EventReportDao{
      CloudId: data.CloudId,
   }
   defer rptdao.Close()
   err = rptdao.FindAndModify(data.MuId, data.ClientId, data.AdvPlatform, extra)
   if err != nil {
      logger.Errorf("qtt Report Item Update State Failed With Reason >>> %+v, %+v, %+v", err.Error(), data, extra)
   }
   logger.Infof("qtt Report Item Update State Success >>> %+v, %+v", data, extra)

   //上报数据
   u := data.Extra["callback_url"]
   u += "&op2=0"
   cli := tools.SimpleHttpClient()
   req, err := http.NewRequest("GET", u, nil)
   rst := tools.MakeHttpRequest(cli, req)
   if rst.Err != nil || rst.StatusCode != http.StatusOK {
      logger.Errorf("qtt Report Faild With Reason >>> %+v, %+v", rst, data)
      return
   }
   logger.Infof("qtt Report Item Success >>> %+v, %+v", data, extra)

}
```

开发这个函数时，我们没有真实的点击数据、真实的激活数据、只知道上报接口的返回值是怎么样的。解决办法就是通过单元测试中的httptest，实现http server，并设置好返回值，那么http.Get(url)的请求就会直接打到单元测试的http server上，同时得到你设置好的返回值，你就可以继续去处理数据，测试你的代码逻辑了。

该方法包括

1、打印激活BI日志

2、修改数据库状态

3、请求上报服务器

需要测试的内容

1、测试HandlePush是否发送了正确的http请求，method是否正确，路径是否正确，请求参数是否正确

2、返回状态码是否正确

3、返回值是我们设置的值

但是当我们没有网络，那么请求上报服务器久肯定无法正常运行，如何去发送一个真正的http request而不去真正的启动一个http server呢？

/dsp_conversion_cb





## 各类设计模式的单元测试该如何写



https://yanyangcode.wordpress.com/2016/06/28/first-blog-post/)