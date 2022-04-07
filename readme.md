# CH32V103R-EVT  RT-Thread Studio开发板支持包制作

CH32V103R-EVT的BSP提交已经合并了，开始制作对应的RT-Thread Studio开发板支持包

# 1 准备

- 最新的RT-Thread Studio

- MounRiver Stuido生成的 RT-Thread Demo 工程

  ![image-20220406151853081](figures/image-20220406151853081.png)

- [官方开发板资源包制作指导](https://www.rt-thread.org/document/site/#/development-tools/rtthread-studio/um/studio-user-manual?id=%e5%bc%80%e5%8f%91%e6%9d%bf%e8%b5%84%e6%ba%90%e5%8c%85%e5%88%b6%e4%bd%9c)

# 2 导入BSP工程

## 2.1 官方 scons --dist-ide

虽然官方推荐在Env里使用下面命令直接生成BSP包，然后直接制作

```shell
LT@DESKTOP-WIN10 E:\WorkSpaces\rt-thread\bsp\wch\risc-v\ch32v103r-evt                 
> scons --dist-ide --project-path=ch32v103r-evt  --project-name=ch32v103r-evt         
scons: Reading SConscript files ...                                                   
Newlib version:3.0.0                                                                  
make distribution....                                                                 
=> ch32v103r-evt                                                                      
=> components                                                                         
=> include                                                                            
=> libcpu                                                                             
=> src                                                                                
=> tools                                                                              
Update configuration files...                                                         
done!                                                                                 
```

生成的工程，未导入library ,配置估计也是错的。所以我们选择先直接导入bsp，调整可正常编译后再制作开发板支持包

![image-20220406210316244](figures/image-20220406210316244.png)





## 2.2  直接导入BSP工程

发现import支持BSP的直接导入，但是需要先把 MounRiver Stuido生成的 RT-Thread Demo 工程的几个文件拷贝到BSP目录，不然导入不成功！

![image-20220406153840370](figures/image-20220406153840370.png)

选择导入`RT-Thread BSP到工作空间`

![image-20220406154116251](figures/image-20220406154116251.png)

目录选择最新master上的CH32V103R-EVT BSP 目录

![image-20220406154445554](figures/image-20220406154445554.png)

导入完成

![image-20220406154733310](figures/image-20220406154733310.png)

## 2.3 libraries

导入的工程明显丢失了BSP上一层的Libraries目录，首先copy过来，重命名libraries(一般小写)

![image-20220406173912903](figures/image-20220406173912903.png)



# 3 调整配置

## 3.1 配置工具链

![image-20220406155155093](figures/image-20220406155155093.png)

## 3.2 同步scons配置到项目

现在工程src, include ,group基本都是错了，包含了很多不该包含的文件，或者找不到文件，需要借助scons 快速生成正确的配置。

![image-20220406160020100](figures/image-20220406160020100.png)

> scons 果然很强大

### 3.2.1 先重配 target

现需要一个编译目标，一般使用Debug

![image-20220406172039777](figures/image-20220406172039777.png)

### 3.2.2 同步scons 

在 .setting 文件夹下新建projcfg.ini 配置

```
#RT-Thread Studio Project Configuration
#Wed Apr 06 15:50:01 CST 2022
cfg_version=v3.0
board_name=CH32V103R-EVT
example_name=
hardware_adapter=J-Link
board_base_nano_proj=False
project_type=rt-thread
chip_name=CH32V103R8T6
bsp_version=1.0.0
selected_rtt_version=4.0.4
os_branch=full
project_base_bsp=true
is_use_scons_build=True
is_base_example_project=True
output_project_path=E\:\\WorkSpaces\\RT_Thread_Studio\\CH32V103R-EVT
project_name=CH32V103R-EVT
bsp_path=
os_version=4.0.4
```

然后右击选择`同步scons配置至项目`

![image-20220406155330854](figures/image-20220406155330854.png)

打开配置发现已自动更新了include and src, 手动删除上面的错误路径，同时发现Libraries路径存在问题（我们之前改成了libraries）

![image-20220406173355311](figures/image-20220406173355311.png)

在SConstruct中修正一下，再次Scons同步

![image-20220406174931821](figures/image-20220406174931821.png)

最终得到正确的路径和符号

![image-20220406175116446](figures/image-20220406175116446.png)

## 3.3 link.lds

再次编译，发现链接文件有问题

![image-20220406175953647](figures/image-20220406175953647.png)

配置正确的.lds路径

```
"${workspace_loc:/${ProjName}/board/linker_scripts/link.lds}"
```

![image-20220406180050831](figures/image-20220406180050831.png)

再次编译，通过

![image-20220406180314782](figures/image-20220406180314782.png)

# 4 测试优化

## 4.1 新固件测试

![image-20220406181139847](figures/image-20220406181139847.png)

看下终端，编译时间正确，LED,MSH运行正常。

![image-20220406181807731](figures/image-20220406181807731.png)

## 4.2 RT-Thread Setting配置

![image-20220406184147392](figures/image-20220406184147392.png)

更新根目录下的Kconfig,再次配置正常

![image-20220406184253458](figures/image-20220406184253458.png)

把暂时未使用的软件定时器关掉，测试

![image-20220406184607168](figures/image-20220406184607168.png)

结果正确，这个就是需要的软件包工程了

![image-20220406184757226](figures/image-20220406184757226.png)

# 5 制作本地开发板支持包

## 5.1 源码精简

拷贝一份源码到另一个目录clone,删除git和package目录

![image-20220406193458972](figures/image-20220406193458972.png)

## 5.2 新建开发板支持包

### 5.2.1 开发板支持包信息

![image-20220406194304250](figures/image-20220406194304250.png)

### 5.2.2 开发板信息

![image-20220406194433061](figures/image-20220406194433061.png)

### 5.2.3 开发板特性

![image-20220406194903251](figures/image-20220406194903251.png)

### 5.2.4 文档信息

![image-20220406210952155](figures/image-20220406210952155.png)

### 5.2.5 工程信息

#### 5.2.5.1 模板工程

这个模板最好选择固定的rtthread 版本, 同时不要移除RTT目录，这样可以最大程度保留项目配置信息，让生成的工程不需要调整即可正常编译。

![image-20220407094417197](figures/image-20220407094417197.png)



#### 5.2.5.2 例子工程

这个和上面的没啥区别，就是基于最新的rt-thread版本，测试一下。 如果像ART-Pi一样有很多不同的DEMO，可以在这里添加。

![image-20220407094844764](figures/image-20220407094844764.png)

>  **Note**: latest 默认去除工程的rtthread源码

#### 5.2.5.3 已知的BUG

这里不得不吐槽一下这个BSP package 制作工具，也许个人用的不多，bug真多，我试了至少20次才达到我的预期，分支部分老是抽风。

模板工程只有一个会存在莫名其妙一大堆问题。两个的话，默认只使用第二个，生成的工程不会有问题。

![image-20220407105816210](figures/image-20220407105816210.png)



## 5.3 生成

先预览一下

![image-20220406200245832](figures/image-20220406200245832.png)

没有问题，查看生成的bsp 软件包，同时压缩成zip

![image-20220406210043575](figures/image-20220406210043575.png)

# 6 测试本地资源包

## 6.1 导入本地资源包

把刚刚生成的zip压缩包打入 SDK管理器

![image-20220406211703684](figures/image-20220406211703684.png)

导入成功

![image-20220406211854036](figures/image-20220406211854036.png)

## 6.2 工程测试

### 6.2.1 模板工程测试

新建模板工程，实际用到的是提交里的RTT源码，latest不保证实时最新，提交时使用的是2022.4.6后的版本。

![image-20220407103854947](figures/image-20220407103854947.png)

> rt-thread 版本未变灰色，不从软件包更新，使用提交的原始的rtthread源码

编译，没有问题

![image-20220407095755162](figures/image-20220407095755162.png)

### 6.2.2 例子工程测试

![image-20220407103913189](figures/image-20220407103913189.png)

> rt-thread 版本变灰色，从软件包根据版本更新rtthread源码

latest package更新的不及时，我新提交合并的ch32v103r-evt还没有更新进来。需要手动copy一份最新rtthread/libcpu/risc-v相关文件，scons 更新一下再编译就OK了

![image-20220407104922359](figures/image-20220407104922359.png)

> 我提交的BSP在2022.4.6合入master分支，之后的rtthread版本不存在这个问题

## 6.3 其他测试结果

这部分模板和例子工程一样

- [x] 开发板实测: LED,MSH ok
- [x] RT-Thread Setting
- [ ] Board Information can't view
- [x] Package update
- [x] Scons update 
- [x] Clean project

# 7 上传开发板支持包

## 7.1 推送本地仓库到github

在5.3 生成的sdk-bsp-ch32v103r-evt推送到自己的github上，远程仓库名字`sdk-bsp-ch32v103r-evt`

```
LT@DESKTOP-Win10 MINGW64 /e/WorkSpaces/RT_Thread_Studio/sdk-bsp-ch32v103r-evt (master)
$ git push origin master -f
Enumerating objects: 1921, done.
Counting objects: 100% (1921/1921), done.
Delta compression using up to 8 threads
Compressing objects: 100% (1867/1867), done.
Writing objects: 100% (1921/1921), 14.45 MiB | 1.53 MiB/s, done.
Total 1921 (delta 458), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (458/458), done.
To github.com:blta/sdk-bsp-ch32v103r-evt.git
 + e718a5d...38e96d7 master -> master (forced update)

```

## 7.2 新建 v1.0.0 release

![image-20220407113006280](figures/image-20220407113006280.png)

## 7.3 更新SDK index

1. Fork 一份RT-Thread Studio 的 SDK 索引仓库 https://github.com/RT-Thread-Studio/sdk-index 到个人 github 账号下,同时clone到本地

2. Copy本地仓库Board_Support_Packages/WCH/sdk-bsp-ch32v307v-r0另存为`sdk-bsp-ch32v103r-evt`,放到同级目录下

![image-20220407115050026](figures/image-20220407115050026.png)

3. 更新WCH/index.json,新增`sdk-bsp-ch32v103r-evt`

![image-20220407125402331](figures/image-20220407125402331.png)

4. 更新sdk-bsp-ch32v103r-evt/index.json


![image-20220407114616293](figures/image-20220407114616293.png)

5. 提交并推送到github

```
   LT@DESKTOP-Win10 MINGW64 /e/Gitea/sdk-index (feature/sdk-bsp-ch32v103r-evt)
   $ git push origin feature/sdk-bsp-ch32v103r-evt
   Enumerating objects: 11, done.
   Counting objects: 100% (11/11), done.
   Delta compression using up to 8 threads
   Compressing objects: 100% (6/6), done.
   Writing objects: 100% (7/7), 805 bytes | 805.00 KiB/s, done.
   Total 7 (delta 3), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
   remote:
   remote: Create a pull request for 'feature/sdk-bsp-ch32v103r-evt' on GitHub by visiting:
   remote:      https://github.com/blta/sdk-index/pull/new/feature/sdk-bsp-ch32v103r-evt
   remote:
   To github.com:blta/sdk-index.git
    * [new branch]      feature/sdk-bsp-ch32v103r-evt -> feature/sdk-bsp-ch32v103r-evt
```

6. pull request

![image-20220407120232643](figures/image-20220407120232643.png)





 **大功告成！**