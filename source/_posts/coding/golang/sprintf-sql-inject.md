---
title: 不要再用Spintf拼Sql了
date: 2022-06-28 11:10:00
tags: 
- Golang
- Mysql
category: Golang
---
说到代码安全，虽然很多时候大家都不当回事，只要实现业务功能就行了，但是一旦出现安全问题都不是小事，突然想起来之前项目有很多类似的写法：
```go
sql := "select * from xxx where 1=1"
if name != "" {
    sql += fmt.Sprintf(" and name = '%s'", name)
}
if address != "" {
    sql += fmt.Sprintf(" and address = '%s'", name)
}
err := db.Raw(sql).Find(&results).Error
if err != nil {
    return nil, err
}
```
为什么要这么写？主要是为了实现一些复杂的列表查询，默认情况下查询所有，但是有很多可选的参数，所以图省事选择了拼Sql，虽然功能上没有什么问题。

但是经过我的测试，上面这种写法存在SQL注入风险，非常危险！！！

<!--more-->

Sprintf这个函数只是帮你格式化，实际上并不会像MySql的预处理一样对特殊字符加以转义等处理，所以当有心之人构造特殊参数输入的话就会产生意外的结果。

比如，当name内容是```' union select 1,2,3,4,5,version(),user(),database()'```的时候，就会执行一条注入的语句，获取数据库的一些信息为进一步的渗透做准备。

## 解决方法
最正确的做法是Prepare预处理，如果你没有使用ORM的话，基本上都需要对所有的SQL做预处理操作防止注入，比较麻烦！

但是现在一般都会使用ORM来操作数据库，ORM本质上也是拼SQL，只不过底层都会统一使用SQL预处理，对注入这种攻击都有防备，所以尽可能的使用ORM封装好的方法，比如上面这种需求在Gorm里面其实可以使用结构体参数或者map参数的方式查询，Gorm会自动忽略零值的参数。
```go
// NOTE When querying with struct, GORM will only query with non-zero fields, 
// that means if your field’s value is 0, '', false or other zero values, 
// it won’t be used to build query conditions, for example:
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
```

千万不要再偷懒手动拼SQL，安全问题无小事，有人可能说我对字符参数校验了，不允许出现那些特殊字符，对于整形参数都会先转int，当然这也是一些参数层面预防方法，但是都不彻底。
