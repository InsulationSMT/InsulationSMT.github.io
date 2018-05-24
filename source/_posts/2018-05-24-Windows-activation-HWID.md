---
title: Windows 10  数字许可证激活(HWID)方法
date: 2018-05-24 20:12:15
tags:
- Windows
---

更新：2018-05-24 21:08

### 前言

在 Windows 10 的所有系统中，无论系统是如何被激活的（通过 Windows7/8.1 升级 或者 购买零售密钥版密钥 或者 嵌入BIOS又名MSDM许可证）都会被转换成基于各自的设备硬件 ID (HWID) 的一个数字许可证。这个证书储存在微软服务器并且每次安装系统时都会激活这台设备。只有当这台设备的硬件被更改才会导致许可证失效。通过将它绑定到一个微软账户 (MSA) 你可以在这种情况下转移证书从而激活这台(硬件被更改的)设备。

这个过程在每台机器上只需要执行一次。在以后的安装中只需要跳过任何需要密钥的地方（在安装的时候选择 ‘我没有激活密钥’），之后第一次联网的时候将自动联系微软服务器注册 HWID 并激活这台设备。

注意：如果从 VLSC 或者 MVS Business ISO 中安装了批量许可证版本（VL版），需要输入默认的零售/OEM 密钥 才能重新获得激活。

这个过程实际上非常简单，并且不会篡改任何系统文件和泄露密钥。

许可证的创建流程已经对所有的 MS SKU 版本进行了适配和优化，所以以下的手动激活模式可以完全适用在这些版本上。自动激活模式是一个最简单的激活方式，并且使用在所有 MS SKU 版本上以及为下面的版本进行了专门的设计优化：

支持的 Windows 10 版本 （Skus）：

- 核心（家庭）版(N）
- 核心单语言版(N）
- 专业版(N）
- 专业教育版(N）
- 专业工作站版(N）
- 教育版(N）
- 企业版(N）
- 企业 S 版(N）

### 手动模式

1. 从 Windows 10 17134 ISO 内 获取 GatherOsState.exe 

2. 从 [GitHub](https://github.com/vyvojar/slshim/releases) 获取最新的 slshim 

3. 提取 slshim32.dll (适用于 x86 ISO) 或者 slshim64.dll (适用于 x64 ISO)

4. 将 GatherOsState.exe 和 提取的 slshim(32/64).dll 放置在相同的文件夹内

5. 将 slshim(32/64).dll 重命名为 slc.dll

6. 导入注册表

   1. 从下列列表中选择对应的 %sku% 值

      ```
      edition=Cloud
      sku=178
      	
      edition=CloudN
      sku=179
      
      edition=Core
      sku=101
      
      edition=CoreCountrySpecific
      sku=99
      
      edition=CoreN
      sku=98
      
      edition=CoreSingleLanguage
      sku=100
      
      edition=Education
      sku=121
      
      edition=EducationN
      sku=122
      
      edition=Enterprise
      sku=4
      
      edition=EnterpriseN
      sku=27
      
      edition=EnterpriseS
      sku=125
      
      edition=EnterpriseSN
      sku=126
      
      edition=Professional
      sku=48
      
      edition=ProfessionalEducation
      sku=164
      
      edition=ProfessionalEducationN
      sku=165
      	
      edition=ProfessionalN
      sku=49
      
      edition=ProfessionalWorkstation
      sku=161
      
      edition=ProfessionalWorkstationN
      sku=162
      ```

   2. 使用对应的 %sku%值 替换 XXX 部分 。 如果使用导入注册表的方法请确保字符串长度为7位，CMD 的方式直接使用上面的值即可。

      CMD：

      ```
      reg add "HKLM\SYSTEM\Tokens" /v "Channel" /t REG_SZ /d "Retail" /f
      reg add "HKLM\SYSTEM\Tokens\Kernel" /v "Kernel-ProductInfo" /t REG_DWORD /d XXX /f
      reg add "HKLM\SYSTEM\Tokens\Kernel" /v "Security-SPP-GenuineLocalStatus" /t REG_DWORD /d 1 /f
      ```

      REG：

      ```
      Windows Registry Editor Version 5.00
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens]
      "Channel"="Retail"
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens\Kernel]
      "Kernel-ProductInfo"=dword:0000XXX
      
      [HKEY_LOCAL_MACHINE\SYSTEM\Tokens\Kernel]
      "Security-SPP-GenuineLocalStatus"=dword:00000001
      ```

7. 从密钥列表中找到对应的默认 零售/OEM 密钥 并安装

   [密钥列表](https://pastebin.com/rYakstDc)

   如果你是用 企业版N 或者 LTSB 2016 N 版请在管理员模式下运行 Powershell 运行以下代码

   ```
   ::EnterpriseN
   ((Get-Content '.\gatherosstate.exe') -replace "`0" | Select-String -Pattern "(.....-){4}C372T" -AllMatches).Matches | Select-Object -ExpandProperty Value
   
   ::EnterpriseSN
   ((Get-Content '.\gatherosstate.exe') -replace "`0" | Select-String -Pattern "(.....-){4}VMJWR" -AllMatches).Matches | Select-Object -ExpandProperty Value
   ```

   这会从 GatherOsState.exe 获取密钥

8. 运行 GatherOsState.exe 。几秒钟后你会获得 GenuineTicket.xml

9. (可选)从注册表中移除 HKEY_LOCAL_MACHINE\SYSTEM\Tokens 

   CMD：

   ```
   reg delete "HKLM\SYSTEM\Tokens" /f
   ```

   REG：

   ```
   Windows Registry Editor Version 5.00
   
   [-HKEY_LOCAL_MACHINE\SYSTEM\Tokens]
   ```

10. 把创建的 GenuineTicket.xml 移动到 C盘根目录 然后 管理员 CMD 执行

    ```
    clipup -v -o -altto c:\
    ```

11. 然后使用以下命令强制激活

    ```
    cscript /nologo %windir%\system32\slmgr.vbs -ato
    ```

完成。恭喜。



### 自动模式

**请不要使用任何代理**

注意：这个工具需要一些时间（取决于你的设备性能）执行系统检查，莫方，稍等即可。谢谢。

**v10.01** 稍微改变了在 Win 7 兼容模式下运行 GatherOsState.exe 的过程，所以创建票据的过程将会把系统信息设置为 Windows 7，这将会更好地模仿从一个 Win 7 系统获得原始票据。优化了 界面

**v9.32** 添加了 nsane 和 aiowares 论坛的链接以获取信息和支持

**v9.25** 在没有用户干预的情况下将更改初始信息界面改为启动屏幕（注：我并没有使用此工具也不知道是什么意思）

**v9.18** 重做了系统检查部分

**v9.11** 添加了对 LTSB 2015 版的支持（仅支持非N版本并且目前尚未测试）添加了静默模式

……（太久远了的更新不想翻译了(╯‵□′)╯︵┻━┻）……

**v9.04** fixed spelling error in splash pic

**v9.01** fixed the KMS detection (will work on activated KMS systems now) amd added silent mode

**v8.13** added Messagebox to inform user tool-start-up might need a moment, fixed tool not closing when done via the 'X'

**v8.06** changed disabled WU handling to: set to auto, start service, activate, stop service and set back to disabled

**v7.99** added last checks and some code cleanup

**v7.77** implemented disabled WU handling.

 使用静默模式

```
hwidgen.mk3 silent
```

 下载地址（本人做了搬运并且将持续更新）

[Google Drive (包含历史版本)](https://drive.google.com/open?id=14swGfWlpXVcfE9U_mu8InPbAqtf78KzO) 

解压密码

3Fs44Rv#tZ4u3UOij656NgF____

各种哈希值

```
BLAKE2sp: d4091d1a9cce5636bc99cfbee6bc6997e613449de620d66e70b221bf81ab4973
CRC64: 4670e3e5936015fa
MD5: 9b113cdf0f1209e63bdfbdcac175ad00
SHA-1: 04b812a9671f7e4fab3b0c128bf39d08b2ada3bd
SHA-256: fa8475fe3f14760d1bedcf6a2a0720fe3b702a23f3c95b989a777e8bbda80583
SHA-384: 951c8679cbddc001eac97ad893a474624b12f887ca254f1fac571b31d85129440e6454411df56b1186abf8e8e6abbec3
SHA-512: 86218362f2231c24565a92b508f3a8007560d6065334acbb0810b60580ed8a2156d82102b5d212d8782a2d3870e2e2cf963f7f3f2cf733358a27d6581e103b2e
SHA3-224: af5d846fb1ab569108e2e3bf65d870b87ccdc475cc9deae3a76275d7
SHA3-256: 69e604c727ca502dda94c728ce69194ec77167adbd748609a8c94fc964cf1731
```

[病毒检测报告](https://www.virustotal.com/de/file/d6117778cd38325cc1572f625a276e286f91f6eff98724456060bcbacf4f8c1f/analysis/1526509270/)

[**原帖地址**](http://www.nsaneforums.com/topic/312871-windows-10-digital-license-hwid-generation-without-kms-or-predecessor-installupgrade/)

- **本文作者:** Justf
- **本文链接:** https://tech.justf.space/2018/05/24/Windows-activation-HWID/
- **版权声明:** 本博客所有文章除特别声明外, 均采用 [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/) 许可协议. 转载请注明出处!

