# Multicores and Parallel Processing


## CMPs: 
Chip Multi-Processors，GPU中的计算单元


对于一个GPU， MEM-CAHCE对中，CACHE里包含chip multiprocessor, 每个processor又有多个 core multithreading，core multithreading又包含多个线程束（warp）。每个线程束包含32个线程，这些线程同时执行相同的指令，但可以访问不同的数据。

core multithreading则共享core资源， cmps则共享cache资源，cache则共享内存资源。

ccNUMA: cache-coherent Non-Uniform Memory Access，指的是多处理器系统中，处理器之间通过共享内存进行通信

## homogeneus 和 heterogeneous computing

homogeneous computing 指的是系统中所有处理器都具有相同的架构和功能，例如传统的CPU集群。而 heterogeneous computing 指的是系统中包含不同类型的处理器，例如CPU和GPU的混合系统。

## DVFS
Dynamic Voltage and Frequency Scaling，指的是根据系统负载动态调整处理器的电压和频率，以达到节能和性能优化的目的。

