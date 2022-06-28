---
title: 好激动！这是我的第一篇文章
date: 2022-5-26
---

# 路由守卫（前端权限控制）

作用：对路由进行权限控制

需要在router的路由配置里面添加mate属性：

全局前置守卫：每次路由切换之前执行

```js
// 全局前置守卫：初始化时、每次路由切换前执行
//函数有三个参数to（里面包含目标的路由信息）,from（当前的路由从哪里来）,next
//mate：路由元信息，即程序员可以自定义的信息
router.beforeEach((to,from,next) => {
	console.log('beforeEach',to,from)
	if(to.meta.isAuth){ // 判断当前路由是否需要进行权限控制
		if(localStorage.getItem('school') === 'atguigu'){ // 权限控制的具体规则
			next()	// 放行
		}else{
			alert('暂无权限查看')
		}
	}else{
		next()	// 放行
	}
})

// 全局后置守卫：初始化时、每次路由切换后执行
router.afterEach((to,from) => {
	console.log('afterEach',to,from)
	if(to.meta.title){ 
		document.title = to.meta.title //修改网页的title
	}else{
		document.title = 'vue_test'
	}
})
```

## 全局后置：每次切换路由之后被调用（用的不多）

routes里面配置title，每次页面跳转，浏览器页签的名字会变为相应的title值

```js
//该文件专门用于创建整个应用的路由器
import VueRouter from "vue-router";
//引入组件
import Home from '../pages/Home'
import About from '../pages/About'
import News from '../pages/News'
import Message from '../pages/Message'
import Detail from '../pages/Detail'


//创建一个路由器
const router = new VueRouter({
    routes:[
        {
            name:'guanyv',
            path:'/about',
            component:About,
            meta:{title:'关于'}
        },
        {
            name:'zhuye',
            path:'/home',
            component:Home,
            meta:{title:'主页'},
            children:[
                {
                    name:'xinwen',
                    path:'news',
                    component:News,
                    meta:{isAuth:true,title:'新闻'}
                },
                {
                    name:'xiaoxi',
                    path:'message',
                    component:Message,
                    meta:{isAuth:true,title:'消息'},
                    children:[
                        {
                            name:'xiangqing',
                            path:'detail',
                            component:Detail,
                            meta:{isAuth:true,title:'详情'},
                            props($route){
                                return {
                                    id:$route.query.id,
                                    title:$route.query.title,
                                }
														}
                        }
                    ]
                }
            ]
        }
    ]
})

// 🔴全局前置路由守卫————初始化的时候、每次路由切换之前被调用
router.beforeEach((to,from,next) => {
    console.log('前置路由守卫',to,from)
    document.title = 'to.mate.title'//获取页签的值
    if(to.meta.isAuth){
        if(localStorage.getItem('school')==='atguigu'){
            next()
        }else{
            alert('学校名不对，无权限查看！')
        }
    }else{
        next()
    }
})

// 🔴全局后置路由守卫————初始化的时候被调用、每次路由切换之后被调用
router.afterEach((to,from)=>{
	console.log('后置路由守卫',to,from)
	document.title = to.meta.title || '硅谷系统'
})

// 导出路由器
export default router
```

## 独享路由守卫：（只有前置，没有后置）

```js
//该文件专门用于创建整个应用的路由器
import VueRouter from "vue-router";
//引入组件
import Home from '../pages/Home'
import About from '../pages/About'
import News from '../pages/News'
import Message from '../pages/Message'
import Detail from '../pages/Detail'


//创建一个路由器
const router = new VueRouter({
    routes:[
        {
            name:'guanyv',
            path:'/about',
            component:About,
            meta:{title:'关于'}
        },
        {
            name:'zhuye',
            path:'/home',
            component:Home,
            meta:{title:'主页'},
            children:[
                {
                    name:'xinwen',
                    path:'news',
                    component:News,
                    meta:{title:'新闻'},
                    // 🔴独享守卫，特定路由切换之后被调用
                    beforeEnter(to,from,next){
                        console.log('独享路由守卫',to,from)
                        if(localStorage.getItem('school') === 'atguigu'){
                            next()
                        }else{
                            alert('暂无权限查看')
                        }
                    }
                },
                {
                    name:'xiaoxi',
                    path:'message',
                    component:Message,
                    meta:{title:'消息'},
                    children:[
                        {
                            name:'xiangqing',
                            path:'detail',
                            component:Detail,
                            meta:{title:'详情'},
                            props($route){
                                return {
                                    id:$route.query.id,
                                    title:$route.query.title,
                                }
                            }
                        }
                    ]
                }
            ]
        }
    ]
})

//全局后置路由守卫————初始化的时候被调用、每次路由切换之后被调用
router.afterEach((to,from)=>{
	console.log('后置路由守卫',to,from)
	document.title = to.meta.title || '硅谷系统'
})

//导出路由器
export default router
```

组件内路由守卫：

```
<template>
    <h2>我是About组件的内容</h2>
</template>

<script>
    export default {
        name:'About',
        // 通过路由规则，进入该组件时被调用
        beforeRouteEnter (to, from, next) {
            console.log('About--beforeRouteEnter',to,from)
            if(localStorage.getItem('school')==='atguigu'){
                next()
            }else{
                alert('学校名不对，无权限查看！')
            }
        },
        // 通过路由规则，离开该组件时被调用
        beforeRouteLeave (to, from, next) {
            console.log('About--beforeRouteLeave',to,from)
            next()
        }
    }
</script>
```

路由工作的两种模式：

hash（路由有#号，兼容性更好）

和history（路由没有#，兼容性略差）

对于一个url来说，什么是hash值？---#及其之后的内容就是hash值（不会作为路由的一部分发给服务器。）

用nodejs和express搭建一个简单的服务器：

npm init--->取名字--->npm i express-->新建server.js-->启动命令：node server.js

```js
const express = require('express')
const  history = require('connect-history-api-fallback')

const app = express()
app.use(history())
app.use(express.static(__dirname+'/static'))
app.get('/p', (request, response) => {
    response.send({
        name:'xiaochen',
        age:20
    })
})

app.listen(5000, () => {
        console.log('服务器启动5000端口!');
})
//要是退出刚才的监听，需要ctrl+C进行退出，不然占着端口5001
```

```js
const router =  new VueRouter({
	mode:'history',
	routes:[...]
})

export default router
```

npm run build--打包部署命令

用history模式部署上线的时候，刷新会因为路径的问题404错误，解决：找后端工程师解决或者node的一个服务器的中间件里面的npm网站里面的connect-history-api-fallback专门用来解决这个问题

npm i connect-history-api-fallback-->引入const  history = require('connect-history-api-fallback')

就可以了。