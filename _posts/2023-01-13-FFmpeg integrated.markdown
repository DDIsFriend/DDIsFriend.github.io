---
layout: default
title:  "FFmpeg integrated"
date:   2023-01-13 13:55:07 +0800
categories: jekyll update
---
# FFmpeg integrated

## 制作iOS可以使用的ffmpeg lib
利用现有的工具[FFmpeg-iOS-build-script](https://github.com/kewlbear/FFmpeg-iOS-build-script)来制作iOS使用的lib，可以得到我们接下来要使用的三个文件夹fftools、include及lib。
制作时，可以根据自己的需要通过修改脚本来集成ffmpeg的功能，这一步不多说，可以自行查资料。
## 将fftools、include及lib集成进xcode
最主要的就是这一步，这一步决定了你的ffmpeg是否能用。  
1. 我们先加依赖库，直接上代码  
```Objective-C
ss.libraries = 'bz2','iconv','z'
ss.frameworks = 'AudioToolbox', 'CoreAudio' , 'VideoToolbox', 'CoreMedia', 'AVFoundation'
```
2. 然后我们解决一些小问题  
    1. 将libavdevice中的time.h改个名，因与系统库重名，可以改为fftime.h。  
    2. 若在pch文件中导入了foundation等的，请先#ifdef objc 再导入。  
    3. header search path，user header search path(可选，可以用quote即”“方式引用)，library search path设置一下，直接上代码  
```Objective-C
ss.pod_target_xcconfig = {
'HEADER_SEARCH_PATHS' => '${PODS_ROOT}/../../DDFFmpegKit_Private/Classes/DDFFmpeg/DDFFmpegBase/include',
'LIBRARY_SEARCH_PATHS' => '${PODS_ROOT}/../../DDFFmpegKit_Private/Classes/DDFFmpeg/DDFFmpegBase/lib'
}  
```  
    4. 上方代码中的`${PODS_ROOT}/../../`为pods的root，`DDFFmpegKit_Private/Classes/DDFFmpeg/DDFFmpegBase/lib`为lib所在的路径。  
3. 去掉include文件下的头文件里某些没导入的include（这些没导入的头文件你可以在制作lib时翻找文件夹找到这些文件，你可以选择导入，也可以选择注释掉,发挥一下聪明才智）  

```Objective-C
nb0_frames = nb_frames = mid_pred(ost->last_nb0_frames[0],
                                  ost->last_nb0_frames[1],
                                  ost->last_nb0_frames[2]);

ff_dlog(NULL, "force_key_frame: n:%f n_forced:%f prev_forced_n:%f t:%f prev_forced_t:%f -> res:%f\n",
            ost->forced_keyframes_expr_const_values[FKF_N],
            ost->forced_keyframes_expr_const_values[FKF_N_FORCED],
            ost->forced_keyframes_expr_const_values[FKF_PREV_FORCED_N],
            ost->forced_keyframes_expr_const_values[FKF_T],
            ost->forced_keyframes_expr_const_values[FKF_PREV_FORCED_T],
            res);

PRINT_LIB_INFO(avresample, AVRESAMPLE, flags, level);
PRINT_LIB_INFO(postproc, POSTPROC, flags, level);

//下面两项可以通过在config中disable掉，也可以选择注释代码
{ "videotoolbox_pixfmt", HAS_ARG | OPT_STRING | OPT_EXPERT, { &videotoolbox_pixfmt}, "" },

{ "videotoolbox",   videotoolbox_init,   HWACCEL_VIDEOTOOLBOX,   AV_PIX_FMT_VIDEOTOOLBOX },
```
4.include文件架下的各个  
![文件](/assets/2023-01-13-FFmpeg integrated/includeList.png){:height="20%" width="20%"}  
将各个version.h的头文件改下名，因为每个lib下的头文件都叫version，重名了。  

5.将ffmpeg.c的main函数改下名字，因为与app的main函数重复了。  

## 最后一步
这个时候你会发现`以运行了，但是每一次执行完ffmpeg，app就退出了，因为`exit_program`这个函数,此时把ffmpeg.c中的`main`函数里的`exit_program`替换成`ffmpeg_cleanup`函数，之后在`ffmpeg_cleanup`函数的`term_exit() `函数前，将各个计数器清零，加入如下代码
```Objective-C
nb_filtergraphs=0;
nb_output_files=0;
nb_output_streams=0;
nb_input_files=0;
nb_input_streams=0;
```


