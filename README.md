# Hybrid Cloud Connectivity

# Introduction

Organisations utilizing more than one cloud provider may have multiple dedicated connections for each of their cloud environments. This can be achieved in a number of ways, such as an SD-WAN or MPLS type of solution. One key challenge is dealing with connectivity between these cloud environments. This guideline will be focusing on a hub-spoke type of topology.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/HybridCloud.png)

We will be connecting Azure and AWS to each other, using ExpressRoute for Azure and DirectConnect for AWS. Please note that this guideline can also be used for other cloud providers such as GCP. 

At the moment of writing, there is no 'native' service offering that allows organizations to connect their Azure and AWS to each other with dedicated links. So we need alternative methods. One of those methods is to use the on-premise environment as the Hub, advertising the IP-ranges over BGP to each dedicated cloud connection. In our scenario, the on-premises environment will be simulated by a NaaS Solution from PacketFabric. Other companies offer similar NaaS Solutions, such as MegaPort. Use the provider that suits you the best. We will be using the Cloud Router service and connect it to our Azure and AWS environment. 

## Cloud Router

> __Note__ Skip this step if you will be using your on-premises routers/firewalls. This is mainly for demo purposes

To start off, we will deploy a new Cloud Router Instance through the portal, this will act as the BGP-Routing Exchange between Azure and AWS. Navigate to the portal and deploy a new Cloud Router, provide the details that are applicable to your environment. You can pick a different ASN and Region if desired.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/CloudRouter1.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/CloudRouter2.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/CloudRouter3.png)

Please wait until the Cloud Router is done provisioning before moving forward with the next step. 

## Azure Connection
Our next step is to create the ExpressRoute connection to Azure. Start by deploying the ExpressRoute Circuit.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn4.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn5.png)

The deployment should be fairly quick and the status of the circuit set to: 'Not Provisioned'. Copy the service key and proceed with the deployment on the PacketFabric portal. Create a new connection object on the Cloud Router.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn2.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn3.png)

Verify the connection deployment, the ExpressRoute circuit in Azure should now show 'Provisioned'. Move forward with the configuration of the BGP-session.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn7.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn8.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn9.png)

Wait until the portal shows a 'BGP estabilished' notification. 
It is assumed a Virtual Network Gateway and vNet have already been deployed, please make sure to follow the Microsoft documentation if this is not done already. Connect the circuit to your Virtual Network Gateway.

![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn10.png)

At this point, we won't be seeing any routes on our circuit other than the connected Virtual Network.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn11.png)

Confirm that Azure is advertising the vNet address space to our Cloud Router.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AzureConn12.png)

## AWS Connection
The AWS Connection is a bit different, start by adding a new connection. You will need the Account ID of your AWS-account. 
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn1.png)

In the AWS-portal, navigate to the AWS Direct Connect page. It should showcase a new entry pending confirmation.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn2.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn3.png)

Create a Direct Connect Gateway and provide an ASN within the valid range. 
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn4.png)

For this demo, I will be using the Transit Gateway. Deploy the Transit Gateway, create the Private Interface Attachment and Associate the Direct Connect Gateway to the Transit Gateway.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn5.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn6.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn7.png)

We also need to connect our VPC containing the VM to our Transit Gateway. Once again, it is assumed this part has been deployed already. If not, please deploy a VPC with a VM using the AWS Documentation.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn9.png)

Configure BGP on the Cloud Router and confirm the BGP status.
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn11-.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn12.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/AWSConn13.png)

## Route Check
Now that we have an DirectConnect and ExpressRoute connection, including BGP-peering, our next step is to check the routing. Confirm the advertised and received routes.

AWS BGP-session:
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck1.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck2.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck3.png)

Azure BGP-Session
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck4.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck5.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/264d0c70089eeb644c97d2027eb49572e7e0a3cd/images/RouteCheck6.png)

To test connectivity, we can send a simple ping from our VM in Azure to our VM in AWS:
![](https://github.com/infobozk/HybridCloudConnectivity/blob/5f050b93c233ecba2b31d4c7ac9d85ca8a81aa95/images/PingTest.png)
![](https://github.com/infobozk/HybridCloudConnectivity/blob/5f050b93c233ecba2b31d4c7ac9d85ca8a81aa95/images/TraceTest.png)

## Summary
There are other methods available to connect different cloud workloads together, alternatives such as setting up a VPN-tunnel or assigning the responsibility to the ServiceProvider / SD-WAN provider. It all depends on your requirements. The benefit of this solution:

+ Using the dedicated connections instead of the internet.
+ Provides flexibility as you have full control over the BGP-routing Exchange.
+ No/limited dependency on external parties. 

That being said, it could not meet your requirements. Just one of the options available to us, Always make sure to have your requirements clear and plan accordingly. This guideline shows us how to use our on-premises device as a next-hop / Hub in a hybrid cloud environment.
