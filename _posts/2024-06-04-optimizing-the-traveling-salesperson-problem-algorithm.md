---
layout: post
title: Optimizing the Traveling Salesperson Problem Algorithm
date: 2024-06-04 17:30:00
description: The Traveling Salesperson Problem (TSP) is a classic optimization problem with numerous real-world applications, from logistics to manufacturing. In this blog post, we’ll explore a brute-force approach to solving the TSP using a recursive algorithm, laying the foundation for more advanced optimization techniques.
tags: AI, Python, Optimization
categories: AI
tabs: true
---

Key takeaways from this post:

1. The Traveling Salesperson Problem is a classic optimization problem with many real-world applications.
2. A brute force approach to solving the TSP involves generating all possible permutations of the destinations and calculating the cost(e.g., distance or emissions) for each permutation.
3. Recursive algorithms can be used to generate permutations and solve the TSP.
4. While brute force approaches are simple to understand and implement, they become impractical for larger problem sizes.

The Traveling Salesperson Problem (TSP) is a classic optimization problem with numerous real-world applications, from logistics to manufacturing. In this blog post, we’ll explore a brute-force approach to solving the TSP using a recursive algorithm, laying the foundation for more advanced optimization techniques.

---

# **The Problem: Shipping Pineapples**

> Imagine that you've been assigned the task to plan the route of a container ship loaded with pineapples. The ship starts in Panama, loaded with delicious Fairtrade pineapples. There are four other ports, New York, Casablanca, Amsterdam, and Helsinki, where pineapple-craving citizens are eagerly waiting. The ship must visit each of the four destination ports exactly once, but the order in which each port is visited is free. The goal is to minimize the carbon emissions, which means that a shorter route is better than a longer one.

## Listing Pineapple Routes

To solve the TSP, we start by listing all possible routes, or permutations, of the ports. We can use a recursive algorithm to generate these permutations:

```python
portnames = ["PAN", "AMS", "CAS", "NYC", "HEL"]

def permutations(route, ports):
    # Write your recursive code here
    if not ports < 1: # check if there is not port in ports
        # Print the port names in route when the recursion terminates
        print(' '.join([portnames[i] for i in route]))
    else:
        for i in range(len(ports)):
            # Add the current port into the route to become a new route
            next_route = route + [ports[i]]
            # Remove the current port from the ports
            remaining_ports = ports[:i] + ports[i+1:] # A method that remove the current ports
            # Recursively generate permutations with the updated route and remaining ports
            permutations(next_route, remaining_ports)

# This will start the recursion with 0 ("PAN") as the first stop
permutations([0], list(range(1, len(portnames))))

```

This recursive algorithm works by:

1. Starting with a route containing only the first port (Panama) and a list of the remaining ports.
2. If there are no remaining ports, the recursion terminates, and the current route is printed.
3. If there are remaining ports, the algorithm iterates through each one, adding it to the current route and removing it from the list of remaining ports.
4. The algorithm then recursively calls itself with the updated route and remaining ports.
5. This process continues until all possible permutations have been generated.

Let’s examine the behavior of this algorithm:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tsp_algorithm_visualization_1.jpg" title="TSP Algorithm Visualization 1" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We keep removing `port` from `ports` and adding to the `route` then keeping the recursive when we reach the first end (which means we print out our first route) :

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tsp_algorithm_visualization_2.jpg" title="TSP Algorithm Visualization 2" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We go back to the previous branch and do the recursive from there ([0, 1, 2, 3] we finish this one from the previous and start from the second `i` from for which is [0, 1, 2, 4])

Then we start a new `for` , choosing `3` as the first one and printing

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tsp_algorithm_visualization_3.jpg" title="TSP Algorithm Visualization 3" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

And so on and so on.

---

Let’s look at the example solution from the course:

```python
def permutations(route, ports):
    if len(ports) < 1:
        print(' '.join([portnames[i] for i in route]))
    else:
        for i in range(len(ports)):
		        # use one line to achieve it
            permutations(route+[ports[i]], ports[:i]+ports[i+1:])

permutations([0], list(range(1, len(portnames))))
```

It’s crazy neat, we learn from it that we can manipulate our variable in parameters.

## Calculating Pineapple Route Emissions

Now we add more details to our problem:

> Having listed the alternatives, next we can calculate the carbon emissions for each of them. Below you'll find the distances between the ports in kilometers in a five-by-five table.
>
> |     | PAN   | AMS  | CAS  | NY   | HEL   |
> | --- | ----- | ---- | ---- | ---- | ----- |
> | PAN | 0     | 8943 | 8019 | 3652 | 10545 |
> | AMS | 8943  | 0    | 2619 | 6317 | 2078  |
> | CAS | 8019  | 2619 | 0    | 5836 | 4939  |
> | NY  | 3652  | 6317 | 5836 | 0    | 7825  |
> | HEL | 10545 | 2078 | 4939 | 7825 | 0     |
>
> Let's assume that the boat is relatively modern and produces 0.020 kg of CO2 emissions per kilometer for the amount of pineapples that we are shipping. Thus, you can calculate the emissions caused by traveling from Panama to Amsterdam by first looking up the distance in the first row, second column of the table (highlighted in the above table): 8943 km, and then multiplying this with 0.020kg/km to get 178.9 kg.

Here's the updated code that calculates the emissions for each route and keeps track of the route with the lowest emissions:

```python
portnames = ["PAN", "AMS", "CAS", "NYC", "HEL"]

# https://sea-distances.org/
# nautical miles converted to km

D = [
        [0,8943,8019,3652,10545],
        [8943,0,2619,6317,2078],
        [8019,2619,0,5836,4939],
        [3652,6317,5836,0,7825],
        [10545,2078,4939,7825,0]
    ]

# https://timeforchange.org/co2-emissions-shipping-goods
# assume 20g per km per metric ton (of pineapples)

co2 = 0.020

# DATA BLOCK ENDS

# these variables are initialised to nonsensical values
# your program should determine the correct values for them
smallest = 1000000
bestroute = [0, 0, 0, 0, 0]

def permutations(route, ports):
    # write the recursive function here
    # Remember to calculate the emissions of the route as the recursion ends
    # and keep track of the route with the lowest emissions
    global smallest, bestroute
    if not ports:
        emissions = 0
        # This `for` part is equivalent to:
        # emissions = co2*(D[route[0]][route[1]] + D[route[1]][route[2]] + D[route[2]][route[3]] + D[route[3]][route[4]])
        # calculate the emissions which is the total distance from the whole route then * co2
        for i in range(len(route) - 1):
            emissions += D[route[i]][route[i+1]]
        emissions *= co2
        if emissions < smallest:
            smallest = emissions
            bestroute = route
    else:
        for i in range(len(ports)):
            permutations(route + [ports[i]], ports[:i] + ports[i+1:])

def main():
    # Do not edit any (global) variables using this function, as it will mess up the testing
    # This will start the recursion
    permutations([0], list(range(1, len(portnames))))

    # print the best route and its emissions
    print(' '.join([portnames[I] for I in bestroute]) + " %.1f kg" % smallest)

main()
```

Let’s check out the example solution:

```python
portnames = ["PAN", "AMS", "CAS", "NYC", "HEL"]

D = [
        [0,8943,8019,3652,10545],
        [8943,0,2619,6317,2078],
        [8019,2619,0,5836,4939],
        [3652,6317,5836,0,7825],
        [10545,2078,4939,7825,0]
    ]

co2 = 0.020

smallest = 1000000
bestroute = None

def permutations(route, ports):
    global smallest, bestroute
    if len(ports) < 1:
        score = co2 * sum(D[i][j] for i, j in zip(route[:-1], route[1:]))
        if score < smallest:
            smallest = score
            bestroute = route
    else:
        for i in range(len(ports)):
            permutations(route+[ports[i]], ports[:i]+ports[i+1:])

def main():
    permutations([0], list(range(1, len(portnames))))

    print(' '.join([portnames[i] for i in bestroute]) + " %.1f kg" % smallest)

main()
```

```python
score = co2 * sum(D[i][j] for i, j in zip(route[:-1], route[1:]))
```

## A Note on zip()

In the example solution, you might have noticed the use of `zip()` to calculate the total distance:

Let’s say you have the following route:

```python
route = [0, 2, 3, 1]
```

```
•	route[:-1] would be [0, 2, 3]
•	route[1:] would be [2, 3, 1]
•	zip(route[:-1], route[1:]) would produce pairs (0, 2), (2, 3), (3, 1)
```

# Conclusion

In this blog post, we explored a brute force approach to solve the Traveling Salesperson Problem using a recursive algorithm. We started by generating all possible permutations of the ports and then calculated the carbon emissions for each route using a distance table.

While this approach works, it becomes increasingly inefficient as the number of ports grows, as it generates and evaluates all possible permutations. In future posts, we’ll explore more advanced optimization techniques that can solve the TSP more efficiently.
