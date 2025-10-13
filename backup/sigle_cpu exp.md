## 单周期处理器实验任务
1.总体说明
单周期CPU是最基础的处理器实现方式，所有指令在一个时钟周期内完成。
预计通过所有实验（六个章节）实现一个支持10条指令的单周期cpu，其中可以选择三种不同的指令集对单周期cpu进行逐步实现，关于指令的讲解在ISA部分中查看。

1.1架构说明
本实验设计搭建了一个基于模块化设计的简易 CPU 系统，其顶层模块为 cpu_sys。该系统通过模块实例化的方式，集成了处理器核心（cpu）、指令存储器（IM）、数据存储器（DM）以及通用寄存器文件（regfile）四个核心组件。各模块之间通过标准的信号接口进行通信，实现了简易指令获取、执行与数据读写操作的基本功能。其中cpu需要学生具体设计实现。

<img width="400" height="450" alt="Image" src="https://github.com/user-attachments/assets/0ea7e236-afd5-43ec-b222-c2eb18741832" />

cpu_sys顶层代码：
```verilog
module cpu_sys(
   input    wire          clk,
   input    wire          rst,
   output   wire          dm_we,
   output   wire [31:0]   dm_addr,
   output   wire [31:0]   dm_wdata,
   output   wire          rf_we,
   output   wire [4:0]    rf_waddr,
   output   wire [31:0]   rf_wdata
);
wire  [31:0] im_addr;
wire  [31:0] im_radta,dm_rdata;
wire  [ 4:0] rf_addr1,rf_addr2;
wire  [31:0] rf_rdata1,rf_rdata2;
loongarch_cpu u_cpu(
   .clk      (clk),
   .rst      (rst),
   //im 与指令存储器的接口
   .im_addr  (im_addr),
   .im_rdata (im_radta),
   //dm 与数据存储器的接口
   .dm_addr  (dm_addr),
   .dm_we    (dm_we),
   .dm_wdata (dm_wdata),
   .dm_rdata (dm_rdata),
   //regfile 与寄存器堆的接口
   .rf_raddr1   (rf_addr1),
   .rf_rdata1   (rf_rdata1),
   .rf_raddr2   (rf_addr2),
   .rf_rdata2   (rf_rdata2),
   .rf_we       (rf_we),
   .rf_waddr    (rf_waddr),
   .rf_wdata    (rf_wdata)
);
im_4k    u_im(
   .addr    (im_addr),
   .rdata   (im_radta)
);
dm_4k    u_dm(
   .clk     (clk),
   .addr    (dm_addr),
   .we      (dm_we),
   .wdata   (dm_wdata),
   .rdata   (dm_rdata)
);
regfile  rf(
   .clk     (clk),
   .raddr1  (rf_addr1),
   .rdata1  (rf_rdata1),
   .raddr2  (rf_addr2),
   .rdata2  (rf_rdata2),
   .we      (rf_we),    
   .waddr   (rf_waddr),
   .wdata   (rf_wdata)
);
endmodule
```


1.2模块说明
CPU模块：cpu
该模块是处理器核心，需要实现指令译码、执行以及控制信号生成等功能。它与以下三个模块通过接口通信：
与指令存储器 IM 通信：通过地址总线读取指令。
与数据存储器 DM 通信：读取或写入数据。
与寄存器文件 regfile 通信：读取源操作数，写回结果。

指令存储器模块：IM
该模块为一个只读的4K字节大小指令存储器，负责根据 CPU 提供的地址返回对应的指令。其接口简单，仅包含地址输入和数据输出端口。
```verilog
module im_4k(
    input    wire  [31:0] addr,
    output   wire  [31:0] rdata
);
    reg [31:0] imem[1023:0];
    assign rdata = imem[addr[11:2]];
    endmodule   
```

数据存储器模块：DM
为一个支持读写操作的数据存储模块，容量为4K字节。CPU 可以通过该模块进行数据加载和存储操作。其接口包括地址、写使能、写入数据和读出数据。
```verilog
module dm_4k(
   input    wire         clk,
   input    wire  [31:0] addr,
   input    wire         we,
   input    wire  [31:0] wdata,
   output   wire  [31:0] rdata
);
     
   reg [31:0] dmem[1023:0];

   always @(posedge clk) begin
      if (we)
         dmem[addr[11:2]] <= wdata;
   end
   
   assign rdata = dmem[addr[11:2]];
endmodule  
```

通用寄存器文件模块：regfile
该模块实现了寄存器堆，包含两个读端口和一个写端口。CPU 可通过此模块读取两个源操作数 ，并将执行结果写回到指定的寄存器地址。
```verilog
module regfile(
    input  wire        clk,
    // 1号读端口
    input  wire [ 4:0] raddr1,
    output wire [31:0] rdata1,
    // 2号读端口
    input  wire [ 4:0] raddr2,
    output wire [31:0] rdata2,
    // 写端口
    input  wire        we,       //写使能，高位有效
    input  wire [ 4:0] waddr,
    input  wire [31:0] wdata
);
    reg [31:0] rf[31:0];
    //写入always @(posedge clk) begin
    if (we) rf[waddr]<= wdata;end
    //1号读端口输出
    assign rdata1 = (raddr1==5'b0) ? 32'b0 : rf[raddr1];
    //2号读端口输出
    assign rdata2 = (raddr2==5'b0) ? 32'b0 : rf[raddr2];
endmodule
```

2.Loongarch指令集

2.1 实验一： 第一条指令ORI的实现

题目说明：基于Loongarch指令集实现可以运行一条指令ORI的单周期处理器，本实验较为基础，所以采用挖空的形式对处理器的代码进行完善和逻辑梳理。通过实现第一条指令(ori指令)，帮助学生理解CPU的数据通路、控制器设计以及指令执行的全过程。

实验一待补充代码框架：
```verilog
module loongarch_cpu(
    input    wire           clk,
    input    wire           rst,
    //IM 指令存储
    output   wire   [31:0]  im_addr,    //访问指令存储器的地址
    input    wire   [31:0]  im_rdata,   //访问指令存储器的读数据
    //DM 数据存储
    output   wire   [31:0]  dm_addr,    //访问数据存储器的地址
    output   reg            dm_we,      //数据存储器的写使能
    output   wire   [31:0]  dm_wdata,   //写入数据存储器的数据
    input    wire   [31:0]  dm_rdata,   //访问数据存储器的读数据
    //regfile 寄存器堆
    // 1号读端口
    output   wire   [ 4:0]  rf_raddr1,  //第一个读端口读取寄存器的地址
    input    wire   [31:0]  rf_rdata1,  //第一个读端口读取寄存器得到的数据
    // 2号读端口
    output   wire   [ 4:0]  rf_raddr2,  //第二个读端口读取寄存器的地址
    input    wire   [31:0]  rf_rdata2,  //第二个读端口读取寄存器得到的数据
    // 写端口
    output   reg            rf_we,      //写使能，高位有效
    output   wire   [ 4:0]  rf_waddr,   //需要写入的寄存器的地址
    output   reg    [31:0]  rf_wdata    //需要写入寄存器的数据
);
    // PC 寄存器与有效标志
    reg  [31:0] pc;
    wire [31:0] next_pc;
    reg         valid;
    always @(posedge clk) begin
        if (rst)
            valid <= 1'b0;
        else
            valid <= 1'b1;
    end
    always @(posedge clk) begin
        if (rst)
            pc <= 32'h0;
        else if (valid)
            pc <= next_pc;
    end
    assign im_addr = ;//使用pc值取值
    assign next_pc = ;//暂时顺序执行每个周期加4，pc+4,下一次pc为当前加4
    //读取指令
    wire [31:0] inst = im_rdata;
    //判断指令
    wire inst_ori = inst[31:22] == 10'b00_0000_1110;//ori指令的31-22位
    // 寄存器地址与立即数提取
    wire [4:0] rj = inst[9:5];
    wire [4:0] rd = inst[4:0];
    wire [11:0] ui12 = inst[21:10];
    wire [31:0] imm = {20'b0, ui12};  // 零拓展
    
    assign rf_raddr1 = ;//读取的寄存器
    // 计算 ori 结果
    wire [31:0] ori_result = ;//填写自己的代码
    assign rf_waddr  = ;// ori 写入目标是 rd
    always @(*) begin
        if (valid && inst_ori)//有效且是ori指令时寄存器写信号有效
            rf_we = 1'b1;
        else
            rf_we = 1'b0;
    end
    always @(*) begin
        if (inst_ori)
            rf_wdata = ;//填写需要写回的数据
        else
            rf_wdata = 32'b0;
    end
endmodule
```

顶层模块对应仿真testbench，其中im与dm内容如何生成并加载至指定文件中让tb读取仅供参考，将txt放置自己的路径中：
```verilog
`timescale 1ns / 1ps

module system_tb();
reg clk;
reg rst;
wire          dm_we,rf_we;
wire [31:0]   dm_addr,dm_wdata;
wire [4:0]    rf_waddr;
wire [31:0]   rf_wdata;
cpu_sys u_cpu_sys(
    .clk        (clk),
    .rst        (rst),
    .dm_we      (dm_we   ),
    .dm_addr    (dm_addr ),
    .dm_wdata   (dm_wdata),
    .rf_we      (rf_we   ),
    .rf_waddr   (rf_waddr),
    .rf_wdata   (rf_wdata)
);
integer i;
initial
begin
    $readmemh( "ORI.txt" , 
                u_cpu_sys.u_im.imem ) ; 
    clk = 1'b0;
    rst = 1'b1;
    #20;
    rst = 1'b0;
    #400;
    $finish;
end
always #5 clk=~clk;
always @(posedge clk) begin
    if (dm_we) begin
        $display("Data Memory Write - Addr: %h, Data: %h", dm_addr, dm_wdata);
    end
end
always @(posedge clk) begin
    if (rf_we) begin
        $display("Register File Write - Addr: %d, Data: %h", rf_waddr, rf_wdata);
    end
end
endmodule
```

实验一对应测试程序汇编与机器码（机器码可用loongarch编译工具链进行生成,也可根据指令格式自行校准）
```asm
    ori $r1,  $r0, 0x000F
    ori $r2,  $r0, 0x00F0
    ori $r3,  $r0, 0x0F00
    ori $r4,  $r2, 0x0005
    ori $r5,  $r3, 0x00F0
    ori $r6,  $r5, 0x000F
    ori $r7,  $r0, 0x0500
    ori $r7,  $r7, 0x0055
    ori $r8,  $r0, 0x0A00
    ori $r8,  $r8, 0x00AA
    ori $r9,  $r0, 0x0F00
    ori $r9,  $r9, 0x00F0
    ori $r9,  $r9, 0x000F
    ori $r10, $r7, 0x0AAA     
    ori $r10, $r7, 0x0A00
    ori $r10, $r10, 0x00AA
    ori $r11, $r10, 0x000F
    ori $r12, $r0, 0x0500
    ori $r12, $r12, 0x00A5
    ori $r13, $r12, 0x000A
    ori $r14, $r0, 0x0F00
    ori $r14, $r14, 0x00F0
    ori $r14, $r14, 0x0005
```

```
03803c01
0383c002
03bc0003
03801444
0383c065
03803ca6
03940007
038154e7
03a80008
0382a908
03bc0009
0383c129
03803d29
03aaa8ea
03a800ea
0382a94a
03803d4b
0394000c
0382958c
0380298d
03bc000e
0383c1ce
038015ce
```

2.2 实验二： addi.w与add.w指令的实现

题目说明：基于Loongarch指令集在已实现一条立即数运算指令的基础上实现addi.w与add.w指令，更深层地理解两条相同运算方式指令的区别。

测试程序：
```asm
addi.w $r1,  $r0, 1       
addi.w $r2,  $r1, 2       
addi.w $r3,  $r2, -1      
addi.w $r4,  $r3, 2047    
addi.w $r5,  $r4, -2048   
addi.w $r6,  $r5, 0       
addi.w $r7,  $r6, -6      
addi.w $r8,  $r7, 1000    
addi.w $r9,  $r8, -500    
addi.w $r10, $r9, 500     
add.w  $r11, $r1, $r2     
add.w  $r12, $r3, $r4     
add.w  $r13, $r5, $r6     
add.w  $r14, $r7, $r8     
add.w  $r15, $r9, $r10    
add.w  $r16, $r0, $r15     
add.w  $r17, $r16,$r15     
add.w  $r18, $r1, $r1     
add.w  $r19, $r18,$r1     
add.w  $r20, $r19,$r17    
ori    $r21, $r0,  0xFFF   
ori    $r22, $r21, 0x001   
ori    $r23, $r22, 0x0F0   
ori    $r24, $r1,  0xAAA   
ori    $r25, $r0,  0x000   
ori    $r26, $r25, 0x001   
ori    $r27, $r26, 0x002   
ori    $r28, $r27, 0x004   
ori    $r29, $r28, 0x008   
ori    $r30, $r29, 0x0F0 
```
机器码：
```
02800401
02800822
02bffc43
029ffc64
02a00085
028000a6
02bfe8c7
028fa0e8
02b83109
0287d12a
0010082b
0010106c
001018ad
001020ee
0010292f
00103c10
00103e11
00100432
00100653
00104674
03bffc15
038006b6
0383c2d7
03aaa838
03800019
0380073a
03800b5b
0380137c
0380239d
0383c3be
```

2.3 实验三： 算数与逻辑指令

题目说明：实现基于Loongarch指令集的ADDI.W, ADD.W, SUB.W, AND, ORI, XOR 六条指令。

测试程序：
```asm
# 测试：addi.w rd, rj, si12
addi.w $r1, $r0, 10      # $r1 = 10
addi.w $r2, $r1, 5       # $r2 = 15
addi.w $r3, $r2, -3      # $r3 = 12
addi.w $r4, $r3, 0       # $r4 = 12
addi.w $r5, $r4, -12     # $r5 = 0

# 测试：add.w rd, rj, rk
add.w $r6, $r1, $r2      # $r6 = 10 + 15 = 25
add.w $r7, $r3, $r4      # $r7 = 12 + 12 = 24
add.w $r8, $r5, $r6      # $r8 = 0 + 25 = 25
add.w $r9, $r7, $r8      # $r9 = 24 + 25 = 49
add.w $r10, $r9, $r1     # $r10 = 49 + 10 = 59

# 测试：sub.w rd, rj, rk
sub.w $r11, $r10, $r1    # $r11 = 59 - 10 = 49
sub.w $r12, $r11, $r2    # $r12 = 49 - 15 = 34
sub.w $r13, $r12, $r3    # $r13 = 34 - 12 = 22
sub.w $r14, $r13, $r4    # $r14 = 22 - 12 = 10
sub.w $r15, $r14, $r5    # $r15 = 10 - 0 = 10

# 测试：and rd, rj, rk
and $r16, $r10, $r9      # $r16 = 59 & 49 = 17
and $r17, $r2, $r3       # $r17 = 15 & 12 = 12
and $r18, $r4, $r5       # $r18 = 12 & 0 = 0
and $r19, $r16, $r17     # $r19 = 17 & 12 = 0
and $r20, $r19, $r18     # $r20 = 0 & 0 = 0

# 测试：ori rd, rj, ui12
ori $r21, $r0,  0xF0F   
ori $r22, $r21, 0x0F0   
ori $r23, $r22, 0x001   
ori $r24, $r23, 0xF00   
ori $r25, $r24, 0x000   

# 测试：xor rd, rj, rk
xor $r26, $r24, $r25     
xor $r27, $r21, $r22     
xor $r28, $r27, $r23     
xor $r29, $r28, $r24     
xor $r30, $r29, $r0    
```
机器码：
```
02802801
02801422
02bff443
02800064
02bfd085
00100826
00101067
001018a8
001020e9
0010052a
0011054b
0011096c
00110d8d
001111ae
001115cf
0014a550
00148c51
00149492
0014c613
0014ca74
03bc3c15
0383c2b6
038006d7
03bc02f8
03800319
0015e71a
0015dabb
0015df7c
0015e39d
001583be
```

2.4 实验四：访存指令

题目说明：在六条运算指令实现的基础上实现Loongarch指令集的st.w，ld.w两条指令。

测试程序：
```asm
# 初始化寄存器
addi.w $r1, $r0, 100       # $r1 = 100  (要写入内存的数据)
addi.w $r2, $r0, 200       # $r2 = 200
addi.w $r3, $r0, 300       # $r3 = 300

# 设置基址
addi.w $r4, $r0, 0             # $r4 = buffer 的地址（基址）

# 测试 st.w：将数据存入内存
st.w $r1, $r4, 0           # buffer[0] = 100
st.w $r2, $r4, 4           # buffer[1] = 200
st.w $r3, $r4, 8           # buffer[2] = 300
st.w $r1, $r4, 12          # buffer[3] = 100
st.w $r2, $r4, 16          # buffer[4] = 200

# 清除目标寄存器
addi.w $r5, $r0, 0
addi.w $r6, $r0, 0
addi.w $r7, $r0, 0
addi.w $r8, $r0, 0
addi.w $r9, $r0, 0

# 测试 ld.w：从内存加载数据到寄存器
ld.w $r5, $r4, 0           # $r5 = buffer[0] = 100
ld.w $r6, $r4, 4           # $r6 = 200
ld.w $r7, $r4, 8           # $r7 = 300
ld.w $r8, $r4, 12          # $r8 = 100
ld.w $r9, $r4, 16          # $r9 = 200
```
机器码：
```
02819001
02832002
0284b003
02800004
29800081
29801082
29802083
29803081
29804082
02800005
02800006
02800007
02800008
02800009
28800085
28801086
28802087
28803088
28804089
```

2.5 实验五：转移指令

题目说明：在八条运算和访存指令实现的基础上实现Loongarch指令集的bne，blt两条指令。

测试程序：
```asm
_start:
    ori    $r12, $r0, 0x1      # r12 = 1 (对应原t0)
    ori    $r13, $r0, 0x1      # r13 = 1 (对应原t1)
    ori    $r3, $r0, 0x4       # r3 = 4 (对应原s1)
    ori    $r8, $r0, 0x100     # r8 = 0x100 (对应原t4)
    addi.w $r4, $r0, 0x0       # r4 = array地址(从零开始)(对应原a0)
    add.w  $r9, $r4, $r8       # r9 = array地址+0x100 (对应原t5)

loop:
    add.w  $r14, $r12, $r13    # r14 = r12+r13 (对应原t2)
    ori    $r12, $r13, 0x0     # r12 = r13 (对应原t0 = t1)
    ori    $r13, $r14, 0x0     # r13 = r14 (对应原t1 = t2)
    st.w   $r13, $r4, 0        # 存储r13到r4地址
    ld.w   $r15, $r4, 0        # 从r4地址加载到r15 (对应原t3)
    bne    $r13, $r15, end     # 如果r13 != r15则跳转到end
    ori    $r0, $r0, 0         # noop
    add.w  $r4, $r4, $r3       # r4 += 4 (对应原a0 += 4)
    bne    $r4, $r9, loop      # 如果r4 != r9则跳转到loop
    ori    $r0, $r0, 0         # noop

end:
    bne    $r3, $r0, end       # 如果r3 != 0则跳转到end (死循环)

.data
array:
```
机器码：
```
0380040c
0380040d
03801003
03840008
02800004
00102089
0010358e
038001ac
038001cd
2980008d
2880008f
5c0015af
03800000
00100c84
5fffe089
03800000
5c000060
```