# Install Moonbit in an air-gapped Windows environment

- Download [moonbit-windows-x86_64-dev.zip](https://cli.moonbitlang.com/binaries/latest/moonbit-windows-x86_64.zip) from internet.

- Download [core-latest.zip](https://cli.moonbitlang.com/cores/core-latest.zip) from internet.

- Copy these files to the air-gapped environment.

- Create a directory of your choice where you want to store Moonbit.

- Set MOON_HOME environment variable to this directory:<br>e.g. ```set MOON_HOME=e:\Projects\WebAssembly\moonbit```

- Unpack moonbit-windows-x86_64-dev.zip in the MOON_HOME directory.

- Change to the MOON_HOME\lib directory.

- Unpack core-latest.zip in this lib directory.

- Add MOON_HOME\bin to your PATH environment variable:<br>```set PATH=%MOON_HOME%\bin;%PATH%```

- Change to the MOON_HOME\lib\core directory.

- Execute the command:<br>```moon.exe bundle --warn-list -a --all```

After completing these steps, Moonbit is ready to use.

To avoid having to set the environment variable and path each time, this can be done using a batch file or in the advanced system settings.
