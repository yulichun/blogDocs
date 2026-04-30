## 模板文件语法

go的“text/template”包是模板引擎，负责加载解析执行模板文件。
可以参考 [go官方api文档](https://studygolang.com/pkgdoc) 的“text/template”包的描述，其详细描述了模板文件的编写语法，并且介绍了自带的函数

## 扩展函数

“text/template”模板引擎可以加载第三方函数实现包，“github.com/Masterminds/sprig/v3”包是常被使用的包。
[github.com/Masterminds/sprig/v3扩展函数](https://github.com/Masterminds/sprig/blob/master/docs) 包括了常用的数学计算、时间格式等函数

