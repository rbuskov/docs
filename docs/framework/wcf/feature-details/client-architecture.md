---
title: "Client Architecture"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 02624403-0d77-41cb-9a86-ab55e98c7966
caps.latest.revision: 7
author: "dotnet-bot"
ms.author: "dotnetcontent"
manager: "wpickett"
---
# Client Architecture
Applications use [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] client objects to invoke service operations. This topic discusses [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client objects, [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client channels, and their relationships to the underlying channel architecture. For a basic overview of [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client objects, see [WCF Client Overview](../../../../docs/framework/wcf/wcf-client-overview.md). [!INCLUDE[crabout](../../../../includes/crabout-md.md)] the channel layer, see [Extending the Channel Layer](../../../../docs/framework/wcf/extending/extending-the-channel-layer.md).  
  
## Overview  
 The service model run time creates [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] clients, which are composed of the following:  
  
-   An automatically generated client implementation of a service contract, which turns calls from your application code into outgoing messages, and turns response messages into output parameters and return values that your application can retrieve.  
  
-   An implementation of a control interface (<xref:System.ServiceModel.IClientChannel?displayProperty=nameWithType>) that groups together various interfaces and provides access to control functionality, most notably the ability to close the client session and dispose the channel.  
  
-   A client channel that is built based on the configuration settings specified by the used binding.  
  
 Applications can create such clients on demand, either through a <xref:System.ServiceModel.ChannelFactory?displayProperty=nameWithType> or by creating an instance of a <xref:System.ServiceModel.ClientBase%601> derived class as it is generated by the [ServiceModel Metadata Utility Tool (Svcutil.exe)](../../../../docs/framework/wcf/servicemodel-metadata-utility-tool-svcutil-exe.md). These ready-built client classes encapsulate and delegate to a client channel implementation that is dynamically constructed by a <xref:System.ServiceModel.ChannelFactory>. Therefore, the client channel and the channel factory that produces them are the focal point of interest for this discussion.  
  
## Client Objects and Client Channels  
 The base interface of [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] clients is the <xref:System.ServiceModel.IClientChannel?displayProperty=nameWithType> interface, which exposes core client functionality as well as the basic communication object functionality of <xref:System.ServiceModel.ICommunicationObject?displayProperty=nameWithType>, the context functionality of <xref:System.ServiceModel.IContextChannel?displayProperty=nameWithType>, and the extensible behavior of <xref:System.ServiceModel.IExtensibleObject%601?displayProperty=nameWithType>.  
  
 The <xref:System.ServiceModel.IClientChannel> interface, however, does not define a service contract itself. Those are declared by the service contract interface (typically generated from service metadata using a tool like the [ServiceModel Metadata Utility Tool (Svcutil.exe)](../../../../docs/framework/wcf/servicemodel-metadata-utility-tool-svcutil-exe.md)). [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client types extend both <xref:System.ServiceModel.IClientChannel> and the target service contract interface to enable applications to call operations directly and also have access to client-side run-time functionality. Creating an [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client provides [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)]<xref:System.ServiceModel.ChannelFactory?displayProperty=nameWithType> objects with the information necessary to create a run time that can connect and interact with the configured service endpoint.  
  
 As mentioned earlier, the two [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client types must be configured before you can use them. The simplest [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client types are objects that derive from <xref:System.ServiceModel.ClientBase%601> (or <xref:System.ServiceModel.DuplexClientBase%601> if the service contract is a duplex contract). You can create these types by using a constructor, configured programmatically, or by using a configuration file, and then called directly to invoke service operations. For a basic overview of <xref:System.ServiceModel.ClientBase%601> objects, see [WCF Client Overview](../../../../docs/framework/wcf/wcf-client-overview.md).  
  
 The second type is generated at run time from a call to the <xref:System.ServiceModel.ChannelFactory%601.CreateChannel%2A> method. Applications concerned with tight control of the communication specifics typically use this client type, called a *client channel object*, because it enables more direct interaction than the underlying client run-time and channel system.  
  
## Channel Factories  
 The class that is responsible for creating the underlying run time that supports client invocations is the <xref:System.ServiceModel.ChannelFactory%601?displayProperty=nameWithType> class. Both [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client objects and [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client channel objects use a <xref:System.ServiceModel.ChannelFactory%601> object to create instances; the <xref:System.ServiceModel.ClientBase%601> derived client object encapsulates the handling of the channel factory, but for a number of scenarios it is perfectly reasonable to use the channel factory directly. The common scenario for this is if you want to repeatedly create new client channels from an existing factory. If you are using a client object, you can obtain the underlying channel factory from a [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client object by calling the <xref:System.ServiceModel.ClientBase%601.ChannelFactory%2A?displayProperty=nameWithType> property.  
  
 The important thing to remember about channel factories is that they create new instances of client channels for the configuration provided to them prior to calling <xref:System.ServiceModel.ChannelFactory%601.CreateChannel%2A?displayProperty=nameWithType>. Once you call <xref:System.ServiceModel.ChannelFactory%601.CreateChannel%2A> (or <xref:System.ServiceModel.ClientBase%601.Open%2A?displayProperty=nameWithType>, <xref:System.ServiceModel.ClientBase%601.CreateChannel%2A?displayProperty=nameWithType>, or any operation on a [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client object), you cannot modify the channel factory and expect to get channels to different service instances, even if you are merely changing the target endpoint address. If you want to create a client object or client channel with a different configuration, you must create a new channel factory first.  
  
 [!INCLUDE[crabout](../../../../includes/crabout-md.md)] various issues using [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client objects and [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client channels, see [Accessing Services Using a WCF Client](../../../../docs/framework/wcf/feature-details/accessing-services-using-a-client.md).  
  
 The following two sections describe the creation and use of [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client channel objects.  
  
#### Creating a New WCF Client Channel Object  
 To illustrate the use of a client channel, assume the following service contract has been generated.  
  
 [!code-csharp[C_GeneratedCodeFiles#12](../../../../samples/snippets/csharp/VS_Snippets_CFX/c_generatedcodefiles/cs/proxycode.cs#12)]  
  
 To connect to an `ISampleService` service, use the generated contract interface directly with a channel factory (<xref:System.ServiceModel.ChannelFactory%601>). Once you create and configure a channel factory for a particular contract, you can call the <xref:System.ServiceModel.ChannelFactory%601.CreateChannel%2A> method to return client channel objects that you can use to communicate with an `ISampleService` service.  
  
 When using the <xref:System.ServiceModel.ChannelFactory%601> class with a service contract interface, you must cast to the <xref:System.ServiceModel.IClientChannel> interface to explicitly open, close, or abort the channel. To make it easier to work with, the Svcutil.exe tool also generates a helper interface that implements both the service contract interface and <xref:System.ServiceModel.IClientChannel> to enable you to interact with the client channel infrastructure without having to cast. The following code shows the definition of a helper client channel that implements the preceding service contract.  
  
 [!code-csharp[C_GeneratedCodeFiles#13](../../../../samples/snippets/csharp/VS_Snippets_CFX/c_generatedcodefiles/cs/proxycode.cs#13)]  
  
#### Creating a New WCF Client Channel Object  
 To use a client channel to connect to an `ISampleService` service, use the generated contract interface (or the helper version) directly with a channel factory, passing the type of the contract interface as the type parameter. Once a channel factory for a particular contract has been created and configured, you can call the <xref:System.ServiceModel.ChannelFactory%601.CreateChannel%2A?displayProperty=nameWithType> method to return client channel objects that you can use to communicate with an `ISampleService` service.  
  
 When created, the client channel objects implement <xref:System.ServiceModel.IClientChannel> and the contract interface. Therefore, you can use them directly to call operations that interact with a service that supports that contract.  
  
 The difference between using client objects and client channel objects is merely one of control and ease of use for developers. Many developers who are comfortable working with classes and objects will prefer to use the [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client object instead of the [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] client channel.  
  
 For an example, see [How to: Use the ChannelFactory](../../../../docs/framework/wcf/feature-details/how-to-use-the-channelfactory.md).
