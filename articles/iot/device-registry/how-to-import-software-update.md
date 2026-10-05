---
title: Import and Define a Software Update
titleSuffix: Azure Device Registry
description: Learn how to import a software update package into Azure Device Registry and define its update manifest so it's ready to deploy to a group.
author: dominicbetts
ms.author: dobett
ms.service: azure-iot
ms.topic: how-to
ms.date: 08/28/2026
#Customer intent: As an IoT solution operator, I want to import a software update package and define its manifest so that I can deploy it to devices through Azure Device Registry.
---

# Import and define a software update

Before you can deploy a software update to a group of devices or a namespace, you need to import the update package into Azure Device Registry and ensure it has a valid update manifest. The update manifest describes the update's identity, compatibility, and installation steps so Azure Device Registry knows which devices the update applies to and how to install it.

This preview is currently scoped to IoT Hub-connected devices in an Azure Device Registry namespace.

This article shows you how to import an update package from an Azure Storage container into the software updates library for a namespace, and how to define a manifest for an update if one isn't already provided.

> [!IMPORTANT]
> Azure Device Registry software updates are currently in preview. This preview is provided without a service-level agreement, and the capability and its portal experience might change before it becomes generally available. For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Prerequisites

- An active Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An existing Azure Device Registry namespace. For setup steps, see [Deploy Azure IoT Hub with ADR integration](../../iot-hub/iot-hub-device-registry-setup.md).
- Permissions to manage software updates in the Azure Device Registry namespace, such as the [Azure Device Registry Credentials Contributor](../../role-based-access-control/built-in-roles/internet-of-things.md#azure-device-registry-credentials-contributor) role.
- An Azure Storage account and container that contains your update files. Supported file types include update payloads (for example, `.swu`, `.sh`). Every software update needs an import manifest file with the `.importmanifest.json` extension.

## Go to the software updates library

1. In the [Azure portal](https://portal.azure.com), go to your Azure Device Registry namespace.
1. In the resource menu, under **Operations**, select **Software updates (preview)**.
1. On the **Software updates (preview)** page, select the **Imports** tab to view the update packages that are already imported for this namespace.

If you didn't import any updates yet, the page shows **No updates to display**. Select **Import update** to start the import process.

## Import an update package

1. On **Software updates (preview)**, select **+ Import**.
1. On **Import software updates**, under **File selection**, select **+ Add files**.

   :::image type="content" source="media/how-to-import-software-update/import-form.png" alt-text="Screenshot of the Import software updates page with the Add files command." lightbox="media/how-to-import-software-update/import-form.png":::

1. Select the storage account that contains your update files, and then select **Next**.

1. Select the container that contains your update files, and then select **Next**.
1. On **Select files**, select the update payload files you want to import. If your container also has an import manifest file (with the `.importmanifest.json` extension), select it too.

   :::image type="content" source="media/how-to-import-software-update/select-update-files.png" alt-text="Screenshot of update payload and import manifest files selected in a storage container." lightbox="media/how-to-import-software-update/select-update-files.png":::

1. Select the files, and then close the file picker to return to **Import software updates**.
1. In the **Description** box, enter an optional description for the update.
1. Under **Malware scan**, select **Scan update files for malware** if you want Azure Device Registry to check the update files for known malware threats before importing. Scanning is available in select regions.

1. Select **Import**.

If the import succeeds, the update appears in the **Imports** list with its name, provider, version, and description.

### Bundle updates with a parent and child manifest

Some updates are made up of multiple related update packages, such as a parent update that references one or more child updates. When you import files that include a parent/child manifest relationship, Azure Device Registry shows the parent update along with its associated child updates after a successful import. Use this structure when you need to describe a bundle of updates that must be installed together or in a specific order.

   :::image type="content" source="media/how-to-import-software-update/nested-import.png" alt-text="Screenshot of nested import showing parent and child updates." lightbox="media/how-to-import-software-update/nested-import.png":::

## Define a manifest for an update

An import manifest describes the update's identity, compatibility requirements, and installation steps. If you select update payload files without an accompanying `.importmanifest.json` file, Azure Device Registry can't determine this information automatically, and the import fails with an error.

1. If your import fails with the message **No import manifest was found**, review the error banner on the **Import software updates** page. The banner reminds you that the file extension for import manifests is `.importmanifest.json`.
1. Select **Replace files**, go back to your storage container, add an `.importmanifest.json` file that describes the update, and reselect your files.
1. After you provide the required manifest information, select **Import** again to complete the import with the newly defined manifest.

## Delete an update

If you no longer need an imported update, you can remove it from the software updates library.

1. On the **Software updates (preview)** page, select the **Imports** tab.
1. Select the checkbox next to the update you want to remove.
1. Select **Delete**.
1. When prompted **Are you sure you want to delete this software update?**, select **Delete** to confirm.

Deleting an update removes it from the software updates library. Devices that already received the update aren't affected, but you can no longer target this update in a new job.

## Next steps

[Deploy a software update to a group](how-to-deploy-software-update-group.md)
