.. _dev-client-winrt:

.. role:: raw-html(raw)
   :format: html

Setting Up a Development Game Client (Windows 8.1+)
==================================================

.. contents::
   :local:

Introduction
------------

Welcome! This guide will help you set up a Brave Frontier game client on your Windows 8.1 or later computer to play offline using a local proxy server. Even if you're new to using a command-line interface (CLI) or programming, we'll walk you through each step with pictures and simple instructions. You'll need Visual Studio 2022, some software tools, and a special game file. Let’s get started!

Requirements
------------

Before you begin, make sure you have the following installed and ready:

- **Windows 8.1 or later** (check your system in Settings > System > About).
- **Visual Studio 2022** with the "Desktop development with C++" workload and Windows 10 SDK. If you don’t have it:
  - Download from https://visualstudio.microsoft.com/downloads/, install, and select the workload during setup.
- **Microsoft Visual C++ Runtime Package 12.0 for x86**: Get it from https://github.com/M1k3G0/Win10_LTSC_VP9_Installer/blob/master/Microsoft.VCLibs.120.00_12.0.21005.1_x86__8wekyb3d8bbwe.appx and install by double-clicking.
- **A copy of the Brave Frontier APPX file**: Download it from https://drive.google.com/file/d/1NB64gzQOe-QQx9fY0mkoZiCSfe3WlTYi/view?usp=sharing (save as `C:\BF\gumi.BraveFrontier_2.19.6.0_x86__tdae4wqex79w6.appx`).
- **Git for Windows**: Download from https://git-scm.com/download/win, install with default settings.
- **Developer Mode enabled**: Go to Settings > Update & Security > For developers, turn on "Developer Mode".

.. warning::
   If Developer Mode is not enabled, the proxy won’t work, and you won’t see the command prompt. Enable it now if it’s off!

.. note::
   You’ll need administrator access for some steps. Right-click PowerShell and select "Run as administrator" when prompted.

Cloning the Repository
----------------------

This step downloads the proxy code. Follow these easy steps:

1. Open **PowerShell** (search "PowerShell" in the Windows Start menu and click it).
2. Type this command and press Enter:

   .. code-block:: console

      git clone --depth=1 https://github.com/decompfrontier/offline-proxy
      cd C:\BF\offline-proxy

   - **What this does**: `git clone` copies the code to your computer, and `cd` moves you to the `C:\BF\offline-proxy` folder. If you saved it elsewhere, adjust the path (e.g., `cd C:\YourFolder\offline-proxy`).
   - **Tip**: If you see an error about "git not recognized," reinstall Git and try again.

.. image:: ../../images/dev-client-winrt/clone_repo.png
   :alt: Cloning the Repository in PowerShell

Building the Proxy
------------------

Now, let’s build the proxy, which is like preparing a tool to help the game run offline. Here’s how:

1. In the same **PowerShell** window, type this command and press Enter:

   .. code-block:: console

      cmake --build . --config Debug

   - **What this does**: This command builds the proxy. The `.` means "in the current folder," and `Debug` makes a version for testing.
   - **Wait a moment**: You might see text scrolling—let it finish.

2. If it fails or you’re unsure, open **Visual Studio 2022**:
   - Double-click `offline-proxy.sln` in `C:\BF\offline-proxy`.
   - At the top, change "Any CPU" to "Win32" and "Release" to "Debug" (use the dropdown menus).
   - Press **F7** or click "Build" > "Build Solution".

3. After success, find `libcurl.dll` in `C:\BF\offline-proxy\Debug\libcurl.dll` (search if needed).

.. warning::
   The client only supports 32-bit (Win32) platforms. Make sure your build targets Win32, or it won’t work!

.. note::
   If you see errors, ensure Visual Studio’s C++ tools are installed. Re-run the Visual Studio Installer to add them if missing.

Generating UWP Development Certificates
---------------------------------------

This step creates a digital "key" to sign your game. Don’t worry—it’s simple!

1. Open **PowerShell** as administrator (right-click > "Run as administrator").
2. Type these commands one by one, pressing Enter after each:

   .. code-block:: powershell

      $certName = "MyBraveFrontier"
      $friendlyName = "Brave Frontier Dev Cert"
      New-SelfSignedCertificate -Type Custom -Subject "CN=$certName" -KeyUsage DigitalSignature -FriendlyName "$friendlyName" -CertStoreLocation "Cert:\CurrentUser\My" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3", "2.5.29.19={text}")

   - **What this does**: Creates a certificate. Look for a Thumbprint (e.g., `ABC123...`) in the output—write it down!

3. Export the certificate:

   .. code-block:: powershell

      $thumbprint = "ABC123..."  # Replace with your Thumbprint
      $password = ConvertTo-SecureString -String "YourStrongPassword" -Force -AsPlainText  # Choose a password
      Export-PfxCertificate -cert "Cert:\CurrentUser\My\$thumbprint" -FilePath C:\BF\MyKey.pfx -Password $password

4. Install the certificate:
   - Double-click `C:\BF\MyKey.pfx`.
   - Click "Next," select "Local Machine," click "Next" again.
   - Choose "Trusted Root Certification Authorities," click "Next," then "Finish," and say "Yes" to the warning.

.. warning::
   Remove these certificates later (via certmgr.msc) when done to keep your system safe.

.. important::
   Use PowerShell as administrator for these steps.

Obtaining the APPX File
-----------------------

The game file (APPX) is needed to start. Here’s how to get it:

- Go to: https://drive.google.com/file/d/1NB64gzQOe-QQx9fY0mkoZiCSfe3WlTYi/view?usp=sharing
- Right-click the download button, select "Save As," and save it as `C:\BF\gumi.BraveFrontier_2.19.6.0_x86__tdae4wqex79w6.appx`.
- Check the file size (~100MB) to ensure it downloaded correctly.

.. note::
   This file isn’t widely available online. If the link fails, try extracting it from an installed app: Open PowerShell and run `Get-AppxPackage *BraveFrontier* | Export-AppxPackage -Path C:\BF\BraveFrontier.appx`.

Modifying Brave Frontier APPX
-----------------------------

Now, let’s prepare the game file with your proxy tool.

1. Open **Developer PowerShell for Visual Studio 2022** (search in Start menu).
2. Unpack the game file:

   .. code-block:: console

      makeappx unpack /p C:\BF\gumi.BraveFrontier_2.19.6.0_x86__tdae4wqex79w6.appx /d C:\BF\BraveFrontierAppxClient

   - **What this does**: Creates a folder `BraveFrontierAppxClient` with the game files.

3. Copy the proxy library:
   - In **File Explorer**, go to `C:\BF\offline-proxy\Debug`, find `libcurl.dll`, and copy it.
   - Go to `C:\BF\BraveFrontierAppxClient`, paste it, and click "Replace" if asked.

4. Delete extra files:
   - In **File Explorer**, go to `C:\BF\BraveFrontierAppxClient`.
   - Delete these folders/files: `AppxMetadata` (right-click > Delete), `AppxSignature.p7x`, `AppxBlockMap.xml`, `ApplicationInsights.config`.

5. Edit the manifest:
   - Open `C:\BF\BraveFrontierAppxClient\AppxManifest.xml` in Notepad (search "Notepad" in Start).
   - Find this line: `<Identity Name="gumi.BraveFrontier" Publisher="CN=5AA816A3-ED94-4AA2-A2B4-3ADDA1FABFB6" ... />`.
   - Change `CN=5AA816A3-ED94-4AA2-A2B4-3ADDA1FABFB6` to `CN=MyBraveFrontier`.
   - (Optional) Change `<DisplayName>Brave Frontier</DisplayName>` to `<DisplayName>Brave Frontier Offline</DisplayName>` under `<Properties>`.
   - Save and close.

.. image:: ../../images/dev-client-winrt/modify_appx.png
   :alt: Modifying the APPX Manifest

.. important::
   Use Developer PowerShell for these commands.

Packing and Signing the Modified Client
---------------------------------------

Let’s pack and sign your modified game file:

1. In **Developer PowerShell**, run:

   .. code-block:: console

      makeappx pack /d C:\BF\BraveFrontierAppxClient /p C:\BF\BraveFrontierPatched.appx
      SignTool sign /a /v /fd SHA256 /f C:\BF\MyKey.pfx /p "YourStrongPassword" C:\BF\BraveFrontierPatched.appx

   - **What this does**: Packs the files into a new `.appx` and signs it with your certificate.
   - Use the same password you chose earlier.

.. note::
   If you see an error, double-check the password and ensure `MyKey.pfx` is in `C:\BF`.

Running the Game
----------------

Time to install and play!

1. Open **PowerShell** (as administrator).
2. Install the game:

   .. code-block:: powershell

      Add-AppxPackage C:\BF\BraveFrontierPatched.appx

3. Enable loopback (so the game talks to your local server):
   - Download the Enable Loopback Utility: https://telerik-fiddler.s3.amazonaws.com/fiddler/addons/enableloopbackutility.exe
   - Double-click to run it.
   - Select "Brave Frontier" from the list, check "Enable loopback," and click "Save Changes".
   - If it errors, go to Settings > Update & Security > For developers, enable Device Portal, and uncheck "Restrict to loopback connections only".

.. image:: ../../images/dev-client-winrt/loopback_win.png
   :alt: Loopback Utility Configuration

4. Launch the game:
   - Search "Brave Frontier" in the Start menu and click it.
   - A black console window should pop up with the game.

.. image:: ../../images/dev-client-winrt/bf_appx_patched.png
   :alt: Running the Patched Game Client

.. warning::
   If no console appears:
   - Check that `libcurl.dll` was copied correctly.
   - Ensure Developer Mode is on.
   - Avoid using the `deploy` preset during build.

Connecting to the Server
~~~~~~~~~~~~~~~~~~~~~~~~

Make sure your development game server (from `Setting Up a Development Game Server <dev-server.html>`_) is running on `127.0.0.1:9960`. With loopback enabled, launch the game, and you should see the Brave Frontier login screen!

Troubleshooting
---------------

- **Build Fails**: Ensure Visual Studio C++ tools are installed. Re-run the Visual Studio Installer to add them.
- **Unpack Error**: Check the APPX path and use Developer PowerShell.
- **Signing Error**: Verify `MyKey.pfx` is installed and the password is correct.
- **No Console**: Confirm `libcurl.dll` replacement and Developer Mode.
