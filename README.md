# SQL Practice Binary Tree + Case study 1

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



# Binary Tree Structure Analysis: A Twitter Retweet Case Study

## Overview
This project applies a binary tree structure to analyze the relationships between tweets and retweets using `parent_tweet_id`. By treating the hierarchical relationship between tweets and retweets as a binary tree, we can trace how a particular tweet spreads through retweets, identify key influencers, and evaluate the depth and breadth of its impact.

### Objectives
1. **Root Identification**: Identify the root tweets that serve as starting points for retweet chains.
2. **Depth Analysis**: Determine how deeply a root tweet spreads through retweets.
3. **Key Influencers**: Identify users whose tweets act as key intermediaries, amplifying the message to a wider audience.
4. **Leaf Node Analysis**: Locate the end points of retweet chains to determine where the spread of the message stops.

---

## Dataset Structure
- **tweet_id**: Unique identifier for the tweet.
- **tweet_content**: The content of the tweet.
- **retweet_count**: Number of retweets for this tweet.
- **parent_tweet_id**: The `tweet_id` of the tweet being retweeted. `NULL` if it's an original tweet.
- **user_id**: ID of the user who posted the tweet.

---

## Analysis Steps and SQL Queries

### 1. Root Identification
Identify whether a tweet is:
- **Root**: An original tweet (parent_tweet_id is NULL).
- **Inner**: A tweet that has both parent and child tweets.
- **Leaf**: A tweet that is only a retweet and has no further retweets.

#### Query
```sql
SELECT 
    tweet_id,
    CASE 
        WHEN parent_tweet_id IS NULL THEN 'root'
        WHEN tweet_id IN (SELECT DISTINCT parent_tweet_id FROM tweets) THEN 'inner'
        ELSE 'leaf'
    END AS type
FROM tweets;
```
#### Outcome
|tweet_id |type|
|---------|----------|
|1	|root|
|2	|inner|
|3	|leaf|
|4	|inner|
|5	|leaf|
|6|	leaf|

---

### 2. Depth and Breadth of Retweets
Analyze the depth of retweet chains using recursive CTEs to track the propagation path of tweets.

#### Recursive Query for Depth and Path
```sql
WITH RECURSIVE tweet_depth AS (
    SELECT 
        tweet_id, 
        parent_tweet_id, 
        1 AS depth, 
        CAST(tweet_id AS CHAR(255)) AS path
    FROM tweets
    WHERE parent_tweet_id IS NULL

    UNION ALL

    SELECT 
        t.tweet_id, 
        t.parent_tweet_id, 
        td.depth + 1 AS depth,
        CONCAT(td.path, ' -> ', t.tweet_id) AS path
    FROM tweets t
    JOIN tweet_depth td ON t.parent_tweet_id = td.tweet_id
)
SELECT tweet_id, depth, path
FROM tweet_depth
ORDER BY depth;
```

#### Outcome
|tweet_id | depth | path|
|-------|-------|------|
|1	|1|	1|
|2	|2	|1 -> 2|
|3	|2	|1 -> 3|
|4	|3	|1 -> 2 -> 4|
|5	|3	|1 -> 2 -> 5|
|6|	4	|1 -> 2 -> 4 -> 6|
---

### 3. Maximum Retweet Depth
Determine the maximum depth of retweet chains for root tweets.
```sql
WITH RECURSIVE tweet_depth AS (
    SELECT 
        tweet_id, 
        parent_tweet_id, 
        1 AS depth
    FROM tweets
    WHERE parent_tweet_id IS NULL

    UNION ALL

    SELECT 
        t.tweet_id, 
        t.parent_tweet_id, 
        td.depth + 1 AS depth
    FROM tweets t
    JOIN tweet_depth td ON t.parent_tweet_id = td.tweet_id
)
SELECT MAX(depth) AS max_depth
FROM tweet_depth;
```
#### Outcome
|max_depth|
|---------|
|4|

---

### 4. Key Influencers
Identify users who contribute most significantly to retweet propagation by analyzing their retweet counts.

#### Query
```sql
SELECT 
    tweet_id, 
    retweet_count, 
    user_id
FROM tweets
WHERE tweet_id IN (SELECT DISTINCT parent_tweet_id FROM tweets)
ORDER BY retweet_count DESC;
```
#### Outcome
|tweet_id | retweet_count | user_id|
|--------|----------|--------|
|2	|2|	102|
|4|	1	|104|

---

### 5. Leaf Nodes
Locate the end points of retweet chains, which represent the last step of message propagation.

#### Query
```sql
SELECT tweet_id, user_id
FROM tweets
WHERE tweet_id NOT IN (SELECT DISTINCT parent_tweet_id FROM tweets);
```

#### Outcome
|tweet_id  | user_id|
|--------|----------|
|3|	103|
|5|	105|
|6|	106|
---

## Results
### 1. Root Identification
- Classifies tweets into root, inner, or leaf nodes.

### 2. Retweet Depth Analysis
- Displays the depth and paths of retweet chains.

### 3. Maximum Retweet Depth
- Determines the deepest propagation level for any root tweet.

### 4. Key Influencers
- Identifies the users driving the most retweets.

### 5. Leaf Node Analysis
- Highlights tweets that mark the end of retweet propagation.

---

## Conclusion
This case study demonstrates how binary tree structures can be applied to social media data to analyze message propagation, identify influencers, and assess the overall impact of a campaign. The approach is scalable and adaptable for various hierarchical datasets.

