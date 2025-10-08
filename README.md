to do:
- initialize the particles in the cloud
Process:
    
Don't Know:
    Visualization

- particle weighting with lasers
Process:
    
Don't Know:
    LaserScans from particles onto a know map

- modify particles based on movement
Process:
    Taking movement from the Neato and applying to the particles

- normalize particles
Process:
    Math to make the sum of weights equal to 1

- resampling of particles
Process:
    Resampling based on areas of high weight
    Some particles still fully random to deal with local minima

- update robot pose estimate
Process:
    find average position within an cluster of higher weight

- anything within the particle class