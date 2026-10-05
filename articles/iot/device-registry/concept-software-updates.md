---
title: Software Updates Concepts (Preview)
titleSuffix: Azure Device Registry
description: Learn how the Azure Device Registry software updates capability lets you prepare update artifacts and deploy standard or onboarding updates to devices.
author: dominicbetts
ms.author: dobett
ms.service: azure-iot
ms.topic: overview
ms.date: 09/16/2026
ai-usage: ai-assisted
#Customer intent: As an IoT solution architect, I want to understand the Azure Device Registry software updates capability so that I can plan how to prepare, deploy, and monitor updates across my device fleet.
---

# Software updates concepts (preview)

*Software updates* is a capability of Azure Device Registry that you use to prepare update artifacts and deploy them to devices by using [jobs](concept-jobs.md), [groups](concept-groups.md), and namespaces. It's a capability of Azure Device Registry, not a separate service.

[!INCLUDE [Relationships between groups, jobs, and software updates](includes/groups-jobs-software-updates.md)]

> [!IMPORTANT]
> Azure Device Registry software updates are currently in preview. For production workloads, continue to use [Device Update for IoT Hub](../../iot-hub-device-update/understand-device-update.md), which remains generally available.

The current preview is scoped to IoT Hub-connected devices.

## Service applicability

| Feature | Azure IoT Operations | Azure IoT Hub |
|---|---|---|
| Software updates | Not supported in this preview | Preview |

## How software updates relates to Device Update for IoT Hub

If you're already using Device Update for IoT Hub in production, continue to use that generally available experience and its existing [documentation](../../iot-hub-device-update/understand-device-update.md). The Azure Device Registry software updates capability described in this article is a new, separate experience built on Azure Device Registry primitives (namespaces, groups, and jobs).

## Architecture

The software updates capability introduces the following resources and concepts:

- **Software update**: You enable software updates at the Azure Device Registry namespace level. You configure a software update to apply to a specific group of devices within the namespace.
- **Onboarding update**: You enable onboarding updates at the Azure Device Registry namespace level. You can configure an onboarding update during initial namespace creation, or create an onboarding update job and link it to an existing namespace.
- **Artifact library**: You import update files or packages into a software updates artifact library before you can reference them from a job. Importing and managing artifacts is a separate step from running a job that applies them.
- **Device SDK**: The unified Azure Device Registry SDK includes an update agent that devices use to receive and apply updates.

## Software update packages

Importing a software update package involves more than uploading the files that a device installs. A package contains one or more update files and an import manifest that describes the update and how to apply it.

A software update package includes:

- **Update files**: The payload can include a firmware image, software package, script, or multiple related files. For example, an update can include both a full image and a delta file so that the device can use the appropriate installation path.
- **Update identity**: The provider, name, and version uniquely identify an update. The provider and name together identify an update family.
- **Compatibility properties**: Properties such as manufacturer and model describe the devices that can install the update. You can also use custom compatibility properties, such as environment, region, or deployment type.
- **Installation instructions**: Install steps, their sequence, and step-handler information tell the device how to apply the update, such as by installing a package, running a script, or writing an image.

The import manifest uses the `.importmanifest.json` file extension.

The following example shows a sample import manifest for a software update package:

```json
{
  "updateId": {
    "provider": "Contoso",
    "name": "Yocto-Image",
    "version": "1.3"
  },
  "isDeployable": true,
  "compatibility": [
    {
      "deviceModel": "Video",
      "deviceManufacturer": "contoso"
    }
  ],
  "instructions": {
    "steps": [
      {
        "type": "inline",
        "description": "Update the device image to version 0.8.0003.0",
        "handler": "microsoft/swupdate:1",
        "files": [
          "adu-update-image-raspberrypi3.swu"
        ],
        "handlerProperties": {
          "installedCriteria": "0.8.0003.0"
        }
      }
    ]
  },
  "files": [
    {
      "filename": "adu-update-image-raspberrypi3.swu",
      "sizeInBytes": 112738816,
      "hashes": {
        "sha256": "wPdNKUSmWTZkR6Pum9oR/EIzdn5AvON4dsXME+/gHVw="
      }
    }
  ],
  "createdDateTime": "2022-02-02T06:56:51.2002204Z",
  "manifestVersion": "5.0"
}
```

## Device compatibility

The compatibility properties in an update package determine which devices can install the update. Each update-capable device reports its own compatibility properties, and Azure Device Registry compares the reported properties with the properties declared in the update package.

For example, an update might declare these compatibility properties:

```text
manufacturer = Contoso
model = IrrigationControllerV2
```

A group can contain devices with different compatibility properties. If a group contains 18 `IrrigationControllerV2` devices and six `SoilSensorV1` devices, an update for `IrrigationControllerV2` is applied to all 18 compatible devices. The six incompatible devices aren't affected.

A standard software update job applies the selected update to every compatible, update-capable device in the target group. Compatible devices that join the group while the deployment is active also receive the update.

## Supported update job types

Software updates supports two job types, both defined and run through [jobs](concept-jobs.md). For the job types, targets, and execution model, see [Supported job types (preview)](concept-jobs.md#supported-job-types).

## Device SDK considerations

- The current preview supports Linux devices.
- You're responsible for integrating the Azure Device Registry SDK, including its update agent, into your device software and validating it in your own environment.

## Relationship to groups and jobs

You apply software updates to devices through a job. Preparing and uploading an artifact makes it available for the [job](concept-jobs.md):

- A standard software update job selects an artifact and a target [group](concept-groups.md).
- An onboarding update job selects an artifact and targets the devices in a namespace instead of a group.

For a standard software update job, the group defines the deployment scope and compatibility properties determine which devices in that scope can install the update. The job applies the update to every compatible device in the group.

## Related content

- [Groups concepts (preview)](concept-groups.md)
- [Jobs concepts (preview)](concept-jobs.md)
