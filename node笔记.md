

## fs模块

```javascript
const fs = require('fs');//引入fs模块  file system(文件系统) fs模块是node内置的核心模块

	1.fs.readFile(文件的路径,回调函数) //回调函数是异步的
            回调函数两个参数
                err  //如果文件的路径不存在 则err就是一个错误对象
                data //如果文件的路径存在 则data就是读取出的数据  数据是Buffer格式的
                Buffer格式的数据可以使用 toString() //转成字符串

    2.fs.readFileSync(文件的路径)  //这个函数是同步的 函数的返回值是读取文件的数据

    3.fs.writeFile(文件的路径,数据,回调函数)
    
    4.fs.writeFileSync(文件的路径,数据)//异步

    5.fs.unlinkSync(路径)//删除文件
    
    6.fs.mkdirSync(路径)	//创建新的空文件夹

    7.fs.rmdirSync(路径)//删除指定的文件夹
   
    8.fs.statSync(路径)//查看文件/文件夹的状态

    9.fs.existsSync(路径)//判断文件是否存在

    10.fs.readdir(路径,回调函数)//读取指定文件夹的数据 

    11.fs.rename(老路径,新路径,回调函数)//把原来的文件从老路径移到新路径 并且修改文件名

    12.fs.watchFile(文件的路径,监听的频率,回调函数)//只要文件的状态改变了就会触发回调函数  
         
        fs.watchFile('./1.txt',{interval:23},function(nextStat,preStat){
            console.log(nextStat.size,preStat.size) 
            //nextStat是改变后的文件状态 
            //preStat改变前的文件状态
        })
```

```javascript
const buf = Buffer.from('海文')
            //Buffer中数据的展示形式都是16进制的
            //buffer.length 表示的是buffer的字节长度
            //a的ascii码 97  97的16进制就是61
```

## path模块

```javascript
const path = require('path') //这个模块专门用来处理路径

1.path.parse(__filename)//序列化路径 返回一个对象

2.path.extname(__filename)//查看文件的后缀名

3.path.basename(__filename)//返回最后一个\后面的数据

4.path.dirname(__filename)//返回最后一个\前面的数据

5.path.join(__filename+'heaven','lalla'))
```

## 流式读取

```java
const rs = fs.createReadStream(路径) //创建一个读取指定文件的可读流

const ws = fs.createWriteStream(路径)//创建一个在指定文件写入的可写流

  //两种方法
1.可以通过rs.readableFlowing查看可读流的状态  默认为null 表示可读流是静止的

    console.log(rs.readableFlowing)
    rs.resume()//让可读流运动 还可以在回调函数中得到可读流读取的数据
    console.time(2)  //197  159  150
    rs.on('data',function(chunk){
        console.log(rs.readableFlowing,chunk.length)
        ws.write(chunk)
    })
    rs.on('end',function () {
        // console.timeEnd(2)
        console.log('数据已经读取完成了')
    })
    console.log(rs.readableFlowing)
    
  2.rs.pipe(ws)

```

## 在node中每个js文件都是一个模块 

```javascript
//每个js文件的所有代码都被一个立即执行函数包裹住  如何验证？？  arguments.callee+''
//立即执行函数的5个形参

     1.exports  //是一个对象 专门用来暴露模块的数据 本质上就是通过module.exports这个对象暴露数据的
    	 console.log(module.exports===exports);//true
     2.require   //函数类型  专门用来引入模块的
     
     3.module    // 模块对象
     
     4.__filename   //文件的绝对路径
  
     5.__dirname      //文件夹的绝对路径
  
  //模块的分类
      1. 系统模块 node内置的  比如 fs  path
      2.自定义模块 自己手写的模块
      3.第三方模块 别人写的模块（npm）
```

## http模块

```javascript
const http = require('http');//这个模块专门用来搭建服务 供客户端请求以及向客户端响应

//创建服务
//当客户端向服务端发起请求时 就会触发回调函数
      回调函数的参数
         req 请求体
         res 响应体
    var server = http.createServer(function(req,res){
        console.log('有人进来了')

        //保证响应的数据不乱码 需要设置响应的头部信息
        res.setHeader('content-type','text/html;charset=utf-8')

        res.write('我是第一波数据') ;//服务端向客户端响应数据
        res.write('我是第二波数据')
        res.write('我是第三波数据')
        res.end('haiwen')  //服务端的响应已经完毕了
    })
    //server.listen(端口号,回调函数) 端口号的范围是 [0,65536)  当3000端口处的服务成功开启时 会自动触发
        server.listen(3000,function(){
            console.log('3000端口成功运行')
        })

```

## http响应页面 获取get数据

```javascript
const url = require('url') //这个模块专门用来处理地址栏的信息
 https:  //www.baidu.com :8080   /p/a/t/h  ?  query=string   #hash
  协议      域名           端口    pathname      search       hash值
  
const {pathname,query} = url.parse(req.url,true) //获取get数据 第二个参数只改变quary 让其变成对象
```

## 获取post数据

```javascript
const qs = require('querystring') //专门处理user=qweqwe&psd=12312

const server = http.createServer(function(req,res){

    const {pathname} = url.parse(req.url,true)
    if(pathname==='/'){
        fs.readFile('./demo.html',function(err,data){
            if(err)return
            res.end(data)
        })
    }else if(pathname==='/heaven'){
        console.log('表单的数据接受到了')
        /*
        *前端的post数据存放在请求体中的 并且是分批次传输
        * */
        var str = ''
        req.on('data',function(chunk){
            str += chunk
        })

        req.on('end',function(){
            console.log(qs.parse(str));
            // console.log('post数据接受完毕')
            res.setHeader('content-type','text/html;charset=utf-8')
            res.end('数据响应完毕')
        })
    }else{
        res.end('404')
    }
})
server.listen(3001,()=>{
    console.log('3001端口已经启动了')
})
```

## cmd命令

###    	 d:  切到d盘目录

###    	 cd  --> change directory 跳转目录

###            cd ../  跳到上一级目录

###            cd /    跳到根目录

###    	mkdir -->  make directory 创建目录

###   	 rmdir -->  remove directory 删除目录

###  	cd.> 1.txt      创建1.txt文件

###   	 del 1.txt       删除1.txt文件

###    	1.txt       打开当前的目录中的1.txt文件

##    npm命令

###        		第三方模块：别人写的模块  npm  全称：node package manager（node的包管理器）

###        		什么是模块：一个js文件

###        		什么是包:加强版的模块

###        		开发环境：在开发阶段运行的代码    --save-dev  -D

###        		生产环境：当代码开发完毕 投放给用户使用时运行的代码  --save  -S  默认

###        		-g 全局安装 在任何目录下都可以使用cnpm指令 相当于环境变量

###  		npm init -y 初始化项目文件

###     		 npm install 包名      下载包

###        		npm uninstall 包名     删除包

###        		npm install     自动下载package.json文件中的依赖



## express 获取get数据 和post数据

```javascript
1.//获取get数据
    const express = require('express') //引入express包
    const app = express()
        //接受get方式发送的数据
            app.get('/heaven',function(req,res){
                // console.log('get请求过来了');
                //获取到get数据
                console.log(req.query)
            })

2.//获取post数据
    const express = require('express')
    const bodyParser = require('body-parser')
    const app = express()
    app.use(bodyParser.urlencoded({ extended: false }))//使用bodyParser

        //接受post方式发送的数据
            app.post('/heaven',function(req,res){
                console.log('post请求过来了');
                console.log(req.body)
                // res.end({name:'heaven'})

                //send函数是express新增的 这个方法可以发送数组和对象给前台
                res.send([1,2,3])
            })     

            //自动为public目录中的文件设置路由
            app.use(express.static('public'))
            app.listen(3000,()=>{
                console.log('3000端口成功运行');
            })

```

## 中间件洋葱模型

```javascript
//匹配到相同地址的时候 假如有多个函数 只会执行第一个 继续执行需要使用next函数
const express = require('express') //引入express包
const app = express()

    app.get('/',function(req,res,next){
        console.log(1)
        next()  //触发下个一个中间件函数
        console.log(2)
    })

    app.get('/',function(req,res,next){
        console.log(3)
        next()
        console.log(4)
    })
    app.listen(3003,()=>{
        console.log('3003端口成功运行');
    })
```

## 动态路由

```javascript
const express = require('express') //引入express包
const app = express()

    //前台的多个路径都可以匹配到后台的一个路径     :号后面是变量
    app.get('/article/:id/:xxx',function(req,res){
        // let {id} = req.params
        console.log(req.params)
        res.send(req.params)
    })

    //自动为public目录中的文件设置路由
    app.use(express.static('public'))
    app.listen(3002,()=>{
        console.log('3002端口成功运行');
    })
```

## ejs模板引擎

```javascript
const express = require('express') 
const ejs = require('ejs')  //引入ejs模快
const app = express()

app.set('view engine','ejs')//使用ejs模板引擎

app.get('/:number',function (req,res) {
    let {number} = req.params
    console.log(number);    //12
    let arr = []
    for(var i=1;i<=number;i++){
        if(number%i==0){
            arr.push(i)
        }
    }
    console.log(arr);
    //render函数自动找views目录中的demo.ejs文件 并且向这个页面注入arr这个数据
    res.render('demo',{
        list:arr
    })
})
app.use(express.static('static'))
app.listen(3000,()=>{
    console.log('3000端口成功运行');
})
```

## express的cookie和session

```javascript
/*
* cookie和session
*   都是把数据储存起来
*   储存时间的自已定义的
*   即使浏览器关闭了  数据依旧会储存
*
*
* cookie的原理
*   在用户第一次访问某个网站的时候 是不会携带cookie的
*   服务器会让浏览器再一次访问的时候必须携带指定的cookie
*   再第二次访问服务器的时候 如果浏览器携带的cookie和服务器要求的cookie是一样
*   则服务器就认识了指定的用户
*
* session的原理
* 在用户第一次访问某个网站的时候 是不会携带cookie的
* 服务器会让浏览器再一次访问的时候必须携带指定的cookie（A）
* 同时还会在服务端生成一个session  这个session是以指定的cookie(A)为属性名的
* 再第二次访问服务器的时候 如果浏览器携带的cookie和服务器session中cookie属性名是一样
* 则服务器就认识了指定的用户
*
*
*
*
*   cookie
*       存储在浏览器
*       不安全
*       存储数据的量比较小 4M
*   session  依赖与cookie实现的
*       存储在服务端
*       安全
*       存储数据的量比较大
*
*
*   localStorage
*       存储数据的时间：永久
*       如果浏览器卸载 数据才会消失
*   sessionStorage
*       存储数据的时间：一次浏览器周期  打开浏览器到关闭浏览器的时间差
* */

const express = require('express')
const cookieParser = require('cookie-parser')

const app = express()
app.use(cookieParser())

app.get('/',(req,res)=>{
    //获取所有去过的城市
    if(req.url==='/favicon.ico')return
    const cityArr =JSON.parse(req.cookies.express_cookie_city||'[]')
    res.send(`你的足迹是${cityArr}`)
})

app.get('/:city',(req,res)=>{
    if(req.url==='/favicon.ico')return
    //获取当前city
    const {city} = req.params
    //读取cookie
    const cityArr = JSON.parse(req.cookies.express_cookie_city||'[]')

    cityArr.push(city)

    //使用res响应体对象  来让前台存储cookie
    //res.cookie(属性名，属性值，cookie的存储时长)
    res.cookie('express_cookie_city',JSON.stringify(cityArr),{
        maxAge:1000*60*60*60,//存储时间为60个小时
    })
    res.send(`你今天去的城市是${city}`)
})

app.listen(3000,()=>{
    console.log('3000端口成功运行');
})
```



```javascript
const express = require('express')
const session = require('express-session')
const cookieParser = require('cookie-parser')

const app = express()
app.use(cookieParser())
app.use(session({
    secret: 'keyboard cat',
    resave: false,
    name:'heaven',  //设置的cookie的属性名
    saveUninitialized: true,
    cookie: {
        secure: false,//可以不写
        maxAge:80000,//必须
    }
}))

app.get('/',(req,res)=>{
    //获取所有去过的城市
    if(req.url==='/favicon.ico')return
    const cityArr = JSON.parse(req.session.express_session_city||'[]')

    res.send(`你的足迹是${cityArr}`)
})

app.get('/:city',(req,res)=>{
    if(req.url==='/favicon.ico')return
    //获取当前city
    const {city} = req.params

    //读取session
    const cityArr = JSON.parse(req.session.express_session_city||'[]')
    //把这次去的地方添加到上次去的数组中
    cityArr.push(city)
    //设置session
    req.session.express_session_city = JSON.stringify(cityArr)
    res.send(`你今天去的城市是${city}`)
})

app.listen(3004,()=>{
    console.log('3004端口成功运行');
})

```

## koa的cookie和session

```javascript
const Koa = require('koa')

const router = require('koa-router')()
const app = new Koa()

//配置路由
app.use(router.routes()).use(router.allowedMethods())


router.get('/',async cxt=>{

   //读取cookie                                                                  
   const cityArr = JSON.parse(cxt.cookies.get('koa2_cookie_city')||'[]')

    console.log(cityArr);
    cxt.body = `你今天去的城市是${cityArr}`
    
})

router.get('/:city',async  cxt=>{
    const {city} = cxt.params


    //读取cookie
    const cityArr = JSON.parse(cxt.cookies.get('koa2_cookie_city')||'[]')

     //把这次去的地方添加到上次去的数组中
    cityArr.push(city)

    //cxt.cookies.set(属性名,属性值,时长)
    cxt.cookies.set('koa2_cookie_city',JSON.stringify(cityArr),{
        maxAge:1000*60*60*60,   //存储60个小时
    })

    cxt.body = `你今天去的城市是${city}`
})


app.listen(3000,()=>{
    console.log('3000端口成功启动')
})
```



```javascript
const Koa = require('koa')

const router = require('koa-router')()
const app = new Koa()

//配置路由
app.use(router.routes()).use(router.allowedMethods())


router.get('/',async cxt=>{

   //读取cookie                                                                  
   const cityArr = JSON.parse(cxt.cookies.get('koa2_cookie_city')||'[]')

    console.log(cityArr);
    cxt.body = `你今天去的城市是${cityArr}`
    
})

router.get('/:city',async  cxt=>{
    const {city} = cxt.params


    //读取cookie
    const cityArr = JSON.parse(cxt.cookies.get('koa2_cookie_city')||'[]')

     //把这次去的地方添加到上次去的数组中
    cityArr.push(city)

    //cxt.cookies.set(属性名,属性值,时长)
    cxt.cookies.set('koa2_cookie_city',JSON.stringify(cityArr),{
        maxAge:1000*60*60*60,   //存储60个小时
    })

    cxt.body = `你今天去的城市是${city}`
})


app.listen(3000,()=>{
    console.log('3000端口成功启动')
})
```

## mongodb数据库

启动mongodb数据库   mongod --dbpath 文件的路径 --port 端口号 （默认是27017）

 新开一个cmd窗口  使用mongo --port 端口 连接数据库

show dbs 展示所有的数据库

use 数据库名字  连接到指定的数据库

db.集合名字.insert()  向数据库的集合中添加数据

db.集合名字.find().limit(数量)      查找集合中的数据（尽可能多）

如果需要限制查找数据的数量 可以使用limit函数

如果需要跳过指定的数据  可以使用skip函数

 如果需要获得查找的数据的长度  可以使用count函数

db.集合名字.findOne()   查找集合中的一个数据

db.集合名字.remove()     删除集合中的数据（尽可能多）

db.集合名字.remove({name:'heaven'},1)   删除集合中的一个数据

## mongDB操作数据库

```javascript
show tables/show collections//可以查看数据库的集合

db.集合.update({name:'heaven'},{$set:{name:'海文'}})  //只能更新一个数据

db.集合.update(  //更新多条数据
       {name:'heaven'},
       {$set:{name:'海文'}},
       {multi:true}
   )
```



## mongoose操作数据库

```javascript
const mongoose = require('mongoose')
// mongoose这个模块的作用是 在node环境中操作mongodb数据库  不需要在cmd窗口中操作mongodb数据库

//链接数据库  如果project这个数据库不存在  就会自动创建的 （是一个promis对象）
mongoose.connect('mongodb://localhost:27017/project',{
    useNewUrlParser: true,
    useUnifiedTopology: true
}).then(async ()=>{
    console.log('数据库链接成功')
    }).catch(()=>{
    console.log('数据库链接失败')
})
//先定义project这个数据库中集合的数据格式  banji121
    //规定数组的类型  可以是 String Number Boolean  Array ...
    let studentSchema = new mongoose.Schema({
        name:String,    //banji121这个集合中 的数据要有name  weight  height
        weight:{
            type:Number,
            required:true       //required 表示必须有这个属性
        },
        height:{
            type:Number,
            default:175         //  设置属性的默认值
        }
    })

    //让banji121这个集合应用studentSchema
    let Student = mongoose.model('banji121',studentSchema)
    
    
    //向集合中添加数据  create函数返回的是promsie
     const result1 = await Student.create({
         weight:67.5
         height:180
     })
     
     //传多个对象
      Student.create([
         {
         name:'海文',
         weight:67.5,
         height:180
     	},
    	 {
    	 name:'heaven',
    	 weight:67.5,
    	 height:180
    	}
   	])


//查看数据库的数据  find函数返回的是promsie
    const result2 = await Student.find()
    
    
//sort 可以对查到的数据 排序  默认是+1  -1是倒序排列
    const result7 = await Student.find({weight:80}).sort({_id:1})
    
//可以通过id查询
    const result8 = await Student.findById('5daf028bb66c2c220889fd57')
    
//删除数据库的数据  deleteOne/deleteMany函数返回的是promsie
     const result3 = await Student.deleteOne({name:"heaven"})
     const result4 = await Student.deleteMany({name:"海文"})
     
//修改数据库的数据  updateOne/updateMany函数返回的是promsie
	const result5 =  await Student.updateOne({weight:67.5},{$set:{weight:80}})
 	const result6 =  await Student.updateMany({weight:67.5},{$set:{weight:80}})
```

