# IRP_MN_DEVICE_USAGE_NOTIFICATION
系统组件发送此IRP以询问设备的驱动程序设备是否可以支持特殊文件。特殊文件包括分页文件，转储文件和休眠文件。如果设备的所有驱动程序成功执行IRP，系统将创建特殊文件。系统还发送此IRP通知驱动程序已从设备中删除特殊文件。

如果功能驱动程序的设备可以包含分页文件，转储文件或休眠文件，则必须处理此IRP。过滤驱动程序必须处理此IRP，如果它们正在过滤的功能驱动程序处理IRP。总线驱动程序必须为其适配器或控制器（总线FDO）及其子设备（子PDO）处理此IRP。

## major Code
IRP_MJ_PNP 

## When Sent
系统在创建或删除分页文件，转储文件或休眠文件时发送此IRP。驱动程序可以发送此IRP以将设备使用信息传播到另一个设备堆栈。

系统组件和驱动程序在IRQL PASSIVE_LEVEL中以任意线程上下文发送此IRP。. 

## Input Parameters
IO_STACK_LOCATION结构的Parameters.UsageNotification.InPath成员是一个BOOLEAN。当此参数为TRUE时，系统正在设备上创建分页，崩溃转储或休眠文件。当InPath为FALSE时，此类文件已从设备中删除。

Parameters.UsageNotification.Type是一个指示文件种类的枚举。此参数具有以下值之一：DeviceUsageTypePaging，DeviceUsageTypeDumpFile或DeviceUsageTypeHibernation。

## Output Parameters
None

## I/O Status Block
驱动程序将Irp->IoStatus.Status设置为STATUS_SUCCESS或适当的错误状态。

驱动程序不修改Irp->IoStatus.Information字段;它由发送IRP的组件设置为零。

## Operation
驱动程序按照IRP的方式处理IRP，并按照IRP的方式备份堆栈。

驱动程序通过以下程序回应此IRP：
如果Parameters.UsageNotification.InPath为TRUE，请确定设备是否支持特殊文件。

驱动程序应该测试驱动程序可以支持的特定Parameters.UsageNotification.Type。以后可能会添加其他通知类型。

请参阅下面的进一步信息，说明支持每种通知类型所需的操作。

如果Parameters.UsageNotification.InPath为TRUE，并且驱动程序无法支持设备上的特殊文件，则驱动程序必须完成IRP并出现故障状态。

如果设备支持特殊文件：
采取适当的措施来反映设备现在包含或不再包含特殊文件。
驱动程序通常会增加或减少计数器。例如，如果Parameters.UsageNotification.Type是DeviceUsageTypePaging和Parameters.UsageNotification.InPath为TRUE，则增加设备上的页面文件数量的计数。某些驱动程序调度程序必须检查计数器。

不应禁用包含特殊文件的设备。驱动程序可以调用IoInvalidateDeviceState，请求PnP管理器重新查询设备的PnP设备状态信息。为了响应IRP_MN_QUERY_PNP_DEVICE_STATE IRP，驱动程序应该设置PNP_DEVICE_NOT_DISABLEABLE标志。

如果InPath为FALSE，驱动程序将在设备的设备对象中设置DO_POWER_PAGABLE位。

将设备使用信息传播到需要信息的任何相关设备。
作为处理IRP_MN_DEVICE_USAGE_NOTIFICATION IRP的一部分，可能需要驱动程序将信息传递给一个或多个其他设备堆栈。这样的驱动程序创建新的IRP_MN_DEVICE_USAGE_NOTIFICATION IRP并将它们发送到相应的设备堆栈。在驱动程序完成处理其接收到的设备使用IRP之前，驱动程序必须等待其发送的任何设备使用通知IRP。

如何识别相关设备是特定于设备和驱动程序的。通常，驱动程序将IRP发送到其将向文件发送I / O请求的其他驱动程序。当总线驱动程序处理这个子设备的请求时，它必须向设备的父设备的堆栈发送使用通知IRP。

例如，当ftdisk运行five-disk条带集时，它会将分页，hibernate和crash dump通知传播到这五个磁盘中的每一个，因为这些设备可能需要处理分页，休眠或崩溃转储文件操作。

在函数或过滤驱动程序中，设置IoCompletion例程。
在函数或过滤驱动程序中，将Irp-> IoStatus.Status设置为STATUS_SUCCESS，设置下一个堆栈位置，并将IRP传递给具有IoCallDriver的下一个较低的驱动程序。不要完成IRP。
在正在处理子进程IRP的总线驱动程序PDO中：设置Irp-> IoStatus.Status并完成IRP（IoCompleteRequest）。
IRP完成处理期间：
如果IoCompletion例程检测到较低级驱动程序发生IRP失败，则功能或过滤驱动程序必须撤消其响应IRP执行的任何操作并传播错误。如果功能或过滤驱动程序将使用信息传播到任何其他设备堆栈，则驱动程序必须向这些堆栈发送另一个使用IRP，以通知他们该故障。

如果状态为STATUS_SUCCESS，InPath为TRUE，请清除DO_POWER_PAGABLE位。


有关处理即插即用次要IRP的一般规则，请参阅即插即用。

在设备上支持寻呼，崩溃转储和休眠文件
当驱动程序的特殊文件计数不为零时，驱动程序必须支持在其设备（或后代设备）上存在特殊文件。

对于在其设备上创建的DeviceUsageTypePaging文件，驱动程序必须执行以下操作：

为其DispatchRead，DispatchWrite，DispatchDeviceControl和DispatchPower例程锁定内存中的代码。
清除设备的设备对象中的DO_POWER_PAGABLE位（在设备堆栈的IRP上）。
对设备的IRP_MN_QUERY_STOP_DEVICE和IRP_MN_QUERY_REMOVE_DEVICE请求失败。

对于设备上的DeviceUsageTypeDumpFile文件，驱动程序必须执行以下操作：

为其DispatchRead，DispatchWrite，DispatchDeviceControl和DispatchPower例程锁定内存中的代码。
不要将设备从D0状态中取出。
不要注册设备进行空闲检测（PoRegisterDeviceForIdleDetection）。如果设备已经注册，请取消注册。如果驱动程序为设备执行自己的空闲检测，则暂停此类检测。

清除设备的设备对象中的DO_POWER_PAGABLE位（在设备堆栈的IRP上）。
对设备的IRP_MN_QUERY_STOP_DEVICE和IRP_MN_QUERY_REMOVE_DEVICE请求失败。

对于设备上的DeviceUsageTypeHibernate文件，驱动程序必须执行以下操作：

为其DispatchRead，DispatchWrite，DispatchDeviceControl和DispatchPower例程锁定内存中的代码。
当驱动程序接收到指示系统即将休眠的S4系统电源IRP时，请确保设备处于D0状态。
请勿关闭设备，以响应作为S4休眠操作的一部分的D3设置电源IRP。有关详细信息，请参阅系统电源操作
在收到这样的D3设置电源IRP后，除了关闭设备电源并通知电源管理器（PoSetPowerState）之外，请执行将设备置于D3状态所需的所有任务。设备必须保持电源，直到写入休眠文件。

清除设备的设备对象中的DO_POWER_PAGABLE位（在设备堆栈的IRP上）。
对设备的IRP_MN_QUERY_STOP_DEVICE和IRP_MN_QUERY_REMOVE_DEVICE请求失败。

有关设备电源状态，电源IRP和驱动程序中的电源管理的更多信息，请参阅电源管理。

### Sending This IRP
驱动程序可以发送IRP_MN_DEVICE_USAGE_INFORMATION IRP，但只能将设备使用信息传播到另一个设备堆栈。驱动程序从来不是设备使用信息的初始来源。

## Requirements
Headers: Declared in Wdm.h. Include Wdm.h, Ntddk.h, or Ntifs.h.

