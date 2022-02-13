---
title: 小米6 Spectrum调度文件记录(过时)
comments: true
tags:
  - Mi6
categories:
  - device
date: 2020-02-3 2:44:12
updated: 2020-02-3 2:44:12
---

小米6超频(过时)

<!--more-->

``` ini

		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_min_freq ""
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq ""
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_min_freq ""
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_max_freq ""


write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/up_rate_limit_us 250             如果schedutil决定增加频率，这将检查以确保自从上一次更改频率的请求以来，至少过去了up_rate_limit_us微秒。
write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/down_rate_limit_us 10000    如果schedutil决定降低频率，这将进行检查以确保自从上次更改频率请求以来，至少过去了down_rate_limit_us微秒。
write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/iowait_boost_enable 1          用于在iowait期间将CPU提升到最大频率。通常会通过移动设备禁用它以节省电池，因此也许@ RenderBroken和我将重新考虑禁用它。
(对于Up_rate_limit，该数字表示向上移动之前在给定频率上停留的时间。因此，这里的数字越大，它会以较低的频率杂散更长的时间，以及增加杂散的时间。

对于down_rate_limit，该数字表示向下移动之前在给定频率上停留多长时间。如果您希望频率更快地降低以节省电池，那么在此较小的频率会更好。如果您想要更好的性能，则需要更大的数量。 老实说，我不会碰这些，因为它们被设置为这些值是有原因的。当前值以微秒为单位。500us表示.5ms，因此每ms表示schedutil将评估是否加速。20000us表示20毫秒，因此意味着每20毫秒schedutil将评估下降时间。电池，那么数量越少越好。如果您想要更好的性能，则需要更大的数量。

请触摸它们，因为它们被设置为这些值是有原因的。当前值以微秒为单位。500us表示.5ms，因此每ms表示schedutil将评估是否加速。20000us表示20毫秒，因此意味着每20毫秒schedutil将评估下降时间。帧需要渲染才能保持60FPS的体验，即16ms。因此，在方案中，当schedutil选择频率时，只要没有任何投票，它将在给定频率下保持20ms或至少1个完整帧

一个主要的考虑因素是每帧渲染以保持60FPS体验所需的时间，即16ms。因此，在方案中，当schedutil选择频率时，只要没有任何表决权，它将在给定的频率上保持20ms或至少1个全帧，然后再增加一些。这是有目的的，因为让我们考虑是否将down值设置为500us。这意味着.5ms将使schedutil在该单个帧中下降32次。为了将其视为透视图，请想象一个动画，或者说是频率很短的工作量，即频率刚被倾倒到空闲的工作中。它将滞后并且表现不佳。这也是音频播放的问题。如果我们在给定的帧中不断降低频率以保持空闲，您将变得口吃，滞后等。我希望这是有道理的。 然后还有一些。这是有目的的，因为让我们考虑是否将down值设置为500us。这意味着.5ms将使schedutil在该单个帧中下降32倍。为了将它放入角度来看，想象的动画，或任何短工作量的频率只是被倾倒到空闲中期工作。它将滞后并且表现不佳。这也是音频播放的问题。如果我们在给定的帧中不断降低频率以保持空闲，您将变得口吃，滞后等。我希望这是有道理的。)
(For Up_rate_limit,the number means how long to stay at a given freq before moving up. So a higher number here means it will stray at lower freqs longer and how the longer it takes to ramp up.

For down_rate_limit, the number means how long to stay at a given freq before moving down. If you would like the freqs to ramp down more quickly to save battery, then a smaller number would be better here. If you want better performance, a larger number is needed.battery, then a smaller number would be better here. If you want better performance, a larger number is needed.

Honestly, I would NOT touch these as they are set to these values for a reason. Currently the values are in micro seconds. 500us means .5ms so every ms means schedutil will evaluate whether or not to ramp up. 20000us means 20ms so that means every 20ms, schedutil will evaluate ramping down. touch these as they are set to these values for a reason. Currently the values are in micro seconds. 500us means .5ms so every ms means schedutil will evaluate whether or not to ramp up. 20000us means 20ms so that means every 20ms, schedutil will evaluate ramping down.

A major consideration is how long each frame takes to render to keep a 60FPS experience and that is 16ms. So in the scheme of things, when schedutil selects a freq, as long as there are not any up votes, it will stay at that given freq for 20ms or at least 1 full frame and then some. This is on purpose as lets consider if we had the down value set to 500us. This means .5ms would make schedutil ramp down 32 times in that single frame. To put that into perspective, imagine an animation, or any short workload that the freq just gets dumped down to idle mid-work. It would lag and not perform very well. This is also a problem with audio playback. If we keep dropping the freq to idle over and over in a given frame, you will get stuttering, lag, etc. I hope this makes sense. frame takes to render to keep a 60FPS experience and that is 16ms. So in the scheme of things, when schedutil selects a freq, as long as there are not any up votes, it will stay at that given freq for 20ms or at least 1 full frame and then some. This is on purpose as lets consider if we had the down value set to 500us. This means .5ms would make schedutil ramp down 32 times in that single frame. To put that into perspective, imagine an animation, or any short workload that the freq just gets dumped down to idle mid-work. It would lag and not perform very well. This is also a problem with audio playback. If we keep dropping the freq to idle over and over in a given frame, you will get stuttering, lag, etc. I hope this makes sense.)


write /sys/module/sync/parameters/fsync_enabled N         关掉提高文件读写，提高出错几率
write /sys/module/msm_thermal/core_control/enabled 0  核心温控
write /sys/module/msm_thermal/parameters/enabled N   温控
write /sys/module/cpu_input_boost/parameters/input_boost_freq_lp "748800"  提升小的频率集群
write /sys/module/cpu_input_boost/parameters/input_boost_freq_hp "902000" 提升大簇的频率
write /sys/module/cpu_input_boost/parameters/input_boost_duration "64"        提升频率持续时间

*.prefer_idle
这是一个标志，向调度程序指示用户空间想要
调度程序专注于能源或性能。
值0（默认值）会向CFS调度程序发出信号，指示该组中的任务
可以根据能量感知唤醒策略放置。
值为1会向CFS调度程序发出信号，表明该组中的任务应为
放置以最小化唤醒延迟。
Android平台通常将此标志用于应用程序任务，
用户当前正在与之交互。


write /sys/class/kgsl/kgsl-3d0/default_pwrlevel 5
write /sys/class/kgsl/kgsl-3d0/max_pwrlevel 0
write /sys/class/kgsl/kgsl-3d0/min_pwrlevel 7
0-7  0性能最强,7最弱
write /sys/class/kgsl/kgsl-3d0/devfreq/adrenoboost 2


write /dev/stune/background/schedtune.boost 100
write /dev/stune/foreground/schedtune.boost 100
write /dev/stune/top-app/schedtune.boost 100
这允许将提升值表示为[0..100]范围内的整数。
值0（默认值）将CFS调度程序配置为最大能量
效率。这意味着sched-DVFS以最低OPP运行任务
需要满足他们的工作量需求。
值为100时，将配置调度程序以实现最高性能，即
选择该CPU上的最大OPP。
可以设置0到100之间的范围以适合其他情况。对于
满足交互式响应或取决于其他系统事件的示例
（电池电量等）。
您可以应用0到100的值。
将其设置为100时，设备将在所有核心最大化的情况下持续运行出来。
我发现2到10的值具有最有效的效果。
我当前使用的是8的提升值。电池寿命仍然很棒。但是，天哪，该设备现在运行得多么流畅。


input_boost_freq_lp 748800
```

我修改的

``` ini
# nnn 小米6 内核调度
# Date:20200222-test1
on property:sys.boot_completed=1
exec u:r:init:s0 -- /init.spectrum.sh exec u:r:su:s0 root root -- /init.spectrum.sh
exec u:r:magisk:s0 root root -- /init.spectrum.sh
exec u:r:supersu:s0 root root -- /init.spectrum.sh
setprop spectrum.support 1
setprop persist.spectrum.kernel stickernel

# 均衡模式
on property:persist.spectrum.profile=0
    #0
	    write /sys/devices/system/cpu/cpu0/online 1
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor "schedutil"
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/up_rate_limit_us 250
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/down_rate_limit_us 500
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/iowait_boost_enable 0
	    write /sys/devices/system/cpu/cpu4/online 1 
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq "1401000"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_max_freq "1497000"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_governor "schedutil"
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/up_rate_limit_us 250
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/down_rate_limit_us 500
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/iowait_boost_enable 0
		write /dev/stune/background/schedtune.boost 2
		write /dev/stune/foreground/schedtune.boost 4
		write /dev/stune/top-app/schedtune.boost 8
		write /dev/stune/background/schedtune.prefer_idle 0
		write /dev/stune/foreground/schedtune.prefer_idle 0
		write /dev/stune/top-app/schedtune.prefer_idle 1
		write /dev/stune/rt/schedtune.prefer_idle 0
		write /dev/stune/schedtune.prefer_idle 1
		write /dev/stune/schedtune.boost 1
		write /sys/class/kgsl/kgsl-3d0/default_pwrlevel 6
		write /sys/class/kgsl/kgsl-3d0/max_pwrlevel 2
		write /sys/class/kgsl/kgsl-3d0/min_pwrlevel 7
		write /sys/class/kgsl/kgsl-3d0/devfreq/adrenoboost 0
		write /dev/stune/top-app/schedtune.boost 8
		write /dev/stune/top-app/schedtune.sched_boost 8
		write /sys/module/sync/parameters/fsync_enabled N
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_lp "1248000"
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_hp "1132800"
		write /sys/module/cpu_input_boost/parameters/input_boost_duration "1000"
		write /sys/module/msm_thermal/parameters/enabled N
		write /sys/module/msm_thermal/core_control/enabled 1
		write /dev/cpuset/top-app/cpus 0-7
		write /dev/cpuset/foreground/cpus 0-3
		write /dev/cpuset/background/cpus 0-3
		write /dev/cpuset/system-background/cpus 0-3
		write /dev/cpuset/restricted/cpus 0-3
		write /dev/cpuset/audio-app/cpus 0-3
		#write /dev/cpuset/vr/cpus 0-3
		#write /dev/cpuset/gamelite/cpus 0-7
		write /dev/cpuset/camera-daemon/cpus 0-5
#end Stickernel

# 性能模式
on property:persist.spectrum.profile=1
    #1
	    write /sys/devices/system/cpu/cpu0/online 1
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor "performance"
	    write /sys/devices/system/cpu/cpu4/online 1 
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_max_freq "2457600"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_governor "performance"
		write /dev/stune/background/schedtune.boost 100
		write /dev/stune/foreground/schedtune.boost 100
		write /dev/stune/top-app/schedtune.boost 100
		write /dev/stune/background/schedtune.prefer_idle 1
		write /dev/stune/foreground/schedtune.prefer_idle 1
		write /dev/stune/top-app/schedtune.prefer_idle 1
		write /dev/stune/rt/schedtune.prefer_idle 1
		write /dev/stune/schedtune.prefer_idle 1
		write /dev/stune/schedtune.boost 1
		write /sys/class/kgsl/kgsl-3d0/default_pwrlevel 0
		write /sys/class/kgsl/kgsl-3d0/max_pwrlevel 0
		write /sys/class/kgsl/kgsl-3d0/min_pwrlevel 0
		write /sys/class/kgsl/kgsl-3d0/devfreq/adrenoboost 3
		write /dev/stune/top-app/schedtune.sched_boost 100
		write /sys/module/sync/parameters/fsync_enabled N
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_lp "1900800"
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_hp "2457600"
		write /sys/module/cpu_input_boost/parameters/input_boost_duration "3000"
		write /sys/module/msm_thermal/parameters/enabled N
		write /sys/module/msm_thermal/core_control/enabled 0
#end Stickernel

# 省电模式
on property:persist.spectrum.profile=2
    #2
	    write /sys/devices/system/cpu/cpu0/online 1
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor "ondemand"
	    write /sys/devices/system/cpu/cpu4/online 1 
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq "1401000"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_max_freq "1344400"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_governor "ondemand"
		write /dev/stune/background/schedtune.boost 0
		write /dev/stune/foreground/schedtune.boost 0
		write /dev/stune/top-app/schedtune.boost 1
		write /dev/stune/background/schedtune.prefer_idle 0
		write /dev/stune/foreground/schedtune.prefer_idle 0
		write /dev/stune/top-app/schedtune.prefer_idle 1
		write /dev/stune/rt/schedtune.prefer_idle 0
		write /dev/stune/schedtune.prefer_idle 0
		write /dev/stune/schedtune.boost 0
		write /sys/class/kgsl/kgsl-3d0/devfreq/adrenoboost 0
		write /sys/class/kgsl/kgsl-3d0/default_pwrlevel 7
		write /sys/class/kgsl/kgsl-3d0/max_pwrlevel 4
		write /sys/class/kgsl/kgsl-3d0/min_pwrlevel 7
		write /dev/stune/top-app/schedtune.sched_boost 1
		write /sys/module/sync/parameters/fsync_enabled Y
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_lp "1248000"
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_hp "1132800"
		write /sys/module/cpu_input_boost/parameters/input_boost_duration "1000"
		write /sys/module/msm_thermal/parameters/enabled N
		write /sys/module/msm_thermal/core_control/enabled 1
		write /dev/cpuset/top-app/cpus 0-7
		write /dev/cpuset/foreground/cpus 0-3
		write /dev/cpuset/background/cpus 0-1
		write /dev/cpuset/system-background/cpus 0-2
		write /dev/cpuset/restricted/cpus 0-1
		write /dev/cpuset/audio-app/cpus 0-3
		#write /dev/cpuset/vr/cpus 0-3
		#write /dev/cpuset/gamelite/cpus 0-7
		write /dev/cpuset/camera-daemon/cpus 0-5
#end Stickernel

# 游戏模式
on property:persist.spectrum.profile=3
    #3
	    write /sys/devices/system/cpu/cpu0/online 1
		write /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor "schedutil"
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/up_rate_limit_us 250
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/down_rate_limit_us 20000
		write /sys/devices/system/cpu/cpu0/cpufreq/schedutil/iowait_boost_enable 1
	    write /sys/devices/system/cpu/cpu4/online 1 
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_max_freq "2457600"
		write /sys/devices/system/cpu/cpu4/cpufreq/scaling_governor "schedutil"
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/up_rate_limit_us 250
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/down_rate_limit_us 20000
		write /sys/devices/system/cpu/cpu4/cpufreq/schedutil/iowait_boost_enable 1
		write /dev/stune/background/schedtune.boost 2
		write /dev/stune/foreground/schedtune.boost 2
		write /dev/stune/top-app/schedtune.boost 100
		write /dev/stune/background/schedtune.prefer_idle 0
		write /dev/stune/foreground/schedtune.prefer_idle 1
		write /dev/stune/top-app/schedtune.prefer_idle 1
		write /dev/stune/rt/schedtune.prefer_idle 1
		write /dev/stune/schedtune.prefer_idle 1
		write /dev/stune/schedtune.boost 1
		write /sys/class/kgsl/kgsl-3d0/default_pwrlevel 1
		write /sys/class/kgsl/kgsl-3d0/max_pwrlevel 0
		write /sys/class/kgsl/kgsl-3d0/min_pwrlevel 7
		write /sys/class/kgsl/kgsl-3d0/devfreq/adrenoboost 2
		write /dev/stune/top-app/schedtune.sched_boost 100
		write /sys/module/sync/parameters/fsync_enabled N
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_lp "1401600"
		write /sys/module/cpu_input_boost/parameters/input_boost_freq_hp "1804800"
		write /sys/module/cpu_input_boost/parameters/input_boost_duration "2800"
		write /sys/module/msm_thermal/parameters/enabled N
		write /sys/module/msm_thermal/core_control/enabled 0
		write /dev/cpuset/top-app/cpus 0-7
		write /dev/cpuset/foreground/cpus 0-7
		write /dev/cpuset/background/cpus 0-3
		write /dev/cpuset/system-background/cpus 0-3
		write /dev/cpuset/restricted/cpus 0-3
		write /dev/cpuset/audio-app/cpus 0-3
		#write /dev/cpuset/vr/cpus 0-3
		#write /dev/cpuset/gamelite/cpus 0-7
		write /dev/cpuset/camera-daemon/cpus 0-7
		write /sys/module/msm_performance/parameters/touchboost 1
#end Stickernel
```

