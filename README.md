# AP-GLLVM

<!-- add hyperlinks here  -->
**Description**
A modified version of gllvm which works on the ArduPilot build system 

See ```gllvm.md``` for the original gllvm README

**Logistical Differences**
- Note that this version of gllvm must be installed with 
    ```bash
    go install github.com/thirdxmatson/ap-gllvm/cmd/...@latest
    ```
    not
    ```bash
    go install github.com/SRI-CSL/gllvm/cmd/...@latest
    ```

- We have also updated the minimum verison of go to 1.17 from 1.16 since version 1.16 is no longer supported

**Modification**
- The only executables that are affected by these modifications are gclang and gclang++

- All modifications are in the file ```shared/parser.go```in the function getArtifactNames
    - The key issue is that gllvm aims to compiler object files and bitcode files separately, before adding the path of the bitcode file to the object file which will be linked in the final executable. However, the gllvm system assumes 2 invariants to hold during the bitcode path addition step, which do not hold for the ArduPilot build system. For more on the ArduPilot build system, see documentation for waf. <!-- add a hyperlink here for waf --> 
        1. All object files must placed in the same directory that they are compiled in
        2. For every ```.c``` or ```.cpp```, in the format ```filename.cpp```, its corresponding object file should be called ```filename.o```
    
    - The first invariant does not hold because the waf executable uses ```ardupilot/build/sitl``` *(or whichever board is being used)* as its working directory for the entire build process rather than ```cd```ing into the directory where object files will be stored before compiling them *(ex. ```ardupilot/build/sitl/libraries/AC_CustomControl/```)*. The second invariant does not hold because most or all ArduPilot object files are compiled into a format like filename.someextnesion.o, where we are compiling filename.cpp. For example, ```AC_CustomControl.cpp``` compiled to ```AC_CustomControl.cpp.0.o``` rather than ```AC_CustomControl.o```.

    - Our solution involves modifying the getArtifactNames