这个图是 Drools规则引擎（Drools Rule Engine）核心运行架构图，如果结合你熟悉的ERP、PMC、MES制造场景来理解，会非常容易。

先看整体：

                    ┌─────────────┐
                    │   Rules     │
                    └──────┬──────┘
                           │
                           ▼
                   Production Memory
                           │
                           ▼
                  ┌─────────────────┐
                  │ Pattern Matcher │
                  └────────┬────────┘
                           │
                           ▼
                      Agenda
                           │
                           ▼
                      执行规则
                           ▲
                           │
                   Working Memory
                           ▲
                           │
                    ┌──────┴──────┐
                    │    Facts     │
                    └──────────────┘

1. Rules（规则）

图左侧：

Rules


就是业务规则。

例如PMC规则：

规则1：
材料=ALUMINUM
且
尺寸>30×30×80

→ 加工线A

规则2：
材料=STAINLESS

→ 加工线B


在Drools中写成：

rule "Aluminum Route"

when
    Part(
        materialCategory=="ALUMINUM",
        length > 30,
        width > 30,
        height > 80
    )

then
    setLine("LINE_A");

end


这些规则会被编译后放进：

Production Memory

2. Production Memory（规则库）

图左边数据库图标。

Production Memory


可以理解成：

规则仓库


里面存放：

Rule1
Rule2
Rule3
Rule4
...
Rule1000


例如制造企业：

材料规则
工艺规则
设备规则
品质规则
交期规则
VIP客户规则


全部在这里。

可以理解为：

ERP中的工艺知识库

3. Facts（事实数据）

图右侧：

Facts


Facts就是当前需要判断的数据。

例如一个订单：

{
   "partNo":"A1001",
   "material":"AL5052",
   "length":50,
   "width":40,
   "height":100,
   "weight":10
}


这个订单放入Drools后：

session.insert(part);


就变成：

Fact


在制造业里Fact可能是：

工单
WorkOrder

零件
Part

产品
Product

客户订单
SalesOrder

设备状态
Machine

4. Working Memory（工作内存）

图右边数据库图标：

Working Memory


存放当前运行的数据。

例如：

工单001

材料: AL5052
尺寸:50×40×100
数量:100


进入Drools：

session.insert(workOrder);


此时就在：

Working Memory


里面。

可以把它理解成：

当前等待判断的业务数据池

5. Pattern Matcher（模式匹配器）

这是整个Drools最核心的部分。

图中央：

Pattern Matcher


内部实现：

Phreak
(Rete改进算法)


作用：

规则 和 数据
进行匹配


例如：

Working Memory里面有：

AL5052

尺寸:
50×40×100


规则库里面有：

Rule1

材料=ALUMINUM
长度>30
宽度>30
高度>80


Pattern Matcher做的事情：

条件1匹配
√

条件2匹配
√

条件3匹配
√

条件4匹配
√


结果：

Rule1 命中


再举个例子

有1000条规则：

Rule1
Rule2
...
Rule1000


有10000个工单：

WO001
WO002
...
WO10000


Pattern Matcher利用：

Phreak/Rete


进行高效匹配。

避免：

10000 × 1000 次暴力判断


这也是Drools性能好的原因。

6. Agenda（议程）

图最下面：

Agenda


很多人第一次学Drools最容易忽略它。

Agenda：

待执行规则队列


例如：

一个零件同时满足：

Rule A

Rule B

Rule C


匹配器发现：

A 命中
B 命中
C 命中


不会立即执行。

而是放到：

Agenda


变成：

Agenda

1.Rule A
2.Rule B
3.Rule C


例如你的PMC场景：

Rule1:
铝材件
→ A线

Rule2:
VIP订单
→ VIP专线

Rule3:
加急订单
→ 插单


同一个工单：

AL6061
VIP
加急


会同时命中：

Rule1
Rule2
Rule3


Agenda决定：

谁先执行
谁后执行

7. Salience（优先级）

Agenda里面经常配合：

salience


使用。

例如：

rule "VIP"
salience 100

rule "Urgent"
salience 50

rule "Normal"
salience 10


执行顺序：

VIP
↓
Urgent
↓
Normal


制造业最常见：

客户优先级
>
交期优先级
>
普通工艺规则

整个流程在PMC中的运行过程

假设有一个工单：

{
    "material":"AL5052",
    "length":50,
    "width":40,
    "height":100,
    "customerLevel":"VIP"
}


步骤一：

ERP产生工单

WO10001


步骤二：

放入Working Memory

session.insert(workOrder);


步骤三：

Pattern Matcher匹配规则

发现：

Rule:
铝件 -> A线

√ 命中

Rule:
VIP -> VIP专线

√ 命中


步骤四：

加入Agenda

Agenda

VIP Rule
Aluminum Rule


步骤五：

按照优先级执行

VIP Rule
先执行

Aluminum Rule
后执行


步骤六：

输出结果

{
   "workOrder":"WO10001",
   "line":"VIP_LINE"
}

用制造业语言总结这张图

你可以把这张图理解成：

Rules
↓
工艺工程师制定的规则

Production Memory
↓
工艺知识库

Facts
↓
工单、零件、产品数据

Working Memory
↓
当前待判断订单池

Pattern Matcher
↓
自动判断哪些规则满足

Agenda
↓
决定规则执行顺序

Result
↓
产线、设备、工艺路线、优先级


所以从ERP/PMC视角看，Drools本质上就是一个：

“工艺决策引擎（Decision Engine）”

输入：

材料
尺寸
重量
工期
客户等级
设备状态


输出：

产线分配
工艺路线
优先级
质量判定
风险预警


而图中的 Pattern Matcher + Agenda，正是Drools能够代替大量if-else、并管理数百上千条制造规则的核心。