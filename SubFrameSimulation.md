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

![img](img/Nesting.png)


## Spawn / adding particles in the system

### Spawn
- Create n particles **Points**
- Labelize them part of simulation **Store Named Attribute**
### Set Position
- Place them regarding
  - Emitter *interpolated* position
  - Random position parameter (seed = subframe nb)
### Set Velocity
- Set (initial) velocity regarding
  - Emitter *interpolated* orientation (must mix with Rotation, not Vector)
  - Scale it by *interpolated* veocity parameter
  - Add randomness from *interpolated* velocity randomness parameter
- Delete particles if we ara at first frame (interpolation will fail because there is no previous frame to get correct values from)
- Put them into particle system **Join Geometry**

![img](img/Spawn.png)


## Set Velocity

- Calculate next position regarding
- Own velocity
- Field (we can add others)
- Gravity

## Collision Detection

Using a **Raycast** makes it really precise:
Every particle is casted on target mesh regarding its own position and velocity. It is asking "Will I hit mesh between now and next subframe?".  
They are then separated into two pipelines:
- Regular simulation
- No simulation but stick to the collider mesh (need constant update if collider moves)

![img](img/CollisionDetection.png)

## Set Position (regular sunulation)

For "Regular simulation", simply adding *velocity* $\times$ *delta time* to position.  
For sticky particles, see below.

## Set Position (stick to collider)

**1** Get the hit face $f$ index (triangle) of the collider  

**2** Describe the position $P$ of the particle only in terms of $f$ vertices $v_0$, $v_1$, $v_2$ and two factors $u$ and $v$.  
These are **local coordinates**.

$$
\\[1em]
\vec{u} = v_1 - v_0
\quad;\quad 
\vec{v} = v_2 - v_0
\\[1em]
P = v_0 + u \, \vec{u} + v \, \vec{v}
$$

**2,1** Get face index and vertices of this face (triangle)
![img](img/VerticesOfFace.png)

Note: we don't need a third coordinate because we assert particle is *on* the triangle, so there is no local height. If needed, complete formula would be
$$
\\[1em]
\vec{u} = v_1 - v_0
\quad;\quad 
\vec{v} = v_2 - v_0
\quad;\quad 
\vec{w} = \vec{u} \times \vec{v}
\\[1em]
P = v_0 + u \, \vec{u} + v \, \vec{v} + w \, \vec{w}
$$

**3** Calculating $u$ and $v$ (and $w$) requires two "helpers" $\vec{p}$ and $o$.  
**3,1** &nbsp;&nbsp; $\vec{p} = p - v_0$ where $p$ is the particle position  
**3,2** &nbsp;&nbsp; $o = \vec{u} \cdot ( \vec{v} \times \vec{w})$  
**3,3**
$$
u = \frac{\vec{p} \cdot ( \vec{v} \times \vec{w})}{o} \quad;\quad
v = \frac{\vec{p} \cdot ( \vec{w} \times \vec{u})}{o} \quad;\quad
w = \frac{\vec{p} \cdot ( \vec{u} \times \vec{v})}{o} \quad;\quad
$$

![img](img/SolveLocal.png)

**4** Store the needed variables in **Store Named Attribute** to get back the global coordinates from the local coordinates:
- face $f$ (index)
- factor $u$ (scalar)
- factor $v$ (scalar)
- (factor $w$ (scalar))  

The involved vertice positions will change over frames, so we need to get their coordinates at each frame

**5** Place the particle back on the mesh  
**5,1** Get $v_0$, $v_1$, $v_2$ updated coordinates  
**5,2** Calculate $\vec{u}$, $\vec{v}$, ($\vec{w}$)  
**5,3** Calculate global coordinate
$$
p = v_0 + u \, \vec{u} + v \, \vec{v} + w \, \vec{w}
$$
Set position to this coordinate with **Set Position**.

![img](img/GlobalPos.png)