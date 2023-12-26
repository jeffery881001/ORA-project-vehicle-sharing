# ORA-project-vehicle-sharing

## Table of Contents
- [Background and Problem Statement](#background-and-problem-statement)
- [Methodology](#methodology)
  - [Phase one model](#phase-one-model)
  - [Phase two greedy algorithm](#phase-two-greedy-algorithm)
- [Example](#example)
- [Numerical Study](#numerical-study)
- [Major contribution](#major-contribution)
- [Limitation](#limitation)
- [Conclusion](#conclusion)
- [Reference](#reference)

## Background and Problem Statement
In this project, our objective is to establish a scheduling system for an autonomous vehicle (referred as AV) sharing company. 
The system will receive orders from customers. And in each order, we will receive information regarding the pickup location, delivery location, pickup time and delivery time.
Based on these information, our system will optimally determine the number of AVs we should dispatched and the trip chain for each AV.

The entire process is illustrated in the figure below. 
Initially, we acquire orders from our customers. 
Then, utilizing our system, we determine that two AVs are required to fulfill these orders.

The first AV is assigned to serve the third order and then returns to the sink node. 
Meanwhile, the second AV begins by serving the first order, followed by the second order, before ultimately returning to the sink node.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/process.png)

In our system, each AV is dispatched from the depot at the beginning of the day and returns to the depot at the end of the day for maintenance. 
We may receive orders both before and after dispatching the vehicles.
Therefore, We divide our relocation plan problem into two phases to simplify the problem-solving process.

As depicted in the figure below, a reserved order is one received before dispatching vehicles, while a real-time order is received after dispatching the vehicles.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/order.jpg)

In phase one, we handle reserved orders(orders received before dispatching) using operations research methods for optimization.
In phase two, we address real-time orders(orders received after dispatching) using a greedy algorithm.
The reason we don't rerun the optimization in the second phase is that, in real-time scenarios, customers want to know immediately whether we can dispatch an AV to pick them up. 
Therefore, we need a method that allows us to promptly determine whether to accept or decline that order.
In the following section, we will introduce how we find the optimal trip chain in both phase one and phase two. 

## Methodology

In this section, we will introduce our model using in phase one and phase two

### Phase one model

In phase model, we use the model in Jiaqi et al(2017) to find the optimal trip chain based on reserved input

#### Input data
Our input data comes from received orders, providing information about pickup location, delivery location, pickup time, and delivery time.

#### Set and indices 

- $I$: set of the orders
- $i^-$: pickup location for order $i\in I$
- $i^+$: delivery location for order $i\in I$
- $t_{i^-}$: start time for order $i\in I$
- $t_{i^+}$: end time for order $i\in I$
- $o$: source node
- $d$: sink node
- $A_r = \lbrace (i,j)| i,j \in I, i \  can \ reach \ in \ j\rbrace$: $A_r$ is a set contains all reachable link
- Service link $(i^-,i^+)$: Each order $i$ is served by service link
- Relocation link $(i^+,j^-)$: after deliver at delivery node $i^+$, AV needs to be relocated to pickup node $j^-$ for next order
- $A_1 = \lbrace (i^-,i^+)|\forall i \in I \rbrace$: $A_1$ is a set that contains all sevice link $(i^-,i^+)$
- $A_2 = \lbrace (i^+,j^-)|\forall (i,j) \in I \rbrace$: $A_2$ is a set that contains all reachable relocation link $(i^+,i^-)$
- $A= A_1 \cup A_2 \cup \lbrace (o,i^-) \rbrace _{i\in I} \cup \lbrace (i^+,d) \rbrace _{i\in I} \cup \lbrace (o,d) \rbrace $, A is a set that conatins all possible links in our relocation problem

#### Parameter

- $F$: Maximum number of AV we can dispatch
- $f$: Fleet cost per vehicle includes expenses related to AV maintenance and refueling
- $d_i$: The dispatch cost refers to the expense associated with initially dispatch an AV to $i^-$ or collect an AV at the end from $i^+$
- $d_{i,j}$: The relocation cost represents the driving expense from $i^+$ to $j^-$ where $(i^+,j^-)\in A_2$. And it contains fuel usage or toll fees
- $l_i$: The cost of losing order $i$
- $c_{ij}$: total cost from location $i$ to location $j$
- $m_{ij}$: link capacity for link $(i,j)$.  $m_{ij}=1$ if $(i,j)\in A \backslash \lbrace (o,d) \rbrace$ and $m_{ij}=F$ if $(i,j)\in (o,d)$

#### Decision variable

- $y_{ij}$ is a binary variable and $y_{i,j}=1$ if link $(i,j)$ is used by an AV; otherwise, $y_{ij}$ is zero

#### Objective

Our objective is minimize the total cost of our system, and the objective function is as following equation

- $min_{y_{ij}}\sum_{((i,j)\in A)}c_{ij}y_{ij}$
- $c_{ij}=f+d_i$ if $\lbrace i=o, j\neq d \rbrace \ or\ \lbrace i\neq o, j=d \rbrace$
- $c_{ij}=d_{i,j}$ if $((i,j) \in A_2)$
- $c_{ij}=d_{i,j}-l_i$ if $((i,j)\in A_1)$

#### Assumption

- Travel time is deterministic and we don't need to refuel our AV between the trip

#### Constraint

1. $t_{j^-}-t_{i^+}\ge$ relocation_time $(i,j)$
2. $\sum_{j}y_{ij}\leq \sum_{j}m_{ij}$
3. $\sum_{j\in I\backslash \lbrace o \rbrace} y_{ij}=F$, $i=o$
4. $\sum_{j\in I\backslash \lbrace d \rbrace} y_{ji}=F$, $i=d$

- In constraint 1, we want to let our AV arrive before trip's pickup time given that the relocation time is deterministic. Therefore, our relocation time in $(i^+,j^-)$ must less than the delivery time of order j minus pickup time of order i
- In constraint 2, we constraint link capacity. For each link, we only allow one AV use it because we only need to serve each demand once.
- Constraint 3 and constraint 4 are balance constraint, we use these two constraints to make sure that number of AVs we collect at the end is same as number of AVs we dispatch at the beginning

After solving above model, we can optimally determined number of AV we should dispatch and the trip chain for each AV based on reserved order.

### Phase two greedy algorithm

- In phase two, as we deal with real-time orders, efficiency becomes crucial. Therefore, we employ a greedy algorithm to address the problem. While this approach may not yield an optimal trip chain, it allows us to quickly obtain a suboptimal trip chain.
- Our greedy algorithm is straightforward: we identify all feasible solutions based on the current trip chain and then insert the new order into the trip chain in a way that minimally increases the cost
- In the later empirical results, we will demonstrate the efficiency of our algorithm and illustrate that the optimality gap between our algorithm and the optimization is not substantial

## Example

In this section, we will present a simple example to facilitate a better understanding of our entire problem and methodology

In our example, we have five reserved orders, and the information is presented in the following figure. We then utilize the information from these five orders and execute the phase one model, resulting in the following optimal trip chains:
  - Car 1’s trip chain: start --- order 2 --- order 5 --- order 1 --- end
  - Car 2’s trip chain: start --- order 3 --- order 4 --- end
 
![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/example1.png)

 
Subsequently, in real-time, we receive order 6, and we identify two possible ways to insert this order:
  - Car 1’s trip chain: start --- order 2 --- order 6 --- order 5 --- order 1 --- end
  - Car 2’s trip chain: start --- order 3 --- order 6 --- order 4 --- end

Finanly, we will choose the trip chain that minimally increases the cost

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/example2.png)


## Numerical Study
In this section, we use some test data to show the effectiveness and efficiency of the mathematical program for phase 1 and our greedy algorithm for phase 2.

### The data set
We use the New York Taxi data set (https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) for this study, which is widely used in the field of car sharing/rental literature. In accordance with the data used in Ma et al. 2017, the testing date we selected is 2013/01/01. After removing anomalous records, there are about 382,862 records on that day, and we will randomly sample some data to test our mathematical model and algorithm.

Every row in the data set represents an order. The information of an order includes departure time, arrival time, traveling distance, departure location ID, arrival location ID, and the amount of revenue lost if the company fails to assign a car to satisfy the demand of that order. Note that the amount of revenue lost is assumed to be proportional to the traveling distance.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/data.png)

### Experiment for phase 1
For phase 1, where we use a mathematical model to find an optimal trip chain for those preserved orders, as the model always reports an optimal solution, we are interested in how much time it spends. For each number of preserved orders, we randomly generate ten instances and calculate the average execution time. The following figure shows that, not surprisingly, the execution time for the model increases when the number of preserved orders increases. Moreover, we may observe that the time spent seems to grow exponentially, this indicates that a heuristic algorithm may be required to accelerate the time to find an initial trip chain.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/phase1_time.png)

### Experiment for phase 2
For phase 2, where we face some real-time orders and every time the company receives a real-time order, we run our algorithm to determine which car is responsible for satisfying that order. We may see that though our algorithm cannot always generate an optimal solution, the algorithm can report a solution that is not too far from an optimal one, in the sense that the average optimality gap is less than 1%.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/phase2_table.png)

Furthermore, we want to emphasize that our algorithm is an efficient one, this is verified through the graph of time comparison. In the graph, the blue line represents the time required for our algorithm and the red line is the time for finding an optimal solution, which is much more efficient for our algorithm. Being an efficient algorithm is also important for a real-world car-sharing company. Typically, a customer goes to the website (or Apps) of the company to place some real-time orders. For such a company to succeed, a critical factor is to response to these requests as soon as possible (tell the customer the order is accepted or not). The is also one of the main features of our algorithm.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/phase2_time.png)

### Further sensitivity analysis
In addition to the basic model, we further conduct sensitivity analysis to test our model and algorithm. For the first part, we fix the number of preserved orders to 100 and increase the number of real-time orders gradually from 100 to 1000. From the results, we may observe that the optimality gap also increases slightly. This is reasonable because our algorithm does not guarantee reporting an optimal solution. When the portion of real-time orders increases, our algorithm will be called many times, leading to a solution that is somewhat less effective. However, a less-than-two-percent optimality gap is still acceptable in many applications.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/sensitivity_realTime.png)

For the second part, we fix the number of real-time orders and increase the number of preserved orders. For this case, the optimality gap decreases when the portion of preserved orders becomes larger. The reason behind this is that as we use a mathematical model to solve the phase 1 problem, which will generate an optimal solution. Therefore, when the number of preserved orders is larger, the smaller the optimality gap.

![image](https://github.com/jeffery881001/ORA-project-vehicle-sharing/blob/main/figures/sensitivity_preserved.png)

## Major contribution
In this project, the major contribution is the greedy algorithm in phase 2 and the insight drawn from the numerical study (note that phase 1 is an implementation practice of an existing model in the literature). Though the algorithm is simple, it can generate a plan for the company to respond to real-time orders as quickly as possible. Without the algorithm, the company either spends a lot of time finding an optimal solution, leading to the situation that the customer will wait for a long time, or generates a solution that is not too good. Furthermore, via numerical experiments and sensitivity analysis, we may see that even when there are thousands of orders, our method still provides a solution that is close enough to the optimal one. Practitioners can also determine whether our algorithm is an appropriate method based on the ratio of the number of preserved orders to that of real-time order.

## Limitation

- Our travel time is deterministic, but in real world, there might be congestion
- During the service or relocation, the AV may need to go to a refueling station for refueling

## Future work

- Consider the real-time order as stochastic, and leverage the real-time order location distribution or pickup time distribution to adjust our phase one optimization and obtain a more optimal trip chain.
- Use real traffic data to get more information about congestion or refueling to make the model more robust

## Conclusion
In our project, we consider a vehicle-sharing company facing reserved orders and real-time orders. For reserved orders, we build a mathematical model based on the network graph to solve the problem optimally. For real-time orders, we propose an efficient greedy algorithm to approach it. The effectiveness and efficiency are demonstrated through numerical experiments.

## Reference 
- Ma, J., Li, X., Zhou, F., Hao, W., 2017. Designing optimal autonomous vehicle sharing and reservation systems: A linear programming approach. Transp. Res. Part C:
Emerg. Technol. 84, 124–141.
