# Cypress好用的测试插件及其文档整理
## 学习资料的整理
* 1 Cypress系列的资料   
 https://www.yuque.com/gstorms/fo7n4g/ugztq4
* 2 Cypress命令大全   
https://www.yuque.com/gstorms/fo7n4g/altgg8
* 3 Selenium系列   
https://www.yuque.com/gstorms/fo7n4g/ez99qe
* 4 Pytest系列   
https://www.yuque.com/gstorms/fo7n4g/xo86te

## 测试运行失败自动重试
* 1 通过插件来完成自动重试功能【cypress5.0以前】  
npm install -D cypress-plugin-retries

* 2 Cypress 自带的重试功能【cypress5.0以后】   
```buildoutcfg
全局配置：
通常需要为 cypress run 和 cypress open 分开定义不同的重试次数
默认在 cypress.json 中进行配置
runMode：定义运行 cypress run 时的重试次数
openMode：定义运行 cypress open 时的重试次数
```
  
## 如何动态挑选待运行测试用例
使用 cypress-select-tests 插件
官方：https://github.com/bahmutov/cypress-select-tests

## 动态生成测试用例
直接看这篇文章：https://www.cnblogs.com/poloyy/p/13042466.html

## Cypress典型的坑
* 1 Cypress命令是异步的   
解析：Cypress的命令在被调用时并不会被马上执行，Cypress会先把所有命令排队-enquene，
  然后再执行。
* 2 慎用箭头函数
* 3 Cypress不支持async和await代码
* 4 赋值永远失败【获取元素返回值并运用到下一个请求，待解析】
* 5 躲不过的同源策略   
解析：同源策略是浏览器的安全基石，意味着当两个iframe直接有访问时，必须满足协议相同，域名相同，端口相同三个条件。
  那么Cypress的原理是什么呢？它为什么可以和应用程序直接通信，浏览器的同源策略是不允许的？   
Cypress通过以下策略绕过了浏览器的限制：   
  * 1 将document.domain注入text/html页面
  * 2 代理所有的HTTP/HTTPS通信
  * 3 更改托管的URL以匹配被测应用程序的URL
  * 4 使用浏览器内部的API进行网络间通信    
 首次加载Cypress时，内部Cypress Web应用程序托管在一个随机端口上，类似于http://localhost: 65874/_/
 在测试中，当第一个cy.visit()命令被发出后，Cypress将更改其URL以匹配远程应用程序的来源，从而解决了同源策略的主要障碍。
 坏处是：一次测试中，访问的域名必须处于同一个超域之下（super domain），否则Cypress测试将会报错
    
## Cypress的独特之处（掌握）
* 1 闭包（closure）
保存一个值或者引用的最好方式是使用闭包。   
.then()是Cypress对闭包的典型应用    
.then()返回上一个命令的结果，并将其注入到下一个命令中   
例子：待补充    

* 2 变量和别名   
作用：实现元素赋值操作或实现变量共享   
场景：当你的测试需要一个前置条件才能执行，比如得到数据库中的某个表的值。
  可以使用.wrap()和.as()命令
  
* 3 fixture()
作用：实现上下文共享   
场景：测试数据需要的变量直接来自外部文件，可以通过fixture()来实现上下文共享
.fixture()用来加载位于文件中的一组固定数据
  
* 4 自定义方法   
作用：通过自定义方法，实现上下文共享
1）在cypress.json文件中，env选项下添加如下配置
```buildoutcfg
  "env": {
    "testVariables": {},
  }
```
2）在测试代码中使用Cypress.env()来设置和获取变量
```buildoutcfg
Cypress.env('testVariables', $el.text())
```


  



 
