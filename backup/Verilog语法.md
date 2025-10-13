### 1.Verilog 与电路的关系

Verilog 是一种硬件描述语言（HDL，Hardware Description Language），它的出现旨在解决传统软件语言在硬件设计中的局限性。与 C、Java 等面向程序执行流程的软件语言不同，Verilog 并不是用来写“程序逻辑”的，而是用来描述硬件电路的结构与行为。换句话说，Verilog 代码本质上是对实际电路的抽象描述，而非计算机的指令序列。

在软件语言中，程序是顺序执行的：计算机从第一行语句开始，逐行执行，直到最后一行结束。这种顺序逻辑适合处理算法和数据处理任务，但在硬件设计中并不适用，因为实际的电子电路是并行工作的。电路中的所有元件，例如与门、或门、触发器等，都会在同一时间响应输入信号的变化。

举例来说，下面是一行 Verilog 代码：

```verilog
assign y = a & b;    // assign 表示连续赋值，用于描述组合逻辑
```

在传统的软件语言中，这行代码可能被理解为“执行一次逻辑与运算，然后将结果赋给变量 y”。然而在 Verilog 中，这条语句实际上描述了一个与门电路：只要输入 a 或 b 发生变化，输出 y 就会自动随之更新，而不需要额外的指令来“触发”这一变化。换言之，assign 表达的是硬件的持续行为，而非一次性操作。

因此，学习 Verilog 时，最重要的一点就是建立代码与硬件的直观联系。初学者往往会把 Verilog 当成普通的程序语言来理解，这是学习 Verilog 的一个常见误区。正确的学习方法应当是：看到代码，就能够在脑海中形成对应的电路图，理解每一条语句在电路中扮演的角色，而非仅仅关注程序的执行顺序。

通过这种方式，Verilog 不仅可以用来描述组合逻辑电路，还可以描述时序逻辑、寄存器传输级（RTL）设计等复杂电路行为，从而成为数字电路设计和 FPGA/ASIC 开发中不可或缺的重要工具。

### 2.模块与信号

在 Verilog 中，模块（module）是设计的最小单位，类似于电路中的一个独立芯片。一个模块可以具有输入端口、输出端口以及内部逻辑结构，它封装了电路的功能，实现了设计的模块化和层次化。通过模块的组合，可以构建复杂的数字系统。

下面用一个简单的与门模块示例来说明模块的基本结构：
```verilog
module and_gate(
    input wire a,    // 输入端口 a
    input wire b,    // 输入端口 b
    output wire y    // 输出端口 y
);
    assign y = a & b; // 使用 assign 描述组合逻辑
endmodule
```
从代码中可以看到：
        模块定义：模块以 module 关键字开始，以 endmodule 结束，模块名为 and_gate。
        端口声明：使用 input 和 output 关键字声明模块的输入输出端口，端口类型可以是 wire 或 reg。
        内部逻辑实现：在声明端口之后，通过 assign 语句或 always 语句实现模块内部逻辑。例如这里的 assign y = a & b; 描述了一个与门逻辑，即当输入 a 和 b 变化时，输出 y 会自动更新。

模块的端口不仅用于模块内部逻辑的连接，还可以与其他模块的端口相连，实现模块间信号的传递

在 Verilog 中，信号类型是模块设计的核心概念，最常用的类型包括 wire 和 reg：
wire：表示电路中的导线，用于连接模块之间或模块内部的组合逻辑。它的值由连接到它的输出驱动源驱动，不能在 always 块中直接赋值。
reg：表示寄存器或仿真变量，可以存储数值。在组合逻辑中，reg 可以在 always 块中作为变量使用，但并不一定对应物理寄存器；只有在时序逻辑中，例如在 always @(posedge clk) 块中使用时，reg 才对应触发器。

理解 wire 与 reg 的区别，是 Verilog 学习中非常重要的一步，它决定了信号的驱动方式和模块内部逻辑实现方式。为了避免混淆，初学者应首先掌握组合逻辑和时序逻辑中信号类型的不同用法，后续章节再深入讨论 reg 在时序电路中的作用。

### 3.赋值方式：阻塞与非阻塞

在 Verilog 中，赋值语句是描述电路行为的核心手段。根据赋值的执行方式不同，Verilog 将赋值分为**阻塞赋值（Blocking Assignment）和非阻塞赋值（Non-blocking Assignment）**两类，分别用符号 = 和 <= 表示。这两种赋值方式在仿真行为和硬件映射上有本质区别。

#### 3.1阻塞赋值（=）

阻塞赋值在过程块（如 always 块）中立即执行，并立即更新左值。赋值语句按书写顺序逐条执行，后续语句可以直接看到前面语句更新后的新值，因此在同一个过程块内，赋值操作是顺序执行的。

示例：
```verilog
//初值：x=1，y=2
x = y;      // x 立刻变 2
y = x + 1;  // 此时 x 已是 2，y 变 3
// 结果：x=2, y=3
```
在上例中，第二条语句使用的是更新后的 x 值，因此 y 的结果不同于初始值。阻塞赋值适合描述组合逻辑或需要严格顺序执行的操作。

#### 3.2非阻塞赋值（<=）

非阻塞赋值则不同。它在赋值时先计算右值并排队等待更新，左值的实际更新会延迟到当前仿真时间点的末尾统一生效。也就是说，同一时间点内的所有非阻塞赋值“约定俗成”地同时更新左值，在右值计算阶段看到的都是旧值。

示例：
```verilog
//初值：x=1，y=2
x <= y;      // 记录：x 下一步设为 2，但并不立即赋值
y <= x + 1;  // 记录：y 下一步设为 1+1=2（x 仍是旧值 1）
             // 本时间点最后统一生效
// 结果：x=2, y=2
```
在非阻塞赋值中，x 和 y 在本时间点计算右值时仍保持初始值，因此赋值结果与阻塞赋值不同。非阻塞赋值通常用于时序逻辑（如触发器），以保证在同一个时钟沿下所有寄存器的同步更新。

### 4. 常见语法结构

在 Verilog 中，除了基本的模块和赋值语句之外，还提供了多种常用语法结构，用于实现条件判断、选择分支、循环操作以及参数化设计。掌握这些语法结构，是高效编写数字电路模块的基础。

#### 4.1 条件语句（if-else）

条件语句用于根据输入信号的状态选择不同的操作，通常用于组合逻辑或控制逻辑的实现。其基本形式如下：
```verilog
if (sel)
    y = a;
else
    y = b;
```
当 sel 为真（逻辑 1）时，输出 y 被赋值为 a；
当 sel 为假（逻辑 0）时，输出 y 被赋值为 b。
if-else 语句可嵌套使用，用于处理更复杂的条件判断。

#### 4.2 case 语句

case 语句用于多路选择的情况，类似于其他编程语言中的 switch-case。它根据输入的不同取值选择不同的操作，通常用于实现多功能选择器或算术逻辑单元（ALU）功能。示例：
```verilog
case(op)
    2'b00: y = a + b;
    2'b01: y = a - b;
    2'b10: y = a & b;
    default: y = 0;
endcase
```
当 op 为 2'b00 时，输出 y 等于 a + b；

当 op 为 2'b01 时，输出 y 等于 a - b；

当 op 为 2'b10 时，输出 y 等于 a & b；

default 分支用于处理未覆盖的情况，保证输出有确定值。

使用 case 语句可以使代码更清晰、逻辑结构更易读。

#### 4.3 循环语句（for）

Verilog 允许在过程块中使用循环语句来重复执行某些操作，尤其适合处理数组、寄存器组或生成器逻辑。示例：
```verilog
integer i;
always @(*) begin
    sum = 0;
    for (i=0; i<4; i=i+1)
        sum = sum + data[i];
end
```
for 循环用于将数组 data[0] 到 data[3] 的元素累加到 sum 中；

integer i; 声明循环变量 i；

循环语句通常与 always 块结合使用，实现组合逻辑或初始化操作。

#### 4.4 参数化（parameter）

参数化是 Verilog 提供的一种灵活机制，可以让模块在不同设计场景下使用不同配置，而无需修改模块内部代码。示例：
```verilog
parameter WIDTH = 8;    //这样模块的位宽就可以灵活调整
reg [WIDTH-1:0] counter;
```
parameter WIDTH = 8; 定义了一个参数 WIDTH，表示模块中信号的位宽；

使用参数化后，只需修改参数值，就可以调整模块内部寄存器或总线的位宽，实现模块的可重用性和可扩展性；

参数化广泛应用于计数器、FIFO、寄存器文件和总线接口设计中。

### 5.层次化设计及：模块实例化

在实际的数字系统设计中，一个完整的系统通常由多个子模块组成。为了提高设计的可读性、可维护性和复用性，Verilog 提供了层次化设计的方法。通过模块实例化，可以像“搭积木”一样，将已定义好的模块组合成更复杂的电路，从而实现从简单功能单元到复杂系统的逐层构建。

下面通过一个具体示例来说明模块实例化的使用。假设我们已经定义了一个基本的与门模块 and_gate，现在希望利用两个这样的与门构建一个三输入与门 and3。在代码中，我们首先定义了输入端口 a、b、c 和输出端口 y，并声明了一个中间信号 t，用于连接两个与门模块。接下来，我们通过实例化两个与门来实现三输入与门的功能：第一个与门实例 u1 将 a 和 b 作为输入，输出到中间信号 t；第二个与门实例 u2 将中间信号 t 和输入 c 作为输入，输出到最终端口 y。通过这种方式，三输入与门的逻辑关系 y = (a & b) & c = a & b & c 得以实现。

```verilog
module and3(
    input wire a, b, c,
    output wire y
);
    wire t;

    and_gate u1 (.a(a), .b(b), .y(t));
    and_gate u2 (.a(t), .b(c), .y(y));
endmodule
```
模块实例化的本质是将模块当作功能单元来复用，通过端口连接实现信号传递。每个模块封装了特定功能，可以在不同设计中重复使用，而无需重新编写内部逻辑。这种层次化设计不仅使代码结构清晰，还便于维护和调试：子模块可以先独立仿真，验证正确后再进行系统集成。同时，修改子模块内部实现不会影响整体系统，只要接口保持一致，系统功能依然正确。通过模块实例化，Verilog 设计过程可以实现高效的模块化和结构化，使复杂系统的开发更加直观和可管理。


### 6.关于wire和reg的讨论

在 Verilog 中，wire 和 reg 是最常见的两种数据类型，它们的使用方式与设计风格紧密相关。wire 表示一根纯粹的“连线”，不能在过程块中赋值，只能通过 assign 语句或模块端口驱动。而 reg 表示一个能够在过程块中存储数值的变量，可以在 always 或 initial 块里赋值。

从建模的角度看，二者并不是绝对对立的。对于同一段组合逻辑，如果我们选择 wire，往往需要写出多条 assign 语句，把不同条件下的结果组合起来；而如果使用 reg，则可以在 always @(*) 块里通过 if-else 或 case 描述逻辑流程。这种写法更接近程序的分支结构，阅读时更直观，尤其当逻辑分支较多时，可读性优于大量 assign 的拼接。

不过，即使使用 reg 在 always 中建模组合逻辑，实质上综合工具仍然会生成纯粹的组合电路，而不是存储单元。换句话说，reg 在这里只是语法上的一种“容器”，帮助我们以过程化的方式组织代码，而不一定对应硬件里的寄存器。为了保持接口一致性或表达设计意图，有时我们甚至会在 always 中得到一个 reg 变量，然后再通过 assign 把它传递给输出端口。

因此，wire 与 reg 的选择更多地影响代码风格而非硬件本质。assign 驱动的 wire 更简洁直接，适合描述简单的逻辑连接；always 驱动的 reg 更便于表达复杂的条件逻辑，能让设计思路更清晰。两者在最终综合出的电路层面往往没有区别，关键在于如何让代码更易读、更易维护。

接下来时四路选择器用两种方式的不同代码实现，4:1 MUX（输入 4 路，选择信号 sel，输出 y）

写法一：wire + assign （结构化风格）
```verilog
module mux4_assign (
    input  wire [1:0] sel,     // 选择信号
    input  wire a, b, c, d,    // 四路输入
    output wire y              // 输出
);
    assign y = (sel == 2'b00) ? a :
               (sel == 2'b01) ? b :
               (sel == 2'b10) ? c :
                                d ;
endmodule
```

特点：逻辑一行写完，简洁明了。
问题：如果分支很多（例如 8:1 MUX 或复杂条件），可读性会迅速下降。

写法二：reg + always （过程化风格）
```verilog
module mux4_always (
    input  wire [1:0] sel,     // 选择信号
    input  wire a, b, c, d,    // 四路输入
    output wire y              // 输出
);
    reg y_reg;

    always @(*) begin
        case (sel)
            2'b00: y_reg = a;
            2'b01: y_reg = b;
            2'b10: y_reg = c;
            2'b11: y_reg = d;
            default: y_reg = 1'b0; // 保底逻辑
        endcase
    end

    assign y = y_reg; // 输出接口仍用 wire
endmodule
```

特点：逻辑分支清晰，容易扩展，尤其适合多条件判断或复杂组合逻辑。
额外一行：用 assign 把 reg 变量传递给 output，保证输出端口仍是 wire 类型。

### 7.实验部分
完成本章的学习后，请读者完成以下实践任务：

任务一：跑马灯
用实验环境中提供的三个文件：scroller.v（设计文件）、scroller.xdc（约束文件）和testbench.v（测试文件），构建Vivado工程，完成跑马灯设计的仿真和上板。

scroller.v:
```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 2017/12/11 00:10:26
// Design Name: 
// Module Name: scroller
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////

module scroller #(
    parameter CNT_1S = 27'd100_000_000
)(
    input         clk,
    input         resetn,
    output reg [15:0] led
);

reg [26:0] cnt;
wire cnt_eq_1s;
assign cnt_eq_1s = cnt==CNT_1S;
always @(posedge clk)
begin
    if (!resetn)
    begin
        cnt <= 27'd0;
    end
    else if (cnt_eq_1s)
    begin
        cnt <= 27'd0;
    end
    else
    begin
        cnt <= cnt + 1'b1;
    end
end

always @(posedge clk)
begin
    if (!resetn)
    begin
        led <= 16'hfffe;
    end
    else if (cnt_eq_1s)
    begin
        led <= {led[14:0],led[15]};
    end
end
endmodule
```

scroller.xdc:
```verilog
#ʱź
set_property PACKAGE_PIN AC19 [get_ports clk]
set_property CLOCK_DEDICATED_ROUTE BACKBONE [get_nets clk]
create_clock -period 10.000 -name clk -waveform {0.000 5.000} [get_ports clk]

#reset
set_property PACKAGE_PIN Y3 [get_ports resetn]

#LED
set_property PACKAGE_PIN K23 [get_ports {led[0]}]
set_property PACKAGE_PIN J21 [get_ports {led[1]}]
set_property PACKAGE_PIN H23 [get_ports {led[2]}]
set_property PACKAGE_PIN J19 [get_ports {led[3]}]
set_property PACKAGE_PIN G9 [get_ports {led[4]}]
set_property PACKAGE_PIN J26 [get_ports {led[5]}]
set_property PACKAGE_PIN J23 [get_ports {led[6]}]
set_property PACKAGE_PIN J8 [get_ports {led[7]}]
set_property PACKAGE_PIN H8 [get_ports {led[8]}]
set_property PACKAGE_PIN G8 [get_ports {led[9]}]
set_property PACKAGE_PIN F7 [get_ports {led[10]}]
set_property PACKAGE_PIN A4 [get_ports {led[11]}]
set_property PACKAGE_PIN A5 [get_ports {led[12]}]
set_property PACKAGE_PIN A3 [get_ports {led[13]}]
set_property PACKAGE_PIN D5 [get_ports {led[14]}]
set_property PACKAGE_PIN H7 [get_ports {led[15]}]

set_property IOSTANDARD LVCMOS33 [get_ports clk]
set_property IOSTANDARD LVCMOS33 [get_ports resetn]
set_property IOSTANDARD LVCMOS33 [get_ports {led[*]}]
```

testbench:
```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 2017/12/11 00:21:48
// Design Name: 
// Module Name: testbench
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module testbench(

    );
reg clk;
reg resetn;
initial
begin
    $dumpfile("dump.vcd");
    $dumpvars;
    clk = 0;
    resetn = 1'b0;
    #200;
    resetn = 1'b1;
end
    always #5 clk <= ~clk;
    
    scroller #(
        .CNT_1S(  27'd100 )
    ) u_scroller (
        .clk(clk),
        .resetn(resetn),
        .led()
    );
endmodule
```

实验二：数码管实验

在完成“跑马灯”实验后，读者应进一步熟悉FPGA对外设的控制方法。本实验的目标是通过Verilog实现一个简单的数码管显示系统，使开发者能够掌握数码管的显示原理和动态扫描方法。实验硬件平台同样为Artix-7 FBG676 FPGA开发板，实验中使用开发板自带的七段数码管作为输出显示器件。

实验设计的主要内容是让数码管循环显示数字0~9，或根据输入信号显示特定数字字符。通过Verilog编写驱动模块seg_display.v，设计者需要将输入的4位二进制数转换为七段显示的对应段码，同时利用时钟分频实现数码管动态扫描，从而在多个数码管上实现稳定显示。约束文件seg_display.xdc用于绑定数码管的段选和位选信号至实际引脚，确保设计在硬件上正确显示。测试文件testbench.v则用于验证显示逻辑的正确性，在仿真环境中输出波形观察段码变化是否与预期一致。

完成设计后，读者需在Vivado中进行功能仿真与时序仿真，确认段码逻辑正确无误。随后下载至开发板，观察数码管是否能够稳定循环显示数字或根据设定变化。通过该实验，读者将理解七段数码管的显示规律、动态扫描原理以及Verilog在时序控制和信号驱动方面的实现方法，为后续更复杂的外设接口实验奠定基础。

设计文件seg_display.v：
```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Design Name: Seven Segment Display
// Module Name: seg_display
// Target Devices: Artix-7
// Description: 循环显示0~9的数码管动态扫描显示
//////////////////////////////////////////////////////////////////////////////////

module seg_display #(
    parameter CNT_1S = 27'd100_000_000
)(
    input  wire clk,
    input  wire resetn,
    output reg [7:0] seg,       // 段选输出（a~g,dp）
    output reg [3:0] an         // 位选输出（4位数码管）
);

reg [26:0] cnt;
reg [3:0]  num;
wire cnt_eq_1s;

assign cnt_eq_1s = (cnt == CNT_1S - 1);

always @(posedge clk or negedge resetn) begin
    if (!resetn)
        cnt <= 0;
    else if (cnt_eq_1s)
        cnt <= 0;
    else
        cnt <= cnt + 1'b1;
end

// 数码管显示数字0~9循环变化
always @(posedge clk or negedge resetn) begin
    if (!resetn)
        num <= 4'd0;
    else if (cnt_eq_1s)
        num <= (num == 4'd9) ? 4'd0 : num + 1'b1;
end

// 位选扫描（4位循环）
reg [1:0] scan_sel;
always @(posedge clk or negedge resetn) begin
    if (!resetn)
        scan_sel <= 2'd0;
    else
        scan_sel <= scan_sel + 1'b1;
end

always @(*) begin
    case (scan_sel)
        2'b00: an = 4'b1110;
        2'b01: an = 4'b1101;
        2'b10: an = 4'b1011;
        2'b11: an = 4'b0111;
    endcase
end

// 段码译码
always @(*) begin
    case (num)
        4'd0: seg = 8'b1100_0000;
        4'd1: seg = 8'b1111_1001;
        4'd2: seg = 8'b1010_0100;
        4'd3: seg = 8'b1011_0000;
        4'd4: seg = 8'b1001_1001;
        4'd5: seg = 8'b1001_0010;
        4'd6: seg = 8'b1000_0010;
        4'd7: seg = 8'b1111_1000;
        4'd8: seg = 8'b1000_0000;
        4'd9: seg = 8'b1001_0000;
        default: seg = 8'b1111_1111;
    endcase
end

endmodule
```

约束文件seg_display.xdc:
```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Design Name: Seven Segment Display
// Module Name: seg_display
// Target Devices: Artix-7
// Description: 循环显示0~9的数码管动态扫描显示
//////////////////////////////////////////////////////////////////////////////////

module seg_display #(
    parameter CNT_1S = 27'd100_000_000
)(
    input  wire clk,
    input  wire resetn,
    output reg [7:0] seg,       // 段选输出（a~g,dp）
    output reg [3:0] an         // 位选输出（4位数码管）
);

reg [26:0] cnt;
reg [3:0]  num;
wire cnt_eq_1s;

assign cnt_eq_1s = (cnt == CNT_1S - 1);

always @(posedge clk or negedge resetn) begin
    if (!resetn)
        cnt <= 0;
    else if (cnt_eq_1s)
        cnt <= 0;
    else
        cnt <= cnt + 1'b1;
end

// 数码管显示数字0~9循环变化
always @(posedge clk or negedge resetn) begin
    if (!resetn)
        num <= 4'd0;
    else if (cnt_eq_1s)
        num <= (num == 4'd9) ? 4'd0 : num + 1'b1;
end

// 位选扫描（4位循环）
reg [1:0] scan_sel;
always @(posedge clk or negedge resetn) begin
    if (!resetn)
        scan_sel <= 2'd0;
    else
        scan_sel <= scan_sel + 1'b1;
end

always @(*) begin
    case (scan_sel)
        2'b00: an = 4'b1110;
        2'b01: an = 4'b1101;
        2'b10: an = 4'b1011;
        2'b11: an = 4'b0111;
    endcase
end

// 段码译码
always @(*) begin
    case (num)
        4'd0: seg = 8'b1100_0000;
        4'd1: seg = 8'b1111_1001;
        4'd2: seg = 8'b1010_0100;
        4'd3: seg = 8'b1011_0000;
        4'd4: seg = 8'b1001_1001;
        4'd5: seg = 8'b1001_0010;
        4'd6: seg = 8'b1000_0010;
        4'd7: seg = 8'b1111_1000;
        4'd8: seg = 8'b1000_0000;
        4'd9: seg = 8'b1001_0000;
        default: seg = 8'b1111_1111;
    endcase
end

endmodule
```

仿真文件testbench.v:
```verilog
`timescale 1ns / 1ps

module testbench_seg();
reg clk;
reg resetn;

initial begin
    clk = 0;
    forever #5 clk = ~clk;   // 100MHz
end

initial begin
    resetn = 0;
    #100;
    resetn = 1;
end

seg_display #(
    .CNT_1S(27'd100)
) u_seg_display (
    .clk(clk),
    .resetn(resetn),
    .seg(),
    .an()
);

endmodule
```
实验三：状态机实验

在掌握基本的逻辑控制与外设驱动之后，状态机实验旨在帮助读者理解有限状态机（Finite State Machine，FSM）在数字电路设计中的作用与实现方法。本实验以Artix-7 FBG676开发板为平台，通过Verilog实现一个典型的有限状态机控制系统，展示如何利用状态机实现有序控制与逻辑判断。

实验内容可以设计为一个简单的交通灯控制器、自动门开关控制器或按键输入序列识别器。以交通灯控制器为例，设计者需要定义若干个状态（如红绿灯），并根据时钟计数或传感输入在状态之间进行转换。每个状态下输出不同的控制信号，用以点亮对应颜色的LED灯。Verilog模块state_machine.v负责描述状态转移逻辑与输出控制，状态转移条件通过时钟和复位信号实现同步。约束文件state_machine.xdc完成输入按键与LED输出引脚的绑定，而testbench.v通过仿真输入信号变化来验证状态转移的正确性与输出响应是否符合预期。

在Vivado中完成编译与仿真后，将程序下载至开发板，观察LED灯的变化是否符合交通灯的工作规律。通过此实验，读者不仅能够理解有限状态机的设计方法，还能体会其在复杂控制逻辑中的核心地位。掌握状态编码、状态转移图设计以及在Verilog中使用组合逻辑与时序逻辑描述FSM的技巧，将为后续系统控制类设计打下坚实基础。

设计文件：traffic_fsm.v
```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Design Name: FSM Traffic Light Controller
// Description: 红绿灯状态机控制实验
//////////////////////////////////////////////////////////////////////////////////

module traffic_fsm #(
    parameter CNT_1S = 27'd100_000_000
)(
    input  wire clk,
    input  wire resetn,
    output reg [2:0] led_rgy   // {红, 黄, 绿}
);

reg [26:0] cnt;
wire tick = (cnt == CNT_1S - 1);

always @(posedge clk or negedge resetn) begin
    if (!resetn)
        cnt <= 0;
    else if (tick)
        cnt <= 0;
    else
        cnt <= cnt + 1'b1;
end

// FSM 状态定义
typedef enum reg [1:0] {
    S_RED   = 2'd0,
    S_GREEN = 2'd1,
    S_YELLOW= 2'd2
} state_t;

state_t state, next_state;

// 状态转移逻辑
always @(posedge clk or negedge resetn) begin
    if (!resetn)
        state <= S_RED;
    else if (tick)
        state <= next_state;
end

always @(*) begin
    case (state)
        S_RED:    next_state = S_GREEN;
        S_GREEN:  next_state = S_YELLOW;
        S_YELLOW: next_state = S_RED;
        default:  next_state = S_RED;
    endcase
end

// 输出控制
always @(*) begin
    case (state)
        S_RED:    led_rgy = 3'b100;
        S_GREEN:  led_rgy = 3'b001;
        S_YELLOW: led_rgy = 3'b010;
        default:  led_rgy = 3'b000;
    endcase
end

endmodule
```

约束文件：traffic_fsm.xdc
```verilog
# 时钟输入
set_property PACKAGE_PIN AC19 [get_ports clk]
create_clock -period 10.000 [get_ports clk]

# 复位输入
set_property PACKAGE_PIN Y3 [get_ports resetn]

# LED 输出（红、黄、绿）
set_property PACKAGE_PIN K23 [get_ports {led_rgy[0]}]
set_property PACKAGE_PIN J21 [get_ports {led_rgy[1]}]
set_property PACKAGE_PIN H23 [get_ports {led_rgy[2]}]

set_property IOSTANDARD LVCMOS33 [get_ports clk]
set_property IOSTANDARD LVCMOS33 [get_ports resetn]
set_property IOSTANDARD LVCMOS33 [get_ports {led_rgy[*]}]
```

仿真文件：testbench.v
```verilog
`timescale 1ns / 1ps

module testbench_fsm();
reg clk;
reg resetn;

initial begin
    clk = 0;
    forever #5 clk = ~clk;
end

initial begin
    resetn = 0;
    #50;
    resetn = 1;
end

traffic_fsm #(
    .CNT_1S(27'd50)
) u_traffic_fsm (
    .clk(clk),
    .resetn(resetn),
    .led_rgy()
);

endmodule
```