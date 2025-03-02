**University of Pennsylvania, CIS 5650: GPU Programming and Architecture,
Project 1 - Flocking**

* Kyrylo Smyrnov
  * [LinkedIn](https://linkedin.com/in/kissmyrnov/)
* Tested on:
  * OS: Windows 11 
  * CPU: Intel Core i5-14400F
  * GPU: NVIDIA GeForce RTX 3060 Ti
  * RAM: 32 GB

## Introduction
Boids is an artificial life program, developed by Craig Reynolds in 1986, which simulates the flocking behaviour of birds, and related group motion.

![CUDA_Flocking_Naive_3000](images/CUDA_Flocking_Naive_30000.gif)
Naive, 30 000 boids, 128 block size

## 3 rules of boid movement

1. <b>Cohesion</b> - boids move towards the perceived center of mass of their neighbors
2. <b>Separation</b> - boids avoid getting to close to their neighbors
3. <b>Alignment</b> - boids generally try to move with the same direction and speed as
their neighbors
</br>
</br>
![Rule_cohesion](images/Rule_cohesion.gif)
![Rule_separation](images/Rule_separation.gif)
![Rule_alignment](images/Rule_alignment.gif)

## 3 approaches to calculations

### 1. Naive

At each simulation step, each boid analyzes the position of <b>all</b> other boids in the system. Based on the data of other boids, each boid adjusts its speed and direction following three rules. After calculating the new speed and direction, the boid moves to a new position.

Since each boid must check all others, the complexity of the algorithm is $O(N^2)$. </br>
Also, unnecessary calculations become a problem when there are a large number of boids, since boids at a large distance have no effect on each other.

### 2. Uniform grid

Before starting the simulation, the whole domain is divided into a uniform grid, where the size of cells is chosen so that it is not less than the maximum interaction distance between neighboring boids.Several arrays in shared memory are used for fast data access.

This approach allows at each simulation step to check only boids in neighboring cells (4 for 2D, 8 for 3D) that can influence the boid behavior.

### 3. Coherent grid

Improved version of Uniform Grid. In this algorithm, boid data (positions, velocities) is reorganized so that all boids within the same grid cell are stored contiguously in memory.

This reduces the need for an additional index array, improving memory coherence and allowing for more efficient, low-latency access to boid attributes.

## Performance analysis

All FPS values are averaged over 50 simulation steps

### Performance change depending on the number of boids

![FPS_change_depending_on_the_number_of_boids_Visualization_ON](/images/FPS_change_depending_on_the_number_of_boids_Visualization_ON.png)

![FPS_change_depending_on_the_number_of_boids_Visualization_OFF](/images/FPS_change_depending_on_the_number_of_boids_Visualization_OFF.png)

The naive method contains a lot of redundant computations, its efficiency is much lower than for the other two methods, this leads to serious scalability problems when the number of boids increases.

The uniform grid and coherent grid algorithms show much better and similar results. The coherent grid reduces the memory access latency, which makes it more efficient.

### Performance change depending on the block size

![FPS_change_depending_on_the_block_size_Visualization_ON_30000_boids](/images/FPS_change_depending_on_the_block_size_Visualization_ON_30000_boids.png)

Performance increases until the block size is 64, after which performance varies in both ways. If the block size is too small, the total number of threads may not be enough to load all GPU cores, and if the size is too large, the performance will decrease due to lack of resources for registers and shared memory.