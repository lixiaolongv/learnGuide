# Cypress最佳实现
## 1.设置全局URL
```buildoutcfg
{
  "baseUrl": "http://localhost:8080",
}
```
解析：   
设置好baseurl之后，不仅可以在运行时节省Cypress匹配被侧应用程序URL的时间，还可以在编写
待访问的url时，忽略baseUrl，直接写后面的路径。

## 2. 避免访问多站点
解析：   
为了绕开同源策略的限制而实现的方案，使得Cypress不能支持在一个测试用例里访问多个不同域名的URL。

## 3，删除等待代码
解析：   
你无需使用显示等待，Cypress的许多命令都自带Retry属性。如需等待一个事件的返回，可以使用别名
```buildoutcfg
cy.intercept('POST', '/Account/Login', {fixture: 'user_login.json'}).as('getUser')
    // 等待拦截接口状态码返回200，确认接口返回正常
    cy.wait('@getUser').its('response.statusCode')
      .should('equal', 200)
```
## 4，停用条件测试
考虑如下的场景：判断一个元素是否存在，当它存在时，执行A操作，当它不存在时，执行B操作，
条件测试是导致测试不稳定的根本原因，因此Cypress建议通过指定前置测试条件来避免操作引发的不确定行为

##