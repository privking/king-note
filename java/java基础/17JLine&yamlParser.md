JLine & yamlPare & textFormatter

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/jline/jline -->
    <dependency>
        <groupId>jline</groupId>
        <artifactId>jline</artifactId>
        <version>2.14.6</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.11</version>
    </dependency>

</dependencies>
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>priv.king.command.Main</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </plugin>
    </plugins>
</build>


  <dependency>
            <groupId>org.yaml</groupId>
            <artifactId>snakeyaml</artifactId>
            <version>1.26</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.11</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.11</version>
        </dependency>
```

## Main方法

```java
public class Main {
    public static void main(String[] args) {
        MapData mapData = new MapData();
        try {
            ConsoleReader reader = new ConsoleReader();
            String line;
            System.out.println("welcome to dataCenter");
            while (!(line = reader.readLine("cmd>")).equals("exit")) {
                if (line.equals("list")) {
                    System.out.print(mapData.getTable());
                } else if (line.startsWith("add")) {
                    String[] s = line.split(" ");
                    if (s.length < 3) System.out.println("error command");
                    else {
                        mapData.addData(s[1], s[2]);
                        System.out.println("success add: " + s[1] + "->" + s[2]);
                    }
                } else if (line.startsWith("del")) {
                    String[] s = line.split(" ");
                    if (s.length < 2) System.out.println("error command");
                    else {
                        String value = mapData.remove(s[1]);
                        System.out.println("success del: " + s[1] + "->" + value);
                    }
                }else{
                    System.out.println("error command");
                }

            }
            System.out.println("bye");
            System.exit(0);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 格式化工具类

```java
package priv.king.command.util.formatter;

import org.apache.commons.lang3.StringUtils;

import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;

/**
 * 生成表格字符串
 * 纯英文 完美对齐
 * 中文略有偏差
 */
public class TextForm {
    /**
     * 左边距
     */
    protected int paddingL = 1;
    /**
     * 右边距
     */
    protected int paddingR = 1;
    /**
     * 标题
     */
    protected List<String> title = new ArrayList<>();
    /**
     * 数据
     */
    protected List<List<String>> datas = new ArrayList<>();
    /**
     * 最大列数
     */
    protected int maxCol = 0;
    /**
     * 每个单元格最大字符数
     */
    protected int colMaxLength = Integer.MAX_VALUE;
    /**
     * 表格组成符号
     */
    protected char separatorCol = '|';

    protected char separatorRow = '-';

    private TextForm() {
    }

    public static TextFormBuilder builder() {
        return new TextFormBuilder(new TextForm());
    }


    /**
     * 直接sout
     */
    public void printConsole() {
        System.out.println(printString());
    }


    /**
     * 直接返回string
     */
    public String printString() {
        List<List<String>> formData = new ArrayList<>();
        formData.add(title);
        formData.addAll(datas);
        //计算每列最长元素长度(中文2个长度，字母一个长度)
        Map<Integer, Integer> colMaxLengthMap = colMaxLength(formData);

        //格式化数据
        for (int i = 0; i < formData.size(); i++) {
            List<String> row = formData.get(i);
            for (int j = 0; j < row.size(); j++) {
                Formatter formatter = new Formatter();
                String str = row.get(j);
                int chineseNum = getChineseNum(str);
                Integer maxLength = colMaxLengthMap.get(j);
                //截取字符串
                if (maxLength == colMaxLength) {
                    str = subString(str, chineseNum, maxLength);
                }
                //将字符串补齐到统一长度(maxLength)
                String val = formatter.format("%-" + (maxLength - chineseNum) + "s", str).toString();
                row.set(j, val);
            }
        }
        ////////////////////////////////////////////////////////////////////////
        //输出 数据+边框
        String line = "";
        List<String> rows = new ArrayList<>();
        for (List<String> strings : formData) {
            String pL = StringUtils.repeat(" ", paddingL);
            String pR = StringUtils.repeat(" ", paddingR);
            String row = separatorCol + pL + String.join(pL + separatorCol + pR, strings) + pR + separatorCol;
            if (line.length() < row.length()) {
                line = StringUtils.repeat(separatorRow, row.length());
            }
            rows.add(row);
        }
        StringBuffer buffer = new StringBuffer();
        buffer.append(line).append("\n");
        for (String row : rows) {
            buffer.append(row).append("\n").append(line).append("\n");
        }
        return buffer.toString();
    }

    /**
     * 截取字符串
     *
     * @param str        原始字符串
     * @param chineseNum 中文数量
     * @param maxLength  最长长度
     * @return
     */
    private String subString(String str, Integer chineseNum, Integer maxLength) {
        //截取
        StringBuffer stringBuffer = new StringBuffer();
        int maxLenTemp = 0;
        for (char c : str.toCharArray()) {
            maxLenTemp++;
            if (getChineseNum(String.valueOf(c)) == 1) {
                maxLenTemp++;
            }
            if (maxLenTemp >= colMaxLength) {
                break;
            }
            stringBuffer.append(c);
        }
        return stringBuffer.toString();


    }


    /**
     * 找到每一列最大的长度
     *
     * @param formData
     * @return
     */
    private Map<Integer, Integer> colMaxLength(List<List<String>> formData) {
        Map<Integer, Integer> map = new HashMap<>();
        //每一行遍历
        for (int i = 0; i < formData.size(); i++) {
            int col = 0;
            //获取每一行的元素集合
            List<String> strings = formData.get(i);
            while (strings.size() > col) {
                //获取到具体的值
                String val = strings.get(col);
                //给 中文算两个长度
                int length = val.length() + getChineseNum(val);
                Integer integer = map.get(col);
                if (integer == null) {
                    map.put(col, Math.min(length, colMaxLength));
                } else {
                    if (integer < length) {
                        map.put(col, Math.min(length, colMaxLength));
                    }
                }
                col++;
            }
        }
        return map;
    }

    /**
     * 获取中文数量
     *
     * @param val
     * @return
     */
    private int getChineseNum(String val) {
        if (val == null) {
            val = "";
        }
        String regex = "[^\\x00-\\xff]";
        ArrayList<String> list = new ArrayList<String>();
        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(val);
        while (matcher.find()) {
            list.add(matcher.group());
        }
        int size = list.size();
        return size;
    }

    public static class TextFormBuilder {

        private TextForm textForm;

        protected TextFormBuilder(TextForm textForm) {
            this.textForm = textForm;
        }

        public TextFormBuilder title(String... titles) {
            if (textForm.maxCol < titles.length) {
                textForm.maxCol = titles.length;
            }
            for (String title : titles) {
                if (title == null) {
                    title = "null";
                }
                textForm.title.add(title);
            }
            return this;
        }

        public TextFormBuilder title(List<String> titles) {
            if (textForm.maxCol < titles.size()) {
                textForm.maxCol = titles.size();
            }
            textForm.title = titles.stream().map(x -> {
                if (x == null) {
                    x = "null";
                }
                return x;
            }).collect(Collectors.toList());
            return this;
        }


        public TextFormBuilder paddingL(int paddingL) {
            textForm.paddingL = paddingL;
            return this;
        }

        public TextFormBuilder paddingR(int paddingR) {
            textForm.paddingR = paddingR;
            return this;
        }

        public TextFormBuilder separatorCol(char separator) {
            textForm.separatorCol = separator;
            return this;
        }

        public TextFormBuilder separatorRow(char separator) {
            textForm.separatorRow = separator;
            return this;
        }

        /**
         * 最长字符串长度，字母算一个长度，中文算两个长度
         *
         * @param colMaxLength
         * @return
         */
        public TextFormBuilder colMaxLength(int colMaxLength) {
            textForm.colMaxLength = colMaxLength;
            return this;
        }

        public TextFormBuilder addRow(String... cols) {
            if (textForm.maxCol < cols.length) {
                textForm.maxCol = cols.length;
            }
            List<String> list = new ArrayList<>(cols.length);
            for (String col : cols) {
                if (col == null) {
                    col = "null";
                }
                list.add(col);
            }
            textForm.datas.add(list);
            return this;
        }

        public TextFormBuilder addRow(List<String> cols) {
            if (textForm.maxCol < cols.size()) {
                textForm.maxCol = cols.size();
            }
            List<String> collect = cols.stream().map(x -> {
                if (x == null) {
                    x = "null";
                }
                return x;
            }).collect(Collectors.toList());
            textForm.datas.add(collect);
            return this;
        }


        public TextFormBuilder addRows(List<List<String>> cols) {
            cols.forEach(x -> addRow(x));
            return this;
        }

        public TextForm build() {
            int titleSize = textForm.title.size();
            if (titleSize < textForm.maxCol) {
                for (int i = 0; i < textForm.maxCol - titleSize; i++) {
                    textForm.title.add(null);
                }
            }
            for (List<String> data : textForm.datas) {
                int dataSize = data.size();
                if (dataSize < textForm.maxCol) {
                    for (int i = 0; i < textForm.maxCol - dataSize; i++) {
                        data.add(null);
                    }
                }
            }
            return textForm;
        }

    }

}
```

## Yaml解析类

```java
package priv.king.command.util.config;


import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.yaml.snakeyaml.Yaml;
import org.yaml.snakeyaml.constructor.Constructor;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.*;

public class YamlParser {
    private static final Logger logger = LoggerFactory.getLogger(YamlParser.class);

    /**
     * yml文件流转成单层map
     * 转Properties 改变了顺序
     *
     * @param
     * @return
     */
    public static Map<String, Object> yamlToFlattenedMap(InputStream inputStream) {
        Yaml yaml = createYaml();
        Map<String, Object> map=new HashMap<>();
        for (Object object : yaml.loadAll(inputStream)) {
            if (object != null) {
                map = asMap(object);
                map=getFlattenedMap(map);
            }
        }
        return map;
    }

    /**
     * yml文件流转成多次嵌套map
     *
     * @param
     * @return
     */
    public static Map<String, Object> yamlToMultilayerMap(InputStream inputStream) {
        Yaml yaml = createYaml();
        Map<String, Object> result = new LinkedHashMap<>();
        for (Object object : yaml.loadAll(inputStream)) {
            if (object != null) {
                result.putAll(asMap(object));
            }
        }
        return result;
    }

    /**
     * 多次嵌套map转成yml
     *
     *
     * @return
     */
    public static String multilayerMapToYaml(Map<String, Object> map) {
        Yaml yaml = createYaml();
        return yaml.dumpAsMap(map);
    }

    /**
     * 单层map转成yml
     *
     *
     * @return
     */
    public static String flattenedMapToYaml(Map<String, Object> map) {
        Yaml yaml = createYaml();
        return yaml.dumpAsMap(flattenedMapToMultilayerMap(map));
    }

    /**
     * 单层map转换多层map
     *
     * @param map
     * @return
     */
    private static Map<String, Object> flattenedMapToMultilayerMap(Map<String, Object> map) {
        Map<String, Object> result = getMultilayerMap(map);
        return result;
    }

    private static Yaml createYaml() {
        return new Yaml(new Constructor());
    }

    @SuppressWarnings("unchecked")
    private static Map<String, Object> asMap(Object object) {
        Map<String, Object> result = new LinkedHashMap<>();
        if (!(object instanceof Map)) {
            result.put("document", object);
            return result;
        }

        Map<Object, Object> map = (Map<Object, Object>) object;
        for (Map.Entry<Object, Object> entry : map.entrySet()) {
            Object value = entry.getValue();
            if (value instanceof Map) {
                value = asMap(value);
            }
            Object key = entry.getKey();
            if (key instanceof CharSequence) {
                result.put(key.toString(), value);
            } else {
                result.put("[" + key.toString() + "]", value);
            }
        }
        return result;
    }

    private static Map<String, Object> getFlattenedMap(Map<String, Object> source) {
        Map<String, Object> result = new LinkedHashMap<>();
        buildFlattenedMap(result, source, null);
        return result;
    }

    private static void buildFlattenedMap(Map<String, Object> result, Map<String, Object> source, String path) {
        for (Map.Entry<String, Object> entry : source.entrySet()) {
            String key = entry.getKey();
            if (!StringUtils.isBlank(path)) {
                if (key.startsWith("[")) {
                    key = path + key;
                } else {
                    key = path + '.' + key;
                }
            }
            Object value = entry.getValue();
            if (value instanceof String) {
                result.put(key, value);
            } else if (value instanceof Map) {
                @SuppressWarnings("unchecked")
                Map<String, Object> map = (Map<String, Object>) value;
                buildFlattenedMap(result, map, key);
            } else if (value instanceof Collection) {
                @SuppressWarnings("unchecked")
                Collection<Object> collection = (Collection<Object>) value;
                int count = 0;
                for (Object object : collection) {
                    buildFlattenedMap(result, Collections.singletonMap("[" + (count++) + "]", object), key);
                }
            } else {
                result.put(key, (value != null ? value.toString() : ""));
            }
        }
    }

    private static Map<String, Object> getMultilayerMap(Map<String, Object> source) {
        Map<String, Object> rootResult = new LinkedHashMap<>();
        for (Map.Entry<String, Object> entry : source.entrySet()) {
            String key = entry.getKey();
            buildMultilayerMap(rootResult, key,entry.getValue());
        }
        return rootResult;
    }

    @SuppressWarnings("unchecked")
    private static void buildMultilayerMap(Map<String, Object> parent, String path,Object value) {
        String[] keys = StringUtils.split(path,".");
        String key = keys[0];
        if (key.endsWith("]")) {
            String listKey=key.substring(0,key.indexOf("["));
            String listPath=path.substring(key.indexOf("["));
            List<Object> chlid =  bulidChlidList(parent, listKey);
            buildMultilayerList(chlid, listPath, value);
        }else{
            if (keys.length == 1) {
                parent.put(key, stringToObj(value.toString()));
            }else{
                String newpath = path.substring(path.indexOf(".") + 1);
                Map<String, Object> chlid = bulidChlidMap(parent, key);;
                buildMultilayerMap(chlid, newpath,value);
            }
        }
    }


    @SuppressWarnings("unchecked")
    private static void buildMultilayerList(List<Object> parent,String path,Object value) {
        String[] keys = StringUtils.split(path,".");
        String key = keys[0];
        int index=Integer.valueOf(key.replace("[", "").replace("]", ""));
        if (keys.length == 1) {
            parent.add(index,stringToObj(value.toString()));
        } else {
            String newpath = path.substring(path.indexOf(".") + 1);
            Map<String, Object> chlid = bulidChlidMap(parent, index);;
            buildMultilayerMap(chlid, newpath, value);
        }
    }


    @SuppressWarnings("unchecked")
    private static Map<String, Object> bulidChlidMap(Map<String, Object> parent,String key){
        if (parent.containsKey(key)) {
            return (Map<String, Object>) parent.get(key);
        } else {
            Map<String, Object> chlid = new LinkedHashMap<>(16);
            parent.put(key, chlid);
            return chlid;
        }
    }

    @SuppressWarnings("unchecked")
    private static Map<String, Object> bulidChlidMap(List<Object> parent,int index){
        Map<String, Object> chlid = null;
        try{
            Object obj=parent.get(index);
            if(null != obj){
                chlid = (Map<String, Object>)obj;
            }
        }catch(Exception e){
            logger.warn("get list error");
        }

        if (null == chlid) {
            chlid = new LinkedHashMap<>(16);
            parent.add(index,chlid);
        }
        return chlid;
    }

    @SuppressWarnings("unchecked")
    private static List<Object> bulidChlidList(Map<String, Object> parent,String key){
        if (parent.containsKey(key)) {
            return (List<Object>) parent.get(key);
        } else {
            List<Object> chlid = new ArrayList<>(16);
            parent.put(key, chlid);
            return chlid;
        }
    }

    private static Object stringToObj(String obj){
        Object result=null;
        if(obj.equals("true") || obj.equals("false")){
            result=Boolean.valueOf(obj);
        }else if(isBigDecimal(obj)){
            if(obj.indexOf(".") == -1){
                result=Long.valueOf(obj.toString());
            }else{
                result=Double.valueOf(obj.toString());
            }
        }else{
            result=obj;
        }
        return result;
    }


    public static boolean isBigDecimal(String str){
        if(str==null || str.trim().length() == 0){
            return false;
        }
        char[] chars = str.toCharArray();
        int sz = chars.length;
        int i = (chars[0] == '-') ? 1 : 0;
        if(i == sz) return false;

        if(chars[i] == '.') return false;//除了负号，第一位不能为'小数点'

        boolean radixPoint = false;
        for(; i < sz; i++){
            if(chars[i] == '.'){
                if(radixPoint) return false;
                radixPoint = true;
            }else if(!(chars[i] >= '0' && chars[i] <= '9')){
                return false;
            }
        }
        return true;
    }


    public static void main(String[] args) throws IOException {
        File file = new File(YamlParser.class.getResource("/").getFile() + "test.yaml");

        Map<String, Object> a= YamlParser.yamlToFlattenedMap(new FileInputStream(file));
        System.out.println(a);

        Map<String, Object> b = YamlParser.yamlToMultilayerMap(new FileInputStream(file));
        System.out.println(b);


        String c = YamlParser.multilayerMapToYaml(b);
        System.out.println(c);
        String d = YamlParser.flattenedMapToYaml(a);
        System.out.println(d);

    }



}
```

## 加载yaml

```java
package priv.king.command.util.config;

import java.io.File;
import java.io.FileInputStream;
import java.util.HashMap;
import java.util.Map;

public class YamlConfigLoader implements ConfigLoader {

    private String fileName = "config.yaml";

    private Map<String, Object> configs = new HashMap<>();

    public YamlConfigLoader(String path) {
        try {
            File file = null;
            if (path != null && path.trim() != "") {
                file = new File(path);
            }
            file = new File(YamlConfigLoader.class.getResource("/").getFile() + fileName);
            configs = YamlParser.yamlToFlattenedMap(new FileInputStream(file));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    @Override
    public Integer getIntOrDefault(String name, Integer defaultValue) {
        Object data = configs.get(name);
        if (data == null) {
            return defaultValue;
        } else {
            try {
                return Integer.parseInt(String.valueOf(data));
            } catch (Throwable t) {
                return defaultValue;
            }
        }
    }

    @Override
    public String getStringOrDefault(String name, String defaultValue) {
        Object data = configs.get(name);
        if (data == null) {
            return defaultValue;
        } else {
            try {
                return data.toString();
            } catch (Throwable t) {
                return defaultValue;
            }
        }
    }

    @Override
    public Double getDoubleOrDefault(String name, Double defaultValue) {
        Object data = configs.get(name);
        if (data == null) {
            return defaultValue;
        } else {
            try {
                return Double.parseDouble(String.valueOf(data));
            } catch (Throwable t) {
                return defaultValue;
            }
        }
    }

    @Override
    public char getCharOrDefault(String name, char defaultValue) {
        Object data = configs.get(name);
        if (data == null) {
            return defaultValue;
        } else {
            try {
                return data.toString().toCharArray()[0];
            } catch (Throwable t) {
                return defaultValue;
            }
        }
    }
}
```

