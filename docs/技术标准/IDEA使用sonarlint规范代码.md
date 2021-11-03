------



## **一、安装SonarLint插件**

   File–>Settings–>Plugins—>Marketplace ,搜索sonarLint,在搜索列表中选择SonarLint进行安装，根据提示重启idea即可。

​	SonarLint安装失败或Marketplace 内搜索失败可以等网络稳定再次尝试，或设置插件安装的代理服务器

##  **二、配置Sonar**

### 1.初始配置

1.   File–>Settings–>Tools->SonarLint ,新增sonar连接

   ![](https://files.catbox.moe/t6iysq.png)

   

2. 选择sonarQube，配置sonar地址为：http://dl-sonar.thunisoft.com/

   ![](https://files.catbox.moe/6khy6o.png)

   

3. 在gitlab中获取token并把token信息填入以在sonarlink验证身份

   ![](https://files.catbox.moe/k9lnz8.png)![image-20210906114852233](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210906114852233.png)

   ![](https://files.catbox.moe/8z59rf.png)

   ![](https://files.catbox.moe/ewfnzp.png)

   ![](https://files.catbox.moe/q5mpsy.png)

4. 界面显示successfully表示创建成功，点击finish结束

   ![](https://files.catbox.moe/fq5ioi.png)

5. 结束后回到新增页面点击"Update binding"，从服务器获取项目列表信息和同步相关规则![](https://files.catbox.moe/fdfkou.png)

6. SonarLint -> project Settings勾选Bind project to SonarQube/SonarCloud    Connection 下拉选择之前创建的连接名，点击 Search in list选择对应sonar项目标识/项目名，配置完成后点击OK

![](https://files.catbox.moe/w9l2dq.png)

### 2.其他可选配置

#### 1)设置sonar检查时忽略resources文件夹

对项目设置

![](https://files.catbox.moe/o6jnah.png)

#### 2)设置提交代码前进行soanr代码检查

该配置仅在IDEA中生效

提交前设置相关选项

![](https://files.catbox.moe/5ly3fz.png)

点击Review issues查看要提交代码的问题

![](https://files.catbox.moe/5tv52b.png)

## 三、使用SonarLint

### 1.自动分析

 File–>Settings–>SonarLint  勾选 Automatically tigger analysis 应用后确认保存，项目打开和添加新代码时会自动进行sonar检查（可能会发生卡顿或卡死的情况，如发生建议关闭自动分析或详见其他事项5）

### 2.手动分析

​	1.在项目目录结构中选择要分析的文件夹或是代码文件（该方式会扫描配置好的忽略文件）,右键菜单**Anaylyze**->**Analyze with SonarLint**  或者是右键 **SonarLint** ->**Analyze with SonarLint **
或者选中后使用快捷键  Ctrl+Shift+S（分析文件过大过多可能会导致IDEA卡死-.-）

​	2.在**sonarLint**菜单下使用相关按钮分析全部文件（该方式不会扫描配置好的忽略文件）

​	![](https://files.catbox.moe/321rj5.png)

​	3.在当前编辑的文件内容中右键直接点击**Analyze with SonarLint **或使用快捷键 Ctrl+Shift+S

### 3.分析结果

​	分析完成的结果会在SonarLint Tool View中显示出检测的问题，点击issue，在右侧会出现对应的Rule，可参照提示进行修改

![](https://files.catbox.moe/07c1fq.png)

以类名称进行分类。各类的issue，分为阻断、严重、主要、提示和次要，问题严重性依次降低，issue对应图标如下。

![](https://files.catbox.moe/0k0mw0.png)

## 四、其他事项

### 1.sonarLint窗口误关闭

可以点击菜单 **view**->**Tool Windows**中找到 **SonarLint**.重新打开SonarLint窗口

### 2.开启IDEA后发现sonarLint未启动

 File–>Settings–>Plugins—>Installed搜索**SonarLint** 勾选后根据提示重启IDEA

### 3.sonarLint快捷键冲突

File–>Settings 搜索**keyMap**,再在右侧搜索sonar,找到SonarLint的快捷键设置,修改为想使用的快捷键.

### 4.打开项目时sonar分析时间过长

 File–>Settings–>SonarLint 取消勾选 Automatically tigger analysis 应用后确认保存

### 5.sonar分析导致IDEA卡死，提示内存溢出

任务管理器关闭后重启，或修改idea64.exe.vmoptions中的设定值Xmx ：`JVM最大堆内存`(该值越小IDEA运行越慢)以提高内存容量

