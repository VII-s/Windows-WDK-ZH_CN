# IRP\_MN\_QUERY\_INTERFACE

IRP\_MN\_QUERY\_INTERFACE请求允许一个驱动程序向其他驱动程序导出一个直接调用接口。

导出接口的总线驱动程序必须处理其子设备\(子PDOs\)的请求。功能驱动和过滤驱动可以选择处理这个请求。

在这个上下文中，一个“接口”由一个或多个例程组成，并且可能是由一个驱动程序或一组驱动程序导出的数据。接口具有描述其内容的结构和标识其类型的GUID。

例如，PCMCIA总线驱动程序导出类型为GUID\_PCMCIA\_INTERFACE\_STANDARD的接口，其中包含诸如获取PCMCIA存储卡的写保护条件等操作的例程。这种存储卡的功能驱动程序可以向父PCMCIA总线驱动程序发送IRP\_MN\_QUERY\_INTERFACE请求以获取PCMCIA接口例程的指针。

这个部分将查询接口IRP描述为一个通用的机制。暴露接口的驱动程序应该提供关于其特定接口的额外信息。

## 主要功能码

IRP\_MJ\_PNP

## 什么时候发送

驱动程序或系统组件发送此IRP以获取有关设备的驱动程序导出的接口的信息。

驱动程序或系统组件在IRQL = PASSIVE\_LEVEL中以任意线程上下文发送此IRP。

在驱动程序的AddDevice例程被调用后，驱动程序可以随时接收该IRP。 当发送此IRP时（也就是说，您不能假定驱动程序已成功完成设备的IRP\_MN\_START\_DEVICE请求），设备可能启动或可能不启动。

## 输入参数

IO\_STACK\_LOCATION结构的Parameters.QueryInterface成员本身就是一个结构，它描述了被请求的接口。 该结构包含以下信息：

```C
CONST GUID *InterfaceType;
USHORT Size;
USHORT Version;
PINTERFACE Interface;
PVOID InterfaceSpecificData
```

结构的成员定义如下：

### InterfaceType

指向正在请求的接口的标识GUID。 GUID可用于系统定义的接口，例如GUID\_BUS\_INTERFACE\_STANDARD或自定义接口。 系统定义接口的GUID列在Wdmguid.h中。 用户界面的GUID应该与Uuidgen一起生成。

### Size

指定要请求的接口的大小。 处理此IRP的驱动程序不能返回大于Size字节的INTERFACE结构。

### Version

指定正在请求的接口的版本。  
如果驱动程序支持多个版本的接口，则驱动程序返回最接近的受支持版本，而不超出所请求的版本。 发送IRP的组件应检查返回的Interface.Version字段，并根据该值确定要执行的操作。

### Interface

指向要返回请求接口的结构。 此结构必须包含INTERFACE结构作为其第一个成员。 发送IRP的组件从分页内存中分配此结构。导出接口的驱动程序定义了包含INTERFACE结构的新结构类型，以及接口中的例程和/或数据的成员。 （驱动程序还定义了接口的GUID，如上面的InterfaceType成员所述）。导出接口的驱动程序定义了接口中每个例程的执行环境，包括可以调用该例程的IRQL等等。

### InterfaceSpecificData

指定有关正在请求的接口的附加信息。  
对于某些接口，发送IRP的组件在此字段中指定附加信息。 通常，此字段为NULL，并且InterfaceType和版本足以标识正在请求的接口。

## 输出参数

成功后，驱动程序将填充Parameters.QueryInterface.Interface结构的成员。

## I/O Status Block

驱动程序将Irp-&gt; IoStatus.Status设置为STATUS\_SUCCESS或适当的错误状态。  
成功后，总线驱动器将Irp-&gt;IoStatus.Information设置为零。  
如果功能驱动和过滤驱动程序不处理此IRP，则调用IoSkipCurrentIrpStackLocation并将IRP传递给下一个驱动程序。 这样的驱动程序不能修改Irp-&gt;IoStatus.Status，不能完成IRP。  
如果总线驱动程序不导出所请求的接口，因此不会为子PDO处理此IRP，则总线驱动程序按照原样离开Irp-&gt;IoStatus.Status并完成IRP。

## 操作

如果参数指定驱动程序支持的接口，驱动程序将处理此IRP。

如果IRP请求驱动程序不支持的接口，驱动程序不得对此IRP进行排队。驱动程序必须在其IO\_STACK\_LOCATION结构中检查Parameters.QueryInterface.InterfaceType。如果接口不是驱动程序支持的，则驱动程序必须将IRP传递到设备堆栈中的下一个较低的驱动程序，而不会阻塞。

每个接口必须提供InterfaceReference和InterfaceDereference例程，导出接口的驱动程序必须在INTERFACE结构中提供这些例程的地址。在驱动程序返回接口以响应IRP之前，它必须通过调用其InterfaceReference例程来增加接口的引用计数。当请求接口的驱动程序完成使用时，该驱动程序必须通过调用接口的InterfaceDereference例程来减小引用计数。

如果发送IRP（驱动程序x）的驱动程序稍后将接口传递给另一个驱动程序（驱动程序y），则驱动程序x必须增加接口的引用计数，并且驱动程序y必须递减它。

处理此IRP的驱动程序应避免将IRP传递到另一个设备堆栈以获取请求的接口。这样的设计将在难以管理的设备堆栈之间创建依赖关系。例如，第二个设备堆栈所表示的设备不能被删除，直到第一个堆栈中的相应驱动程序取消引用接口。

接口可以是总线特定的或总线独立的。总线专用接口在这些总线的头文件中定义。该系统定义了一个总线独立接口BUS\_INTERFACE\_STANDARD，用于导出标准总线接口。

有关处理即插即用minor IRP的一般规则，请参阅即插即用。

该IRP专门用于在设备的分层内核模式驱动程序之间传递常规入口点。不要将本IRP暴露的接口与设备接口混淆。设备接口主要用于暴露设备的路径以供用户模式组件或其他内核组件使用。有关设备接口的更多信息，请参阅设备接口类。

### 发送 IRP

有关发送IRP的信息，请参阅处理IRP。以下步骤专门适用于本IRP：

* 从分页池分配一个INTERFACE结构，并将其初始化为0。如果接口将在IRQL&gt; = DISPATCH\_LEVEL中调用，则根据接口协议，调用者可以将内容复制到从非分页池分配的内存。
* 在IRP的下一个I/O堆栈位置设置值：将MajorFunction设置为IRP\_MJ\_PNP，将MinorFunction设置为IRP\_MN\_QUERY\_INTERFACE，并在Parameters.QueryInterface中设置适当的值。
* 初始化IoStatus.Status到STATUS\_NOT\_SUPPORTED。
* 在不再需要IRP和INTERFACE结构时取消分配。
* 使用接口规范中描述的接口例程和上下文参数。
* 当不再需要接口时，使用InterfaceDereference例程减少引用计数。取消引用接口后，不要调用任何接口例程。

驱动程序通常将该IRP发送到驱动程序所在的设备堆栈顶部。如果驱动程序将此IRP发送到不同的设备堆栈，则如果其他设备不是驱动程序正在维护的设备的祖先，则驱动程序必须在其他设备上注册目标设备通知。这样的驱动程序使用EventCategoryTargetDeviceChange的EventCategory调用IoRegisterPlugPlayNotification。当驱动程序收到类型为GUID\_TARGET\_DEVICE\_QUERY\_REMOVE的通知时，驱动程序必须取消引用该接口。如果该接口接收到后续的GUID\_TARGET\_DEVICE\_REMOVE\_CANCELLED通知，驱动程序可以对接口进行重新调用。

## 要求

| 头文件 | Wdm.h \(include Wdm.h, Ntddk.h, or Ntifs.h\) |
| :--- | :--- |




