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
- The only executables that are affected by these modifications are ```gclang``` and ```gclang++```

- All modifications to the source are in the file ```shared/parser.go``` in the function getArtifactNames *(aside from updating github urls throughout the repository for consistency from ```github.com/SRI-CSL/gllvm/``` to ```github.com/thirdxmatson/ap-gllvm/```)*
    - The key issue is that gllvm aims to compile object files and bitcode files separately, before adding the path of the bitcode file to the object file which will be linked in the final executable. However, the gllvm system assumes 2 invariants to hold during the bitcode path addition step, which do not hold for the ArduPilot build system (*for more on the ArduPilot build system, see documentation for [waf](https://waf.io/book/)*): <!-- add a hyperlink here for waf --> 
        1. All object files must placed in the same directory that they are compiled in
        2. For every ```.c``` or ```.cpp```, in the format ```filename.cpp```, its corresponding object file should be called ```filename.o```

    - The first invariant does not hold because the waf executable uses ```ardupilot/build/sitl``` *(or whichever board is being used)* as its working directory for the entire build process rather than ```cd```ing into the directory where object files will be stored before compiling them *(ex. ```ardupilot/build/sitl/libraries/AC_CustomControl/```)*. The second invariant does not hold because most or all ArduPilot object files are compiled into a format like filename.someextnesion.o, where we are compiling filename.cpp. For example, ```AC_CustomControl.cpp``` compiled to ```AC_CustomControl.cpp.0.o``` rather than ```AC_CustomControl.o```.

    - A secondary issue is that there were a few instances where a different version of ```filename.cpp``` existed in multiple different directories. However, since the entire compilation process was taking place in ```ardupilot/build/sitl```, the ```filename.bc``` files were being places in the same directory, causing one or more to be overwritten and lost.
    
    - Our solution involves modifying the getArtifactNames which returns the name of the object file that has just been generated and the corresponding bitcode file that gllvm will generate in the next step in order to attach it to the object file. We modify the function to keep track of the exact file name of the object file by referring the the output argument of the compile command, rather than the input argument as was the original functionality. Further, we obtain the relative path of the compile command argument and add this to both of our return variables (object file name and bitcode file name). We add to the object file to ensure that gllvm can find the object file to link the corresponding bitcode file. We add it to the bitcode file name to ensure that it is placed in the same directory as the object file, thus negating the problem of same name bitcode files overwritting one another.
