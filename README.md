# Solving the Traveling Salesman Problem with Profits Using Genetic Algorithm

In this project, we are going to solve the traveling salesperson problem with profits (TSPP) 
where the salesperson collects some profit for visiting each customer. As opposed to the classical TSP, 
there is no requirement to visit all the customers. The objective of TSPP is to determine the best subset of customers 
to be visited so as to maximize the total net profit, which is equal to the total profit earned from visited customers 
less the total cost of the tour. The latter can be taken as the total length of the tour calculated as the Euclidean distance.


There are three data sets (eil51,eil76, eil101) given in the Excel file called “dataset- TSPP.xls”. 
It contains three data sets with low and high customer profits as well as the coordinates of the customer locations. 
The first customer indexed by “0” is the depot location, i.e., the salesperson starts its tour from this location. 
Therefore, the number of customers is equal to 50, 75, and 100 for eil51,eil76, and eil101 respectively. 
Hence, the input data consists of the customer locations and the profits associated with each customer. 

### Solution Representation:
For solution representation, we keep visit order of customers in the tour.
It always contains depot, so all solutions in the populations are feasible.
To check feasibility, we have a function called “verify_soln” that controls if there are repetitions in the tour and the existence of depot in the tour. 
We use genetic algorithm (GA) as our heuristics method and the following sections explain the details of each stage in the GA.

### Initial population:
The population size is a hyperparameter to be adjusted.It is chosen as 50 in our algorithm. 
The initial population consists of two types of solutions. Some are from a construction heuristic algorithm and the others are partially random solutions.
Construction heuristic takes tour length (k) as an input. It calculate a score for each customer based on their profit and their average distance to other
customers. The score of a customer i, 𝑠𝑖 , is:

𝑠𝑖 = 𝑝𝑖 − 1 ∑ 𝑑(𝑖, 𝑗)

 where 𝑝𝑖 is profit of custemer i, 𝑑(𝑖, 𝑗) is distance between customer i and j. 
 Customers are sorted in descending order and the first k customer are selected to construct a tour.
 (k is the tour length without depot). The depot is included in the tour after customers are determined. 
 To construct the tour, we use a greedy approach that is we start from a random customer, and we always visit the nearest customer from unvisited ones. 
 Thus, some solutions are created by this construction heuristics with different tour lengths. 
 The tour lenghts are multiples of 5 and goes until the number of customers. 
 For example, for eil-51, we have tours of length 5,10,15 ..... 50 without depot. 
 The number of solutions created by construction heuristic is determined by “p_const_heur” parameter i.e. p_const_heur * n , 
 where n is the number of solutions in a population. So, we divide this number to number of different tour lengths and generate 
 that many solutions for each tour length. The remaining solutions are generated more randomly. 
 We randomly select a tour length and choose that many customers randomly to construct a tour. 
 The tour visit order is determined by the same greedy procedure in construction heuristic i.e the nearest customer is selected each time. 
 The order is not selected totally randomly because it gives poor objective values and starting from those solutions would require much more effort 
 to obtain a relatively reasonable tour.
 
 After obtaining the initial population, we update all solutions in this population to avoid redundancy in the tour by eliminating customers that affect the tour negatively. 
 The “update_soln” function in code remove a customer by considering its length to two neighboring customers in the tour, it's
profit and the length between its two neighboring customers in the tour.
For each customer in the tour, we calculate the gain obtained by removing this customer, 𝑔𝑖 , using the following formula: 

𝑔𝑖 = 𝑑(𝑖, 𝑖𝑝𝑟𝑒𝑣) + 𝑑(𝑖, 𝑖𝑛𝑒𝑥𝑡) − 𝑑(𝑖𝑝𝑟𝑒𝑣, 𝑖𝑛𝑒𝑥𝑡) − 𝑖𝑝𝑟𝑜𝑓𝑖𝑡   

where 𝑑(𝑖, 𝑖𝑝𝑟𝑒𝑣 ) is the distance between customer i and the previous customer in the tour,
𝑑(𝑖, 𝑖𝑛𝑒𝑥𝑡) is the distance between customer i and the next customer, 𝑑(𝑖𝑝𝑟𝑒𝑣, 𝑖𝑛𝑒𝑥𝑡) is the distance between previous customer and 
the next customer and 𝑖𝑝𝑟𝑜𝑓𝑖𝑡 is the profit of this customer. If 𝑔𝑖 is positive, then we remove this customer from tour and look for the other customers.
We proceed until no such customer is found. Actually, this solution update is also applied for the offsprings in each generation.

### Selection :
Tournament selection procedure is applied to select parents. Each time a pool of solutions with size “t” are selected randomly and the best one is put into parent list. The tournament size and the parent list size are hyperparameters to be chosen. We use 3 for tournament size and 10 for parent list size.
Reproduction:
Reproduction consists of cross over and mutation operations. We didnt use a mutation operator for this problem. Since the solution representation is a permutation, we cannot directly apply a common cross over operators like two point crossover, one point crossover as they may generate infeasible solutions. Rather, we need specially designed crossover operators. In the literature there are different suggestions for a TSP problem. Yet, in our problem the parent solutions may have different tour lengths and thus different customers, so we need some modifications in the proposed crossover operators. We chose order crossover (OX) as a baseline and modified slightly. For the first offspring, we choose two points randomly in the first parent’s solution representation and take the customers in between directly (by keeping their order). We call this as “middle section” of the offspring. Then, we take the customers in other parent which are not in the “middle section”. Half of these customers are added to the beginning of the offspring and the other are appended to the end by keeping their relative order. The other offspring is obtained in the same way starting from the second parent. 
An example is shown below:

parent_1: 5,7,2,8,3,0,9,6. <br>
parent_2: 9,1,3,4,0,7,2,8,5,6 <br>
offspring_1: 9,1,4,2,8,3,0,7,5,6 <br>
offspring_2: 5,8,3,4,0,7,2, 9,6 <br>

The parent pairs are taken from parent list in their order in the list, and two offspring is generated for each pair. So, we generate 10 offsprings for each generation.

### Replacement:
For the replacement procedure, we tried different approaches and chose the best performing one. In the first stage, the offsprings are added to the current population. Afterwards, we eliminate the duplicate solutions. The probability of having two different solutions with the same objective value is too small, so we eliminate solutions by comparing their objective values to obtain efficient computation. If there are less solution than population size, we generate partial random solutions as described in the initial solution stage to fill the gap. Then we choose best solutions with “p_elit” ratio from all solutions. The others are created as extended versions of these elite solutions. That is, to generate a solution we take an elite solution and add some customers outside the tour. The customers are determined by their distance to the customers in the tour. That is, their distance to the closest customer in the tour are calculated and the top 5 cities are chosen to be added to the tour. In the tour visit order, they are added before the customer to which they are closest.
We have also another function called “clear_copy_solns” which removes the copies of the best solution in the population to increase divergence. This function applies this exclusion if more than 20% of the solutions are copies of the best solution and it is called once after each 50 iterations.
 

### Stopping condition:
To stop the algorithm, we keep number of consecutive iterations without improvement in the incumbent solution. We have a parameter called “n_non_improv_iter” and if it exceeds this value, we stop the algorithm. It is set as 400 in our algorithm. We have also an absolute maximum number of iterations to prevent too long running times, but it never hit this limit in our examples. 
 
### Parameters:


|Parameter | Description | Value | 
|:------:|:---:|:----:|
n | Population size |50 
p_const_heur | Ratio of initial pop. Obtained with construction heuristic solns | 0.6
p | Selection procedure parent size | 10
t | Tournament size | 3
p_elit | Ratio of elite individuals in replacement Max number of iterations | 0.7
n_iter | Max number of iterations without improvement | 5000
n_non_improv_iter | Max number of iterations without improvement | 400
problem | Problem instance | eil51, eil76 or eil101
profit | Problem instance profit case | High or Low

We run each instance 5 times by setting seed values from 0 to 4 and took the best solution obtained.The results are shown in the table below.

|Instance |	Best objective value |	No of customers visited	|	CPU Time |
|:------:|:---:|:----:|:----:|
eil51-LP |	50.74 | 	22	|	88.13 
eil51-HP |	158.01 	| 46 |		125.69 
eil76-LP |	44.14 	| 41 |	106.64 
eil76-HP |	384.5 	| 73 |		166.98 
eil101-LP |	239.45 |	68 |		164.00 
eil101-HP |	1577.78 |	93 |		97.67 



