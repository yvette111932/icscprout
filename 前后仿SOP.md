> [!TIP]
> 本教程使用*smic55ll_ulp_09121825_oa_cds_v1.16_2*库，以简单反相器电路为例，整理前仿、DRC验证、LVS验证、寄生参数提取和后仿等流程。需要使用Cadence Virtuoso、Mentor Calibre、StarRC等软件。

# 前仿真
此步骤建立在电路原理图和cellview已绘制完成，且工艺库已添加的基础上。

在终端输入`virtuoso`打开，在CIW对话框选择*Tools*，*Library Manager*，选择添加的库，选择*cell*，打开电路原理图*schematic*。

<div align=center><img src="https://github.com/user-attachments/assets/49dc184d-ae29-469b-9483-a350787c6035" width=50% height=50%/></div>

*以简单的反相器电路为例，选择*launch*，*ADE L*，选择*set up*，*stimuli*，修改stimuli信息。选择*gnd*激励源，点击*Enabled*，Function选择*dc*，在DC voltage部分输入*0*。**注意修改完成后需要点击*change*以保存。** 同理，修改*vcc*输入电压为*1.8*。
接着选择*in*，Function选择*pulse*，Voltage 1初始为0无需修改，修改Voltage 2为*1.8*，将Delay time、Rise time和Fall time均改为*0.1n*。修改Pulse width为*5u*、Period为*10u*，保存后点击*OK*。

<div align=center><img src="https://github.com/user-attachments/assets/e52299d6-51af-4599-a9eb-cfb701b73bdf" width=30% height=30%/></div>

在*set up*，*Model Libraries*中，选择添加Model File。选择添加调用的库文件，**注意需要保持调用的库统一，并取消选择系统初始库，否则可能造成重复define error！** 找到*Models*文件夹，选择*spectre文件*，点击*spe.lib*文件并选择。选择Section为*tt*，点击*OK*。

<div align=center><img src="https://github.com/user-attachments/assets/e31d672a-1ac3-40ed-9d61-472f81a08bdc" width=50% height=50%/></div>

选择*Analyses*，*Choose*，修改Stop Time为*50u*，选择*conservative*，点击*OK*。

<div align=center><img src="https://github.com/user-attachments/assets/16b0daa7-afea-4acb-9bd2-e70cc4218a57" width=30% height=30%/></div>

接着选择*Outputs*，*To Be Plotted*，*Select On Design*，在电路图上选择*input电路和output电路*。选择*Simulation*，*Netlist and Run*，得到波形图。选择右上角*Split All Strips*，将input与output波形图分开。

<div align=center><img src="https://github.com/user-attachments/assets/f7da72e2-e32b-4589-9fe4-8093ab643fe8" width=60% height=60%/></div>

选择*Simulation*，*Netlist*，*Create*，生成*scs网表*并保存，方便后仿时调用。至此，前仿完成。


# DRC验证
> DRC验证是对设计版图进行检查，以版图层为目标，对相同及相邻版图层之间的关系及尺寸进行规则检查，目的是保证版图满足流片厂家的设计规则[^1]。

此步骤建立在电路原理图、电路版图和cellview已绘制完成，且工艺库已添加的基础上。

打开电路版图*layout*，选择*Calibre*，*Run nmDRC*，关闭*Load Runset File*窗口，选择*DRC Rules File*。在*Calibre文件夹*中选择*DRC文件夹*，选择*drc*文件并确定，点击*Run DRC*。

<div align=center><img src="https://github.com/user-attachments/assets/ee703472-69b5-4bbc-8649-dd12b014263a" width=50% height=50%/></div>

根据验证结果修改错误，密度错误可暂时忽略。至此，DRC验证完成。

<div align=center><img src="https://github.com/user-attachments/assets/0b873031-fc80-4f0c-b75e-e64caa5bc7f7" width=60% height=5=60%/></div>


# LVS验证
> LVS (Layout Versus Schematic) 验证目的在于检查人工绘制的版图是否和电路结构相符[^1]。

此步骤建立在电路原理图、电路版图和cellview已绘制完成、工艺库已添加，且DRC验证已通过的基础上。

打开电路版图*layout*，选择*Calibre*，*Run nmLVS*。同理，选择*LVS Rules File*。出现*√*和*☺*则表示LVS验证通过。至此，DRC验证完成。

<div align=center><img src="https://github.com/user-attachments/assets/08bd844c-fdcb-41c8-810d-227cd6fcf3bd" width=70% height=70%/></div>


# 寄生参数提取

此步骤建立在电路原理图、电路版图和cellview已绘制完成、工艺库已添加，且DRC、LVS验证已通过的基础上。本教程将以CCI StarRC CUI-Flow[^2]为例，即Run StarRC command line file for CUI。

首先建议新建一个文件夹，在文件夹中复制以下文件进来：
- CDL netlist（前仿得到）
- Calibre_LVS_runset_CCI文件，即SMIC库中的lvs文件
- layout生成的gds文件（在版图界面选择*File*，*Export Stream from VM*）
- CCI_cmd文件（在smic库中找到CCI_StarRC文件夹内的CCI_flow_for_CUI文件夹）
- query_cmd文件（同上）
- nxtgrd文件（在smic库中NXTGRD文件夹内，选择**RCMAX**文件）
- mapping file（在smic库中找到CCI_StarRC文件夹内的CCI_flow_for_CUI文件夹，选择**tran.map**文件）
- 

先进行环境配置。打开终端，输入`ls`显示文件夹中所有文件名，输入`vi`打开并修改LVS runset文件。输入`/`搜索要修改的位置，点击`n`跳到下个搜索到的文字，此处可搜索`TOPMETAL`。输入`i`进入insert模式，根据所使用的total metal layer数量修改数字，此处修改为`TOPMETAL 8`。修改TOP_METAL_NUM，可以根据文件夹名*SMIC 1TM*或是*SMIC 2TM*修改为`SINGLE`或是`DOUBLE`。点击*esc*退出编辑模式。

<div align=center><img src="https://github.com/user-attachments/assets/c2009974-1e0b-4518-94dd-565024fc2a47" width=50% height=50%/></div>

同理，修改*RC_RUN TRUE*和*Back_Annotation_Flow*。若想要StarRC output格式为Extracted View，则修改为*1*；若想要格式为netlist，则修改为*2*。

<div align=center><img src="https://github.com/user-attachments/assets/c9a2f044-6208-40f5-9b05-5cf483f6da52" width=50% height=50%/></div>

添加`MASK SVDB DIRECTORY "sdvb" QUERY CCI`，或是`CCI QUERY`，需要根据前几行格式自行判断。若已经有这行了，则**无需添加**，否则会报错。

<div align=center><img src="https://github.com/user-attachments/assets/a201c216-44df-47d7-8423-59c399217ecb" width=50% height=50%/></div>

修改cdl文件和gds文件的路径和cell name。输入指令`:wq!`强制保存并退出。

<div align=center><img src="https://github.com/user-attachments/assets/4f65192f-4ef1-4a25-ac85-b7d5c1b5b99f" width=50% height=50%/></div>

为防止寄生电容电阻参数被二次提取，在终端输入`calibre -lvs -hier -hcell hcell_list \u2013spice ./svdb/<top_cell>.sp <Calibre_LVS_runset_CCI>`，注意替换top cell为cell name，Calibre_LVS_runset_CCI为LVS runset file文件名。注意此处command为case-sensitive。

修改*CCI_cmd*文件，修改为`MAGNIFY_DEVICE_PARAMS: NO`（可能无需修改），`BLOCK: <cell name>`。在`TCAD_GRD_FILE`后修改nxtgrd文件名，在`MAPPING_FILE`后修改map file名字。定义生成的寄生参数文件，以*spf*为后缀。将`CALIBRE_RUNSET`修改为LVS runset文件名。

<div align=center><img src="https://github.com/user-attachments/assets/70940332-28c5-4104-b617-e44bb07d7e20" width=50% height=50%/></div>

修改*query_cmd*文件，将文件中所有`TOP_CELL`的部分全都替换为自己的cell name，共有9处。

<div align=center><img src="https://github.com/user-attachments/assets/eb6e34ef-ceb1-4d16-88ba-4b026145a05c" width=50% height=50%/></div>

用前面修改好的.lvs规则文件跑一遍版图的lvs，记住需要勾选下图所示选项[^3]，生成svdb文件夹：

<div align=center><img src="https://github.com/user-attachments/assets/ac567737-b73d-4bfd-aee4-1c5f60ee3388" width=50% height=50%/></div>

在终端输入`calibre -query_input query_cmd -query svdb <top_cell>`，修改cell name，生成StarRC output database文件。

接着输入`StarXtract -clean CCI_cmd`，生成寄生参数网表*spf*文件。至此，寄生参数提取完成。


# 后仿真








# 参考文献
[^1]: CMOS模拟集成电路版图设计与验证——基于Cadence Virtuoso与Mentor Calibre
[^2]: Quick-start on SMIC Calibre Connectivity Interface (CCI) - StarRC Flow
[^3]: StarRC 寄生参数提取与后仿
