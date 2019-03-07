---
description: How to set up a new development PC for WEM development
---

# Development setup

## Introduction

This document explains the steps needed to run and debug the WEM modeler, preview and runtime websites locally on a development PC.

## 0. Prerequisites

* The development PC should have an `E:\` drive with at least 10 GB of free space.
* SQL Server 2016 and [SQL Management Studio](https://go.microsoft.com/fwlink/?linkid=847722)
* Visual Studio Enterprise 2017 \(version 15.5.6 or later\)
* [ReSharper](https://www.jetbrains.com/resharper/) \(latest version\)
* [Beyond Compare 3.3.13](http://www.scootersoftware.com/BCompare-3.3.13.18981.exe) \(we don't have a license for Beyond Compare 4\)
* [TypeScript 2.7](https://www.microsoft.com/en-us/download/details.aspx?id=55258) \(or later\) for Visual Studio 2017
* [Node.js 8.9.4](https://nodejs.org/en/) \(or later\)

SQL Server 2016 and Visual Studio 2015 are aquired via our MSDN subscription. Ask Elian Ebbing or Gijs Reijner for the installation images.

### Beyond Compare

License key

> -YD41um+vIqe8WlgxnyLIago498asN-t4OcdIsXyYUqNQPyYPoWre3NTH  
>  GbDk9vhqy-3CizIMUIzDktSuunw6jIYsrZp8ZxIOh0FQvv1HJL-Hm0Qqs  
>  XG31jBrsZGtLi5t7tD7xcf82LH8+sHxCeTjmqJ5f6SwLmj9j++EZo4V4g

#### Integrate in Visual Studio

**Diff** 

1. In Visual Studio Choose **Options** from the **Tools** menu. 
2. Expand **Source Control** in the treeview. 
3. Click **Visual Studio Team Foundation Server** in the treeview. 
4. Click the **Configure User Tools** button. 
5. Click the **Add** button. 
6. Enter `.*` in the **Extension** edit. 
7. Choose **Compare** in the Operation combobox. 
8. Enter the path to BComp.exe in the **Command** edit. \(most likely `C:\Program Files (x86)\Beyond Compare 3\BComp.exe`\) 
9. In the Arguments edit, use: `%1 %2 /title1=%6 /title2=%7`

**3-way Merge \(v3 Pro\)** 

1. Follow steps 1-6 above. 
2. Choose **Merge** in the **Operation** combobox. 
3. Enter the path to BComp.exe in the **Command** edit. 
4. In the **Arguments** edit, use: `%1 %2 %3 %4 /title1=%6 /title2=%7 /title3=%8 /title4=%9`

### PostSharp

[PostSharp](https://www.postsharp.net/) is stored in TFS and does not need to be installed. However, you will be prompted to register PostSharp when running WEM on a development machine. The license key is:

> 21639-AUADSQQQZEQEQSE4588MXASDFZEWX8SASQ-QYQYTQXGV75B5TS2449WQC99HB2BFP7W2XT5U2CH-6P4R76P9SWEJ37EYPX369SD4BL7NRAQUPA

The license key is also stored in source control.

```text
$/WemCore/Main/Packages/PostSharp/PostSharp.custom.targets
```

### Webpack

We use the webpack javascript bundler to bundle all the modeler javascript files. In order to get the bundler to work, you need to make sure that node.js is installed. Open a console and type the following command to install webpack globally:

```text
npm install webpack -g
```

After webpack is installed, you can install the **WebPack Task Runner** extension by Mads Kristensen. To do this, open Visual Studio 2017 and go to `Tools > Extensions and Updates...`. Select "online" and search for "WebPack". Note that the plugin description talks about Visual Studio 2015, but this plugin supports VS2017 as well.

## 1. Prepare file system

A WEM development environment needs to have an `E:\` drive. This drive must contain the following folders:

* `E:\TFS` - This must be the local folder that is mapped to the root of the TFS repository.
* `E:\Wem\Export Cache` - This folder is used by the modeler to store zipped portal definitions.
* `E:\Wem\Runtime\Portals` - This folder is used by the live runtime website to store portal definitions.
* `E:\Wem\File service` - This folder is used by the wem runtime file service.
* `E:\Wem\Services` - The location of four windows services that are needed for WEM.

## **2. Connect to TFS in Visual Studio**

* Connect to TFS in `Team Explorer` &gt; `Manage Connections` and follow the wizard. Use the following url when prompted: [http://tfs.wem.dev:8080/tfs](http://tfs.wem.dev:8080/tfs)
* In `Source Control Explorer` map the `Local Path` to `E:\TFS`.
* Navigate to `tfs.wem.dev\MainCollection\WemCore\Main`, right-click &gt; `Get Latest Version`.

## **3. Create databases**

WEM requires some database to be present on SQL Server installation.

> A script to completely initialize a fresh SQL Server installation can be found [here](/developer-setup/database-init).

The following SQL databases are expected to exist locally \(on a SQL Server 2016 installation\):

1. **WemActivityLog** - Logs the HTTP page requests and concurrent sessions of the WEM runtime websites.
2. **WemLog** - Logs errors, warings, info messages. This database is used by both the modeler and the runtime websites.
3. **WemModeler** - Contains all the modeler data \(modeler users, projects, flowcharts, etc.\)

You have to create these databases mannually. You also have to create following database logins:

| Username | Password | User mappings / server roles |
| :--- | :--- | :--- |
| wem\_user | wem | `db_owner` of `WemActivityLog`, `WemLog` and `WemModeler` |
| wem\_runtime\_user | wem\_runtime | n/a |
| wem\_runtime\_schema\_user | wem\_runtime\_schema | server role `sysadmin` |

The `DeveloperTool` will create the database schema for `WemModeler` \(the developer tool is explained in section 5\).

The schemas for `WemActivityLog` and `WemLog` have to be created manually with the following database scripts:

```text
$/WemCore/Main/SQL/ZoomBim.Wem.Runtime.ActivityLogServer/Database definition.sql
$/WemCore/Main/SQL/ZoomBim.Utilities.Logging/Log DB table definitions.sql
```

## 4. WEM Master Certificate

You have to install a certificate for WEM to run. This certficate is stored in TFS on the following location:

```text
$/WemCore/Main/Certificate/wem_master_certificate.pfx
```

To install this certificate, perform the following steps:

1. Right-click on the `.pfx` file and choose **Install PFX**.
2. Choose the store location **Local Machine** and click _Next_ twice.
3. Enter the password `ZoomB1m!` and click _Next_.
4. Choose "Place all certificates in the following store" and select the folder "Personal". Click _Next_.
5. Click _Finish_ to complete the installation.

Or open a command prompt in Admin mode and run the following command:

```text
CERTUTIL -f -p ZoomB1m! -importpfx "E:\TFS\WemCore\Main\Certificate\wem_master_certificate.pfx"
```

> Note that this certificate is for development purposes only. This certificate is not used in the test nor the production environment. It is safe to store this certficate in source control.

## 5. Install windows services

WEM requires four windows services to run. These services can be installed on the following locations:

| Location | Project | Description |
| :--- | :--- | :--- |
| E:\Wem\Services\ActivityLogServer | ZoomBim.Wem.Runtime.ActivityLogServer | Counts page requests and concurrent sessions |
| E:\Wem\Services\FileServer | ZoomBim.Wem.Runtime.FileServer | Manages temporary files for the runtime |
| E:\Wem\Services\MailServer | ZoomBim.Wem.Runtime.MailServer | Sends mail messages for the runtime |
| E:\Wem\Services\SessionServer | ZoomBim.Wem.Runtime.SessionServer | Manages sessions for the runtime |

Before you can install and start these windows services, perform the following steps:

1. Open the `WemCore` solution and rebuild the entire solution \(debug build\).
2. Copy the contents of the `bin\debug` folders of the projects above to their corresponding directories within `E:\Wem\Service`.

The services use the library `TopShelf`, which means that the services can be installed via the command line.

To install the services, open a Visual Studio command prompt in Admin mode and run the following commands:

```text
cd /d "E:\wem\services\ZoomBim.Wem.Runtime.ActivityLogServer"
ZoomBim.Wem.Runtime.ActivityLogServer.exe install
ZoomBim.Wem.Runtime.ActivityLogServer.exe start

cd /d "E:\wem\services\ZoomBim.Wem.Runtime.FileServer"
ZoomBim.Wem.Runtime.FileServer.exe install
ZoomBim.Wem.Runtime.FileServer.exe start

cd /d "E:\wem\services\ZoomBim.Wem.Runtime.MailServer"
ZoomBim.Wem.Runtime.MailServer.exe install
ZoomBim.Wem.Runtime.MailServer.exe start

cd /d "E:\wem\services\ZoomBim.Wem.Runtime.SessionServer"
ZoomBim.Wem.Runtime.SessionServer.exe install
ZoomBim.Wem.Runtime.SessionServer.exe start
```

## 6. Run the DeveloperTool

The WEM solution contains the console application `DeveloperTool`. Set this application as startup project and start the tool with `F5` \(Alternative: right klick in Solution Explorer-Debug-Run Instance\). Choose the option `x` to create the database schemas for the modeler, preview and live runtime websites. It will also \(re\)create some test users and test projects.

> If you don't already have a test user account for the develpment environment, you can modify the method `Create()` of the class `ZoomBim.Wem.DeveloperTool.TestEnvironmentCreator` by adding an account for yourself. Make sure to make your account a global administrator. You can checkin this change under work item 1374: "_Add a new user to the developer tool_".

## 7. Android development

The TFS branch `$/WemCore/Mobile Main` contains the mobile runtime. This VS solution has some extra preqequisites:

1. Check the Visual Studio installer to make sure that **Mobile development with .NET** is enabled. 
2. In Visual Studio, go to **Tools &gt; Android &gt; Android SDK Manager** and make sure that at least the following SDK versions are installed:
   * Android 8.1 - Oreo \(API Level 27\)
   * Android 8.0 - Oreo \(API Level 26\)
   * Android 5.0 - Lollipop \(API Level 21\)
3. In Visual Studio, go to **Tools &gt; Android &gt; Android SDK Manager** and open the tab **Tools**. Make sure that version **26.1.1** of the Android SDK Tools is selected.
4. Visit the \([https://docs.microsoft.com/en-us/xamarin/android/get-started/installation/android-emulator/xamarin-device-manager?tabs=vswin\)\(Xamarin](https://docs.microsoft.com/en-us/xamarin/android/get-started/installation/android-emulator/xamarin-device-manager?tabs=vswin%29%28Xamarin) Android Device Manager\) page to download the latest Xamarin Android Device Manager.

## **8. \(Optional\) Extras**

ReSharper extension: ForTea \(Can be installed with the "Extension Manager" of ReSharper in Visual Studio\) \([https://github.com/MrJul/ForTea](https://github.com/MrJul/ForTea)\)  
 Adds support for T4 \(.tt\) files such as syntax highlighting and symbol navigation.

Visual Studio extension: Productivity Power Tools \([https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.ProductivityPowerPack2017\)\(Updated](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.ProductivityPowerPack2017%29%28Updated) for VS 2017\)  
 For better overview of build errors in the solution, file tabs color linked to project, and more.

Window title: \([https://marketplace.visualstudio.com/items?itemName=mayerwin.RenameVisualStudioWindowTitle](https://marketplace.visualstudio.com/items?itemName=mayerwin.RenameVisualStudioWindowTitle)\)  
 Change Visual Studio window title for working with multiple instances on different branches.

