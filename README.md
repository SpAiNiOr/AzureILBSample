# How to create Internal Load Balancer for PaaS Cloud Service

## Introduction
Sometime, we want only allow cloud service be accessed within a virtual network. Or we want access cloud service by static IP within a virtual network.
This example is to help you configure Internal Load Balancer for web role. This ILB can be also used with worker role. And in this sample, we will be binding an ILB with port 8080 for web role.
**Please note, if you binding a port for ILB, this port will be only accessible by ILB**.  

## Prerequisites
1	*__Virtual Network__*

Internal Load Balance needs deploy to a Virtual Network, please follow the following wizard to create one.
https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-vnet-classic-portal/

## Building the Sample
1	*__Open Solution in Visual Studio 2015__*

Now you can use Visual Studio 2015 to build a cloud service solution.
This solution contains cloud service project with 1 web role project and 1 work role. 

2	*__Configure Service Definition File__*

Open ServiceDefinition.csdef, add a __Endpoint2__ for load balancer to use.
Change the following settings as yours.
```c#
  <WebRole name="WebRole1" vmsize="Small">
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="Endpoint1" endpointName="Endpoint1" />
          <Binding name="Endpoint2" endpointName="Endpoint2" />
        </Bindings>
      </Site>
    </Sites>
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
    </ConfigurationSettings>
    <Endpoints>
      <InputEndpoint name="Endpoint1" protocol="http" port="80" />
      <InputEndpoint name="Endpoint2" protocol="http" port="8080" loadBalancer="name of the load balancer" />
    </Endpoints>
    <Imports>
      <Import moduleName="RemoteAccess" />
    </Imports>
  </WebRole>
```
This cloud service has 2 endpoints, once we binding endpoint2 with ILB, endpoint2 will no longer accessible by cloudservice.cloudapp.net:endpoint2. This endpoint2 only can be accessible by ILB’s IP + port.
If we don’t want cloud service be accessible from outside the VNet, just remove the endpoint1. 

3	*__Configure Service Configuration File__*

Open ServiceConfiguration.Cloud.cscfg and change content as below.
Change the following settings as yours.
```c#
<NetworkConfiguration>
    <VirtualNetworkSite name="name of virtual network" />
    <AddressAssignments>
      <InstanceAddress roleName="WebRole1">
        <Subnets>
          <Subnet name="Subnet-1" />
        </Subnets>
      </InstanceAddress>
      <InstanceAddress roleName="WorkerRole1">
        <Subnets>
          <Subnet name="Subnet-1" />
        </Subnets>
      </InstanceAddress>
    </AddressAssignments>
    <LoadBalancers>
      <LoadBalancer name="name of the load balancer">
        <FrontendIPConfiguration type="private" subnet="Subnet-1" staticVirtualNetworkIPAddress="static-IP-address" />
      </LoadBalancer>
    </LoadBalancers>
  </NetworkConfiguration>
```
Add above configuration under last  __&lt;/ Role>__.

## Running the Sample

Right click cloud service project and choose “Publish…” command.

![1](https://github.com/SpAiNiOr/AzureILBSample/blob/master/Images/1.jpg)

Choose or create an Cloud Service.

![2](https://github.com/SpAiNiOr/AzureILBSample/blob/master/Images/2.jpg)

Choose or create an Storage Account

![3](https://github.com/SpAiNiOr/AzureILBSample/blob/master/Images/3.jpg)

Then Publish

![4](https://github.com/SpAiNiOr/AzureILBSample/blob/master/Images/4.jpg)

__How to verify?__ 

We can login to a VM within that VNet, then try access the Cloud Service by ILB's IP  with port 8080 to verify if the internal load balancer works fine or not.

## Third-party solution disclaimer

The information and the solution in this document represent the current view of Microsoft Corporation on these issues as of the date of publication. This solution is available through Microsoft or a third-party provider. We do not specifically recommend any third-party provider or third-party solution that this article might describe. There might also be other third-party providers or third-party solutions that this article does not describe. Because we must respond to changing market conditions, this information should not be interpreted as a commitment by Microsoft. We cannot guarantee or endorse the accuracy of any information or of any solution that is presented by Microsoft or by any mentioned third-party provider. 
Microsoft makes no warranties and excludes all representations, warranties, and conditions whether express, implied, or statutory. These include but are not limited to representations, warranties, or conditions of title, non-infringement, satisfactory condition, merchantability, and fitness for a particular purpose, with regard to any service, solution, product, or any other materials or information. In no event will Microsoft be liable for any third-party solution that this article mentions.
