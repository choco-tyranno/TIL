<h1>LinearLayoutManager</h1>

> LayoutPosition

> #getChildAt(int layoutPosition)

`````java
LinearLayoutManager layoutManager;
int position;
...
//position is layoutPosition. not adapterPosition.
//layoutPosition : position between currently displaying items.
layoutManager.getChildAt(position);
`````

