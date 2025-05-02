---
title: "Euler Graph and Leetcode Problem 2097 Valid Arrangement of Pairs"
seoTitle: "Mastering Eulerian Graphs: Solving LeetCode Problem 2097 Step-by-Step"
seoDescription: "Explore Euler graphs and solve Leetcode Problem 2097, finding a valid arrangement of pairs using Eulerian paths"
datePublished: Mon Dec 02 2024 06:10:14 GMT+0000 (Coordinated Universal Time)
cuid: cm46msfll000v08jl88ig8m1v
slug: euler-graph-and-leetcode-problem-2097-valid-arrangement-of-pairs
tags: problem-solving, dsa, leetcode, graphs

---

In this article we will be exploring the euler graph. So basically i was solving (trying to solve 🫠) the leetcode POTD (i.e [2097\. Valid Arrangement of Pairs](https://leetcode.com/problems/valid-arrangement-of-pairs/)) , now in this problem if we closely observe ( or simply looks in the topics section 😌) we will get to know that what we need to do is treat all the queries as edges of the graph and we just need to find a path a such that every edge should be visited only once .

## ***Pretty Complex right ?***

First we will understand what is a euler graph

An **Euler graph** is a connected graph in which there exists an **Eulerian circuit**, meaning a path that starts and ends at the same vertex while traversing every edge of the graph exactly once.

For a graph to be an Euler graph, it must satisfy these conditions:

1. **Every vertex has an even degree** (an even number of edges connected to it).
    
2. The graph must be connected, meaning there is a path between any two vertices.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733119579930/3336b79a-3c46-4ff6-a837-36d13e9a3340.png align="left")

So to find out a euler path we need a starting node so that we traverse all the edges without repeating any edge. Aren't we just finding a path where every delivery route connects seamlessly, and you never retrace your path unnecessarily. This is essentially what Problem 2097 asks: **Can you create a valid arrangement of pairs that forms a complete delivery path?**

Here's where the **Euler graph** swoops in to save the day! 🦸‍♂️

* **Step 1:** Treat each delivery location 📍 as a vertex.
    
* **Step 2:** Treat each pair of start and end points as directed edges 🔄.
    

To solve the problem, you're really looking for an **Eulerian path**:

1. **Does the graph have exactly two nodes with odd degrees?** Those will be your starting and ending points. ( There will be as per the problem ).
    
2. If all nodes have even degrees, congrats 🎉—you can even make a full round trip (Eulerian circuit)!
    

### But how will we find odd and even degrees in directed graph ?

Eulerian **path** allows you to start and end at different vertices while still traversing every edge exactly once.

* To ensure this:
    
    * The **start vertex** should have one extra outgoing edge than incoming (you leave it first).
        
    * The **end vertex** should have one extra incoming edge than outgoing (you end there).
        
    * All other vertices must balance their in-degrees and out-degrees because you must "pass through" them without getting stuck. So can't we just say that our starting node will be that node which will have it's out degree exactly one greater than the indegree. And that's it that was the intuition we needed to solve the problem.
        

## **What** **will be the steps to solve this problem.**

1. Find the starting node
    
2. Traverse all the nodes
    

Let's understand by this drawing ( you can also say it a diagram )

In this example we can see that we are given this *\[ \[5,1\],\[4,5\],\[11,9\],\[9,4\] \]* as pairs now what a valid arrangement would be , it will be \[ \[11,9\] , \[9,4\] , \[4,5\] , \[5,1\] \] so what we can observe that we start from a node ( the main mystery is which node 🤔 ) and the next node should be the edge's next node itself and the next node should be the node connected to our current node and all this stuff needs to be done such that we can't repeat any edge.

Similiary for \[ \[1,3\] , \[3,2\] , \[2,1\] \] we will need to find a eulerian path between the nodes \[ 1,2,3 \]

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733119597470/a187bd45-5ac0-493f-8fda-c8bdef2b9831.png align="left")

Now this is euler path and also the euler circuit as the start ans end node is same , which also means that the indegrees and outdegrees of each node in this graph is same and even

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733119632488/084b8fa9-b3c2-4729-9c1b-54bb18d0555a.png align="left")

#### **So I think we now know how we can solve the problem , that is just find the euler path**

And for finding the Euler path we can use HireHolzer's algorithm

* **Construct the Graph**:
    
    * Represent the graph using adjacency lists or matrices.
        
* **Start at a Vertex**:
    
    * Begin at any vertex with an edge (for a circuit) or the start vertex (for a path).
        
* **Traverse the Graph**:
    
    * Use DFS to follow edges, removing them as you go.
        
    * If a vertex has no more edges, backtrack and add that vertex to the path.
        
* **Build the Path**:
    
    * The final sequence of vertices forms the Eulerian path or circuit.
        

And For this problem we need to do one more extra step

1\. Construct the answer from that euler-path

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733119651861/66fa40b9-8e65-476d-ba98-2d6aa1170b60.jpeg align="center")

```java

class Solution {
      private Map<Integer, List<Integer>> map = new HashMap<>();
  
    private List<Integer> eulerPath = new ArrayList<>();
    public int[][] validArrangement(int[][] pairs) {

    
        HashMap<Integer,Integer> in = new HashMap<>();
        HashMap<Integer,Integer> out = new HashMap<>();
        
for (int[] edge : pairs) {
            int u = edge[0];
            int v = edge[1];

           map.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
            out.put(u, out.getOrDefault(u, 0) + 1);
            in.put(v, in.getOrDefault(v, 0) + 1);
        }
  int ls = pairs[0][0];

   for(int key : map.keySet())
   {

    int indeg = in.getOrDefault(key,0);
    int outdeg = out.getOrDefault(key,0);
    if(outdeg-indeg ==1)
    {
        ls = key;
    }
   }
    


  dfs(ls);
	  /*
	  since we are using recursion the euler path array will be the reverse euler path hence we need to reverse it
	  */
Collections.reverse(eulerPath);
int[][] resA = new int[eulerPath.size()-1][2];
for(int i =0;i<eulerPath.size()-1;i++){
resA[i] =new int[]{ eulerPath.get(i),eulerPath.get(i+1)};
}

    return resA;
    }
     private void dfs(int node) {
        while (map.containsKey(node) && !map.get(node).isEmpty()) {
            int nextNode = map.get(node).remove(map.get(node).size() - 1); 
            dfs(nextNode);  
        }
        eulerPath.add(node);
    }
}
```

I hope I was able to explain both the concept of euler graph and the problem to you . Thanks for reading , will see you next time . Bye 🙏