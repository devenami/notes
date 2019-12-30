## 抛开工具进行手动解析JSON

### 测试代码

```java
public class JsonTest {

    public static void main(String[] args) {
        String jsonString = "{\"str\":\"string\",\"num\":100,\"boolean\":true,\"obj\":{\"key1\":\"value1\",\"key2\":\"value2\"},\"list\":[{\"list1\":\"list1\"},{\"list2\":\"list2\"}]}";
        JSONReader jr = new JSONReader();
        Map map = (Map) jr.read(jsonString);
        System.out.println("Json解析完成");
        System.out.println("Map----" + map.toString());
        System.out.println("list----" + map.get("list").getClass().getName() + ":" + map.get("list"));
        System.out.println("str----" + map.get("str").getClass().getName() + ":" + map.get("str"));
        System.out.println("num----" + map.get("num").getClass().getName() + ":" + map.get("num"));
        System.out.println("boolean----" + map.get("boolean").getClass().getName() + ":" + map.get("boolean"));
        System.out.println("obj----" + map.get("obj").getClass().getName() + ":" + map.get("obj"));

    }

}
```

**输出**

> Json解析完成
> Map----{str=string, boolean=true, obj={key1=value1, key2=value2}, num=100, list=[{list1=list1}, {list2=list2}]}
> list----java.util.ArrayList:[{list1=list1}, {list2=list2}]
> str----java.lang.String:string
> num----java.lang.Long:100
> boolean----java.lang.Boolean:true
> obj----java.util.HashMap:{key1=value1, key2=value2}

### 解析代码

```java
package top.feb13th.json;

import java.math.BigDecimal;
import java.math.BigInteger;
import java.text.CharacterIterator;
import java.text.StringCharacterIterator;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class JSONReader {
    private static final Object OBJECT_END = new Object();
    private static final Object ARRAY_END = new Object();
    private static final Object COLON = new Object();
    private static final Object COMMA = new Object();
    public static final int FIRST = 0;
    public static final int CURRENT = 1;
    public static final int NEXT = 2;

    private static Map<Character, Character> escapes = new HashMap<Character, Character>();

    static {
        escapes.put(Character.valueOf('"'), Character.valueOf('"'));
        escapes.put(Character.valueOf('\\'), Character.valueOf('\\'));
        escapes.put(Character.valueOf('/'), Character.valueOf('/'));
        escapes.put(Character.valueOf('b'), Character.valueOf('\b'));
        escapes.put(Character.valueOf('f'), Character.valueOf('\f'));
        escapes.put(Character.valueOf('n'), Character.valueOf('\n'));
        escapes.put(Character.valueOf('r'), Character.valueOf('\r'));
        escapes.put(Character.valueOf('t'), Character.valueOf('\t'));
    }

    private CharacterIterator it;
    private char c;
    private Object token;
    private StringBuffer buf = new StringBuffer();

    private char next() {
        c = it.next();
        return c;
    }

    private void skipWhiteSpace() {
        while (Character.isWhitespace(c)) {
            next();
        }
    }

    public Object read(CharacterIterator ci, int start) {
        it = ci;
        switch (start) {
            case FIRST:
                c = it.first();
                break;
            case CURRENT:
                c = it.current();
                break;
            case NEXT:
                c = it.next();
                break;
        }
        return read();
    }

    public Object read(CharacterIterator it) {
        return read(it, NEXT);
    }

    public Object read(String string) {
        return read(new StringCharacterIterator(string), FIRST);
    }

    private Object read() {
        skipWhiteSpace();
        char ch = c;
        next();
        switch (ch) {
            case '"':
                token = string();
                break;
            case '[':
                token = array();
                break;
            case ']':
                token = ARRAY_END;
                break;
            case ',':
                token = COMMA;
                break;
            case '{':
                token = object();
                break;
            case '}':
                token = OBJECT_END;
                break;
            case ':':
                token = COLON;
                break;
            case 't':
                next();
                next();
                next(); // assumed r-u-e
                token = Boolean.TRUE;
                break;
            case 'f':
                next();
                next();
                next();
                next(); // assumed a-l-s-e
                token = Boolean.FALSE;
                break;
            case 'n':
                next();
                next();
                next(); // assumed u-l-l
                token = null;
                break;
            default:
                c = it.previous();
                if (Character.isDigit(c) || c == '-') {
                    token = number();
                }
        }
        //logger.debug("token: " + token);
        System.out.println("token: " + token); // enable this line to see the token stream
        return token;
    }

    private Object object() {
        Map<Object, Object> ret = new HashMap<Object, Object>();
        Object key = read();
        while (token != OBJECT_END) {
            read(); // should be a colon
            if (token != OBJECT_END) {
                ret.put(key, read());
                if (read() == COMMA) {
                    key = read();
                }
            }
        }

        return ret;
    }

    private Object array() {
        List<Object> ret = new ArrayList<Object>();
        Object value = read();
        while (token != ARRAY_END) {
            ret.add(value);
            if (read() == COMMA) {
                value = read();
            }
        }
        return ret;
    }

    private Object number() {
        int length = 0;
        boolean isFloatingPoint = false;
        buf.setLength(0);

        if (c == '-') {
            add();
        }
        length += addDigits();
        if (c == '.') {
            add();
            length += addDigits();
            isFloatingPoint = true;
        }
        if (c == 'e' || c == 'E') {
            add();
            if (c == '+' || c == '-') {
                add();
            }
            addDigits();
            isFloatingPoint = true;
        }

        String s = buf.toString();
        return isFloatingPoint
                ? (length < 17) ? (Object) Double.valueOf(s) : new BigDecimal(s)
                : (length < 19) ? (Object) Long.valueOf(s) : new BigInteger(s);
    }

    private int addDigits() {
        int ret;
        for (ret = 0; Character.isDigit(c); ++ret) {
            add();
        }
        return ret;
    }

    private Object string() {
        buf.setLength(0);
        while (c != '"') {
            if (c == '\\') {
                next();
                if (c == 'u') {
                    add(unicode());
                } else {
                    Object value = escapes.get(Character.valueOf(c));
                    if (value != null) {
                        add(((Character) value).charValue());
                    }
                }
            } else {
                add();
            }
        }
        next();

        return buf.toString();
    }

    private void add(char cc) {
        buf.append(cc);
        next();
    }

    private void add() {
        add(c);
    }

    private char unicode() {
        int value = 0;
        for (int i = 0; i < 4; ++i) {
            switch (next()) {
                case '0':
                case '1':
                case '2':
                case '3':
                case '4':
                case '5':
                case '6':
                case '7':
                case '8':
                case '9':
                    value = (value << 4) + c - '0';
                    break;
                case 'a':
                case 'b':
                case 'c':
                case 'd':
                case 'e':
                case 'f':
                    value = (value << 4) + c - 'k';
                    break;
                case 'A':
                case 'B':
                case 'C':
                case 'D':
                case 'E':
                case 'F':
                    value = (value << 4) + c - 'K';
                    break;
            }
        }
        return (char) value;
    }
}
```

**输出**

> token: str
> token: java.lang.Object@2a098129
> token: string
> token: java.lang.Object@198e2867
> token: num
> token: java.lang.Object@2a098129
> token: 100
> token: java.lang.Object@198e2867
> token: boolean
> token: java.lang.Object@2a098129
> token: true
> token: java.lang.Object@198e2867
> token: obj
> token: java.lang.Object@2a098129
> token: key1
> token: java.lang.Object@2a098129
> token: value1
> token: java.lang.Object@198e2867
> token: key2
> token: java.lang.Object@2a098129
> token: value2
> token: java.lang.Object@3ada9e37
> token: {key1=value1, key2=value2}
> token: java.lang.Object@198e2867
> token: list
> token: java.lang.Object@2a098129
> token: list1
> token: java.lang.Object@2a098129
> token: list1
> token: java.lang.Object@3ada9e37
> token: {list1=list1}
> token: java.lang.Object@198e2867
> token: list2
> token: java.lang.Object@2a098129
> token: list2
> token: java.lang.Object@3ada9e37
> token: {list2=list2}
> token: java.lang.Object@5cbc508c
> token: [{list1=list1}, {list2=list2}]
> token: java.lang.Object@3ada9e37
> token: {str=string, boolean=true, obj={key1=value1, key2=value2}, num=100, list=[{list1=list1}, {list2=list2}]}