# IDEA模板相关标准

## 设置自动导入和优化引用的包

File-->Settings

![](https://files.catbox.moe/f62d12.png)

## 设置代码自动格式化

File->Settings->plugins 

搜索Eclipse Code Formatter，点击安装

搜索save actions，点击安装

重启idea后进行配置；

File->Settings->Eclipse Code Formatter ，Rename重命名为thunisoft，配置thunisoft_format.xml,

thunisoft_format.xml文件从团队的共享文件夹获取

共享文件夹地址：`\\Usermic-rm2uled\讯问指挥团队共享文件夹` 或  ` http://172.18.33.131/`

![](https://ftp.bmp.ovh/imgs/2021/09/60c54da05762b887.png)

File->Settings->save actions 

![](https://ftp.bmp.ovh/imgs/2021/09/62d1b6371472d212.png)

## 设置生成文件时注释自动添加

File->Setting->Editor->File and Code Templates->Includes

添加如下图配置

![](https://ftp.bmp.ovh/imgs/2021/09/9183a9c5dd399a2d.png)

ThunisoftJavaClass

```
/**
 * ${NAME}
 * @description ${description}
 * @author gaoweihan
 * @date ${DATE} ${TIME}
 *
 */
```

然后选择Files

分别修改Class Interface Enum Annotation的配置如下图

![image-20210909143228387](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210909143228387.png)

class

```
#parse("ThunisoftJavaClass.java")
public class ${NAME} {
}
```

Interface

```
#parse("ThunisoftJavaClass.java")
public interface ${NAME} {
}
```

Enum

```
#parse("ThunisoftJavaClass.java")
public enum ${NAME} {
}
```

Annotation

```
#parse("ThunisoftJavaClass.java")
public @interface ${NAME} {
}
```

## 使用模版来快速增加类或方法的注释

File->Setting->Editor->Live Templates 

![](https://files.catbox.moe/92i949.png)

然后在该分组下创建模版

![](https://files.catbox.moe/bvt5p1.png)

### 类注释模板详细配置如下：

![](https://files.catbox.moe/533s2d.png)

```
/**
 * $NAME$
 * @description $description$
 * @author gaoweihan
 * @date $date$ $time$ 
 *
 */
```

点击Edit variables为注释注入值

![](https://files.catbox.moe/84808u.png)

### 方法注释模板详细配置如下：

方法注释模板需要在方法内才能识别方法的param和return属性，所以这里先输入/*将方法包围在输入m+ENTER即可在方法顶部生成相关注释

![](https://files.catbox.moe/feyoon.png)

```
*
 * $name$
 * 
 * @author gaoweihan
 * @date $date$ $time$
 * $params$$returns$
 */
```

returns

```
groovyScript("def p=\"${_1}\";if(p=='null'||p=='void'){null}else{'@return '+\"${_1}\"}", methodReturnType())
```

params:

```
groovyScript("def result=''; def params=\"${_1}\".replaceAll('\\\\[|\\\\]|\\\\s','').split(',').toList(); for(i = 0; i < params.size(); i++) {if(params[i].size()==0)continue;result+='@param ' + params[i] + ' ' + params[i] + '\\n * '}; return result", methodParameters())
```

### 使用演示：

类注释

![](https://ftp.bmp.ovh/imgs/2021/09/1117478103b59d28.gif)

方法注释

![](https://ftp.bmp.ovh/imgs/2021/09/a14dfea9d4bfdd55.gif)