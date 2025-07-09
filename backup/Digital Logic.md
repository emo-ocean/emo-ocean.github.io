## 逻辑电路

###逻辑门

逻辑门御三家：

<img width="340" height="98" alt="Image" src="https://github.com/user-attachments/assets/f430aa6f-b9d1-4058-89ef-58f2fa1aff11" />

其他门电路均可由御三家来实现，例如异或门：

<img width="611" height="380" alt="Image" src="https://github.com/user-attachments/assets/e53fe675-c735-4401-a4e4-3ba987cdd772" />

其余三个只需要已有门前加非即可：

<img width="362" height="115" alt="Image" src="https://github.com/user-attachments/assets/829adac3-f65d-4c6b-b755-ca63efbb726d" />


### 布尔代数
用来描述逻辑电路的代数方法，所有变量值非0即1

运算律太多了自己学去，证明方法也在内：https://www.geeksforgeeks.org/digital-logic/boolean-algebra/
甚至存在一些布尔运算计算器：https://www.boolean-algebra.com/
时至今日此类运算可以很快通过ai或计算器得到答案，但毕竟是很底层的逻辑层面的计算，希望除了会画真值表之外还会使用venn图或其他逻辑证明过程来对其进行验证。

在游玩Turing Complete的模拟创造简易计算机过程中，感受到简单情况下从真值表生成布尔代数式更加普遍的的方法是卡诺图（Karnaugh maps），可以当作通解或者集大成者。

<img width="320" height="320" alt="Image" src="https://github.com/user-attachments/assets/a2a23f10-7824-4520-8b80-acd7a8cc7225" />

此处需要提及一个sum of products和product of sums的概念，从语法上讲前词为主语，因此SOP的含义为先与后或的表达方式，相反地POS就是先或后与的逻辑表达。

卡诺图的横轴纵轴设置问题：三元则xy与z，四元则x1x2与x3x4，五元则绘制两个四元图，通过x5来控制两张图，这里也凸显出卡诺图的弊端，涉及参数过多会变得很复杂以至于更高元的情况无法使用。

<img width="320" height="350" alt="Image" src="https://github.com/user-attachments/assets/8006f55b-1c2b-4a1b-a4e6-1f59c60f513a" />

在卡诺图中如若圈1来判断范式则会生成sop形式的范式，也是常规去生成的；pos则是通过圈0来实现，值得一提的是d值为don‘t care conditioin，把他当作0或1没有区别。

<img width="256" height="320" alt="Image" src="https://github.com/user-attachments/assets/48cafa8c-96a9-4268-b6cd-ea93b832e85c" />

### 门级原语

Gate Level Primitives, Verilog中用来描述常用逻辑门的代码。
