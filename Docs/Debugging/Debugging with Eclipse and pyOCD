#Debugging with Eclipse and pyOCD

This article explains how to debug an exported mbed project with Eclipse and pyOCD. To debug a project, you need to first export it from mbed as a makefile project, import the makefile project into Eclipse and finally setup debugger configurations. Below are these steps but in much more detail.

##Initial Setup
1. Download the latest version of eclipse with CDT.
1. Install the GNU ARM Eclipse plugin.
	1. Open Eclipse.
	1. Click the **Help** menu item and select **Install New Software**.
	1. In the **Work with** box paste the install address and press enter - [http://redmine.laoslaser.org/boards/3/topics/635](http://sourceforge.net/projects/gnuarmeclipse/files/Eclipse/updates/).
  1. The package **GNU ARM C/C++ Cross Development Tools** should appear. Select it.
  1. Click **Next** repeatedly and accept licence agreements.
  1. Click **Finish**. If prompted to restart Eclipse, select **Yes**.
1. [Install tools necessary for makefile projects](https://developer.mbed.org/handbook/Exporting-to-Make):
  1. [GCC ARM Embedded](https://launchpad.net/gcc-arm-embedded).
  1. Windows only
    1. Add GCC ARM Embedded bin directory to path after installing - Ex. C:\Program Files (x86)\GNU Tools ARM Embedded\4.9 2015q2\bin.
    1. Install **make** and add it to path. Make is available as one of the tools here: [http://gnuwin32.sourceforge.net/](http://gnuwin32.sourceforge.net/).
1. Install python 2.7.9 or above
  1. Windows Only - Make sure to select "Add to path" when installing.
1. Install pyOCD by running "pip install pyocd".

##Importing to Eclipse and building
1. In the mbed online compiler, export the project you want to debug as **GCC (Arm Embedded)**.
1. Extract the project archive exported from mbed.
1. Open Eclipse.
1. Click the **File** menu item, mouse over **New** and select **Makefile Project with Existing Code**.
  1. Paste the location of your project into **Existing Code Location**.
  1. **Project Name** should automatically populate. Update name here if desired.
  1. In **Toolchain for Indexer Settings**, select **Cross ARM GCC**.
  1. Click **Finish** to create the project.
1. To set up building in Eclipse, go to the **Project** menu item, and select **Properties**.
  1. Select **C/C++ Build**.
  1. Unselect **Use default build command**.
  1. For **Build command**, enter **make DEBUG=1**. The option "DEBUG=1" configures the make file to build optimized code, which makes debugging easier.
  1. Click **Apply**, and then click **OK**.
  <span class="images">![](Images/30_-_build_config.png)</span>
1. Click the build button in Eclipse, and verify the project builds successfully.

##Setting up debug configuration
The last step before actually debugging is to set up the debug configuration. This only needs to be done once for per project. Once created, the configuration can be checked into revision control, allowing anyone else working on the project to use it without needing to go through setup again.

1. On the **Run** menu item, select **Debug Configurations...** to bring up configurations.
1. Right click on **GDB pyOCD Debugging** and select **New**.
1. Open **Main** tab, and verify **Project** is correct and **C/C++ Application** correctly points to the .elf file.
<span class="images">![](Images/10_-_debug_main_3.png)</span>
1. Open **Debugger** tab.
  1. In the **pyOCD Setup** box:
    1. Check **Start pyOCD locally** if it isn't checked already.
    1. Set **Executable:** to **pyocd-gdbserver** or the full path to your installation.
    1. Set **GDB port:** to a free port. The default value of 3333 should work.
  1. In the **GDB Client Setup** box:
    1. Set **Executable** to **arm-none-eabi-gdb.exe**.
    1. In **Commands:**, add **set mem inaccessible-by-default off** if it is not present.
  1. It should look something like this when configured: 
  <span class="images">![](Images/11_-_debug_debugger_2.png)</span>
  1. Defaults will work in **Startup** tab.
  1. Defaults will work in **Source** tab.
  1. Open **Common** tab.
    1. Under **Save as**, select **Shared file**. The default name will work. This allows the debug configuration to be saved locally and checked into the current project.
    1. In **Display in favorites menu box**, check **Debug**.

##Debugging
To start a debugging session, open the drop-down menu next to the bug icon, and select the debug configuration created earlier. This will automatically load the current program and begin a debugging session. From there, you can do things such as set breakpoints, set watchpoints, view registers, view disassembly, browse memory and examine the callstack.

<span class="images">![](Images/21_-_debugging_4.png)</span>
______
Copyright © 2016 ARM Ltd. All rights reserved.
