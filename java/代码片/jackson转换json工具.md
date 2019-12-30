## 使用Jackson操作JSON数据

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import java.io.IOException;
import java.math.BigDecimal;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;
import java.util.List;

/**
 * json字符串和对象之间互转
 *
 * @author zhoutaotao
 * @date 2019/5/15
 */
public class JsonUtil {

  private static final ObjectMapper mapper;

  static {
    mapper = new ObjectMapper();
    initMapper(mapper);
  }

  /**
   * 获取 mapper
   */
  public static ObjectMapper getMapper() {
    return mapper;
  }

  /**
   * 初始化 mapper
   */
  public static void initMapper(ObjectMapper mapper) {
    // 注册不序列化 null 对象
    mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
    // 序列化配置
    mapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
    mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, true);
    // 反序列化配置
    mapper.configure(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT, true);
    mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    mapper.configure(DeserializationFeature.FAIL_ON_INVALID_SUBTYPE, false);
    // 注册自定义类型
    mapper.registerModule(numericalAccuracy());
    mapper.registerModule(dateModule());
    mapper.registerModule(localDateModule());
    mapper.registerModule(localDateModuleTime());
  }


  /**
   * 处理数值精度的模块
   */
  public static SimpleModule numericalAccuracy() {
    SimpleModule module = new SimpleModule();
    // 序列化
    module.addSerializer(new ToStringSerializer(Long.TYPE));
    module.addSerializer(new ToStringSerializer(Long.class));
    module.addSerializer(new ToStringSerializer(BigDecimal.class));

    // 反序列化
    module.addDeserializer(Long.TYPE, new JsonDeserializer<Long>() {
      @Override
      public Long deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null : Long.parseLong(value.trim());
      }
    });
    module.addDeserializer(Long.class, new JsonDeserializer<Long>() {
      @Override
      public Long deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null : Long.parseLong(value.trim());
      }
    });
    module.addDeserializer(BigDecimal.class, new JsonDeserializer<BigDecimal>() {
      @Override
      public BigDecimal deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null : BigDecimal.valueOf(Long.parseLong(value.trim()));
      }
    });
    return module;
  }

  /**
   * java.util.Date 数据处理模块
   */
  public static SimpleModule dateModule() {
    SimpleModule module = new SimpleModule();
    module.addSerializer(Date.class, new JsonSerializer<Date>() {
      @Override
      public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers)
          throws IOException {
        gen.writeObject(ObjectUtil.isNull(value) ? value : DateUtil.getTimestamp(value));
      }
    });
    module.addDeserializer(Date.class, new JsonDeserializer<Date>() {
      @Override
      public Date deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null : DateUtil.parseDate(Long.parseLong(value.trim()));
      }
    });
    return module;
  }

  /**
   * java.time.LocalDate 处理模块
   */
  public static SimpleModule localDateModule() {
    SimpleModule module = new SimpleModule();
    module.addSerializer(LocalDate.class, new JsonSerializer<LocalDate>() {
      @Override
      public void serialize(LocalDate value, JsonGenerator gen, SerializerProvider serializers)
          throws IOException {
        gen.writeObject(ObjectUtil.isNull(value) ? value : DateUtil.getTimestamp(value));
      }
    });
    module.addDeserializer(LocalDate.class, new JsonDeserializer<LocalDate>() {
      @Override
      public LocalDate deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null
            : DateUtil.parseLocalDate(Long.parseLong(value.trim()));
      }
    });
    return module;
  }

  /**
   * java.time.LocalDateTime 处理模块
   */
  public static SimpleModule localDateModuleTime() {
    SimpleModule module = new SimpleModule();
    module.addSerializer(LocalDateTime.class, new JsonSerializer<LocalDateTime>() {
      @Override
      public void serialize(LocalDateTime value, JsonGenerator gen, SerializerProvider serializers)
          throws IOException {
        gen.writeObject(ObjectUtil.isNull(value) ? value : DateUtil.getTimestamp(value));
      }
    });
    module.addDeserializer(LocalDateTime.class, new JsonDeserializer<LocalDateTime>() {
      @Override
      public LocalDateTime deserialize(JsonParser p, DeserializationContext ctx)
          throws IOException, JsonProcessingException {
        String value = p.getValueAsString();
        return StringUtil.isBlank(value) ? null
            : DateUtil.parseLocalDateTime(Long.parseLong(value.trim()));
      }
    });
    return module;
  }

  /**
   * 把 java 对象转换为 json 字符串
   */
  public static <T> String toJson(T data) {
    try {
      return mapper.writeValueAsString(data);
    } catch (JsonProcessingException e) {
      throw new RuntimeException(e);
    }
  }

  /**
   * 将 json 字符串转换为 java 对象
   */
  public static <T> T fromJson(String json, Class<T> clazz) {
    try {
      return mapper.readValue(json, clazz);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }

  /**
   * 将 json 字符串数组转换为 java 数组
   */
  public static <T> List<T> fromJsonArray(String json, TypeReference<List<T>> typeReference) {
    try {
      return mapper.readValue(json, typeReference);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
}
```

