---
description: >-
  MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。MongoDB
  是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的
---

# mongoDB

MongoDB增删改查操作实战案例（非模块化） 案例：用户信息增删改查 搭建网站服务器，实现客户端与服务器端的通信 连接数据库，创建用户集合，向集合中插入文档 当用户访问/list时，将所有用户信息查询出来 将用户信息和表格HTML进行拼接并将拼接结果响应回客户端 当用户访问/add时，呈现表单页面，并实现添加用户信息功能 当用户访问/modify时，呈现修改页面，并实现修改用户信息功能 当用户访问/delete时，实现用户删除功能&#x20;

```javascript
// 1. 搭建网站服务器，实现客户端与服务器端的通信
// 2. 连接数据库，创建用户集合，向集合中插入文档
// 3. 当用户访问`/list`时，将所有用户信息查询出来
    //  实现路由功能
    // 呈现用户信息页面
    // 从数据库中查询用户信息，将用户信息展示在列表中
// 4. 将用户信息和表格HTML进行拼接并将拼接结果响应回客户端
// 5. 当用户访问`/add`时，呈现表单页面，并实现添加用户信息功能
// 6. 当用户访问`/modify`时，呈现修改页面，并实现修改用户信息功能
    // 1. 增加页面路由 呈现页面
        // (1) 在点击修改按钮的时候，将用户 id  传递到当前页面
        // (2) 在数据库中查询当前用户信息，将用户信息展示到页面中
    // 2. 实现用户修改功能
        // （1）指定表单的提交地址以及请求方式
        //  (2) 接受客户端传递过来的修改信息 找到用户 将用户信息更改为最新的
// 7. 当用户访问`/delete`时，实现用户删除功能
// const { spawn } = require('child_process')
// const { readdirSync } = require('fs')
const http = require('http')
const mongoose = require('mongoose')
const url = require('url')
const querystring = require('querystring')

// 数据库连接 27017是mongodb数据库的默认端口
mongoose.connect('mongodb://localhost/zhangbing',{ useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('数据库连接成功'))
    .catch(() => console.log('数据库连接失败'))

// 创建用户集合规则
const userSchema = new mongoose.Schema({
    name:{
        type:String,
        required:true,
        minlength:2,
        maxlength:20
    },
    age:{
        type:Number,
        min:18,
        max:80
    },
    password:String,
    email:String,
    hobbies:[String] 
})

const User = mongoose.model('User',userSchema)


// 创建服务器
const app = http.createServer()

// 为服务器对象添加请求事件
app.on('request', async (req,res) => {
    // 请求方式
    const method = req.method;
    // 请求地址(可以通过对象解构出来)
    const { pathname, query } = url.parse(req.url, true)
    if (method == 'GET') {
        // 呈递用户列表页面
        if (pathname == '/list') {
            // 查询用户信息
            let users = await User.find() // 数组
            // html 字符串
            let list = `<!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <title>用户列表</title>
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
            </head>
            <body>
                <div class="container">
                    <h6>
                        <a href="/add" class="btn btn-primary">添加用户</a>
                    </h6>
                    <table class="table table-striped table-bordered">
                        <tr>
                            <td>用户名</td>
                            <td>年龄</td>
                            <td>爱好</td>
                            <td>邮箱</td>
                            <td>操作</td>
                        </tr>
            `;
            // 对数据进行循环操作 item 是每一个对象
            users.forEach((item) => {
                list += `
                    <tr>
                    <td>${item.name}</td>
                    <td>${item.age}</td>
                    <td>
                `;
                // hobbies 是一个数组 可以遍历
					item.hobbies.forEach((item) => {
                        list += `<span>${item}</span>`;
                    })

				list += `</td>
				<td>${item.email}</td>
				<td>
					<a href="/remove?id=${item.id}" class="btn btn-danger btn-xs">删除</a>
					<a href="/modify?id=${item.id}" class="btn btn-success btn-xs">修改</a>
				</td>
			</tr>
                `;
            })
            
            list += `</table>
            </div>
        </body>
        </html>
            `;
            res.end(list)
        } else if (pathname == '/add') {
            let add = `
            <!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>用户列表</title>
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
</head>
<body>
	<div class="container">
		<h3>添加用户</h3>
		<form method='post' action="/add">
		  <div class="form-group">
		    <label>用户名</label>
		    <input name="name" type="text" class="form-control" placeholder="请填写用户名">
		  </div>
		  <div class="form-group">
		    <label>密码</label>
		    <input name="password" type="password" class="form-control" placeholder="请输入密码">
		  </div>
		  <div class="form-group">
		    <label>年龄</label>
		    <input name="age" type="text" class="form-control" placeholder="请填写年龄">
		  </div>
		  <div class="form-group">
		    <label>邮箱</label>
		    <input name="email"type="email" class="form-control" placeholder="请填写邮箱">
		  </div>
		  <div class="form-group">
		    <label>请选择爱好</label>
		    <div>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="足球" name="hobbies"> 足球
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="篮球" name="hobbies"> 篮球
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="橄榄球" name="hobbies"> 橄榄球
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="敲代码" name="hobbies"> 敲代码
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="抽烟" name="hobbies"> 抽烟
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="喝酒" name="hobbies"> 喝酒
		    	</label>
		    	<label class="checkbox-inline">
		    	  <input type="checkbox" value="烫头" name="hobbies"> 烫头
		    	</label>
		    </div>
		  </div>
		  <button type="submit" class="btn btn-primary">添加用户</button>
		</form>
	</div>
</body>
</html>`;

            res.end(add)
        } else if (pathname == '/modify') {
            // 根据待修改的用户的 id 去数据库中查询用户信息
            let user = await User.findOne({_id:query.id})
            let hobbies = ['足球','篮球','橄榄球','敲代码','抽烟','喝酒','烫头', '吃饭', '睡觉', '打豆豆']
            // console.log(user)
            // 呈现添加用户表单页面
            let modify = `
            <!DOCTYPE html>
            <html lang="en">
            <head>
              <meta charset="UTF-8">
              <title>用户列表</title>
              <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
            </head>
            <body>
              <div class="container">
                <h3>修改用户</h3>
                <form method='post' action="/modify?id=${user._id}">
                  <div class="form-group">
                    <label>用户名</label>
                    <input value="${user.name}" name="name" type="text" class="form-control" placeholder="请填写用户名">
                  </div>
                  <div class="form-group">
                    <label>密码</label>
                    <input value="${user.password}" name="password" type="password" class="form-control" placeholder="请输入密码">
                  </div>
                  <div class="form-group">
                    <label>年龄</label>
                    <input value="${user.age}" name="age" type="text" class="form-control" placeholder="请填写年龄">
                  </div>
                  <div class="form-group">
                    <label>邮箱</label>
                    <input value="${user.email}" name="email"type="email" class="form-control" placeholder="请填写邮箱">
                  </div>
                  <div class="form-group">
                    <label>请选择爱好</label>
                        <div>		    	
              `;     
       
        hobbies.forEach (item => {
            // 判断当前循环项在不在用户的爱好数据组
            let isHobby = user.hobbies.includes(item)
            if (isHobby) {
                modify += `
                <label class="checkbox-inline">
                  <input type="checkbox" value="${item}" name="hobbies" checked> ${item}
              </label>`          
            } else {
                modify += `
                <label class="checkbox-inline">
                  <input type="checkbox" value="${item}" name="hobbies"> ${item}
              </label>` 
            }
      })

    modify += `</div>
        </div>
        <button type="submit" class="btn btn-primary">修改用户</button>
        </form>
        </div>
        </body>
        </html>
    `;
    res.end(modify)

        } else if (pathname == '/remove') {
            // res.end(query.id) 
            await User.findOneAndDelete({_id:query.id})
            res.writeHead(301, {
              Location:'/list'
          })
          res.end();
        }

    } else if (method == 'POST') {
        // 用户添加功能
        if (pathname == '/add') {
            // 接收用户提交的信息
            let formData = '';
            // 接受 post 参数
            req.on('data', param => {
                formData += param;
            })
            // post 参数接受完毕
            req.on('end', async () => {
                // console.log(formData) // name=123&password=123456&age=20&email=123%40123.com&hobbies=%E8%B6%B3%E7%90%83&hobbies=%E7%AF%AE%E7%90%83
                // console.log(querystring.parse(formData)) // [Object: null prototype] {
                                                                //   name: '123',
                                                                //   password: '123456',
                                                                //   age: '20',
                                                                //   email: '123@123.com',
                                                                //   hobbies: [ '足球', '篮球' ]
                                                                // }
                let user = querystring.parse(formData)
                // 将用户提交的信息提交到数据库中
                await User.create(user)
                // 301 代表重定向
                // location 跳转地址
                res.writeHead(301, {
                    Location:'/list'
                })
                res.end();
            }) 
        } else if (pathname == '/modify') {
            // 接受用户提交的信息
			let formData = '';
			// 接受post参数
			req.on('data', param => {
				formData += param;
			})
			// post参数接受完毕
			req.on('end', async () => {
				let user = querystring.parse(formData)
				// 将用户提交的信息添加到数据库中
				await User.updateOne({_id: query.id}, user);
				// 301代表重定向
				// location 跳转地址
				res.writeHead(301, {
					Location: '/list'
				});
				res.end();
        })
    }
    }
})

// 监听端口
app.listen(3000)
————————————————
版权声明：本文为CSDN博主「1oneLee」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_27575925/article/details/109232759
```
