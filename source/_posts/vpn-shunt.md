title: 通过路由表进行VPN国内外分流的办法
date: 2015/12/2 20:49:00
tag:
	- VPN
	- Mac

---

## Chnroutes 项目

> 教程的目的是要达到通过执行路由脚本,可以让用户在使用vpn作为默认网络网关的时候, 不使用vpn进行对中国国内ip的访问, 从而减轻vpn的负担, 和增加访问国内网站的速度。

需要利用一个开源项目 https://github.com/fivesheep/chnroutes 这个项目提供了可以直接使用的路由表和简易教程。

<!--more-->

## Mac用户的安装

- 下载 `chnroutes.py` 
- 从终端进入下载目录, 执 `python chnroutes.py -p mac` , 执行完毕之后同一目录下将生成两个文件 `ip-up` 和 `ip-down`
- 把这两个文件copy到 `/etc/ppp` 目录, 并使用 `sudo chmod a+x ip-up ip-down` 命令把它们设置为可执行
- 设置完毕, 重新连接vpn

## Windows用户的安装

- 下载 `chnroutes.py`
- 从终端进入下载目录, 执行 python chnroutes.py -p win, 执行之后会生成 `vpnup.bat` 和 `vpndown.bat` 两个文件.

> 由于Windows上的pptp不支持拨号脚本, 所以也只能在进行拨号之前手动执行 `vpnup.bat` 文件以设置路由表. 而在断开vpn之后, 如果你觉得有必要, 可以运行 `vpndown.bat` 把这些路由信息给清理掉。

## 测试

> 连接VPN后打开http://www.ip138.com/ 和 https://www.whatismyip.com/

前者会显示你的国内上网ip，后者会显示你国外VPN的ip


## 缺点

- 路由表每隔两三个月需要更新一次

**另外，对于Windows的用户而言：**

- 重启后添加的所有规则会失效
可以替换bat中「route add」为「route -p add」设置永久路由。
使用一个叫SetRoute的软件，可以备份系统当前路由表方便下次使用。

- 批处理运行速度过慢。
因为需要一次性添加几千行，会比较慢，办法可以参考上面的两个办法，慢就慢一次。

## chnroutes.py

文件源码如下：

```python
#!/usr/bin/env python

import re
import urllib2
import sys
import argparse
import math
import textwrap


def generate_ovpn(metric):
    results = fetch_ip_data()  
    rfile=open('routes.txt','w')
    for ip,mask,_ in results:
        route_item="route %s %s net_gateway %d\n"%(ip,mask,metric)
        rfile.write(route_item)
    rfile.close()
    print "Usage: Append the content of the newly created routes.txt to your openvpn config file," \
          " and also add 'max-routes %d', which takes a line, to the head of the file." % (len(results)+20)


def generate_linux(metric):
    results = fetch_ip_data()
    upscript_header=textwrap.dedent("""\
    #!/bin/bash
    export PATH="/bin:/sbin:/usr/sbin:/usr/bin"
    
    OLDGW=`ip route show | grep '^default' | sed -e 's/default via \\([^ ]*\\).*/\\1/'`
    
    if [ $OLDGW == '' ]; then
        exit 0
    fi
    
    if [ ! -e /tmp/vpn_oldgw ]; then
        echo $OLDGW > /tmp/vpn_oldgw
    fi
    
    """)
    
    downscript_header=textwrap.dedent("""\
    #!/bin/bash
    export PATH="/bin:/sbin:/usr/sbin:/usr/bin"
    
    OLDGW=`cat /tmp/vpn_oldgw`
    
    """)
    
    upfile=open('ip-pre-up','w')
    downfile=open('ip-down','w')
    
    upfile.write(upscript_header)
    upfile.write('\n')
    downfile.write(downscript_header)
    downfile.write('\n')
    
    for ip,mask,_ in results:
        upfile.write('route add -net %s netmask %s gw $OLDGW\n'%(ip,mask))
        downfile.write('route del -net %s netmask %s\n'%(ip,mask))

    downfile.write('rm /tmp/vpn_oldgw\n')


    print "For pptp only, please copy the file ip-pre-up to the folder/etc/ppp," \
          "and copy the file ip-down to the folder /etc/ppp/ip-down.d."

def generate_mac(metric):
    results=fetch_ip_data()
    
    upscript_header=textwrap.dedent("""\
    #!/bin/sh
    export PATH="/bin:/sbin:/usr/sbin:/usr/bin"
    
    OLDGW=`netstat -nr | grep '^default' | grep -v 'ppp' | sed 's/default *\\([0-9\.]*\\) .*/\\1/' | awk '{if($1){print $1}}'`
    if [ ! -e /tmp/pptp_oldgw ]; then
        echo "${OLDGW}" > /tmp/pptp_oldgw
    fi
    
    dscacheutil -flushcache
    route add 10.0.0.0/8 "${OLDGW}"
    route add 172.16.0.0/12 "${OLDGW}"
    route add 192.168.0.0/16 "${OLDGW}"
    """)
    
    downscript_header=textwrap.dedent("""\
    #!/bin/sh
    export PATH="/bin:/sbin:/usr/sbin:/usr/bin"
    
    if [ ! -e /tmp/pptp_oldgw ]; then
            exit 0
    fi
    
    ODLGW=`cat /tmp/pptp_oldgw`
    route delete 10.0.0.0/8 "${OLDGW}"
    route delete 172.16.0.0/12 "${OLDGW}"
    route delete 192.168.0.0/16 "${OLDGW}"
    """)
    
    upfile=open('ip-up','w')
    downfile=open('ip-down','w')
    
    upfile.write(upscript_header)
    upfile.write('\n')
    downfile.write(downscript_header)
    downfile.write('\n')
    
    for ip,_,mask in results:
        upfile.write('route add %s/%s "${OLDGW}"\n'%(ip,mask))
        downfile.write('route delete %s/%s ${OLDGW}\n'%(ip,mask))
    
    downfile.write('\n\nrm /tmp/pptp_oldgw\n')
    upfile.close()
    downfile.close()
    
    print "For pptp on mac only, please copy ip-up and ip-down to the /etc/ppp folder," \
          "don't forget to make them executable with the chmod command."

def generate_win(metric):
    results = fetch_ip_data()  

    upscript_header=textwrap.dedent("""@echo off
    for /F "tokens=3" %%* in ('route print ^| findstr "\\<0.0.0.0\\>"') do set "gw=%%*"
    
    """)
    
    upfile=open('vpnup.bat','w')
    downfile=open('vpndown.bat','w')
    
    upfile.write(upscript_header)
    upfile.write('\n')
    upfile.write('ipconfig /flushdns\n\n')
    
    downfile.write("@echo off")
    downfile.write('\n')
    
    for ip,mask,_ in results:
        upfile.write('route add %s mask %s %s metric %d\n'%(ip,mask,"%gw%",metric))
        downfile.write('route delete %s\n'%(ip))
    
    upfile.close()
    downfile.close()
    
#    up_vbs_wrapper=open('vpnup.vbs','w')
#    up_vbs_wrapper.write('Set objShell = CreateObject("Wscript.shell")\ncall objShell.Run("vpnup.bat",0,FALSE)')
#    up_vbs_wrapper.close()
#    down_vbs_wrapper=open('vpndown.vbs','w')
#    down_vbs_wrapper.write('Set objShell = CreateObject("Wscript.shell")\ncall objShell.Run("vpndown.bat",0,FALSE)')
#    down_vbs_wrapper.close()
    
    print "For pptp on windows only, run vpnup.bat before dialing to vpn," \
          "and run vpndown.bat after disconnected from the vpn."

def generate_android(metric):
    results = fetch_ip_data()
    
    upscript_header=textwrap.dedent("""\
    #!/bin/sh
    alias nestat='/system/xbin/busybox netstat'
    alias grep='/system/xbin/busybox grep'
    alias awk='/system/xbin/busybox awk'
    alias route='/system/xbin/busybox route'
    
    OLDGW=`netstat -rn | grep ^0\.0\.0\.0 | awk '{print $2}'`
    
    """)
    
    downscript_header=textwrap.dedent("""\
    #!/bin/sh
    alias route='/system/xbin/busybox route'
    
    """)
    
    upfile=open('vpnup.sh','w')
    downfile=open('vpndown.sh','w')
    
    upfile.write(upscript_header)
    upfile.write('\n')
    downfile.write(downscript_header)
    downfile.write('\n')
    
    for ip,mask,_ in results:
        upfile.write('route add -net %s netmask %s gw $OLDGW\n'%(ip,mask))
        downfile.write('route del -net %s netmask %s\n'%(ip,mask))
    
    upfile.close()
    downfile.close()
    
    print "Old school way to call up/down script from openvpn client. " \
          "use the regular openvpn 2.1 method to add routes if it's possible"


def fetch_ip_data():
    #fetch data from apnic
    print "Fetching data from apnic.net, it might take a few minutes, please wait..."
    url=r'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest'
    data=urllib2.urlopen(url).read()
    
    cnregex=re.compile(r'apnic\|cn\|ipv4\|[0-9\.]+\|[0-9]+\|[0-9]+\|a.*',re.IGNORECASE)
    cndata=cnregex.findall(data)
    
    results=[]

    for item in cndata:
        unit_items=item.split('|')
        starting_ip=unit_items[3]
        num_ip=int(unit_items[4])
        
        imask=0xffffffff^(num_ip-1)
        #convert to string
        imask=hex(imask)[2:]
        mask=[0]*4
        mask[0]=imask[0:2]
        mask[1]=imask[2:4]
        mask[2]=imask[4:6]
        mask[3]=imask[6:8]
        
        #convert str to int
        mask=[ int(i,16 ) for i in mask]
        mask="%d.%d.%d.%d"%tuple(mask)
        
        #mask in *nix format
        mask2=32-int(math.log(num_ip,2))
        
        results.append((starting_ip,mask,mask2))
         
    return results


if __name__=='__main__':
    parser=argparse.ArgumentParser(description="Generate routing rules for vpn.")
    parser.add_argument('-p','--platform',
                        dest='platform',
                        default='openvpn',
                        nargs='?',
                        help="Target platforms, it can be openvpn, mac, linux," 
                        "win, android. openvpn by default.")
    parser.add_argument('-m','--metric',
                        dest='metric',
                        default=5,
                        nargs='?',
                        type=int,
                        help="Metric setting for the route rules")
    
    args = parser.parse_args()
    
    if args.platform.lower() == 'openvpn':
        generate_ovpn(args.metric)
    elif args.platform.lower() == 'linux':
        generate_linux(args.metric)
    elif args.platform.lower() == 'mac':
        generate_mac(args.metric)
    elif args.platform.lower() == 'win':
        generate_win(args.metric)
    elif args.platform.lower() == 'android':
        generate_android(args.metric)
    else:
        print>>sys.stderr, "Platform %s is not supported."%args.platform
        exit(1)
```