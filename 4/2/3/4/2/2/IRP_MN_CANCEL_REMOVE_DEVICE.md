# IRP_MN_CANCEL_REMOVE_DEVICE
所有PnP驱动程序必须处理此IRP。

## Major Code
IRP_MJ_PNP

## When Sent
PnP管理器发送此IRP通知驱动程序设备将不会被删除。

PnP管理器在系统线程的上下文中将IRP发送到IRQL PASSIVE_LEVEL。

## Input Parameters
None

## Output Parameters
None

## I/O Status Block
驱动程序必须将Irp-> IoStatus.Status设置为此IRP的STATUS_SUCCESS。 如果驱动程序未通过此IRP，则设备处于不一致状态。

## Operation
该IRP必须首先由设备的父总线驱动程序处理，然后由设备堆栈中的每个较高的驱动程序处理。

响应此IRP，驱动程序将设备恢复到接收IRP_MN_QUERY_REMOVE_DEVICE请求之前的状态。

如果设备在驱动程序接收到IRP时已经启动，则驱动程序只需将状态设置为成功，并将IRP传递给下一个驱动程序（如果驱动程序是总线驱动程序，则完成IRP）。对于这样的取消删除IRP，功能或过滤驱动程序不需要设置完成例程。设备可能未处于移除待处理状态，因为例如，驱动程序在以前的IRP_MN_QUERY_REMOVE_DEVICE中失败。

PnP管理器在IRP_MN_CANCEL_REMOVE_DEVICE请求完成后调用具有GUID_TARGET_DEVICE_REMOVE_CANCELLED的任何EventCategoryTargetDeviceChange通知回调。这种回调通过调用IoRegisterPlugPlayNotification在设备上注册。 PnP管理器还通过调用RegisterDeviceNotification来调用在设备上注册通知的任何用户模式组件。

如果设备上装载了文件系统，则它必须撤消对响应query-remove通知所做的任何操作。

有关处理删除IRP的详细信息以及处理所有即插即用次要IRP的一般规则，请参阅即插即用。

### Sending This IRP
保留供系统使用。 驱动不得发送此IRP。


## Requirements
Headers: Declared in Wdm.h. Include Wdm.h, Ntddk.h, or Ntifs.h.

