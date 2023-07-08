---
layout: post
title: Metasploit渗透测试指南(1)-信息搜集
subtitle: Author：a-fickle
date: 2023-07-08 00:00:00 +0800
tag: [Metasploit,渗透测试]
cover-img: /assets/img/2023-07-08-Metasploit渗透测试指南(1)-信息搜集/cover.png
---

**前言：**            
本文介绍的渗透测试技术基本都是围绕Metasploit的使用。       
本文所介绍的渗透测试技术均以建设安全、健康的网络环境为目的，本着探讨网络安全技术的初衷和目的，不得用于任何恶意或非法的用途，一切渗透测试行为均需要得到测试目标的书面授权并且在合法的情况下才能进行。            
          
           
## 1. 信息搜集：          
### 1.1 被动信息搜集：        
#### 1.1.1 whois查询：               
* 作用：查找域名服务商、域名注册等信息。       
* 命令：msf > whois [域名]                  
* 注意：域名服务器可能由非测试目标厂商的其他厂商提供，如果未授权，不能对这种域名服务器进行任何测试行为。            
#### 1.1.2 Netcraft：       
* 作用：发现为该域名提供服务的服务器IP地址。           
* 网址：http://searchdns.netcraft.com/                  
* 注意：在发现特定服务器IP地址后，可以针对IP地址再一次whois查询，得到该域名所属组织的服务器的IP地址段，也可以根据地址段猜测是组织。            
#### 1.1.3 NSLookup：       
* 作用：DNS查询，获得更详细的信息。           
* 命令：           
root@bt:~# nslookup                  
set type=mx     
\> [域名]             
         
### 1.2 主动信息搜集：            
#### 1.2.1 Nmap：               
* 作用：端口扫描。       
* 命令：nmap -sS -Pn (-A) [IP adress]                  
* 高级：TCP空闲扫描（找到一台空闲主机，冒充该空闲主机的IP地址对目标主机的某个端口进行探测，空闲主机要求使用递增IP ID）            
1、Metasploit寻找满足条件的空闲主机：            
msf > use auxiliary/scanner/ip/ipidseq      
msf auxiliary(ipidseq) > set RHOSTS 192.168.1.0/24           
msf auxiliary(ipidseq) > set THREADS 50    
msf auxiliary(ipidseq) > run        
2、从结果中找到 Incremental 的主机，使用它的IP地址通过nmap进行端口扫描：         
nmap -Pn -sI [空闲主机IP] [目标IP]               
#### 1.2.2 SMB协议扫描：             
* 作用：获取Windows系统版本号。       
* 命令：             
msf > use scanner/smb/smb_version              
msf auxiliary(smb_version) > set RHOSTS 192.168.1.155                            
msf auxiliary(smb_version) > run              
#### 1.2.3 mssql扫描：             
* 作用：搜寻配置不当的Microsoft SQL Server。       
* 命令：             
msf > use scanner/mssql/mssql_ping              
msf auxiliary(mssql_ping) > set RHOSTS 192.168.1.0/24                            
msf auxiliary(mssql_ping) > set THREADS 255                    
msf auxiliary(mssql_ping) > run              
#### 1.2.4 SSH服务器扫描：             
* 作用：对运行着SSH的主机的SSH版本进行识别，找到该版本没有安装补丁的安全漏洞。       
* 命令：             
msf > use scanner/ssh/ssh_version              
msf auxiliary(ssh_version) > set THREADS 50                    
msf auxiliary(ssh_version) > run              
#### 1.2.5 FTP扫描：             
* 作用：对使用FTP协议的服务器进行扫描、识别和查点。       
* 命令：             
     
msf > use scanner/ftp/ftp_version (识别出FTP服务器)              
msf auxiliary(ftp_version) > set RHOSTS 192.168.1.0/24                            
msf auxiliary(ftp_version) > set THREADS 255                    
msf auxiliary(ftp_version) > run              
    
msf > use auxiliary/scanner/ftp/anonymous (检查FTP服务器是否允许匿名用户登录,匿名用户拥读/写权限)              
msf auxiliary(anonymous) > set RHOSTS 192.168.1.0/24                            
msf auxiliary(anonymous) > set THREADS 50                    
msf auxiliary(anonymous) > run              
#### 1.2.6 SNMP扫描：             
* 作用：对使用SNMP协议的网络设备进行扫描，对一个IP或一段IP使用字典来猜解SNMP团体字符串(相当于查询设备信息或写入设备配置参数时所需的口令)。       
* 命令：             
     
msf > use scanner/snmp/snmp_enum (扫描SNMP网络设备)              
            
msf > use scanner/snmp/snmp_login (猜解SNMP团体字符串)              
msf auxiliary(ftp_version) > set RHOSTS 192.168.1.0/24                            
msf auxiliary(ftp_version) > set THREADS 50                    
msf auxiliary(ftp_version) > run              
#### 1.2.6 编写自定义扫描器：             
* 实例代码（Ruby）：          

#Metasploit          
require 'msf/core'           
class Metasploit3 < Msf::Auxiliary          
&emsp;include Msf::Exploit::Remote::Tcp               
&emsp;include Msf::Auxiliary::Scanner         
&emsp;def initialize          
&emsp;&emsp;super(             
&emsp;&emsp;&emsp;'Name'         => 'My custom TCP scan',                
&emsp;&emsp;&emsp;'Version'      => '$Revision: 1 $',                
&emsp;&emsp;&emsp;'Description'  => 'My quick scanner',                
&emsp;&emsp;&emsp;'Author'       => 'Your name here',                
&emsp;&emsp;&emsp;'License'      => 'MSF_LICENSE'                
&emsp;&emsp;)           
&emsp;&emsp;register_options(             
&emsp;&emsp;&emsp;[            
&emsp;&emsp;&emsp;&emsp;Opt::RPORT(12345)                 
&emsp;&emsp;&emsp;],self.class)               
&emsp;end           
(保存在路径module/auxiliary/scanner/下，命名为simple_tcp.rb)           
         
* 命令：msf > use auxiliary/scanner/simple_tcp              
              
           
&nbsp;             
**参考文献：**《Metasploit-The Penetration Tester's Guide》                
               
              

















