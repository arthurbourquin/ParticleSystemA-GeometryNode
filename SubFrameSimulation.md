# Particle System


![Full Node Tree](img/ParticleSystemA_FullTree.png)


## Elementary particle system to build on

### Main features
- **Sub-frame**
- Allow **Field(s)** (and gravity)
- **Euler Explicite** type
- Animated mesh collider stick
- Basic emission parameters
- **Emitter** Object driven 
- Particle deletion
- Scene framerate independant

### Overall principles of this tree

- Units in **meter** and **second**
- Particle amount $=$ particle count $\times$ subframe count ; so **adding subframes adds particles**
- **Subframe** count and **framerate** **don't change overall behaviour** ; increasing subframe doesn't make particles faster or slower, or following different paths
- **Keyframe iterpolations are linear**

<hr>


# Component Descriptions


## Nested Repeat Zone

### Main problematics:
- geting / counting the **current subframe** (~easy)
- **interpolate** keyframed parameter values (~hard)  

### Main variables:
- framerate
- current subframe
- subframe duration (delta time dt)
- previous values for interpolation coulutation


## Spawn / adding particles in the system

- 