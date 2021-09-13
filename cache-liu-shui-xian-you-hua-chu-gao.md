# cache流水线优化

虽然cache得到了实现，但实际上仍然有一个较大的弊端：在启动状态机后无论是否命中，都要进行额外的判断。一个流水化的操作能使性能得到大幅提升。

## 逻辑设计



![&#x53CC;&#x72B6;&#x6001;&#x673A;&#x8F6C;&#x6362;&#xFF1A;&#x5706;&#x5F62;&#x4E3A;&#x4E3B;&#x72B6;&#x6001;&#x673A;&#xFF0C;&#x65B9;&#x5F62;&#x4E3A;write buffer&#x72B6;&#x6001;&#x673A;](.gitbook/assets/need%20%281%29.png)

最终总体上有两个状态机：一个称为**主状态机**，负责实现整个访存过程的主体状态转换；一个称为**WriteBuffer**状态机，专门负责在命中cache内部的时刻更新cache的内容以及相关的valid、dirty表示。两个状态机之间属于并行操作，因此可以互不干扰地高效运行。

## 状态转移

主状态转换设置如下：  


* **IDLE→IDLE**：这一拍，流水线没有新的Cache访问请求，或者有请求，但因该请求与Hit Write冲突而无法被Cache接收；
* **IDLE→LOOKUP**：这一拍，Cache接收了流水线发来的一个新的Cache访问请求 \(必定与Hit Write无冲突\)；
* **LOOKUP→IDLE**：当前处理的操作是Cache命中的，且这拍流水线没有 新的Cache 访问请求，或者有请求但因该请求与Hit Write冲突而无法被Cache接收；
* **LOOKUP→LOOKUP**：当前处理的操作是Cache命中的，且这一拍Cache接收了流水线发来的一个新的Cache访问请求，且与Hit Write无冲突；
* **LOOKUP→MISS**： 当前处理的操作是Cache缺失的；
* **MISS→MISS**：AXI总线提口模块反馈回来的id为0；
* **MISS→REPLACE**： 16字节写缓存为空， AXI总线接口模块反馈回来的wr\_rdy为1\(表示AXI总线内部 可以接收wr\_req\)。当看到wr\_rdy为1时，会对Cache发起替换的读请求，并转到REPLACE状态
* **REPLACE→REPLACE**：AXI 总线接口模块反馈回来的rd\_rdy为0。刚进入REPLACE的第一拍，会得到被替换的cache行数据，并发起wr\_req至AXI总线接口（由于 wr\_rdy为1，故wr\_req一定会被接收）；同时，对AXI 总线发起缺失Cache的读请求；
*  **REPLACE→REFILL**：AXI总线接口模块反馈回来的rd\_rdy为1，表示对AXI总线 发起的缺失Cache的读请求将被接收；
* **REFILL→REFILL**：缺失Cache行的最后一个 32位数据（即`ret_yalid==1 && ret_last==1` ）尚未返回；
* **REFILL→IDLE**： 缺失Cache行的最后一个32位数据从 AXI总线接口模块返回。

 WriteBuffer状态机里的各状态间的转换条件说明如下：

* **IDLE→IDLE**：这一拍，Write Buffer没有待写的数据，并且主状态机没有新的Hit Write；
* **IDLE→WRITE：**这一拍，Write Buffer没有待写的数据，并且主状态机发现新的Hit Write（主状态机处于LOOKUP状态且发现Store操作命中Cache）；
* **WRITE→WRITE**：这一拍，Write Buffer有待写的数据，并且主状态机发现新的Hit Write；
* **WRITE→IDLE**：这一拍，Write Buffer有待写的数据，并且主状态机没有新的Hit Write。

{% hint style="info" %}
#### 关于Hit Write以及Hit Write冲突：

Hit Write指的是访问请求来临时，**该操作为一个写操作且在cache内命中**。这一步将会把cache的脏位设置为1且更新cache内的数据。

由于两个状态机是并行的，因此当

* 主状态机在LOOKUP出现Hit Write的请求，而流水线又发来一个写后读相关的读请求；
* WriteBuffer状态机在Write状态处理，而主状态机又出现Hit Write的请求重新激活；

的时候，操作将会发生冲突。通过**LOOKUP→LOOKUP**等待过程可避免这一现象。
{% endhint %}



