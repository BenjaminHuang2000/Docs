.. meta::
  :description: Global Transit Network
  :keywords: Transit Network, Transit hub, AWS Global Transit Network, Encrypted Peering, Transitive Peering, Insane mode, Transit Gateway, TGW


===============================================
Insane Mode CloudN Deployment Checklist
===============================================

When Insane Mode is applied to improve encryption performance between on-prem and cloud, you need to deploy Aviatrix hardware appliance CloudN. Making this use case work requires edge router configurations. This document lists the checklist you should follow in 
successfully deploying Insane Mode for hybrid connection. 


Step 1. Deployment  Architecture 
---------------------------------------

The first step is to understand how routing works in this use case, as demonstrated in the diagram below.

|insane_mode_howto| 

The key ideas for this scenario are:

 -  The edge (WAN) router runs a BGP session to VGW (AWS) where the edge router advertises CloudN WAN subnet network and VGW advertises the Transit VPC CIDR.
 -  CloudN LAN interface runs a BGP session to the edge router where the edge router advertises on-prem network address range to CloudN LAN interface.
 -  CloudN WAN interface runs a BGP session to Aviatrix Transit Gateway in the Transit VPC where Aviatrix Transit Gateway advertises all Spoke VPC CIDRs to CloudN and CloudN advertises on-prem network to the Aviatrix Transit Gateway. 

Following are a few common deployment architecture. 

Single Aviatrix CloudN Appliance 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|deployment|

And the sample configuration on an ISR is as follows.

|ISR-sample-config|

Aviatrix CloudN Appliance with HA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|deployment_ha|

Redundant DX Deployment 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|deployment_dual_dx|

Step 2. Pre-deployment Request Form 
------------------------------------

After you understand the deployment architecture and decide to move forward for this deployment, the next step is to fill the `CloudN
Appliance Request Form. <https://s3-us-west-2.amazonaws.com/aviatrix-download/InsaneMode_CloudN_Prep.docx>`_   

Aviatrix support team configures CloudN appliance based on your input in the Request Form, then 
ship the appliance.  Deployment topology for Aviatrix CloudN is as follows:

|InsaneBeta|

The key information in the Request Form that you must fill are explained below. 

=====================  ==================  ===========  ===============  ==================  =====================  =============================================================
CloudN Interface       Private IP Address  Subnet Mask  Default Gateway  Primary DNS Server  Secondary DNS Server   Note
=====================  ==================  ===========  ===============  ==================  =====================  =============================================================
1- WAN                                                  Not Required     Not Required        Not Required           WAN port that connects edge router
2- LAN                                                  Not Required     Not Required        Not Required           LAN port that connects edge router
3- MGMT                                                                                                             Management port for CloudN configuration and software upgrade
4- HPE iLO (optional)                                                    Not Required        Not Required           HP Integrated Lights-Out
=====================  ==================  ===========  ===============  ==================  =====================  =============================================================


2.1 Internet Access
~~~~~~~~~~~~~~~~~~~~~~~~
CloudN appliance does not require public IP address, but the management port requires outbound internet access on the management port for software upgrade. 

2.2 BGP Requirement
~~~~~~~~~~~~~~~~~~~~~~~
BGP is required between LAN port of the appliance and the on-prem router for route propagation.

Step 3. Deployment Check List
-----------------------------------

3.1 Before Powering Up CloudN
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Before powering up CloudN, make sure 
 
 a. CloudN WAN cable, LAN cable and Management cable are properly plugged in to ASR and switches.
 #. Check the interface of ASR to CloudN WAN interface, make sure Proxy ARP is enabled (ip proxy-arp). 
 #. ASR DX (Direct Connect) interface should only advertise CloudN WAN interface subnet network to VGW
 #. ASR LAN (Datacenter facing) interface does not advertise Transit VPC CIDR to datacenter.
 #. ASR to CloudN LAN interface advertises datacenter networks.
 #. VGW is attached to the Transit VPC. 
 #. AWS Transit VPC Route Propagation is enabled. 

3.2 Power up CloudN
~~~~~~~~~~~~~~~~~~~~~~~

After you power up CloudN, first test the CloudN interfaces are alive and connected properly by doing the following tests.  

 a. From ASR,  ping CloudN LAN interface, WAN interface and Mgmt interface.
 #. CloudN mgmt interface can ping Internet (From CloudN clish console)

3.3 Upgrade CloudN to the Latest Software
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 a. Login to the CloudN console. Open a browser console and type: https://CloudN_Mgmt_IP_Address
 #. Login with username "admin" and password "Aviatrix 123#" (You can change the password later)
 #. Upgrade CloudN to the latest.

3.4 Configure Insane Moode
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the Controller in AWS, configure Transit Setup Step 3 to CloudN, make sure to select all the correction options.

.. 

 a. CloudN IP Address is the CloudN WAN IP address
 #. CloudN Neighbor IP Address is the ASR to CloudN LAN interface IP address
 #. After configuration, download the configure file and import to CloudN.
 #. If there is HA, import to CloudN HA.

3.5 Troubleshooting Tips
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 a. Check on CloudN Console. Go to Site2Cloud, make sure the tunnel is up. 
 #. Check on CloudN Console, Go to Troubleshoot -> Diagnostics -> BGP, make sure tunnel is up. Check BGP learned routes.
 #. Check on Controller. Go to Transit Network -> Advanced Config -> BGP, make sure BGP is learning routes. Also Diagnostics to execute BGP commands.
 #. Check on Controller. Go to Controller -> Site2Cloud, , site2cloud and BGP status.
 

.. |tunnel_diagram| image:: insane_mode_media/tunnel_diagram.png
   :scale: 30%


.. |insane_tunnel_diagram| image:: insane_mode_media/insane_tunnel_diagram.png
   :scale: 30%

.. |insane_transit| image:: insane_mode_media/insane_transit.png
   :scale: 30%

.. |insane_datacenter| image:: insane_mode_media/insane_datacenter.png
   :scale: 30%

.. |datacenter_layout| image:: insane_mode_media/datacenter_layout.png
   :scale: 30%

.. |deployment| image:: insane_mode_media/deployment.png
   :scale: 30%

.. |deployment_ha| image:: insane_mode_media/deployment_ha.png
   :scale: 30%

.. |deployment_dual_dx| image:: insane_mode_media/deployment_dual_dx.png
   :scale: 30%

.. |ISR-sample-config| image:: insane_mode_media/ISR-sample-config.png
   :scale: 50%

.. |insane_routing| image:: insane_mode_media/insane_routing.png
   :scale: 30%

.. |insane_mode_howto| image:: insane_mode_media/insane_mode_howto.png
   :scale: 30%

.. |InsaneBeta| image:: insane_mode_media/InsaneBeta.png
   :scale: 30%

.. disqus::
