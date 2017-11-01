# IRP_MN_CANCEL_STOP_DEVICE
所有PnP驱动程序必须处理此IRP。

## Major Code
IRP_MJ_PNP

## When Sent
PnP管理器在IRP_MN_QUERY_STOP_DEVICE之后的某个时间发送此IRP，通知驱动程序设备不会被禁用（仅适用于Windows 98 / Me）或停止资源重新配置。

PnP管理器在IRQL 为PASSIVE_LEVEL的系统线程的上下文中发送此IRP。

## Input Parameters
None

## Output Parameters
None

## I/O Status Block
驱动程序必须将Irp-> IoStatus.Status设置为此IRP的STATUS_SUCCESS。如果驱动程序未通过此IRP，则设备处于不一致状态。

## Operation
该IRP必须首先由设备的父总线驱动程序处理，然后由设备堆栈中的每个较高的驱动程序处理。

响应此IRP，驱动程序将设备恢复到启动状态。驱动程序启动设备处于停止待机状态时所持有的任何IRP。

如果设备在驱动程序接收到IRP时已经处于活动状态，则功能或过滤驱动程序只需将状态设置为成功，并将IRP传递给下一个驱动程序。父母总线驱动程序完成IRP。对于这种取消停止IRP，功能或过滤驱动程序不需要设置完成例程。

有关处理停止IRP的详细信息以及处理所有即插即用小型IRP的一般规则，请参阅即插即用。

### Sending This IRP
保留供系统使用。驱动不得发送此IRP


## Requirements
Headers: Declared in Wdm.h. Include Wdm.h, Ntddk.h, or Ntifs.h.

