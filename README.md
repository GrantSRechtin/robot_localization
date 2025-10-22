# Robot Localization Project

## Project Overview
The goal of this project is to use a particle filter system to determine the location of our Neato in the context of the world map. To do this, we used a combination of information from the Neato's laser scanner as well as the distance of our established particles and compared the distances to the obstacles around them to estimate the Neato's location in the map.

## Conceptual Breakdown

### Architecture Overview
We started by estabishing the topics our particles would subscribe and publish to, and indicated the role of each topic in the context of the particle filter.

![alt text](original-FB01AECD-6203-4AB9-A840-AFD7AE8F66CF.jpeg)

<p align="center"><strong>Figure 1.</strong> Particle Filter Subscribers and Publishers.</p>

<span style="color: red;">Edit image to Include pose estimate topic and type: Pose. Topic: </span> 

#### Initializing Particles
We initialized 1000 particles in the world view. Through testing values ranging from 300, the starting value, to 2000, we found that generally more particles works better up to a certain point when the process takes too long to keep up with the neato. 1000 ended up as a good value to maximize both speed and accuracy.  This cloud of 1000 particles was randomly scattered across our map using a uniform distribution to randomly scatter the x,y, and theta of the particles. As we don’t have an initial guess for the location of the neato a uniform distribution is best to cover as many possibilities as possible.

#### Updating Particles with ODOM
Following the initialization of the particles, once the Neato has moved a set distance from its initial or previous position, repeat the same movement for each particle within their own respective frame. To do this, using the stored changes in x,y, and theta for our Neato, we calculate each particle's new x,y, and theta. The following figure shows the particle's transformation relative to its initial placement:

$$
\Delta x = \Delta x_n * cos(\theta_p)
$$
$$
\Delta y = \Delta y_n * sin(\theta_p)
$$
$$
\Delta \theta = \theta_p + \theta_n
$$


<p align="center"><strong>Figure 2.</strong> Particle update equations.</p>
<p align="center"><em>Subscripts denote: n = neato, p = particle.</em></p>


The biggest difficult with this process is thinking about the frames in which we are dealing with, as the movement given is using the changes in the Neato's odom frame, the movement done by the particles has to be performed in each particle's respective frame, but the new positions of each particle are within the world frame.

#### Assigning Particle Weights
For every particle in particle_cloud, we initialize it with a weight of 1, updating it based on how the particle aligns with the LIDAR data. We collected a small sample of the laser data with num_rays to reduce the computational load. Next, we extracted a small range from the laser data, making sure to skip any invalid r_i readings such as NaN or infinite. We then calculated the probability that a particle is expected to be in the map frame using the logic below:

$$
x_{hit} = p.x + r_i * cos(p.\theta + \theta _i)
$$
$$
y_{hit} = p.y + r_i * sin(p.\theta + \theta _i)
$$
<p align="center"><em>Subscripts denote: p.x,p.y,and p.theta are the particles position and orientation. r_i and theta.i correspond to the lidar data</em></p>

Next, we found the expected distance to the nearest obstacle using the helper function self.occupancy_field.get_closest_obstacle_distance(x_hit,y_hit). We used a Guassian Probability distribution to determine the difference between the expected distance and r_i.

$$
probability = exp(- \frac{expected\ dist^2}{2* uncertainty^2})
$$

We set the uncertainty of the Guassian Probability 0.2 after experimenting with values that worked best. We multiplied the weight of each particle by the probability to see if the expected distance aligns closely with the reading from the sensor data. Particles that align more closely with the lidar data are assigned a higher weight.


#### Normalizing and Resampling Particles
Normalizing particles is necessary for functions such as resampling particles, where the weights will be interpreted as probabilities. We normalized the particles by ensuring that the sum of all the weights is equal to 1. We simply calculated the total sum of particle weights in particle cloud, and divided each particle weight by the total sum.


$$
p_w = \frac{p_w}{sum\ of\ particle\ weights} 
$$

We collected the particle weights from each particle and used numpy.random to randomly choose from the list of particle weights. 300 new particles are chosen, and the probability of sampling each particle is proportional to its weight. This means that particles with a higher weight are likely to be chosen, but we will still include a small portion of particles sampled away from the highest concentrated areas to prevent poor convergence and particle death. The chosen particles are then varied by 0.125 in each direction. This was arbitrarily chosen to add more variance. Our filter then uses the new regions with the highest weights to estimate the Neato's new pose.



## How to interact with our code
<span style="color: red;">??? idk bro.:: from the website, there are some things to run (running the bags, the map, and the code, rviz too. Mention before this that you need to download this repo, and the neatopackages repo found on comprobo25.github.io</span> 

## Team Member Contributions

### Grant
* Assigned the latest robot pose
* Modified particles using delta
* Normalized Particles
* Edited particle functions for different cases
* What we learned and challenges Writeup

### Tchenzie
* Created Particle_cloud
* Update Particles with Laser
* Conceptual Breakdown Writeup


## What We Learned and Challenges
### Challenges we faced along the way:
I think the biggest challenges to us overall were much more general, stemming partially from having the large set of starter code we did. It felt for a long while like we didn’t fully have an understanding of what we were actually doing. This got better as we looked into the starter code and talked with CAs, but the process from learning about the project to actually coding it took much longer than the previous project. Once we actually had an understanding of the code generally, the process wasn’t the most complex and expansive project since most of the portions we wrote were rather small, however we ran into more issues actually running it. Because of how so much of the code was already there it was hard to understand where a lot of our errors were. While we ended up getting it working (assuming we do), it took much more CA and teacher assistance than the first project.

### How we would improve with more time:
I think the biggest aspect we could work on improving is finding the robot pose from the particles. Since we felt we didn’t fully understand the whole process we opted for just averaging the positions of the most heavily weighted particles for our robot position. This doesn’t account for multiple clusters so it’s suboptimal. Having a solution for multiple clusters would be the best next option for a tangible improvement for the project.
Did you learn any interesting lessons for future robotic programming projects? These could relate to working on robotics projects in teams, working on more open-ended (and longer term) problems, or any other relevant topic.

### Valuable lesson
One valuable lesson that having the sample code did show us was the importance of separating projects into different parts and thinking about them one at a time. It helped us, once we started coding, to go through them in the order they were called and coding in a similar manner in the future could be extremely beneficial.
