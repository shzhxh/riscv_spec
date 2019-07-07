### RISC-V 手册上卷(riscv-spec-v2.2)

#### 1. 介绍

   设计目标。不使用商业ISA的原因。不使用OpenRISC的原因。

   1. RISC-V ISA概述

      共有3种类型的指令：基本指令，标准扩展指令和特殊扩展指令。基本指令被命名为I，是不可更改，必须被包含的指令，它们都是整数指令。标准扩展指令包括M(扩展乘除指令)，A(扩展原子指令)，F(单精度浮点指令扩展)，D(双精度浮点指令扩展)，G(IMAFD五种指令的总和，提供了完整的通用标量指令集，是默认工具链支持的指令集)。特殊扩展指令只用于特定领域，不适用于全部通用领域。

   2. 指令长度编码

      基本指令固定是32位的，然而扩展指令可以是变长的，但必须是16的倍数。当低2位不是11时指令长度为16位，当低2位是11时指令长度为32位，当低5位为11111时指令长48位，当低6位为111111时指令长64位，当低7位全1则要计算[14:12]位的数才能得出指令长度。

      基本指令集是小端系统，但扩展指令集即可做成小端也可做成大端。

   3. 异常、陷阱和中断

      异常是指令错误引起的，陷阱是线程自主调用的，中断是外部事件引起的。

#### 2. RV32I指令集

   RV32I是基本指令集，它被设计足以支持一个现代的操作系统，而且除了A扩展外它基本上能模拟所有的其它扩展。RV32I共计47条指令。

#####   1. 程序员模型

      用户可见的寄存器有3种：1个恒为0的x0，31个通用寄存器x1-x31，pc。

#####   2. 基本指令格式

      在基本ISA里，所有指令都是32位且必须4字节对齐。共有四种基本格式：R-type是寄存器-寄存器操作，I-type是寄存器-立即数操作，S-type，U-type。
    
      立即数字段里放的是立即数的位置，而不是它本身。
    
      R-type: **funct7**[31:25] **rs2**[24:20] **rs1**[19:15] **funct3**[14:12] **rd**[11:7] **opcode**[6:0]
    
      I-type: **imm[11:0]**[31:20] **rs1**[19:15] **funct3**[14:12] **rd**[11:7] **opcode**[6:0]
    
      S-type: **imm[11:5]**[31:25] **rs2**[24:20] **rs1**[19:15] **funct3**[14:12] **imm[4:0]**[11:7] **opcode**[6:0]
    
      U-type: **imm[31:12]**[31:12] **rd**[11:7] **opcode**[6:0]

#####   3. 立即数编码的变体

      B是S的变体，J是U的变体
    
      B-type: **imm[12]imm[10:5]**[31:25] **rs2**[24:20] **rs1**[19:15] **funct3**[14:12] **imm[4:1]imm[11]**[11:7] **opcode**[6:0]
    
      J-type:**imm[20]imm[10:1]imm[11]imm[19:12]**[31:12] **rd**[11:7] **opcode**[6:0]

#####   4. 整数计算指令(共计22条指令)

      | imm+rs1+funct3+rd+opcode                 | funct3 | opcode | 意义                                       |
      | ---------------------------------------- | ------ | ------ | ---------------------------------------- |
      | ADDI rd, rs1, imm                        | ADDI   | OP-IMM | rd = rs1 + imm                           |
      | SLTI rd, rs1, imm (set less than immediate) | SLTI   | OP-IMM | rd = (rs1<imm) ? 1 : 0(用于有符号数)           |
      | SLTIU rd, rs1, imm                       | SLTIU  | OP-IMM | rd = (rs1<imm) ? 1 : 0(用于无符号数);**注sltiu rd,rs,1的意思是rd = (rs==0)?1:0;** |
      | ANDI rd, rs1, imm                        | ANDI   | OP-IMM | rd = rs1 & imm                           |
      | ORI rd, rs1, imm                         | ORI    | OP-IMM | rd = rs1 \| imm                          |
      | XORI rd, rs1, imm                        | XORI   | OP-IMM | rd = rs1 ^ imm                           |
      | SLLI rd, rs1, imm                        | SLLI   | OP-IMM | rd = rs1 << imm[4:0]                     |
      | SRLI rd, rs1, imm                        | SRLI   | OP-IMM | rd = (unsigned) rs1 >> imm[4:0]          |
      | SRAI rd, rs1, imm                        | SRAI   | OP-IMM | rd = (signed) rs1 >> imm[4:0]            |
    
      | imm+rd+opcode                            | opcode | 意义                    |
      | ---------------------------------------- | ------ | --------------------- |
      | LUI rd, imm (load upper immediate)       | LUI    | rd = (imm << 12)      |
      | AUIPC rd, imm (add upper immediate to pc) | AUIPC  | rd = (imm << 12) + PC |
    
      | funct7+rs2+rs1+funct3+rd+opcode | funct7  | funct3 | opcode | 意义                                       |
      | ------------------------------- | ------- | ------ | ------ | ---------------------------------------- |
      | ADD rd, rs1, rs2                | 0000000 | ADD    | OP     | rd = rs1 + rs2                           |
      | SLT rd, rs1, rs2                | 0000000 | STL    | OP     | rd = (rs1<rs2) ? 1 : 0 (用于有符号数)          |
      | SLTU rd, rs1, rs2               | 0000000 | STLU   | OP     | rd = (rs1<rs2) ? 1 : 0 (用于无符号数);**注sltu rd, x0, rs的意思是rd = (rs != 0) ? 1 : 0;** |
      | AND rd, rs1, rs2                | 0000000 | AND    | OP     | rd = rs1 & rs2                           |
      | OR rd, rs1, rs2                 | 0000000 | OR     | OP     | rd = rs1 \| rs2                          |
      | XOR rd, rs1, rs2                | 0000000 | XOR    | OP     | rd = rs1 ^ rs2                           |
      | SLL rd, rs1, rs2                | 0000000 | SLL    | OP     | rd = rs1 << (rs2 & 0x1F )                |
      | SRL rd, rs1, rs2                | 0000000 | SRL    | OP     | rd = (unsigned) rs1 >> (rs2 & 0x1F)      |
      | SUB rd, rs1, rs2                | 0100000 | SUB    | OP     | rd = rs1 - rs2                           |
      | SRA rd, rs1, rs2                | 0100000 | SRA    | OP     | rd = (signed) rs1 >> (rs2 & 0x1F)        |
    
      NOP指令的编码：ADDI x0, x0, 0

#####   5. 控制转移指令(8条指令)

- 无条件跳转(U-type or J-type)
| 指令                                      | 编码                              | 意义                          |
| ----------------------------------------- | --------------------------------- | ----------------------------- |
| JAL rd, imm(jump and link)                | imm+rd+JAL(opcode)                | rd = pc + 4; pc += imm;       |
| JALR rd, rs1, imm(jump and link register) | imm+rs1+0(funct3)+rd+JALR(opcode) | rd = pc + 4; pc += rs1 + imm; |

- 条件分支(S-type or B-type)
| imm+rs2+rs1+funct3+imm+opcode | funct3   | opcode | 意义           |
| ----------------------------- | -------- | ------ | ------------ |
| BEQ rs1,rs2,offset            | BEQ      | BRANCH | if(rs1==rs2) |
| BNE rs1,rs2,offset            | BNE      | BRANCH | if(rs1!=rs2) |
| BLT/BLTU rs1, rs2, offset     | BLT/BLTU | BRANCH | if(rs1<rs2)  |
| BGE/BGEU rs1, rs2, offset     | BGE/BGEU | BRANCH | if(rs1>=rs2) |

#####   6. LOAD和STORE指令(8条指令)

      | imm+rs1+funct3+rd+opcode | funct3 | opcode | 意义            |
      | ------------------------ | ------ | ------ | --------------- |
      | LW rd, imm(rs1)          | 宽度   | LOAD   | rd=mem(rs1+imm) |
      | LH rd, imm(rs1)          | 宽度   | LOAD   | rd=mem(rs1+imm) |
      | LHU rd, imm(rs1)         | 宽度   | LOAD   | rd=mem(rs1+imm) |
      | LB rd, imm(rs1)          | 宽度   | LOAD   | rd=mem(rs1+imm) |
      | LBU rd, imm(rs1)         | 宽度   | LOAD   | rd=mem(rs1+imm) |
    
      | imm+rs2+rs1+funct3+imm+opcode | funct3 | opcode | 意义               |
      | ----------------------------- | ------ | ------ | ---------------- |
      | SW(32位)                       | 宽度     | STORE  | mem(rs1+imm)=rs2 |
      | SH(16位)                       | 宽度     | STORE  | mem(rs1+imm)=rs2 |
      | SB(8位)                        | 宽度     | STORE  | mem(rs1+imm)=rs2 |
    
      ​

#####   7. 内存模型(2条指令)

本节已 **过时**，新的内存模型正在修订中。
| imm[11:0]                         | rs1     | funct3  | rd      | opcode   | 意义        |
| --------------------------------- | ------- | ------- | ------- | -------- | --------- |
| imm[11:8]保留，imm[7:4]前续，imm[3:0]后续 | 保留，应置为0 | FENCE   | 保留，应置为0 | MISC-MEM | 顺序化IORW访问 |
| imm[11:0]保留，应置为0                  | 保留，应置为0 | FENCE.I | 保留，应置为0 | MISC-MEM | 同步指令和数据流  |


​      

#####   8. 控制和状态寄存器指令(6条指令)
- CSR指令

| csr+rs1+funct3+rd+opcode | rs1       | funct3 | opcode | 意义                         |
| ------------------------ | --------- | ------ | ------ | ---------------------------- |
| csrrw rd, csr, rs1       | source    | CSRRW  | SYSTEM | tmp=rs1; rd=csr; csr=tmp     |
| csrrs rd, csr, rs1       | source    | CSRRS  | SYSTEM | tmp=rs1; rd=csr; csr\|=tmp   |
| csrrc rd, csr, rs1       | source    | CSRRC  | SYSTEM | tmp=rs1; rd=csr; csr &= !tmp |
| csrrwi rd, csr, rs1      | uimm[4:0] | CSRRWI | SYSTEM | rd=csr; csr=uimm[4:0]        |
| csrrsi rd, csr, rs1      | uimm[4:0] | CSRRSI | SYSTEM | rd=csr; csr\|=uimm[4:0]      |
| csrrci rd, csr, rs1      | uimm[4:0] | CSRRCI | SYSTEM | rd=csr; csr \&= !uimm[4:0]   |

- 计时器和计数器(伪指令)
  
| csr+rs1+funct3+rd+opcode | csr           | rs1  | funct3 | opcode | 意义             |
| ------------------------ | ------------- | ---- | ------ | ------ | -------------- |
| rdcycle[h] rd            | RDCYCLE[H]    | 0    | CSRRS  | SYSTEM | 读取cycle CSR    |
| rdtime[h] rd             | RDTIME[H]     | 0    | CSRRS  | SYSTEM | 读取time CSR     |
| rdinstrent[h] rd         | RDINSTRENT[H] | 0    | CSRRS  | SYSTEM | 读取instrent CSR |

##### 9. 环境调用和断点(2条指令)

      | funct12 | rs1  | funct3 | rd   | opcode | 意义           |
      | ------- | ---- | ------ | ---- | ------ | ------------ |
      | ECALL   | 0    | PRIV   | 0    | SYSTEM | 用于产生对执行环境的请求 |
      | EBREAK  | 0    | PRIV   | 0    | SYSTEM | 调试器用它返回调试环境  |
    
      ECALL指令只是触发一个当前特权级下的exception，并不执行其它操作。
    
      注：ECALL可以在各自的特权级下产生不同的exception,所以可以选择性地委派call exception。对于Unix-like操作系统来说，一个典型的用法是通过U-mode下的调用切换到S-mode。

#### 3. RV32E：RV32I的精简版，为嵌入式系统而设计

      1. 程序员模型：通用寄存器从31个(x1~x31)减少到15个(x1~x15)
      2. 指令集：与RV32I相同，但计时器和计数器伪指令不是必须的。
      3. 扩展：可进行M,A,C扩展，只有两种特权级(用户模式和机器模式)

#### 4. RV64I：RV32I的变体，只描述与RV32I的不同之处。

   1. 寄存器状态：寄存器扩展为64位，且支持64位用户地址空间。

   2. 整数计算指令(增加了9条特有指令)

      RV64I的指令长度也是32位的，操作的是64位数值。但也提供了变种指令来操作32位数值，这些指令的操作码后面都增加了后缀“W"，是RV64I特有的指令。

      | imm       | rs1  | funct3 | rd   | opcode    |
      | --------- | ---- | ------ | ---- | --------- |
      | imm[11:0] | src  | ADDIW  | dest | OP-IMM-32 |
      | imm[4:0]  | src  | SLLIW  | dest | OP-IMM-32 |
      | imm[4:0]  | src  | SRLIW  | dest | OP-IMM-32 |
      | imm[4:0]  | src  | SRAIW  | dest | OP-IMM-32 |

      | funct7  | rs2  | rs1  | funct3    | rd   | opcode |
      | ------- | ---- | ---- | --------- | ---- | ------ |
      | 0000000 | src2 | src1 | ADDW      | dest | OP-32  |
      | 0000000 | src2 | src1 | SLLW/SRLW | dest | OP-32  |
      | 0100000 | src2 | src1 | SUBW/SRAW | dest | OP-32  |

      ​

   3. Load和Store指令(增加了3条指令)

      增加LD(64位)，LWU(32位无符号)，SD(64位）三条指令。

   4. 系统指令：与RV32I完全相同，只是操作的对象换成了64位的。需要注意的是，伪指令rdcycle,rdtime,rdinstret分别读取的是相应寄存器的全部64位(cycle,time,instret计数器)，所以rdcycleh,rdtimeh,rdinstreth在RV64I里就不需要了，是非法的。

#### 5. RV128I

#### 6. M：整数乘除扩展

1. 乘法操作

   | funct7+rs2+rs1+funct3+rd+opcode | funct7 | funct3          | opcode | 意义                                  |
   | ------------------------------- | ------ | --------------- | ------ | ----------------------------------- |
   | MUL/MULH\[[S]U] rd, rs1, rs2    | MULDIV | MUL/MULH\[[S]U] | OP     | MUL将乘法的低位放入目标寄存器，MULH则是放高位          |
   | MULW rd, rs1, rs2               | MULDIV | MULW            | OP-32  | 仅用于RV64，计算源寄存器的低32位，再用MUL指令获取高32位的值 |

   ​

2. 除法操作

   | funct7+rs2+rs1+funct3+rd+opcode | funct7 | funct3          | opcode | 意义                     |
   | ------------------------------- | ------ | --------------- | ------ | ---------------------- |
   | DIV[U]/REM[U] rd, rs1, rs2      | MULDIV | DIV[U]/REM[U]   | OP     | rs1/rs2。DIV提供商，REM提供余数 |
   | DIV[U]W/REM[U]W rd, rs1, rs2    | MULDIV | DIV[U]W/REM[U]W | OP-32  | 仅用于RV64,源寄存器的低32位相除。   |

   ​

#### 7. A：原子扩展

原子指令是原子性地“读-改-写”内存，以支持同一内存空间中多个硬件线程之间的同步。共有两种形式的原子指令，load-reserved/store-conditional指令和原子“获取-操作”内存指令。它们都支持各种内存一致性排序，都支持RCsc内存一致性模型。

1. 原子指令的顺序

   基本RISCV ISA使用的是宽松的内存模型，用FENCE指令来施加额外的次序约束。

   为了更有效地支持一致性，每个原子指令都有两个位，aq和rl，用于指定其他的RISC-V harts所观察到的额外的内存排序约束。

   如果两个比特都清空了，那么就不会对原子内存操作施加额外的排序约束。如果只设置了aq位，则将原子内存操作视为**获取访问**，即：在**获取访问**操作之前，在这个RISC-V hart中，没有任何后续的内存操作可以被观察到。如果只设置rl位，则将原子内存操作视为一个**释放访问**，即：在这个RISC-V hart中，任何早期内存操作之前，释放内存操作都不能观察到。如果aq和rl位都设置了，则在所有早期的内存操作之前或后期的内存操作之后，在相同的RISCV hart上，原子内存操作是顺序一致的且不能被观察到，而只能被全局次序中的其它hart观察到。

2. load-reserved/store-conditional指令

   | funct5+aq+rl+rs2+rs1+funct3+rd+opcode | funct5 | rs2  | opcode | 意义                                                         |
   | ------------------------------------- | ------ | ---- | ------ | ------------------------------------------------------------ |
   | lr rd, (rs1)                          | LR     | 0    | AMO    | 原子交换的前半部分：load且将内存地址注册为reservation        |
   | sc rd, rs2, (rs1)                     | SC     | src  | AMO    | 原子交换的后半部分：(rs1)=rs2如果内存地址的reservation有效,成功则rd为0,失败则rd非0. |

   在完成上一个LR之前,一个SC指令不能被另一个RISC-V hart观察到。由于LR/SC序列的原子性，在LR和一个成功的SC之间，任何hart的内存操作都不能被观测到。LR/SC序列可以通过在SC指令上设置aq位来赋予**获取语义**。LR/SC序列可以通过在LR指令上设置rl位来赋予**释放语义**。在LR指令上设置aq和rl位，并在SC指令上设置aq位，使得LR/SC序列与其他顺序一致的原子操作顺序一致。

   如果LR/SC的位均没有设置，则LR/SC序列是可以观测到的。这适用于并行下降操作。

3. 原子内存操作atomic memory operation(AMO)

   | funct5+aq+rl+rs2+rs1+funct3+rd+opcode | funct5        | opcode | 意义                         |
   | ------------------------------------- | ------------- | ------ | -------------------------- |
   | amoswap.w/d rd, rs2, (rs1)            | AMOSWAP.W/D   | AMO    | rd=(rs1);                  |
   | amoadd.w/d rd, rs2, (rs1)             | AMOADD.W/D    | AMO    | rd=(rs1);(rs1)=rd+rs2      |
   | amoand.w/d rd, rs2, (rs1)             | AMOAND.W/D    | AMO    | rd=(rs1);(rs1)=rd&rs2      |
   | amoor.w/d rd, rs2, (rs1)              | AMOOR.W/D     | AMO    | rd=(rs1);(rs1)=rd\|rs2     |
   | amoxor.w/d rd, rs2, (rs1)             | AMOXOR.W/D    | AMO    | rd=(rs1);(rs1)=rd^rs2      |
   | amomax[u].w/d rd, rs2, (rs1)          | AMOMAX[U].W/D | AMO    | rd=(rs1);(rs1)=max(rd,rs2) |
   | amomin[u].w/d rd, rs2, (rs1)          | AMOMIN[U].W/D | AMO    | rd=(rs1);(rs1)=min(rd,rs2) |

   原子内存操作(AMO)指令为多处理器同步执行"读-修改-写"操作，并使用r类型指令格式进行编码。这些AMO指令从rs1代表的内存地址原子地加载数据值，将该值放入rd，对rd的值和rs2的值进行操作，然后将结果存储回rs1的内存地址。AMOs可以在内存中操作64位(仅限RV64)或32位的数据。对于RV64，AMOs总是对rd中的值进行符号扩展。rs1中的地址必须与操作数的大小自然对齐(即，8字节对齐64位字，4字节对齐32位字)，否则就会造成未对齐地址异常。

   支持的操作有swap, add, and, or, xor, 符号和非符号整数的maximum和minimum。除了排序约束，这些AMOs也可以实现并行下降操作，一般会通过写入x0的方式来丢弃返回值。

   为了帮助实现多处理器同步，AMOs可选地提供发布一致性语义。如果设置了aq位，那么在这个riscv - v hart中，就不能观察到在AMO之前发生的内存操作。相反，如果设置了rl位，那么其他的RISC-V harts将不会在这个riscv - v hart的内存访问之前观察AMO。

#### 8. F：浮点扩展

1. F寄存器

   F扩展增加了32个浮点寄存器和1个浮点控制和状态寄存器fcsr。浮点寄存器是f0-f31，每个32位宽。浮点控制和状态寄存器则包含了浮点单元的操作模式和异常状态。

2. 浮点控制和状态寄存器

3. NaN的生成与传播

4. 低能的算术

5. 单精度Load和Store指令

6. 单精度浮点计算指令

7. 单精度浮点转换和移动指令

8. 单精度浮点比较指令

9. 单精度浮点类型指令

#### 9. D：双精度扩展

1. D寄存器

   D将32个浮点寄存器扩展到64位，F寄存器现在可以保存32位或64位浮点值了。

2. NaN Boxing of Narrower Values

3. 双精度Load和Store指令

4. 双精度浮点计算指令

5. 双精度浮点转换和移动指令

6. 双精度浮点比较指令

7. 双精度浮点类型指令

#### 10. Q：四精度扩展

#### 11. L：十进制浮点扩展

#### 12. C：压缩指令扩展

#### 13. B：位操作扩展

#### 14. J：动态翻译语言扩展

#### 15. T：事物内存扩展

#### 16. P：封装的单指令多数据流扩展

#### 17. V：向量操作扩展

#### 18. N：用户级中断扩展

#### 19. G：RV32/64G 指令集列表

#### 20. RISC-V汇编程序员手册

- 寄存器在标准调用规约中的角色

  | 寄存器    | ABI名字 | 描述            | 由谁保存 |
  | ------ | ----- | ------------- | ---- |
  | x0     | zero  | 固定为0          | 无    |
  | x1     | ra    | 返回地址          | 调用者  |
  | x2     | sp    | 栈指针           | 被调用者 |
  | x3     | gp    | 全局指针          | 无    |
  | x4     | tp    | 线程指针          | 无    |
  | x5     | t0    | 临时寄存器/备用连接寄存器 | 调用者  |
  | x6-7   | t1-2  | 临时寄存器         | 调用者  |
  | x8     | s0/fp | 保存寄存器/桢指针     | 被调用者 |
  | x9     | s1    | 保存寄存器         | 被调用者 |
  | x10-11 | a0-1  | 函数参数/返回值      | 调用者  |
  | x12-17 | a2-7  | 函数参数          | 调用者  |
  | x18-27 | s2-11 | 保存寄存器         | 被调用者 |
  | x28-31 | t3-6  | 临时寄存器         | 调用者  |

- 普通伪指令

  | 伪指令                          | 对应的基本指令                                  | 意义                         |
  | ---------------------------- | ---------------------------------------- | -------------------------- |
  | la rd, symbol                | auipc rd, symbol[31:12] ; addi rd, rd, symbol[11:0] | rd = pc + symbol，载入地址      |
  | l{b\|h\|w\|d} rd, symbol     | auipc rd, symbol[31:12]; l{b\|h\|w\|d} rd, symbol[11:0]\(rd) | 载入全局地址                     |
  | s{b\|h\|w\|d} rd, symbol, rt | auipc rt, symbol[31:12]; s{b\|h\|w\|d} rd, symbol[11:0]\(rt) | 存储全局地址                     |
  | nop                          | addi x0, x0, 0                           | 空操作                        |
  | li rd, immediate             | 无穷序列                                     | rd = immediate，存储立即数       |
  | mv rd, rs                    | addi rd, rs, 0                           | rd = rs + 0,复制寄存器          |
  | not rd, rs                   | xori rd, rs, -1                          | rd = rs ^ -1, rs的补码        |
  | neg rd, rs                   | sub rd, x0, rs                           | rd = 0 - rs, 补码            |
  | negw rd, rs                  | subw rd, x0, rs                          | rd = 0 - rs, 补码            |
  | sext.w rd, rs                | addiw rd, rs, 0                          | rd = rs + 0, 符号扩展          |
  | seqz rd, rs                  | sltiu rd, rs, 1                          | rd = (rs == 0) ? 1 : 0;    |
  | snez rd, rs                  | sltu rd, x0, rs                          | rd = (rs != 0) ? 1 : 0;    |
  | sltz rd, rs                  | slt rd, rs, x0                           | rd = (rs < 0) ? 1 : 0;     |
  | sgtz rd, rs                  | slt rd, x0, rs                           | rd = (rs > 0) ? 1 : 0;     |
  | beqz rs, offset              | beq rs, x0, offset                       | if (rs == 0), branch       |
  | bnez rs, offset              | bne rs, 0x, offset                       | if( rs != 0), branch       |
  | blez rs, offset              | bge x0, rs, offset                       | if(rs <= 0), branch        |
  | bgez rs, offset              | bge rs, x0, offset                       | if(rs >= 0), branch        |
  | bltz rs, offset              | blt rs, x0, offset                       | if(rs < 0), branch         |
  | bgtz rs, offset              | blt x0, rs, offset                       | if(rs > 0), branch         |
  | bgt rs, rt, offset           | blt rt, rs, offset                       | if(rs > rt), branch        |
  | ble rs, rt, offset           | bge rt,rs, offset                        | if(rs <= rt), branch       |
  | bgtu rs, rt, offset          | bltu rt, rs, offset                      | if( rs > rt), branch       |
  | bleu rs, rt, offset          | bgeu rt, rs, offset                      | if(rs <= rt), branch       |
  | j offset                     | jal x0, offset                           | pc += offset               |
  | jal offset                   | jal x1, offset                           | pc += offset; x1 = pc + 4; |
  | jr rs                        | jalr x0, rs, 0                           | pc = rs + 0                |
  | jalr rs                      | jalr x1, rs, 0                           | pc = rs + 0; x1 = pc + 4;  |
  | ret                          | jalr x0, x1, 0                           | pc = x1 + 0                |
  | call offset                  | auipc x6, offset[31:12]; jalr x1, x6, offset[11:0] | pc += offset; x1 = pc + 4; |
  | tail offset                  | auipc x6, offset[31:12]; jalr x0, x6, offset[11:0] | pc += offset               |
  | fence                        | fence iorw, iorw                         | 隔离内存和IO                    |

  ​

- 访问CSR的伪指令

  | 伪指令             | 对应的基本指令                  | 意义                        |
  | --------------- | ------------------------ | ------------------------- |
  | rdinstret[h] rd | csrrs rd, instret[h], x0 | rd = instret[h]，读取指令退休计数器 |
  | rdcycle[h] rd   | csrrs rd, cycle[h], x0   | rd = cycle[h]，读取循环计数器     |
  | rdtime[h] rd    | csrrs rd, time[h], x0    | rd = time[h], 读取实时时钟      |
  | csrr rd, csr    | csrrs rd, csr, x0        | rd = csr, 读取csr           |
  | csrw csr, rs    | csrrw x0, csr, rs        | csr = rs, 写入csr           |
  | csrs csr, rs    | csrrs x0, csr, rs        | csr \|= rs, csr置位         |
  | csrc csr, rs    | csrrc x0, csr, rs        | csr &= !rs, csr清位         |
  | csrwi csr, imm  | csrrwi x0, csr, imm      | csr = imm, 立即数写入csr       |
  | csrsi csr, imm  | csrrsi x0, csr, imm      | csr \|= imm, 立即数置位csr     |
  | csrci csr, imm  | csrrci x0, csr, imm      | csr &= !imm, 立即数清位csr     |

  ​

#### 21. 扩展RISC-V

#### 22. ISA子集命名约定

#### 23. 历史与致谢

#### 
