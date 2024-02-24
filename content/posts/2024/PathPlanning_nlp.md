+++ 
draft = false
date = 2024-02-24T07:42:21-08:00
title = "Path Planning with NLP"
description = ""
slug = ""
authors = []
tags = ["essay"]
categories = []
externalLink = ""
series = []
+++



Path planning is the challenge of calculating the best path for a race car to take through a given track.

The example below solves for the positions, velocity, and orientation of the car at every point in the optimal path (racing line).

![example racing line](/img/examplePath.png)


After developing an RRT* based approch to path planning, my friend [Reid Dye](https://reid.xz.ax) introduced me to a Non-Linear programming based way to solve the problem. His way was faster, more accurate, more custamizable with things like friction and velocity constraints, and overall more organized.

So I decided to replicate his approach, and create a NLP (non-linear programming) based solution. 

Here is a summary.

## Final solution

{{< youtube id="4rIr03vNlNk" >}}

After solving for the path with a few points, a subdivide algorithm breakes the path into 0.1 second increments so we have many datapoints in the final path.

If there is a steep angle in the track, the initial guess of the heading may be looking the opposite way as it should be, and this usually confuses the solver to converging to an infeasible solution. This is solvable with a smarter intial guess, but its not a fun problem so I decided not to resolve the issue.


## How it works

Reid has made a [really good explanation](https://reid.xz.ax/global_opt_docs) of the approach for the whole FEB team.

My implimentation assumes constant wheel torque with fixed max velocity and max acceleration.
I also assume a constant acceleration envelope.


## The process

Here are the issues I faced along the way:


- I expllicitly solved for the vehicle dynamics (with sinh and cosh), so the dynamics of the car had a division by 0 when the steering was set to 0. This led to the solver crashing when it got near steering = 0. The solution was to phrase the dynamics as an initial value problem with differential equations, then letting casadi solve for the function.

<div style="background-color: #333; color: #fff; padding: 10px;">
```python
def continuous_dynamics_fixed_x_order(x, u, car_params={'l_r': 1.4987, 'l_f':1.5213, 'm': 1.}):
        """Defines dynamics of the car, i.e. equality constraints.
        parameters:
        state x = [xPos,yPos,v,theta]
        input u = [F,phi]
        """
        beta = arctan(car_params['l_r']/(car_params['l_f'] + car_params['l_r']) * tan(u[1]))
        xdot = x[2]*cos(x[3] + beta)
        ydot = x[2]*sin(x[3] + beta)
        vdot = u[0]
        thetadot = x[2] / car_params['l_r'] * sin(beta)

        return vertcat(xdot,
                        ydot,
                        vdot,  
                        thetadot)                 
    
    def discrete_custom_integrator(n = 3, car_params={'l_r': 1.4987, 'l_f':1.5213, 'm': 1.}):
        x0 = MX.sym('x0', 4)
        u = MX.sym('u', 2)
        dt = MX.sym('t')
        xdot = NLPSolver.continuous_dynamics_fixed_x_order(x0, u, car_params)
        f = Function('f', [x0, u], [xdot])
        
        x = x0
        for i in range(n):
            xm = x + f(x, u)*(dt/(2*n))
            x = x+f(xm, u)*(dt/n)
        return Function('integrator', [x0, u, dt], [x])

```
</div>

- The solver kept on converging to a local infeasable point, and the path it showed was completely ignoring the dynamics of the car. It turns out I was passing in the first control input for every single dynamics calculation regrdless of which point I was trying to calculate (I messed up u[i, :] vs u[:, i]). The purple path below is the path produced based on the dynamics of the car and control inputs (you can see it differs from the outputted path). The problem is simple, but took me way too long to catch. I even made a game so I could control a car following the dynamics using arrow keys to test my dynamics.

![broken dynamics constraint](/img/broken.png)

- The initial guess has to be a pretty good guess. The most straightforward method of making an initial guess is following the midpoints, and setting the headings to just rotate from 0 to 2pi. However, if there is a steep angle in the track, the initial guess of the heading may be looking the opposite way as it should be, and this usually confuses the solver to converging to an infeasible solution.

![initial guess](/img/initialguess.PNG)

![buggy path](/img/buggedpath.PNG)









