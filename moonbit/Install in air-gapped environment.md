
# Install Moonbit in an air-gapped environment

## Node

- [Download Node.js](https://nodejs.org/en/download) from internet<br>
  Windows: node-v\*-win-x64.zip<br>
  Linux: node-v\*-linux-x64.tar.xz

- Copy this file to the air-gapped environment

- Create a directory of your choice where you want to store Node

- Set the NODE_PATH environment variable to this directory:<br>
    Windows: ```set NODE_PATH=e:\Users\Public\ProgramFiles\Node```<br>
    Linux: ```export NODE_PATH=/mnt/e/Users/Public/ProgramFilesLinux/Node```

- Unpack node-v\* in the NODE_PATH directory

- Add NODE_PATH\bin to your PATH environment variable:<br>
    Windows: ```set PATH=%NODE_PATH%;%PATH%```<br>
    Linux: ```export PATH=$NODE_PATH/bin:$PATH```<br>

## Moonbit

- Download Moonbit from internet<br>
  Windows: [moonbit-windows-x86_64-dev.zip](https://cli.moonbitlang.com/binaries/latest/moonbit-windows-x86_64-dev.zip)<br>
  Linux: [moonbit-linux-x86_64-dev.tar.gz](https://cli.moonbitlang.com/binaries/latest/moonbit-linux-x86_64-dev.tar.gz) for Linux

- Download [core-latest.zip](https://cli.moonbitlang.com/cores/core-latest.zip) from internet

- Copy these files to the air-gapped environment

- Create a directory of your choice where you want to store Moonbit

- Set MOON_HOME environment variable to this directory:<br>
    Windows: ```set MOON_HOME=e:\Users\Public\ProgramFiles\Moonbit```<br>
    Linux: ```export MOON_HOME=/mnt/e/Users/Public/ProgramFilesLinux/Moonbit```

- Unpack moonbit-* in the MOON_HOME directory

- Change to the MOON_HOME\lib directory

- Unpack core-latest.zip in this lib directory

- Add MOON_HOME\bin to your PATH environment variable:<br>
    Windows: ```set PATH=%MOON_HOME%\bin;%PATH%```<br>
    Linux: ```export PATH=$MOON_HOME/bin:$PATH```

- Change to the MOON_HOME\lib\core directory

- Execute the command:<br>
    Windows: ```moon.exe bundle --warn-list -a --all```<br>
    Linux: ```moon bundle --warn-list -a --all```


After completing these steps, Moonbit is ready to use.

To avoid having to set the environment variable and path each time, this can be done using a batch or shell script file or in the advanced system settings in Windows.
