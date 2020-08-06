# SMC_600Test

LEADSHINE SMC600 
默认连接IP 192.168.1.77
5axis test


2019.12.05
---------------
X/Y轴脉冲当量 1600pluse/unit  
导程4mm ；驱动32细分 ；57步进电机 200pluse/r
Z轴脉冲当量	1280pluse/unit
导程5mm ；驱动32细分 ；57步进电机 200pluse/r
1 unit = 1mm

X/Y轴脉冲模式  2, PUL+ DIR- 高脉冲/低方向

机械找零时候如何 停止

2019.12.06单条直线插补指令，smc_line_unit，不能连续使用。必须一条执行完毕后，再发送吓一调。
------------
连续插补函数，可以缓冲执行。
smc_conti_open_list（）；
smc_conti_start_list（）；
smc_conti_line_unit();
最多可缓冲 5000行，通过smc_conti_remain_space(_ConnectNo, 0)可查。

2019.12.12测试指令，使用Dictionary 用于存储已有的指令坐标参数。
----------------
G01 X10.15 Y10.22 Z-1.5 F20
G01 X9 Z-9 F20
G01 X130 Y150 Z-15 F10
G01 X150 Y0 Z-5 F15
G01 X0 Y0 Z0 F20

2020.04.1实验 M101 M103指令对应功能；
--------------
smc_write_outbit 这个指令开IO。
smc_conti_ahead_outbit_to_stop 
连续插补中相对于轨迹段起点IO滞后输出	1.提前关丝指令倒是有用，但是针对下一条指令的。
2.必须插入在两个 smc_conti_start_list 中间才有用
smc_conti_write_outbit 
插补中立刻操作IO
1.插补中立刻操作IO,可以用来M101 M103
2.只能在非前瞻模式下使用
M101
G01 X10.15 Y10.22 Z-1.5 F20
G01 X9 Y10 Z-9 F20
G01 X130 Y100 Z-15 F20
G01 X150 Y0 Z-5 F15
M103
G01 X0 Y0 Z0 F20
M101
G01 X10.15 Y10.22 Z-1.5 F20
G01 X9 Y10 Z-9 F20
M103

2020.04.1  添加的工件坐标模式，原有gcode需要切换成工件模式
--------------
smc_set_homemode /smc_set_pulse_outmode 先需要通过原点模式+脉冲模式+固件参数，设置好设备的坐标系。
先建好坐标系和方向。
Dist[0] = gcodeParameter["X"] + DistWork[0];    //添加工件坐标偏置
工件坐标系，直接在发送坐标然后再加偏置。不要改变方向。
smc_set_softlimit_unit 再添加软限位
M101
G01 X-90.22 Y-90.22 Z1.5 F20
G01 X-95 Y-95 F20
G01 X30 Y0 Z15 F20
G01 X50 Z5 F15
M103
G01 X-100 Y-100 Z0 F20
M101
G01 X-90.15 Y-90.22 Z1.5 F20
G01 X-99 Y-99 Z9 F20
M103


G01 X10 Y10 Z10 F10
G01 Y22 Z13 F12		//暂时不知道为啥会跟软限位冲突

2020.04.7  测试多次速度前瞻模式。添加查看插补状态
--------------

圆弧插补路径不见了

2020.07.27 添加旋转轴U轴解析代码
--------------

2020.07.29 好像之前的路径不能工作了,最后发现是 DLL库的问题，604 和606所使用的库不一样 ！！！！
--------------
通过拔取SMC Basic Studio 软件的运行参数，可以用来设置真实的参数。
 11:29:24 : smc_conti_set_lookahead_mode( 0 , 0 , 0 , 200 , 0.1 , 10000 ) = 0  设置前瞻
 11:29:24 : smc_set_vector_profile_unit( 0 , 0 , 0 , 15 , 0.2 , 0.1 , 0 ) = 0	设置插补运动速度
 11:29:24 : smc_set_vector_s_profile( 0 , 0 , 0 , 0.05 ) = 0	插补运动平滑参数
 11:29:24 : smc_conti_set_blend( 0 , 0 , 0 ) = 0	使能Blend功能
 11:29:24 : smc_set_arc_limit( 0 , 0 , 1 , 0 , 0 ) = 0	设置圆弧限速功能使能
 11:29:24 : smc_stop_multicoor( 0 , 0 , 1 ) = 0	插补运动减速停止时间
 11:29:24 : smc_conti_open_list( 0 , 0 , 3 , [0,1,2] ) = 0	打开连续插补 
 11:29:24 : smc_conti_change_speed_ratio( 0 , 0 , 1 ) = 0	动态调整连续插补速度比例
 11:29:24 : smc_conti_start_list( 0 , 0 ) = 0	开始连续插补
 11:29:24 : smc_conti_line_unit( 0 , 0 , 3 , [0,1,2] , [10,10,-10] , 1 , 1 ) = 0	添加直线插补段
 11:29:24 : smc_conti_delay( 0 , 0 , 0.5 , 2 ) = 0
 11:29:24 : smc_conti_line_unit( 0 , 0 , 3 , [0,1,2] , [20,15,-15] , 1 , 3 ) = 0	添加直线插补段
 11:29:24 : smc_conti_close_list( 0 , 0 ) = 0	停止插补

eq1,三轴前瞻连续插补
M101
G01 X-90.22 Y-90.22 Z1.5 F20
G01 X-95 Y-95 F20
G01 X30 Y0 Z15 F20
G01 X50 Z5 F15
M103
G01 X-100 Y-100 Z0 F20
M101
G01 X-90.15 Y-90.22 Z1.5 F20
G01 X-99 Y-99 Z9 F20
M103

eq2,四轴前瞻连续插补
M101
G01 X-90.22 Y-90.22 Z1.5 A40 F20
G01 X-95 Y-95 A100 F20
G01 X30 Y0 Z15 A60 F20
G01 X50 Z5 A170 F15
M103
G01 X-100 Y-100 Z0 A300 F20
M101
G01 X-90.15 Y-90.22 Z1.5 A40 F20
G01 X-99 Y-99 Z9 A180 F20
M103

2020.08.05 添加的指令A轴 B轴，连接了单脉冲的驱动器
--------------
eq1,五轴前瞻连续插补
M101
G01 X-90 Y-90 Z0 A0 B0 F20
G01 X-90 Y-90 Z0 A40 B40 F20
G01 X-90 Y-90 Z0 A180 B180 F20
G01 X-90 Y-90 Z0 A160 B160 F20
G01 X-90 Y-90 Z0 A190 B190 F20
G01 X-90 Y-90 Z0 A30 B30 F20
G01 X-90 Y-90 Z0 A355 B0 F20
M103

尽量让驱动器的脉冲当量设定为整数，保证累加后输出的脉冲有足够的精度。
Dist[3] 为理论输出的机械坐标
pos 为实际的设备的机械坐标
需要处理NX后处理精度，同时消除F1000的速度
https://wenku.baidu.com/view/6472561b26d3240c844769eae009581b6bd9bdd7.html  UG后处理制作全过程跟重要参数