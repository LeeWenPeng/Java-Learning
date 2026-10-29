除了二进制序列化方式，还可以选择 JSON 序列化。它的性能比二进制序列化方式差，但是可读性非常好，且广泛应用于 Web 领域。

## 1  JSON 标准

- JSON 只有两种结构：对象和数组
- 对象用 `{}` 包裹，对象内由键值对 (`"key" : "value"`) 组成，键值对之间通过 `,` 分隔
- 数组用 `[]` 包裹，数组内值通过 `,` 分隔
- 基本数据类型有七种
	- `false null true object array number string`

实例

```json
{
	"Image" : {
		"Width" : 800,
		"Height" : 600,
		"Title" : "View from 15th Floor",
		"Thumbnail": {
		      "Url": "http://www.example.com/image/481989943",
		      "Height": 125,
		      "Width": "100"
		},
		"IDs": [116, 943, 234, 38793]
	}
}
```

## 2  JSON 库

Java 中比较流行的 JSON 库有 (性能自上而下)：

- [Fastjson](https://github.com/alibaba/fastjson) - 阿里巴巴开发的 JSON 库，性能十分优秀。
- [Jackson](http://wiki.fasterxml.com/JacksonHome) - 社区十分活跃且更新速度很快。Spring 框架默认 JSON 库。
- [Gson](https://github.com/google/gson) - 谷歌开发的 JSON 库，目前功能最全的 JSON 库 。

## 3  JSON 编码指南

> 遵循好的设计与编码风格，能提前解决 80% 的问题，个人推荐 Google JSON 风格指南。
>
> - 英文版 [Google JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml)
> - 中文版 [Google JSON 风格指南](https://github.com/darcyliu/google-styleguide/blob/master/JSONStyleGuide.md)

## 4  序列化技术的选型

- JDK 序列化：性能较差，且有很多使用限制，不建议使用。
- [Thrift](https://github.com/apache/thrift)、[Protobuf](https://github.com/protocolbuffers/protobuf)：适用于**对性能敏感，对开发体验要求不高**。
- [Hessian](http://hessian.caucho.com/doc/hessian-overview.xtp)：适用于**对开发体验敏感，性能有要求**。
- [Jackson](https://github.com/FasterXML/jackson)、[Gson](https://github.com/google/gson)、[Fastjson](https://github.com/alibaba/fastjson)：适用于对序列化后的数据要求有**良好的可读性**（转为 json 、xml 形式）。
