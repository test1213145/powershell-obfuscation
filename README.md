# powershell-obfuscation

简单有效的powershell免杀混淆器的小工具，VT全绿，可过Defender、360等，可执行上线cobaltstrike等操作。

AMSI混淆绕过+powershell命令混淆绕过。

**请勿使用于任何非法用途，由此产生的后果自行承担。**

上述测试环境均为实体机。

## 思路

这段时间看了看powershell反混淆相关的内容与论文，目前反混淆效果最好的应该是2022年qax的《Invoke-Deobfuscation: AST-Based and Semantics-Preserving Deobfuscation for PowerShell Scripts》，该论文延续了19年CCS浙大的思路并进行了改进，使用了变量追踪并在AST层面上进行了invoke解混淆，属于静态分析加部分的动态分析，比defender和VT的效果好不少。

**不过论文中也提到了当前powershell反混淆研究的难点，一个是自定义function加密解不开，一个是很难去追踪循环中的变量。**

由于当前的powershell反混淆研究中对静态代码分析研究的并不算很多，暂未像C\Java中的静态代码分析一样可以有效追踪变量的调用与传播，因此对于循环中变量的传播暂时没有很好的解决办法。

同时function的调用可能需要更复杂的动态分析，还要考虑到function嵌套的问题，因此暂时也没有很好的解决办法。

上述两个点可能是未来powershell反混淆需要去研究的内容。

由于部分的反混淆工具会在AST层面上进行反混淆的工作，因此powershell自带的大部分加密解密/编码解码的函数是形同虚设的，如[System.Convert]::FromBase64String等。应该尽可能去使用自定义的加密解密的function。

这里针对这两个学术界研究的难点，写了一个简单的powershell混淆器，事实证明效果确实也不错。具体思路如下：

1、自定义加密解密function，function中进行字符串的逆序（逆序没有用powershell自带的函数，防止AST层面上解混淆）与位的+运算（不使用异或运算的原因是defender对-bxor监控很严格）。

2、对上述function进行几次循环的运算。

3、为了能让字符有效地输出，最后用base64编码了一下（即便在AST层面上解开也无所谓，因为解开了的内容仍是混淆之后的）

同时对AMSI绕过与powershell命令进行了混淆。

这里仅仅实现了一个简单的混淆器demo，可以自由发挥。

实验了一下，用qax的反混淆工具与Unit42团队的反混淆工具都是解不开的。

## 使用的方法

```./powershell-obfuscation.ps1 -c "whoami"``` 来混淆命令

```./powershell-obfuscation.ps1 -f "filename"``` 来混淆指定的文件

结果会输出在当前目录下的bypass.ps1中

以cs的beacon.ps1为例

混淆前VT如下：

![image](https://github.com/H4de5-7/powershell-obfuscation/blob/main/VTorigin.png)

混淆后VT如下：

![image](https://github.com/H4de5-7/powershell-obfuscation/blob/main/VTbypass.png)

上线：

![image](https://github.com/H4de5-7/powershell-obfuscation/blob/main/CS.png)
