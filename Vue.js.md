[来自B站视频][https://www.bilibili.com/video/av76249419?p=37]

# 一、VUE基础

需要先了解HTML，CSS，JavaScript，AJAX等知识。

## 1、导入VUE

```HTML
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```

## 2、声明式渲染

Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统：

```vue
<div id="app">
  {{ message }}
</div>

<script src="../src/vue.js"></script>
<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello  Vue!'
  }
})
</script>
```



## el：挂载点

VUE的作用范围是el选项命中的元素及其内部的后代元素;

VUE不能挂载在<html>和<body>标签上；

## data：数据对象

VUE中用到的数据定义在data中

data中可以写复杂类型的数据

渲染复杂类型数据时，遵循JS的语法

```vue
<div id="app">
			<h2> {{hello}}</h2>
			<p>{{school.location}} {{school.name}} {{school.department[0]}} {{hello}} </p>
		</div>

		<script src="../src/vue.js"></script>
		<script>
			var app = new Vue({
				el: "#app",
				data: {
					hello: "hello",
					school: {
						name: "运城学院",
						location: "山西运城",
						department: ["数信学院", "机电系"],
					},
				}
			})
            </script>
```

# 二、本地应用

## 1、内容绑定

### **v-text** 指令

**v-text** 指令等同于{{ message}}

**v-text** 指令作用是设置标签的内容(textContent)；

默认的写法会替换全部的内容，使用差值表达式{{}}可以替换制定的内容

{{ message +"！" }}



```vue
<div id="app">
			<p v-text="message"></p>
			<p> {{message }}</p>
</div>
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			message: "666",
		},
	})
```

### **v-html**指令

**v-html**指令的作用是：设置元素的innerHTML，内容中有HTML结构会被解析为标签。

```vue
<div id="app">
	<p v-text="message"></p>
	<p> {{message }}</p>
	<!-- 上面两个输出文本 -->
	<p v-html="message"></p>
	<!-- 这个输出一个超链接 -->
</div>
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			message: "<a href='http://www.ycu.edu.cn/'> YCU</a>",
				},
	})
</script>
```

### v-on指令

**v-on**指令：是为元素绑定事件，事件名不需要再写on；

**v-on**指令可以简写为**@**；

绑定的方法定义再methods属性中

```vue
<div id="app">
	<p @click="changeFood">{{  food  }}</p>
	<input type="button" value="单击事件" v-on:click="addFood">
	<input type="button" value="单击简写" @click="addFood">
	<input type="button" value="双击事件" @dblclick="addFood">
</div>
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			food:"西红柿炒蛋"
		},
		methods:{
			changeFood:function(){
				this.food="蛋炒西红柿"
				},
			addFood:function(){
				this.food+="好吃！"
			},
		},
			
	})
</script>
```



### 计数器例子

```vue
<div id="app">
	<button type="button" @click="sub">-</button>
	<span>{{num}}</span>
	<button type="button" @click="add">+</button>
</div>
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			num: 1
		},
		methods: {
			sub: function() {
				if (this.num <= 0) {
					alert("已达到最小值");
				} else {
					this.num--;
				}
			},
			add: function() {
				if (this.num >= 10) {
					alert("已达到最大值");
				} else {
					this.num++;
				}
			}
		},
	})
</script>
```



## 2、显示切换、内容绑定

### v-show指令

**v-show**指令的作用是：根据真假切换元素的显示状态；

**v-show**指令的原理是：修改元素的display，实现显示隐藏；

指令后面的内容最终都会解析为布尔值，ture元素显示，false元素隐藏；

```html
<div id="app">
	<div>
		<button type="button" @click="changeIsShow">changeIsShow</button>
		<button type="button" @click="addAge">addAge</button>
	</div>
	<p>{{  "new age :"+ age}}</p>
	<div>
		<img src="../src/img/logo.png" alt="" v-show="isShow">
		<!-- 用v-show的false确定是否显示图片 -->
		<img src="../src/assets/logo.png" v-show="age>=6" >
		<!-- 用判断语句确定是否显示图片 -->
	</div>
</div>
```

```vue
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			isShow:false,
			age:3
		},
		methods:{
            //写一个方法改变isShow的属性
			changeIsShow:function(){
				this.isShow = !this.isShow;
			},
            //写一个方法改变age的属性
			addAge:function(){
				this.age+=1;
			},
				
		}
	})
</script>
```

### v-if指令

用法于v-show大同小异；

**v-if**指令的作用是：根据表达式的真假切换元素的显示状态；

本质是通过操作dom元素来切换显示状态，

表达式的值为ture，元素存在于dom树中，为false，从dom树中移除；

数据改变后，对应元素的显示状态会同步更新；

### v-bind指令

**v-bind**指令的作用是：为元素绑定属性；

完整的写法是v-bind:属性名；

简写可以直接省略v-bind，只保留:属性名；

需要动态增删class 建议使用对象的方式；

```css
<style type="text/css">
	.active{
		border: 1px solid red;
	}
</style>
```



```html
<div id="app">
	
	<!-- <img v-bind:src="imgSrc" > v-bind可以简写成冒号: --> 
	<img :src="logoSrc" alt="" :title="logoTitle+'好厉害'":class="{active:isActive}" 			@click="changeActive">
</div>
```

**v-bind** 可以简写为冒号：  

```vue
<script>
	var app = new Vue({
		el: "#app",
		data: {
			logoSrc:"http://www.ycu.edu.cn/style/logo.png",
			logoTitle:"运城学院",
			isActive:false
		},
		methods:{
			changeActive:function(){
				this.isActive = !this.isActive;
			}
		}
	})
</script>
```

## 3、列表循环，表单元素绑定

### v-for指令

**v-for**指令的作用是根据数据生成列表结构，经常与数组一起使用；

语法:

```vue
v-for="(item,index) in 数据"
```

数组长度的更新会同步到页面上，是响应式的;



```HTML
<div id="app">
	<ul>
		<li v-for="i in arr">
			{{ "运城学院有"+ i }}
		</li>
		<br>
		<li v-for="(i,j) in lession">
			{{ j+1 +"  "+i.name +"在"+i.time+"上课"}}
		</li>
	</ul>
</div>
```



```VUE
<script>
	var app = new Vue({
		el: "#app",
		data: {
			arr: ["数信学院", "机电系", "中文系", "政法系"],
			lession: [{
					name: "web前端",
					time: "周二上午"
				},
				{
					name: "vue开发",
					time: "周三上午"
				}
			]
		},
	})
</script>
```



### v-on指令补充

[v-on API][https://cn.vuejs.org/v2/api/#v-on]

事件绑定的方法写成函数调用的形式，可以传入指定因参数

定义方法时需要定义形参来接受传入的实参；

事件的后面跟上.修饰符 可以对事件进行限制，.enter 可以限制触发的按钮为回车，事件修饰符有多种

```html
<div id="app">
	<input type="button" @click="message(1)" value="加1" />
	<input type="text" @keyup.enter="message(2)" value="2" />
</div>
```



```vue
<script>
	var app = new Vue({
		el: "#app",
		data: {
			num:1,
			currentNum:10,
		},
		methods:{
			message:function(i){
				this.num+=i;
				console.log(this.num)
			}
		}
	})
</script>
```







### v-model指令

**v-model指令**指令的作用是便捷的设置和获取表单元素的值；

绑定的数据会和表单元素值相关联；

双向绑定 数据与表单元素的值；改变一个两个都会发生变化；



```vue
<div id="app">
	<input type="text" v-model="message" />
	{{ message }}
</div>
<script src="../src/vue.js"></script>
<script>
	var app = new Vue({
		el: "#app",
		data: {
			message:"运城学院",
		},
	})
</script>
```

### todolist例子

```vue
<body>
		<div id="app">
			<!-- 输入框用v-model连接data @keyup.enter 输入回车执行函数-->
			<input type="text" v-model="addMessage" @keyup.enter="addArr" />
			
			<!-- 表格内容 -->
			<div class="form">
				<ul>
					<!-- 用f-for显示所有的objArr -->
					<li v-for="(i,j) in objArr">
						<div class="view">
							<!-- v-text的缩写形式 -->
							<span class="index">{{j+1 +"."}}</span>
							<label>{{ i }}</label>
							<!-- 一个控制删除的按钮，传入一个当前index参数， -->
							<button type="button" class="deleMessage" @click="deleMessage(j)"> X </button>
						</div>
					</li>
				</ul>
			</div>
			
			<!-- 底部显示总数 v-show如果数组长度等于0就隐藏-->
			<div class="bottom" v-show="objArr.length!=0">
				<strong>{{ objArr.length }}</strong> items left
				<!-- 一个清空按钮 -->
				<button type="button" @click="clearAll">clear all</button>
			</div>
		</div>
		
    
    	<script src="../src/vue.js"></script>
		<script>
			var app = new Vue({
				el: "#app",
				data: {
					addMessage: "运城学院",
					objArr: ["数信学院", "政法系"，"机电系"],
				},
				methods: {
					addArr: function() {
						this.objArr.push(this.addMessage);
					},
					deleMessage: function(i) {
						// objArr.split(i,j),从i开始删除j个数据
						this.objArr.splice(i, 1);
					},
					clearAll: function() {
						//清空不需要删除,直接等于空就行了
						this.objArr = [];
					}
				}
			})
		</script>
	</body>
```





## 三、网络应用

### 1、axios 网络请求库

[axiosGithub文档][https://github.com/axios/axios]

**axios**的get与post指令的使用方法；

```js
axios.get(地址?key=value&key2=value2).then(function(response){},function(err){})

axios.post(地址,{key:value&key2:value2}).then(function(response){},function(err){})
//then后第二个函数是回调函数，可以获取响应内容，或错误信息
```

导入：

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

实例

```html
<body>
	<div id="app">
		<input type="button" value="get" class="get">
		<input type="button" value="post" class="post">
	</div>
	<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
	<script>
		// 接口1:随机笑话,
		// 请求地址:https://autumnfish.cn/api/joke/list
		// 请求方法:get
		// 请求参数:num(笑话条数,数字)
		// 响应内容:随机笑话,
        //querySelector()方法返回文档中匹配指定 CSS 选择器的一个元素。
		document.querySelector(".get").onclick = function() {
			axios.get("https://autumnfish.cn/api/joke/list?num=3").then(function(respones) {
				console.log(respones)
			}, function(err) {
				console.log(err)
			})
		}
		//接口2:用户注册
		// 请求地址:https://autumnfish.cn/api/user/reg
		// 请求方式:post
		// 请求参数:username 
		// 响应内容:注册成功或者失败
		document.querySelector(".post").onclick = function() {
			axios.post("https://autumnfish.cn/api/user/reg", {
				username: "tom"
			}).then(function(respone) {
				console.log(respone);
			}, function(err) {
				console.log(err);
			})
		}
	</script>
</body>
```

### 2、axios+vue

**axios**回调函数中的this已经改变，无法访问到vuedata的数据，可以把this保存起来，回调函数中直接使用保存过的this参数即可；

与本地应用的最大区别就是改变的数据的来源；



```html
<body>
	<div id="app">
		<input type="button" value="获取一条笑话" class="get" @click="getJoke">
		{{ joke }}
	</div>
	<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
	<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js">
	</script>
	<script>
		// 接口1:随机笑话,
		// 请求地址:https://autumnfish.cn/api/joke
		// 请求方法:get
		// 请求参数:num(笑话条数,数字)
		// 响应内容:随机笑话,
		var Vue = new Vue({
			el: "#app",
			data: {
				joke: "",
			},
			methods: {
				getJoke: function() {
					console.log(this.joke) //rerurn 有数据
					var that = this;	//内外this不一样
					axios.get("https://autumnfish.cn/api/joke").then(function(respones) {
						console.log(this.joke) //undefine 内外两个this不一样
						that.joke=respones.data;
					}, function(err) {
						console.log(err)
					})
				},
			}
		})
	</script>
</body>
```



### 3、天气查询实例





```html
<div id="app">
    <!-- 放个logo -->
	<div style="text-align: center;"> <img src="../src/img/img0.png" ></div>
    
    <!-- 主界面 -->
	<div style="text-align: center;" class="main">
        <!-- v-model 绑定元素 -->
		<input  v-model="location" placeholder="请输入要查询的城市" @keyup.enter="search">
		<input type="button" value="查询"  @click="search">
	</div>
	<div class="forecast">
		<ul>	<!-- v-for 输出数组 -->
			<li v-for="(forecast,j) in forecast">
				<div><span>{{forecast.type}}</span></div>
				<div> <b>{{forecast.high}}~{{forecast.low}}</b></div>
				<div><span>{{forecast.date}}</span></div>
				<br>
			</li>
		</ul>
	</div>
</div>
```



```vue
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>
	var Vue = new Vue({
		el: "#app",
		data: {
            //一个获取输入框，一个获取网络的数据；
			location: "运城",
			forecast: [],
		},
		methods: {
			search: function() {
				var that = this;
				axios.get('https://wthrcdn.etouch.cn/weather_mini?city=' + this.location).then(function(respones) {
					console.log(respones.data.data.forecast);
					that.forecast = respones.data.data.forecast;
				}, function(err) {
					console.log(err)
				})
			},
		}
	})
</script>
```





# 四、综合应用

### 音乐播放器例子

[API文件][https://github.com/Binaryify/NeteaseCloudMusicApi]

服务器返回的数据比较复杂时，获取的时候需要注意层级结构

```html
<!DOCTYPE html>
<html lang="en">

	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width,initial-scale=1.0">
		<title>test1</title>
	</head>
	<body>
		<div id="app">

			<div class="searchBar">
				<input type="text" v-model="query" @keyup.enter="searchMusic()" />
				<input type="button" value="搜索" @click="searchMusic()" />
			</div>

			<div class="video" v-show="videoUrl!='' ">
				<video width="800" height="" :src="videoUrl" controls="controls">

				</video>
			</div>

			<div class="audio" v-show="MusicUrl!='' ">
				<audio ref="audio" :src="MusicUrl" controls autpplay loop class="controls">
					当前浏览器不支持audio
				</audio>
			</div>

			<div class="photo" v-show="imgUrl!='' ">
				<img :src="imgUrl">
			</div>

			<div class="hotComment" v-show="hotComments.length!=0">
				<ul>
					<p>热门评论:</p>
					<li v-for="(item,j) in hotComments">
						{{j+1}}
						<img :src="item.user.avatarUrl" height="100" width="100"> <br>
						{{item.user.nickname}} : {{item.content}} <br>
					</li>
				</ul>
			</div>

			<div class="songWrapper">
				<ul>
					<li v-for="item in musicList">
						{{item.name}}
						<a @click="playMusic(item.id)">播放</a>
						<a v-if="item.mvid!=0" @click="playMV(item.mvid)"> 播放mv</a>
					</li>
				</ul>
			</div>


		</div>
		<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
		<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
		<script src="../src/main.js"></script>
	</body>
</html>

```



```js
// 请求地址：https://autumnfish.cn/search
// 请求方法:get
// 请求关键字:keywords 
// 响应内容:歌曲搜索结果

// 请求地址：https://autumnfish.cn/song/url
// 请求方法:get
// 请求关键字:id(歌曲ID)
// 响应内容:歌曲的url地址

// 请求地址：https://autumnfish.cn/song/detail
// 请求方法:get
// 请求关键字:ids(歌曲ID)
// 响应内容:歌曲详情 封面信息

// 请求地址：https://autumnfish.cn/mv/url
// 请求方法:get
// 请求关键字:id(mvid,为0说明没有mv)
// 响应内容：mv的地址
var Vue = new Vue({
	el: "#app",
	data: {
		query: "",
		musicList: [],
		MusicUrl: "",
		imgUrl: "",
		hotComments: [],
		videoUrl: "",
	},
	methods: {
		//搜索音乐功能
		searchMusic: function() {
			this.MusicUrl = "";
			this.hotComments = [];
			this.imgUrl = "";
			this.videoUrl = "";
			var that = this;
			axios.get("https://autumnfish.cn/search?keywords=" + this.query)
				.then(function(response) {
					console.log(response)
					that.musicList = response.data.result.songs;
				}, function(err) {
					console.log(err);
				})
		},
		playMusic: function(MusicId) {
			var that = this;

			//获取MusicUrl
			axios.get("https://autumnfish.cn/song/url?id=" + MusicId)
				.then(function(response) {
					that.MusicUrl = response.data.data[0].url;
				}, function(err) {
					console.log(err);
				})

			//获取封面imgUrl
			axios.get("https://autumnfish.cn/song/detail?ids=" + MusicId)
				.then(function(response) {
					that.imgUrl = response.data.songs[0].al.picUrl;
				}, function(err) {
					console.log(err);
				})

			//获取hotComments
			axios.get("https://autumnfish.cn/comment/hot?id=" + MusicId + "&type=0")
				.then(function(response) {
					that.hotComments = response.data.hotComments;
					console.log(response);
				}, function(err) {
					console.log(err);
				})
		},
		playMV: function(mvid) {
			var that = this;
			axios.get("https://autumnfish.cn/mv/url?id=" + mvid)
				.then(function(response) {
					that.videoUrl = response.data.data.url;
				}, function(err) {
					console.log(err)
				})
		}
	}

})

```

