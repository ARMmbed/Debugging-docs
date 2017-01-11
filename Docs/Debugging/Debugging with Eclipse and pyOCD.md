#Debugging with Eclipse and pyOCD

Through debugging, you can do things such as set breakpoints, set watchpoints, view registers, view disassembly, browse memory and examine the callstack. This article explains how to debug an exported mbed project with Eclipse and pyOCD. To debug a project, first, import it with mbed CLI, then create an Eclipse Makefile project and finally set up debugger configurations. You can find more detailed explanations of these steps below.

##Initial setup
1. Download the latest version of Eclipse with CDT.
1. Install the GNU ARM Eclipse plugin.
	1. Open Eclipse.
	1. Click the **Help** menu item and select **Install New Software**.
	1. In the **Work with** box, paste the install address and press Enter: http://sourceforge.net/projects/gnuarmeclipse/files/Eclipse/updates/. If this does not work for you, please see the [GNU ARM Eclipse solutions and workarounds](http://gnuarmeclipse.github.io/blog/2016/12/02/plugins-install-issue/).
  1. The package **GNU ARM C/C++ Cross Development Tools** appears. Select it.
  1. Click **Next** repeatedly and accept licence agreements.
  1. Click **Finish**. If prompted to restart Eclipse, click **Yes**.
1. [Install tools necessary for Makefile projects](https://docs.mbed.com/docs/mbed-os-handbook/en/5.3/dev_tools/third_party/):
  1. [GCC ARM Embedded](https://launchpad.net/gcc-arm-embedded).
    1. Windows only: add the GCC ARM Embedded's bin directory. For example, `C:\Program Files (x86)\GNU Tools ARM Embedded\4.9 2015q2\bin`.
  1. Python
    1. Windows Only: select "Add to path" when installing.
1. Install pyOCD by running `pip install pyocd`.
1. Install mbed CLI by running `pip install mbed-cli`.

##Importing to Eclipse and building
1. Import the project with `mbed <project>`.
1. Open Eclipse.
1. **File** > **New** > **Makefile Project with Existing Code**.
  1. Paste the location of your project into **Existing Code Location**.
  1. **Project Name** automatically populates. You can update the name here.
  1. In **Toolchain for Indexer Settings**, select `<none>`.
  1. Click **Finish** to create the project.
1. To set up building in Eclipse, go to the **Project** menu item, and select **Properties**.
  1. Select **C/C++ Build**.
  1. Clear the **Use default build command** check box.
  1. For **Build command**, enter **mbed compile -j0 -t <toolchain> -m <target> -o debug-info**.
  	- For **<toolchain>**, you can choose between **GCC_ARM**, **ARM** and **IAR**.
	- For **<target>**, you can choose any supported mbed target, such as **K64F**.
	- Note: The option **-j0** instructs mbed CLI with all cores.
	- Note: The option **--profile debug** instructs mbed CLI to build with debugging information.
  1. Click **Apply**. 
  1. Click **OK**.
  <span class="images">![](Images/Eclipse_pyOCD_1.webp)</span>
1. Click the build button in Eclipse, and verify the project builds successfully.

##Setting up debug configuration
The last step before actually debugging is to set up the debug configuration. This only needs to be done once each project. Once created, the configuration can be checked into revision control, allowing anyone else working on the project to use it without needing to go through setup again.

1. To change configurations, **Run** > **Debug Configurations...**.
1. Right click on **GDB pyOCD Debugging** and select **New**.
1. Move to the **Main** tab. 
1. Verify **Project** is correct and **C/C++ Application** points to the .elf file.
<span class="images">![](Images/Eclipse_pyOCD_2.webp)</span>
1. Move to the **Debugger** tab.
  1. In the **pyOCD Setup** section:
    1. Check **Start pyOCD locally** if it isn't checked already.
    1. Set **Executable:** to **pyocd-gdbserver** or the full path to your installation.
    1. Set **GDB port:** to a free port. The default value of 3333 should work.
  1. In the **GDB Client Setup** section:
    1. Set **Executable** to **arm-none-eabi-gdb.exe**.
    1. In **Commands:**, add **set mem inaccessible-by-default off** if it is not present. It should look something like this when configured: 
  <span class="images">![](Images/Eclipse_pyOCD_3.webp)</span>
  1. Move to the **Common** tab.
    1. To save the debug configuration locally and check it into the current project, select **Shared file** under **Save as**.
    1. In **Display in favorites menu box**, check **Debug**.

##Debugging
To start a debugging session, open the drop-down menu next to the bug icon and select the debug configuration created earlier. This will automatically load the current program and begin a debugging session. 

<span class="images">![](Images/Eclipse_pyOCD_4.webp)</span>
