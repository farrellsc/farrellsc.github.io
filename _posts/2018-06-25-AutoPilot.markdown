---
layout:     post
title:      "AutoPilot"
subtitle:   "Udacity Auto Pilot Course"
date:       2018-06-25 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - AI
    - Auto Pilot
---

> Old notes; mathjax to be fixed

## Auto Pilot 

### 1. Histogram Filter

#### 1.1 Localization Overview
GPS is inaccurate, resulting in 2-10 meters of error. Need to lower the error bound down to 2-10 cm.

#### 1.2 Initiatives of algorithm
Maintain a probability distribution of localization $X$, take prob-distribution at cell i as $X_i$.
Update it after sensing and moving.
Moving would blur the prob-distribution, but absorb more information.
Sensing would provide a new measurement $Z$ and  perform a re-locate, make the prob-distribution more accurate.
$P(X_i|Z) = \frac{P(Z|X_i)\times P(X_i)}{P(Z)}$
After each sensing-moving step, normalize prob-distribution, sum up to 1. Below $\alpha$ is the normalization factor
$\alpha = \Sigma_i P(Z|X_i)\times P(X_i)$
So currently the location of the vehicle is at the max-likelihood in probability distribution $X$
$location=max(X/\alpha)$

#### 1.3 Sensing
Keywords: Multiplication, Bayes, Accuracy
Map local map to surrounding features, and adjust current distribution by the observation.

#### 1.4 Motion
Keywords: Convolution, Total Probability, Prediction
Predict the probability of the vehicle falling in one grid, which is the summation of probabilities from all neighboring grids multiplied by a transition rate.

#### 1.5 Input: 
- Movement Accuracy as transition probabilities
- Sensing Accuracy as transition probabilities
- World Map
- Initial probability of localization

#### 1.6 Output
Current location probability distribution at World Map

#### 1.7 Algorithm
```python
#Modify the previous code so that the robot senses red twice.

p=[0.2, 0.2, 0.2, 0.2, 0.2]
world=['green', 'red', 'red', 'green', 'green']
measurements = ['red', 'red']
motions = [1,1]
pHit = 0.6
pMiss = 0.2
pExact = 0.8
pOvershoot = 0.1
pUndershoot = 0.1

def sense(p, Z):
    q=[]
    for i in range(len(p)):
        hit = (Z == world[i])
        q.append(p[i] * (hit * pHit + (1-hit) * pMiss))
    s = sum(q)
    for i in range(len(q)):
        q[i] = q[i] / s
    return q

def move(p, U):
    q = []
    for i in range(len(p)):
        s = pExact * p[(i-U) % len(p)]
        s = s + pOvershoot * p[(i-U-1) % len(p)]
        s = s + pUndershoot * p[(i-U+1) % len(p)]
        q.append(s)
    return q

for k in range(len(measurements)):
    p = sense(p, measurements[k])
    p = move(p, motions[k])
    
print p         

```

#### Q&A
- What is world map exactly?
A very detailed global map, used to map the local map provided by sensors to perform localization
- Can localization be performed in different conditions such as weather changes and light changes?
Can handle some conditions pretty well such as rain. But others fail, such as snow.
- How to choose parameters?
Heavily depend on sensors. In real scenarios matching functions are used for local map and global map matching, instead simply using pHit and pMiss.



### 2. Kalman Filter

#### 2.1 Overview
Kalman Filter is used to tracking vehicles, observing the positions and velocities of vehicles to predict their future locations.
It's based on radar and laser ranged data.

Kalman filter tries to solve a set of $(\mu, \sigma^2)$ for a Gaussian Distribution representing the probability. The analysis system prefers a Gaussian Distribution with a large $\sigma$, which makes the peak more robust, indicating the model is more certain of the whereabouts of a vehicle.

The reason for using a continuous Gaussian Distribution instead of a discrete histogram used in localization is that, the speed of a vehicle is continuous while the feature of a landscape changes discretely.

#### 2.2 Initiatives of algorithm
Maintain a 2D Gaussian distribution on velocity and location. 
Possible actions are sensing(observing location) and motion. Sensing could only update information along location axis and motion could only update information along motion axis.
When sensing, accuracy of the Gaussian distribution is of utmost importance, so we apply a multiplication on old distribution and the new observed location distribution.
When moving, we inevitably lose information, so we merely add up old distribution with the transition distribution.

#### 2.3 Sensing
Keywords: Multiplication, Bayes, Accuracy
Multiply current Gaussian Distribution with a World Map Gaussian Distribution, to update current World View.

#### 2.4 Motion
Keywords: Convolution, Total Probability, Prediction
Merely add current distribution with a motion distribution to predict next distribution state after motion.

#### 2.5 Input
Distance range data from laser and radar sensors.

#### 2.6 Output
The uncertainty level and predictions of the locations of vehicles nearby.

#### 2.7 Algorithm
```python
measurements = [1, 2, 3]

x = matrix([[0.], [0.]]) # initial state (location and velocity)
P = matrix([[1000., 0.], [0., 1000.]]) # initial uncertainty covariance matrix
u = matrix([[0.], [0.]]) # external motion, commonly not used, but provide a possible way to add manual acceleration 
F = matrix([[1., 1.], [0, 1.]]) # next state function
H = matrix([[1., 0.]]) # measurement function
R = matrix([[1.]]) # measurement uncertainty
I = matrix([[1., 0.], [0., 1.]]) # identity matrix

def kalman_filter(x, P):
	# x for estimate
	# P for uncertainty covariance
	# F for state transition matrix
	# u for motion vector
	# z for measurement
	# H for measurement function
	# R for measurement noise
	# I for identity matrix
	
    for n in range(len(measurements)):    
        # measurement update
        y = matrix([[measurements[n]]]) - H * x
        S = H * P * H.transpose() + R
        K = P * H.transpose() * S.inverse()
        x = x + K*y
        P = (I - K * H) * P
        
        # prediction
        x = F * x + u
        P = F * P * F.transpose()
        
    return x,P
```

#### 2.8 The Maths
**Here I mainly use a 2D kalman filter, but in real scenarios often 4D kalman filters are applied. (2D space + 2D velocities accordingly)**

**State Transition Matrix F**:
F is a covariance matrix, the larger $F_{ij}$ is, the closer these two elements are correlated
2D e.g.$$ 
\begin{pmatrix}
 newLoc \\
 newVelo
 \end{pmatrix} 
 = 
 \begin{pmatrix}
  1 & 1 \\
  0 & 1
 \end{pmatrix} _F
 \begin{pmatrix}
 oldLoc \\
 oldVelo
 \end{pmatrix} 
 $$
4D e.g.$$
 \begin{pmatrix}
  NewLocX \\
  NewLocY \\
  NewVeloX \\
  NewVeloY
 \end{pmatrix}
 = 
 \begin{pmatrix}
  1 & 0 & 1 & 0 \\
  0 & 1 & 0 & 1 \\
  0 & 0 & 1 & 0 \\
  0 & 0 & 0 & 1
 \end{pmatrix} _F
 \begin{pmatrix}
  LocX \\
  LocY \\
  VeloX \\
  VeloY
 \end{pmatrix}
 $$


**Measurement Function H**:
2D e.g.$$
measurement = 
\begin{pmatrix}
 1 & 0
\end{pmatrix}_H
 \begin{pmatrix}
 oldLoc \\
 oldVelo
 \end{pmatrix} 
$$
4D e.g.$$
measurement
 = 
 \begin{pmatrix}
  1 & 0 & 0 & 0 \\
  0 & 1 & 0 & 0
 \end{pmatrix} _H
 \begin{pmatrix}
  LocX \\
  LocY \\
  VeloX \\
  VeloY
 \end{pmatrix}
 $$

**Measurement update**:
**this whole procedure resembles the multiplication of two Gaussian distributions.**

$y = z- H * x$
error = true_measurement - self-maintained_location

$S = H * P * H^T+ R$
S = location_uncertainty + sensor_noise
S can be understood as comprehensive uncertainty as a single number.

$K = P * H^T * S^{-1}$
K is Kalman gain
K can be understood as location-related-covariances ($P*H$) multiply a certainty rate ($S^{-1}$)
So K is the gained knowledge on location after an iteration

$x = x + K*y$
K is like a learning ratio while y is the error. $K * y$ means the errors accepted in an iteration.
update self-maintained location, since K and y are only about location

$P = (I - K * H) * P$
$K*H$ just expand $K$ to the same size as $P$. 
$I - K*H$ means uncertainty rate after adjust by $K$ (standard - certainty gain = uncertainty)
Notably $P$ is only updated on location dimension since $K$ is only related to location.
update uncertainty 

**Prediction**:
$x = F * x + u$
new_self-maintained_comb = state_transition_matrix * original_comb + external_motion
Just as $\mu_1 + \mu_2$

$P = F * P * F^T$
rough estimate of new_uncertainty
Just as $\sigma_1^2 + \sigma_2^2$

### 3. Particle Filter

#### 3.1 Initiatives of algorithm
An algorithm for localization. Perform localization by spread uniform-distributed particles, then perform rounds of resampling to filter out the particles with correct positions. The comparison is done by comparing distance to several landmarks the sensors detected. The model is a continuous multimodel but with low efficiency.

#### 3.2 Importance Weight
$\alpha$ = actual measurement
$\beta$ = particle measurement
$l_i$ = coordinates of the $i_{th}$ landmark
$p_i$ = coordinates of the $i_{th}$ particle 
$iw_j$ = Importance weight of the $j_{th}$ particle
$z$ = coordinates of the robot

$\alpha=[D(z, l_i)]$
$\beta_j=[D(p_j, l_i)]$ 
$iw_j=\prod_i{GaussProb(\alpha_i, \beta_{ji})}$

#### 3.3 Resampling Wheel
**Resampling Wheel**: Time cost log(N)
Set the weight of all particles as a pie. Use a pointer to move around the pie, each time jumping $Uniform(0, w_{max})$. Wherever it lands, pick the share once. Each particle could be picked zero to multiple times.
```python
p3 = []
wmax = max(w)
windex = 0
beta = 0
for i in range(N):
    beta += random.uniform(0, wmax)
    while w[windex] < beta:
        beta -= w[windex]
        windex = (windex + 1) % N
    p3.append(p[windex])
```

#### 3.4 Input
- World Map
  - the algorithm assume that the ground map is static, but it use prior and posterior scans to determine mobile o
- Landmark coordinates

#### 3.5 Output
- a particle cloud indicating the approximate position of the vehicle

#### 3.6 Optimizing
trade off between particle numbers and accuracy:
Look at non-normalized importance weights. If they are large, then you are doing a good job tracking, maybe using too many particles.

#### 3.7 Expansion Reading - Google Auto Drive 2012

**Robot Model**
See a vehicle as four wheels, two steerable and two non-steerable.
Now I'm only viewing car as a (x, y, orientation) tuple.

**Sensor Data**: 
Match local view to world map, to perform localization.
Also include other sensors such as GPS, Inertial sensors.

#### 3.8 Filters Comparison
**Particle Filter**:
- easiest to implement
- time cost scales exponentially
- can hardly self correct
- scenario: multimodal distribution with small search space

**Histogram Filter**:
- limited accuracy
- similar application as Particle Filter
- more systematic, can regain correctness after losing it

**Kalman Filter**:
- unimodel, can't describe complicated situation
- time cost scales quadratically 
- continuous space with unimodal distribution

**mixed rough Kalman FIlter**:
- upgrade version to Kalman Filter

**to combine multiple filters**:
- not recommended, would cause inconsistency
- Rao-Blackwellized FIlter:
  - given high dimensional data
  - apply particle filter to a certain dimension
  - it so happens that other dimensions would shrink to unimodal or Gaussian distribution
  - apply Kalman filter to other dimensions

### 4. Search

Algorithms to find the shortest path when given a road map.
Currently the car relies on google map for a longterm route. Search algorithms are used when they are off route or dealing with trivial problems on road.

#### 4.1 Input
- Real World map, including obstacles
- Noticeably the World map would change, due to sensor updates, as the car moves, and routes are constantly updated based on fresh info.

#### 4.2 Output
- route plan/plans

#### 4.3 A*
**estimated total length**: $f$
**heuristic function**: h(x,y)
**length of path from start**: g
$f=g+h(x,y)$
Intuitively $h(x,y)$ is the direct length from current position to destination. So intuitively shortest total length is length already passed + direct length to dest.

Maintain a grid matrix and a stack. Each step pick a tuple with minimum f in stack and use it to update neighbors in grid matrix into the stack, until the destination is loaded into stack. All grids that have been loaded are usable paths.

```python
grid = [[0, 1, 0, 0, 0, 0],
        [0, 1, 0, 0, 0, 0],
        [0, 1, 0, 0, 0, 0],
        [0, 1, 0, 0, 0, 0],
        [0, 0, 0, 0, 1, 0]]
heuristic = [[9, 8, 7, 6, 5, 4],
             [8, 7, 6, 5, 4, 3],
             [7, 6, 5, 4, 3, 2],
             [6, 5, 4, 3, 2, 1],
             [5, 4, 3, 2, 1, 0]]

init = [0, 0]
goal = [len(grid)-1, len(grid[0])-1]
cost = 1

delta = [[-1, 0 ], # go up
         [ 0, -1], # go left
         [ 1, 0 ], # go down
         [ 0, 1 ]] # go right

delta_name = ['^', '<', 'v', '>']

def search(grid,init,goal,cost,heuristic):
    # ----------------------------------------
    # modify the code below
    # ----------------------------------------
    closed = [[0 for col in range(len(grid[0]))] for row in range(len(grid))]
    closed[init[0]][init[1]] = 1

    expand = [[-1 for col in range(len(grid[0]))] for row in range(len(grid))]
    action = [[-1 for col in range(len(grid[0]))] for row in range(len(grid))]

    x = init[0]
    y = init[1]
    g = 0
    h = heuristic[x][y]
    f = g + h

    open = [[f, g, h, x, y]]

    found = False  # flag that is set when search is complete
    resign = False # flag set if we can't find expand
    count = 0
    
    while not found and not resign:
        if len(open) == 0:
            resign = True
            return "Fail"
        else:
            open.sort()
            open.reverse()
            next = open.pop()
            x = next[3]
            y = next[4]
            g = next[1]
            expand[x][y] = count
            count += 1
            
            if x == goal[0] and y == goal[1]:
                found = True
            else:
                for i in range(len(delta)):
                    x2 = x + delta[i][0]
                    y2 = y + delta[i][1]
                    if x2 >= 0 and x2 < len(grid) and y2 >=0 and y2 < len(grid[0]):
                        if closed[x2][y2] == 0 and grid[x2][y2] == 0:
                            h2 = heuristic[x2][y2]
                            g2 = g + cost
                            f2 = g2 + h2
                            open.append([f2, g2, h2, x2, y2])
                            closed[x2][y2] = 1

    return expand
```

#### 4.4 Dynamic Programming
Dynamically calculate the length from destination to all grids in the map.

Maintain a value matrix, recording cost from dest to current grid.
 
Could add stochastic functions, indicating that motion could be wrong each step sideways.

Iterate the value matrix until no value changes any more. Pick the direction with the lost cost $0.25*(sideway1 + sideway2) + 0.5*front$ for each grid.

### 5. PID Control
After finding the route, first smoothen the route, then use PID control to make the vehicle follow the route without oscillation and overshoot.


#### 5.1 Smoothing
To perform smoothing is to make the smoothed path similar to the original path while becoming more continuous than the original one.

$x_i$ for the $i_{th}$ point in the original path
$y_i$ for the $i_{th}$ point in the smoothed path
Minimize the equation:
$\alpha * (x_i - y_i)^2 + \beta * (y_{i+1} - y_i)^2 \rightarrow min$

#### 5.2 PID Control

**P(Proportinal) Contoller**:
![](/img/in-post/2018-06-25-AutoPilot/1520052335876.png)
**steering = -tau * crosstrack_error; tau is a ratio constant** as
$\alpha = -\tau_p * CTE$
P Controller means to select an steering angle by ratio to CTE. The problem is that it never converge, finally retaining a state of "Marginally Stable"(overshooting).
![](/img/in-post/2018-06-25-AutoPilot/1520052745615.png)

```python
for i in range(iterations):
	CTE = myrobot.y
	steer = -tau * CTE
	myrobot = myrobot.move(steer, speed)
	print myrobot, steer
```

**PD(Proportinal Differential) Contoller**:
$\alpha = -\tau_p * CTE - \tau_p * \frac{d}{dt}CTE$
$\frac{d}{dt}CTE = \frac{CTE_t - CTE_{t-1}}{\delta t}$
use differential to counter oscillations

```python
for i in range(n):
    if len(y_trajectory) == 0: steering = -tau_p * robot.y
     elif len(y_trajectory) == 1: steering = -tau_p * robot.y - tau_d * (y_trajectory[-1] - 1)
     else: steering = -tau_p * y_trajectory[-1] - tau_d * (y_trajectory[-1] - y_trajectory[-2])
     robot.move(steering, speed)
     print robot, steering
```

**PID(Proportinal Differential Integral) Contoller**:
PD can't solve the system steering bias (steering drift).

$\alpha = -\tau_p * CTE - \tau_p * \frac{d}{dt}CTE - \tau_i  * \Sigma CTE$
$\frac{d}{dt}CTE = \frac{CTE_t - CTE_{t-1}}{\delta t}$
use Sum(CTE) to balance system steering bias

```python
inte_CTE = 1
for i in range(n):
     if len(y_trajectory) == 0: steering = -tau_p * robot.y - tau_i * inte_CTE
     elif len(y_trajectory) == 1: steering = -tau_p * robot.y - tau_d * (y_trajectory[-1] - 1) - tau_i * inte_CTE
     else: steering = -tau_p * y_trajectory[-1] - tau_d * (y_trajectory[-1] - y_trajectory[-2]) - tau_i * inte_CTE
     robot.move(steering, speed)
     x_trajectory.append(robot.x)
     y_trajectory.append(robot.y)
     inte_CTE += robot.y
```

#### 5.3 Parameter Optimization
Very basic optimization technique, optimize parameters in each dimension recursively.

```python
def twiddle(tol=0.2): 
    # TODO: Add code here
    # Don't forget to call `make_robot` before you call `run`!
    p = [0.0, 0.0, 0.0]
    dp = [1.0, 1.0, 1.0]
    robot = make_robot()
    x_trajectory, y_trajectory, best_err = run(robot, p)

    it = 0
    while sum(dp) > tol:
        # print("Iteration {}, best error = {}".format(it, best_err))
        for i in range(len(p)):
            p[i] += dp[i]
            robot = make_robot()
            x_trajectory, y_trajectory, err = run(robot, p)

            if err < best_err:
                best_err = err
                dp[i] *= 1.1
            else:
                p[i] -= 2 * dp[i]
                robot = make_robot()
                x_trajectory, y_trajectory, err = run(robot, p)

                if err < best_err:
                    best_err = err
                    dp[i] *= 1.1
                else:
                    p[i] += dp[i]
                    dp[i] *= 0.9
        it += 1
```

### 6.  SLAM
A method of mapping. Building 2D/3D environmental maps.
SLAM = Simultaneous localization and mapping

Since inevitably robots have drifting problems, they must perform localization when mapping, otherwise the map is distorted.

#### 6.1 Graph SLAM: Easiest SLAM 

**Input**:
The measurement of three constraints.
- initial location constraints (as a non-biased base)
- relative motion constraints
- relative measurement constraints
![](/img/in-post/2018-06-25-AutoPilot/1520167771680.png)

**Math**:
According to Kalman Filter, the position of the robot is a overlap of Gaussian distributions.
$x_0\rightarrow x_1:\frac{1}{\sqrt{2\pi \sigma^2}}\exp -\frac{1}{2}\frac{(x_1-x_0-10)^2}{\sigma^2}$
$x_1\rightarrow x_2:\frac{1}{\sqrt{2\pi \sigma^2}}\exp -\frac{1}{2}\frac{(x_2-x_1-5)^2}{\sigma^2}$
After Multiplication:
$\frac{1}{2\pi \sigma^2}\exp-\frac{1}{2}\frac{(x_1-x_0-10)^2 + (x_2-x_1-5)^2}{\sigma^2}$
Which is controlled by the variance of:
$\frac{x_1-x_0-10}{\sigma}$ and $\frac{x_2-x_1-5}{\sigma}$

Here $\sigma$ is confident factor in code.
Maintain a matrix $\Omega$ as correlation between all positions and landmarks.
Maintain a matrix $\xi$ as distance matrix.
$\mu=\Omega^{-1}*\xi$ is the coordinates of all positions and landmarks on map.

**Algorithm**:
```python
# 1D graph SLAM with only one landmark
def GraphSLAM(init, moves, toLandmark, confidence_factor):
	element_num = len(toLandmark) + 1
	Omega = np.zeros([element_num, element_num])
	Xi = np.zeros([element_num, 1])
	
	Omega[0,0] = 1/confidence_factor
	Xi[0,0] = init/confidence_factor

	for i in range(len(moves)):
		move = moves[i]
		correlation = 1/confidence_factor
		value = move/confidence_factor
		Omega[i,i] += correlation
		Omega[i+1, i+1] += correlation
		Omega[i, i+1] -= correlation
		Omega[i+1, i] -= correlation
		Xi[i, 0] -= value
		Xi[i+1, 0] += value
	for i in range(len(toLandmark)):
		correlation = 1/confidence_factor
		value = toLandmark[i]/confidence_factor
		Omega[i, i] += correlation
		Omega[-1, -1] += correlation
		Omega[-1, i] -= correlation
		Omega[i, -1] -= correlation
		Xi[i, 0] -= value
		Xi[-1, 0] += value
		
	mu = Omega.inverse() * Xi
	return mu
```

#### 6.2 Problems of graph SLAM:

**Efficientcy of graph SLAM**:
If you use graph SLAM for a very very long time, the number of landmarks tend to be unchanged, but there would be huge numbers of positions, making the graph too big to analyze.
**Solution**: Only take recent positions into graph.
![](/img/in-post/2018-06-25-AutoPilot/1520219139159.png)

**Correspondence Problem**:
How do graph SLAM deal with non-distinguishable landmarks.
**ransack**: randomly assign correspondence for 3 or 4 landmarks, solve the equation and look for the best nearby matches to fill them in.

**Landmark growth**:
Can just update the landmarks in graph using the same techniques as in motion update.

**What is landmark?**:
Not traditional landmarks, but a scan of area as a single landmark.

**Extract uncertainty (covariance) from Omega**:
Omega is an inverse covariance matrix, just invert the matrix, and you will get full covariances for all of the landmarks and positions.

### 7. Putting things together

#### 7.1 Localization

\-|MultiModal|Exponential|Useful
:--:|:--:|:--:|:--:
Kalman|||yes
Histogram|yes|yes|yes
Particles|yes|yes|yes

#### 7.2 Planning

\-|Continuous|Optimal|Universial|Local
:--:|:--:|:--:|:--:|:--:
BreadFirst||yes||
A star||yes||
DP||yes|yes|
Smoothing|yes|||yes

#### 7.3 Control

\-|Avoid Overshoots|Minimizing Error|Compensating Drifts
:--:|:--:|:--:|:--:
Proportion||yes|
Integral|||yes
Differential|yes||