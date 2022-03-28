create: 2022/3/11 19:58

update: 2022/3/12 22:19

最近接触到qgc，蛮有趣的low code工具跟之前autosar那套东西挺像的。。记录一下

## qgc
就这网络。。做镜像之前一定要换源…………@mirror.ustc

https://dev.qgroundcontrol.com/master/en/getting_started/container.html

```c
/libevents/libevents/libs/cpp/parse/parser.cpp:1:
/project/source/libs/libevents/libevents/libs/cpp/parse/parser.h:13:10: fatal error: nlohmann_json/include/nlohmann/json_fwd.hpp: No such file or directory
   13 | #include "nlohmann_json/include/nlohmann/json_fwd.hpp"
      |          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

`git submodule update --recursive` 前改下协议跟depth，这网太慢了

