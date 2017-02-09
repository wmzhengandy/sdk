# 运行平台
Android平台上目前支持所有主流体系结构，如armv8，armv7，mips，x86。其它平台可以提供Java版SDK，目前暂不向外开放。如有相关需求，请与我们联系。
# 使用说明
##初始化
    // 非正式Appkey， 仅提供给开发者Demo使用
    private static final String sAppKey = "com.mobvoi.test";
    SpeechClient.getInstance().init(context, sAppKey, true, true);
##热词唤醒
实现接口

    public interface HotwordListener {
        void onHotwordDetected();
    }
打开热词监听

    SpeechClient.getInstance().addHotwordListener(new HotwordListener() {
        @Override
        public void onHotwordDetected() {
        // 此时添加热词唤醒后的处理内容
        }
    });
    
    SpeechClient.getInstance().startHotword();
 
关闭热词监听

    SpeechClient.getInstance().removeHotwordListener(new HotwordListener() {               
        @Override
        public void onHotwordDetected() {
        // 此时添加热词唤醒后的处理内容 
        }
    });
    
    SpeechClient.getInstance().stopHotword();

##语音识别

设置回调

    SpeechClient.getInstance().setClientListener(“ClientName”, new SpeechClientListenerImpl()); 
    
而SpeechClientListenerImpl需要实现接口SpeechClientListener

    private class SpeechClientListenerImpl implements SpeechClientListener {
        
        //开始提供录音数据给语音识别引擎时回调
        public void onStartRecord() {}

        //远端检测到静音时回调
        public void onRemoteSilenceDetected() {}
        
        //输入语音数据实时的音量（db值）回调
        public void onVolume(double volume) {}
        
        // 语音识别部分结果返回，比如“今天天气怎么样”，会按顺序返回“今天”，“今天天气”，“今天天气怎么样”，
        // 前两个就属于Partial Transcription
        public void onPartialTranscription(final String result) {}

        // 语音识别最终结果返回，比如“今天天气怎么样”，会按顺序返回“今天”，“今天天气”，“今天天气怎么样”，
        // 最后一个就是Final Transcription
        public void onFinalTranscription(final String result) {}
        
        // 语音搜索结果返回
        public void onResult(final String result) {}

        // 错误码返回
        public void onError(final int errorCode) {}
        
        //本地静音时回调
        public void onLocalSilenceDetected() {}

        //无语音输入时回调
        public void onNoSpeechDetected() {}
        
        //检测到语音时回调
        public void onSpeechDetected() {}
        
        // 语音识别服务已经成功初始化
        public void onReady() {}
    }

    
语音识别

* Mobvoi支持多种语音识别方式： ASR，仅语音识别，无语义分析，无搜索结果。
* Semantic, 语音识别，返回语义分析，无搜索结果。
* Onebox，语音识别，并返回搜索结果。
* Mix，离线在线混合的语音识别，返回结果结合二者的优势。当无网络连接时，自动回退到离线。
* Offline，离线语音识别，支持命令词识别或通讯录识别，如打电话给王路，打开支付宝等。需要用户提供APP列表或通讯录列表。

每种识别方式，调用的接口是非常类似的，均是三个接口：

* Start，启动语音识别，此时系统会录音，并把录音流式发送到语音服务器。
* Stop，停止系统录音，等待识别结果返回。不调用此函数的话系统会自动做静音检测，若检测到静音就认为录音结束。
* Cancel，取消此次语音识别，系统不会返回任何结果。

例子：

    SpeechClient.getInstance().startMixRecognizer(deviceName);
    SpeechClient.getInstance().stopRecognizer(deviceName);
    SpeechClient.getInstance().cancelReconizer(deviceName);
##语音搜索
    SpeechClient.getInstance().startTextSearch(var1, var2, new SearchListener() {
        @Override
        public void onBeginSearch() {
            //开始语音搜索
        }
                
        @Override
        public void onResult(String s) {
            //搜索结果
        }

        @Override
        public void onError(int i) {
            //搜索出错时
        }
    });
##语义分析
    SpeechClient.getInstance().startTextSemantic(var1, new SearchListener() {
        @Override
        public void onBeginSearch() {
            //开始语音搜索
        }

        @Override
        public void onResult(String s) {
            //搜索结果
        }

        @Override
        public void onError(int i) {
            //搜索出错时
        }
    });
##热词+语音搜索
支持热词+语音搜索一次触发（Oneshot）模式。 比如可以直接说“你好问问今天天气怎么样”， 这样在某些场景下， 用户会感觉交流更自然， 反应更快捷。 针对不同的语音搜索， 我们提供了不同的Oneshot接口， 包括： Mix， Onebox, Semantic和Offline, 以下以Mix为例：

打开Oneshot

    SpeechClient.getInstance().startOneshotMixRecognizer(deviceName);
    
关闭Oneshot
    
    SpeechClient.getInstance().stopOneshotRecognizer(deviceName);
    
取消Oneshot
    
    SpeechClient.getInstance().cancelOneshotRecognizer(deviceName); 

##语音合成

    SpeechClient.getInstance().startTTS(ttsRequest, new TTSListener() {
        @Override
        public void onStart() {
            // 当TTS开始播放时回调
        }

        @Override
        public void onError() {
            // 当TTS播放出错时回调
        }

        @Override
        public void onDone() {
            // 当TTS播放结束时回调
        }
    });

关闭语音合成

    SpeechClient.getInstance().stopTTS();


# 返回结果

## 格式

结果采用标准Json返回。比如“今天天气怎么样？”的搜索结果如下：

``` json
{
    "content": {
        "task": "public.weather",
        "query": "天气",
        "searchQuery": "",
        "msgId": "",
        "data": [{
            "source": "中国气象局",
            "params": {
                "tts": "科尔沁右翼中旗今天多云转晴，20°C到33°C",
                "pageData": [{
                    "weatherBg": "cloudy",
                    "wind": "6级",
                    "weekDay": "今天",
                    "location": "科尔沁右翼中旗",
                    "status": "success",
                    "date": "08月12日",
                    "minTemp": "20",
                    "imgUrl": "http://onebox.oss.aliyuncs.com/img/weather/cloudy_98_98.png",
                    "maxTemp": "33",
                    "unit": "C",
                    "currentTemp": "33",
                    "sunset": "19:05:00",
                    "aqi": "77",
                    "weather": "多云转晴",
                    "pm25": "77",
                    "sunrise": "04:52:00",
                    "tips": "",
                    "windDir": "西南风"
                }, {
                    "weatherBg": "clear",
                    "maxTemp": "32",
                    "imgUrl": "http://onebox.oss.aliyuncs.com/img/weather/sun_48_48.png",
                    "wind": "4-5级",
                    "unit": "C",
                    "weekDay": "周六",
                    "status": "default",
                    "aqi": "41",
                    "weather": "晴",
                    "minTemp": "18",
                    "date": "08-13",
                    "windDir": "西北风"
                }, {
                    "weatherBg": "clear",
                    "maxTemp": "31",
                    "imgUrl": "http://onebox.oss.aliyuncs.com/img/weather/sun_48_48.png",
                    "wind": "3-4级",
                    "unit": "C",
                    "weekDay": "周日",
                    "status": "default",
                    "aqi": "41",
                    "weather": "晴",
                    "minTemp": "18",
                    "date": "08-14",
                    "windDir": "西北风"
                }, {
                    "weatherBg": "cloudy",
                    "maxTemp": "31",
                    "imgUrl": "http://onebox.oss.aliyuncs.com/img/weather/cloudy_48_48.png",
                    "wind": "3-4级",
                    "unit": "C",
                    "weekDay": "周一",
                    "status": "default",
                    "aqi": "41",
                    "weather": "多云",
                    "minTemp": "18",
                    "date": "08-15",
                    "windDir": "西风"
                }],
                "yesterday": {
                    "weatherBg": "clear",
                    "maxTemp": "39",
                    "imgUrl": "http://onebox.oss.aliyuncs.com/img/weather/sun_48_48.png",
                    "wind": "3-4级",
                    "unit": "C",
                    "weekDay": "昨天",
                    "status": "default",
                    "aqi": "91",
                    "weather": "晴",
                    "minTemp": "23",
                    "date": "08-11",
                    "windDir": "西北风"
                }
            },
            "type": "weather_one"
        }],
        "semantic": {
            "action": "com.mobvoi.semantic.action.WEATHER",
            "extras": {}
        },
        "confidence": 0.8,
        "taskName": "查天气",
        "dataSummary": {
            "title": "科尔沁右翼中旗 今天 多云转晴 PM2.5 77",
            "hint": "20-33°",
            "type": "weather_one"
        }
    },
    "status": "success"
}
```

## 搜索领域

| 垂直领域               	| Onebox        	| 查询示例                                     	|
|------------------------	|---------------	|----------------------------------------------	|
| 无分类                 	| OTHER         	| 帮我解一个微分方程                           	|
| 天气                   	| WEATHER       	| 今天的天气怎么样（明天上海的呢）             	|
|                        	|               	| 北京明天的天气                               	|
|                        	|               	| 今天的空气质量                               	|
| 打电话                 	| CALL          	| 打电话给李志飞                               	|
|                        	|               	| 打电话给10086                                	|
|                        	|               	| 呼叫13888888888                              	|
| 餐厅                   	| RESTAURANT    	| 附近的川菜馆                                 	|
|                        	|               	| 北京有什么好吃的                             	|
|                        	|               	| 帮我找一下附近人均100元左右的川菜馆，带WI-FI 	|
| 预设答案的查询         	| FAQ           	| 你好问问，你是谁                             	|
| 附近POI地点            	| POI           	| 附近的厕所/酒店/银行                         	|
| 导航                   	| NAVI          	| 帮我导航到北京大学东门                       	|
| 打车                   	| TAXI          	| 我要打车/打车去***/帮我打一个滴滴出租车      	|
| 设置提醒               	| REMINDER      	| 提醒我5分钟后喝水/提醒我晚上8点给家里打电话  	|
| 百度百科               	| BAIKE         	| 刘德华是谁                                   	|
|                        	|               	| 张碧晨的百科                                 	|
|                        	|               	| 习近平的资料                                 	|
| 火车                   	| TRAIN         	| 帮我查一下明天到上海的火车                   	|
| 查询地址               	| WHERE         	| 我在哪                                       	|
| 搜电影                 	| CINEMA        	| 最近有什么好看的电影                         	|
| 查新闻                 	| NEWS          	| 今天有什么新闻                               	|
|                        	|               	| 今天的体育新闻                               	|
|                        	|               	| 热门新闻                                     	|
|                        	|               	| 刘德华的相关新闻                             	|
| 音乐                   	| MUSIC         	| 我要听刘德华的歌                             	|
|                        	|               	| 来首邓紫棋的泡沫                             	|
| 控制手机（开启蓝牙等） 	| CONTROL       	| 关机，重启，重置系统                         	|
| 查日历（农历）         	| CALENDAR      	| 今天是农历几号/今天是农历初几                	|
| 查航班                 	| FLIGHT        	| 帮我查一下明天到上海的航班                   	|
| 查美女帅哥             	| BEAUTY        	| 附近的美女                                   	|
|                        	|               	| 附近的帅哥                                   	|
| 问笑话                 	| JOKE          	| 给我讲一个笑话                               	|
| 软件                   	| APP           	| 帮我下载一个微信                             	|
| 搜酒店                 	| HOTEL         	| 附近的汉庭酒店                               	|
| 百度问答知道           	| WENDA         	| 小孩子发烧怎么办                             	|
|                        	|               	| 肚子疼怎么办                                 	|
| 黄页                   	| YELLOWPAGE    	| 北京工商局的电话                             	|
| 购物 （taobao)         	| SHOPPING      	| 我要买裙子                                   	|
|                        	|               	| 什么值得买                                   	|
| 问时间                 	| TIME          	| 美国时间现在是几点钟                         	|
|                        	|               	| 现在几点                                     	|
|                        	|               	| 纽约现在几点了                               	|
| 发短信                 	| SMS           	| 发短信给***                                  	|
| 影视                   	| VIDEO         	| 太阳的后裔                                   	|
| 计算器                 	| CAL           	| 5+8等于几                                    	|
| 查地铁                 	| SUBWAY        	| 帮我查一下地铁10号线的首末班时间             	|
| 翻译                   	| TRANSLATE     	| 我爱你用韩语怎么说                           	|
| 查公交                 	| BUS           	| 帮我查一下365路公交                          	|
|                        	|               	| 帮我查一下附近的公交站                       	|
| 查星座                 	| CONSTELLATION 	| 今天出生的是什么星座                         	|
| 找文章小说             	| READING       	| 帮我查一下琅琊榜小说                         	|
| 查路况                 	| ROADTRAFFIC   	| 海淀区苏州街的路况                           	|
| 查股票                 	| STOCK         	| 百度的股价                                   	|
| 游记攻略               	| TRAVELGUIDE   	| 北京旅游攻略                                 	|
| 赛事                   	| GAMESCORE     	| 欧洲杯比赛情况                               	|
|                        	|               	| 2016年奥运会                                 	|
| 查电视节目             	| TV            	| 极限挑战                                     	|
| 查快递                 	| DELIVERY      	| 帮我查一下我的快递                           	|
|                        	|               	| 我要寄快递                                   	|
| 搜图片                 	| PICTURE       	| 帮我搜一下狗狗的图片                         	|
| 查网站                 	| WEBSITE       	| 打开百度                                     	|
| 查菜单                 	| RECIPE        	| 红烧肉怎么做                                 	|
| 找工作                 	| JOB           	| 我要找工作                                   	|
| 租房                   	| HOUSING       	| 我要租房                                     	|
| 生活服务               	| LIFESERVICE   	| 帮我找一个保洁                               	|
| 查线下活动             	| LOCALEVENT    	| 北京今天有什么线下活动                       	|
| 查团购                 	| GROUPON       	| 旅游团购                                     	|
| 查找优惠               	| COUPON        	| 最近有什么优惠信息                           	|
| 航班实时信息           	| FLIGHT-INFO   	| 帮我查一下海南航空HU7609起飞了吗             	|
| 设置闹钟               	| ALARM         	| 设置一个10分钟后的闹钟/设置一个9点的闹钟     	|
| 倒计时                 	| COUNTDOWN     	| 设置一个5分钟的倒计时                        	|
| 日程                   	| AGENDA        	| 提醒我晚上9点给家里打电话                    	|
| 查彩票                 	| LOTTERY       	| 大乐透中奖信息                               	|
| 查违章                 	| VIOLATION     	| 帮我查一下我的违章信息                       	|
| 找代驾                 	| DAIJIA        	| 帮我找一个代驾                               	|
| 查询帮助               	| HELP          	| 帮助                                         	|
| 单位换算               	|               	| 1美元等于多少人民币                          	|
|                        	|               	| 1公里等于多少米                              	|

# 错误码

| 错误码 	| 描述             	|
|--------	|------------------	|
| 0      	| 语音服务器错误   	|
| 1      	| 网络错误         	|
| 2      	| 无网络           	|
| 3      	| 录音设备错误     	|
| 4      	| 识别内容为空     	|
| 5      	| 输入语音过长     	|
| 6      	| 起始静音时间过长 	|
| 7      	| 网络太慢         |
 

