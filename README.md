<h1 aling="center">CAN Transceivers on Jetson AGX Xavier</h1>

<h1 aling="center">Description</h2>

<p align="center">In this repository you will find a simplified method of installing CAN transceivers on the Jetson AGX Xavier board. After much research and study, I created these steps in the way I would have liked to find when I was doing the installation.</p>

![Badge](https://img.shields.io/static/v1?label=Ubuntu&message=18.04&color=orange&style=%3CSTYLE%3E&logo=linux)    ![Badge](https://img.shields.io/static/v1?label=Nvidia&message=Jetson%20AGX%20Xavier&color=green&style=%3CSTYLE%3E&logo=Nvidia)    ![Badge](https://img.shields.io/static/v1?label=Necessary%20Knowledget&message=Shell%20Script&color=red&style=%3CSTYLE%3E&logo=%3CLOGO%3E)      ![Badge](https://img.shields.io/static/v1?label=Necessary%20Knowledget&message=CAN%20Protocol&color=yellow&style=%3CSTYLE%3E&logo=%3CLOGO%3E)

------

## Enabling CAN

﻿Leave the board in "full" mode -> MODE 30W ALL

This mode can be found in the upper right corner of the screen, in the power mode options, the image below shows where these options are.
</br>

<p align="center">
  <img width="460" height="300" src="/img/image5_9.png">
</p>
</br>

To perform the installation and configuration of the CAN transceivers, you must enter root mode on the AGX Xavier board to perform the modifications with privileges. Root mode is accessed by the command:



```sudo su    ```



Confirm that the channels are not active with the ```ifconfig``` command, which will return all the set network ports of the card. At this point no can0 or can1 should appear.﻿﻿﻿



------



## ﻿Connect the CAN Transceivers to the board

AGX Xavier's doors are referenced by the image below or access the [site](https://www.jetsonhacks.com/nvidia-jetson-agx-xavier-gpio-header-pinout/) to see the table in better quality.


<p align="center">
  <img width="460" height="300" src="/img/image.png">
</p>
</br>


Connect the transceivers according to the image below, tx to tx and rx to rx. I know it gets confusing when I say tx to tx, but in short, connect CAN0_DIN to the rx of the transceiver and CAN0_DOUT to the tx of the transceiver. The can1 connections follow the same logic. VCC of the transceiver to the 3.3V pin of the board, as well as GND of the transceiver to GND of the board.



![CAN Transceivers conectors](https://user-images.githubusercontent.com/64169072/130858690-c08a59e9-c07c-4f21-9daa-5db7a2fffe88.png)



Check the status of the can-controllers by command:

``` cat /proc/device-tree/mttcan\@c310000/status```

```@c310000 - endereço do can-controller 1```

```@c320000 - endereço do can-controller 2```

The return from the command to be "okay".



------



## Installing Busybox

Install the busybox tool to configure the pinmux values, which is the pin reference on Xavier's bus. The installation is done by the command:

```apt-get install busybox```



----



#### Setting the Addresses



After the installation, it is time to configure the addresses. To configure the addresses, run the commands below and check that the values are according to the specific card, this information is located in the table below in the Pinmux property.



|                            Property | Jetson Xavier NX & Jetson TX2 NX       | Jetson AGX Xavier series                                     | Jetson TX2                                                   |
| ----------------------------------: | :------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|               Number of controllers | 1                                      | 2                                                            | 2                                                            |
|             Controller base address | mttcan@c310000                         | mttcan@c310000  mttcan@c320000                               | mttcan@c310000  mttcan@c320000                               |
|        Pins on Jetson carrier Board | J17;<br />CAN_RX: [1]<br />CAN_TX: [2] | J30 (40-pin header):<br />CAN0_DIN: [29]<br />CAN0_DOUT: [31]<br />CAN1_DIN: [37]<br />CAN1_DOUT: [33] | J26 (30-pin header):<br />CAN0_RX: [5]<br />CAN0_TX: [7]<br />CAN1_RX: [15]<br />CAN1_TX: [17] |
|                          **Pinmux** |                                        |                                                              |                                                              |
|  can0_din<br />Address:<br />Value: | <br/>0x0c303020<br />0x0458            | <br/>0x0c303018<br />0x0458                                  | <br/>0x0c303020<br />0x0458                                  |
| can0_dout<br />Address:<br />Value: | <br/>0x0c303018<br />0x0400            | <br/>0x0c303010<br />0x0400                                  | <br/>0x0c303018<br />0x0400                                  |
|  can1_din<br />Address:<br />Value: | n/a                                    | <br/>0x0c303008<br />0x0458                                  | <br/>0x0c303010<br />0x0458                                  |
| can1_dout<br />Address:<br />Value: | n/a                                    | <br/>0x0c303000<br />0x0400                                  | <br/>0x0c303008<br />0x0400                                  |
|           Default pin configuration | SFIO: CAN functionality                | GPIO                                                         | SFIO: CAN functionality                                      |



﻿﻿Then run this command ```busybox devmem 0x0c303020 w 0x458``` for each port, notice that there are four different ones.
﻿

```﻿﻿0x0c303020``` - address taken from the table for the TX2 and Xavier NX card, remember that we work with the Xavier AGX, so the values are different.

```﻿﻿0x458``` - value

The table can also be found from the documentation in [CAN (Controller Area Network)](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_can.html). Enable the pins to use CAN through the Jetson-IO tool interface, please read the [documentation](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_io.html#wwpID0E0ZE0HA) for more information.



﻿Run the command below to open the interface.

```sudo /opt/nvidia/jetson-io/jetson-io.py```



The interface will be opened and you will see the screen below.



![Jetson-IO Tool](https://user-images.githubusercontent.com/64169072/130858841-db16dc0f-ca5d-4f0c-b529-36d161671113.png)



Use the up, down and enter arrows to navigate between options. Select "Configure 40-pin expansion header" to modify the pins. Check if the screen below was opened. ﻿﻿



![Jetson-IO Tool](https://user-images.githubusercontent.com/64169072/130858884-9f1c881b-839c-44fc-8e5f-8f8d5a16c316.png)



In this screen you will select with up/down and next enter to define the pin to mark on can0 and can1 like the image below.



![Jetson-IO Tool](https://user-images.githubusercontent.com/64169072/130858918-e7b73b74-93db-4f09-bb02-866010c9798a.png)



﻿﻿Go back to the main screen with the back command, check if different options appear and if the CAN inputs and outputs on the bus are listed like the image below.



![Jetson-IO Tool](https://user-images.githubusercontent.com/64169072/130858941-7ce206c9-4bde-4e3b-8abf-2aad82749d5c.png)



In the options, select "Save and reboot to reconfigure pins", this option will leave the pins pre-set every time the board is booted and will reboot the board.



![Jetson-IO Tool](https://user-images.githubusercontent.com/64169072/130858970-ab6e6a5a-d5e3-4bd9-91c1-e77ffc67e110.png)



After the board is completely connected, perform the next steps....



---



## Enabling Kernel Drivers

The next step is to load the Kernel drivers and in the following order, follow the order described below:



1. ﻿Insert the CAN BUS subsystem support module:
      ﻿```modprobe can```

2. Insert the raw CAN protocol module (CAN-ID filtering):

   ``modprobe can_raw``

3. Add real CAN interface support (for Jetson, mttcan):

   ``` modprobe mttcan```



---



## CAN network management

To use the network, you need to manage the network. This management is done by the commands below, one for can0 and one for can1.
﻿
﻿﻿These example commands set the network interface to use FD (Flexible Data) mode with a bus bit rate of 500 kbps and a data bit rate of 1 Mbps, enter the commands:



```ip link set can0 up type can bitrate 500000 dbitrate 1000000 berr-reporting on fd on```

```ip link set can1 up type can bitrate 500000 dbitrate 1000000 berr-reporting on fd on```



Verify that the ports appeared by the ```ifconfig``` command, it should return the CAN ports in the <UP, RUNNIG< NOARP> state. We can validate this by looking at the image below:



![Verify_ifconfig](https://user-images.githubusercontent.com/64169072/130859024-11600c1f-6dee-46b6-b276-d4448d212f22.png)



---



## Installing the can-utils 

It is necessary to install the **can-utils** library to use some commands like ***cansend*** (sends message over the bus) and ***candump*** (monitors and displays the message).

Now install the CAN utility library by the command below:

```apt-get install can-utils```

---

After performing these steps, it is time to test the CAN-Bus. To send data from can0/can1, use the command below, but define what the channel will be:



```cansend can0 123#abcdabcd``` - single message

```cangen -v can0``` - automatic random messages



Finally, to monitor, open another terminal and run the command and select the channel to monitor:



```candump can1﻿﻿```



----



For more details, I recommend reading the documentation listed here, which are the references that I used to develop these steps.

[CAN (Controller Area Network)](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra Linux Driver Package Development Guide/hw_setup_jetson_can.html)

[NVIDIA Jetson AGX Xavier GPIO Header Pinout](https://www.jetsonhacks.com/nvidia-jetson-agx-xavier-gpio-header-pinout/)

[Pinmux Settings](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-325/index.html#page/Tegra Linux Driver Package Development Guide/mb1_platform_config_xavier.html#wwpID0E0240HA)

[Pinmux Forum](https://forums.developer.nvidia.com/t/change-boot-pinmux-settings-without-full-flash/170392)

[Jetson-IO Tool](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra Linux Driver Package Development Guide/hw_setup_jetson_io.html#wwpID0E0ZE0HA)



---

Natanael Vitorino 

