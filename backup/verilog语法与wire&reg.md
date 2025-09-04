1.Verilog 与电路的关系

Verilog 是一种硬件描述语言（HDL, Hardware Description Language）。与 C、Java 这样的软件语言不同，Verilog 并不是用来写“执行步骤”的程序，而是用来描述“电路结构和行为”。

当我们写 C 语言时，计算机会从第一行执行到最后一行，语句是顺序执行的；而 Verilog 描述的电路是并行运行的。所有电路在同一时间都在工作，就像现实世界里的电路板一样。例如：

```verilog
assign y = a & b;    //assign的含义是组合逻辑中的连续赋值
```

这行代码在 C 语言里好像是“执行一次逻辑与运算，然后把结果给 y”。但在 Verilog 里，它表示一个与门电路，一旦 a 或 b 改变，y 就会自动随之改变。学习 Verilog 时，最重要的第一点就是要把代码和硬件电路建立联系，不要把它当成普通的程序语言。

2.模块与信号

Verilog 的最小单位是模块（module），它就像电路中的一个芯片，有输入端口、输出端口。

这里用一个与门模块示例来进行说明：
```verilog
module and_gate(
    input wire a,
    input wire b,
    output wire y
);
    assign y = a & b;
endmodule
```
模块始于module，在endmodule处结束，input对应的a和b含义为输入端口，output对应y为输出端口，在声明IO端口后使用assign赋值语句或always语句来实现与门的内部逻辑。

此处的wire和reg指的是信号类型，wire表示电路中的导线，用来连接；reg表示一个寄存器，可以存储数值，但在 Verilog 中，reg 不一定就是物理寄存器，它也可能只是仿真时的变量。但在 always @(posedge clk) 时使用时，才真正对应到触发器。这里后文会加以探讨，再此不过多赘述。

3.赋值方式：阻塞与非阻塞

阻塞赋值 =：在过程块内立刻执行并更新左值；代码按书写顺序一条一条往下走，后面的语句能立刻看到前面刚写入的新值。

非阻塞赋值 <=：右值先被计算并排队，左值的更新延后到本时间点最后统一生效；同一时间点里，无论你在多少地方写了 <=，大家“约好一起换新值”，因此彼此在本时间点的右值计算阶段都只能看到旧值。

可以从如下代码的不同结果来观测二者的差距：
阻塞：
```verilog
//初值：x=1，y=2
x = y;      // x 立刻变 2
y = x + 1;  // 此时 x 已是 2，y 变 3
// 结果：x=2, y=3
```
非阻塞：
```verilog
//初值：x=1，y=2
x <= y;      // 记录：x 下一步设为 2，但并不立即赋值
y <= x + 1;  // 记录：y 下一步设为 1+1=2（x 仍是旧值 1）
             // 本时间点最后统一生效
// 结果：x=2, y=2
```

4.常见语法结构
条件语句
```verilog
if (sel)
    y = a;
else
    y = b;
```
case语句
```verilog
case(op)
    2'b00: y = a + b;
    2'b01: y = a - b;
    2'b10: y = a & b;
    default: y = 0;
endcase
```
循环语句
```verilog
integer i;
always @(*) begin
    sum = 0;
    for (i=0; i<4; i=i+1)
        sum = sum + data[i];
end
```
参数化
```verilog
parameter WIDTH = 8;    //这样模块的位宽就可以灵活调整
reg [WIDTH-1:0] counter;
```

5.层次化设计及：模块实例化

在实际设计中，一个系统由很多子模块组成，模块可以像“搭积木”一样被嵌套。

示例：两个与门组成一个三输入与门：
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
在该代码之中，and_gate 是之前写好的与门模块，在同一文件或同一根目录下。u1、u2 是将与门and_gate实例化的两个具体电路，实现了 y = t & c = (a & b) & c = a & b & c  的三输入与门设计，涉及的思路就是把模块当作“零件”，实例化就是把零件拼起来。
