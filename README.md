# Borland-AccuRev-StackoverFlow
i try to Exploit recently published  vunluerblity in Borland AccuRev 

http://www.zerodayinitiative.com/advisories/ZDI-15-416/

````
his vulnerability allows remote attackers to execute arbitrary code on 
vulnerable installations of Borland AccuRev. Authentication is not required
to exploit this vulnerability.

The specific flaw exists within the service_startup_doit functionality
of the Reprise License Manager service. The issue lies in the handling 
of the licfile parameter which can result in overflowing a stack-based buffer.
An attacker could leverage this vulnerability to execute code under the context of SYSTEM.
````
there was tow fake information in ZDI report

1. maping between string and function is "service_Setup_doit" not "service_startup_doit"
2. vunlrabe parameter is debuglog not licfile 

RLM come with AccuRev but it can be directly accsible  by flowing link:  http://www.reprisesoftware.com/license_admin_kits/rlm.v11.3BL1-x86_w1.admin.exe

searching service_startup_doit inside rlm.exe dont have result  so i changed it to service_  to get flowling strings

![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/stringInIdapro.png)

ZDI siad vunlrable  function is aacceible remotly so i start read RLM manal to find  out how it work 
RLM have Web interface it start http server on port 5054 , with the help of burpsuite we can view all http parameters is all post or get requests 
after fuzzing Web interface in 2 minite i found our target "licfile" parameter.
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/burpsuite.png)

for checking licfile parameter vunlarblity  with help of BurpSuite i send big string parameter
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/bigbuffer.png)
but rlm.exe send error  response  without crash 
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/httperror.png)
i set breakpoint in Immunity debugger to track how rlm check size of string 
![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/asm-check.png)

based  on above asm code rlm check string size be  less than or egaul to 0x400 so i thout my version is not vulrable but i start fuzz other filds so if you send big  string in debuglog parameter and  if string is less than  0x400 to bypass above check you can overiride ret value in stack  with  notoruis 0x41414141 in rlm.exe so i as said  vunlrable function is debuglog not licfile .

![alt tag](https://raw.githubusercontent.com/Rootkitsmm/Borland-AccuRev-StackoverFlow/master/eip.png)

i check rlm.exe and  it dose not support ASLR 
