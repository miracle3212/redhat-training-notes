Overview
========

Course info
-----------
* [Day 1: Units 1, 2, 3](day1.md)
* [Day 2: Units 5, 6, 7, 8, 9](day2.md)
* [Day 3: Units 10, 11, 12, 13](day3.md)
* [Day 4: Units 14, 15, 16](day4.md)
* Day 5: Exam

NOT on this exam:
* Unit 9 (it's on the RHCE exam)
* Encryption part of Unit 10
* Units 15 and 16

Pay special attention to:
* Unit 1
* Unit 13

Without them it won't be possible to even access the exam

70% is needed to pass the exam.

Machines in the network
-----------------------
<table>
  <tr>
    <th>Name</th><th>IP address</th>
  </tr>
  <tr>
    <td>instructor</td><td>192.168.0.254</td>
  </tr>
  <tr>
    <td>demo</td><td>192.168.0.250</td>
  </tr>
  <tr>
    <td>desktopX</td><td>192.168.0.X</td>
  </tr>
  <tr>
    <td>serverX</td><td>192.168.0.100+X</td>
  </tr>
</table>

Users:
* root/redhat
* student/student

General
-------
serverX is a vm on desktopX
* ```virt-viewer -c qemu:///system vserver``` to access serverX from desktopX
* ```su -``` - always use su with -, it sets the environment correctly
* ```ssh -X``` use ssh with -X to be able to launch graphical application remotely
* if the vm dies, run ```lab-resetvm``` to restore it from snapshot
* 


Good links
----------
http://www.stephenlang.net/tag/rhcsa/
