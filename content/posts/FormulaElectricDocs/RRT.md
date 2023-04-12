+++ 
draft = false
date = 2023-04-11T21:24:49-07:00
title = "Documentation for FEB RRT algorithm"
description = ""
slug = ""
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

[Click here for the parent post about Formula Electric](/posts/formulaelectric/)


# Result

{{< youtube id="mXnNdAj1WYo" >}}


# Overview

Our implimentation is heavily based on [this paper](https://drive.google.com/file/d/1isQxfyFiHrueqK4rpEJMvF1BJfzxMYbw/view)

We still have optimizations for RRT*, like 
- limiting the space we randomly sample points from to in front of the car
- punishing steep angle changes

Here is a good video explaining RRT*:

{{< youtube id="Ob3BIJkQJEw" >}}


- The main program is RRT.py
- RRT.py has an infinite loop which instantiaites and calls other classes.
- RRT.py uses pygame, TreePath, and Obstacle datastructures to do and display the RRT algorithm.
- treepath.py is the main data structure representing the treelike path of rrt.
- Obstacle.py has the Obstacle class that uses the Point data structure to represent obstacles.

## Classes

- point.py
- Obstacle.py
- Tests.py
- treepath.py
- RRT.py

# Deep dive into each Class

## point.py

Super simple class that represents a point (for obstacle class)

It is easier to read code when we write point.x instead of point[0]

### dependencies

None

### methods

makes a point

```python
def __init__(self, x, y):
        self.x = x
        self.y = y
```

### attributes

```python
self.x # x coordinate of point

self.y # y coordinate of point
```


## Obstacle.py

Represent an obstacle in 2 dimentions. We want to feed in a list of Point objects and create an obstacle. This obstacle should be able to draw itself and handle collisions with paths.

### dependencies

```python
import pygame
from shapely.geometry import LineString
import numpy as np

```

### methods

Take in a list of Point objects

```python
# initialize obstacle with its edge points (use list of Point data structure)
	def __init__(self, points):
		self.points = points

```

Checks if  path from node N to node K intersects itself

```python
	# checks if a line from node N to node K intersects with it
	def instersects(self, N, K):
		# make line segment data structure
		pathline = LineString([(N.xpos, N.ypos), (K.xpos, K.ypos)])

		# check if pathline intersects with any of the obstacle edges
		for i in range(len(self.points)):
			tempLine = LineString([(self.points[i].x, self.points[i].y), (self.points[(i+1)%len(self.points)].x, self.points[(i+1)%len(self.points)].y)])
			if pathline.intersects(tempLine):
				return True
		
		# if no intersection with any edge is found, return false
		return False
	
```

Draw itself

```python
	# draw the obstacle in pygame
	def draw(self, surface):
		drawPoints = []
		for i in range(len(self.points)):
			drawPoints.append((self.points[i].x, self.points[i].y))
		pygame.draw.polygon(surface, obstacleColor, drawPoints)

```

## treepath.py

Main datastructure. It is a node with position attributes, a parent attribute, and a list of children.
A root node is initialized and the successive nodes are added as children to the tree.

### dependencies

```python
import numpy as np
import pygame
```

### methods

Initialize attributes

```python
def __init__(self, xpos, ypos, parent = None):

        self.xpos = xpos
        self.ypos = ypos
        self.parent = parent
        self.children = []
        self.path = [self]
        nodeList.append(self)

```

gets the distance between two nodes (static)

```python
def getDistance(leaf1, leaf2):
        x = abs(leaf1.xpos - leaf2.xpos)
        y = abs(leaf1.ypos - leaf2.ypos)

        return np.sqrt(x*x + y*y)

```
gets distance from self to a node

```python
def getDistanceTo(self, leaf1):
        x = abs(leaf1.xpos - self.xpos)
        y = abs(leaf1.ypos - self.ypos)

        return np.sqrt(x*x + y*y)

```

Creates a node at specified position and adds it as a child.

```python
def addChild_byPosition(self, xpos, ypos):
        self.children.append(TreePath(xpos, ypos, self))
        

```

Adds a specified node as a child to self

```python
def addChild_byPointer(self, tree):
        self.children.append(tree)
        tree.parent = self
        
    
```

removes a specified child

```python
def removeChild(self, tree):
        self.children.remove(tree)


    # return the node most towards x_new from x_nearest under the constraints (disctance)
```

modifies the coordinates of x_new to be exactly steerDistance away from x_nearest

```python
def distanceSteer(self, x_nearest, x_new):	
        
        dist = x_nearest.getDistanceTo(x_new)
        x_new.xpos = x_nearest.xpos + steerDistance* (x_new.xpos - x_nearest.xpos)/dist
        x_new.ypos = x_nearest.ypos + steerDistance* (x_new.ypos - x_nearest.ypos)/dist

```

same as above but makes sure x_new is not going "backwards" with respect to the path.\

```python
def direction_distance_steer(self, x_nearest, x_new):

        # distance constraint
        dist = x_nearest.getDistanceTo(x_new)
        x_new.xpos = x_nearest.xpos + steerDistance* (x_new.xpos - x_nearest.xpos)/dist
        x_new.ypos = x_nearest.ypos + steerDistance* (x_new.ypos - x_nearest.ypos)/dist

        # dot product for direction correcting
        if x_nearest.parent != None:
            if (x_new.xpos - x_nearest.xpos) * (x_nearest.parent.xpos - x_nearest.xpos) + (x_new.ypos - x_nearest.ypos) * (x_nearest.parent.ypos - x_nearest.ypos) > 0:
                x_new.xpos *= -1
                x_new.ypos *= -1
    
        
    #return node close to node closest to K
```

gets the nearest node to K out of all children

```python
def nearest(self, K):
        # we only want the nearest node, not the distance to it
        
        node, dist = self.nearestHelper(K)
        return node
    
```


gets the nearest node to K out of all children and its distance

```python
def nearestHelper(self, K):
        
        distanceToK = self.getDistance(K) # NEED to impliment distance for tree (not just leafs), also impliment constraints here

        # add itself to possible nodes to consider
        nodes = [self]
        distances = [distanceToK]

        # add the closest nodes in children trees
        # print("DO I have myself as child?:  " + str(self in self.children))
        for child in self.children:
            # print("my xpos", child.xpos)
            tempNode, tempDist = child.nearestHelper(K) # nearestHelper(K, self, distanceToK)
            nodes.append(tempNode)
            distances.append(tempDist)
        
        # find lowest path out of children and self
        tempMin = np.inf
        for i in range(len(distances)):
            if tempMin > distances[i]:
                lowestNode = nodes[i]
                tempMin = distances[i]

        # return nearest node and the distance to the node
        return lowestNode, tempMin
```

Removes a specified child node

```python
def removeChild(self, child):
        self.children.remove(child)

    #return node and distance of node closest to K recursive
```

Checks if the path from self to node K intersects any obstacles

```python
def ObstacleFree(self, K, obstacles):

        #iterates through a list of obstacle objects
        for obstacle in obstacles:
            if obstacle.instersects(self, K):
                return False
        return True


    # takes in radius and node, returns all nodes within distance r to node
```

Gets all nodes within radius of N.

```python
def Near(self, radius, N):
        
        returnArray = []
        #recursive call either including itself or not
        if (self.getDistance(N) <= radius):
              returnArray.append(self)
        for i in self.children:
            returnArray.extend(i.Near(radius, N))
        return returnArray

```

returns the cost (distance it takes from the root node) to target

```python
def cost(target):
        if target.parent == None:
            return 0
        return TreePath.getDistance(target, target.parent) + TreePath.cost(target.parent)
    


#gets total euclidian distance required to travel along certain path 
#input is list of nodes in order, output is distance
```

returns the cost (distance) to traverse the list of nodes

```python
def c(self, pathlist):
        if len(pathlist) == 0:
            return 0
        else:
            totalDist = 0
            for i in range(len(pathlist) - 1):
                totalDist += TreePath.getDistance(pathlist[i-1], pathlist[i])
            
        return totalDist
                
```

get a new random node

```python
def sample_free(xdimension, ydimension):
        newx = np.random(0, xdimension)
        newy = np.random(0, ydimension)
        return TreePath(newx, newy)

```

draw the tree

```python
def draw(self,surface):
        pygame.draw.circle(surface, nodeColor, (self.xpos, self.ypos), nodeRadius)
        # print("drawing childern: ", self, self.children)
        for a in self.children:
            pygame.draw.line(surface, edgeColor, (self.xpos, self.ypos), (a.xpos, a.ypos))
            a.draw(surface)

```

draw the path from the root node to node

```python
def drawPathTo(surface, node):
        if node.parent == None:
            return
        pygame.draw.line(surface, FoundPathColor, (node.xpos, node.ypos), (node.parent.xpos, node.parent.ypos), width=foundPathWidth)
        TreePath.drawPathTo(surface, node.parent)

```


### attributes

```python
self.xpos = xpos
self.ypos = ypos
self.parent = parent
self.children = []
self.path = [self]
nodeList.append(self)
```


## RRT.py

Main program we run that uses pygame to display the RRt algorithm.

We create an obstacles list Obstacles, and set various settings such as the goal node location, optimization steps.

The infinite loop displays the algorithm every iteration and waits 1 second when the algorithm is completed.


### dependencies

```python
import sys
sys.path.append('../')
from penisC.Tree.treepath import TreePath
from penisC.Tree.Obstacle import Obstacle
from penisC.Tree.point import Point
import pygame
import numpy as np
from numpy import random
```


### methods

return a random node from the search space (the screen)
```python
# return a new node in a random place on the screen
def SAMPLE_FREE(width, height):
    x = random.rand() * width
    y = random.rand() * height
    return TreePath(x, y)
```

### attributes

Dimentions of the game area:

```python
width = 700
height = 700
dim = [width, height]
# Set up the drawing window
screen = pygame.display.set_mode(dim)
```

List of Obstacles

```python
Obstacles = [Obstacle([Point(0,0), Point(100,0), Point(100,100), Point(0,100)]), 
             Obstacle([Point(200,200), Point(200,270), Point(150,360), Point(10,400)]),
             Obstacle([Point(500,200), Point(500,270), Point(600,360), Point(400,400)]),
             Obstacle([Point(300, 0), Point(400, 0), Point(300, 600), Point(400,600)])
             
             ]
```

Initialize Goals with goal node radius 50 and location 600, 100:

```python
goalTolerance = 50
goalNode = TreePath(600, 100)
```

used to exit out of window

```python
running = True
```

Instantiaite the root node of the rrt tree. Set the root node to location 100, 200.

```python
rrt = TreePath(100,200)
```

found does nothing in practice, but turns to true when a path to the goal is found.
optimization steps is the amount of additional RRT iterations we do after we find a path to the goal (to hopefully optimize our path).

```python
found = False
optimizationSteps = 400
```

x_rand is the new node initialized as a random node from the sample space.
x_nearest is the nearest node in the rrt tree to x_rand.
```python
x_rand = SAMPLE_FREE(width, height)
        x_nearest = rrt.nearest(x_rand)
```

near Nodes is a list of nodes that are within 100 units of x_rand. Used to optimize the treepath.

```python
nearNodes = rrt.Near(100, x_rand)
```

## Tests.py

This is where we have the unit tests for all other classes.

### dependencies

```python
import treepath
from treepath import TreePath
from obstacle import Obstacle
from point import Point
import unittest
```

### methods


Tests if the constructor of treepath is working correctly

```python
def test_TreeInitialization(self):
        t1 = TreePath(0,0)
        self.assertEqual(t1.xpos, 0, 'The xpos is wrong')
        self.assertEqual(t1.ypos, 0, 'The ypos is wrong')
        self.assertEqual(t1.parent, None, 'The parent is wrong')

        t2 = TreePath(-5,7.9)
        self.assertEqual(t2.xpos, -5, 'The xpos is wrong')
        self.assertEqual(t2.ypos, 7.9, 'The ypos is wrong')
        self.assertEqual(t2.parent, None, 'The parent is wrong')
        self.assertNotEqual(t2, t1, 'different initializations are equal')

        t3 = TreePath(-5,7.9)
        self.assertNotEqual(t2, t3, 'different initializations, with same coordinate are equal when they shouldnt be')

    
```

Tests if adding children is working properly

```python
def test_TreeStructure(self):
        n1 = TreePath(0,0)
        n2 = TreePath(2,2)
        n3 = TreePath(9,9)
        n4 = TreePath(10,10)
        n5 = TreePath(10,10) # try out duplicates
        n6 = TreePath(10,15)
        n1.addChild_byPointer(n2)
        n1.addChild_byPointer(n3)
        n2.addChild_byPointer(n4)
        n2.addChild_byPointer(n5)
        n2.addChild_byPointer(n6)

        # structure:
        #      n1
        #     /  \
        #    n2   n3
        #  / | \  
        # n4 n5 n6



        self.assertEqual(n1.children, [n2, n3])
        self.assertEqual(n2.children, [n4, n5, n6])
        self.assertEqual(n4.children, [])
        self.assertEqual(n2.parent, n1)
        self.assertEqual(n5.parent, n2)

```

Tests if removing children is working properly.

```python

def test_RemoveChild(self):
        n1 = TreePath(0,0)
        n2 = TreePath(2,2)
        n3 = TreePath(9,9)
        n4 = TreePath(10,10)
        n5 = TreePath(10,10) # try out duplicates
        n6 = TreePath(10,15)
        n1.addChild_byPointer(n2)
        n1.addChild_byPointer(n3)
        n2.addChild_byPointer(n4)
        n2.addChild_byPointer(n5)
        n2.addChild_byPointer(n6)

        # structure:
        #      n1
        #     /  \
        #    n2   n3
        #  / | \  
        # n4 n5 n6


        n2.removeChild(n5)
        n2.removeChild(n4)
        # structure:
        #      n1
        #     /  \
        #    n2   n3
        #  / 
        # n6
        self.assertEqual(n2.children, [n6])
        self.assertEqual(n1.children, [n2, n3])


        n1.removeChild(n2)
        # structure:
        #      n1
        #     /  
        #    n3
        self.assertEqual(n1.children, [n3])
        
```

Tests sanity

```python
def test_sanity(self):
        assert 1 > 0
    
```

Tests get distance function


```python
def test_getDistanceTo(self):
        
        x1 = 1.2
        y1 = 5.3
        
        x2 = -2.3
        y2 = 100.5

        t1 = TreePath(x1,y1)
        t2 = TreePath(x2, y2)

        
        self.assertAlmostEqual(t1.getDistanceTo(t2), 95.2643165094)


```

Tests get distance function


```python
def test_getDistance(self):
        
        x1 = 1.2
        y1 = 5.3
        
        x2 = -2.3
        y2 = 100.5

        t1 = TreePath(x1, y1)
        t2 = TreePath(x2, y2)

        self.assertAlmostEqual(TreePath.getDistance(t1, t2), 95.2643165094)

```

Tests adding children by specifying the position where the new child should be created.

```python
def test_addChild_byPosition(self):
        treepath.nodeList = []
        t = TreePath(1,2)
        t.addChild_byPosition(3,45)
        self.assertEqual(t.children[0].parent, t)
        self.assertEqual(treepath.nodeList[0], t)
        self.assertEqual(treepath.nodeList[1], t.children[0])

        t.addChild_byPosition(3,22)

        t.children[0].addChild_byPosition(23,4)
        
        self.assertEqual(t.children[0].children[0], treepath.nodeList[3])

```

Tests adding children.

```python
def test_addChild_byPointer(self):
        treepath.nodeList = []
        t = TreePath(1,2)
        c1 = TreePath(3,4)
        c2 = TreePath(5,6)
        t.addChild_byPointer(c1)
        c1.addChild_byPointer(c2)

        self.assertEqual(t.children, [c1])
        self.assertEqual(c1.children, [c2])

        t.addChild_byPosition(3,22)

        self.assertEqual(len(t.children), 2)

```

Tests the nearest function that returns the nearest node to a given node.

```python
def test_nearest(self):
        n1 = TreePath(0,0)
        n2 = TreePath(2,2)
        n3 = TreePath(9,9)
        target = TreePath(10,10)
        n1.addChild_byPosition(2,3)
        n1.addChild_byPosition(3,3)
        n1.addChild_byPosition(3,2)
        n1.addChild_byPosition(3,5)
        n1.addChild_byPointer(n2)
        n2.addChild_byPosition(5,5)
        n2.addChild_byPosition(6,6)
        n2.addChild_byPointer(n3)

        self.assertEqual(n1.nearest(target), n3)

```

Tests Obstacle free.


```python
def test_ObstacleFree(self):
        n1 = TreePath(-1,-1)
        n2 = TreePath(10,0)
        n3 = TreePath(10, 10)
        n4 = TreePath(0, 10)
        n5 = TreePath(-10,-10)

        obstacles = [Obstacle([Point(0,0), Point(1,0), Point(1,1), Point(0,1)])]


        # on the board should return false?

        self.assertEqual(n1.ObstacleFree(n2, obstacles), True) 
        self.assertEqual(n1.ObstacleFree(n3, obstacles), False)
        self.assertEqual(n1.ObstacleFree(n4, obstacles), True) 
        self.assertEqual(n1.ObstacleFree(n5, obstacles), True)

        # same both ways
        self.assertEqual(n2.ObstacleFree(n1, obstacles), True)
        self.assertEqual(n3.ObstacleFree(n1, obstacles), False)
        self.assertEqual(n4.ObstacleFree(n1, obstacles), True) 
        self.assertEqual(n5.ObstacleFree(n1, obstacles), True)

    # returns itself as well if th enode is already added to tree
```

Tests Near  which returns nodes within a given radius to a node.

```python
def test_Near(self):
        t1 = TreePath(0,0)
        n1 =TreePath(3,3)
        t1.addChild_byPointer(n1)
        t1.addChild_byPosition(10,10)
        n2 = TreePath(3,4)
        # t1.addChild_byPointer(n2)
        self.assertEqual(t1.Near(2, n2), [n1], 'Near is wrong. Case: node not already in tree')

        t1.addChild_byPointer(n2)
        self.assertIn(t1.Near(2, n2), [[n1, n2], [n2, n1]], 'Near is wrong. Case: node already in tree')

```

tests the obstacles class and instersections of paths.


```python
def test_Obstacle(self):
        # make a square of length 1
        o = Obstacle([Point(0,0), Point(1,0), Point(1,1), Point(0,1)])
        n1 = TreePath(0,0)
        n2 = TreePath(10,0)
        n3 = TreePath(10, 10)
        n4 = TreePath(0, 10)
        n5 = TreePath(-10,-10)

        o = Obstacle([Point(0,0), Point(1,0), Point(1,1), Point(0,1)])
        
        self.assertEqual(o.instersects(n1, n3), True) 
        self.assertEqual(o.instersects(n3, n5), True) 
        self.assertEqual(o.instersects(n1, n2), True) # on the board should return true?
        self.assertEqual(o.instersects(n4, n3), False) 

        # both ways
        self.assertEqual(o.instersects(n3, n1), True) 
        self.assertEqual(o.instersects(n5, n3), True) 
        self.assertEqual(o.instersects(n2, n1), True) # on the board should return true?
        self.assertEqual(o.instersects(n3, n4), False) 
        



```

Tests if nodeList is being updated every time we make a new node.

```python
def test_randomTest(self):
        treepath.nodeList = []
        t = TreePath(0,0)
        x = t
        for i in range(100):
            x.addChild_byPosition(0,0)
            x = x.children[0]
        
        self.assertEqual(len(treepath.nodeList), 101, 'the list does not contain them all.')
        

```

Tests the cost function (the distance it takes to get to it from the root node)


```python
def test_Cost(self):
        treepath.nodeList = []
        t = TreePath(0,0)
        for i in range(10):
            t.addChild_byPosition(1,i)
            t1 = t.children[0]
            for j in range(10):
                t1.addChild_byPosition(2,i)
                t2 = t1.children[0]
    
    #     .     .
    #     .     .
    #     t?   t?
    #   /    /
    # t - t1 - t2
        self.assertAlmostEqual(2.0, t2.xpos)
        self.assertAlmostEqual(0.0, t2.ypos)
        self.assertAlmostEqual(1.0, TreePath.cost(t1))
        self.assertAlmostEqual(2.0, TreePath.cost(t2))
    
```

Tests the cost of traversing a given list of nodes (total distance)

```python
def test_C(self):
        t = TreePath(0,0)
        t.addChild_byPosition(10,10)
        t.addChild_byPosition(20,20)
        t.addChild_byPosition(15,15)

        self.assertAlmostEqual(t.c([t.children]), 35.35535)


```

Not implimented


```python
def test_Nearest(self):
        pass




