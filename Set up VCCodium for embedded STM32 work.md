# Setting up STM32 toolchain with Visual Studio

This is a guide on how I set up Visual Studio Code (vscodium in my case) on linux with appropiate toolchains.

Please beware that file locations might change, URLs might change and more happen during the time since I've written this, but this is how I did it when I first set up everything.

I believe that you should already have VSCodium/Visual Studio Code installed, so I won't go over the details on how to install that.

Do note please too that this is a distro-specific guide as it was made for my laptop which runs a modified version of Ubuntu 20.04 LTS made by Tuxedo Computers. Different distros might use different package managers and more, so just take this as a reference and not a absolute guide.

Software that we will install are:
|Name|Description|
|---|---|
|STM32CubeMX|This is just to get the extremely helpful tools that ST has made to set up or edit a project without having to dig into registers.|
|arm-none-eabi-{gcc/g++/gdb/size}|The ARM equivilent to gcc/g++/gdb/size that is installed for x86_64|
|st-link|Open source version of ST's STLINK tools|
|Cortex-debug|Extension for VSCodium/Visual Studio Code that makes debugging extremely easy and uploading too instead of having to manually type and upload code.|

***

## Getting STM32CubeMX
First of all, open a web browser and head to https://www.st.com/en/development-tools/stm32cubemx.html and download the software from there. ST are really annoying and need a email address they will send the download link to unless you log in, but from my experience they don't spam you more than just the download link email.

Install stm32cubemx after unpacking it, in my case I used `sudo apt install "./SetupSTM32CubeMX-6.3.0"` to install it after `cd`-ing into the directory where the files were extracted.

After all of that, open up the installed STM32CubeMX and generate a project into a folder you would like, but you need to select `Makefile` option under the `Code Generator` when it comes to the project settings.

After that, just close the software once it has generated all the things and it tells you it's done.

* Sources: None

## Getting arm-none-eabi packages
Well, first of all, this a bit weird since we can't use apt for this since apparently, ARM has decided to make our life a bit more difficult!

Check out the source down below in the sources, since this is where I got how to set up arm-none-eabi-gdb, but I will list the steps I took (which is basically the same as the askubuntu answer).

Go to https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads and download the latest linux tarball (Should look something like this: `Linux x86_64 Tarball` in the second paragraph)

Check the MD5 checksum, just for security, but if you just want to ignore checking downloads, feel free to skip this step.

Unpack the files into some directory, personally I used the same as the askubuntu answer since I like to use that location for software. My command was `sudo tar xjf "gcc-arm-none-eabi-10.3-2021.07-x86_64-linux.tar.bz2" -C /usr/share/`

Now, we need to symlink the binaries so we can call them in bash or other programs that relies on `$PATH` to find the executables.

`sudo ln -s /usr/share/gcc-arm-none-eabi-10.3-2021.07-x86_64-linux/bin/* /usr/bin/`

Do note, I have no idea what happens incase there is any similar stuff in between the bin folder and /usr/bin/, but it didn't break anything for me. You could alternately add the bin folder to `$PATH`, but I refrain from trying to modify that variable since I've broken quite a lot of stuff that way.

Now to fix that `arm-none-eabi-gdb` will probably complain about missing ncurses, run `sudo apt install libncurses5` in command line.

See if you have successfully gotten the needed software installed by issuing the following commands where you expect a version number from them.

* `arm-none-eabi-gcc --version`
* `arm-none-eabi-g++ --version`
* `arm-none-eabi-gdb --version`
* `arm-none-eabi-size --version`

should all produce a version number, or complain about a missing dependency. If you miss a dependency, go ahead and install it and see if that fixes the issue.

* Sources: https://askubuntu.com/a/1243405

## Installing Open Source STLink
You might wonder why I tell you to install from github instead of Debian/Ubuntu repositories, but the answer is that my board and programmer wasn't recogniced by the old versions in the repository.

If you use a rolling release distro, go ahead and follow the recommended way to install the tools on the [github page](https://github.com/stlink-org/stlink#installation).

On Ubuntu/Debian, it's recommended to use the deb from the github releases which you can download from https://github.com/stlink-org/stlink/releases

Once you have downloaded the deb file, use `apt` to install the file by calling on the file by using something like `sudo apt install "./stlink_1.7.0-1_amd64.deb"`.

Now, we come to the fun part, in my case the stlink apt install failed to install any udev rules, which are quite needed to be able to program a microcontroller without having to enter your sudo password nonstop.

* Sources: https://github.com/stlink-org/stlink

## Setting up stlink udev rules
First of all, this ain't really a recommended way to do this, this is something I did late at night since I was tired and exhausted over broken udev rules.

First, we need to head to where the udev rules are stored in a console of your choice.

`cd` into `/etc/udev/rules.d/`

Head to https://github.com/stlink-org/stlink/tree/develop/config/udev/rules.d and for each of them, open the file in github, right-click the button `raw` in the corner of the preview window and copy the link.

Head back to the console and do `sudo wget "insert the raw link here"` and repeat the wget download for every file in the directory.

After you've downloaded the udev rules, write the following command into the console:

`sudo udevadm control --reload-rules && sudo udevadm trigger`

That command will reload all the udev rules and then tell udevadm to basically unplug and replug all the devices connected, but it does this internally and not telling any USB device connected that it reloaded all the rules.

Lastly, check that your user is in the plugdev group by using the command `groups usernameHere` and check if plugdev is listed there. If not, you can add yourself to the plugdev group, or you can modify the udev rules to only allow access to your account for those devices.

* Sources: basically all of https://github.com/stlink-org/stlink

## Setting up Cortex-Debug in VSCodium/Visual Studio Code
Open VSCodium/Visual Studio Code (hereafter refered to as VSCodium) and head into the extensions tab.

I've made it so my VSCodium instance uses the normal Microsoft Extensions list through some modifications, so please just use this as a reference, or if you have the official Visual Studio Code installed, please continue reading.

Install the `Cortex-Debug` extension, along with the `C/C++` extension (mine is developed my microsoft, no idea what a eqivilent might be for VSCodium Extension list).

Open the folder to where you created the project that you want to use vscodium with.

First, we need to configure Cortex-Debug to use stlink utils, in my case I was prompted to add default configuration when I pressed `F5` on my keyboard and Cortex-Debug took care of basically everything, except where the built files where.

In `.vscode/launch.json`, I wrote the following launch setup for Cortex-Debug in the configuration array of the JSON file:

```json
{
	"cwd": "${workspaceRoot}",
	"executable": "./build/ProjectNameHere.elf",
	"name": "Debug Microcontroller",
	"preLaunchTask": "Build makefile",
	"request": "launch",
	"type": "cortex-debug",
	"showDevDebugOutput": false,
	"servertype": "stutil"
}
```

Now, I had to make a task to actually run the command `make` in the root dorectory to actually make it compile the firmware that is about to be uploaded.

In `.vscode/tasks.json` I added the following JSON:

```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "Build makefile",
            "command": "make",
            "options": {
                "cwd": "${workspaceRoot}"
            },
            "group": "build"
        }
    ],
    "version": "2.0.0"
}
```

That made me able to just press `F5` and make VSCodium able to compile and upload to the microcontroller without having to manually compile the project and uploading.

Now, we need to grab some definitions from `Makefile` file in the directory to be able to get the `C/C++` extension able to understand the HAL and stuff we write.

In `Makefile`, check where it has something that looks somewhat like this:

```Makefile
#######################################
# CFLAGS
#######################################
# cpu
CPU = -mcpu=cortex-m7

# fpu
FPU = -mfpu=fpv5-d16

# float-abi
FLOAT-ABI = -mfloat-abi=hard

# mcu
MCU = $(CPU) -mthumb $(FPU) $(FLOAT-ABI)

# macros for gcc
# AS defines
AS_DEFS = 

# C defines
C_DEFS =  \
-DUSE_HAL_DRIVER \
-DSTM32H743xx
```

We are after the `C_DEFS` item. From the things there, we see that we define `USE_HAL_DRIVER` and `STM32H743xx`, those we will copy into the clipboard!

Now, head into the `C/C++` configuration by opening the command pallette (CTRL+SHIFT+P for me) and select `C/C++: Edit Configurations (UI)`.

In the compiler path, set it to the arm-none-eabi-gcc compiler that we installed in a previous step when we had to install the software straight from ARM's website, which in my case was `/usr/bin/arm-none-eabi-gcc` since it's symlinked.

Compiler arguments we leave alone unless you want to add something, this is due to the makefile having compiler flags that are a bit harder to copy.

Set `IntelliSence` mode to `linux-gcc-arm`.

In the box `Defines`, paste the `C_DEFS` we copied from the `Makefile` file and remove the `-D` before the actual define. VSCodium also tells you to put only one define per line, so make sure that you have them on seperate lines!

I personally set the C standard to `C 17` due to me struggling to set everything up in the first place, but if it works, you can leave it alone.

* Source: Hours and hours of me struggling to set everything up

***
That should be it and VSCodium should now work as a sort-of fully fledged IDE with code completion, though there might be cases where you need to actually use arm-none-eabi-gdb to decode what is happening, but normal coding should be alright now!

Good luck developing firmware for microncontrollers, I wish you the very best of luck!