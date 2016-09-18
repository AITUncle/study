# GSON vs FastJson vs JackSon（on android）
---

### 对应版本和jar大小

| lib | size(b) |
| ------| ------ |
| com.google.code.gson:gson:2.6.2 | 229650 |
| com.alibaba:fastjson:1.1.54.android | 203475 |
| jackson-2.7.3(core+databind+annotations) |252518+1202276+50897=1505691|

jackson 1.5M，尽管他高效的反序列化（比GSON要快很多），功能齐全以及非常好的拓展性，但依赖太多导致jar太大往往直接被android开发人员嫌弃继而抛弃。




###国内Top500和Google Play Top200 Android应用FastJson Gson JackSon使用情况：
---

|| Gson | FastJson | JackSon|
| ------| ------| ------ | ------ |
|国内top500| 227 | 87 | 26 |
|google paly top200| 57 | 0 | 15 |

数据来源：[国内top500点这里](http://mp.weixin.qq.com/s?__biz=MzA5OTMxMjQzMw==&mid=2648112527&idx=1&sn=b23c1b5f3e32e343ad96d705bd4d63ff&scene=2&srcid=0711GL3B90iyRPmjRKTBN1I0&from=timeline&isappinstalled=0#wechat_redirect)
[google play top 200点这里](http://mp.weixin.qq.com/s?__biz=MzA5OTMxMjQzMw==&mid=2648112540&idx=1&sn=c2dc3d17f561337ce9b0fe93537d769f&scene=1&srcid=09177ZRJvPhvNdC2rPIyPFbD#rd)




###github数据对比,2016年9月
---
| 名称 | star数量 | Issues数量 |Pull requests数量|fork数量|网址|
| ------| ------ | ------ | ------ | ------ | ------ |
| GSON| 6037 | 180 | 28 | 1442 |https://github.com/google/gson|
| FastJson| 6378 | 106 | 0 | 2462 |https://github.com/alibaba/fastjson/issues|

通过github上的数据看出GSON和FastJson都很受欢迎，社区的人都比较活跃。但从官方提供在github上的文档来看,很明显GSON远比FastJson要详细，通熟易懂。虽然FastJson有中文和英文两份文档，但是我个人感觉GSON的文档更符合程序员的口味。





### 效率对比GSON vs FastJson
---
一段非常简单的json结构，反序列化对比：

```json
{
    "age": 10,
    "array": [
        "abc",
        "123",
        "345",
        "cdf"
    ],
    "isTrue": true,
    "name": "vectorzeng"
}
```



首次反序列化测试
---
| GSON(ms) | FastJson(ms) | JsonObject(常规方法ms)|
| ------| ------ | ------ |
| 81| 37| 2|
| 79| 66| 2|
| 69| 49| 2|



非首次反序列化测试
---
| GSON(ms) | FastJson(ms) |JsonObject(常规方法ms)|
| ------| ------ | ------ |
| 0 | 0 | 0 |
| 1 | 1 | 0 |
| 0 | 0 | 0 |

之前的demo测试的FastJson比GSON快6倍，是因为我对GSON的使用不当导致的差异。实际对于上面这种常见的小数据来看并没有6倍。数据告诉我们FastJson确实比GSON要快。
官方提供数据，他们的测试可能更全面一些。但他们并不是针对android版本的测试。
[https://github.com/alibaba/fastjson/wiki/%E5%90%84%E7%A7%8DJSON%E5%BA%93%E7%9A%84%E6%AF%94%E8%BE%83](https://github.com/alibaba/fastjson/wiki/%E5%90%84%E7%A7%8DJSON%E5%BA%93%E7%9A%84%E6%AF%94%E8%BE%83)



### 容错：
---

#### json字段缺失与冗余，序列化以及反序列化时，Gson以及FastJson都有很好的容错性



---
#### 反序列化时类型转换容错测试
---

| int类型转换 | GSON | FastJson |JSONObject.optInt|JSONObject.getInt|
| ------| ------ | ------ | ------ | ------ |
| "10"-->int  | 10 | 10 | 10 | 10 |
| 1.02-->int  | 崩溃 | 1 | 1 | 1 |
| "1.88"-->int  | 崩溃 | 崩溃 | 1 | 1 |
| null-->int | 0 | 0 | 0 | JSONException |
| "abc"-->int | 崩溃 | 崩溃 | 0 | JSONException |
| ["abc","123"]-->int | 崩溃 | 崩溃 | 0 | JSONException |
| {"abc":"123"}-->int | 崩溃 | 崩溃 | 0 | JSONException |


| String类型转换 | GSON | FastJson |JSONObject.optString|JSONObject.getString|
| ------| ------ | ------ | ------ | ------ |
| 10-->String  | "10" | "10" | "10" | "10" |
| null-->String | null | null | null | null |
| ["abc","123"]-->int | 崩溃 | "["abc","123"]" | "["abc","123"]" | "["abc","123"]" |
| {"abc":"123"}-->int | 崩溃 | "{"abc":"123"}" | "{"abc":"123"}" | "{"abc":"123"}" |


| Boolean类型转换 | GSON | FastJson |JSONObject.optBoolean|JSONObject.getBoolean|
| ------| ------ | ------ | ------ | ------ |
| "true"-->booleab  | true | true | true | true |
| "1"-->booleab | false | true | false | JSONException |
| 1-->boolean | 崩溃 | true | false | JSONException |
| ["abc","123"]-->boolean | 崩溃 | 崩溃 | false | JSONException |
| {"abc":"123"}-->boolean | 崩溃 | 崩溃 | false | JSONException |

FastJson和Gson都可以通过二次开发定制类型转换的结果。




---
### Gson FastJson使用中的一些差异：
- Gson对所有访问权限的成员变量都可以序列化和反序列化，FastJson只对有访问权限的成员变量或者有配有public set\*方法的成员变量才会反序列化。只对有访问权限的成员变量或者配有public get\*方法的成员变量才会序列化。
有点难理解，看例子就一目了然了如下举例

``` java
//Bean.java
class Bean{
    private int age = 99;				//private访问权限，既没有getAge(),也没有setAge()方法；FastJson对这种属性即不序列化，也不反序列化
    public String name = "vz123";
    private boolean isMan = true;		//private权限，只有getIsMan()，没有setIsMan()方法，这种属性在FastJson中只会序列化并不会反序列化
    private boolean isTrue = true;		//private权限，没有getIsTrue()，只有setIsTrue(),FastJson对这种属性只会反序列化，并不会序列化
    public boolean getIsMan(){
        return this.isMan;
    }
    public void setIsTrue(boolean t){
        this.isTrue = t;
    }
}
//Gson序列化结果:{"age":99,"isMan":true,"isTrue":true,"name":"vz123"}
//FastJson序列化结果：{"isMan":true,"name":"vz123"}
////////////////
//Gson反序列化：所有属性都会被反序列化
//FastJson反序列化,只有如下属性:name,isTrue
```


默认情况下，在序列化和反序列化的规则上FastJson更为复杂。而Gson简单明了通熟易懂。而且以上规则官方并没有在文档中阐述，而是靠开发人员自己使用中发觉。不知道FastJson的作者是故意如此设计还是无意为止。

---
- 序列化属性名的差异：Gson vs FastJson

```java
public class Bean6 {

    public boolean isTrue;

    public boolean getTrue1(){
        return isTrue;
    }

    public boolean getTrue2(){
        return isTrue;
    }
}
//Gson序列化结果：{"isTrue":false}
//FastJson序列化结果：{"isTrue":false,"true1":false,"true2":false}
```

由上面的测试结果可以推测，FastJson以一种简单粗暴的方式：
    将所有的可以访问的成员变量以及get/*成员方法都给序列化了，而且以一种简单粗暴的方式生成了json的属性名。
而Gson始终以成员变量保持一致，json的属性名也与成员变量名保持一致。




------
------
---------
以上所有测试都是使用Gson以及FastJson标准的使用方法，并没有做任何差异化配置。
测试设备是：魅蓝3s手机，android5.1,Flyme5.1.5.0A

作者：vectorzeng









