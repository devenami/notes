## 使用JAVA8在Date、LocalDateTime等等之间的转换

```java

import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.format.DateTimeFormatter;
import java.util.Date;

/**
 * 日期工具类
 *
 * @author feb13th
 * @since 2019/5/16 0:27
 */
public class DateUtil {

  public static final String FORMAT_DATA_SIMPLE = "yyyymmdd";
  public static final String FORMAT_DATE = "yyyy-mm-dd";
  public static final String FORMAT_DATETIME = "yyyy-mm-dd HH:MM:ss";

  /**
   * 将日期格式化成 yyyy-mm-dd格式
   */
  public static String formatDate(LocalDate date) {
    return formatDate(date, FORMAT_DATE);
  }

  /**
   * 将日期格式化成指定格式
   */
  public static String formatDate(LocalDate date, String format) {
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(format);
    return date.format(dateTimeFormatter);
  }

  /**
   * 将时间格式化成 yyyy-mm-dd HH:MM:ss 格式
   */
  public static String formatDateTime(LocalDateTime datetime) {
    return formatDateTime(datetime, FORMAT_DATETIME);
  }

  /**
   * 将时间格式化成指定格式
   */
  public static String formatDateTime(LocalDateTime dateTime, String format) {
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(format);
    return dateTime.format(dateTimeFormatter);
  }

  /**
   * 获取 Date 对应的时间戳
   */
  public static long getTimestamp(Date date) {
    return date.getTime();
  }

  /**
   * 获取 LocalDate 对应的时间戳
   */
  public static long getTimestamp(LocalDate localDate) {
    return localDate.atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli();
  }

  /**
   * 获取 LocalDateTime 对应的时间戳
   */
  public static long getTimestamp(LocalDateTime localDateTime) {
    return localDateTime.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
  }

  /**
   * 根据时间戳获取 Date
   */
  public static Date parseDate(long timestamp) {
    return new Date(timestamp);
  }

  /**
   * 根据时间戳获取 LocalDate
   */
  public static LocalDate parseLocalDate(long timestamp) {
    return Instant.ofEpochMilli(timestamp).atZone(ZoneId.systemDefault()).toLocalDate();
  }

  /**
   * 根据时间戳获取 LocalDateTime
   */
  public static LocalDateTime parseLocalDateTime(long timestamp) {
    return Instant.ofEpochMilli(timestamp).atZone(ZoneId.systemDefault()).toLocalDateTime();
  }

  /**
   * 转换为 Date
   */
  public static Date convertToDate(LocalDate localDate) {
    return Date.from(localDate.atStartOfDay(ZoneId.systemDefault()).toInstant());
  }

  /**
   * 转换为 LocalDate
   */
  public static LocalDate convertToLocalDate(Date date) {
    return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
  }

  /**
   * 转换为 LocalDateTime
   */
  public static LocalDateTime convertToLocalDateTime(Date date) {
    return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
  }
}
```

