# SQL_Practice_Binary_Tree

# SQL Learning Notes

## Goals
This repository focuses on practicing SQL queries involving hierarchical data, specifically working with a binary tree structure. The goal is to practice categorizing nodes based on their relationship within the tree and gain a deeper understanding of SQL operations.

## Problem (Adapted from HackerRank)
You are given a table, binary_tree, containing two columns: N and P, where:  

-N represents the value of a node in a Binary Tree.  
-P represents the parent of node N.

The task is to write a query to determine the type of each node in the Binary Tree, ordered by the value of the node. The possible node types are:  

-Root: If the node is the root node (it has no parent).  
-Leaf: If the node is a leaf node (it has no children).  
-Inner: If the node is neither root nor leaf (it has children).

## Database
-Table: binary_tree 

|N |    P|
|---|-----|
|1|	NULL|
|2|	1|
|3|	1|
|4|	2|
|5|	2|
|6|	3|
|7|	3|
|8|	4|
|9|	4|

```sql
select 
 	distinct n,
 	case when p is null then 'root'
 		 when n in (select distinct p from binary_tree) then 'Inner'
 		 else 'leaf'
 	end as type
 from binary_tree;
```


## Query Results Table


|N  | Type|
|---|-----|
|1|	root|
|2|	Inner|
|3|	Inner|
|4|	Inner|
|5|	leaf|
|6|	leaf|
|7|	leaf|
|8|	leaf|
|9|	leaf|



## Explanation
-Root: The root node is identified by having a NULL value in the P column (no parent).  
-Inner: An inner node is one that appears as a parent (i.e., its value appears in the P column for another node).  
-Leaf: A leaf node is one that does not appear as a parent, meaning its value does not appear in the P column.
