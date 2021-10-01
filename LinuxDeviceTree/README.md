# 设备树

Linux内核从3.x开始引入设备树的概念，用于实现驱动代码与设备信息相分离。 设备树主要需要提供设备的相关接口通讯信息，包括：握手，电平，中断，名称等。

官方对设备树的描述是，一种描述硬件资源的数据结构。它通过bootloader将硬件资源传给内核，使得内核和硬件资源描述相对独立。

设备树的主要优势：对于同一SOC的不同主板，只需更换设备树文件.dtb即可实现不同主板的无差异支持，而无需更换内核文件。


### 关于 ARM Linux：
1. DTC（device tree compiler）设备树编译器
2. .dts(device tree source) 设备树源文件，设备树源文件一般放在arch/arm/boot/dts/目录内
3. .dtsi 同样是设备树源文件，主要的作用是给多个设备树共享设备的时候可以引用，减少代码冗余
    通常dts的内容比较少，dtsi公共内容比较多，类似于公共库的感觉
4. .dtb（device tree blob）设备树二进制文件，bootloader在引导内核的时候，会预先读取.dtb到内核解析


# 设备树的组织形式和代码解释：

1. 每个设备树文件都有一个根节点，每个设备都是一个节点。
2. 节点间可以嵌套，形成父子关系，这样就可以方便的描述设备间的关系。
3. 每个设备的属性都用一组key-value对(键值对)来描述。
4. 每个属性的描述用;结束

        {                                   //根节点
            node1{                          //node1是节点名，是/的子节点
                key=value;                  //node1的属性
                ...
                node2{                      //node2是node1的子节点
                    key=value;              //node2的属性
                    ...
                }
            }                               //node1的描述到此为止
            
            node3{                          // 新的node3 
                key=value;
                ...
            }
        }


## 节点名

### 节点命名

理论个节点名只要是长度不超过31个字符的ASCII字符串即可。在Linux中，设备名通常写成形如\<name>[@<unit_address>]的形式，其中name就是设备名，最长可以是31个字符长度。unit_address一般是设备地址，用来唯一标识一个节点，下面就是典型节点名的写法：

        firmware@0203F000{
            compatible = "samsung, secure-firmware"; 
            reg = <0x0203F000 0x1000>
        }

节点的名称：**firmware**  
节点的路径： **/firmware@0203F000**  
注意：根据节点名查找节点的API的参数是不能有"@xxx"这部分的 


### 特殊节点

Linux中的设备树还包括几个特殊的节点，比如chosen，chosen节点不描述一个真实设备，而是用于firmware传递一些数据给OS，比如bootloader传递内核启动参数给内核

        chosen{
            bootargs = “console=ttySAC2, 115200”; //参数
            stdout-path = &serial_2;
        }


### 节点名的引用

当我们找一个节点的时候，我们必须书写完整的节点路径，这样当一个节点嵌套比较深的时候就不是很方便，所以，设备树允许我们用下面的形式为节点标注引用(起别名)，借以省去冗长的路径。这样就可以实现类似函数调用的效果。编译设备树的时候，相同的节点的不同属性信息都会被合并，相同节点的相同的属性会被重写，使用引用可以避免移植者四处找节点，直接在板级.dts增改即可。        

        node_nickname: node_fullname@address {
            ...
            ...
        };


下面的例子中就是直接引用了dtsi中的一个节点(&node_name)，并向其中添加/修改新的属性信息
        
        &node_nickname{
            key = <&buck2_reg>
        }
    




## 键 / 属性

在设备树中，键值对是描述属性的方式，比如，Linux驱动中可以通过设备节点中的"compatible"这个属性查找设备节点。

Linux设备树语法中定义了一些具有规范意义的属性，包括：compatible, address, interrupt等，这些信息能够在内核初始化找到节点的时候，自动解析生成相应的设备信息。此外，还有一些Linux内核定义好的，一类设备通用的有默认意义的属性，这些属性一般不能被内核自动解析生成相应的设备信息，但是内核已经编写的相应的解析提取函数，常见的有 "mac_addr"，"gpio"，"clock"，"power"。"regulator" 等等。

### compatible （常见）

定义一系列的字符串, 用来指定内核中哪个 machine_desc 可以支持本设备，即这个板子兼容哪些平台

设备节点中对应的节点信息已经被内核构造成struct platform_device。驱动可以通过相应的函数从中提取信息。compatible属性是用来查找节点的方法之一，另外还可以通过节点名或节点路径查找指定节点。dm9000驱动中就是使用下面这个函数通过设备节点中的"compatible"属性提取相应的信息，所以二者的字符串需要严格匹配。
在下面的这个dm9000的例子中，我们在相应的板级dts中找到了这样的代码块：

        ethernet@5000000 {
            compatible = "davicom,dm9000";
        }

然后我们取内核源码中找到dm9000的网卡驱动，从中可以发现这个驱动是使用的设备树描述的设备信息(这不废话么，显然用设备树好处多多)。我们可以找到它用来描述设备信息的结构体，可以看出，驱动中用于匹配的结构使用的compatible和设备树中一模一样，否则就可能无法匹配，这里另外的一点是struct of_device_id数组的最后一个成员一定是空，因为相关的操作API会读取这个数组直到遇到一个空。



### #address-cells  &  #size-cells （常见）


- #address-cells，用来描述子节点"reg"属性的地址表中用来描述首地址的cell的数量，
- #size-cells，用来描述子节点"reg"属性的地址表中用来描述地址长度的cell的数量。


        parent_node {  
            #address-cells = <2>  
            #size-cells = <1>;  

            sub_node {  
                reg = <0 0 0x1000>;   // 地址占两个cells， 长度占1个cells
            };  
        }

可以看到，此时子节点上的reg参数里面，一共有3个数（32bit），其中前两位代表了地址（64bit），后一个数（32bit）代表了占用的地址长度（0x1000）。


### reg 属性

        reg = \<[addr1 len1] [addr2 len2] [addr3 len3]>...

addr由#address-cells个uint32值组成  
len由#size-cells个uint32值组成。表明了设备使用的一个地址范围。


        /{
            compatible = "acme,coyotes-revenge";
            #address-cells = <1>;
            #size-cells = <1>;
            serial@101f2000 {  
                    compatible = "arm,pl011";  
                    reg = <0x101f2000 0x1000 >;  

                interrupts = < 2 0 >;  
                };
        ｝




### 中断 (重要)

关于中断的系统架构：

    SOC内部
             ------------------------------------------------
     内核  --|-  cpu -----  GIC---------中断控制器 ........---|---外设
             ------------------------------------------------
          


代码示例

        / {
            compatible = "acme,coyotes-revenge";
            #address-cells = <1>;
            #size-cells = <1>;
            interrupt-parent = <&intc>;//指定依附的中断控制器是intc

            serial@101f0000 {   //子节点：串口设备
                compatible = "arm,pl011";
                reg = <0x101f0000 0x1000 >; 
                interrupts = < 1 0 >; //使用父节点的（intc 并且中断2）
            };

            intc: interrupt-controller@10140000 { //intc中断控制器
                compatible = "arm,pl190";
                reg = <0x10140000 0x1000 >;
                interrupt-controller;//定义为中断控制器设备
                #interrupt-cells = <2>; // <中断号 触发类型>
            };
        ｝


#### 中断控制器

- interrupt-controller （必需） 一个空属性用来声明这个node接收中断信号，即这个node是一个中断控制器。
- #interrupt-cells （必需），是中断控制器节点的属性，用来标识这个控制器需要几个单位做中断描述符,用来描述子节点中"interrupts"属性使用了父节点中的interrupts属性的具体的哪个值。'#' 表示描述的是子节点的属性。

    
    interrupt-cells = 1 子节点的interrupt <Number>
    interrupt-cells = 2 子节点的interrupt <Number Triger_Type>
    interrupt-cells = 3 子节点的interrupt <Type Number Triger_Type>


#### 中断节点

- interrupt-parent,标识此设备节点属于哪一个中断控制器，如果没有设置这个属性，会自动依附父节点的
- interrupts,一个中断标识符列表，表示每一个中断输出信号   


示例：
    
        interrupt-parent = <&int1>;
        interrupts = <5 0>, <6 0>;



#### 关于中断类型：
    1. SPI，共享外设中断(由GIC内部的distributor来分发到相关CPU)，中断号：32~1019
    2. PPI，私有外设中断(指定CPU接收)，中断号：16~31
    3. SGI中断：软件触发中断(Software Generated Interrupt)，通常用于多核间通讯，最多支持16个SGI中断，硬件中断号从ID0~ID15。


#### 关于中断号码：
    1. 32~1019给SPI
    2. 16~31给PPI


#### 关于触发方式 bits[3:0] （其他方式可以通过组合触发实现）：

1. 0 = 内核不改变它，开机或uboot设置它是什么样就什么样
2. 1 = low-to-high edge triggered，上升沿触发
3. 2 = high-to-low edge triggered，下降沿触发
4. 4 = active high level-sensitive，高电平触发
5. 8 = active low level-sensitive，低电平触发





#### interrupts-extended 写法

当一个节点需要引用多个中断控制器中的中断时，可以使用 interrupts-extended 属性。interrupts-extended 的每个入口包含了中断控制器的 phandle 和中断标识。 interrupts-extended 只有节点需要多个中断控制器的中断时才使用，其他均不适用。

一个“interrupts-extended”属性就可以既指定“interrupt-parent”，也指定“interrupts”，比如：

    interrupts-extended = <&intc1 5 1>, <&intc2 1 0>;

