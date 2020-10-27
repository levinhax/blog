## 使用Json的好处（什么是Schema）

- 描述现有的数据格式
- 提供清晰的人工和机器可读文档
- 完整的数据结构，有利于自动化测试
- 完整的数据结构，有利于验证客户端提交数据的质量

## 什么是JSON Schema

JSON Schema本身就是一种数据结构，可以清晰的描述JSON数据的结构。是一种描述JSON数据的JSON数据。

## 使用JSON Schema的好处

- JSON Schema 非常适用于基于JSON的HTTP的API。
- JSON Schema从Java的基本数据类型中对JSON结构进行校验，所以对JSON结构的校验可以理解为对每种不同数据类型的相应校验。
- 接口测试中可以快速的定位到自己数据格式的正确性。

```
{
    "$schema":"http://json-schema.org/draft-04/schema#",
    "title":"book info",
    "description":"some information about book",
    "type":"object",
    "properties":{
        "id":{
            "description":"The unique identifier for a book",
            "type":"integer",
            "minimum":1
        },
        "name":{
            "type":"string",
            "pattern":"^#([0-9a-fA-F]{6}$",
            "maxLength":6,
            "minLength":6
        },
        "price":{
            "type":"number",
            "multipleOf":0.5,
            "maximum":12.5,
            "exclusiveMaximum":true,
            "minimum":2.5,
            "exclusiveMinimum":true
        },
        "tags":{
            "type":"array",
            "items":[
                {
                    "type":"string",
                    "minLength":5
                },
                {
                    "type":"number",
                    "minimum":10
                }
            ],
            "additionalItems":{
                "type":"string",
                "minLength":2
            },
            "minItems":1,
            "maxItems":5,
            "uniqueItems":true
        }
    },
    "minProperties":1,
    "maxProperties":5,
    "required":[
        "id",
        "name",
        "price"
    ]
}
```

[JsonSchema官方文档](http://json-schema.org/)
[入门文档](http://json-schema.org/learn/getting-started-step-by-step.html)
