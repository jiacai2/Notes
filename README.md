# 致被 Intel VMD(Volume Management Device) / Intel RST(Rapid Storage Technology) / Intel optane memory / Intel CAS(Cache Acceleration Software) 困扰的朋友

本文系我反复踩坑经验，成文于2025年3月16日。当前Windows11系统版本为 24H2。使用CPU为Intel 13代，主板为Z790。

## 结论

Windows下，需要raid的场合，只要成本允许，应当尽可能购买阵列卡，避免使用VMD+RST。需要对HDD磁盘加速的场景，尽可能避开VMD/RST/CAS，推荐使用 **PrimoCache**。

## 背景

我因需要本地部署DeepSeek-R1而显存内存不足，最终结合Ktransformer和unsloth方案，在22g显存上部署MLA层，在内存中部署热点专家组，在raid傲腾中部署冷门专家组。虽然跑通，但2080ti缺乏kt需要的marlin算子，最终效果很慢不理想。

扯远了回到raid话题上。目前已知，Windows系统下，使用Disk manager制作的Striped Volume带区卷（软raid）性能最差，使用Intel主板芯片组VMD控制的raid性能稍强一些，使用Raid阵列卡的性能表现最好。但Raid阵列卡整体部署成本较高，因此有时不得不使用Intel VMD。

Intel在消费级储存/内存加速上基本是一笔烂账。Intel曾经一度对Optane傲腾系列寄予厚望，但最终Optane商业失败，Intel存储业务也整个卖掉了。当然当前这个时间点（2025年），Intel麻烦远不止储存一个业务，不如说Intel现在自身都朝夕不保，处境艰难。

- **CAS** 最早起源于2009年前后，最终在2023年2月6日 EOL终止。其功能中一小部分集成进RST中。
- **Intel Optane Memory** 发售于2017年，整条产品线最终于2022年8月公布结束。
- 目前存续的消费产品线仅剩 **VMD** 一个。

## 在全新安装Windows11时启动VMD

**再次提醒你，Intel VMD 已经是一笔烂账，如果有条件，请购买阵列卡raid卡，不要开VMD！**  
RST驱动下的高性能PCIE4.0/ PCIE5.0 NVME盘即便不组raid也会损失至多10%性能。有条件时请买阵列卡raid卡，不要开VMD！

1. 首先你需要去你所用主板的官方网站下载RST驱动软件，将软件包解压，找exe（例如我的主板是20.0.0.1038版本的SetupRST.exe），将这个exe复制一份，并把exe后缀改为zip，然后解压至SetupRST文件夹，其中应当有至少一个iaStorVD.inf文件。确认有之后将SetupRST.exe和SetupRST文件夹都放在U盘Driver文件夹下。关机。
   
   - **重要！** 如果你的电脑上有任何傲腾Optane盘，特别是M10和H10/H20盘，都会导致后续驱动安装失败，一定要关机后拔掉。我在这里一步被卡了至少2星期，最终才在官网上看到 2021年1月，RST19.0版本开始，这个死妈驱动已经不再支持傲腾Optane盘。一旦检测到傲腾盘会立刻报错终止系统安装。这个设定真是太恶心了。
   - **重要2！** 开启VMD后加载RST驱动时，电脑上不能有已经建成的软硬raid盘组存在，如果你的电脑上有（比如win系统的Striped Volume），趁关机时先拔掉。否则系统安装失败退出。

2. 开机，疯狂按下del键，直到进入bios。在bios中开启VMD（也可能叫RST或optane memory support，总之都是一回事）。关闭CSM支持，只保留UEFI开启。插入U盘，在BIOS中将uefi U盘顺序调整至第一位。保存修改，重启。

3. 正常安装win11系统，到选择磁盘步骤应当看不见任何磁盘，或只有U盘分区存在。选择加载驱动按钮，选择U盘上的SetupRST文件夹。如果正常加载了驱动，你有一定几率能看见SSD和HDD盘，将你的目标系统盘格式化后继续安装系统。

   - 如果你加载驱动后仍看不见硬盘，或者直接报错退出，不要担心这很常见，请重新进入BIOS关闭VMD后，参考下一章的“在已有windows11上启动VMD”。
   - 如果你非要在新装win11系统里直接加载RST驱动，也有办法，使用uupdump.net制作windows 11 pro或以上包含updates更新的系统，用NTLite添加RST驱动，然后用rufus制作系统盘。

4. 进入系统后，进行Windows update，全部更新完之后，再点SetupRST.exe，此时应当能正常在系统内安装Intel RST。

5. 安装RST完成后，关机。重新插上你的傲腾Optane盘，和之前拔下的Raid盘。开机，用RST创建或管理raid组。

## 在已有Windows11上启动VMD

**第三次提醒你，Intel VMD 已经是一笔烂账，如果有条件，请购买阵列卡raid卡，不要开VMD！**  
如果你开启VMD后安装系统失败，或者你脑子抽了就是要开VMD还非要在当前win11系统上弄，虽然麻烦也有办法。

1. 在github上搜索Dism++，下载，将压缩包解压。
2. 去你所用主板的官方网站下载RST驱动软件，将软件包解压，找exe（例如我的主板是20.0.0.1038版本的SetupRST.exe），将这个exe复制一份，并把exe后缀改为zip，然后解压至 SetupRST 文件夹，其中应当有至少一个iaStorVD.inf文件。
3. 打开Dism++x64.exe，点Drivers，点Add，路径选择刚才的SetupRST文件夹。如果提示安装成功1个或多个就是成功。关机。
4. 开机，疯狂按Del进入BIOS，开启VMD，保存重启。
5. 罕见运气好的情况下，你能直接进入win11登录界面。大多数人会遭遇蓝屏提示无法找到硬盘。此时看到蓝屏就按机箱电源键关机，反复第三次或第四次会进入蓝色的启动设置修复节目，在高级里找到安全模式启动。在安全模式选择列表里选择带网络安全模式。
6. 大多数情况下都能看到windows登录界面，此时系统有微软账户的话可能无法登录。没有关系只要能看见登录界面就可以直接点重启，然后进入开启了VMD支持的win11系统。
7. 进入系统后，进行Windows update，全部更新完之后，再点SetupRST.exe，此时应当能正常在系统内安装Intel RST。
8. 安装RST完成后，关机。重新插上你的傲腾Optane盘，和之前拔下的Raid盘。开机，用RST创建或管理raid组。

   - 如果还是无法正常进入系统，尝试在带网络安全模式中运行Windows update和SetupRST.exe。
   - 如果带网络安全模式中运行Windows update和SetupRST.exe 后，还是无法正常进入系统。建议放弃，如果还是还是不想放弃，可以用pe盘启动，然后向系统注入Intel RST驱动。(小心第三方pe几乎百分百全员带毒, 就算不带毒也会安装一大堆广告)

## 配置Intel RST app

- **制作raid组**：在bios里或者Intel RST app里都可以配置raid组，注意不管是新建还是拆解raid。盘上数据都会全部丢失，务必小心操作，请一定要做好备份。

- **开启傲腾Optane加速**：由于傲腾支持停止，EOL。虽然这个选项在RST app里存在，但所有傲腾盘都无法触发这个选项。需要对机械盘加速，考虑把傲腾当作普通SSD，使用 **PrimoCache** 给HDD加速。

- **要不要额外装CAS**：CAS已经被放弃很久了，一直存在的Windows App报错问题最终也没人修复. 它的一部分功能可以在Intel RST app可以找到，但一如既往的弱鸡。而且安装CAS还必须有至少一块intel盘在电脑里。过去有一些人会购买大约10rmb价格的16gb傲腾M10盘，以满足CAS的安装条件来使用。但2025了，有加速需求优先考虑使用 **PrimoCache**，不想用这收费软件可以考虑 **Open Cache Acceleration Software**，这是CAS的开源版本。

最后, 朋友感谢你看到这里, 祝你不被这一塌糊涂的Intel VMD坑害. 再见.
