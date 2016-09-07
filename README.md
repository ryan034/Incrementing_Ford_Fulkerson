Specialized max flow problem involving an bipartite graph. Finds the minimum capacity of the edges leading from the source needed to achieve maximum flow between the two independent sets. Has O(logV*VE^2) complexity.

The example application below involves the senario between donors and celebrities.

import math

class Edge(object):
    def __init__(self, u, v, w):
        self.source = u
        self.sink = v  
        self.capacity = w
    
    def __repr__(self):
        return "%s->%s:%s" % (self.source, self.sink, self.capacity)
    
    def increment_cap(self,cap):
        self.capacity = cap

class FlowNetwork(object):
    
    def __init__(self):
        self.adj = {}
        self.flow = {}
 
    def add_vertex(self, vertex):
        self.adj[vertex] = []
 
    def get_edges(self, v):
        return self.adj[v]
 
    def add_edge(self, u, v, w=0):
        if u == v:
            raise ValueError("u == v")
        edge = Edge(u,v,w)
        redge = Edge(v,u,0)
        edge.redge = redge
        redge.redge = edge
        self.adj[u].append(edge)
        self.adj[v].append(redge)
        self.flow[edge] = 0
        self.flow[redge] = 0
    
    def clear_flow(self):
        for edge_flow in self.flow.keys():
            self.flow[edge_flow] = 0

    def bfs(self, source, sink, path, sum):
        if source == sink:
            for edge in path:
                self.flow[edge] += 1
                self.flow[edge.redge] -= 1
            return 1     
        for edge in self.get_edges(source):
            if path !=[] and (path[-1].capacity - self.flow[path[-1]]) <= 0:
                break			
            elif edge.capacity - self.flow[edge]> 0 and edge not in path:
                sum += self.bfs(edge.sink, sink, path + [edge], 0)
        return sum

    
    
def min_capacity_max_flow(donor_dict, celeb_list):
    
    donors_playing = len(donor_dict)
    lower_bound = math.ceil(donors_playing/len(celeb_list))
    
    g = FlowNetwork()
    g.add_vertex('s')
    g.add_vertex('t')
    
    for celeb in celeb_list:
        
        g.add_vertex(celeb)
        g.add_edge('s', celeb, lower_bound)
            
    for donor, celeb_pref in donor_dict.items():
        
        g.add_vertex(donor)
        g.add_edge(donor, 't', 1)
        
        for celeb in celeb_pref:
            
            g.add_edge(celeb, donor, 1)          
        
    lower = lower_bound
    upper = donors_playing
    mid_flow = 0
    
    while upper - lower > 1 :    
        
        mid = math.ceil((lower + upper)/2)
        
        for edge in g.get_edges('s'):

            edge.increment_cap(mid)

        mid_flow += g.bfs('s', 't', [], 0)
        
        if mid_flow < donors_playing:
            
            lower = mid
            
        else:    
       
            upper = mid
            g.clear_flow()
            mid_flow = 0
    
    return upper
    
    
