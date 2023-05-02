Install OpenVPN Client

MAC: Go into terminal folder in /Applications/Utilities

Type `curl --output ~/Desktop/Tunnelblick.dmg --location`\


![Screen Shot 2022-12-06 at 11.54.24 AM.png](<files/fabcc430-69d4-47e9-bd11-b861a9968f52/Screen Shot 2022-12-06 at 11.54.24 AM.png>)

Login to Leaseweb Customer Portal and go to "Actions" Menu&#x20;

![Screen Shot 2022-12-06 at 11.53.38 AM.png](<files/8d380075-a555-4dad-b383-0d961ebba86e/Screen Shot 2022-12-06 at 11.53.38 AM.png>)

Go to Remote Management and Download the VPN Configuration File for the Server location. Top install simply have Tunnelblick open and double click the downloaded file.&#x20;

![Screen Shot 2023-03-13 at 4.26.25 PM.png](<files/4cc3b9f2-261d-4f0b-a254-e24949eebeda/Screen Shot 2023-03-13 at 4.26.25 PM.png>)

Login to TunnelBlick:

![Screen Shot 2023-03-13 at 4.29.04 PM.png](<files/f8a926df-0525-47c3-8ce0-a5d3e3f987ab/Screen Shot 2023-03-13 at 4.29.04 PM.png>)

Use login info on the Remote Management Page generated:

![Screen Shot 2023-03-13 at 4.31.19 PM.png](<files/f4cd9ce2-c6bc-4e22-9ce4-f3709cb84b40/Screen Shot 2023-03-13 at 4.31.19 PM.png>)

Login to IPMI Interface

![Screen Shot 2023-03-13 at 4.33.33 PM.png](<files/5d869f05-df48-4f14-a4ff-a77e5ea71b8c/Screen Shot 2023-03-13 at 4.33.33 PM.png>)

![Screen Shot 2023-03-13 at 4.33.08 PM.png](<files/9931d45e-b2b6-44f7-8fff-bf1046e5ccf8/Screen Shot 2023-03-13 at 4.33.08 PM.png>)

Go to Configuration → BIOS Settings

![Screen Shot 2023-03-13 at 4.35.59 PM.png](<files/53ccd07a-5b52-4f53-823a-6cc9a849bc17/Screen Shot 2023-03-13 at 4.35.59 PM.png>)

Disable "Logical Processor" or Hyperthreading:

![Screen Shot 2023-03-13 at 4.37.08 PM.png](<files/f46fa2ea-2b22-4c3b-874b-dcf7d27cfa06/Screen Shot 2023-03-13 at 4.37.08 PM.png>)

In System Security change Intel(R)SGX from "Software" to "On"

![Screen Shot 2023-03-13 at 4.41.05 PM.png](<files/5b531d14-790c-47df-9550-26f49718e136/Screen Shot 2023-03-13 at 4.41.05 PM.png>)

\
INSTALL DRIVER UPDATES:\
\
1\. Download the most up to date driver for your machine (for this vulnerability it was 2.11.1):\
<https://www.dell.com/support/home/en-pr/product-support/product/poweredge-r240/drivers>\
2\. Go to Maintenance → System Update

![Screen Shot 2023-03-13 at 5.01.06 PM.png](<files/bb33dfcb-cbbd-4480-af1c-20194aa2d449/Screen Shot 2023-03-13 at 5.01.06 PM.png>)

3.Upload the driver file

![Screen Shot 2023-03-13 at 5.03.21 PM.png](<files/3f998171-ab4c-478a-ad4c-f89f2439ea34/Screen Shot 2023-03-13 at 5.03.21 PM.png>)

\
\
