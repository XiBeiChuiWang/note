# JVM常用参数

1. ### 栈

2. ### 堆

   -XX:+PrintFlagsInitial    查看所有的参数的默认初始值

   -XX:+PrintFlagsFinal      查看所有参数的最终值

   -Xms                                初始堆空间大小（默认为物理内存的 **1/64**）

   -Xmx                                 最大堆空间大小（默认为物理内存的 **1/4**）

   -Xmn                                设置新生代大小（初始值及最大值）

   -XX:NewRatio                 配置新生代与老年代的占比（默认为2，表示老年代占堆的2/3）

   -XX:+SurvivorRatio    设置新生代中Eden和幸存者空间的比例

   -XX:MaxTenuringThreshold   设置新生代垃圾的最大年龄

   -XX:+PrintGCDetails           输出详细的GC处理日志

    	打印gc简要信息      

   ​	 -XX:+PrintGC

   ​	-verbose:gc

   -XX:HandlePromotionFailure            是否设置空间分配担保

   -server：       启动server模式，因为在server模式下，才可以启用逃逸分析

   -XX:+DoEscapeAnalysis  :     启用逃逸分析

   -XX:+EliminateAllocations :开启标量替换（默认打开）

   |            命令            |                          作用                          |            默认值            |
   | :------------------------: | :----------------------------------------------------: | :--------------------------: |
   |   -XX:+PrintFlagsInitial   |               查看所有的参数的默认初始值               |                              |
   |    -XX:+PrintFlagsFinal    |                  查看所有参数的最终值                  |                              |
   |            -Xms            |                     初始堆空间大小                     |     物理内存的 **1/64**      |
   |            -Xmx            |                     最大堆空间大小                     |      物理内存的 **1/4**      |
   |            -Xmn            |                     设置新生代大小                     |                              |
   |        -XX:NewRatio        |                配置新生代与老年代的占比                | 默认为2，表示老年代占堆的2/3 |
   |     -XX:+SurvivorRatio     |           设置新生代中Eden和幸存者空间的比例           |           8：1：1            |
   |  -XX:MaxTenuringThreshold  |                设置新生代垃圾的最大年龄                |              15              |
   |    -XX:+PrintGCDetails     |                  输出详细的GC处理日志                  |                              |
   | -XX:HandlePromotionFailure |                  是否设置空间分配担保                  |                              |
   |          -server           | 启动server模式，因为在server模式下，才可以启用逃逸分析 |                              |
   |   -XX:+DoEscapeAnalysis    |                      启用逃逸分析                      |                              |
   | -XX:+EliminateAllocations  |                      开启标量替换                      |             打开             |

   

3. 