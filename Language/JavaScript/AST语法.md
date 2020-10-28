抽象语法树 (Abstract Syntax Tree)，是将代码逐字母解析成 树状对象 的形式。这是语言之间的转换、代码语法检查，代码风格检查，代码格式化，代码高亮，代码错误提示，代码自动补全等等的基础。例如:

```
function square(n){
    return n * n
}
```

通过解析转化成的AST如下图:
![AST](images/021.png)
