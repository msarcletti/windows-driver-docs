---
Description: 'Starting with Windows 8, a miniport driver can use IPortClsRuntimePower, a new interface, for private context sharing with the Windows Power Engine Plug-in (PEP).'
MS-HAID: 'audio.portcls\_private\_pep\_context\_sharing'
MSHAttr: 'PreferredLib:/library/windows/hardware'
title: PortCls Private PEP Context Sharing
---

# PortCls Private PEP Context Sharing


Starting with Windows 8, a miniport driver can use IPortClsRuntimePower, a new interface, for private context sharing with the Windows Power Engine Plug-in (PEP).

The audio port class driver (PortCls) has been updated to expose the new interface, IPortClsRuntimePower, on the WaveRT port. In order for a miniport driver to send private power controls to the operating system's PEP, the miniport driver first has to gain access to the IPortClsRuntimePower interface of its associated port. The miniport driver then registers a callback that is invoked at the appropriate time, allowing the miniport driver to send the private power controls.

## <span id="Accessing_IPortClsRuntimePower"></span><span id="accessing_iportclsruntimepower"></span><span id="ACCESSING_IPORTCLSRUNTIMEPOWER"></span>Accessing IPortClsRuntimePower


The miniport driver gains access to its port's IPortClsRuntimePower via the following sequence of events:

1. The miniport driver calls [**PcNewPort**](audio.pcnewport) and supplies IID\_IPortWaveRT as the REFID.

2. **PcNewPort** creates a port interface (Pport) of type [IPortWaveRT](audio.iportwavert).

3. The miniport driver then calls QueryInterface in the newly created **IPortWaveRT** port interface, and specifies IID\_IPortClsRuntimePower as the interface GUID.

4. The **IPortWaveRT** port interface provides the miniport driver with a pointer to its [**IPortClsRuntimePower**](audio.iportclsruntimepower) interface.

The *Portcls.h* header file defines the GUID for the IPortClsRuntimePower as follows:

``` syntax
// {E057C351-0430-4DBC-B172-C711D40A2373}
DEFINE_GUID(IID_IPortClsRuntimePower,
0xe057c351, 0x430, 0x4dbc, 0xb1, 0x72, 0xc7, 0x11, 0xd4, 0xa, 0x23, 0x73);
```

## <span id="Registering_a_callback"></span><span id="registering_a_callback"></span><span id="REGISTERING_A_CALLBACK"></span>Registering a callback


The miniport driver uses the [**IPortClsRuntimePower::RegisterPowerControlCallback**](audio.iportclsruntimepower_registerpowercontrolcallback) method to register a callback. This method is invoked either when the PEP initiates a private request, or in response to a private request that is initiated by the miniport driver itself. The callback registration should typically be performed while the driver is handling the IRP\_MN\_START\_DEVICE PNP Irp.

Aside from the Context pointer that is supplied in the callback, the other parameters are defined identically to the definitions for the runtime power framework’s PowerControlCallback. Additionally, the miniport’s callback must be of type PCPFNRUNTIME\_POWER\_CONTROL\_CALLBACK, as defined in the following snippet from the *Portcls.h* header file.

```ManagedCPlusPlus
typedef
NTSTATUS
_IRQL_requires_max_(DISPATCH_LEVEL)
(*PCPFNRUNTIME_POWER_CONTROL_CALLBACK)
(
    _In_        LPCGUID PowerControlCode,
    _In_opt_    PVOID   InBuffer,
    _In_        SIZE_T  InBufferSize,
    _Out_opt_   PVOID   OutBuffer,
    _In_        SIZE_T  OutBufferSize,
    _Out_opt_   PSIZE_T BytesReturned,
    _In_opt_    PVOID   Context
);
```

When the driver is stopped or removed, it must use the [**IPortClsRuntimePower::UnregisterPowerControlCallback**](audio.iportclsruntimepower_unregisterpowercontrolcallback) method to unregister any registered callbacks.

## <span id="Sending_private_power_controls"></span><span id="sending_private_power_controls"></span><span id="SENDING_PRIVATE_POWER_CONTROLS"></span>Sending private power controls


After the miniport establishes access to an **IPortClsRuntimePower** interface, and uses the interface's **RegisterPowerControlCallback** method to register a callback, it is now ready to send private power controls. When the callback method is invoked, the miniport driver uses the [**IPortClsRuntimePower::SendPowerControl**](audio.iportclsruntimepower_sendpowercontrol) method to send the private power controls to the Windows PEP.

With the exception of the *DeviceObject* parameter, all other parameters are defined identically to those for the runtime power framework’s [PoFxPowerControl](http://msdn.microsoft.com/en-us/library/windows/hardware/hh439518.aspx) method.

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[audio\audio]:%20PortCls%20Private%20PEP%20Context%20Sharing%20%20RELEASE:%20%287/14/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/en-us/default.aspx. "Send comments about this topic to Microsoft")

