---
title: "Leetcode Database Problems"
date: 2021-04-24
author: 'Chris Russell'
linktitle: Leetcode Database Problems
draft: true
slug: leetcode-sql-practice
tags:
  - leetcode
  - sql
  - learning
  - programming
---
I really need to learn SQL... Got a full day of flying ahead of me, leaving New Hampshire for Los Angeles. Lets see how many I can get done. Begin!

## 175. Combinging Two Tables

Table: Person
```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
```

Table: Address
```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.
```


Write a SQL query for a report that provides the following information for each person in the Person table, regardless if there is an address for each of those people:i 
    
    FirstName, LastName, City, State
    
## Solution

Using left join

```mysql
# Write your MySQL query statement below

SELECT FirstName, LastName, City, State 
FROM person
LEFT JOIN address
ON  address.PersonId = person.PersonId
```


