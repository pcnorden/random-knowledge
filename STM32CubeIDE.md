Error codes:

# `Could not determine GDB version using command:`

If STM32CubeIDE throws something that says like
> Launching \<PROJECT NAME HERE\> has enountered a problem.
> Could not determine GDB version using command: arm-none-eabi-gdb --version

With the details

> Could not determine GDB version using command: arm-none-eabi-gdb --version
> 	arm-none-eabi-gdb: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file of directory

Well, here is my fix for the problem. Please note, this is linux only, and I run Manjaro which is a rolling distro instead of something like ubuntu that just picks a version and just sticks with it and backportig patches to the kernel/software.

***

> *Quick note about Ubuntu: Some users apparenly have had success installing the package `libncurses5:i386`*

> *Quick note for people in general: Please only use this as a last-resort fix, software installed via your package manager could be too new or too old for this to work, I just wrote what I did to get STM32CubeIDE to work.*

***

* Step 1: Open STM32CubeIDE and select a active project.
* Step 2: Right-click the project in STM32CubeIDE and select properties.
* > *This is so we can locate where the tools STM32CubeIDE uses are on the machine.*
* Step 3: Expand the tab `C/C++ Build` tab and select `Enviroment`.
* Step 4: Select the listing called `PATH` in the list and click `Edit...` button on the right side.
* Step 5: Select everything in the `Value:` textbox and copy it to your favourite text editor of choice.
* Step 6: Look for `:` in the text and replace them with a newline (select the `:` symbol and press enter on it to replace them)
* Step 7: Look for something that includes `/tool/bin` at the end of the line.
* * Example from the file I extracted from my `PATH` values: `/home/pcnorden/st/stm32cubeide_1.4.0/plugins/com.st.stm32cube.ide.mcu.externaltools.gnu-tools-for-stm32.9-2020-q2-update.linux64_1.5.0.202011040924/tools/bin`
* Step 8: Go to the folder that is found using above method using whatever method you want.
* Step 9: In the folder, there should be software that was downloaded by STM32CubeIDE, such as `arm-none-eabi-gdb` and several other pieces of software. Check if they are installed in your `$PATH` enviroment variable. In my case, I just wrote `arm-none` and double-tapped the TAB key in bash and it gave me all the software that was installed. Check if the tools installed on your system correspond to the downloaded tools that are in the folder.
* > If missing tools, please install them via your distrubutions package manager, or compile from source and add the compiled location to `$PATH`.
* Step 10: Make double sure that all the tools in the folder is installed via your method of choice!
* Step 11: If all the tools are installed via your selected method, go back to STM32CubeIDE and edit the `PATH` listing again. Remove the folder where the tools are listed in (DO NOT REMOVE EVERYTHING IN THAT TEXTBOX, ONLY THE FOLDER WITH THE SOFTWARE WE DON'T NEED ANYMORE!).
* * You remove it by removing the text, the `:` are seperators for `$PATH`, so only remove (for example my system) `/home/pcnorden/st/stm32cubeide_1.4.0/plugins/com.st.stm32cube.ide.mcu.externaltools.gnu-tools-for-stm32.9-2020-q2-update.linux64_1.5.0.202011040924/tools/bin` and if in the middle of the string, make sure to only leave one (1) `:` symbol!
* Step 12: Click `OK`.
* Step 13: Click `Apply and Close` on the dialog box.
* Step 14: Try to compile and upload to your microcontroller, this time it should hopefully not error out on you.
