---
title           : "ZeroMemory与类对象"
date            : 2012-09-20
tags            : ["技术研究"]
category        : "研发"
isCJKLanguage   : true
---

今天看到一段同事的代码：  

{{< highlight cpp >}}
ZeroMemory(&m_PacketInfo, sizeof(packet_info));

struct packet_info
{
    string m_strModule;                 //模块
    string m_strProtocol;               //协议
    string m_strClientHostIP;           //客户端IP
    string m_strClientHostPort;         //客户端端口
    ...
};
{{< /highlight >}}

按照我的经验，这种对类对象进行ZeroMemory或者memset的代码会导致程序崩掉。因为会覆盖掉类对象中的关键域，比如虚表。但是同事在vs2005中这样做却很正常。让我很奇怪，于是花了点时间分析了下。

首先，std::string的vc实现并没有虚表，如下：
```
0:000> dt string
QueueTest!string
   +0x000 _Myproxy         : Ptr32 std::_Container_proxy
   +0x004 _Bx              : std::_String_val::_Bxty
   +0x014 _Mysize          : Uint4B
   +0x018 _Myres           : Uint4B
   +0x01c _Alval           : std::allocator
   =00000000`01202180
   std::basic_string::npos : Uint4B
```

所以ZeroMemory后不会导致非法访问。

翻了一下std::string的实现，发现就是一普通的模板类，没有用到继承等OO特性。跟一普通的结构体和一系列配套函数的简单组合没太大区别。

于是写了一小段测试代码

{{< highlight cpp >}}
class Father
{
public:
    Father() {}
    ~Father(){}

    virtual void Show()
    {
        printf("Father\n");
    }
};

class Son: public Father
{
public:
    void Show()
    {
        printf("Son\n");
    }
};

void main()
{
    Son son;

    son.Show();
    ZeroMemory(&son, sizeof(Son));
    son.Show();

    ...
}
{{< /highlight >}}


我对这段代码的预期是，因为有继承以及虚函数，所以会有虚表。ZeroMemory会导致虚表被破坏，然后在第二次Show的时候非法访问。但实际运行效果依然正常。  

于是检查了一下man的反汇编代码：
```
0:000:x86> uf main
QueueTest!main:
002fd120 push    ebp
002fd121 mov     ebp,esp
...
002fd15d lea     ecx,[ebp-14h]
002fd160 call    QueueTest!ILT+3860(??0SonQAEXZ) (002faf19)
002fd165 mov     dword ptr [ebp-4],0
002fd16c lea     ecx,[ebp-14h]
002fd16f call    QueueTest!ILT+4840(?ShowSonUAEXXZ) (002fb2ed)
002fd174 push    4
002fd176 push    0
002fd178 lea     eax,[ebp-14h]
002fd17b push    eax
002fd17c call    QueueTest!ILT+1520(_memset) (002fa5f5)
002fd181 add     esp,0Ch
002fd184 lea     ecx,[ebp-14h]
002fd187 call    QueueTest!ILT+4840(?ShowSonUAEXXZ) (002fb2ed)
002fd18c mov     dword ptr [ebp-4],0FFFFFFFFh
002fd193 lea     ecx,[ebp-14h]
002fd196 call    QueueTest!ILT+4125(??1SonQAEXZ) (002fb022)
...
002fd1c9 mov     esp,ebp
002fd1cb pop     ebp
002fd1cc ret
```


可以看出来，这段代码里根本就没有用到虚表，而是直接把内嵌函数地址。仔细琢磨琢磨，这样做确实有些道理：

1. son是明确的Son类对象，son.Show是一个明确的函数，无需使用虚表进行间接调用。
2. 效率更高。

再想想什么时候才需要虚表呢？动态绑定。  
于是改了改测试代码，如下（Father和Son的定义不变）：

{{< highlight cpp >}}
void main()
{
    Son son;
    Father *pson = (Father*)&son;

    pson->Show();
    ZeroMemory(&son, sizeof(Son));
    pson->Show();
    ...
}
{{< /highlight >}}


　　看看反汇编码：
```
0:000:x86> uf main
QueueTest!main:
011ad120 push    ebp
011ad121 mov     ebp,esp
...
011ad15d lea     ecx,[ebp-14h]
011ad160 call    QueueTest!ILT+3860(??0SonQAEXZ) (011aaf19)
011ad165 mov     dword ptr [ebp-4],0
011ad16c lea     eax,[ebp-14h]
011ad16f mov     dword ptr [ebp-20h],eax
011ad172 mov     eax,dword ptr [ebp-20h]
011ad175 mov     edx,dword ptr [eax]
011ad177 mov     esi,esp
011ad179 mov     ecx,dword ptr [ebp-20h]
011ad17c mov     eax,dword ptr [edx]
011ad17e call    eax <--------
011ad180 cmp     esi,esp
011ad182 call    QueueTest!ILT+3870(__RTC_CheckEsp) (011aaf23)
011ad187 push    4
011ad189 push    0
011ad18b lea     eax,[ebp-14h]
011ad18e push    eax
011ad18f call    QueueTest!ILT+1520(_memset) (011aa5f5)
011ad194 add     esp,0Ch
011ad197 mov     eax,dword ptr [ebp-20h]
011ad19a mov     edx,dword ptr [eax]
011ad19c mov     esi,esp
011ad19e mov     ecx,dword ptr [ebp-20h]
011ad1a1 mov     eax,dword ptr [edx]
011ad1a3 call    eax <--------
011ad1a5 cmp     esi,esp
011ad1a7 call    QueueTest!ILT+3870(__RTC_CheckEsp) (011aaf23)
011ad1ac mov     dword ptr [ebp-4],0FFFFFFFFh
011ad1b3 lea     ecx,[ebp-14h]
...
011ad1e9 mov     esp,ebp
011ad1eb pop     ebp
011ad1ec ret
```

可以看到这次用了虚表。再运行，果然崩了。

但是gcc的实现不同，于是最开始的那行ZeroMemory一跑就崩。

