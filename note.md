## 第一周  编程工具演练

### 一、版本控制简介

版本控制：记录文件内容变化，以便查阅特定版本的修订情况。

diff：用来比较两个文件或目录之间的差异。

~~~txt
diff文件内容：
	（1）差异结果头部：记录比较原始文件与目标文件的文件名及最后一次变化的时间
	（2）定位符号：-表示原始文件行数，+表示目标文件行数
	（3）原始文件的行和目标文件的行
~~~

path:是diff的反向操作，根据差异文件推算出目标文件或原始文件。

CVS和SVN：集中式版本控制工具，用于本地开发，与服务器直接交互。提交需要排队，不能同时修改，缺乏代码门禁。数据安全性差，如单点故障、黑客攻击。

git:分布式版本控制工具。

集中式和分布式版本控制数据存储的区别：记录差异还是记录快照。集中式版本控制工具保存一组基本文件和差异文件，中间差异文件损坏，影响其他版本的恢复。git在每次提交更新时，对当时的全部文件制作一个快照并保存这个快照的索引。

~~~txt
集中式和分布式特点对比：
集中式：服务器压力大，所有操作与服务器交互；存在单点故障；操作受限于带宽，不适合开源项目。
分布式：数据安全，无带宽瓶颈限制；快速，全是离线操作；提交使用SHA1哈希，保证数据完整性。

SVN不适合领域：
	跨地域的协同开发；对代码高质量追求和代码门禁。
git不适合的领域：
	不适合word等二进制文档的版本控制，git无锁定/解锁模式，不能排他式修改。
~~~

### 二、git的安装和配置

通过vagrant 安装Linux 虚拟机：

~~~txt
1.安装virtualbox和vagrant
2.vagrant init centos/7     //查找box
3.vagrant up 启动，没有安装会在线安装
4.vagrant box add --name centos7 ./cento7.box 离线安装
5.vagrant box list 查看box列表
6.vagrant ssh
7.vagrant reload 让文件vagrantfile生效
scp 命令实现文件互传

vagrant 关闭命令
vagrant halt
挂起命令
vagrant suspend
唤醒命令
vagrant resume

导出box
vboxmanage list vms 查看虚拟机列表
vagrant package --base new_box --output ./centos7.box

设置登录用户，修改vagrantfile:
config.ssh.username = 'root'
vagrant 配置共享文件目录
vagrant plugin install vagrant-vguest
    config.vm.synced_folder   
       "your_folder"(必须)   //物理机目录，可以是绝对地址或相对地址，相对地址是指相对与vagrant配置文件所在目录
      ,"vm_folder(必须)"    // 挂载到虚拟机上的目录地址
      ,create(boolean)--可选     //默认为false，若配置为true，挂载到虚拟机上的目录若不存在则自动创建
      ,disabled(boolean):--可选   //默认为false，若为true,则禁用该项挂载
      ,owner(string):'www'--可选   //虚拟机系统下文件所有者(确保系统下有该用户，否则会报错)，默认为vagrant
      ,group(string):'www'--可选   //虚拟机系统下文件所有组( (确保系统下有该用户组，否则会报错)，默认为vagrant
      ,mount_options(array):["dmode=775","fmode=664"]--可选  dmode配置目录权限，fmode配置文件权限  //默认权限777
      ,type(string):--可选     //指定文件共享方式，例如：'nfs'，vagrant默认根据系统环境选择最佳的文件共享方式
6.配置硬盘大小
vagrant plugin install vagrant-disksize

Vagrantfile:
config.disksize.size='50GB'

~~~

Linux 下安装：

~~~xml
1.ubuntu系统
sudo aptitude install git
sudo aptitude git-doc git-svn git-email gitk
2.centos系统
yum install git
yum install git-svn git-email gitk
3.源码安装(太复杂直接放弃)
wget https://github.com/git/git/archive/v2.21.0.tar.gz

tar -zxvf v2.17.tar.gz
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

yum remove git

cd git-2.19.0

make prefix=/usr/local/git all 执行编译
make prefix=/usr/local/git install 安装指定路径
vim /etc/profile 修改配置文件
export PATH=$PATH:/usr/local/git/bin
source /etc/profile 配置生效


make prefix=/usr/local doc info
sudo make prefix=/usr/local install install-doc install-html install-info
可能出问题：asciidoc:command not found
tar -zxvf asciidoc
cd asciioc-8
./configure
make install 
yum install xmlto
yum install autoconf


4.命令补全
将git源码包中的命令补齐脚本复制到bash-completion 对应的目录中
cp contrib/completion/git-completion.bash /etc/bash_completion.d/
重新加载自动补齐脚本，使之在当前shell中生效：
./etc/bash_completion
为了能在终端开启时自动加载bash_completion脚本，需要在本地配置文件~/.bash_profile或全局文件/etc/bashrc文件中添加下面的内容：
if [-f /etc/bash_completion.d/git-completion.bash ];then
	/etc/bash_completion.d/git-completion.bash
fi
~~~

基本配置：

~~~txt
1.三种配置：
	系统配置：存放在%git%/etc/gitconfig 
		命令：git config --system cor.autocrlf
	用户配置：~/.gitconfig
		命令：git config --global user.name
	仓库配置：工作目录.git/config
		命令：git config --local remote.origin.url
	每一个级别的配置会覆盖上层的相同配置
2.配置个人身份
	git config --global user.name ""
	git config --global user.email XXX@qq.com
3.文本换行符设置
	在window系统签出代码时将LF转换成CRLF
		git config --global core.autocrlf true
	在linux系统中，在提交时将CRLF转换成LF
		git config --global core.autocrlf input
	只开发运行在window上的项目，可以取消此功能
    	git config --global core.autocrlf false
 4.文本编码配置
 	i18n.commitEncoding log存储的编码方式
 	i18n.logOutputEncoding 查看git log时，采用的编码方式
 	git config --global i18n.commitenconding utf-8
 	git config --global i18n.logoutputencoding utf-8
 	显示路径中的中文：git config --global core.quotepath false
 5.与服务器的认证配置
 	http/https协议认证和ssh协议认证
 	ssh认证配置：：
 	（1）生成公钥
 		ssh-keygen -t rsa -C "XXX@qq.com"
	(2)将公钥拷到：Profile Settings->SSH Keys->Add SSH key->Public key
~~~

### 三、git基本命令

1.工程区域和文件状态

~~~
工程区域：
1.版本库.git文件夹
2.工作区代码文件
3.暂存区.git/index
文件状态：
1.已提交，文件被保存到本地数据库
2.已修改，修改了文件，但没有提交保存
3.已暂存，把修改的文件放到下次提交的清单中
~~~

2.常用命令

~~~
工程准备：git init projectname/git clone

查看工作区: 
	git diff：查看工作区的修改内容 
	git status：查看文件的状态

新增/删除/移动文件到暂存区: git add/ git rm/ git mv
提交更改的文件：git commit(-am 批量提交)
推送到远端仓库: git push （origin branch_name：new branch_name）

查看当前分支日志：git log

分支管理：
	列出本地分支：git branch (列出本地分支，-r列出远端分支，-a列出所有的分支)
    切换分支：git checkout 
    删除分支：git branch -d（删除本地分支，-D强制删除分支，-d -r 删除远端分支，git push origin：branch_name）
    更新分支：git pull(origin remote_branch:local_branch,如果本地分支与远端分支相同，则直接执行git pull origin remote_branch)
    	git fetch:获取分支更新后，不会进行合并
	分支合并：git merge （将某个分支合并到当前分支上）/git rebase
	新建分支：git branch /git checkout -b
强制回退到历史节点：git reset（--hard commit_id）
回退本地所有修改未提交：git checkout .
					git checkout -filename 
					git checkout -commit_id
git reflog:查看最近的命令
~~~

git log --name-status 查看做了哪些修改操作

git commit --amend 修改最后一次提交的内容

git log  -1 查看最近一条命令

### 四、安装VSCode

~~~bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'

yum install epel-release
yum install dnf

sudo dnf check-update
sudo dnf install code
~~~

配置linux系统网卡

~~~
ip addr
ip link set wlp2s0 up（wlp2s0是我的网卡）
wpa_supplicant -B -i wlp2s0 -c <(wpa_passphrase “WIFI名称” “密码”)
dhclient wlp2s0
ping 网址（测试）

使wifi可用
sudo rfkill unblock all
sudo rfkill list
sudo modprobe -r ideapad_laptop
synclient TapButton1=1  //设置触摸板点击

启动时执行脚本
/etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local 在里面追加命令或者bash脚本





~~~







