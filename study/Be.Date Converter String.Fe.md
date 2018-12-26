## 前后端字符串类型日期与Date类型日期的转换和显示
### 要点提示
- 主要包括前端字符串类日期经后端处理，转换为Date类型日期并存入数据库；以及从数据库中将Date类型日期值取出，并在前端以指定格式显示。即存和取两方面。
- 如对日期显示格式有特殊要求，建议将后端model类日期类数据域及对应数据库表相关属性定义为Date类型，由此引来的问题及解决方法，即就是本文所要讲解的，否则建议将后端日期类数据域定义为String类型。
---
### 过程一：存（String -> Date）
> 在springMVC.xml文件中添加如下代码
~~~xml
<!--前端数据存储过程中，字符串类型转换为Java对应类型配置-->
    <mvc:annotation-driven conversion-service="converterAndFormatter"/>
    <bean id="converterAndFormatter" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <!--string转换为Boolean类型的配置-->
                <!--类StringToBooleanConverter不可被修改-->
                <bean class="org.springframework.core.convert.support.StringToBooleanConverter"/>
                <!--string类型日期格式化为date类型日期的配置,方法一-->
                <!--类MyDateConverter实现了Converter接口-->
                <bean class="com.huanyu.util.MyDateConverter"/>
            </set>
        </property>
    </bean>
~~~
> 其中MyDateConverter代码如下
~~~java
import org.springframework.core.convert.converter.Converter;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 字符串类型日期转换为Date类型日期
 *
 * @author huxiaolong
 * @date 2018/8/18
 */
public class MyDateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        // 指定字符串日期格式，与前端日期显示样式相同
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try {
            return simpleDateFormat.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }
}
~~~
### 过程二：取（Date - > String）
> 前端代码中添加的Date日期formatter方法可以将形如*Wed Aug 29 22:52:25 CST 2018*的日期值格式化为指定样式的字符串，而从数据库中取出的jdbc类型为Date（或TIMESTAMP）（由java类型为Date的日期对象存入）对象的值也形如*2018-08-29T22:53:27.192+0800*，并且该对象还有两个属性，其一为fastTime，为long类型毫秒数，其二为cdate，值类型与Date对象的类似，具体见下图。
而前端接收的不是Date类型对象，而是其属性fastTime，从而formatter方法无法格式化fastTime，导致异常出现。为解决此问题，比较简单的方法是在model类的Date类型数据域及getter上添加注释，注释代码如下
![image_1](https://github.com/HUANYU2015/NOTE/blob/master/image.png)

~~~java
@JsonFormat(pattern = "yyyy-MM-dd", timezone = "GMT+08:00")
~~~

![image_2](https://github.com/HUANYU2015/NOTE/blob/master/2018-12-26.png)