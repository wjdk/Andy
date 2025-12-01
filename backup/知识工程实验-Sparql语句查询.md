### 实验介绍

编写SPARQL查询语句并检验效果。
[项目地址](https://github.com/eclipse-rdf4j/rdf4j/tree/main/examples/src/main/java/org/eclipse/rdf4j/examples/repository)

修改**Example15SimpleSPARQLQuery.java**

### 实验要求
针对下列自然语言，使用SPARQL查询语句返回结果: 
1. List the creators (including paintings) of Guernica and Sunflowers, respectively. 
2. List all the artists (including living places) who live in Spain or other places. 
3. List all paintings, their names, and the corresponding techniques

### Sparql语句

1. 
```Sparql
SELECT ?artist ?painting
WHERE {
    ?painting rdfs:label ?paintingLabel .
    FILTER(?paintingLabel IN ("Guernica", "Sunflowers"))
    
    ?artist a ex:Artist ;
    ex:creatorOf ?painting .
}
```
2. 
```Sparql
SELECT ?artist ?country
WHERE {    
    ?artist a ex:Artist ;
    ex:homeAddress ?address .
    ??address ex:country ?country .
}
```
3. 
```Sparql
SELECT ?painting ?name ?techniche
WHERE {    
    ?painting a ex:Painting ;
    rdfs:label ?name;
    ex:technique ?techniche.
}
```

### 代码和运行结果
1. 
![alt text](image-4.png)
![alt text](image-1.png)
![alt text](image-2.png) 
2.  
![alt text](image-5.png)
![alt text](image-6.png)
![alt text](image-7.png)
3.  
![alt text](image-8.png)
![alt text](image-9.png)
![alt text](image-10.png)
### 实验总结

?变量名表示需要在匹配过程中绑定的变量；
多个三元组模式并列时，所有模式必须同时匹配；
同一查询可能产生多个绑定集合（即多行结果）。
FILTER：对结果进行约束（如数值比较、字符串匹配）

### 扩展思考

1. 
   也可以用Union实现

```Sparql
SELECT ?artist ?painting
WHERE {
     {?painting rdfs:label "Guernica" . } UNION {?painting rdfs:label "Sunflowers" . }
    
    ?artist a ex:Artist ;
    ex:creatorOf ?painting .
}
```