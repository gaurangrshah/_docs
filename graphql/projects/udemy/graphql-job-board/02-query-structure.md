---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: Nested Objects in GraphQL
details:
  section: 2
  lesson: 13
---



## GraphQL Query Structure

One of the biggest differences between graphql and other data querying methods such as REST, is that with graphql you must query for the exact data that you require. Where as with methods like a REST api, you essentially get back the whole data structure and have to usually parse thru the data to find what you're looking for. Generally this practice leads to large server calls, and has the added issue of having to make those large calls multiple times depending on the application. 

![image-20201015232215235](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr0dlzcdjj30jd0bk0tj.jpg)

So graphql solves these to major problems for querying apis. With graphql you can query for the exact data-structure you expect, this prevents "over-fetching" and can easily make several server requests fairly quickly when necessary.

> [GraphQL vs REST](https://www.toptal.com/api-development/graphql-vs-rest-tutorial)



<u>Graphql Query Syntax</u>:

![image-20201015232406984](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr0fj4xwbj30m80bct9q.jpg)



> [The Anatomy of a GraphQL Query](https://www.apollographql.com/blog/the-anatomy-of-a-graphql-query-6dffa9e9e747/)
>
> ![image-20201015233050614](https://tva1.sinaimg.cn/large/007S8ZIlly1gjr0mjobz0j30sg0ak0ty.jpg)



