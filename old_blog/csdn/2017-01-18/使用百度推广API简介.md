最近公司使用了百度推广服务，为了筛选出合适的词条(每个词条进入百度推广的搜索都是收费的，显然提取一些"物美价廉"的词条是有必要的)，需要对接百度推广的API，从搜索状态报告中提取有用的信息加以分析。
因为自己之前从未接触这方面的开发，因此遇到不少困难，今天终于从百度服务端获取到了数据，记录下来，算是一个印迹吧 ：)
#### 关于JSON
处于网络上不同的机器不同应用需要实现互相交流，一种被双方普遍认可的信息数据格式是很必要的，json作为一种轻量级的数据交换格式，是百度推广API建议开发者首选的方式。这是很不错的选择，毕竟，让人类去玩XML简直是太残忍了。具体到本篇文章，开发语言选择的是python，可以认为需要与百度服务器端交互时候，传给他的数据就是一个字典。
#### 示例代码
示范代码如下：
```
from sms_v3_KeywordService import sms_v3_KeywordService
keyWordservice = sms_v3_KeywordService()
ReportService = sms_v3_ReportService()
'''
下面是重新设置用户名等认证信息，其实上一步时候会从baidu-api.properties里加载默认的用户信息，包括url、username、password、token、target、accessToken等
'''
keyWordservice.setUsername("xxxx")
keyWordservice.setPassword("xxxx")
keyWordservice.setToken("xxxx")
Reportservice.setUsername("xxxx")
Reportservice.setPassword("xxxx")
Reportservice.setToken("xxxx")
＃组织与百度端交互所使用的数据
request = {"ids":[1234567],"type": 3}
res = service.getKeywordStatus(request)#获取关键字信息
printJsonResponse(res)
request = {"realTimeRequestTypes":
              {"performanceData":["click","impression"],
              "startDate":"2015-04-12T10:00:00.000",
              "endDate":"2015-04-12T11:00:00.000",
              "levelOfDetails":11,
              "statRange":11,
              "unitOfTime":5,
              "reportType":14,
              "device":0,
              }
          }
res = service.getRealTimeData(request)＃获取实时统计数据报告
printJsonResponse(res)
```
当然在做这一切之前，还需要安装request插件，下载百度的APK等，具体可以参考百度开发者的指导文档。
祝大家好运 ：)