# Error : cannot resolve 'layout' element 

`````markdown
compile error :

The layout in layout has no declaration in the base `layout` folder; this can lead to crashes when the resource is queried in a configuration that does not match this qualifier.
`````

### My 3 way how : 

+ Check layout file name is not uppercase.
+ Rebuild project.
+ Erase the layout element and try inserting the layout element.  

