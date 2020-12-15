from search import *
import math
from heapq import *
class RoutingGraph(Graph):
    """Yuquan Lu"""
    
    def __init__(self, map_str):
        """Yuquan lu"""
        self.string = map_str
        
        
        
    def my_map(self):
        """initialised the map"""
        good_string = self.string.split('\n')
        for i in range(len(good_string)):
            good_string[i] = good_string[i].strip()
        the_map = [['' for j in range(len(good_string[0]))] for i in range(len(good_string)-1)]
        for i in range(len(good_string)-1):
            for j in range(len(good_string[i])):
                the_map[i][j] = good_string[i][j]
        return the_map
    
    def is_goal(self, node):
        """Returns true if the given node is a goal state, false otherwise."""
        the_map = self.my_map()
        row, col, fuel = node
        return the_map[row][col] == 'G'
        
    def starting_nodes(self):
        """Returns a sequence of starting nodes. Often there is only one
        starting node but even then the function returns a sequence
        with one element. It can be implemented as an iterator if
        needed.
        """
        the_map = self.my_map()
        result = []
        for i in range(len(the_map)):
            for j in range(len(the_map[i])):
                if the_map[i][j] == 'S':
                    result.append((i, j, math.inf))
                elif the_map[i][j].isnumeric():
                    result.append((i, j, int(the_map[i][j])))
        return result
    
    def goal_finder(self):
        """yuquan lu"""
        the_map = self.my_map()
        goal_node = []
        for row in range(len(the_map)):
            for col in range(len(the_map[row])):
                if the_map[row][col] == 'G':
                    goal_node.append((row,col))
        
        return goal_node
        
        
    def outgoing_arcs(self, tail_node):
        """Given a node it returns a sequence of arcs (Arc objects)
        which correspond to the actions that can be taken in that
        state (node)."""
        the_map = self.my_map()        
        result = []
        row, col, fuel = tail_node
        if fuel != 0 and the_map[row-1][col] in ' SFG0123456789':
 
            result.append(Arc(tail_node, (row-1, col, fuel-1), 'N', 5))
                
        if fuel != 0 and the_map[row][col+1] in ' SFG0123456789':

            result.append(Arc(tail_node, (row, col+1, fuel-1), 'E', 5))
        
        if fuel != 0 and the_map[row+1][col] in ' FSG0123456789':
            result.append(Arc(tail_node, (row+1, col, fuel-1), 'S', 5))  
        if fuel != 0 and the_map[row][col-1] in ' FSG0123456789':
            result.append(Arc(tail_node, (row, col-1, fuel-1), 'W', 5))
        if the_map[row][col] == 'F' and fuel < 9:
            fuel = 9
            result.append(Arc(tail_node, (row, col, fuel), 'Fuel up', 15))

        return result
    def estimated_cost_to_goal(self, node):
        """Return the estimated cost to a goal node from the given
        state. This function is usually implemented when there is a
        single goal state. The function is used as a heuristic in
        search. The implementation should make sure that the heuristic
        meets the required criteria for heuristics."""
        goal_list = self.goal_finder()
        node_row, node_col, fuel = node
        #if node == None:
            #return None
        #else:
            #node_row, node_col, fuel = node
        result_list = []
        for aaa in goal_list:
            goal_row, goal_col = aaa
            result = 5 * (abs(goal_row-node_row) + abs(goal_col - node_col))
            result_list.append(result)
        return result_list
    

class AStarFrontier(Frontier):
    """yuquan lu"""
    def __init__(self, graph):
        """yuquan lu"""
        self.container = []
        self.repead = []
        self.graph = graph
        self.count = 0
    def add(self, path):
        """Adds a new path to the frontier. A path is a sequence (tuple) of
        Arc objects. You should override this method.

        """
        cost = 0
        total_cost_list = []
        for pat in path:
            cost += pat.cost
        if isinstance(self.graph.estimated_cost_to_goal(path[-1].head),int) == False:
            for i in self.graph.estimated_cost_to_goal(path[-1].head):
                
                estimated_cost = i
                totalcost = cost + estimated_cost
                total_cost_list.append(totalcost)
        else:
            estimated_cost = self.graph.estimated_cost_to_goal(path[-1].head)
            totalcost = cost + estimated_cost
            total_cost_list.append(totalcost)            
        self.count += 1
        
        for i in total_cost_list:
            if path[-1].head not in self.repead:
                heappush(self.container, (i, self.count, path))
            else:         
                return None
        
        
    def __iter__(self):
        """We don't need a separate iterator object. Just return self. You
        don't need to change this method."""
        return self

    
    def __next__(self):

        """Selects, removes, and returns a path on the frontier if there is
        any.Recall that a path is a sequence (tuple) of Arc
        objects. Override this method to achieve a desired search
        strategy. If there nothing to return this should raise a
        StopIteration exception.

        """
        if len(self.container) > 0:
            aaa = heappop(self.container)[-1]
            self.repead.append(aaa[-1].head)
            return aaa# FIX THIS return something instead

        else:
            raise StopIteration   # don't change this one          
def print_map(mapp, frontier, solution):
    """yuquan lu"""
    
    my_map = mapp.my_map()
    
    
    if solution != None:
        for step in solution[:-1]:
            row, col, fuel = step.head
            if my_map[row][col] == ' ':
                my_map[row][col] = "*"
    
    for row, col, fuel in frontier.repead:
        if my_map[row][col] == ' ':     
            my_map[row][col] = "."
    
    result = [[''] for i in range(len(my_map))]
    for i in range(len(my_map)):
        for j in range(len(my_map[i])):
            result[i][0] += my_map[i][j]

    for i in range(len(result)):
        print(result[i][0])
    

    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
def main():
    map_str = """\
    +---------+
    |         |
    |    G    |
    |         |
    +---------+
    """
    
    map_graph = RoutingGraph(map_str)
    frontier = AStarFrontier(map_graph)
    solution = next(generic_search(map_graph, frontier), None)
    print_map(map_graph, frontier, solution)
if __name__ == "__main__":
    main()
