---
title: Unified Azure IoT SDKs (preview) for IoT Hub and Azure Device Registry
titleSuffix: Azure Device Registry
description: Learn how the unified Azure IoT SDKs give developers one lifecycle-aware library to provision, connect, secure, and update devices across Azure IoT services.
author: dominicbetts
ms.author: dobett
ms.service: azure-iot
ms.topic: concept-article
ms.date: 09/29/2026
ai-usage: ai-assisted
#Customer intent: As a developer building an end-to-end Azure IoT solution, I want to understand the unified Azure IoT SDK strategy and device lifecycle so that I can choose the supported SDK path from provisioning through ongoing operation.
---

# Unified Azure IoT SDKs (preview) for IoT Hub and Azure Device Registry

> [!IMPORTANT]
> The unified Azure IoT SDKs are currently in public preview. Preview functionality is provided without a service-level agreement.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The unified Azure IoT SDKs (preview) are a set of libraries that give you a single, lifecycle-aware way to build device applications that provision, connect, secure, and update devices across Azure IoT services. Instead of stitching together separate service-specific libraries for the Device Provisioning Service, Azure IoT Hub, Azure Device Registry, certificate management, and Device Update for IoT Hub, you work with one coherent SDK that models the whole device lifecycle.

The unified SDKs exist because a production device rarely uses one Azure IoT service in isolation. Over its lifetime, a device is provisioned, connects and exchanges data, is represented and governed as a resource, and keeps its credentials current. The unified SDKs bring those stages together so that you write less integration code and follow a supported path from the first provisioning call to ongoing operation.

This article helps you understand the unified SDK strategy and how the SDKs model the end-to-end device lifecycle, so you can choose and adopt the right SDK for your language. It's for developers building an end-to-end Azure IoT solution.

## The unified SDK strategy

The unified SDKs (preview) center on one shared *connection client* and a set of capability-focused *feature clients*. The connection client owns the responsibilities that every device shares: provisioning through the Device Provisioning Service, selecting the endpoint returned by provisioning, setting up the protocol, handling certificates, reconnecting after transient failures, and reprovisioning when required. The feature clients build on that shared connection to expose specific capabilities, such as sending telemetry, working with device twins, handling direct methods, managing certificates, and applying device updates.

This design puts the strategy before the packages. Rather than asking you to choose and coordinate a provisioning library, a messaging library, and a certificate library, the unified SDKs present the device lifecycle as one model. The shared connection client coordinates the services, and the feature clients give you the capabilities you need without duplicating connection or credential logic.

A key part of the strategy is that the SDK, not your application, owns protocol selection. Your device application uses the endpoint returned by the Device Provisioning Service and doesn't explicitly choose between MQTT 3.1.1 and MQTT 5 implementations. The SDK uses:

- MQTT 3.1.1 for endpoints of IoT Hub without Azure Device Registry integration.
- MQTT 5 for endpoints of IoT Hub with Azure Device Registry integration.

> [!NOTE]
> You can use DPS to configure what type of IoT Hub your device connects to.

The unified Azure IoT SDKs are brand-new SDKs, not revisions of the existing Azure IoT Hub and Device Provisioning Service SDKs. The existing SDKs continue to work with existing Azure IoT hubs and Azure Device Registry-enabled IoT hubs. Devices that use the existing SDKs can be projected into Azure Device Registry without device-side code changes. To use new device-side capabilities such as certificate management and device updates, adopt the unified SDK.

In an Azure Device Registry-enabled architecture, the Device Provisioning Service is the only onboarding path for devices. The unified SDK incorporates recommended provisioning, connectivity, reconnection, and reprovisioning patterns. For the underlying guidance, see [Device Provisioning Service deployment-at-scale best practices](../../iot-dps/concepts-deploy-at-scale.md).

## The end-to-end Azure IoT device lifecycle

The unified SDKs model the device lifecycle as a continuous journey rather than a series of disconnected service calls. A typical device moves through the following stages:

- **Start with an onboarding credential.** The device begins with an initial credential that establishes its identity for provisioning.
- **Provision through the Device Provisioning Service.** The connection client provisions the device and receives the assigned Azure IoT Hub endpoint. Your application uses that endpoint rather than hard-coding a hub.
- **Connect and operate.** The device connects to the assigned Azure IoT Hub endpoint and exchanges telemetry and commands through the feature clients.
- **Manage certificates.** The device can request an operational X.509 certificate during provisioning and renew it while connected. Your application decides when to renew the certificate and reconnects the device to apply the renewed identity.
- **Apply updates.** The Software Update client verifies update manifests, downloads and installs updates, and reports update status.
- **Reconnect or reprovision.** After a transient connection failure, the SDK reconnects to the assigned Azure IoT Hub endpoint. The SDK provisions again through the Device Provisioning Service if Azure IoT Hub rejects the device identity or if repeated connection failures indicate that the cached assignment might be stale.

Because the connection client coordinates these stages, your application follows one consistent flow. The SDK handles transient connection recovery, but a retry can repeat an operation, so your application remains responsible for making non-idempotent operations safe to retry or for detecting duplicates.

## Map Azure IoT capabilities to SDK clients

Each stage of the lifecycle maps to a capability, and a device client exposes each capability.

| Lifecycle stage | Capability | Client type |
|---|---|---|
| Provision | Device Provisioning Service | Device client |
| Connect, telemetry, commands | Azure IoT Hub | Device client |
| Represent and manage devices | Azure Device Registry | Service and management-plane client |
| Certificate management | Certificate enrollment and renewal | Device and service clients |
| Device updates | Device Update for IoT Hub | Device client |

Device clients handle provisioning, connection, telemetry, and commands from the device itself. 

## Choose the unified Azure IoT SDK for your language

Microsoft provides unified Azure IoT SDKs for .NET and C. Both language options are currently in  preview. Choose the SDK that matches your device application language, and then follow the linked documentation for package details and samples.

The unified SDK provides a single C client suite that covers the full range of C-based devices, from constrained embedded targets such as bare-metal and FreeRTOS devices to higher-end Linux devices. You use the same C SDK regardless of your device class.

| Language | Package | Version | Documentation | Availability |
|---|---|---|---|---|
| .NET | `Microsoft.Azure.Iot.Device` | `1.0.0` | [.NET SDK documentation](/dotnet/api/overview/azure/iot) | Preview |
| .NET | `Microsoft.Azure.Iot.Device` | `2.0.0-preview` | [.NET SDK documentation](/dotnet/api/overview/azure/iot) | Preview |
| C | GitHub source | See documentation | [Unified C SDK documentation](https://github.com/Azure/azure-iot-sdk/blob/releases/public-preview/c/README.md) | Preview |

Use version `1.0.0` of the .NET SDK for production scenarios that use IoT Hub with Azure Device Registry integration for GA capabilities.

Use version `2.0.0-preview` of the .NET SDK for non-production scenarios that rely on preview capabilities of IoT Hub with Azure Device Registry integration for testing features in the preview release.
