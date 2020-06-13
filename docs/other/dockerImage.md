# win10 修改docker镜象位置


在windows10下安装docker，随着用docker pull image文件后，C盘的容量越来越小了，需要把这些镜象修改到其它盘符，比如D盘。

## 方法解析

windows上安装的docker其实本质上还是借助与windows平台的hyper-v技术来创建一个linux虚拟机，之后执行的所有命令其实都是在这个虚拟机里执行的，所以所有pull到本地的image都会在虚拟机的Virtual hard disks目录的文件中，这个文件就是虚拟硬盘文件。如果要想改变路径只需要在hyper-v管理器里设置就可以了。  

默认的安装路径是C:\Users\Public\Documents\Hyper-V\Virtual hard disks下。

## 步骤

1. 开始菜单右键->控制面板->管理工具->Hyper-V 管理器  
在弹出的面板中左侧选择Hyper-V 管理器子选项，右侧会出现虚拟机列表，右键对应虚拟机->移动  

2. 在弹出的面板选择“选择移动选项”，右侧选第一项后，点下一步  

3. 左侧选择虚拟机，右侧选择对应的位置后，点"下一步"完成操作  

