
UNDER construction! 

This content is outdated! It belongs to the previous openstack installer I have written a year ago. Please discard it. DO NOT READ FURTHER.  

<<<<<<< HEAD
# openstack-ocata-installer

Copyright 2017 Kasidit Chanchio 

Author: กษิดิศ ชาญเชี่ยว <br>
Contact: kasiditchanchio@gmail.com <br>
Department of computer science <br>
Faculty of science and technology <br>
Thammasat University.

<p>
<h2>Tutorial: การติดตั้งระบบ OpenStack Ocata แบบ Multi-node & DVR ด้วย installation scripts บน ubuntu 16.04 </h2> <br>
<p>
ให้ นศ เตรียมเครื่องตามส่วนที่ 1 และหลังจากนั้นเลือกเอาอันใดอันหนึ่งว่าจะติดตั้งด้วย scripts(ส่วนที่ 2) หรือด้วยมือ (ส่วนที่ 3)  
<ul>
 <li> 1. <a href="#part1">เตรียมเครื่องและเนตสำหรับติตดั้ง</a>
      <ul>
       <li> <a href="#kvmhost">1.1 การเตรียมเครื่องเพื่อติดตั้งบน kvm vm หรือเครื่องจริง</a>
       <li> <a href="#vboxhost">1.2 การเตรียมเครื่องสำหรับติดตั้งบน vbox vm</a>
       <li> <a href="#btrfssnapshot">1.3 การสร้าง snapshot บน btrfs บน ubuntu 16.04 host</a>
      </ul>
 <li> 2. <a href="#part2">ติดตั้งด้วย scripts</a> 
      <ul>
       <li> <a href="#downloadinstaller">2.1 ดาวน์โหลด openstack-ocata-installer scripts</a>
       <li> <a href="#paramrc">2.2 กำหนดค่าพารามีเตอร์สำหรับการติดตั้ง </a>
       <li> <a href="#usescript">2.3 ติดตั้ง OpenStack ocata ด้วย scripts </a> 
       <li> <a href="#testhorizon">2.4 ใช้งาน OpenStack Horizon</a>
      </ul>
 <li> 3. <a href="#part3">ติดตั้งด้วยมือ</a> 
      <ul>
       <li> <a href="#ubunupdate">3.1 update ubuntu บน ทุก node</a>
       <li> <a href="#setnicchrony">3.2 กำหนดค่า Network Interfaces และ Time Synchronization (chrony)</a>
       <li> <a href="#installmysql">3.3 ติดตั้ง mysql </a>
       <li> <a href="#installrabbitmq">3.4 ติดตั้ง rabbitmq </a>
       <li> <a href="#installkeystone">3.5 ติดตั้ง keystone </a>
       <li> <a href="#setendpoints">3.6 ติดตั้ง OpenStack Service Endpoint </a>
       <li> <a href="#installglance">3.7 ติดตั้ง glance </a>
       <li> <a href="#installnova">3.8 ติดตั้ง nova</a>
       <li> <a href="#installneutron">3.9 ติดตั้ง neutron </a>
       <li> <a href="#installhorizon">3.10 ติดตั้ง horizon </a>
       <li> <a href="#setdvr">3.11 กำหนด neutron ให้เป็นแบบ Distributed Virtual Routers </a>
       <li> <a href="#testnetwork">3.12 สร้าง network เริ่มต้น</a>
       <li> <a href="#testopenstack">3.13 ทดสอบ OpenStack</a>
      </ul>
 <li><a href="#part4">ส่วนที่ 4: สรุป</a>
</ul>
<table>
<tr><td><b>แจ้งปัญหาหรือข้อผิดพลาด:</b> หากมีส่วนใดของเเนื้อหาใน web นี้ที่เป็น BUGS หรือมีข้อแนะนำขอให้แจ้งได้ที่ kasiditchanchio@gmail.com นะครับ ขอบคุณครับ </td></tr>
</table>
<p>
<a id="part1"><h3>ส่วนที่ 1: เตรียมเครื่องและเนตสำหรับติดตั้ง</h3></a>
<p><p>
 เราจะมีการเตรียมการสองแบบคือ การเตรียมการสำหรับการติดตั้งโดยใช้ kvm vm หรือโดยใช้ virtual box vm (vbox)
<p>
 <i><a id="kvmhost"><h4>1.1 การเตรียมเครื่องเพื่อติดตั้งบน kvm vm หรือเครื่องจริง</h4></a></i>
<p> 
  นศ จะเตรียมเครื่อง ubuntu 16.04.x จำนวน 4 เครื่องเชื่อมต่อกันบนเนตดังภาพที่ 1 ได้แก่เครื่องชื่อ controller network compute และ compute1 (ชื่อเครื่องต้องตรงกับผลจากคำสั่ง hostname) จากภาพกำหนดให้เครื่องที่ controller มี spec แนะนำคือ cpu 4 cores RAM 6 ถึง 8 GB Disk 16-20 GB เครื่อง network มี cpu 1-2 cores RAM 512MB-1GB Disk 8-10 GB เครื่อง compute และ compute1 มี cpu 4 cores RAM 2-4 GB Disk 16-20 GB (เป็น spec ใช้สำหรับการศึกษา ถ้าจะ deploy ขอให้ดู official OpenStck doc) 
  <p>
  <img src="documents/architecture.png"> <br>
   ภาพที่ 1 <br>
กำหนดให้ทุกเครื่องมี username คือ opensatck และ password คือ openstack และเพื่อความสะดวกแนะนำว่าให้ทำให้ทุกเครื่องใช้ sudo โดยไม่ต้องป้อน password อีกอย่างที่สำคัญคือเครื่องเหล่านี้ควรมีเวลาใกล้เคียงกัน
<p><p>
สำหรับเนต (network) ที่จะใช้ในการติดตั้ง เรา <b>ASSUME</b> ว่ามี  management network รวมทั้ง network gateway ที่ใช้งานได้แอยู่เรียบร้อยแล้ว และมี data tunnel network และ vlan network ที่พร้อมจะใช้เชื่อมต่อกับเครื่องที่จะติดตั้งเรียบร้อยแล้ว 
<p><p> 
network ที่ใช้ในการติดตั้งได้แก่ 
 <ul>
 <li> management network: มี cidr 10.0.10.0/24 และ gateway คือ 10.0.10.1  openstack ใช้เนตนี้เป็นเนตหลักเพื่อออกอินเตอเนตและส่งคำสั่งระหว่างโหนด(หรือเครื่องทั้ง 4)ต่างๆของมัน  
 <li> data tunnel network: ใช้สร้าง tunnel สำหรับส่งข้อมูลของ vm ที่จะถูกสร้างขึ้นภายใน openstack เนตนี้ใช้สำหรับส่งข้อมูลระว่าง vm กันเอง (east-west) และระหว่าง vm กับ internet (north-south)
 <li> vlan network: ใช้ส่งข้อมูลระหว่าง vm ภายใน openstack กับ vlan network ภายนอก openstack 
 <li> external network: คือเนตที่เป็น internet service provider ของ openstack ในที่นี้เราจะใช้ management network 
 </ul>
จากภาพที่ 1 สมมุตว่า NIC ที่ 1 คือ ens3 NIC ที่ 2 คือ ens4 NIC ที่ 3 คือ ens5 NIC ที่ 4 คือ ens6 จะเห็นว่าเครื่อง conroller มี ens3 อันเดียว เครื่อง network compute แบะ compute1 ทั้งหมด มี ens3 ถึง ens6 
<p><p>
<table>
<tr><td>
<details>
 <summary><b>[กดเพื่อดูรายละเอียด] สำหรับวิชา คพ. 449: คำอธิบายการจำลองการติดตั้งโดยใช้ KVM virtual machines และ openvswitch network bridges</b></summary> 
เราจะจำลองการติดตั้งโดยใช้ kvm vm 4 เครื่องเชื่อมต่อกับ openvswitch network bridges บนเครื่อง server ใน lab ดังภาพที่ 2 
  <p>
  <img src="documents/architecturetunnel.png"> <br>
   ภาพที่ 2 <br>
จากภาพ สมมุติว่า server มี IP address คือ 10.100.13.13 เราจะใช้ openvswitch bridge จำลอง management network data tunnel network และ vlan network และใช้ KVM จำลอง openstack nodes บนเครื่องนั้น ผู้ใช้สามารถเข้าถึง vm ได้สองวิธี ได้แก่ 
 <ul>
    <li>การเข้าถึงโดยใช้ ssh tunneling ผ่าน putty โดยใช้เครื่อง server 10.100.13.13 เป็นตัวกลาง 
        <details>
         <summary><b>[กดเพื่อดูรายละเอียด] สำหรับวิชา cs449</b> นศ สามารถดูวิธีการทำ ssh tunneling ด้วย putty ได้ที่นี่</summary>
        <p><p>
        ในส่วนนี้ สมมุติว่าเครื่อง server คือ 10.100.20.133 และ นศ ต้องการ map ports ต่อไปนี้บนเครื่อง localhost ไปที่ endpoints ต่างๆ ได้แก่ localhost port 8080 map ไปที่ 10.1.10.11:80, localhost port 8011 map ไปที่ 10.1.10.11:22, localhost port 8021 map ไปที่ 10.1.10.21:22, localhost port 8031 map ไปที่ 10.1.10.31:22, localhost port 8031 map ไปที่ 10.1.10.31:22 นศ สามารถทำได้ด้วยวิธีต่อไปนี้
        <ul>
        <li>ให้เรียก putty และกำหนดค่า 10.100.20.133 ในช่อง Host Name (or IP address) ใน Putty Session window และเขียนชื่อ “mySession1” ในช่อง Save Session ดังภาพที่ s-1
          <p>
          <img src="documents/puttytunnel1.png"> <br>
          ภาพที่ s-1 <br>         
        <li>ถัดจากนั้นให้ดูที่ Category: ภายใต้ Connection ให้คลิ้กที่ SSH และให้ นศ เลือก Tunnel นศ จะได้ window ในภาพที่ s-2
         <p>
          <img src="documents/puttytunnel2.png"> <br>
          ภาพที่ s-2 <br>
        <li>ถัดจากนั้น ใน textbox “Source Port” และ “Destination” ให้กำหนดค่า mapping ดังที่ได้กล่าวข้างต้น และเมื่อกำหนดค่าตามแต่ละบรรทัดในตารางแล้วให้กดปุ่ม “Add” นศ จะได้ภาพดังภาพที่ s-3
         <p>
          <img src="documents/puttytunnel3.png"> <br>
          ภาพที่ s-3 <br>
        <li>หลังจากนั้น ให้ นศ คลิ้กที่ “Session” ภายใต้กล่อง Category: และให้กดปุ่ม “Save” session “mySession1” นี้ดังภาพที่ s-4
         <p>
          <img src="documents/puttytunnel4.png"> <br>
          ภาพที่ s-4 <br>
        <li>หลังจากนั้นให้ นศ login ด้วยการ load “mySession1” profile บน putty และ login เข้าสู่เครื่อง 10.100.20.133 โดยใช้ login id “cs449project” และพาสเวิด “openstack” ดังภาพที่ s-5
         <p>
          <img src="documents/puttytunnel5.png"> <br>
          ภาพที่ s-5 <br>
        <li>หลังจาก login เสร็จแล้วให้ นศ เปิด session นั้นทิ้งเอาไว้ และมีทางเลือกที่จะ login เข้าสู่เครื่อง controller network compute และ compute1
        <li>เปิด putty session ใหม่ดังในภาพที่ s-6 และ s-7 และกำหนดให้ Host Name (or IP address) เป็น “localhost” และ Port เป็น 8011 ซึ่งจะทำให้สามารถ login ผ่าน SSH Tunnel ที่สร้างไว้ใน session ก่อนหน้าเข้าสู่เครื่อง 10.1.10.11 
         <p>
          <img src="documents/puttytunnel6.png"> <br>
          ภาพที่ s-6 <br>
         <p>
          <img src="documents/puttytunnel7.png"> <br>
          ภาพที่ s-7 <br>
        </ul>
        </details>
    <li>การใช้ VNC client 
     <details>
         <summary><b>[กดเพื่อดูรายละเอียด] สำหรับวิชา cs449</b> นศ สามารถดูวิธีการใช้ vnc client ได้ที่นี่</summary>
        <p><p>
        สำหรับเครื่อง controller network compute compute1 ทุกเครื่องคือ kvm vm ที่มี vnc server รันอยู่ที่ port ต่อไปนี้ เครื่อง controller vm มี vnc server ที่ vnc endpoint 10.100.20.133:86 เครื่อง network vm มี vnc server ที่ vnc endpoint 10.100.20.133:87 เครื่อง compute vm มี vnc server ที่ vnc endpoint 10.100.20.133:88 เครื่อง compute1 vm มี vnc server ที่ vnc endpoint 10.100.20.133:89 
        <p><p>
        นศ สามารถติดตั้ง VNC เช่น TightVNC viewer บนเครื่อง client ของ นศ และเข้าถึงเครื่อง controller network compute และ compute1 ด้วย VNC viewer software ดังภาพที่ s-8 และภาพที่ s-9 
         <p>
          <img src="documents/puttytunnel8.png"> <br>
          ภาพที่ s-8 <br>
         <p>
          <img src="documents/puttytunnel9.png"> <br>
          ภาพที่ s-9 <br>      
     </details>
 </ul>
 </details>
 </td></tr> 
</table> 
<p><p>
เพื่อให้การติดตั้งเร็วขึ้นให้ นศ กำหนดค่า apt configuration ของเครื่องต่างๆให้ใช้ ubuntu repository ในประเทศไทย โดยกำหนดค่าใน /etc/apt/sources.list ด้วยมือ หรือใช้คำสั่ง sed ข้างล่าง บน openstack node ทุกเครื่อง 
<pre>
 $ sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
 $ sudo apt-get update
</pre>
และให้ dist-upgrade ทุกเครื่องด้วย 
<pre>
$ sudo dist-upgrade 
</pre>
ให้กำหนด network configuration ของทุกเครื่องบน management network ดังตัวอย่างข้างล่าง (เรา ASSUME ว่าทุก interface มี MTU คือ 1500 bytes) 
<p>
 <p>
 <b>เครื่อง controller </b> 
<pre>
openstack@controller:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.11
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8

openstack@controller:~$
</pre>
<pre>
openstack@controller:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:20:17
          inet addr:10.0.10.11  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:2017/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17777 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12906 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5715989 (5.7 MB)  TX bytes:2963058 (2.9 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@controller:~$
</pre>
<pre>
openstack@controller:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:20:17 brd ff:ff:ff:ff:ff:ff
openstack@controller:~$
</pre>
<p><p>
 <b>เครื่อง network </b>
<pre>
openstack@network:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.21
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@network:~$
</pre>

<pre>
openstack@network:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:21:17
          inet addr:10.0.10.21  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:2117/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4053 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3014 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4715574 (4.7 MB)  TX bytes:255812 (255.8 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)

openstack@network:~$

</pre>
<pre>
openstack@network:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:16 brd ff:ff:ff:ff:ff:ff
openstack@network:~$
</pre>
<p><p>
 <b>เครื่อง compute </b>
<pre>
openstack@compute:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.31
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@compute:~$
</pre>
<pre>
openstack@compute:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:31:17
          inet addr:10.0.10.31  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:3117/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16953 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10473 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:23969227 (23.9 MB)  TX bytes:771168 (771.1 KB)
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@compute:~$
</pre>
<pre>
openstack@compute:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:16 brd ff:ff:ff:ff:ff:ff
openstack@compute:~$
</pre>
<p><p>
 <b>เครื่อง compute1 </b>
<pre>
openstack@compute1:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.32
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@compute1:~$

</pre> 
<pre>
openstack@compute1:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:32:17
          inet addr:10.0.10.32  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:3217/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16827 errors:0 dropped:0 overruns:0 frame:0
          TX packets:9776 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:24038372 (24.0 MB)  TX bytes:693719 (693.7 KB)
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@compute1:~$
</pre>
<pre>
openstack@compute1:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:16 brd ff:ff:ff:ff:ff:ff
openstack@compute1:~$
</pre>
<p><p>
ขอให้ นศ make sure ว่า นศ สามารถใช้ NIC ทุกอันส่งข้อมูลได้ นศ อาจใช้วิธี ping IP address ใน management network โดยเช็คว่าสามารถ ping จาก controller ผ่าน ens3 ไปยัง IP ของ ens3 บนเครื่องอื่นทุกๆเครื่องได้ 
<p><p>
สำหรับ ens4 ens5 ens6 ให้<b>แอบกำหนดค่า IP</b> (หมายถึงกำหนดแล้วลบทิ้ง คือกำหนดเพื่อเช็คต่อไปนี้เฉยๆ แล้วลบทิ้ง ifdown หรือ ifconfig down ก่อนติดตั้งในส่วนที่ 2 หรือ 3) ให้กับ ensx interface ทุกๆอันที่เหลือและให้เช็คว่า ens4 IP ของทุกเครื่องสามารถ ping กันได้ และ ens5 IP ของทุกเครื่องสามารถ ping กันได้ และ ens6 IP ของทุกเครื่อง ping กันและกันได้ 
<p><p>
<b>หมายเหตุ</b> ขอให้ระวังว่า ens4 IP ไม่ควร ping ens3 IP หรือ ens5 IP หรือ ens6 IP ได้ พูดอีกอย่างคือ  data tunnel network subnet และ vlan network subnet และ management network subnet ต้อง isolate จากกัน 
<p><p>
เมื่อเช็คเสร็จแล้วให้ ลบ และ ifdown หรือ ifconfig down IP address ของ ens4 ens5 ens6 บนทุกเครื่องออก เราจะใช้ installation scripts กำหนดค่า หรือกำหนดค่าเองด้วยมือภายหลัง   
<p><p>
 <i><a id="vboxhost"><h4>1.2 การเตรียมเครื่องสำหรับติดตั้งบน vbox vm </h4></a></i>
<p>
นศ สามารถอ่านคำอธิบายการเตรียม vbox vm สำหรับติดตั้ง openstack ocata ได้ที่เอกสาร <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/openstack-ocata-vbox-vm-preparation.pdf">documents/openstack-ocata-vbox-vm-preparation.pdf</a> ขอให้สร้าง vm และกำหนดค่าต่างๆตามนั้น 
<p><p>
นศ สามารถดูคลิป youtube ประกอบได้ที่ (ผมไม่ได้เตียม script พูดประกอบ clip ขอให้ทนฟังการพูดตะกุกตะกักหน่อยนะครับ :-) ) <a href="https://www.youtube.com/watch?v=AkDoef8gUJY&index=1&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">set up virtual gateway on controller</a> และ <a href="https://www.youtube.com/watch?v=N3AfvrlJw2M&index=2&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">install and setup network, compute, and compute1 vbox vms</a>
<p><p>
<i><a id="btrfssnapshot"><h4>1.3 การสร้าง snapshot บน btrfs บน ubuntu 16.04 host</h4></a></i>
<p>
เนื้อหาในส่วนนี้ใช้สำหรับผู้ที่ประสงค์ใช้ btrfs เป๋น file system ของ host computers ที่จะใช้ติดตั้ง openstack และต้องการทำ snapshot ของ partition ที่ใช้ในการติดตั้งบนแต่ละเครื่องเป็นระยะๆ ถ้า นศ ไม่ได้ใช้ btrfs ก็ให้ข้ามส่วนนี้ไป
<p><p>
นศ สามารถติดตั้ง btrfs บน ubuntu 16.04 บนเครื่อง controller network compute compute1 hosts ระหว่างการติดตั้ง OS เมื่อกำหนด disk partitioning
<ul>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 1 </summary> 
  <p>
  <img src="documents/btrfssetup1.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 2</summary> 
  <p>
  <img src="documents/btrfssetup2.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 3</summary> 
  <p>
  <img src="documents/btrfssetup3.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 4</summary> 
  <p>
  <img src="documents/btrfssetup4.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 5</summary> 
  <p>
  <img src="documents/btrfssetup5.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 6</summary> 
  <p>
  <img src="documents/btrfssetup6.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 7</summary> 
  <p>
  <img src="documents/btrfssetup7.png"> <br>
</details>
<li>
<details>
<summary>[กดเพื่อดูรายละเอียด] ภาพ setup btrfs ที่ 8</summary> 
  <p>
  <img src="documents/btrfssetup8.png"> <br>
</details>
</ul>
หลังจากนั้นให้ นศ ติดตั้ง ubuntu ต่อตามปกติ 
<p><p>
เมื่อติดตั้งเสร็จแล้ว ให้ นศ login เข้าสู่เครื่องนั้นและดู btrfs subvolume ที่มีอยู่ในเครื่อง host ซึ่งหลังจากการติดตั้งข้างต้น ubuntu 16.04 จะสร้าง btrfs subvolmes สำหรับ / และ /home directory ให้ตั้งแต่เริ่มต้น
<pre>
$ sudo su
# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           396M  5.5M  390M   2% /run
/dev/sda1        10G  2.0G  6.3G  25% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/sda1        10G  2.0G  6.3G  25% /home
tmpfs           396M     0  396M   0% /run/user/1000
# 
# mount /dev/sda1 /mnt 
#
</pre>
นศ ควร modify ไฟล์ /etc/fstab ด้วยการเพิ่มบรรทัดข้างล่าง เพื่อให้มีการสร้าง /mnt directory และ mount เข้ากับ /dev/sda1 device โดยอัตโนมัติเมื่อมีการ reboot
<p><p>
<pre>
# vi /etc/fstab
...
/dev/sda1       /mnt            btrfs   defaults   0    1
...
(ให้เซฟไฟล์ และออกจาก vi)
#
</pre>
ในอันดับถัดไป นศ list btrfs subvolume ซึ่ง ubuntu จะสร้าง subvolume /mnt/@ สำหรับ / directory และ /mnt/@home สำหรับ /home directory
<p><p>
<pre>
# btrfs subvolume list /mnt
ID 261 gen 7810 top level 5 path @
ID 262 gen 7702 top level 5 path @home
#
</pre>
นศ สามารถทำ defragmentation ด้วยคำสั่งต่อไปนี้
<p><p>
<pre>
# btrfs filesystem defrag /mnt
</pre>
นศ สามารถทำ snapshot ของ /mnt/@ และ /mnt/@home ดังนี้
<p><p>
<pre>
# <b>btrfs subvolume snapshot /mnt/@ /mnt/@_snapshot1</b>
Create a snapshot of '/mnt/@' in '/mnt/@_snapshot1'
# <b>btrfs subvolume snapshot /mnt/@home /mnt/@home_snapshot1</b>
Create a snapshot of '/mnt/@home' in '/mnt/@home_snapshot1'
# btrfs subvolume list /mnt
ID 261 gen 7812 top level 5 path @
ID 262 gen 7813 top level 5 path @home
ID 264 gen 7812 top level 5 path @_snapshot1
ID 265 gen 7813 top level 5 path @home_snapshot1
#
</pre>
หลังจากนั้น ถ้า นศ ติดตั้ง openstack แล้วเกิดความผืดพลาดขึ้น นศ สามารถกู้คืน / และ /home ด้วยคำสั่งต่อไปนี้ 
<p><p>
<pre>
# mv /mnt/@ /mnt/@_badroot
# mv /mnt/@home /mnt/@_badhome
# mv /mnt/@_snapshot1 /mnt/@
# mv /mnt/@home_snapshot1 /mnt/@home
#
# reboot
</pre>
เมื่อ reboot เสร็จ ให้ login เข้าเครื่อง sudo เป็น root แล้ว ลบ /mnt/@_badroot และ /mnt/@_badhome
<p><p>
<pre>
# btrfs subvolume delete /mnt/@_badroot
# btrfs subvolume delete /mnt/@_badhome
</pre>
หลังจากนั้นให้สร้าง snapshot ของ /mnt/@ และ /mnt/@home อีกครั้งหนึ่ง
<p><p>
<pre>
# btrfs subvolume snapshot /mnt/@ /mnt/@_snapshot1
# btrfs subvolume snapshot /mnt/@home /mnt/@home_snapshot1
# btrfs filesystem defrag /mnt
</pre>
ผม recommend ให้ นศ ทำ snapshot ของ /mnt/@ และ /mnt/@home เมื่อผ่านการติดตั้งที่สำคัญๆ เผื่อว่าการติดตั้งในอนาคตผิดพลาด นศ จะได้ recover snapshot ล่าสุดได้
<a id="part2"> 
<h3>ส่วนที่ 2: ติดตั้งด้วย scripts</h3>
</a>
<p>
<i><a id="downloadinstaller"><h4>2.1 ดาวน์โหลด openstack-ocata-installer scripts</h4></a></i>
<p><p>
นศ จะใช้เครื่อง controller เป็นหลักในการติดตั้งด้วย script เริ่มต้นด้วยการ login เข้า openstack user (makes sure ว่า username และ password คือ "openstack" บนทุกเครื่อง) และ download script ด้วยคำสั่ง (ดู <a href="https://www.youtube.com/watch?v=a0JQYmsaPIs&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=3">youtube video</a>)
<pre>
$ cd $HOME
$ git clone https://github.com/kasidit/openstack-ocata-installer
$ cd openstack-ocata-installer
</pre>
<p>
เมื่อดู content ของ directory จะมีไฟล์และ subdirectory ดังนี้
<pre>
$ ls
config.d   exe-config-installer.sh  LICENSE                README.md
documents  install-paramrc.sh       OPSInstaller-init.tar
$
</pre>
<p>
ในกรณีที่ นศ จะติดตั้งบน vbox vm ขอให้ copy ไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.vbox.install-paramrc.sh">Example.vbox.install-paramrc.sh</a> มาเป็น install-paramrc.sh ใน openstack-ocata-installer directory
<pre>
$ cp documents/Example.vbox.install-paramrc.sh install-paramrc.sh
</pre>
<p>
แต่ถ้าติดตั้งบน kvm หรือเครื่องจริงก็ให้ใช้ไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/install-paramrc.sh">install-paramrc.sh</a> ที่มีอยู่แต่เดิมเป็นตัวอย่าง <br> 
<p>
<i><a id="paramrc"><h4>2.2 กำหนดค่าพารามีเตอร์สำหรับการติดตั้ง </h4></a></i>
<p>
ต่อไป นศ จะกำหนด configuration สำหรับการติอตั้งโดยกำหนดค่าในไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/install-paramrc.sh">install-paramrc.sh</a> ซึ่งถ้า นศ กำหนดค่า vm และเนตตามที่ระบุใน ส่วนที่ 1.1 และติดตั้งบน kvm (ที่ใช้รหัส ens เป็นชื่อ NIC) นศ ก็สามารถใช้ไฟล์ install-paramrc.sh นี้ได้เลย และในกรณีที่ นศ ใช้ vbox ติดตั้งและกำหนดค่าต่างๆเหมือนที่อธิบายในส่วน 1.2 นศ ก็สามารถใช้ไฟล์ Example.vbox.install-paramrc.sh ได้เลยเช่นกัน (ข้ามไปทำส่วนที่ 2.3 ได้)
<p><p>
แต่อย่างไรก็ตาม หาก นศ ติดตั้งบนเครื่องจริง ชื่อ NICs และค่าอื่นๆก็อาจเปลี่ยนไป ดังนั้นผมจะอธิบายความหมายของตัวแปรต่างๆในไฟล์ install-paramrc.sh เพื่อที่จะได้กำหนดค่าอย่างถูกต้อง (ดู <a href="https://www.youtube.com/watch?v=a0JQYmsaPIs&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=3">youtube video</a>) อันดับแรก environment variables สามอันแรกในไฟล์นั้นได้แก่
<pre>
export INSTALL_TYPE=full
export NETWORK_TYPE=dvr_ovs
export PASSWD_TYPE=studypass
</pre>
มีความหมายดังนี้ INSTALL_TYPE เป็นแบบ "full" คือเป็นการติดั้งแบบ 4 nodes ถ้าเปลี่ยนค่าเป็น "compact" จะหมายถึงติดตั้งแบบ 3 nodes ได้แก่ controller network และ compute ส่วน NETWORK_TYPE เป็นตัวแปรที่ระบุชนิดของ network deployment ถ้ากำหนดค่าเป็น "dvr_ovs" หมายถึงใช้ neutron ที่สร้างด้วย openvswitch และปฏิบัติงานแบบ Distributed Virtual Router (DVR) ซึ่งเป็น default configuration ถ้าเปลี่ยนค่าเป็น "classic_ovs" จะหมายถีง neutron ที่สร้างด้วย openvswitch แต่ปฏิบัติงานแบบธรรมดา ไม่มี DVR high availabiility feature สำหรับตัวแปร PASSWD_TYPE เป็นตัวแปรที่ระบุชนิดของ password ที่จะถูกกำหนดสำหรับการติดตั้ง component ต่างๆของ openstack ถ้า่าเป็น "studypass" หมายถึงการกำหนดค่า password ที่เป็น string ธรรมดาที่สื่อความหมายว่าเป็น password ของ component ใด ในทางตรงกันข้าม ถ้ากำหนดค่าเป็น "randompass" จะหมายถึงการกำหนดค่า password สำหรับการติดตั้ง component เหล่านั้นให้เป็นตัวเลข random
<pre>
export OPS_LOGIN_NAME=openstack
export OPS_LOGIN_PASS=openstack
export OPS_TIMEZONE=Asia\\/Bangkok
</pre>
ตัวแปร OPS_LOGIN_NAME และ OPS_LOGIN_PASS ในที่นี้เรากำหนดให้เป็น "openstack" ทั้งคู่ ค่า OPS_LOGIN_NAME และ OPS_LOGIN_PASS นี้ต้องตรงกับชื่อ login name และค่า password ของ Linux account ที่ นศ จะใช้ติดตั้ง OpenStack บนทุก node ส่วน OPS_TIMEZONE นั้นใช้กำหนดค่ว TIMEZONE ซึ่งในทีนี้คือ Asia/Bangkok
<p><p>
อันดับถัดไปคือการกำหนดค่า root password ของ mysql ซึ่ง นศ ต้องจำด้วยว่ากำหนดค่าตัวแปรนี้ว่าอะไร เพราะในระหว่างการติดตั้ง นศ ต้องป้อนค่านี้ด้วยตนเองอีกครั้งหนึ่ง สำหนับ DEMO_PASS และ ADMIN_PASS คือค่า password ของ "demo" user และ "admin" user หลังจากเสร็จสิ้นการติดตั้ง OpenStack 
<pre>
export OPS_MYSQL_PASS=mysqlpassword
export DEMO_PASS=demopassword
export ADMIN_PASS=adminpassword
#
export HYPERVISOR=qemu
</pre>
ถัดจจากนั้นจะเป็นการกำหนดค่า HYPERVISOR ให้เป็น "qemu" ในกรณีที่ นศ ติดตั้งบนเครื่องจริง ให้เปลี่ยนค่าของตัวแปรนี้เป็น "kvm" แทน สำหรับตัวแปรถัดไปต่อไปนี้ นศ ไม่ต้องไปยุ่งกับมันก็ได้ มันเป็นการกำหนดค่า url ของ cirros OS image ที่ script จะไป download มา ตัวแปร LOCAL_REPO เป็นการกำหนดค่า apt repository และตัวแปร NTP_SERVER เป็นตัวแปรกำหนดค่า NTP server ซึ่งถ้าติดตั้งในเมืองไทยคงไม่ต้องเปลี่ยนอะไร
<pre>
export INIT_IMAGE_LOCATION=http:\\/\\/download.cirros-cloud.net\\/0.3.5\\/cirros-0.3.5-x86_64-disk.img
export INIT_IMAGE_NAME=cirros
#
export DOMAINNAME=cs.tu.ac.th
#
# Ubuntu Repository Parameters
#
export LOCAL_REPO=th.archive.ubuntu.com
export LOCAL_SECURITY_REPO=security.ubuntu.com
#
# ntp servers
export NTP_SERVER0=0.th.pool.ntp.org
export NTP_SERVER1=1.th.pool.ntp.org
export NTP_SERVER2=2.th.pool.ntp.org
export NTP_SERVER3=3.th.pool.ntp.org
#
# Deprecate
export NTP_SERVER_LOCAL=10.0.10.126 
</pre>
<p>
อันดับถัดไปจะเป็นการกำหนดค่า network configuration ในกรณีที่ นศ จะติดตั้งด้วย script และต้องการกำหนดค่าตัวแปรที่แตกต่างจากที่ระบุในส่วนที่ 1 นศ ควรทราบความหมายของตัวแปรเหล่านี้ 
<details>
<summary><b>[กดเพื่อดูรายละเอียด] ภาพที่ 3 แสดงการ mapping ของค่าตัวแปรใน install-paramrc.sh กับค่า network configuration ในภาพที่ 1</b></summary> 
  <p>
  <img src="documents/architecturevariables.png"> <br>
   ภาพที่ 3 <br>
</details>
<p>
<p>
จากภาพ ตัวแปรต่อไปนี้ใช้กำหนดค่าของ management network 
<pre>
export MANAGEMENT_NETWORK_NETMASK=255.255.255.0
export MANAGEMENT_NETWORK=10.0.10.0
export MANAGEMENT_BROADCAST_ADDRESS=10.0.10.255 
export DNS_IP=8.8.8.8
</pre>
ถัดจากนั้นเป็นตัวแปร GATEWAY_IP_NIC ซึ่งจะใช้สำหรับการติดตั้งด้วย vbox (ซึ่งจะใช้ controller เป็น virtual gateway) เท่านั้น เนื่องจากเราไม่ได้ใช้ vbox นศ จึงไม่ต้องกำหนดค่าให้ตัวแปรนี้ 
<p><p>
ตัวแปร CONTROLLER_IP และ CONTROLLER_IP_NIC ใช้ระบุค่า IP address และ NIC แรกของเครื่อง controller และตัวแปร GATEWAY_IP ใช้ระบุค่า IP address ของ gateway router ของ management network ซึ่งในที่นี้จะหมายถึง IP address ของ gateway router ของ external network ด้วย เพราะเราจะใช้ management network เป็น external network ในการติดตั้งของเรา
<pre>
export CONTROLLER_IP=10.0.10.11
export CONTROLLER_IP_NIC=ens3
#
export GATEWAY_IP=10.0.10.1
</pre>
<p>
ต่อไปเป็นการกำหนดค่า IP address ของ network node (ตัวแปร NETWORK_IP) และค่า NIC ของ network node ที่เชื่อมกับ management network (NETWORK_NODE_IP_NIC) ถัดจากนั้นจะเป็นการกำหนดค่าตัวแปรสำหรับ NIC ที่เชื่อมต่อ Data tunnel network ของ network node ได้แก่ DATA_TUNNEL_NETWORK_NODE_IP และ DATA_TUNNEL_NETWORK_NODE_IP_NIC และ DATA_TUNNEL_NETWORK_ADDRESS และ DATA_TUNNEL_NETWORK_NETMASK 
<pre>
export NETWORK_IP=10.0.10.21
export NETWORK_IP_NIC=ens3
#
export DATA_TUNNEL_NETWORK_NODE_IP=10.0.11.21
export DATA_TUNNEL_NETWORK_NODE_IP_NIC=ens4
export DATA_TUNNEL_NETWORK_ADDRESS=10.0.11.0
export DATA_TUNNEL_NETWORK_NETMASK=255.255.255.0
</pre>
นอกจากเชื่อมต่อกับ management และ data tunnel network แล้ว network node ยังต่อกับ Vlan network และ External network ด้วยซึ่ง นศ จะกำหนดค่าของทั้งสอง network ดังนี้
<pre>
export VLAN_NETWORK_NODE_IP_NIC=ens5
#
export EXTERNAL_CIDR=10.0.10.0\\/24
export EXTERNAL_CIDR_NIC=ens6
#
export START_FLOATING_IP=10.0.10.100
export END_FLOATING_IP=10.0.10.200
</pre>
<p>
จะเห็นได้ว่า การกำหนดค่าของ vlan network นั้นไม่ต้องทำอะไรมาก แค่กำหนดค่าตัวแปร VLAN_NETWORK_NODE_IP_NIC เพื่อระบุว่า NIC ไหนบน network node เชื่อมต่อกับ Vlan network (openstack จะมี CLI ให้ผู้ใช้ๆกำหนดค่าของ vlan network ได้หลังจากการติดตั้ง)  
<p><p>
ส่วนตัวแปร EXTERNAL_CIDR_NIC คือการบอก OpenStack ว่า NIC ไหนบน network node ที่จะใช้ติดต่อกับ network ที่ติดต่อกับ Internet ได้ (หรือ External network) 
และเนื่องจากเครื่อง compute และ compute1 ก็มี NIC ต่อกับ External network เช่นกัน ผมจะใช้ค่าของตัวแปร EXTERNAL_CIDR_NIC ตัวแปรเดียวเพื่อกำหนดค่า NIC ดังกล่าวสำหรับ network compute และ compute1 node 
<p><p>
ถัดจากนั้น นศ จะต้องระบุ CIDR ของ External network (ด้วยตัวแปร EXTERNAL_CIDR) ซึ่งในการติดตั้งนี้เราจะใช้ CIDR ของ management network 
<p><p>
หลังจากนั้น นศ ต้องกำหนดค่า range ของ Floating IP โดยให้ตัวแปร START_FLOATING_IP เป็นค่า IP address เริ่มต้น และตัวแปร END_FLOATING_IP เป็นค่า IP address สุดท้าย  
<p><p>
นศ ต้องระบุค่าตัวแปร COMPUTE_NODE_IP เพื่อกำหนดค่า IP address บน management network ของ compute node และกำหนดค่า COMPUTE_NODE_IP_NIC เพื่อระบุว่า NIC ไหนของ compute node ที่ใช้ต่อกับ management network ตัวแปร DATA_TUNNEL_COMPUTE_NODE_IP และ DATA_TUNNEL_COMPUTE_NODE_IP_NIC ใช้กำหนดค่า IP address และ NIC ที่เชื่อมต่อกับ data tunnel network ส่วน VLAN_COMPUTE_NODE_IP_NIC ใช้ระบุค่า NIC ที่เชื่อมต่อกับ Vlan network
<pre>
export COMPUTE_IP=10.0.10.31
export COMPUTE_IP_NIC=ens3
export DATA_TUNNEL_COMPUTE_NODE_IP=10.0.11.31
export DATA_TUNNEL_COMPUTE_NODE_IP_NIC=ens4
export VLAN_COMPUTE_NODE_IP_NIC=ens5
</pre>
ในไฟล์ install-paramrc.sh เรากำหนดค่าตัวแปรสำหรับ compute1 node ในแบบเดียวกันกับการกำหนดค่าของ compute node ข้างต้น 
<p>
<p>
<i><a id="usescript"><h4>2.3 การติดตั้ง OpenStack ocata ด้วย scripts </h4></a></i>
<p>
<p>
เริ่มต้นการติดตั้งด้วยคำสั่งต่อไปนี้ (หมายเหตุ นศ ต้องออกคำสั่งใน user mode คือเป็น openstack user ห้ามใช้ sudo จนจบ script เหล่านี้)
<pre>
$ cd $HOME/openstack-ocata-installer
$ ./exe-config-installer.sh
</pre>
คำสั่ง ./exe-config-installer.sh จะนำค่าที่กำหนดใน install-paramrc.sh ไปแทนค่า template ของ scripts สำหรับติดตั้ง openstack ในไฟล์ OPSInstaller-init.tar และสร้าง directory ใหม่ชือ OPSInstaller ขึ้น (ดู <a href="https://www.youtube.com/watch?v=zIVLVEvaDgs&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=4">youtube video</a>)
<p><p>
ให้ นศ cd เข้า directory ดังกล่าวดังนี้ 
<pre>
$ cd OPSInstaller/installer
</pre>
 ในกรณีที่ นศ ติดตั้งบน vbox นศ จะต้อง script OS-installer-00-0-set-gateway.sh ข้างล่างนี้เพื่อทำให้เครื่อง controller เป็น virtual gateway สำหรับ management network <b>ถ้าไม่ได้ใช้ vbox ให้ข้ามไปรัน script ถัดไปเลย</b>
<pre>
$ ./OS-installer-00-0-set-gateway.sh 
</pre>
ถัดจากนั้นรัน script แรกเพื่อทำให้สามารถ remote ssh จาก controller ไปยังเครื่องอื่นๆได้โดยไม่ต้องใส่ password (<b>หมายเหตุ:</b> ในกรณีที่ นศ จะใช้ script ติดตั้งเพื่อใช้งานจริง หลังจากติดตั้งเสร็จเรียบร้อยแล้ว นศ ต้องทำสองอย่างได้แก่ (1) เปลี่ยน password ของ openstack user บนทุกเครื่องและ (2) ลบเนื้อหาของไฟล์ $HOME/.ssh/id_rsa และไฟล์ $HOME/.ssh/authorized_keys ใน openstack user บนทุกเครื่อง) (ดู <a href="https://www.youtube.com/watch?v=zIVLVEvaDgs&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=4">youtube video</a>)
<pre>
$ ./OS-installer-00-1-set-remote-access.sh
</pre>
script ที่สองจะ update ubuntu 16.04 บนโหนดต่างๆให้เป็นเวอรชันล่าสุดและกำหนด cloud repository สำหรับ openstack ocata installation
<pre>
$ ./OS-installer-00-2-update-ubuntu.sh 
</pre>
script จะ remote ssh เข้าไปที่เครื่อง controller network compute และ compute1 และในระหว่างที่ update ubuntu ของแต่ละเครื่อง มันจะถามให้ นศ กด [ENTER] เครื่องละครั้ง หลังจาก update ubuntu บนแต่ละเครื่องเสร็จมันจะ reboot เครื่องเหล่านั้น โดยจะ reboot เครื่อง controller หลังสุด
<p><p>
ในอันดับถัดไป เราจะเริ่มต้นด้วยการกำหนดค่า network configurations ที่จำเป็นสำหรับการติดตั้ง openstack ด้วย OS-installer-01-node-setups.sh ซึ่งจะกำหนดค่าและ ifup interfaces ต่างๆบนทุกๆเครื่องในภาพที่ 1 และติดตั้ง chrony เพื่อ sync เวลาระหว่าง NTP server กับ controller และระหว่าง controller กับทุกๆ node (ดู <a href="https://www.youtube.com/watch?v=ii7Ty4cW6mQ&index=5&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">youtube video</a>)
<pre>
$ ./OS-installer-01-node-setups.sh
</pre>
หลังจากนั้น นศ จะติดตั้ง mysql ด้วย script OS-installer-02-mysql.sh (ดู <a href="https://www.youtube.com/watch?v=pYuxnxX_WZw&index=6&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">youtube video</a>)
<pre>
$ ./OS-installer-02-mysql.sh
</pre>
ขอให้ นศ จำรหัสผ่านสำหรับ root ของ mysql ที่กำหนดไว้ใน install-paramrc.sh ด้วย (ในที่นี้คือ "mysqlpassword") ระหว่างที่รัน script นี้ นศ จะต้องระวังและป้อนค่าตามที่ script ต้องการดังนี้
<ul>
<li> หลังจากติดตั้ง mysql แล้ว script จะให้ป้อนค่า root password ซึ่งไม่มีเพราะเป็นการติดตั้งใหม่ ดังนั้น นศ ต้อง กด <b>ENTER</b>
<li> ถัดจากนั้นมันจะให้ป้อน password (<b>mysqlpassword</b>) สองครั้ง
<li> คำถามที่เหลือตอบ y ให้หมด 
</ul>
<p><p>
นศ จะติดตั้ง rabbitmq ซึ่งเป็น message queue software ที่ components ของ openstack ใช้สื่อสารกัน 
<pre>
$ ./OS-installer-03-rabbitmq.sh
</pre>
ถัดจากนั้นจะติดตั้ง keystone 
<pre>
$ ./OS-installer-04-keystone.sh
</pre>
ตามด้วย glance
<pre>
$ ./OS-installer-05-glance.sh
</pre>
และ nova (ดู <a href="https://www.youtube.com/watch?v=dRQ9GPtPCZs&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=7">youtube video</a>)
<pre>
$ ./OS-installer-06-nova.sh
</pre>
และ neutron (ดู <a href="https://www.youtube.com/watch?v=5gC8dntxaE8&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=8">youtube video</a>)
<pre>
$ ./OS-installer-07-neutron.sh
</pre>
ติดตั้ง horizon web gui (ถ้าเครื่อง cpu หรือ memory น้อย ผมแนะนำให้ใช้ CLI แทน web interface) 
<pre>
$ ./OS-installer-08-horizon.sh
</pre>
กำหนดค่า network ให้เป็น Distributed Virtual Router (DVR) (ดู <a href="https://www.youtube.com/watch?v=A-NkW1xYylY&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&index=9">youtube video</a>)
<pre>
$ ./OS-installer-09-set-dvr.sh
</pre>
ใช้ script สร้าง network เริ่มต้นและทดสอบ network
<pre>
$ ./OS-installer-10-initial-user-network.sh
</pre>
<p>
<p>
<i><a id="testhorizon"><h4>2.4 ใช้งาน OpenStack Horizon</h4></a></i>
<p>
<p>
ในกรณีที่ติดตั้งบนเครื่องจริง นศ ควรจะเข้าใช้ web interface ของ openstack ได้ที่ http://10.0.10.11:80/horizon/ (ดู <a href="https://www.youtube.com/watch?v=uXjlmfOvFCs&index=10&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">youtube video</a>)
<p>
<table>
<tr><td>
<details>
<summary><b>[กดเพื่อดูรายละเอียด] สำหรับวิชา คพ. 449: คำอธิบายการเข้าถึง horizon web interface ผ่าน ssh tunneling</b></summary>
เนื่องจากเราใช้ KVM (ดูภาพที่ 2) นศ ต้องสร้าง ssh tunnel โดยใช้ "tunnel" feature ของ putty และกำหนดให้ port 8088 ของเครื่อง client computer ที่ นศ ใช้ (เครื่อง localhost) เข้ากับ URI 10.0.10.11:80 ที่เข้าถึงได้จากเครื่อง server ที่ นศ ติดตั้ง KVM (เครื่อง 10.100.13.13 ในภาพที่ 2) 
<p><p>
หลังจาก login ด้วย putty เข้าสู่เครื่อง server แล้ว นศ สามารถเข้าถึง web interface ของ openstack จาก client computer ที่ นศ รัน putty ได้ที่ URL http://localhost:8088/horizon/
</details>
</td></tr>
</table>
<p>
<a id="part3"> 
<h3>ส่วนที่ 3: ติดตั้งด้วยมือ</h3>
</a>
<p><p>
การติดตั้งด้วยมือเป็นวิธีการติดตั้งที่จะทำให้ผู้ติดตั้งได้มีโอกาสเรียนรู้จักส่วนประกอบของ OpenStack มากที่สุด OpenStack ประกอบไปด้วย software component หลาย software ผู้ใช้จะต้องติดตั้งและกำหนดค่าในไฟล์ configuration ของ component เหล่านั้น ซึ่งส่วนใหญ่จะอยู่ใน /etc directory 
<p><p>
เพื่อความสะดวกในการเขียนคู่มือนี้ ผมจะอ้างอิงถึง configuration ไฟล์ใน "files" subdirectory นศ สามารถดูตัวอย่างไฟล์ได้ตาม URL link บนแต่ละไฟล์และ เปรียบเทียบกับ official openstack ocata installation document [2] ที่ผมจะแนบ link ไว้ให้เป็นระยะๆแล้ว copy เนื้อหาของ configuration ไฟล์ไปใช้ตามที่จำเป็น หรือทำอีกวิธีหนึ่งคือ
<table><tr><td>
<details>
<summary>[กดเพื่อดูรายละเอียด] ใช้ scriptsสร้าง configuration ไฟล์ใน "files" directory แล้ว scp ไปไว้บนทุกๆเครื่องด้วยตนเอง  (คล้ายส่วนที่ 2.1 และ 2.2)</summary>
<pre>
$ cd $HOME
$ git clone https://github.com/kasidit/openstack-ocata-installer
$ cd openstack-ocata-installer
</pre>
กำหนดค่าใน install-paramrc.sh (<a href="#paramrc">ดูส่วนที่ 2.2</a>) และรัน ./exe-config-installer.sh
<pre>
$ ./exe-config-installer.sh
</pre>
หลังจากนั้น script จะสร้าง installation scripts ใน $HOME/openstack-ocata-installer/OPSInstaller/* directory และสร้าง configuration ไฟล์ใน  $HOME/openstack-ocata-installer/OPSInstaller/*/files/ directories ซึ่ง นศ สามารถอ้างอิงถึง configuration ไฟล์ใน "files" directory และดูเป็นตัวอย่างได้ 
<p><p>
<ul>
<li>ถ้า นศ ติดตั้งด้วยมือบนเครื่อง controller ให้ดูตัวอย่าง configuration ไฟล์ที่  $HOME/openstack-ocata-installer/OPSInstaller/controller/files/ 
<li>ถ้า นศ ติดตั้งด้วยมือบนเครื่อง network ให้ดูไฟล์ที่  $HOME/openstack-ocata-installer/OPSInstaller/network/files/ 
<li>ถ้า นศ ติดตั้งด้วยมือบนเครื่อง compute ให้ดูไฟล์ที่  $HOME/openstack-ocata-installer/OPSInstaller/compute/files/
<li>ถ้า นศ ติดตั้งด้วยมือบนเครื่อง compute1 ให้ดูไฟล์ที่  $HOME/openstack-ocata-installer/OPSInstaller/compute1/files/
</ul>
ถ้า นศ ต้องการ sudo cp ไฟล์เหล่านี้ เหมือนในคำสั่งที่จะใช้ต่อไปในการติดตั้งด้วยมือ นศ ต้อง tar ไฟล์ใน "files" directory เหล่านี้และ scp ไปยังเครื่อง controller network compute และ compute1 และ untar ด้วยตนเองก่อนการติดตั้ง 
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller
$ cd controller 
$ tar cvf controller-files.tar files
$ cp controller-files.tar $HOME/
$ cd $HOME 
$ tar xvf controller-files.tar
$ ls
</pre>
ls จะแสดง directory "files" ใน openstack account บนเครื่อง controller
<p><p>
ถ่ายโอนข้อมูลจาก directory network/files ไปที่เครื่อง network node
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller
$ cd network 
$ tar cvf network-files.tar files
$ scp network-files.tar openstack@10.0.10.21:/home/openstack/
$ ssh openstack@10.0.10.21 tar xvf /home/openstack/network-files.tar
$ ssh openstack@10.0.10.21 ls -l
</pre>
ls จะแสดง directory "files" ใน openstack account บนเครื่อง network
<p><p>
ถ่ายโอนข้อมูลจาก directory compute/files ไปที่เครื่อง compute node
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller
$ cd compute 
$ tar cvf compute-files.tar files
$ scp compute-files.tar openstack@10.0.10.31:/home/openstack/
$ ssh openstack@10.0.10.31 tar xvf /home/openstack/compute-files.tar
$ ssh openstack@10.0.10.31 ls -l
</pre>
ls จะแสดง directory "files" ใน openstack account บนเครื่อง compute
<p><p>
ถ่ายโอนข้อมูลจาก directory compute1/files ไปที่เครื่อง compute1 node
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller
$ cd compute1 
$ tar cvf compute1-files.tar files
$ scp compute1-files.tar openstack@10.0.10.32:/home/openstack/
$ ssh openstack@10.0.10.32 tar xvf /home/openstack/compute1-files.tar
$ ssh openstack@10.0.10.32 ls -l
</pre>
ls จะแสดง directory "files" ใน openstack account บนเครื่อง compute1
</details>
</td></tr></table>
<p><p>
ขอให้ นศ ศึกษารายละเอียดของการกำหนดค่า configuration ใน [1][2][3] และตอบคำถาม PROJECT ของวิชา คพ 449 ที่จะถามในคู่มือนี้เป็นระยะๆ ขอให้บรรยายคำตอบในรายงานอย่างชัดเจน  
<p><p>
<i><a id="ubunupdate"><h4>3.1 update ubuntu บน ทุก node </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
login เข้า user openstack และ modify ไฟล์ /etc/hosts 
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller/controller
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/hosts">files/hosts</a> /etc/hosts
</pre>
รันคำสั่งต่อไปนี้
<pre>
$ sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -y install software-properties-common
$ sudo add-apt-repository cloud-archive:ocata
</pre>
คำสั่ง add-apt-repository จะรอให้ นศ กด [ENTER] 
<p>
<pre>
$ sudo apt-get update 
$ sudo apt-get -y dist-upgrade
$ sudo apt-get -y install python-openstackclient
$ sudo reboot
</pre>
<p><p>
<b>เครื่อง network</b>
<p><p>
login เข้า user openstack และใช้คำสั่งต่อไปนี้
<pre>
$ cd $HOME/openstack-ocata-installer/OPSInstaller/
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/hosts">files/hosts</a> /etc/hosts
$ sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -y install software-properties-common
$ sudo add-apt-repository cloud-archive:ocata
</pre>
คำสั่งสุดท้ายจะรอให้ นศ กด [ENTER] 
<p>
<pre>
$ sudo apt-get update 
$ sudo apt-get -y dist-upgrade
$ sudo apt-get -y install python-openstackclient
$ sudo reboot
</pre>
<p><p>
<b>เครื่อง compute</b> 
<p><p>
login เข้า user openstack และใช้คำสั่งต่อไปนี้
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/hosts">files/hosts</a> /etc/hosts
$ sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -y install software-properties-common
$ sudo add-apt-repository cloud-archive:ocata
</pre>
คำสั่งสุดท้ายจะรอให้ นศ กด [ENTER] 
<p>
<pre>
$ sudo apt-get update 
$ sudo apt-get -y dist-upgrade
$ sudo apt-get -y install python-openstackclient
$ sudo reboot
</pre>
<p><p>
<b>เครื่อง compute1</b> 
<p><p>
login เข้า user openstack และใช้คำสั่งต่อไปนี้
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/hosts">files/hosts</a> /etc/hosts
$ sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
$ sudo apt-get update
$ sudo apt-get -y install software-properties-common
$ sudo add-apt-repository cloud-archive:ocata
</pre>
คำสั่งสุดท้ายจะรอให้ นศ กด [ENTER] 
<p><pre>
$ sudo apt-get update 
$ sudo apt-get -y dist-upgrade
$ sudo apt-get -y install python-openstackclient
$ sudo reboot
</pre>
<p><p>
<i><a id="setnicchrony"><h4>3.2 กำหนดค่า Network Interfaces และ Time Synchronization (chrony) </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ sudo apt-get -y install chrony
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (1) ขอให้ นศ อธิบายว่า chrony ใช้ทำอะไร และค่าที่กำหนดในไฟล์ chrony.conf หมายถึงอะไร</td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-controller.html">Network Time Protocol on controller</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/chrony.conf.nocomment">chrony.conf</a>
</td></tr>
</table> 
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/chrony.conf">files/chrony.conf</a> /etc/chrony/chrony.conf
$ 
$ sudo service chrony restart
</pre>
<p><p>
<b>เครื่อง network</b>
<p><p>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (2) ไฟล์ interfaces ต่างจาก interfaces ใน ส่วนที่ 1 อย่างไร </td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/interfaces">files/interfaces</a> /etc/network/interfaces
</pre>
<pre>
$ sudo ifdown ens3
$ sudo ifup ens3
$ sudo ifdown ens4
$ sudo ifup ens4
$ sudo ifdown ens5
$ sudo ifup ens5
$ sudo ifdown ens6
$ sudo ifup ens6
$ ifconfig
$
$ sudo apt-get -y install chrony
</pre>
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (3) ค่าที่กำหนดในไฟล์ chrony.conf หมายถึงอะไร </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-other.html">Network Time Protocol on controller</a>, <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-verify.html">Verify</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/chrony.conf.nocomment">chrony.conf</a>
</td></tr> 
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/chrony.conf">files/chrony.conf</a> /etc/chrony/chrony.conf
$ sudo service chrony restart
</pre>
<p><p>
<b>เครื่อง compute</b>
<p><p>
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (4) ไฟล์ interfaces ต่างจาก interfaces ใน ส่วนที่ 1 อย่างไร </td></tr></table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/interfaces">files/interfaces</a> /etc/network/interfaces
$ sudo ifdown ens3
$ sudo ifup ens3
$ sudo ifdown ens4
$ sudo ifup ens4
$ sudo ifdown ens5
$ sudo ifup ens5
$ sudo ifdown ens6
$ sudo ifup ens6
$ 
$ sudo apt-get -y install chrony
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (5) ค่าที่กำหนดในไฟล์ chrony.conf หมายถึงอะไร </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-other.html">Network Time Protocol on controller</a>, <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-verify.html">Verify</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/chrony.conf.nocomment">chrony.conf</a>
</td></tr>  
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/chrony.conf">files/chrony.conf</a> /etc/chrony/chrony.conf
$ sudo service chrony restart
</pre>
<p><p>
<b>เครื่อง compute1</b>
<p><p>
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (6) ไฟล์ interfaces ต่างจาก interfaces ใน ส่วนที่ 1 อย่างไร </td></tr></table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/interfaces">files/interfaces</a> /etc/network/interfaces
$ sudo ifdown ens3
$ sudo ifup ens3
$ sudo ifdown ens4
$ sudo ifup ens4
$ sudo ifdown ens5
$ sudo ifup ens5
$ sudo ifdown ens6
$ sudo ifup ens6
$ 
$ sudo apt-get -y install chrony
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (7) ค่าที่กำหนดในไฟล์ chrony.conf หมายถึงอะไร </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-other.html">Network Time Protocol on controller</a>, <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-verify.html">Verify</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/chrony.conf.nocomment">chrony.conf</a>
</td></tr>   
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/chrony.conf">files/chrony.conf</a> /etc/chrony/chrony.conf
$ sudo service chrony restart
</pre>
<p><p>
<i><a id="installmysql"><h4>3.3 ติดตั้ง mysql </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ sudo chronyc sources
$ sudo apt-get -y install mariadb-server
$ sudo apt-get -y install python-pymysql
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (8) นศ ต้องกำหนดค่าอะไรบ้างในไฟล์  99-openstack.cnf </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-sql-database.html">Install/config mysql</a></td></tr>
</table>
<pre>
cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/99-openstack.cnf">files/99-openstack.cnf</a> /etc/mysql/mariadb.conf.d/99-openstack.cnf
service mysql restart
mysql_secure_installation
</pre>
คำสั่ง mysql_secure_installation script เป็น script สำคัญที่จะให้ นศ กำหนดค่าเริ่มต้นของ mysql database script นี้จะถามคำถามต่อไปนี้
<ul>
<li> 1. ให้ผู้ใช้กรอก root password ของ mysql เนื่องจาก นศ เพิ่งเริ่มติดตั้งจะไม่มี root password ให้ enter  
<li> 2. ให้ นศ กรอก password ใหม่ของ mysql root (กำหนดให้เป็นค่า mysqlpassword) โดย script จะถามให้กรอกค่านี้สองรอบ 
<li> 3. หลังจากนั้น script จะถามว่า ลบ test account ใหม่ และอื่นๆ ให้ตอบ "y" ให้หมด
</ul>
<p><p>
<b>เครื่อง compute</b>
<p><p>
<pre>
$ sudo chronyc sources
</pre>
<table>
 <tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (9) ผลการรัน chronyc ต่างจากบนเครื่อง controller อย่างไร </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-ntp-verify.html">Verify</a></td></tr>
</table>
<p><p>
<b>เครื่อง compute1</b>
<p><p>
<pre>
$ sudo chronyc sources
</pre>
<p><p>
<i><a id="installrabbitmq"><h4>3.4 ติดตั้ง rabbitmq </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ sudo apt-get -y install rabbitmq-server
$
$ sudo rabbitmqctl add_user openstack RABBIT_PASS
$ sudo rabbitmqctl set_permissions openstack ".*" ".*" ".*"
$
$ sudo apt-get install memcached python-memcache
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (10) memcache คืออะไร มีการกำหนดค่าอะไรใน memcached.conf </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-messaging.html">RabbitMQ</a>, <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/environment-memcached.html">Memcache</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/memcached.conf.nocomment">memcached.conf</a>
</td></tr> 
</table>
</pre>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/memcached.conf">files/memcached.conf</a> /etc/memcached.conf
$ sudo service memcached restart
</pre>
<p><p>
<i><a id="installkeystone"><h4>3.5 ติดตั้ง keystone </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
สร้าง database สำหรับ keystone ทบทวนความจำว่า root password ของ mysql ที่เรากำหนดในตัวอย่างนี้คือ mysqlpassword
<pre>
$ sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE keystone;"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';"
</pre>
ติดตั้ง keystone 
<p>
<pre>
S sudo apt-get -y install keystone
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (11) มีการกำหนดค่าอะไรใน keystone.conf  </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/keystone.html">Install Keystone</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/keystone.conf.nocomment">keystone.conf</a>
</td></tr>  
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/keystone.conf">files/keystone.conf</a> /etc/keystone/keystone.conf
$ sudo su -s /bin/sh -c "keystone-manage db_sync" keystone
$
$ sudo keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
$ sudo keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
$
$ sudo keystone-manage bootstrap --bootstrap-password adminpassword \
--bootstrap-admin-url http://controller:35357/v3/ \
--bootstrap-internal-url http://controller:5000/v3/ \
--bootstrap-public-url http://controller:5000/v3/ \
--bootstrap-region-id RegionOne
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (12) มีการกำหนดค่าอะไรใน apache2.conf  </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/keystone.html">Install Keystone</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/apache2.conf.nocomment">apache2.conf</a>
</td></tr>  
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/apache2.conf">files/apache2.conf</a> /etc/apache2/apache2.conf
$ sudo service apache2 restart
$ sudo rm -f /var/lib/keystone/keystone.db
</pre>
<p><p>
<i><a id="setendpoints"><h4>3.6 ติดตั้ง OpenStack Service Endpoint </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
กำหนด environment variables เพื่อปฏิบัติการแบบเป็น admin user ซึ่งทำให้ นศ สามารถใช้คำสั่ง "openstack" (ถัดไป)เพื่อสร้าง service endpoint และสร้าง user ใหม่คือ demo user 
<p>
<pre>
$ export OS_USERNAME=admin
$ export OS_PASSWORD=adminpassword
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
$
</pre>
คำสั่งต่อไปนี้สร้าง service project สร้าง demo user สร้าง user role และ associate role นั้นให้กับ demo user   
<p>
<pre>
$ openstack project create --domain default --description "Service Project" service
$ openstack project create --domain default --description "Demo Project" demo
$ openstack user create --domain default --password demopassword demo
$ openstack role create user
$ openstack role add --project demo --user demo user
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (13) มีการกำหนดค่าอะไรใน keystone-paste.conf (คำถามนี้อาจถูกลบออกเนื่องจากบรรทัดถัดไปอาจ Deprecate)</td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/keystone-verify.html">Verify Keystone</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/keystone-paste.ini.nocomment">keystone-paste.ini</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/keystone-paste.ini">files/keystone-paste.ini</a> /etc/keystone/keystone-paste.ini
$
</pre>
คำสั่งสองคำสั่งถัดไป ทดสอบว่า keystone ทำงานถูกต้อง โดยใช้สองคำสั่งถัดไปเรียกดู token สำหรับ admin และ demo users
<p>
<pre>
$ unset OS_AUTH_URL OS_PASSWORD
$ 
$ openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin --os-password adminpassword token issue
$
$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo --os-password demopassword token issue
$
</pre>
ให้ นศ สร้างไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/admin-openrc.sh">./admin-openrc.sh</a> และทดสอบว่าสามารถเรียกใช้คำสั่ง openstack CLI ได้
<p>
<pre>
$ vi <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/admin-openrc.sh">./admin-openrc.sh</a>
$ source ./admin-openrc.sh
$ openstack token issue
</pre>
<p><p>
<i><a id="installglance"><h4>3.7 ติดตั้ง glance </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
สร้าง database สำหรับ glance
<p>
<pre>
$ sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE glance;"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';"
</pre>
สร้าง endpoint สำหรับติดต่อ glance service
<p>
<pre>
$ source ./admin-openrc.sh
$
$ openstack user create --domain default --password GLANCE_PASS glance
$ openstack role add --project service --user glance admin
$ openstack service create --name glance \
  --description "OpenStack Image" image
$
$ openstack endpoint create --region RegionOne \
  image public http://controller:9292
$ openstack endpoint create --region RegionOne \
  image internal http://controller:9292
$ openstack endpoint create --region RegionOne \
  image admin http://controller:9292
$  
</pre>
ติดตั้ง glance-registry และ glance-api
<p>
<pre>
$ sudo apt-get -y install glance 
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (14) มีการกำหนดค่าอะไรใน glance-api.conf และ glance-registry.conf </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/glance.html">Install Glance</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/glance-api.conf.nocomment">glance-api.conf</a>, 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/glance-registry.conf.nocomment">glance-api.conf</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/glance-api.conf">files/glance-api.conf</a> /etc/glance/glance-api.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/glance-registry.conf">files/glance-registry.conf</a> /etc/glance/glance-registry.conf
$
$ sudo su -s /bin/sh -c "glance-manage db_sync" glance
$
$ sudo service glance-registry restart
$ sudo service glance-api restart
</pre>
ทดสอบ glance (upload cirros image เข้า glance)
<p>
<pre>
$ source ./admin-openrc.sh
$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img
$
$ openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
$
$ openstack image list
$ rm cirros-0.3.5-x86_64-disk.img
</pre>
<p><p>
<i><a id="installnova"><h4>3.8 ติดตั้ง nova </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE nova_api;"
$ sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE nova;"
$ sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE nova_cell0;"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';"
$ sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';"
</pre>
สร้าง nova service endpoint
<p>
<pre>
$ source ./admin-openrc.sh
$
$ openstack user create --domain default --password NOVA_PASS nova
$ openstack role add --project service --user nova admin
$ openstack service create --name nova \
  --description "OpenStack Compute" compute
$ openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1/
$ openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1/
$ openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1/
$
$ openstack user create --domain default --password NOVA_PASS placement
$ openstack role add --project service --user placement admin
$ openstack service create --name placement \
  --description "Placement API" placement
$ openstack endpoint create --region RegionOne \
  placement public http://controller:8778
$ openstack endpoint create --region RegionOne \
  placement internal http://controller:8778
$ openstack endpoint create --region RegionOne \
  placement admin http://controller:8778
$
</pre>
ติดตั้ง nova software components ได้แก่ nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler nova-placement-api
<p>
<pre>
$ sudo apt-get -y install nova-api nova-conductor nova-consoleauth \
  nova-novncproxy nova-scheduler nova-placement-api
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (15) มีการกำหนดค่าอะไรใน nova.conf </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/nova.html">Install Nova</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/nova.conf.nocomment">nova.conf</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nova.conf">files/nova.conf</a> /etc/nova/nova.conf
$
$ sudo su -s /bin/sh -c "nova-manage api_db sync" nova
$ sudo su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
$ sudo su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
$ sudo su -s /bin/sh -c "nova-manage db sync" nova
$
$ sudo nova-manage cell_v2 list_cells
$
$ sudo service nova-api restart
$ sudo service nova-consoleauth restart
$ sudo service nova-scheduler restart
$ sudo service nova-conductor restart
$ sudo service nova-novncproxy restart
</pre>
<p><p>
<b>เครื่อง compute</b>
<p><p>
<pre>
$ sudo apt-get -y install nova-compute
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (16) มีการกำหนดค่าอะไรใน nova.conf nova-compute.conf (บนเครื่อง compute)</td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/nova.html">Install Nova</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/nova.conf.nocomment">nova.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/nova-compute.conf.nocomment">nova-compute.conf</a> 
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nova.conf">files/nova.conf</a> /etc/nova/nova.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nova-compute.conf">files/nova-compute.conf</a> /etc/nova/nova-compute.conf
$
$ sudo service nova-compute restart
</pre>
<p><p>
<b>เครื่อง compute1</b>
<p><p>
<pre>
$ sudo apt-get -y install nova-compute
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (17) มีการกำหนดค่าอะไรใน nova.conf nova-compute.conf (บนเครื่อง compute)</td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/nova.html">Install Nova</a></td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/nova.conf.nocomment">nova.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/nova-compute.conf.nocomment">nova-compute.conf</a> 
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nova.conf">files/nova.conf</a> /etc/nova/nova.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nova-compute.conf">files/nova-compute.conf</a> /etc/nova/nova-compute.conf
$
$ sudo service nova-compute restart
</pre>
<p><p>
<b>เครื่อง controller</b>
<p><p>
discovery หา compute nodes
<pre>
$ source ./admin-openrc.sh
$
$ openstack hypervisor list
$ sudo su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
$
$ openstack compute service list
</pre>
ลอง discovery ถ้ายังไม่เจอ compuet แบะ compute1
<pre>
$ source ./admin-openrc.sh
$
$ openstack hypervisor list
$ sudo su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
</pre>
แสดงรายการของ compute service 
<pre>
$ source ./admin-openrc.sh
$
$ openstack compute service list
$ openstack catalog list
$ openstack image list
$ sudo nova-status upgrade check
</pre>
<p><p>
<i><a id="installneutron"><h4>3.9 ติดตั้ง neutron </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
สร้าง database สำหรับ neutron
<p>
<pre>
sudo mysql -u root -pmysqlpassword -e "CREATE DATABASE neutron;"
sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'NEUTRON_DBPASS';"
sudo mysql -u root -pmysqlpassword -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'NEUTRON_DBPASS';"
</pre>
สร้าง service endpoints สำหรับ neutron
<p>
<pre>
$ source ./admin-openrc.sh
$
$ openstack user create --domain default --password NEUTRON_PASS neutron
$ openstack role add --project service --user neutron admin
$ openstack service create --name neutron \
  --description "OpenStack Networking" network
$ openstack endpoint create --region RegionOne \
  network public http://controller:9696
$ openstack endpoint create --region RegionOne \
  network internal http://controller:9696
$ openstack endpoint create --region RegionOne \
  network admin http://controller:9696
$
</pre>
ติดตั้ง neutron-server neutron-plugin-ml2 python-neutronclient และ openvswitch-switch
<p>
<pre>
$ sudo apt-get -y install neutron-server neutron-plugin-ml2 python-neutronclient
$ sudo apt-get -y install openvswitch-switch
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (18) มีการกำหนดค่าอะไรใน neutron.conf ml2_conf.ini nova.conf </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/neutron.conf.nocomment">neutron.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/ml2_conf.ini.nocomment">ml2_conf.ini</a>,  
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/nova-stage23.conf.nocomment">nova-stage23.conf</a> 
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/neutron.conf">files/neutron.conf</a> /etc/neutron/neutron.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/ml2_conf.ini">files/ml2_conf.ini</a> /etc/neutron/plugins/ml2/ml2_conf.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nova-stage23.conf">files/nova-stage23.conf</a> /etc/nova/nova.conf
$
$ sudo su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
$
$ sudo service nova-api restart
$ sudo service neutron-server restart
</pre>
ทดสอบ neutron ขั้นแรก
<p>
<pre>
$ source ./admin-openrc.sh
$ openstack extension list --network
</pre>
<p><p>
<b>เครื่อง network</b>
<p><p>
<pre>
$ sudo apt-get -y install neutron-plugin-ml2 neutron-l3-agent 
$ sudo apt-get -y install neutron-openvswitch-agent openvswitch-switch
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (19) มีการกำหนดค่าอะไรใน neutron.conf ml2_conf.ini </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/neutron.conf.nocomment">neutron.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/ml2_conf.ini.nocomment">ml2_conf.ini</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/neutron.conf">files/neutron.conf</a> /etc/neutron/neutron.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/ml2_conf.ini">files/ml2_conf.ini</a> /etc/neutron/plugins/ml2/ml2_conf.ini
$
$ sudo service openvswitch-switch start
</pre>
สร้าง provider network และ vlan network 
<p>
<pre>
$ sudo ovs-vsctl add-br br-provider
$ sudo ovs-vsctl add-port br-provider ens6
$ sudo ovs-vsctl add-br br-vlan
$ sudo ovs-vsctl add-port br-vlan ens5
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (20) มีการกำหนดค่าอะไรใน openvswitch_agent.ini l3_agent.ini </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/openvswitch_agent.ini.nocomment">openvswitch_agent.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/l3_agent.ini.nocomment">l3_agent.ini</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/openvswitch_agent.ini">files/openvswitch_agent.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/l3_agent.ini">files/l3_agent.ini</a> /etc/neutron/l3_agent.ini
$
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-l3-agent restart
</pre>
<p><p>
<b>เครื่อง controller</b>
<p><p>
เช็ค neutron
<pre>
$ source ./admin-openrc.sh
$ openstack extension list --network
</pre>
<p><p>
<b>เครื่อง compute</b>
<p><p>
<pre>
$ sudo apt-get -y install neutron-plugin-ml2 neutron-openvswitch-agent openvswitch-switch 
$ sudo apt-get -y install neutron-dhcp-agent neutron-metadata-agent 
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (21) มีการกำหนดค่าอะไรใน neutron.conf ml2_conf.ini openvswitch_agent.ini dhcp_agent.ini metadata_agent.ini </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/neutron.conf.nocomment">neutron.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/ml2_conf.ini.nocomment">ml2_conf.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/openvswitch_agent.ini.nocomment">openvswitch_agent.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/dhcp_agent.ini.nocomment">dhcp_agent.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/metadata_agent.ini.nocomment">metadata_agent.ini</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/neutron.conf">files/neutron.conf</a> /etc/neutron/neutron.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/ml2_conf.ini">files/ml2_conf.ini</a> /etc/neutron/plugins/ml2/ml2_conf.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/openvswitch_agent.ini">files/openvswitch_agent.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/dhcp_agent.ini">files/dhcp_agent.ini</a> /etc/neutron/dhcp_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/metadata_agent.ini">files/metadata_agent.ini</a> /etc/neutron/metadata_agent.ini
$
$ sudo service openvswitch-switch start
$
</pre>
สร้าง provide bridge และ vlan bridge บน compute node
<pre>
$ sudo ovs-vsctl add-br br-provider
$ sudo ovs-vsctl add-port br-provider ens6
$ sudo ovs-vsctl add-br br-vlan
$ sudo ovs-vsctl add-port br-vlan ens5
$ 
$ sudo service openvswitch-switch restart
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (22) มีการกำหนดค่าอะไรใน nova.conf </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/nova-stage29.conf.nocomment">nova-stage29.conf</a>
</td></tr> 
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nova-stage29.conf">files/nova-stage29.conf</a> /etc/nova/nova.conf
$
$ sudo service nova-compute restart
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-metadata-agent restart
$ sudo service neutron-dhcp-agent restart
</pre>
<p><p>
<b>เครื่อง compute1</b>
<p><p>
<pre>
$ sudo apt-get -y install neutron-plugin-ml2 neutron-openvswitch-agent openvswitch-switch 
$ sudo apt-get -y install neutron-dhcp-agent neutron-metadata-agent 
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (23) มีการกำหนดค่าอะไรใน neutron.conf ml2_conf.ini openvswitch_agent.ini dhcp_agent.ini metadata_agent.ini </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/neutron.conf.nocomment">neutron.conf</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/ml2_conf.ini.nocomment">ml2_conf.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/openvswitch_agent.ini.nocomment">openvswitch_agent.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/dhcp_agent.ini.nocomment">dhcp_agent.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/metadata_agent.ini.nocomment">metadata_agent.ini</a>
</td></tr> 
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/neutron.conf">files/neutron.conf</a> /etc/neutron/neutron.conf
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/ml2_conf.ini">files/ml2_conf.ini</a> /etc/neutron/plugins/ml2/ml2_conf.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/openvswitch_agent.ini">files/openvswitch_agent.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/dhcp_agent.ini">files/dhcp_agent.ini</a> /etc/neutron/dhcp_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/metadata_agent.ini">files/metadata_agent.ini</a> /etc/neutron/metadata_agent.ini
$
$ sudo service openvswitch-switch start
$
</pre>
สร้าง provide bridge และ vlan bridge บน compute node
<pre>
$ sudo ovs-vsctl add-br br-provider
$ sudo ovs-vsctl add-port br-provider ens6
$ sudo ovs-vsctl add-br br-vlan
$ sudo ovs-vsctl add-port br-vlan ens5
$ 
$ sudo service openvswitch-switch restart
$
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (24) มีการกำหนดค่าอะไรใน nova.conf </td></tr>
<tr><td>Hint: <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/neutron.html">Neutron Basic Install</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs.html">Neutron OVS Mechanisms</a>,<a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-provider.html">Neutron OVS provider</a>, <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-selfservice.html">Neutron OVS Self-service</a>  </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/nova-stage29.conf.nocomment">nova-stage29.conf</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nova-stage29.conf">files/nova-stage29.conf</a> /etc/nova/nova.conf
$
$ sudo service nova-compute restart
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-metadata-agent restart
$ sudo service neutron-dhcp-agent restart
</pre>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<pre>
$ source ./admin-openrc.sh
$ openstack extension list --network
</pre>
<p><p>
<i><a id="installhorizon"><h4>3.10 ติดตั้ง horizon </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
ถ้าเครื่องที่ นศ ติดตั้งมี RAM น้อยหรือ CPU ไม่แรง ผมแนะนำให้ไม่ต้องติดตั้ง horizon และใช้ OpenStack CLI เป็นหลัก 
<p>
<pre>
$ sudo apt-get -y install openstack-dashboard
</pre>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (25) มีการกำหนดค่าอะไรใน local_setting.py </td></tr>
<tr><td>Hint : See <a href="https://docs.openstack.org/ocata/install-guide-ubuntu/horizon.html">Install Horizon</a></td></tr>
</table>
<p><pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/local_settings.py">files/local_settings.py</a> /etc/openstack-dashboard/local_settings.py
$ sudo service apache2 reload
$
$ sudo chown www-data:www-data /var/lib/openstack-dashboard/secret_key
$ sudo service apache2 reload
</pre>
<p><p>
<i><a id="setdvr"><h4>3.11 เปลี่ยน neutron ให้เป็นแบบ Distributed Virtual Routers (DVR)</h4></a></i>
<p><p>
หลังจาก install neutron นศ จะได้ neutron ที่ทำงาน classic openvswitch การกำหนดค่า configuration ต่อไปนี้จะเปลี่ยน neutron ให้เป็นแบบ DVR
<p><p>
<b>เครื่อง controller</b>
<p><p>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (26) มีการกำหนดค่าอะไรใน neutron.conf </td></tr>
<tr><td>Hint: See <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-ha-dvr.html#deploy-ovs-ha-dvr">Openvswitch HA with DVR</a> </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/nocomment/neutron_stage33.conf.nocomment">neutron_stage33.conf</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/controller/files/neutron_stage33.conf">files/neutron_stage33.conf</a> /etc/neutron/neutron.conf
$ sudo service neutron-server restart
</pre>
<p><p>
<b>เครื่อง network</b>
<p><p>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (27) มีการกำหนดค่าอะไรใน openvswitch_agent.ini l3_agent.ini </td></tr>
<tr><td>Hint: See <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-ha-dvr.html#deploy-ovs-ha-dvr">Openvswitch HA with DVR</a> </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/openvswitch_agent_stage34.ini.nocomment">openvswitch_agent_stage34.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/nocomment/l3_agent_stage34.ini.nocomment">l3_agent_stage34.ini</a>
</td></tr>
</table>
<pre>
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/openvswitch_agent_stage34.ini">files/openvswitch_agent_stage34.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/network/files/l3_agent_stage34.ini">files/l3_agent_stage34.ini</a> /etc/neutron/l3_agent.ini
$
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-l3-agent restart
</pre>
<p><p>
<b>เครื่อง compute</b>
<p><p>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (28) มีการกำหนดค่าอะไรใน openvswitch_agent.ini l3_agent.ini </td></tr>
<tr><td>Hint: See <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-ha-dvr.html#deploy-ovs-ha-dvr">Openvswitch HA with DVR</a> </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/openvswitch_agent_stage35.ini.nocomment">openvswitch_agent_stage35.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/nocomment/l3_agent_stage35.ini.nocomment">l3_agent_stage35.ini</a>
</td></tr>
</table>
<pre>
$ sudo apt-get -y install neutron-l3-agent 
$
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/openvswitch_agent_stage35.ini">files/openvswitch_agent_stage35.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute/files/l3_agent_stage35.ini">files/l3_agent_stage35.ini</a> /etc/neutron/l3_agent.ini
$
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-l3-agent restart
</pre>
<p><p>
<b>เครื่อง compute1</b>
<p><p>
<table>
<tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (29) มีการกำหนดค่าอะไรใน openvswitch_agent.ini l3_agent.ini </td></tr>
<tr><td>Hint: See <a href="https://docs.openstack.org/ocata/networking-guide/deploy-ovs-ha-dvr.html#deploy-ovs-ha-dvr">Openvswitch HA with DVR</a> </td></tr>
<tr><td>ตัวอย่าง configuration files ที่ลบ comments ออก: 
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/openvswitch_agent_stage36.ini.nocomment">openvswitch_agent_stage36.ini</a>,
<a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/nocomment/l3_agent_stage36.ini.nocomment">l3_agent_stage36.ini</a>
</td></tr>
</table>
<pre>
$ sudo apt-get -y install neutron-l3-agent 
$
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/openvswitch_agent_stage36.ini">files/openvswitch_agent_stage36.ini</a> /etc/neutron/plugins/ml2/openvswitch_agent.ini
$ sudo cp <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.OPSInstaller/compute1/files/l3_agent_stage36.ini">files/l3_agent_stage36.ini</a> /etc/neutron/l3_agent.ini
$
$ sudo service openvswitch-switch restart
$ sudo service neutron-openvswitch-agent restart
$ sudo service neutron-l3-agent restart
</pre>
<p><p>
<i><a id="testnetwork"><h4>3.12 สร้าง network เริ่มต้น </h4></a></i>
<p><p>
<b>เครื่อง controller</b>
<p><p>
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (30) ขอให้ นศ อธิบายความหมายของ CLI ข้างล่าง </td></tr></table>
<pre>
$ source ./admin-openrc.sh
$
$ openstack network create --share --provider-physical-network provider \
 --provider-network-type flat provider1
$
$ openstack subnet create --subnet-range 10.0.10.0/24 --gateway 10.0.10.11 \
 --network provider1 --allocation-pool start=10.0.10.100,end=10.0.10.200 \
 --dns-nameserver 8.8.4.4 provider1-v4
$
$ openstack security group rule create --proto icmp default
$
$ openstack security group rule create --proto tcp --dst-port 22 default
$
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
$
$ openstack flavor create --id 1 --vcpus 1 --ram 512 --disk 1 m1.mini
$
$ ping -c 4 10.0.10.100
</pre>
หลังจากรันคำสั่ง ping คำสั่งนี้ควรจะทำให้ นศ ping 10.0.10.100 ได้ ถ้าไม่ได้แสดงว่าการติดตั้ง neutron ผิดพลาด
<p><p>
ถ้าผ่านการ ping มาได้อันดับถัดไปจะให้ นศ สร้าง network สำหรับ admin user
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (31) ขอให้ นศ อธิบายความหมายของ CLI ข้างล่าง </td></tr></table>
<pre>
$ openstack network set --external provider1
$
$ openstack network create selfservice1
$
$ openstack subnet create --subnet-range 192.0.2.0/24 \
  --network selfservice1 --dns-nameserver 8.8.4.4 selfservice1-v4
$
$ openstack router create router1
$
$ openstack router add subnet router1 selfservice1-v4
$
$ openstack router set router1 --external-gateway provider1
$
$ openstack floating ip create provider1
$
</pre>
<p><p>
<i><a id="testopenstack"><h4>3.13 ทดสอบ OpenStack</h4></a></i>
<p><p>
<table><tr><td>คำถาม <b>PROJECT</b> วิชา คพ. 449: (32)
ในอันดับถัดไป นศ จะต้องใช้ horizon เข้าไปสร้าง "cirros" vm instance 2 เครื่อง (สมมุติว่าชื่อ cirros1 และ cirros2) บน selfservice1-v4 network และทำตามคำสั่งหรือตอบคำถามต่อไปนี้
<p><p>
<ul>
<li> กำหนดค่า floating IP ให้กับ cirros vm เครื่องหนึ่ง (สมมุติว่าชื่อ cirros1)
<li> ใช้ ssh จากเครื่อง controller เข้าสู่ floatinf IP นั้น
<li> หลังจาก login เข้า cirros1 แล้วให้ ping www.google.com
<li> ให้ใช้ ifconfig ดูว่าลักษณะของ network interface เป็นอย่างไร มีค่า MTU เท่าไรแตกต่างจากบนเครื่อง controller network compute และ compute1 หรือไม่ และทำไมถึงเกิดความแตกต่างขึ้น
<li> ให้ นศ ssh จาก cirros1 ไปยัง cirros2 
<li> จาก cirros2 ให้ ping www.google.com ทำได้หรือไม่
<li> ให้ poweroff เครื่อง network node 
<li> จากเครื่อง cirros2  นศ ping google.com ได้หรือไม่
<li> ให้ exit จาก cirros2 กลับมาที่ cirros1 
<li> ให้ลอง ping google.com ทำได้หรือไม่ เพราะอะไร 
<li> ให้ boot เครื่อง network node อีกครั้งหนึ่ง
<li> ให้ลอง ssh จาก cirros1 ไปยัง cirros2 อีกครั้ง 
<li> ให้ลอง ping google.com ทำได้หรือไม่ เพราะอะไร 
</ul>
ดูตัวอย่างได้ที่ <a href="https://www.youtube.com/watch?v=uXjlmfOvFCs&index=10&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ&t=1484s">youtube video ทดสอบ OpenStack</a>
</td></tr></table>
<p><p>
<a id="part4"> 
<h3>ส่วนที่ 4: สรุป</h3></a>
<p><p>
OpenStack ประกอบไปด้วย software component ที่ออกแบบมาให้รองรับธรรมชาติของ distributed systems ที่ประสานงานกันแบบหลวม (หรือ loosely coupling) เมื่อคอมพิวเตอร์ ซอฟต์แวร์ หรือ network ในสภาพแวดล้อมของ OpenStack มีปัญหา software component ที่ไม่ได้รับผลกระทบก็จะยังคงทำงานต่อได้ และหลังจากระบบที่มีปัญหาได้รับการซ่อมแซม ระบบก็จะกลับมาปฏิบัติงานได้ตามปกติ 
<p><p>
สำหรับ system admin ของระบบ OpenStack การเรียนรู้และเข้าใจส่วนประกอบของ OpenStack เป็นเรื่องสำคัญและไม่ไช่เรื่องที่ยากจนเกินไป ทั้งนี้เพราะ OpenStack เป็นระบบเปิดและสร้างด้วย Open Source software ทั้งหมด 
<p><p>Sys admin ไม่จำเป็นต้องอ่าน code ของ OpenStack (แต่ถ้าทำก็จะดีมาก) แต่ควรจะเข้าใจส่วนประกอบต่างๆว่ามี agents/services อะไรบ้างและเข้าใจการกำหนดค่า configuration ต่างๆใน configuration files ว่ามีความสามารถอะไรบ้างที่นำมาใช้งานได้และพารามีเตอร์ต่างๆความหมายอย่างไร 
<p><p>
การศึกษา OpenStack โดยได้ติดตั้งด้วยมือและลองผิดลองถูกเป็นวิธีการที่ดีในการเรียนรู้ ด้วยเหตุนี้ผมจึงอยากให้ นศ ลองสร้างระบบสำหรับทดสอบขึ้นไม่ว่าจะเป็นเครื่องจริงหรือ vm และทดลองติดตั้งด้วยมือโดยศึกษาเวปเพจนี้ ประกอบกับข้อมูลจาก Official OpenStack Installation Manual 
<p><p>
การติดตั้ง OpenStack มีรูปแบบและวิธีการหลากหลายในปัจจุบัน ผู้ใช้จะติดตั้งบน node เดียว 2 nodes หรือ multi-node แล้วแต่ resources ที่มี และสามารถเลือก component ได้ตามที่ต้องการใช้งาน วิธีการติดตั้งในปัจจุบันมีหลายแบบ อาทิเช่น การติดตั้งด้วยมือ การติดตั้งแบบใช้ scripts (เช่นในเวปนี้ หรือ RDO ของ Redhat) และการติดตั้งโดยใช้ container และ Ansible (เช่น OpenStack Kolla) 
<p><p>
อย่างไรก็ตาม ไม่ว่าจะติดตั้งแบบใด sys admin ของระบบ OpenStack ควรมี (1) ความเข้าใจการทำงานของ OpenStack components (2) ความคุ้นเคยกับ configuration files เพื่อให้สามารถดูแลและปรับเปลี่ยนระบบได้อย่างมีประสิทธิภาพในระยะยาว (3) รู้จักคำสั่งที่จำเป็นในการ start/stop/restart OpenStack services (4) คุ้นเคยกับ OpenStack CLI หรือ API 
<p><p> 
การเรียนรู้ที่ดีที่สุดคือการลองทำด้วยตนเอง ผมหวังว่า นศ จะได้เรียนรู้ด้วยตนเองจากการลองติดตั้งและพยายามตอบคำถาม PROJECT วิชา คพ. 449 ใน web นี้ 
<p><p>
<h3>อ้างอิง</h3>
<p><p>
[1] http://docs.openstack.org/ <br>
[2] https://docs.openstack.org/ocata/install/ubuntu-services.html <br>
[3] https://docs.openstack.org/ocata/networking-guide/ <br>
[4] <a href="https://www.youtube.com/playlist?list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">OpenStack ocata installation youtube video playlist</a>
=======
# openstack-queens-installer
>>>>>>> 52c01fb39622b69bc429df14d50b01e2e45a48ed
