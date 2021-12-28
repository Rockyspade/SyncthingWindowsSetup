<!-- omit in toc -->
# Syncthing Windows Setup

Syncthing Windows Setup is a lightweight yet full-featured Windows installer for the open-source [Syncthing](https://syncthing.net) file-synchronization application.

---

<!-- omit in toc -->
## Table of Contents

- [Download](#download)
- [Background](#background)
- [Setup Command Line Parameters](#setup-command-line-parameters)
- [Administrative vs. Non Administrative Installation Mode](#administrative-vs-non-administrative-installation-mode)
  - [Administrative (All Users) Installation Mode](#administrative-all-users-installation-mode)
  - [Non Administrative (Current User) Installation Mode](#non-administrative-current-user-installation-mode)
- [Windows Service Installation](#windows-service-installation)
- [Granting Folder Permissions for the Service Account](#granting-folder-permissions-for-the-service-account)
- [Managing Automatic Upgrades](#managing-automatic-upgrades)
- [Start Menu Shortcuts](#start-menu-shortcuts)
- [Checking If Syncthing Is Running](#checking-if-syncthing-is-running)
- [Windows Firewall Rules](#windows-firewall-rules)
  - [Firewall Rule Creation](#firewall-rule-creation)
  - [Creating the Firewall Rule Manually](#creating-the-firewall-rule-manually)
  - [Firewall Rule Removal](#firewall-rule-removal)
  - [Removing the Firewall Rule Manually](#removing-the-firewall-rule-manually)
- [Scripts](#scripts)
- [Finding the Syncthing Configuration Folder](#finding-the-syncthing-configuration-folder)
- [Uninstalling Syncthing](#uninstalling-syncthing)
- [Silent Install and Uninstall](#silent-install-and-uninstall)
  - [Silent Administrative (All Users) Installation](#silent-administrative-all-users-installation)
  - [Silent Non Administrative (Current User) Installation](#silent-non-administrative-current-user-installation)
  - [Silent Uninstall](#silent-uninstall)
- [Acknowledgments](#acknowledgments)

---

## Download

You can download the latest version from the Github Releases page:

https://github.com/Bill-Stewart/SyncthingWindowsSetup/releases/

## Background

Syncthing Windows Setup (herein referred to as "Setup") provides a [Syncthing](https://syncthing.net/) installer for Windows, built using [Inno Setup](https://www.jrsoftware.org/isinfo.php). It provides the following features:

* Installs the appropriate 32-bit or 64-bit version of Syncthing using a single installer

* Supports administrative (all users) and non administrative (current user) installation (see [Administrative vs. Non Administrative Installation Mode](#administrative-vs-non-administrative-installation-mode))

* When installing for all users, installs Syncthing as a Windows service using [NSSM](https://nssm.cc) (see [Windows Service Installation](#windows-service-installation))

* When installing for the current user, creates a scheduled task that starts Syncthing in a hidden window at logon

* Allows configuration of GUI configuration page listen IP address and TCP port number during installation

* Supports adding a Windows Firewall rule for Syncthing (see [Windows Firewall Rules](#windows-firewall-rules))

* Installs a set of scripts for ease-of-use (see [Scripts](#scripts))

* Supports silent (hands-free) installation (see [Silent Install and Uninstall](#silent-install-and-uninstall))

* Allows localization of Setup and scripts (see `building.md` file for details)

## Setup Command Line Parameters

The following table lists the most common Setup command line parameters:

Parameter                  | Description
---------                  | -----------
`/allusers`                | Runs Setup in administrative (all users) installation mode (see [Administrative vs. Non Administrative Installation Mode](#administrative-vs-non-administrative-installation-mode)).
`/currentuser`             | Runs Setup in non-administrative (current user) installation mode (see [Administrative vs. Non Administrative Installation Mode](#administrative-vs-non-administrative-installation-mode)).
`/dir="`_location_`"`      | Specifies the installation folder. The default installation folder depends on whether Setup runs in administrative (all users) or non administrative (current user) installation mode.
`/group="`_name_`"`        | Specifies the Start Menu group name. The default group name is **Syncthing**.
`/noicons`                 | Prevents creation of a Start Menu group.
`/silent`                  | Runs Setup without requiring user interaction. (You must also specify either `/allusers` or `/currentuser` for a fully hands-free silent installation.)
`/log="`_filename_`"`      | Logs Setup activity to the specified file. The default is not to create a log file.
`/listenaddress=`_address_ | Specifies the listen address for the web GUI configuration page. The default listen address is `127.0.0.1`.
`/listenport=`_port_       | Specifies the TCP port number for the web GUI configuration page. The default port number is `8384`.
`/nostart`                 | Prevents Syncthing from starting automatically after the installation completes. The default is to start Syncthing when installation completes.

> NOTE: Please read the [Syncthing documentation page about the GUI listen address](https://docs.syncthing.net/users/guilisten.html) before changing the listen address and port numbers from the defaults.

See [Inno Setup's documentation](https://jrsoftware.org/ishelp/index.php?topic=setupcmdline) for more details about the above parameters (except for the `/listenaddress` and `/listenport` parameters, which are custom).

## Administrative vs. Non Administrative Installation Mode

Setup supports both administrative (all users) and non administrative (current user) installation modes. Setup will display a dialog box requesting whether you want to install for all users (administrative installation mode) or for the current user (non administrative installation mode). You can bypass the dialog by specifying either `/allusers` or `/currentuser` on Setup's command line (see [Setup Command Line Parameters](#setup-command-line-parameters)).

The main advantage of installing in administrative (all users) installation mode is that Syncthing runs as a Windows service and runs without any users being logged on; however, you must manually configure folder permissions to add folders to the Syncthing configuration (see [Granting Folder Permissions for the Service Account](#granting-folder-permissions-for-the-service-account)).

See below for the differences between the two modes.

### Administrative (All Users) Installation Mode

The following notes apply to administrative (all users) installation mode:

* Setup installs Syncthing for all users of the computer

* The default installation folder is _ProgramFiles_`\Syncthing` (where _ProgramFiles_ is the system's `Program Files` folder)

* Setup installs Syncthing as a Windows service (see [Windows Service Installation](#windows-service-installation))

* Syncthing starts automatically when the system boots

* Syncthing runs when no users are logged on

* Starting and stopping Syncthing is managed by stopping and starting the Windows service

* Setup automatically creates a Windows firewall rule

* By default, Setup starts the Syncthing service after installation completes (you can change this by deselecting the checkbox on the last Setup wizard page or by specifying the `/nostart` parameter on Setup's command line)

* You must manually grant folder permissions for folders you want to add to the Syncthing configuration (see [Granting Folder Permissions for the Service Account](#granting-folder-permissions-for-the-service-account))

* Administrative permissions are required to make changes to files in the Syncthing configuration folder

* Setup does not add any configuration data to the registry

### Non Administrative (Current User) Installation Mode

The following notes apply to non administrative (current user) installation mode:

* Setup installs Syncthing for the current user only

* The default installation folder is _UserProfile_`\AppData\Local\Programs\Syncthing` (where _UserProfile_ is the current user's profile folder)

* Setup does not install Syncthing as a Windows service

* Setup creates a scheduled task that starts Syncthing in a hidden window when the current user logs on

* Syncthing runs only when the installing user logs on

* Starting and stopping Syncthing is managed by [Start Menu shortcuts](#start-menu-shortcuts)

* Setup prompts to create a Windows firewall rule

* By default, Setup starts Syncthing after installation completes if the firewall rule exists (you can change this by deselecting the checkbox on the last Setup wizard page or by specifying the `/nostart` parameter on Setup's command line)

* No special folder permissions are required to add folders to the Syncthing configuration

* Administrative permissions are not required to make changes to files in the Syncthing configuration folder

* Setup adds configuration data to the `HKEY_CURRENT_USER\Software\Syncthing` registry subkey

## Windows Service Installation

When you run Setup in administrative (all users) installation mode, it installs a Windows service for Syncthing using [NSSM](https://nssm.cc). Setup configures the service settings as follows:

NSSM Setting       | Value                     | Description
------------       | -----                     | -----------
Startup type       | Automatic (Delayed Start) | Prevents startup contention
Logon account      | `LOCAL SERVICE`           | Limited account with sufficient permissions
Process priority   | Below normal              | Prevents excessive CPU usage
Console window     | Disabled                  | Do not create a console window
Shutdown timeouts  | 10000 ms                  | Provides sufficient time for Syncthing processes to exit
Exit code 0 action | Exit                      | Syncthing normal exit code: Stop service
Exit code 3 action | Restart                   | Syncthing restart exit code: Restart service
Exit code 4 action | Restart                   | Syncthing upgrade exit code: Restart service

The Windows service uses a command line like the following to run Syncthing:

`"`_InstallFolder_`\syncthing.exe" --no-browser --no-restart --gui-address="`_ListenAddress_`:`_ListenPort_`"`

Where:

* _InstallFolder_ is the Syncthing installation folder
* _ListenAddress_ is the GUI configuration page listen address
* _ListenPort_ is the GUI configuration page listen port

The **Configure Syncthing Service** shortcut (see [Start Menu Shortcuts](#start-menu-shortcuts)) lets you modify the service configuration using NSSM (recommended for advanced users only).

## Granting Folder Permissions for the Service Account

If you install in administrative (all users) installation mode, Syncthing runs using the `LOCAL SERVICE` account. Normally the `LOCAL SERVICE` account does not have permissions to folders you want to synchronize using Syncthing.

You can grant the `LOCAL SERVICE` account modify permissions to a folder using the Windows File Explorer. Alternatively, you can run the `icacls` command from the command line; e.g.:

    icacls "C:\Users\MyUserName\Documents" /grant "*S-1-5-19:(OI)(CI)M" /t

(Replace `C:\Users\MyUserName\Documents` with the correct folder name)

Once the `LOCAL SERVICE` account has modify permissions for the folder, you can add it to the Syncthing configuration.

> NOTE: Granting the `LOCAL SERVICE` account permission to folders is only needed if you installed Syncthing in administrative (all users) installation mode.

## Managing Automatic Upgrades

On the **Select Additional Tasks** wizard page, the **Allow Syncthing to upgrade automatically** checkbox determines whether Syncthing will attempt to automatically upgrade itself. If you uncheck this checkbox, Syncthing will run with the `--no-upgrade` command-line option and will not attempt to automatically upgrade itself.

You can disable automatic upgrades by default by specifying `/tasks="!allowautoupgrade"` on Setup's command line.

The **Select Additional Tasks** page does not appear when you are installing in administrative (all users) installation mode if the Windows service is already installed. If you want to change the automatic upgrade configuration for the Windows service after installation has completed, you can do either of the following:

* Uninstall and reinstall Syncthing (no Syncthing configuration settings are lost), or

* Configure the Windows service (see [Start Menu Shortcuts](#start-menu-shortcuts)) and either add or remove the `--no-upgrade` parameter from the **Arguments** field on the **Application** tab in the NSSM configuration.

## Start Menu Shortcuts

Setup creates the following Start Menu shortcuts, depending on the [installation mode](#administrative-vs-non-administrative-installation-mode):

Shortcut                     | Installation Mode | Description
--------                     | ----------------- | -----------
FAQ                          | Both              | Opens the 'FAQ' PDF document
Getting Started              | Both              | Opens the 'Getting Started' PDF document
Syncthing Configuration Page | Both              | Opens the Syncthing configuration page using the default browser
Configure Syncthing Service  | All users         | Allows configuration of the Windows service using NSSM
Start Syncthing              | Current user      | Starts Syncthing for the current user in a hidden window
Stop Syncthing               | Current user      | Stops Syncthing

* The **Syncthing Configuration Page** shortcut opens the `ConfigurationPage.url` file in the Syncthing installation folder (i.e., it opens the Syncthing GUI configuration page).

* The **Configure Syncthing Service** runs the `ConfigSyncthingService.js` script (see [Scripts](#scripts)) to allow modification of the Windows service configuration (recommended for advanced users only). This shortcut is installed only in administrative (all users) installation mode.

* The **Start Syncthing** and **Stop Syncthing** shortcuts run the `StartSyncthing.js` and `StopSyncthing.js` scripts, respectively (see [Scripts](#scripts)). These shortcuts are installed only in non administrative (current user) mode.

## Checking If Syncthing Is Running

To check if Syncthing is running, run the following command from a command prompt or PowerShell window:

    tasklist /fi "imagename eq syncthing.exe"

* If the **Session Name** column in the output is **Services**, then Syncthing is running as a Windows service.

* If the **Session Name** column in the output is **Console**, then Syncthing is not running as a Windows service.

## Windows Firewall Rules

Syncthing requires permission to communicate through the Windows firewall. Creating and removing firewall rules requires administrative privileges.

### Firewall Rule Creation

* If you run Setup in administrative (all users) installation mode, Setup will create the firewall rule automatically, without prompting.

* If you run Setup in non administrative (current user) installation mode, Setup will prompt to create the firewall rule if it doesn't exist (requires administrative permissions).

* When you use the `/currentuser` and `/silent` parameters (see [Setup Command Line Parameters](#setup-command-line-parameters)), Setup will not create the firewall rule, and you will need to create it manually.

### Creating the Firewall Rule Manually

To create the firewall rule manually, open a PowerShell or command prompt window and use a command like the following:

    cscript "C:\Users\username\AppData\Local\Programs\Syncthing\SyncthingFirewallRule.js" /create

(where `C:\Users\username\appData\Local\Programs\Syncthing` is the Syncthing installation folder)

### Firewall Rule Removal

If you uninstall Syncthing (see [Uninstalling Syncthing](#uninstalling-syncthing)), the same considerations as above apply, except Setup removes the firewall rule rather than creating it:

* If you uninstall for all users, the uninstall process will remove the firewall rule automatically, without prompting.

* If you uninstall for the current user, the uninstall process will prompt to remove the firewall rule if it exists (requires administrative permissions).

* If you uninstall for the current user using `/silent`, the uninstall process will not remove the firewall rule, and you will need to remove it manually.

### Removing the Firewall Rule Manually

To remove the firewall rule manually, open a PowerShell or command prompt window and use a command like the following:

    cscript "C:\Users\username\AppData\Local\Programs\Syncthing\SyncthingFirewallRule.js" /remove

(where `C:\Users\username\appData\Local\Programs\Syncthing` is the Syncthing installation folder)

## Scripts

Setup installs a set of scripts to the installation folder to facilitate ease-of-use, depending on the [installation mode](#administrative-vs-non-administrative-installation-mode), as described in the following table.

Script                      | Installation Mode | Description
------                      | ----------------- | -----------
`SyncthingFirewallRule.js`  | Both              | Adds, removes, and tests for the existence of a Windows firewall rule for Syncthing (prompts for administrative permissions if required)
`ConfigSyncthingService.js` | All users         | Configures the Syncthing Windows service using NSSM (prompts for administrative permissions if required)
`SyncthingLogonTask.js`     | Current user      | Adds or removes a scheduled task that runs the `StartSyncthing.js` script at logon
`StartSyncthing.js`         | Current user      | Starts Syncthing for the current user using "below normal" process priority in a hidden window
`StopSyncthing.js`          | Current user      | Stops Syncthing

## Finding the Syncthing Configuration Folder

The location of the Syncthing configuration folder depends on whether you run Setup in administrative (all users) or non administrative (current user) installation mode.

* If you install for all users, the Syncthing configuration folder is in the following location:

    _WindowsFolder_`\ServiceProfiles\LocalService\AppData\Local\Syncthing`

  where: _WindowsFolder_ is the operating system folder (e.g., `C:\Windows`)

  You will need administrator permissions to access the Syncthing configuration folder if you run the Windows service.

* If you install for the current user, the Syncthing configuration folder is in the following location:

    _UserProfileFolder_`\AppData\Local\Syncthing`

  where: _UserProfileFolder_ is the current user's profile folder (e.g., `C:\Users\UserName`)

## Uninstalling Syncthing

* You can uninstall Syncthing by using the standard Windows application management list. The uninstall process does not remove any files Setup did not install.

* If you installed Syncthing for the current user, note the following:

  * If not uninstalling silently, the uninstall process will prompt to remove the firewall rule if it exists (this will prompt for administrative credentials if required)

  * If uninstalling silently, the uninstall process will not prompt to remove the firewall rule if it exists (the rule will have to be removed manually)

* See [Inno Setup's documentation](https://jrsoftware.org/ishelp/index.php?topic=uninstcmdline) for more uninstaller command line parameters.

> NOTE: Uninstalling Syncthing does not remove any Syncthing configuration files. If you want to remove the Syncthing configuration folder, determine the location of the Syncthing configuration folder (see [Finding the Syncthing Configuration Folder](#finding-the-syncthing-configuration-folder)) and remove it after uninstalling.

## Silent Install and Uninstall

Setup supports silent (hands-free) install and uninstall mode using the `/silent` command line parameter.

### Silent Administrative (All Users) Installation

To install silently in administrative installation (all users) mode, specify both the `/allusers` and `/silent` command line parameters on Setup's command line. In this mode:

* Setup installs and starts the Windows service
* Setup creates a firewall rule automatically
* Setup starts the Syncthing service after installation completes (unless you also specify `/nostart` on Setup's command line)

### Silent Non Administrative (Current User) Installation

To install silently in non administrative (current user) installation mode, specify both the `/currentuser` and `/silent` command line parameters on Setup's command line. In this mode:

* Setup does not install the Windows service
* Setup does not create a firewall rule
* Setup starts Syncthing for the current user if the firewall rule is in place (unless you also specify `/nostart` on Setup's command line)

To ensure Syncthing works correctly after a non administrative (current user) silent installation, create the firewall rule manually (see [Creating the Firewall Rule Manually](#creating-the-firewall-rule-manually)) before starting Syncthing.

### Silent Uninstall

To uninstall silently, specify `/silent` on the uninstaller's command line (the uninstaller executable is located in the `uninstall` directory inside the Syncthing installation folder).

If you installed Syncthing for the current user, you will need to remove the firewall rule manually (see [Removing the Firewall Rule Manually](#removing-the-firewall-rule-manually)) before uninstalling.

## Acknowledgments

Special thanks to the following:

* Syncthing maintainers

* Jordan Russell and Martijn Laan for [Inno Setup](https://www.jrsoftware.org/isinfo.php)