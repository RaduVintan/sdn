# SLF/RLF Solver

This is a solver I implemented as part of my Master's Thesis, which will be available soon in this repo. It solves the Strong Loop-Freedom (SLF) and Relaxed Loop-Freedom (RLF) Problems as introduced in [1]. The software allows the user to solve any of the two problems either optimally, or using one of the provided heuristics which are faster and provide decent approximations. Details regarding the used techniques and algorithms are available in the Thesis. I will often make reference to Sections or names/definitions introduced in the Thesis.

&nbsp;

### Structure
- the **sdn** folder contains the C++ implementation of the non-LP based heuristics and the main logic of the solver. For the LP-based algorithms it uses the optimizer.
- the **optimizer** folder contains the Java programs that construct and solve the LPs which our LP-based algorithms use, as well as a Python script that generates all cycles of a graph (required by one of the algorithms).
- the **Results** folder contains the data I sampled and used for the Thesis.

&nbsp;

### Dependencies and Requirements

This software has been only tested on Linux. (MacOS should work too in principle.)

For running the LP-based algorithms **Java** needs to be installed and the *java* command must be added onto the PATH system variable. Moreover, the **Gurobi** library [2] must be present (see section on installation).

For running the inefficient LP introduced in Subsection 3.3.1 of the Thesis **Python 3** is required as well as the library **networkx**. The software expects the command *python3* to launch Python 3. For most users this requirement should be entirely optional, as it is not required for the efficient LP solvers.

The **make** command is required, as well as a also a local installation of the **g++** compiler. (These are provided by default on most Linux distributions).

&nbsp;

### Installation

This first part of the installation is technically required only if you plan to run any of the LP-based algorithms. Note that these algorithms are the only ones which can compute the optimal solutions for the SLF and RLF Problems:

In the folder **optimizer** create a new folder **lib**, in which the file **gurobi.jar** from your local licensed installation of the **Gurobi** library should be copied. Note that Gurobi offers a free Academic License.

After copying the library, execute inside the **optimizer/src** folder:
```sh
javac -cp "../lib/gurobi.jar" *.java
```
to  compile all Java files. 

The second part of the installation is to compile the C++ files from the **sdn** folder. This step is mandatory. Go into the folder and simply run:
```sh
make
```
The provided Makefile should do everything for you.

&nbsp;

### Usage single instance

I begin with commands that run on one single instance. All these commands read the instance from the file **sdn.in**. An instance is to be stored like in the following example:
```sh
7
1 5 2 6 4 3 7 
```
where the permutation on the second row describes the new path (the old path is assumed to be the identity permutation, e.g. 1 2 3 4 5 6 7). It is not allowed to input permutations where *k* is followed by *k+1* for some k. (Note that in this case the node *k+1* is redundant.)


To use the Highest Lower Bound (HLB) Algorithm (Algorithm 1 in Thesis), run:
```sh
./sdn hlb
```
To use the HLB Heuristic (Algorithm 2 in Thesis), run:
```sh
./sdn hlbHeuristic
```
To run the Local Search routine (in Thesis: Algorithm 3 for SLF and Algorithm 6 for RLF), run:
```sh
./sdn localsearch slf/rlf
```
where **/** is used to point out the different options of this command (*slf* for the SLF variant and *rlf* for the RLF variant).

To apply the (inefficient) LP from Subsection 3.3.1 to solve an instance of the SLF Problem, you can use:
```sh
./sdn solveLPinefficient
```

To apply the improved (and recommended) LP Algorithms (in Thesis: Subsection 3.3.2 for SLF and Subsection 4.2.1 for RLF), use:
```sh
./sdn solveLP fractional/integral slf/rlf
```
where the options *fractional* and *integral* allow you to specify whether the computed optimum should be a fractional or an integral optimum, and the *slf* and *rlf* options, as always, specify whether you want to solve the SLF or RLF variant.

If only an approximation of the optimum is needed, you can use the rounding procedure:
```sh
./sdn roundLP slf/rlf
```

To apply the Peacock algorithm introduced in [3] for the RLF Problem (but discussed also in the Thesis), use:
```sh
./sdn peacock
```
To apply the Short Path algorithm for the RLF Problem (Algorithm 7 in Thesis), use:
```sh
./sdn shortpath
```

### Usage statistics

The following commands can be used to gather statistics for randomly generated permutations using any of our algorithms for the SLF and RLF Problems. An example:
```sh
./sdn statistics solveLP fractional/integral slf/rlf 60 200 (randomseed)
```
calculates either the fractional or integral OPT for either the SLF or RLF problems for *200* randomly generated permutations/instances with *60* nodes. The last parameter *randomseed* is optional and should be included only if a random seed is desired. If this argument is missing then our software will use a fixed seed. This allows to reproduce the results used for the Thesis. For example, the above command ran with the options *integral*, *rlf* and without *randomseed* allows to reproduce the data for OPT used to generate Figure 3.7a from the Thesis. If *fractional* is used it computes the data for LP_OPT.

The following commands are used to gather statistics with the other algorithms. They should now be self-explanatory:
```sh
./sdn statistics roundLP slf/rlf size tries (randomseed)
./sdn statistics hlb size tries (randomseed)
./sdn statistics localsearch rlf/slf size tries (randomseed)
./sdn statistics peacock size tries (randomseed)
./sdn statistics shortpath size tries (randomseed)
```
There are two special experiments in the Thesis which cannot be simulated using the above instructions. To run the Experiment used to generate the data shown in Figure 3.6 (comparison of approximation factors) for the SLF Problem, use:
```sh
./sdn statistics SLF_experiment
```
To run the Experiment used to generate the data shown in Figure 4.7 (comparison of heuristics for large number of nodes) for the RLF Problem, use:
```sh
./sdn statistics RLF_experiment
```

### Usage generating graphs
Some interesting types of graphs/instances can be generated in the **sdn.in** file automatically by using certain commands. 

To generate a ShortIsBad instance with *k* purple blocks and 4k+2 nodes (see Subsection 3.1 and Figure 3.1 in Thesis), use:
```sh
./sdn generate shortIsBad k
```
In Figure 8 in [3] a family of graphs is shown for which the greedy strategy of updating all forward nodes in the first round turns out to be a very bad strategy for the SLF Problem. (The ShortIsBad graphs from before also have this property.) To generate such a graph with k blocks (5k+4 nodes), use:
```sh
./sdn generate greedyGraph k
```
In Figure 2 in [3] a family of graphs is shown for which any SLF solution requires \Omega(n) rounds. To generate the corresponding graph with *k* nodes, use:
```sh
./sdn generate hardGraph k
```
In [3] a family of graphs G(k) is given with 2^(k+3) nodes each and the claim that the Peacock algorithm requires \Omega(k) rounds for solving these instances. As we show in the Thesis, this construction is not entirely correct. To generate the graph G(k) use:
```sh
./sdn generate wrongPeacockGraph k
```
In the Thesis we present a corrected construction, where G(k) has 2^k nodes and indeed forces Peacock to use \Omega(k) rounds. To generate the corrected G(k) use:
```sh
./sdn generate correctPeacockGraph k
```

&nbsp;
&nbsp;
&nbsp;

#### References
**[1]** S. Amiri, A. Ludwig, J. Marcinkowski, and S. Schmid. **Transiently Consistent SDN Updates: Being Greedy is Hard.** In: International Colloquium on Structural Information and Communication Complexity. July 2016, pp. 391–406

**[2]** Gurobi Optimization, LLC. Gurobi Optimizer Reference Manual. 2022. URL: https://www.gurobi.com.

**[3]** K.- T. Foerster, A. Ludwig, J. Marcinkowski, and S. Schmid. **Loop-Free Route Updates for Software-Defined Networks.** In: IEEE/ACM Transactions on Networking 26.1 (2018), pp. 328–341.

