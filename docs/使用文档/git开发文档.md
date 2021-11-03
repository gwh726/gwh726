

# git开发文档

# 一，统一git工具

监委业务管理平台V2.5.3统一git管理工具为 **Idea中的git插件**，用该插件进行代码提交与维护

# 二，建立自己的分支

## 1.切换本地当前分支到dev分支

（develop-V2.5.3）

<img src="https://files.catbox.moe/e6r57y.png" style="zoom:80%;" />

<img src="https://files.catbox.moe/jbpk12.png" style="zoom: 67%;" />

## 2.拉取远端dev分支

<img src="https://files.catbox.moe/c364gj.png" style="zoom: 67%;" />

## 3.在最新的节点上右键新建自己的分支

<img src="https://files.catbox.moe/3j57lh.png" style="zoom: 67%;" />

分支命名规则为 **feature-版本-姓名-时间**  例：*feature-V2.5.3-mine-09081506*   当前时间为9月8日15:06

![](https://files.catbox.moe/r1nosz.png)

创建完成后就可以开始开发了

# 三，代码提交

每一块功能代码编写完成后，均进行一次代码的提交，避免一个commit的内容过多，不方便代码复查

## 1.Commit 把写好的代码提交到本地

（提交完！注意！不！要！push）

<img src="https://files.catbox.moe/kodf1m.png" style="zoom: 67%;" />

<img src="https://files.catbox.moe/j7ay6t.png" style="zoom:67%;" />

# 四，代码push

## 1.切换到dev分支，再拉取一遍

​	切换到dev分支（develop-V2.5.3）

​	同上文 ***二,建立自己的分支*** 中第一步，第二步

## 2.这时分为两种情况

### 1）情况1 dev最新的节点为自己的

![](https://files.catbox.moe/ly75du.png)

这种情况下切换到自己的分支，

![](https://files.catbox.moe/yc0iq7.png)

push上到远端然后直接提交合并申请就行,见  ***五，代码合并请求***

![](https://files.catbox.moe/dfvg28.png)

### 2）情况2 dev最新节点为他人

(注：示例分支为*feature-V2.5.3-mine-09081707*)

![](https://files.catbox.moe/ii4mz3.png)

见图，最新节点不是mine

在最新节点上拉出来一条新的分支（mine二世：*feature-V2.5.3-mine-09081721*），新建分支见 ***二,建立自己的分支*** 中第三步

![](https://files.catbox.moe/p8woad.png)

将mine一世（原来开发的分支）与mine二世（刚从新的节点上建立的分支）进行合并

![](https://files.catbox.moe/4gb7iw.png)

此时，最新节点也属于mine二世（*feature-V2.5.3-mine-09081721*）了

![](https://files.catbox.moe/zi3kbx.png)

删除掉旧的分支 mine一世（*feature-V2.5.3-mine-09081707*）

![](https://files.catbox.moe/4rtfq4.png)

将mine二世（*feature-V2.5.3-mine-09081721*）push到远端

<img src="https://files.catbox.moe/mmqp2p.png" style="zoom: 80%;" />

然后提交合并请求就可以啦，见 ***四，代码合并请求***

## 3.冲突的解决

在代码进行合并时会有可能产生冲突，原因是两个分支节点中的代码都有对同一个地方的修改，系统也无法判定到底谁修改的对，这时就需要你手动批阅了

![](https://files.catbox.moe/y2v974.png)

双击中间提示产生冲突的文件

![](https://files.catbox.moe/8cqkjf.png)

将左右两边中有用的代码都整合进中间的代码中

一定要与左边的开发者确认下，以防覆盖了别人有用的代码

![](https://files.catbox.moe/t2weoo.png)

将整合后合并

## 4.继续开发

合并请求提交后，在自己唯一存在的一个分支上继续开发，即刚刚提交合并请求的分支上继续开发，开发过程中及时，多次提交代码，阶段性地进行push，如此循环完成开发流程。

# 五，代码合并请求

## 1.打开监委业务管理平台的git

[地址](http://git.thunisoft.com/JCW_PRD_FZ_SXJDZHXT/jwywglpt)

## 2.点击合并请求

![](https://files.catbox.moe/umhl3f.png)

## 3.创建新的合并请求

![](https://files.catbox.moe/6mv1kp.png)

## 4.选择分支

![](https://files.catbox.moe/a2qsak.png)

![](https://files.catbox.moe/hh9163.png)

## 5.点击那个绿色的 

上图compare branches and continue

## 6.再点个绿的

 submit merge request

![](https://files.catbox.moe/8fillm.png)