# Custom Deserializer error: not close json text, token : }

Fastjson 自定义反序列化报错，“not close json text, token : }”，异常类似 [1.2.62 自定义反序列化报错 not close json text, token : }](https://github.com/alibaba/fastjson/issues/2848)

PS: 由于仓库已归档，无法评论，故在此记录。

```java
public class RuleDeserializable implements ObjectDeserializer {
    @Override
    public List<Stage> deserialze(DefaultJSONParser defaultJSONParser, Type type, Object fieldName) {
        String input = defaultJSONParser.getInput();
        List<Stage> stages = new ArrayList<>();
        JSONObject jsonObject = JSONObject.parseObject(input);
        // ...
        return stages;
    }
}
```

以上代码报错的原因是 `deserialze(...)` 方法虽然返回了期望的 `List<Stage>` 结果，但是没有正确更新 json 解析偏移量。

应该调用 `DefaultJSONParser#parseXXX` 方法更新 json 解析偏移量。

最小改动（性能差）：

```java
public class RuleDeserializable implements ObjectDeserializer {
    @Override
    public List<Stage> deserialze(DefaultJSONParser defaultJSONParser, Type type, Object fieldName) {
        // 字段解析，忽略返回值，二次解析性能差。
        defaultJSONParser.parse();

        String input = defaultJSONParser.getInput();
        List<Stage> stages = new ArrayList<>();
        JSONObject jsonObject = JSONObject.parseObject(input);
        // ...
        return stages;
    }
}
```

使用 `DefaultJSONParser#parseXXX` 方法返回值，避免二次解析：

```java
public class RuleDeserializable implements ObjectDeserializer {
    @Override
    public List<Stage> deserialze(DefaultJSONParser defaultJSONParser, Type type, Object fieldName) {
        List<Stage> stages = defaultJSONParser.parseArray(Stage.class);
        // ...
        return stages;
    }
}
```
