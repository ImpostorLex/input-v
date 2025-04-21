---
{"dg-publish":true,"permalink":"/cards/trend-micro-vision-one/enabling-xdr-sensors-on-the-network/"}
---

[[cards/trend-micro-vision-one/Vision One for Administrators\|Vision One for Administrators]]

Trend Micro Vision One is capable of monitoring the network through Trend Micro Deep Discovery Inspector and Trend Micro TippingPoint.
## Provisioning a new Deep Discovery Inspector Device
---
This can be accessed in Trend Micro Vision One console:

1. Click the **Network Inventory** app. The first time you access this app, it will prompt to **Select the product or service to use to connect network sensors**. Select **Network Inventory** as this will provide the latest cloud-based version of the device.

2. Once the Network Inventory service has deployed, click **+ Connect Network Sensor** to display the **Connect Network Sensor** configuration window. Select **New Deep Discovery Inspector** if you plan to deploy the virtual device in your on-premises environment and click to **Download Disc Image**.

![Enabling XDR Sensors on the Network.png|450](/img/user/cards/trend-micro-vision-one/images/Enabling%20XDR%20Sensors%20on%20the%20Network.png)

3. View the full steps for creating the Deep Discovery Inspector virtual machine on:

- [VMware ESXi, Microsoft Hyper-V or CentOS KVM](https://docs.trendmicro.com/en-us/enterprise/trend-micro-vision-one/networksecurityopera/network-inventory/connecting-and-deplo/deploying-a-deep-dis.aspx)
- [AWS](https://docs.trendmicro.com/en-us/enterprise/trend-micro-vision-one/networksecurityopera/network-inventory/connecting-and-deplo/deploying-a-deep-dis_001.aspx) 

Once the virtual device has been created, log into the Deep Discovery Inspector console, and click **Administration**. Under **Integrated Products/Services** in the left-hand frame, click **Trend Micro Vision One**.

![Enabling XDR Sensors on the Network-1.png|450](/img/user/cards/trend-micro-vision-one/Enabling%20XDR%20Sensors%20on%20the%20Network-1.png)
### Connecting the Deep Discovery Inspector Device to the Service Gateway
---
1. In the Trend Micro Vision One console, click the **Network Inventory** app.
2. Select one or more network sensors, and then click **Connect Service Gateway**. The Connect Service Gateway panel appears.
3. Select a Service Gateway and click **Connect**.



