# HHP（欢网 Hybrid Plugin）
一款基于Android平台开发的热更新热加载框架，支持js

关注 [欢视网](http://www.tvhuan.com/).

## Gradle 引用方式

```
1.Add it in your root build.gradle at the end of repositories （添加maven仓库到根目录下的build.gradle）:

allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}

2.Add the dependency（添加依赖到app下的build.gradle）

dependencies {
	...
	compile 'com.github.xuehuiniaoyu:HHybridPlugin:1.1.2'
}

```

classes.jar 可以直接引用到项目中，解决gradle不能使用provided的bug。


## 产品定位及功能介绍

	HHybridPlugin（HHP）是一款Android应用层的热更新框架。

	优点：

	1.支持动态UI布局。
	2.支持动态解析布局。
	3.支持布局中内嵌js脚本。
	4.配置文件灵活管理项目。
	5.本地缓存可以设置有效时间，过期后自动删除。
	6.支持自定义组件等。
	7.插件支持.so库文件。
	8.主要使用java classLoader和reflect技术实现，是android应用层的扩展。

	缺点：

	定制性交高，工程和插件必须使用相同版本进行约束。

## app-config 核心配置文件
	
	一个HHP项目都会有一个最顶级的配置文件 app_config.xml(名称可自定义，结构必须遵守规范，参见：hhp.dtd)

## 配置文件

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE app-config SYSTEM "http://com.huan.hhp">
<app-config package="com.huan.appstore" cacheable="true">
    <persistence-resources>
        <resource name="project" value="http://172.10.10.211:8080/testhhp" />
		<resource name="main" value="/main.xml" ver="1.1" keepMilliseconds="2000" />
    </persistence-resources>
    <resources>
        <resource name="def-application" value="com.huan.hhp.PluginApplication" />
        <resource name="def-activity" value="com.huan.hhp.SimplePluginActivity" />
    </resources>
    <plugin-list>
        <plugin id="app-1" name="appName-1" project="{project}" file="{?}" isMain="true">
            <application class="{def-application}">
                <activity name="main" class="{def-activity}" layout="{main}" isMain="true">
			<intent key1="value1" key2="value2" />
		</activity>
            </application>
        </plugin>
    </plugin-list>
    <clear ids=""></clear>
</app-config>
```

* `package` 包名，全局唯一 用来区分工作空间。因为一个项目中可能有多个同样的配置文件。
* `cacheable` 是否将配置文件缓存到本地。

-----------------------------------------------------------
* `<persistence-resources>` 持久化资源池。
* `<resources>` 非持久化资源池。
* `<resource>` 资源。键值的形式出现，`name`是资源的引用标签，`value`是资源的内容。`ver`为资源的版本`<persistence-resources>`时生效。
`keepMilliseconds="2000"` 表示该资源会在退出2000毫秒后删除本地缓存。

-----------------------------------------------------------
* `<plugin-list>` 表示插件池，其中可能会出现多个插件即`<plugin>`
* `<plugin>` 插件，其中 `id="app-1"` app-1是该插件的唯一id，`name="appName-1"` appName-1 是插件的名称，也就是插件的key 也可以写成 `name="{appName-1}"`
这样的话 appName-1 就是资源(resource)的name。

* `project` 项目更目录
* `file` 插件压缩文件所在目录 `file="{?}"` 表示没有压缩文件。
* `isMain="true"` 表示第一次运行的插件。 	

-----------------------------------------------------------
* `<application class="{def-application}">` 插件运行时。如果是自定义类需要继承：`com.huan.hhp.PluginApplication` , 否则直接用 `com.huan.hhp.PluginApplication` 。

-------------------------------------
* `<activity name="main" class="{def-activity}" layout="{main}" isMain="true">` `class` 是activity的类名需继承`com.huan.hhp.PluginActivity`或`com.huan.hhp.SimplePluginActivity`
`layout` 是布局文件 如：/main.xml

* `<intent key1="value1" key2="value2" />` 打开activity时带的一些参数.

```
获取方式为：

Intent intent = getIntent();
String value1 = intent.getStringExtra("key1");
String value2 = intent.getStringExtra("key2");

```

## 布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView>
	<LinearLayout id="aaaa" background="#454545" orientation="vertical">

		<script>
			function onKeyDown(keyCode){
				if(keyCode == 21 || keyCode == H.getEventCode("menu")){
					H.startActivity("com.huan.appstore.MainActivity");
					return true;
				}
				return false;
			}
		</script>


		<RelativeLayout width="fill_parent" height="wrap_content">
			<Button id="button1" text="hello world" background="#cc0000" />
			<Button id="button2" text="hello world1" layout_below="button1" background="#cccccc" />

			<TextView text="hello world" layout_centerInParent="true" background="#cccccc" />
		</RelativeLayout>

	</LinearLayout>
</ScrollView>
```
布局文件和普通的android布局是几乎一样的，除了可以省略 `android:` 和支持了一些扩展属性之外还支持了 `<script>` 标签。

## 让Activity具备js能力

* `context` 是上下文， 可以通过 `context.findViewById('button1'); ` 来获取View
* `H` 上工具类

## Script介绍

* 第一种

```
<script>
	// 初始化方法
	function onReady(){
		ver button1 = context.findViewById('button1')；
		button1.setText("HHP-Button1")；
	}

	// 界面被唤醒
	function onResume(){
	
	}

	// 界面被暂停
	function onPause(){
	
	}

	function onKeyDown(keyCode){
		if(keyCode == H.getEventCode("left")){
			H.alert("按键为左键")
		}
	}

	function onKeyUp(keyCode){
	}
</script>
```


`H 工具类提供的键值有:`
```
left   左
up     上
right  右
down   下
center OK
enter  确认/回车
back   返回
menu   菜单

使用：function onKeyDown(keyCode) {
	if(keyCode == "left")

      }

```
	

* 第二种

```
<script src="/index.js" />
```
		

## ajax

```
H.ajax({
	method:'get',
	url:'http://com.huan.tv',
	body:function(){
		return "body";
	},
	contentType:"application/json"
	
}, 

{
	success:function（data）{

	},

	error:function(data){

	}
});

```

## 异步任务

```
H.asyncTask({
	run:function(arg1, arg2){
		// 一大堆代码

		return arg1+" "+arg2;
	}
}, 

{"hello world", "HHP"}, 

{
	run:function(obj){
		H.alter("执行完返回的结果是："+obj);
	}
});
```

## 循环调用之 loop

```
function timer(){

}

H.loop("timer", 1000, -1); // 无限次循环
```

```
H.loop("timer", 1000, 3); // 3次循环
```

```
function timer(arg1, arg2 ...){

}

H.loop("timer", 1000, -1, arg1, arg2 ...)
```


## 停止循环调用

```
H.stopLoop("timer");
```

## 获取分辨率显示单位

```
var width = H.rso(1024);
H.alter("width="+width);
```

## 弹框 alert

```
H.alert("hello world");
```

## 弹层 toast

```
H.toast("hello world");
```

## Log 记录行为

var TAG = "LOG_TAG-Main";

* H.logI(TAG, "log ...")；
* H.logD(TAG, "log ...")；
* H.logE(TAG, "log ...")；

## 输出到控制台

* H.print("hello world");
* H.println("hello world");

## 动画缩放

```
var view1 = context.findViewById("view1");

// 放大动画
H.scale(view1, 1.1f, 200, {
	begin: function(){
		// 开始
	},

	end: function（）{
		// 结束
	}
}

// 缩小动画
H.shrink(view1, 1.1f, 200, {
	begin: function(){
		// 开始
	},

	end: function（）{
		// 结束
	}
}
```

## 自定义动画

```
var view1 = context.findViewById("view1");
var animUtil = H.createAnimUtil(view1);
animUtil.layout(1, 10, 100, 100);

animUtil.animate(1, 10, 100, 100, {
	end: function(){
		H.alert("over!");

		animUtil.animate(1, 10, 200, 200);
	}
});

```

## 获取文件MD5

```
var md5 = H.getFileMD5("/data/data/com.example.TestHHP/cache/a.txt");
```

## 启动Activity

```
H.startActivity("main_layout") main_layout是配置文件中的name

H.startActivity("com.example.aaa.ui.MainActivity") // 跳转到系统Activity

H.startActivity("com_huan_action") // 跳转到action
```

## 获取插件

```
var plugin = H.getPluginInfoByName("plugin1");
H.alert(plugin.getName());
H.runPlugin(plugin); // 在新窗口中运行插件
```

## 获取插件的workspace

```
var workspace = H.getPluginWorkspace("plugin1");
```

## 获取资源

```
var resource = H.getResource("resource1");
var name = resource.getName();
var value = resource.getValue();
```


## 关于自动化配置

HHPlugin.jar 根目录有一个文件 `hhp.dtd` 如果你是用IDEA或Eclipse开发，请先安装dtd文件。

比如说域名为 http://com.huan.hhp 那么使用的时候就如下：

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE app-config SYSTEM "http://com.huan.hhp">
<app-config>
</app-config>
```
工具都会带有自动提示功能。


## 自定义控件 ViewMapping

ViewMapping 是View的映射，举个栗子：


```
public class TextViewMapping extends ViewMapping {
    // 把属性映射到方法
    {
        mapping("text", new StringTypeOf("setText", CharSequence.class));
        mapping("textSize", new DimenTypeOf("setTextSize", float.class));
        mapping("textColor", new TypeOf("this.setTextColor", String.class));
    }

    ...
}

```

布局的时候就可以写么写：

```
<TextView text="hello" textSize="12sp" textColor="#cc0000" />

```


那么问题来了！
这些属性是如何被解析呢？



```
protected void mapping(String from, TypeOf to){
	this.put(from, to);
}

```

看过源代码中mapping方法就会发现，这是一个以键值形式作为映射的。第一个参数form是键也就是属性名，第二个参数to是值，也就是属性对应的内容。比如：
name="张三"

TypeOf又是个什么东西呢？
继续往下看


```
public TypeOf(String name, Class<?> type) {
	this.name = name;
	this.type = type;
}

```

还是键值形式，第一个参数name是java类的方法名或属性名，第二个参数type是方法对应的参数类型，目前版本之提供了接收一个参数。
那么前面说了，name是java类的一个方法，这个java类到底怎么确定？
用两个例子说明：
mapping("textSize", new DimenTypeOf("setTextSize", float.class));
mapping("textColor", new TypeOf("this.setTextColor", String.class));

这是两个行映射，我们可以发现唯一的区别是第二条比第一条在TypeOf参数中，多了一个`this.`
`this.` 代表从当前类取方法；如果没有`this.` 就从mView中取方法。（mView）是当前类映射的 Android控件

当然你也可以 this.mView.setText 这样也是可以得，你有多少个对象你就可以无限的 `.` 下去。


说完方法映射，再来说下如何绑定View给当前类。



```
public class TabPagerMapping extends ViewGroupMapping {
    {
        forTag(TabPager.class.getName());
    }

    public TabPagerMapping(Context context, String name) {
        super(context, name);
    }
}

```

forTag 后 mView 对象就被创建。


最后看下整个自定义View的步骤：

1. 创建 ViewMapping

```
public class TabPagerMapping extends ViewGroupMapping {
    {
        forTag(TabPager.class.getName());
    }

    public TabPagerMapping(Context context, String name) {
        super(context, name);
    }
}

```

forTag 传递的class 是Android原生View

2. 注册ViewMapping
在Application onCreate 方法中添加以下代码

// 这里把该注册的类的注册了, 注意：这里注册时是Mapping 而不是 View
HwMappings.getSingleInstance().addMappingReference("TabPager", TabPagerMapping.class.getName());

这样就完成了映射，注意Application一定是PluginApplication的子类。

3. 在布局中使用


```
<TabPager />

```

记住这里是 TabPager 不是 TabPagerMapping


好了，至此一个自定义View就完成了。是不是很方便？