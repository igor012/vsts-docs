---
title: Keep your OSS packages safe with upstream sources | Azure Artifacts
description: Use upstream sources in Azure Artifacts to keep your OSS packages safe from failing dependencies.
ms.technology: devops-artifacts
ms.reviewer: amullans
ms.date: 12/04/2018
monikerRange: '>= tfs-2017'
---

# Protect your open-source software packages with upstream sources

Upstream sources enable you to manage your product's OSS dependencies in a single feed. Using upstream sources makes it easy to use your favorite OSS packages, and can also give you additional protection against outages and corrupted or compromised packages. You can also publish private dependencies in the same feed that manages your OSS dependencies. Read all about [upstream sources and their benefits](/azure/devops/artifacts/concepts/upstream-sources).

This tutorial covers how to upgrade an existing project that uses OSS packages from public registries like nuget.org, npmjs.com, etc. to instead get those dependencies from an Azure Artifacts feed with upstream sources.

In this tutorial, you will:

>[!div class="checklist"]  
> * Create a new feed using upstream sources
> * Replace the public registry in your configuration files
> * Run an initial package restore to populate your feed
> * Check your feed to see the saved copy of everything you used from the public registry

## Create a feed with upstream sources enabled

1. Navigate to **Azure Artifacts**:

::: moniker range=">= azure-devops-2019"

   > [!div class="mx-imgBorder"] 
   >![Go to Azure Artifacts](../media/goto-feed-hub-azure-devops-newnav.png)

::: moniker-end

::: moniker range="<= tfs-2018"

   ![Go to Azure Artifacts](../media/goto-feed-hub.png)

::: moniker-end

1. Select **+ New feed**:

::: moniker range=">= azure-devops-2019"

   > [!div class="mx-imgBorder"] 
   >![New feed button](../media/new-feed-button-azure-devops-newnav.png)

::: moniker-end

::: moniker range="<= tfs-2018"

   ![New feed button](../media/new-feed-button.png)

::: moniker-end

1. In the dialog, provide a feed name and click _Create_. 

::: moniker range=">= azure-devops-2019"

   > [!div class="mx-imgBorder"] 
   >![New feed dialog](../media/new-feed-dialog.png)

::: moniker-end

::: moniker range="<= tfs-2018"

   ![New feed dialog](../media/new-feed-dialog.png)

::: moniker-end

::: moniker range=">=tfs-2017 < azure-devops"

1. Navigate to the **Packages** page:

   ![Go to Azure Artifacts](../media/goto-feed-hub.png)

1. Select **+ New feed**:

   ![New feed button](../media/new-feed-button.png)

1. In the dialog:

   - Give the feed a name.
   - Choose who can read and contribute (or update) packages in your feed.
   - Select _Use packages from public sources through this feed_
   - When you're done, choose **Create**.

   ![New feed dialog](../media/new-feed-dialog.png)

::: moniker-end

Now that you've created the feed that we will use to store your packages, you will update your configuration files to point to the newly created feed.

## Replace the public registry in configuration files with the Azure Artifacts feed

The next step is to update your configuration file to point to the new Azure Artifacts feed instead of the public registry. There are two steps to achieve this:

1. Get your feed's URL
2. Update the configuration file with the feed URL

#### [npm](#tab/npm/)
::: moniker range=">= azure-devops-2019"

1. From your **Packages** page, click _Connect to Feed_

   > [!div class="mx-imgBorder"] 
   >![Connect to feed button in Azure Artifacts](../media/connect-to-feed-azure-devops-newnav.png)

2. Select the **npm** tab under the **npm** header

> [!NOTE]
> If you don't have **npm** or the **artifacts-credhelper** installed, select **Get the tools** in the top right and follow steps **1** and **2** to get the tools to continue.

3. Follow the instructions under **Project setup** :

   > [!div class="mx-imgBorder"] 
   >![Connect to feed from Azure Artifacts](../media/connect-to-feed-npm-registry-azure-devops-newnav.png)

::: moniker-end

::: moniker range="< azure-devops-2019"

1. From your **Packages** page, click _Connect to Feed_

    ![Connect to feed button in Azure Artifacts](../media/connect-to-feed.png)

2. Copy the "registry" text:

    ![Connect to feed from Azure Artifacts](../media/connect-to-feed-npm-registry.png)

::: moniker-end

After you've got the feed URL, create a new text file named `.npmrc` in the root of your project (in the same folder as your `package.json` file). Open your new `.npmrc` file and paste the text that you copied in step 2 above.

#### [NuGet](#tab/nuget/)

1. Go to your feed [or create a feed if you haven't](https://docs.microsoft.com/azure/devops/artifacts/get-started-nuget?view=azure-devops&tabs=new-nav#create-a-feed). 

2. Select **Connect to feed**:

   > [!div class="mx-imgBorder"] 
   >![Connect to feed button on the upper right of the page](../media/connect-to-feed-nuget-exe-config-azure-devops-newnav.png)
   >

3. Select the **NuGet.exe** tab under the **NuGet** header 

> [!NOTE]
> If you don't have **NuGet** or the **credential provider** installed, select **Get the tools** in the top right and follow steps **1**  and **2** to get the tools to continue.

4. Copy the XML snippet under **Project Setup** to your clipboard.

   > [!div class="mx-imgBorder"] 
   >![Connect to feed button on the upper right of the page](../media/connect-to-feed-azure-devops-newnav.png)
   >

5. Create a new file named `nuget.config` in the root of your project.

6. Paste the XML snippet from your clipboard to your `nuget.config` file.
 
* * *

## Run an initial package restore to populate your feed

Now that you have upstream packages set up, you'll need to run an initial package restore to populate your new feed with the upstream packages.

The basic steps are to clear your local package cache and then do a clean install of all the packages used by the project so that Azure Artifacts can save them from the upstream source. 

# [npm](#tab/npm)

Remove the `node_modules` folder in your project (find out more about the [node_modules folder](https://docs.npmjs.com/files/folders.html#node-modules)), and rerun:

```
npm install --force
```

> The `-force` option is to ensure the cache is bypassed. 

# [NuGet](#tab/nuget)

Clear your local package cache:

```
nuget locals --clear all
```

Then, download and install packages from the upstream sources:

```
nuget restore
```

---

The instructions above show the simplest way to populate your feed. In larger projects, you can also consider setting up a continuous integration (CI) build that has a clean cache on each build run. This build will then save any new packages from upstream sources as they're used.

## Check your feed to see the saved copy of everything you used from the public registry

Navigate to the feed you created in [Step 1](#create-a-feed-with-upstream-sources-enabled). This feed should now be populated with the packages that are used in your project. The **Source** field contains the public registry, or other upstream source, that you were using before completing this tutorial.
