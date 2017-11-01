# 即插即用次要 IRPs
本节介绍发送到驱动程序的PnP IRP。所有PnP IRP具有主要功能码IRP_MJ_PNP和指示特定PnP请求的次要功能码。

本节提供各个IRP的参考信息。有关IRP发送顺序的说明，请参阅即插即用，有关如何处理DispatchPnP例程中IRP的讨论，以及PnP概念和术语的一般讨论。

对于每个IRP和每种驱动程序，需要一个驱动程序来处理IRP，可以选择处理IRP，也不能处理IRP。请参阅下表以确定您的驱动程序将处理哪些IRP，然后查阅参考页面了解有关各个IRP的信息。 IRP按IRP参考页中的表格和字母顺序列出。

如果IRP在特定驱动程序的表中标记为“否”，则该驱动程序不能处理IRP。驱动程序必须将IRP传递给设备堆栈中的下一个驱动程序，如IRP参考页所述。

PnP经理发送这些IRP。 PnP驱动程序可以发送这些IRP中的一些，但只能在本节中提到。

以下是PnP IRP的小功能代码以及处理它们的驱动程序类型：

| PnP IRP次要功能代码                 | 非总线设备的功能<br/>或过滤驱动程序 | 总线设备<br/>功能驱动程序<br/>（用于总线FDO） | 总线驱动程序<br/>或总线过滤驱动程序<br/>（用于子PDO） |
| ----------------------------------- | ------------------------------ | ----------------------------------- | ------------------------------------------- |
| IRP_MN_START_DEVICE                 | 必须                           | 必须                                | 必须                                        |
| IRP_MN_QUERY_STOP_DEVICE            | 必须                           | 必须                                | 必须                                        |
| IRP_MN_STOP_DEVICE                  | 必须                           | 必须                                | 必须                                        |
| IRP_MN_CANCEL_STOP_DEVICE           | 必须                           | 必须                                | 必须                                        |
| IRP_MN_QUERY_REMOVE_DEVICE          | 必须                           | 必须                                | 必须                                        |
| IRP_MN_REMOVE_DEVICE                | 必须                           | 必须                                | 必须                                        |
| IRP_MN_CANCEL_REMOVE_DEVICE         | 必须                           | 必须                                | 必须                                        |
| IRP_MN_SURPRISE_REMOVAL             | 必须                           | 必须                                | 必须                                        |
| IRP_MN_QUERY_CAPABILITIES           | 可选                           | 可选                                | 必须                                        |
| IRP_MN_QUERY_PNP_DEVICE_STATE       | 可选                           | 可选                                | 可选                                        |
| IRP_MN_FILTER_RESOURCE_REQUIREMENTS | 可选 (1)                       | 可选 (1)                            | No                                          |
| IRP_MN_DEVICE_USAGE_NOTIFICATION    | 必须 (1)                       | 必须 (1)                            | 必须 (1)                                    |
| IRP_MN_QUERY_DEVICE_RELATIONS       |                                |                                     |
| - BusRelations                      | 可选 (1)                       | 必须                                | No (2)                                      |
| - EjectionRelations                 | No                             | No                                  | 可选                                        |
| - RemovalRelations                  | 可选                           | 可选                                | No                                          |
| - TargetDeviceRelation              | No                             | No                                  | 必须                                        |
| IRP_MN_QUERY_RESOURCES              | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_QUERY_RESOURCE_REQUIREMENTS  | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_QUERY_ID                     |                                |                                     |
| - BusQueryDeviceID                  | No                             | No                                  | 必须                                        |
| - BusQueryHardwareIDs               | No                             | No                                  | 可选                                        |
| - BusQueryCompatibleIDs             | No                             | No                                  | 可选                                        |
| - BusQueryInstanceID                | No                             | No                                  | 可选                                        |
| - BusQueryContainerID               | No                             | No                                  | 必须 (3)                                    |
| IRP_MN_QUERY_DEVICE_TEXT            | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_QUERY_BUS_INFORMATION        | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_QUERY_INTERFACE              | 可选                           | 可选                                | 必须 (1)                                    |
| IRP_MN_READ_CONFIG                  | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_WRITE_CONFIG                 | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_DEVICE_ENUMERATED            | No                             | No                                  | 必须 (1)                                    |
| IRP_MN_SET_LOCK                     | No                             | No                                  | 必须 (1)                                    |
（1）在某些情况下必需或可选。 有关详细信息，请参阅IRP的参考页面。
（2）总线过滤驱动程序可能会处理BusRelations的查询。
（3）Windows 7及更高版本的Windows支持