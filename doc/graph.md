# Graph
There are two kinds of graphs, directed and undirected. Since we are specifying
the directions of edges (i.e. inward and outward,) we are talking about
directed graphs here. But we can always generalize the data structure to be
undirected by making sure the edges go both ways. However, topological sort
requires directed acyclic graphs.

Two vertices are *connected* if there is a path between them. The edge typically
carries information of the cost or distance of the path.

## Graph API
* `#initialize(num_of_vertices)`
* `#addEdge` - add connection between any two vertices
* `#eachVertex` - takes in a block, iterate through all vertices

## Implementation
``` ruby
class Vertex
  attr_accessor :in_edges, :out_edges, :value

  def initialize(value)
    @value = value
    @in_edges = []
    @out_edges = []
  end

  def delete_in_edge(edge)
    @in_edges.each_index do |idx|
      if @in_edges[idx] == edge
        @in_edges.delete_at(idx)
      end
    end
  end

  def delete_out_edge(edge)
    @out_edges.each_index do |idx|
      if @out_edges[idx] == edge
        @out_edges.delete_at(idx)
      end
    end
  end
end

class Edge
  attr_reader :source, :destination, :cost
  def initialize(source, destination, cost = 1)
    @source = source
    @destination = destination
    @cost = cost
    @source.out_edges << self
    @destination.in_edges << self
  end

  def destroy!
    @source.delete_out_edge(self)
    @destination.delete_in_edge(self)
    @source = nil
    @destination = nil
  end
end
```
## Graph Representation
### Adjacency List
For each vertex i, store an array of adjacent vertices to it. We typically
have an array of adjacency lists; each vertex gets a list (of neighbors.) We can also use a hash.

Example:

![graph][graph]
[graph]:../img/graph.png

```
i
0 - Calvin => [Matt, Steven]
1 - Matt => [Calvin, Steven]
2 - Steven => [Calvin, Matt, Loki]
3 - Loki => [Steven]
```

Example of Java implementation
``` java
public class Graph
{
  private final int V;
  private Bag<Integer>[] adjList;

  public Graph(int V)
  {
    this.V = V;
    adjList = (Bag<Integer>[]) new Bag[V];
    for (int v = 0; v < V; v++) {
      adjList[v] = new Bag<Integer>();
    }
  }

  public void addEdge(int v, int w)
  {
    adjList[v].add(w);
    adjList[w].add(v);
  }
}
```

### Adjacency Matrix
We can also use a matrix to represent the neighboring relationship above.
The matrix should be symmetric for undirected graphs.
```
matrix =
_ 0 1 2 3  
0 * 1 1 0
1 1 * 1 0
2 1 1 * 1
3 0 0 1 *

matrix[0][1] => connect from 0 to 1, in this case it's Calvin to Matt.
If such connection exists, set matrix[0][1] = 1.

matrix[0][0] is self connection, just give it any symbol; I used *
```

### Density
Density: 2|E|/(|V| |V - 1|)

If density is close to 1, it's a dense graph. Use adjacency matrix for dense graphs and use adjacency list for sparse graphs.

## Connected Component
Vertices v and w are connected if there is a path between them.

### Properties
The relation "is connected to" is an equivalence relation:
* Reflexive: *v* is connected to *v*
* Symmetric: if *v* is connected to *w*, then *w* is connected to *v*
* Transitive: if *v* connected to *w* and *w* connected to *x*, then *v* is
connected to *x*


A *connected component* is a maximal set of connected vertices

The goal is to partition a set of vertices into connected component, we
will achieve this in linear time by iterating through all the vertices
contained in a graph.

### Approach
1. Initialize all vertices as unmarked
2. For each unmarked vertex *v*, run DFS to identity all vertices discovered as
part of the same component
3. Create a hash map, using vertex as the key, and assign an ID number as value,
then mark the vertex as visited
4. ID number begins at 1, it should stay at 1 for the whole DFS search.
5. Upon completion of the first DFS search, ID should increment.
6. Iterate to next vertex, if it's visited, skip. Once we find an unvisited
node, we fire DFS again but now ID = 2.

``` ruby
class ConnectedComponent
  def initialize(graph)
    @components = Hash.new
    @graph = graph
    mapConnectedComponents
  end

  def mapConnectedComponents
    id = 1
    @graph.vertices.each do |vertex|
      unless vertex.visited?
        dfs(vertex, id)
        id += 1
      end
    end
    @count = id
  end

  def count
    @count
  end

  def dfs(vertex, id)
    # should be something like...
    @components[vertex] = id
    vertex.visit!
    vertex.out_edges.each do |edge|
      dfs(edge.to_vertex, id)
    end
  end
end
```

## Interivew Problems
__Route Between Nodes__(CTCI): Given a directed graph, design an algorithm to find out
whether there is a route between two nodes
``` ruby
def is_there_route?(source, dest)
  dfs(source, dest) == dest
end

def dfs(node, target)
  return nil if node.nil? || node.visit?
  node.visit!
  return node if node == target
  node.out_edges.each do |out_edge|
    search_result = dfs(out_edge.to_vertex, target)
    return search_result unless search_result.nil?
  end
  nil
end
```

__Deep Copy of a Graph__: Given a node, and this node contains other nodes,
they together form a graph. Write a function to deep copy this node and
everything inside it.
```
Example 1
A contains B and C. B & C contain nothing
A.nodes = [B, C]
B.nodes = []
C.nodes = []

Example 2
A contains itself
A.nodes = [A]

Example 3
Cyclic Graph
A.nodes = [B]
B.nodes = [C]
C.nodes = [A]

Example 4 => What is the time complexity of your algorithm, given this example?
D.nodes = [E, F, D, E, E, E, E, F]
E.nodes = [G, H]
F.nodes = []
G.nodes = []
H.nodes = []
```

``` ruby
# N = number of nodes
# M = sum of the sizes of all node lists
# Time: O(N + M)
# Space: O(N) + O(M)
class Node
  def initialize
    @nodes = []
  end

  def deepCopy(dup_record = Hash.new)
    new_node = Node.new
    dup_record[self] = new_node
    self.nodes.each do |node|
      # It's already been copied, so use the copied version
      if dup_record[node]
        new_node.nodes << dup_record[node]
      else
        # It hasn't been copied, so make a copy
        new_node.nodes << node.deepCopy(dup_record)
      end
    end
    new_node
  end
end
```
