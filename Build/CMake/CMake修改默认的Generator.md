### Windows下指定Generator
`cmake -G "Unix Makefiles" ..`

不知道为什么Windows下总是出现不使用CMakeLists.txt中指定的编译器和Generator

上面的命令可以指定对应的Generator, 从而使用正确的编译器（CMakeLists.txt中指定的）