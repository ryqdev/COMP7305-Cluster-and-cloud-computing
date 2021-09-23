# Proj 1

## How to login in

https://yukun4119.github.io/2021/09/16/ssh-gateway-script/

Use shell script to remote access my machine



## Experiment 1: CPU Performance Test

### Q1: Experiment (1.1)

```shell
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=N run
# N=1, 2, 4, 8, 16
```

#### on `master container`

* N = 1

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%206.00.44%20PM.png)



* N = 2

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%206.01.22%20PM.png)



* N = 4

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%206.01.48%20PM.png)



* N = 8

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%206.02.15%20PM.png)

* N =16

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%206.02.47%20PM.png)





#### on `physical machine`

* N = 1

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.38.08%20PM.png" style="zoom: 50%;" />



* N = 2

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.41.02%20PM.png)

* N = 4

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.42.03%20PM.png)



* N = 8

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.42.37%20PM.png)



* N = 16

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.43.12%20PM.png)





### Q2 (Discussion)

When we increase N from 1 to 16, the CPU speed `does not ` increse linearly.

The smallest value of N on `studentXX-master` is 4

The smallest value of N on `studentXX` is 6

Since my machine is **six-core**, there will six thread working parallelly if no dead-lock happens.

As for master container



To prove my point, I run the command when N = 6 on studentXX ( physical machine )

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%205.54.38%20PM.png" style="zoom:50%;" />





## Experiment 2: Effect of CPU Isolation

### Q3: Experiment (2.1)

use `cmd + shift + i` to operate multiple sessions at the same time in iTerm2

* On physical machine , run

```shell
docker stats
```

* On other three containers, run the following command at the same time

```shell
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=4 run
```



![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%207.05.11%20PM.png)

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%207.17.20%20PM.png)



### Q4 (Discussion)

* “CPU speed” measured on the three K8S containers are roughly the same.
* The CPU speed reported by student-XX-master is much lower than one in Experiment (1.1) with N=4.  Since in Experiment 2, there are three containers running at the same time, the CPU resources should be amortized by containers.
* What is the highest CPU usage (“CPU %”)  on “docker stats” is about 130%. It be higher than 100%, because there are six cores in physical machine. The sum of usage of CPU in three containers could be 600% in theory.



## Experiment 3: Memory Speed Test

### Q5 Experiment (3.1)

#### on student-XX-master container

```shell
sysbench --test=memory --memory-block-size=1M, --memory-total-size=10G --num-threads=4 --memory-oper=write run
```

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%207.23.59%20PM.png" style="zoom:50%;" />

#### on physical machine

```shell
sysbench --test=memory --memory-block-size=1M, --memory-total-size=10G --num-threads=4 --memory-oper=write run
```

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%207.23.16%20PM.png" style="zoom:50%;" />

#### Discussion:

* The memory throughput reported on master container is slower than physical machine by 17.4%
* The reported results on container vary noticeably.



### Q6 Experiment (3.2)

On all the 3 K8S containers, run the command concurrently.

```shell
sysbench --test=memory --memory-block-size=1M, --memory-total-size=10G --num-threads=4 run
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%207.34.36%20PM.png)



* master container: 23741.71 MiB/sec

* X1 container: 25576.45 MiB/sec
* X2 container: 23074.36 MiB/sec

#### Discussion:

* The memory throughputs measured on the 3 containers are roughly the same.
* The memory throughput of master container is much **lower** than that in Experiment (3.1).



### Experiment 4: File I/O Test

(0%, this part is optional. Do it if you are interested):











### Experiment 5: Interference Analysis of Co-Located Container Workloads

#### Experiment (5.1):

run the command on the same time

##### On master container:

```shell
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=8 run
```

##### On X1 container:

```shell
sysbench fileio --num-threads=1 --file-num=1 --file-total-size=2G --file-block-size=1024K --file-test-mode=seqwr --time=60 --max-requests=0 run
```

![](https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%209.01.21%20PM.png)



##### Observation and Discussion

* On master container, the CPU speed is roughly the same with the result reported in Experiment 1.1
* ?????? Experiment 4???

#### Experiment (5.2)

##### On X1 container

```
sysbench fileio --num-threads=8 --file-num=1 --file-total-size=2G --file-block-size=1024K --file-test-mode=seqwr --time=60 --max-requests=0 run
```

<img src="https://raw.githubusercontent.com/Yukun4119/BlogImg/main/img/Screenshot%202021-09-23%20at%209.07.12%20PM.png" style="zoom: 33%;" />

The writes and fsyncs on File operations is slower than Experiment 5.1



## Construct Cluster

