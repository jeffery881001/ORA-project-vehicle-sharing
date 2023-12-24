# ORA-project-vehicle-sharing
In this project, we use the methods in Operations Research to optimize the relocation plan of a vehicle sharing company



# Numerical Study
In this section, we use some test data to show the effectiveness and efficiency of the mathematical program for phase 1 and our greedy algorithm for phase 2.

### The data set
We use the New York Taxi data set (https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) for this study, which is widely used in the field of car sharing/rental literature. In accordance with the data used in Ma et al. 2017, the testing date we selected is 2013/01/01. After removing anomalous records, there are about 382,862 records on that day, and we will randomly sample some data to test our mathematical model and algorithm.

Every row in the data set represents an order. The information of an order includes departure time, arrival time, traveling distance, departure location ID, arrival location ID, and the amount of revenue lost if the company fails to assign a car to satisfy the demand of that order. Note that the amount of revenue lost is assumed to be proportional to the traveling distance.

### Experiment for phase 1
For phase 1, where we use a mathematical model to find an optimal trip chain for those preserved orders, as the model always reports an optimal solution, we are interested in how much time it spends. For each number of preserved orders, we randomly generate ten instances and calculate the average execution time. The following figure shows that, not surprisingly, the execution time for the model increases when the number of preserved orders increases. Moreover, we may observe that the time spent seems to grow exponentially, this indicates that a heuristic algorithm may be required to accelerate the time to find an initial trip chain.

### Experiment for phase 2
For phase 2, where we face some real-time orders and every time the company receives a real-time order, we run our algorithm to determine which car is responsible for satisfying that order. We may see that though our algorithm cannot always generate an optimal solution, the algorithm can report a solution that is not too far from an optimal one, in the sense that the average optimality gap is less than 1%.

Furthermore, we want to emphasize that our algorithm is an efficient one, this is verified through the graph of time comparison. In the graph, the blue line represents the time required for our algorithm and the red line is the time for finding an optimal solution, which is much more efficient for our algorithm. Being an efficient algorithm is also important for a real-world car-sharing company. Typically, a customer goes to the website (or Apps) of the company to place some real-time orders. For such a company to succeed, a critical factor is to response to these requests as soon as possible (tell the customer the order is accepted or not). The is also one of the main features of our algorithm.

### Further sensitivity analysis
In addition to the basic model, we further conduct sensitivity analysis to test our model and algorithm. For the first part, we fix the number of preserved orders to 100 and increase the number of real-time orders gradually from 100 to 1000. From the results, we may observe that the optimality gap also increases slightly. This is reasonable because our algorithm does not guarantee reporting an optimal solution. When the portion of real-time orders increases, our algorithm will be called many times, leading to a solution that is somewhat less effective. However, a less-than-two-percent optimality gap is still acceptable in many applications.

For the second part, we fix the number of real-time orders and increase the number of preserved orders. For this case, the optimality gap decreases when the portion of preserved orders becomes larger. The reason behind this is that as we use a mathematical model to solve the phase 1 problem, which will generate an optimal solution. Therefore, when the number of preserved orders is larger, the smaller the optimality gap.

# Conclusion
In our project, we consider a vehicle-sharing company facing reserved orders and real-time orders. For reserved orders, we build a mathematical model based on the network graph to solve the problem optimally. For real-time orders, we propose an efficient greedy algorithm to approach it. The effectiveness and efficiency are demonstrated through numerical experiments.
