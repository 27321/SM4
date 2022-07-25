# SM4
【项目名称】SM4的普通实现及使用讲过的SIMD进行优化

【项目简介】进行SM4的普通实现，然后使用SIMD对其进行优化，通过测试得出，优化大概提升了七倍左右的速度。

【代码说明】
对SM4进行优化，在一般的加密解算法中，我们的每一轮的轮函数，Xi+4=Xi异或T（Xi+1异或Xi+2异或Xi+3异或rki），合成置换T分为非线性变换τ和线性变换L，非线性变换τ由四个并行的S盒组成，然后对非线性变换τ进行线性变换L，在这里，可以将S盒与后续的循环移位L合并，L(S(x0),S(x1),S(x2),S(x3))=L(S(x0)<<24)异或L(S(x1)<<16)异或L(S(x2)<<8)异或L(S(x3))，可以定义四个查找表：BOX0[x]=L(S(x)<<24)，BOX1[x]=L(S(x)<<16)，BOX2[x]=L(S(x)<<8)，BOX3[x]=L(S(x))，然后返回BOX0[x0]异或BOX1[x1]异或BOX2[x2]异或BOX3[x3]。可以使用SIMD指令并行查表进行优化，上面的算法我们是每32-bit一组，使用256-bit寄存器，我们可以将此分为八组并行进行循环移位、异或等操作，加速加解密的速度。

（1）定义合并类型与几个BOX盒，lo是低位打包，hi是高位打包，其中BOX0[x]=L(S(x)<<24)，BOX1[x]=L(S(x)<<16)，BOX2[x]=L(S(x)<<8)，BOX3[x]=L(S(x))。
![image](https://user-images.githubusercontent.com/105579212/180782976-3c9a510f-e439-4937-905e-839f0a099faa.png)

（2）使用_mm256_loadu_si256()加载数据，利用（1）所定义的合并类型合并每组128bit数据的某32bit字。
![image](https://user-images.githubusercontent.com/105579212/180782959-77e1975b-ccc1-498f-9a36-d02579ddab74.png)

（3）vindex为重排的顺序，使用_mm256_shuffle_epi8（）进行按字节为单位的重排。
![image](https://user-images.githubusercontent.com/105579212/180782943-bb1f0b7d-4481-4f0b-b180-e9d054859343.png)

（4）进行32轮迭代，enc=0表示加密，enc=1表示解密，加密密钥正序，解密密钥逆序使用。使用_mm256_xor_si256（）进行并行异或，使用_mm256_i32gather_epi32（）进行并行查表，查找表BOX，即将S盒与循环移位L合并，大大加速了加解密速度。
![image](https://user-images.githubusercontent.com/105579212/180782923-2f65ab80-3794-4627-9261-156de72a2be1.png)

（5）最后进行恢复分组并填充。
![image](https://user-images.githubusercontent.com/105579212/180782899-b40ab708-ffef-4188-84af-7dfb764bd8c0.png)

（6）加密函数即传入enc=0，解密函数即传入enc=1。
![image](https://user-images.githubusercontent.com/105579212/180782874-247d77fb-ebfb-4210-85e9-9ebba4e8ac82.png)

【运行结果】
![image](https://user-images.githubusercontent.com/105579212/180782383-e48c3db4-c0f4-432c-b20d-4957cb4f58c4.png)
