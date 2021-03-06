### 项目说明文件
基于express 搭建
后台ui使用layui
使用的技术 layui+jquery+aysnc/awit
### 运行项目
npm i 安装项目所有的依赖项
打开data导入sql文件，在修改config/db.json数据库配置信息。
执行npm run serve 启动项目
这是在github上面手动修改的，需要执行git pull 进行拉取
again
加入进度条 NProgress
在网页加载完毕之前加载进度条，加载完毕取消进度条即可
```js
    function startNProgress(){
        NProgress.set(0.4);// 默认设置40% NProgress.set(0) 等价与 NProgress.start()
        let interval = setInterval(function(){
            NProgress.inc(); // 以小量递增
        },200)

        $(window).on('load',()=>{
            clearInterval(interval)
            NProgress.done()
        })
    }

    startNProgress();
```
高版本的jquery不支持 $(window).on('load')的写法，改为on绑定即可. 参考

### 文章编辑
-1.先实现数据在表单中的回显，要获取到文章的id去发起请求获取数据
-2.实现update入库
-集成富文本编辑器wangEditor
-官网

### 文档

初始化
```js
<body>
    <div id="div1"></div>
</body>
<script src="https://cdn.jsdelivr.net/npm/wangeditor@latest/dist/wangEditor.min.js"></script>
<script type="text/javascript">
    const E = window.wangEditor
    const editor = new E('#div1')
    editor.config.uploadImgShowBase64 = true; // 可以实现手动上传图片(转换成base64格式)
    // 或者 const editor = new E( document.getElementById('div1') )
    editor.create()
</script>
```
获取和设置内容api
### 防止用户翻墙访问（需要做权限验证）
- 说明： 有些路由需要登录后台有权限之后才可以进行操作，有session就说明有权限。

- 思考问题：哪些路由需要判断权限防翻墙，哪些不需要验证权限（放行）？

- 基本只要进入到后台执行的路由都需要权限验证
在系统外面的路由就不需要验证即放行。如： /login , /singin , /logout 出来这三个之外其他路由都需要权限验证所以我可以定义一个中间件，在路由请求之前获取当前路由判断是否有权限
```js
// 定义中间件，托管静态资源
app.use('/public',express.static(path.join(__dirname,'public')));
app.use('/uploads',express.static(path.join(__dirname,'uploads')));


// 在进入到路由匹配函数之前，要进行验证权限
app.use(function(req,res,next){
    // 1.获取当前访问路由
    let path = req.path.toLowerCase();
    // 2. 定义放行的路由，即不需要权限验证
    let noCheckAuth = ['/login','/signin','/logout']
    if(noCheckAuth.includes(path)){
        // 需要放行,不做任何限制
        next();
    }else{
        // 不在放行之外，需要验证权限（session）
        // addArt
        if(req.session.userInfo){
            // 有权限，可以继续操作
            next()
        }else{
            res.redirect('/login')
        }
    }
});

// 使用路由中间件 req.body
app.use(router)
```
## 时间
```js
# 获取今年（当前年）一月份发布的文章
select title,publish_date from article where month(publish_date) = 1 and year(publish_date) = year(now())

#  获取今年（当前年）一月份发布的文章总数
select month(publish_date) month,count(*) as total from article where month(publish_date) = 1 and year(publish_date) = year(now())

# 获取今年（当前年）每月份发布的文章总数
SELECT
	MONTH (publish_date) MONTH,
	count(*) AS total
FROM
	article
WHERE  
	YEAR (publish_date) = YEAR (now())
GROUP BY
	MONTH (publish_date)


# 字符串拼接
select CONCAT(month(publish_date),'月') month,count(*) as total from article  where year(publish_date) = year(now()) group by month(publish_date)
```
## mysql模糊查询
# 模糊查询:   字段名  like   '%a%'
# %fs%    查询标题含有 fs字符 
# fs%    查询标题以fs字符 开头
# %fs    查询标题以fs字符 结尾
```js
select title,status from article where 1 and  title like '%b%' and status = 0;
```
- where 1 仅仅是为了拼接多个查询条件，从而避免没有查询条件时的出错。