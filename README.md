# Network-Design-optimization-
Prescriptive analysis - Optimization problem
"""
Created on Wed May  2 01:53:49 2018

@author: Sri Varsha Bandi
"""

import os
              
from gurobipy import*
#import scipy 
#import numpy
os.chdir('C:\\Project_247\\NetworkDesign\\dc20_cust2000_inst1')
os.getcwd()
import csv
dc_cap=[]
with open('dc_cap.txt', 'r') as csvfile:
    spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
    for row in spamreader:
        dc_cap.append(int(''.join(row)))
        
print(dc_cap)


dc_cost=[]
with open('dc_cost.txt', 'r') as csvfile:
    spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
    for row in spamreader: 
        dc_cost.append(int(''.join(row)))
        
print(dc_cost)


demand=[]
with open('demand.txt', 'r') as csvfile:
    spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
    for row in spamreader:
        demand.append(int(''.join(row)))
        
print(demand)

unit_tran_cost=[]
with open('unit_tran_cost.txt', 'r') as csvfile:
    spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
    for row in spamreader:
        print(row)
        unit_tran_cost.append(list(map(lambda x: int(x),''.join(row).split(','))))
        
print(unit_tran_cost[1][1])

set_no_dc=range(len(dc_cap))
set_no_demand=range(len(demand))
set_no_dccost=range(len(dc_cost))

print(set_no_dc)
print(set_no_demand)

## create a new model
m = Model("mymodel1")
##add variables
x = {}
y = {}  
for i in (set_no_dc) :
  for j in (set_no_demand) :
      x[(i,j)] = m.addVar(vtype=GRB.CONTINUOUS, name="x%d%d" % (i,j))
for j in set_no_demand:
    unit_tran_cost.append([])
    for i in set_no_dc:
        unit_tran_cost[j].append(m.addVar(obj=unit_tran_cost[j][i],
                      name="x[%d,%d]" % (j, i)))
for k in set_no_dccost:
    y[k] = m.addVar(vtype=GRB.BINARY, name ="y%d" % (k)) 
      
m.update()

# Objective function
m.setObjective((quicksum(dc_cost[k]*y[k] for k in set_no_dccost)) + (quicksum(quicksum(unit_tran_cost[j][i]*x[(i,j)] for j in set_no_demand) for i in set_no_dc)))

m.optimize

# Add constraints
for i in set_no_dc:
    m.addConstr(quicksum(x[(i,j)] for j in set_no_demand) <= dc_cap[i], GRB.EQUAL, 0)
    
for j in set_no_demand:
    m.addConstr(quicksum(x[i,j] for i in set_no_dc) == demand[j], GRB.EQUAL, 0) 
    

m.update()
m.optimize()
m.write("mymodel1.lp")
# optimal decision variable values
for v in m.getVars():
    if v.x >0:
        print (v.varName,v.x)
