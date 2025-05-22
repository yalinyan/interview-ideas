1. Design a system to support shopping system, huge mount of products info, what kind of db and how to handle huge traffic 

2. Coding: read csv file, in each line, the first element is the leaf id, the second element is the leaf name, following will be the path from root element(id & name) to parent of the leaf

    Leaf-ID,Leaf-Name,Level-1,Level-Name,Level-2,Level-Name,Level-3,Level-Name,....
    1051,LN151,1011,RN111,1021,LN121,1031,LN131,1041,LN141,
    1022,LN122,1011,RN111
    2031,LN231,2011,RN211,2021,LN221
    3071,LN371,3011,RN311,3021,LN321,3031,LN331,3041,LN341,3051,LN351,3061,LN361