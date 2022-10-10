# 电信宽带送的光猫进阶玩法，修改配置表，开启相关功能。
## 新光猫已经不适合本文操作。

自从上回搞定了光猫管理员权限，获得整个设备的控制权，网络世界仿佛豁然开朗，
深入了解一番，发现中兴光猫的系统还是非常有趣的，它是一个嵌入式Linux系统，文件系统/结构根据光猫运行环境定制而来，
既然这样，我们就可以用强大的命令行来控制光猫来做我们需要的事情。

上回已经讲到了如何获取root控制权，此文不再赘述。在命令开始前我们需要知道中兴光猫控制表的命令语法：
```shell
sendcmd 1 DB all  //列出所有表名
sendcmd 1 DB p 表名  //显示某个表的详细信息
sendcmd 1 DB set 表名 表号 项目名字 目标值 //更改某个表里面那一项的值，下面会举例说具体用法
sendcmd 1 DB save  //保存更改，实际上保存之还得重启光猫才能生效
reboot //重启光猫命令
```

其他命令跟Linux一样或者类似，需要的话自行搜索相关用法。

下面讲我说设置的实例：
首先putty用telnet登录

![1](https://user-images.githubusercontent.com/24214894/194789908-2757f41a-39c6-4bdf-a43c-727b274e5f37.jpg)

先看看这玩意有几个表，输入命令回车

sendcmd 1 DB all

![2](https://user-images.githubusercontent.com/24214894/194789911-75f0c5d6-1c45-41e7-84e8-f5ad975adeca.jpg)

在putty里面选中按CTRL+shift+C复制出来到记事本中

可以看到有两百多个表，它们分别控制特定功能的参数，接下来我要查看宽带密码，输入sendcmd 1 DB p WANCPPP

这时我们要的宽带账号和密码都出来了
```xml
<Tbl name="WANCPPP" RowCount=”1″>
  <Row No=”0″>
    <DM name=”ViewName” val=”IGD.WD1.WCD2.WCPPP1″/>
    <DM name=”UserName” val=”0777*******”/>
    <DM name=”Password” val=”89******”/>    //就是这里
    <DM name=”ConnTrigger” val=”0″/>
    <DM name=”AuthType” val=”0″/>
    <DM name=”IdleTime” val=”1200″/>
    <DM name=”AutoDisconnTime” val=”0″/>
    <DM name=”WarnDisconnTime” val=”0″/>
    <DM name=”MaxMRU” val=”1492″/>
    <DM name=”MTU” val=”1492″/>
    <DM name=”EchoTime” val=”30″/>
    <DM name=”EchoRetry” val=”20″/> 
    <DM name=”PPPoEACName” val=””/>
    <DM name=”PPPoEServiceName” val=””/>
    <DM name=”EnableProxy” val=”0″/>
    <DM name=”MaxUser” val=”4″/>
    <DM name=”EnablePassThrough” val=”0″/>
    <DM name=”PassThroughViewName” val=””/>
    <DM name=”ValidWANRx” val=”0″/>
    <DM name=”ValidLANTx” val=”1″/>
    <DM name=”HostTrigger” val=”1″/>
    <DM name=”TtyDialNum” val=””/>
    <DM name=”TtyAPN” val=””/>
    <DM name=”TtyPDPType” val=”0″/>
    <DM name=”PPPEncapsType” val=”0″/>
    <DM name=”EncapsID” val=””/>
    <DM name=”GUATrigger” val=”0″/>
    <DM name=”DNSv6Trigger” val=”0″/>
    <DM name=”PrefixTrigger” val=”0″/>
    <DM name=”AFTRTrigger” val=”0″/>
  </Row>
</Tbl>
```


这个有什么用呢？如果光猫后面接了个路由器，全部设备都连路由器上网，那就相当于数据包经过了两次NAT转发才能出网，效率略有降低。现在我们知道了账号和密码就能够将光猫设计为桥接模式，用后面的路由器来拨号上网，这样数据包仅用一台设备就能够出网，效率高了许多。关于如何设置桥接模式用路由器拨号，请听下回分解～

进入光猫管理界面会发现它有个远程下载功能，下载文件保存在外置USB存储设备在光猫上的挂载点是`/mnt/USB设备名字/`

## 开启FTP服务器
那我们就可以打开光猫的FTP服务器功能，在它下载完文件后我们能够直接从浏览器打开FTP服务界面来下载存好的文件，当然也可以在资源管理器里面输入FTP://IP地址  来像本地磁盘一样管理文件。下面继续命令和表的研究：

`sendcmd 1 DB p FTPServerCfg`

得到
```xml
<Tbl name=”FTPServerCfg” RowCount=”1″>
  <Row No=”0″>
    <DM name=”FtpEnable” val=”0″/>  //FTP服务器关闭
    <DM name=”ServerPort” val=”21″/> //默认端口为21
    <DM name=”WanIfEnable” val=”0″/> //控制从外网访问的参数
    <DM name=”FtpAnon” val=”0″/>
    <DM name=”WanID0″ val=””/>
    <DM name=”WanID1″ val=””/>
    <DM name=”WanID2″ val=””/>
    <DM name=”WanID3″ val=””/>
    <DM name=”WanID4″ val=””/>
    <DM name=”WanID5″ val=””/>
    <DM name=”WanID6″ val=””/>
    <DM name=”WanID7″ val=””/>
    <DM name=”MaxClient” val=”5″/>
    <DM name=”MaxPerIp” val=”5″/>
    <DM name=”MaxRate” val=”250000″/>
  </Row>
</Tbl>
```

想要打开FTP，我们输入
`sendcmd 1 DB set FTPServerCfg 1 FtpEnable 1`

这时候表的项目值已经被修改了
随手保存
`sendcmd 1 DB save`

现在已经打开了FTP服务，但是为了安全起见强烈建议不要打开从WAN口访问，以防被攻击，为了安全我们再配置下用户帐户：

`sendcmd 1 DB p FTPUser`

得到
```xml
<Tbl name=”FTPUser” RowCount=”8″>
  <Row No=”0″>
    <DM name=”ViewName” val=”IGD.FTPUSER0″/>
    <DM name=”Username” val=”admin”/>    //用户名
    <DM name=”Password” val=”admin”/>    //用户密码
    <DM name=”Location” val=”/”/>        //用户目录
    <DM name=”UserRight” val=”3″/>       //用户权限
    </Row>
    <Row No=”1″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”2″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”3″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”4″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”5″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”6″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
    </Row>
    <Row No=”7″>
    <DM name=”ViewName” val=””/>
    <DM name=”Username” val=””/>
    <DM name=”Password” val=””/>
    <DM name=”Location” val=””/>
    <DM name=”UserRight” val=”0″/>
  </Row>
</Tbl>
```

### 修改用户名
`sendcmd 1 DB set FTPUser 1 Username 要设置的用户名`
`sendcmd 1 DB set FTPUser 1 Password 要设置的密码`

到这里用户帐户已经配置好了，迫不及待打开浏览器跃跃欲试？然而 被 拒绝！

需要稍后保存表值和重启光猫才行！

还有一个比较重要的表：TelnetCfg

`sendcmd 1 DB p TelnetCfg`

打开它

得到
```xml
<Tbl name=”TelnetCfg” RowCount=”1″>
    <Row No=”0″>
  	<DM name=”TS_Enable” val=”1″/>   //服务打开
  	<DM name=”Wan_Enable” val=”0″/>  //WAN连接权限
  	<DM name=”Lan_Enable” val=”0″/>
  	<DM name=”TS_Port” val=”23″/>     //端口
  	<DM name=”TS_UName” val=”root”/>  //用户名
  	<DM name=”TS_UPwd” val=”Zte521″/> //密码
  	<DM name=”Max_Con_Num” val=”5″/>  //最大用户数
  	<DM name=”ProcType” val=”0″/>
  	<DM name=”Lan_EnableAfterOlt” val=”1″/> //LAN连接权限
    <DM name=”WanWebLinkToTS” val=”1″/>
  </Row>
</Tbl>
```
咱们先把密码改了吧

`sendcmd 1 DB set TelnetCfg 1 TS_UPwd 要修改的密码`

后面根据自行需要来修改，同样建议不打开WAN口连接，非常危险

### 修改WiFi前缀
电信给的光猫的WiFi 名字前面总有Chinanet这个恶心的东西，搞得不是自己家的WiFi似的，接下来我们就要修改掉这个前缀，让我们的WiFi名字由我们自己设定

我们找到表WLANCfg
可以看到
```xml
<Tbl name="WLANCfg" RowCount="4">
  <Row No="0">
 <DM name="ViewName" val="IGD.LD1.WLAN1"/>
 <DM name="LANDViewName" val="IGD.LD1"/>
 <DM name="ValidIf" val="1"/>
 <DM name="InstExist" val="1"/>
 <DM name="Enable" val="1"/>
 <DM name="ESSID" val="LXG"/>
 <DM name="ESSIDPrefix" val="ChinaNet"/>
 <DM name="Priority" val="0"/>
 <DM name="ACLPolicy" val="0"/>
 <DM name="BeaconType" val="6"/>
 <DM name="ESSIDHideEnable" val="0"/>
 <DM name="BeaconEnabled" val="1"/>
 <DM name="WEPAuthMode" val="2"/>
 <DM name="WEPLevel" val="1"/>
 <DM name="WEPKeyIndex" val="1"/>
 <DM name="WPAEncryptType" val="1"/>
 <DM name="WPAAuthMode" val="0"/>
 <DM name="11iEncryptType" val="1"/>
 <DM name="11iAuthMode" val="0"/>
 <DM name="WPAGroupRekey" val="0"/>
 <DM name="WPAEAPServerIp" val="192.168.1.1"/>
 <DM name="WPAEAPSecret" val="12345678"/>
 <DM name="MaxUserNum" val="32"/>
 <DM name="VapIsolationEnable" val="0"/>
 <DM name="BasicEncryptionModes" val="0"/>
 <DM name="LocalSetEnable" val="1"/>
 </Row>
 <Row No="1">
 <DM name="ViewName" val="IGD.LD1.WLAN2"/>
 <DM name="LANDViewName" val="IGD.LD1"/>
 <DM name="ValidIf" val="1"/>
 <DM name="InstExist" val="0"/>
 <DM name="Enable" val="0"/>
 <DM name="ESSID" val="LXG_2.4G"/>
 <DM name="ESSIDPrefix" val="ChinaNet"/>
 <DM name="Priority" val="0"/>
 <DM name="ACLPolicy" val="0"/>
 <DM name="BeaconType" val="2"/>
 <DM name="ESSIDHideEnable" val="0"/>
 <DM name="BeaconEnabled" val="1"/>
 <DM name="WEPAuthMode" val="2"/>
 <DM name="WEPLevel" val="1"/>
 <DM name="WEPKeyIndex" val="1"/>
 <DM name="WPAEncryptType" val="1"/>
 <DM name="WPAAuthMode" val="0"/>
 <DM name="11iEncryptType" val="1"/>
 <DM name="11iAuthMode" val="0"/>
 <DM name="WPAGroupRekey" val="0"/>
 <DM name="WPAEAPServerIp" val="192.168.1.1"/>
 <DM name="WPAEAPSecret" val="12345678"/>
 <DM name="MaxUserNum" val="32"/>
 <DM name="VapIsolationEnable" val="0"/>
 <DM name="BasicEncryptionModes" val="0"/>
 <DM name="LocalSetEnable" val="0"/>
 </Row>
 <Row No="2">
 <DM name="ViewName" val="IGD.LD1.WLAN3"/>
 <DM name="LANDViewName" val="IGD.LD1"/>
 <DM name="ValidIf" val="1"/>
 <DM name="InstExist" val="0"/>
 <DM name="Enable" val="0"/>
 <DM name="ESSID" val="SSID3"/>
 <DM name="ESSIDPrefix" val=""/>
 <DM name="Priority" val="0"/>
 <DM name="ACLPolicy" val="0"/>
 <DM name="BeaconType" val="2"/>
 <DM name="ESSIDHideEnable" val="0"/>
 <DM name="BeaconEnabled" val="1"/>
 <DM name="WEPAuthMode" val="2"/>
 <DM name="WEPLevel" val="1"/>
 <DM name="WEPKeyIndex" val="1"/>
 <DM name="WPAEncryptType" val="1"/>
 <DM name="WPAAuthMode" val="0"/>
 <DM name="11iEncryptType" val="1"/>
 <DM name="11iAuthMode" val="0"/>
 <DM name="WPAGroupRekey" val="0"/>
 <DM name="WPAEAPServerIp" val="192.168.1.1"/>
 <DM name="WPAEAPSecret" val="12345678"/>
 <DM name="MaxUserNum" val="32"/>
 <DM name="VapIsolationEnable" val="0"/>
 <DM name="BasicEncryptionModes" val="0"/>
 <DM name="LocalSetEnable" val="0"/>
 </Row>
 <Row No="3">
 <DM name="ViewName" val="IGD.LD1.WLAN4"/>
 <DM name="LANDViewName" val="IGD.LD1"/>
 <DM name="ValidIf" val="1"/>
 <DM name="InstExist" val="0"/>
 <DM name="Enable" val="0"/>
 <DM name="ESSID" val="SSID4"/>
 <DM name="ESSIDPrefix" val=""/>
 <DM name="Priority" val="0"/>
 <DM name="ACLPolicy" val="0"/>
 <DM name="BeaconType" val="2"/>
 <DM name="ESSIDHideEnable" val="0"/>
 <DM name="BeaconEnabled" val="1"/>
 <DM name="WEPAuthMode" val="2"/>
 <DM name="WEPLevel" val="1"/>
 <DM name="WEPKeyIndex" val="1"/>
 <DM name="WPAEncryptType" val="1"/>
 <DM name="WPAAuthMode" val="0"/>
 <DM name="11iEncryptType" val="1"/>
 <DM name="11iAuthMode" val="0"/>
 <DM name="WPAGroupRekey" val="0"/>
 <DM name="WPAEAPServerIp" val="192.168.1.1"/>
 <DM name="WPAEAPSecret" val="12345678"/>
 <DM name="MaxUserNum" val="32"/>
 <DM name="VapIsolationEnable" val="0"/>
 <DM name="BasicEncryptionModes" val="0"/>
 <DM name="LocalSetEnable" val="0"/>
 </Row>
</Tbl>
```

在表中可以看到名字ESSIDPrefix，后面的值就是ChinaNet，我们把它删掉
`sendcmd 1 DB set WLANCfg 1 ESSIDPrefix  `

注意后面跟的是两个空格
save && reboot我们就可以享用劳动成果了，还有其他两百多个表自行研究～

### 另外，新的千兆光猫已经不适用这种表方式存储设置，并精简了许多Linux的命令，等待新的研究
### [本人博客](http://www.gzjnas.xyz)
