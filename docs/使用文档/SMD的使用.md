# SMD的使用

## SMD说明

### 文件结构

- bin\        包括`bat文件`和`control.xml`

- changelog\     包括升级`日志文件`

- doc\        包括`使用说明文档`、模板文件和检查用VBA脚本

- lib\        运行时依赖的`jar文件`

### 使用指南

​			SMD可以同过命令行的形式执行，也可以通过.bat脚本执行

​			可以使用SMD生成SQL和前后台代码,还可以通过SMD直接把SQL执行到数据库中。

### SMD安装

> ​	从maven仓库下载最新版本的SMD包（zip格式）。
>
> ​	下载地址：http://repo.thunisoft.com/maven2/index.html#nexus-search;quick~smd
>
> ​	打开`bin\smd.bat`文件，修改`set SMD_HOME=`的值为`bin文件夹所在的目录`即可。默认情况下，SMD使用环境变量	SMD_HOME查找control文件。
>
> ​	也可以将`%SMD_HOME%\bin`加入到PATH环境变量中。
>
> ​	CMD下，进入SMD所在文件夹的bin目录下，通过命令行运行SMD，可以得到参数解释。代表安装成功，可以使用。



## SMD文件生成规则

### 字符规则和注意事项

> &lt;&lt;&lt;&lt; `表示数据结束，以下的数据不再解析，必须标记在第一列上`

> '---- `表示此行为注释，必须标记在第一列上`

> &gt;&gt;&gt;&gt; `表示初始化脚本，再此处做一个事务提交。写在第一列`

> C_&quest;&quest;&quest; `表示字段是varchar类型，数据类型列里写VC`

> DT_&quest;&quest;&quest; `表示字段是TIMESTAMP类型，数据类型列里写DT`

> N_&quest;&quest;&quest; `表示字段是INT类型，数据类型列里写INT`

> T_&quest;&quest;&quest; _&quest;&quest;&quest; `一般表示表格第一个下划线后的表示模式名/存储空间名,第二个下划线后表示表名`

> TAG `指定导出数据文件的%TAG%，如果没有定义，将使用命令行参数定义的TAG`

> `解析程序从第三行开始分析数据`

### 表结构的SMD

- LOGLIST

  > 编辑者的信息，用于识别修改人，修改时间。

  > | 版本  | 日期                | 作者 | 描述                                                 |
  > | ----- | ------------------- | ---- | ---------------------------------------------------- |
  > | 1.0.0 | 2020/12/14 10:48:00 | GWH  | 对&quest;&quest;&quest;做了&quest;&quest;&quest;更改 |
  > | 1.0.1 | 2020/12/14 10:49:00 | GWH2 | 对&quest;&quest;&quest;做了&quest;&quest;&quest;更改 |

- TAB

  > 表名及表位置

  > |     表中文名     |     表名     | 存储空间 | 单独大对象表空间 | 数据缓存 | 记录事务日志 | 分区 | 版本  |
  > | :--------------: | :----------: | :------: | :--------------: | :------: | :----------: | :--: | :---: |
  > |    表中文名1     | T_TAB_TABLE1 |   TAB    |                  |          |              |      | 1.0.1 |
  > |    表中文名2     | T_TAB_TABLE2 |   TAB    |                  |          |              |      | 1.0.2 |
  > | &lt;&lt;&lt;&lt; |              |          |                  |          |              |      |       |

- COL

  > 表字段结构

  > | 表名             | 字段         | 字段中文名         | 主键 | 不为空 | 数据类型 | 数据 | 数据精度 | 版本  | 说明     | 代码类型 | 扩展属性 |
  > | ---------------- | ------------ | ------------------ | ---- | ------ | -------- | ---- | -------- | ----- | -------- | -------- | -------- |
  > | ----             | 这是一行注释 |                    |      |        |          |      |          |       |          |          |          |
  > | T_TAB_TABLE1     | C_ID         | 主键               | 1    | 1      | VC       | 32   |          | 1.0.1 | 这是主键 |          |          |
  > | T_TAB_TABLE1     | C_NAME       | 名字               |      |        | VC       | 300  |          | 1.0.1 |          |          |          |
  > | &gt;&gt;&gt;&gt; |              |                    |      |        |          |      |          |       |          |          |          |
  > | ----             | 一行注释     |                    |      |        |          |      |          |       |          |          |          |
  > | T_TAB_TABLE2     | C_ID         | 主键               | 1    | 1      | VC       | 32   |          | 1.0.2 |          |          |          |
  > | T_TAB_TABLE2     | DT_TIME      | &quest;&quest;时间 |      |        | DT       |      |          | 1.0.2 |          |          |          |
  > | T_TAB_TABLE2     | N_NUMBER     | &quest;&quest;数量 |      |        | INT      |      |          | 1.0.2 |          |          |          |
  > | &gt;&gt;&gt;&gt; |              |                    |      |        |          |      |          |       |          |          |          |
  > | &lt;&lt;&lt;&lt; |              |                    |      |        |          |      |          |       |          |          |          |

-  INDEX

  > 索引

  > | 表名         | 索引名         | 索引类型 | 字段名   | 存储空间 | 单独大对象空间 | 记录事务日志 | 说明 | 版本  |
  > | ------------ | -------------- | -------- | -------- | -------- | -------------- | ------------ | ---- | ----- |
  > | ----         | 外键索引       |          |          |          |                |              |      |       |
  > | T_TAB_TABLE2 | N_NUMBER_INDEX | INT      | N_NUMBER | TAB      |                |              |      | 1.0.2 |
  > |              |                |          |          |          |                |              |      |       |

### 表数据的SMD

- LOGLIST

  > 编辑者的信息，用于识别修改人，修改时间。

  > | 版本  | 日期                | 作者 | 描述                                                 |
  > | ----- | ------------------- | ---- | ---------------------------------------------------- |
  > | 1.0.0 | 2020/12/14 10:48:00 | GWH  | 对&quest;&quest;&quest;做了&quest;&quest;&quest;更改 |

- DATA.TAB

  > 表名及表位置

  > |     表中文名     |     表名     | 存储 | TAG  | 数据范围 |
  > | :--------------: | :----------: | :--: | :--: | :------: |
  > |    表中文名1     | T_TAB_TABLE1 | TAB  |      |          |
  > | &lt;&lt;&lt;&lt; |              |      |      |          |

- DATA.COL

  > 表字段结构

  > | 表名             | 字段         | 字段索引 | 字段类型 | 格式长度 | 字段描述 |
  > | ---------------- | ------------ | -------- | -------- | -------- | -------- |
  > | ----             | 这是一行注释 |          |          |          |          |
  > | T_TAB_TABLE1     | C_ID         | 1        | 1        | 1        |          |
  > | T_TAB_TABLE1     | C_NAME       | 名字     |          |          |          |
  > | &lt;&lt;&lt;&lt; |              |          |          |          |          |

- `表名：` T_TAB_TABLE1 

  > 初始化数据

  > | C_ID             | C_NAME   |
  > | ---------------- | -------- |
  > | 111111111        | GGGGGGGG |
  > | 222222222        | WWWWWW   |
  > | 333333333        | HHHHHHHH |
  > | &lt;&lt;&lt;&lt; |          |

## SMD操作步骤

### 	SMD转SQL

#### 	配置SMD目录下的 `生成SQL.bat` 脚本

​		将写好的SMD的存放路径配置到脚本中

​		配置导出路径，如：

```sh
@echo off

setlocal
rem 设置参数，如果路径包含空格两边要用引号
set OLD_CD=%CD%
set SMD_BIN=smd.bat
set SMD_DIR=E:\summer-cmpt-smd-1.2.9.3\bin\
set DOC_PATH=E:综合管理系统SMD\  

cd %SMD_DIR%
%SMD_DIR:~0,1%:

echo %DATE% %TIME% 开始生成脚本  > %DOC_PATH%\日志\生成脚本日志.txt
rem 以下是表初始化(create)脚本的生成
echo 生成create脚本 ...
set SMD_OP=-DB ABase -OUT DBDEFINE -CTRLFILE control.xml -LOG 4
set BUILDPATH=%DOC_PATH%\sql\01_TABLES
goto realCreate
:realCreate
call %SMD_BIN% %SMD_OP% -PK PK -DIR %BUILDPATH% -METAFILE %DOC_PATH%\DBDEFINE\SMD.xls -TAG TMS >> %DOC_PATH%\日志\生成脚本日志（createdata）.txt
echo 生成create脚本完毕！

```

#### 	执行 `生成SQL.bat` 脚本

> ​		执行脚本后会提示生成脚本完毕，此时就可以在之前配置的文件储存路径下看到通过SMD生成的sql文件

### 	SMD代码自动生成

#### 		通过官网生成

> ​		[http://](http://stage.thunisoft.com/form/stageaasHome/display)[stage.thunisoft.com/form/stageaasHome/display](http://stage.thunisoft.com/form/stageaasHome/display)

#### 		项目中使用artery自动生成

> ​		右键项目选择New->New Artery->数据字典和sql

​		![image-20201214142714365.png](https://i.loli.net/2020/12/14/fCZSmhrg4eJ7Unp.png)

> ​		选择SMD文件后`->OK`，（输出目录默认为项目内，选择的SMD文件后缀名应为.xlsx）

![image-20201214142900058.png](https://i.loli.net/2020/12/14/vrMItoOAWp9PsVF.png)

> ​		生成成功后会自动在项目内生成sql文件夹存放SQL脚本，`src/main/resource/artery/dic/table`下生成表的`.dict.xml`文件
>
> ​		右键想要一键生成的表对应的`.dict.xml`文件，选择`New->New Artery->一键生成`
>
> ​		在打开的页面中确认字段信息后`->next`，选择生成表单页面或后端代码`->next`，查看或修改文件地址的配置信息`->next`
>
> ​		选择想要保留的自动生成文件(有冲突的文件默认不选中)`->Finish`
>
> ​		自动导入完成！

### 	SQL批量注入

​		运行SMD目录下的` run_windows.bat` 脚本

​		![image-20201214141921935.png](https://i.loli.net/2020/12/14/efcls91YZ2bxVMR.png)

​		在该操作界面中`->浏览`选择`config.ini`文件

​		可以在`config.ini`中修改配置也可以在图形界面上`->编辑配置文件开始配置`

![image-20201214142508892.png](https://i.loli.net/2020/12/14/yv37lzgkOLBWCMt.png)

​		配置数据库的连接信息，`->添加sql`，选择之前SMD生成的SQL文件，此时应注意SQL执行顺序如先建数据库和模式再建表。配置完成后`->保存`，然后`->开始执行`,完成注入。