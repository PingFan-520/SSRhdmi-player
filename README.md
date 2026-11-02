# 万象智屏 —— 基于安路 EG4S20 的多功能智能显示广告牌

## 一、目录说明

| 目录 / 文件 | 内容 | 是否必需 |
| --- | --- | --- |
| `rtl/` | 全部自研 RTL（17 个功能模块，约 8 000 行 Verilog） | 必需 |
| `constraints/` | 引脚约束 `board.adc` 与时序约束 `*.sdc` | 必需 |
| `prj/yuanshen_hdmi.al` | TD 工程文件（器件、源文件列表、编译选项） | 必需 |
| `sim/` | 20+ 个测试台与批处理（`run_*.bat`），不含 work 目录 | 调试用 |
| `bit/*.bit` | 已综合布线的码流，可直接烧 Flash | 上板用 |
| `sd_card/` | 已转好的 5 张 960×540 BMP，拷到卡根目录即可 | 可选 |

## 二、从源码重建

```bat
:: 综合（约 40 s）
cd prj\yuanshen_hdmi_Runs\syn_1
E:\TD\bin\td_commands_prompt.exe yuanshen_hdmi.tcl
（路径修改为自己的）
:: 布局布线（约 90~280 s）
cd ..\phy_1
E:\TD\bin\td_commands_prompt.exe yuanshen_hdmi.tcl
```

产物：`prj/yuanshen_hdmi_Runs/phy_1/yuanshen_hdmi.bit`。
也可以直接用 TD 图形界面打开 `prj/yuanshen_hdmi.al`，点「综合 → 布局布线 → 生成码流」。
**注意：不要在 GUI 里改布局参数**，GUI 保存工程会把 `phy_1` 的 tcl 重置成默认模板；
本工程已压缩寄存器位宽，默认脚本即可布通。

资源占用：LUT 14 888 / 19 600（76.0%）、FF 8 699、BRAM 31 / 64；
时序：clk_pix 同域 +1.3 ns 以上、clk_sd +15 ns，均无违例。

## 三、上板

1. 把 `sd_card/*.BMP` 拷到 SD 卡**根目录**（FAT32，8.3 大写短名）；
   音乐需单声道 48 kHz/16 bit WAV，用 `tools/make_wav_mono.py 源.wav sd_card/M1.WAV` 生成。
2. 卡插入板载卡座，HDMI 接显示器，上电。
3. 烧录：TD → 下载 → 选 `bit/*.bit` → 目标选 **Flash** → 擦除后写入 → 断电重上电。

上电自检：约 2 s 出第一张图。六位数码管静止时显示
`加载次数 | 出错状态 | 图片数 | 音乐数`（十六进制），例如 `1 0 0 5 0 2`
= 1 次加载无错、5 张图、2 首音乐；加载过程中前两位变为 `00~99` 进度。

## 四、仿真

```bat
cd sim
run_fat.bat            :: FAT32 簇链 + 目录扫描 + 故障注入
run_frame_full.bat     :: 帧缓存全帧写入/读出
run_osd.bat            :: 菜单 48 项断言
run_menu_fade.bat      :: 菜单淡入淡出逐帧轨迹
```

## 五、关键参数

| 项目 | 值 |
| --- | --- |
| 器件 | 安路 EG4S20BG256（康芯 HX4S20C 板） |
| 输出 | 1280×720 @ 30 Hz，HDMI 1.4b，含音频数据岛 |
| 时钟 | clk_sd 25 MHz / clk_mem 125 MHz / clk_pix 37.5 MHz |
| 帧缓存 | 960×540 RGB565 双缓冲，2.07 MB |
| 单张切图 | 1.3 s（清屏 35 ms + 读图 1.0 s + 淡入 270 ms） |
| 音频 | 48 kHz / 16 bit 单声道，嵌入 HDMI |
| 交互 | 4 键（A2↑ B2↓ B1 菜单 C1 列表）+ 4 页 OSD 菜单 |
