# Refactoring

## GameOfLife

From Chanathip Thumkanon's Game of Life code: https://github.com/OOP2020/pa4-kaesrel.git

### generateNext() Method

In the `src/LifeAlgo.java` class 

https://github.com/OOP2020/pa4-kaesrel/blob/master/src/LifeAlgo.java

consider this code:

```java
	/**
     * Generate next generation of live cells.
     */
    public synchronized void generateNext()
    {
        nextAlive = new CellSet();
        nextDead = new CellSet();

        for(Cell cell : aliveCells)
        {
            int xbeg = (cell.x == 0)? 0 : (cell.x - 1);
            int xend = (cell.x+2 > COLS) ? (cell.x+1) : (cell.x + 2);
            int ybeg = (cell.y == 0)? 0 : (cell.y - 1);
            int yend = (cell.y+2 > ROWS) ? (cell.y+1) : (cell.y + 2);

            for(int x = xbeg; x < xend; x++)
                for(int y = ybeg; y < yend; y++)
                {
                    Cell newCell = new Cell(x,y);
                    if(nextAlive.contains(newCell) || nextDead.contains(newCell))
                        continue;

                    int neighbor = countNeighbor(newCell);
                    if(neighbor == 3 || (neighbor == 2 && aliveCells.contains(newCell)))
                        nextAlive.add(newCell);
                    else
                        nextDead.add(newCell);
                }
        }
		...
    }
```

* Refactoring Signs: 
  - three "for" loops are nested
  - too many ternary operators are used

* Refactoring Techniques: 

Extract Method - break down the complicated nested method.
Rename variable name "neighbor" to "neighborCount" to distinguish between neighbor "Cell" and "Count". 


```java
    /**
     * Generate a list of neighbor cells.
     */
    private CellSet generateNeighborCells(Cell cell)
    {
        int xbeg = (cell.x == 0)? 0 : (cell.x - 1);
        int xend = (cell.x+2 > COLS) ? (cell.x+1) : (cell.x + 2);
        int ybeg = (cell.y == 0)? 0 : (cell.y - 1);
        int yend = (cell.y+2 > ROWS) ? (cell.y+1) : (cell.y + 2);
        CellSet neighbors = new CellSet();
        for(int x = xbeg; x < xend; x++)
            for(int y = ybeg; y < yend; y++)
            {
                Cell newCell = new Cell(x,y);
                neighbors.add(newCell);
            }
        return neighbors;
    }
    
    /**
     * Generate next generation of live cells.
     */
    public synchronized void generateNext()
    {
        nextAlive = new CellSet();
        nextDead = new CellSet();

        for(Cell cell : aliveCells)
        {
            CellSet neighbors = generateNeighborCells(cell);
            

            for (Cell newCell : neighbors)
            {
                if(nextAlive.contains(newCell) || nextDead.contains(newCell))
                    continue;

                int neighborCount = countNeighbor(newCell);
                if(neighborCount == 3 || (neighborCount == 2 && aliveCells.contains(newCell)))
                    nextAlive.add(newCell);
                else
                    nextDead.add(newCell);
            }

        }

        aliveCells = nextAlive;
        nextAlive = nextDead = null;
    }
```

Revisiting the refactored "generateNeighborCells" to cope with excessive uses of ternary operators.

from:

```java
    private CellSet generateNeighborCells(Cell cell)
    {
        int xbeg = (cell.x == 0)? 0 : (cell.x - 1);
        int xend = (cell.x+2 > COLS) ? (cell.x+1) : (cell.x + 2);
        int ybeg = (cell.y == 0)? 0 : (cell.y - 1);
        int yend = (cell.y+2 > ROWS) ? (cell.y+1) : (cell.y + 2);
		...
    }
```


* Refactoring:

Substitute Algorithm - replace ternary operators to comparators in the loop.


```java
    private CellSet generateNeighborCells(Cell cell)
    {
        int xbeg = cell.x - 1;
        int xend = cell.x + 2;
        int ybeg = cell.y - 1;
        int yend = cell.y + 2;
        CellSet neighbors = new CellSet();
        for(int x = xbeg; x < xend; x++)
        {
            if (x < 0 || x >= COLS)
                continue;
            for(int y = ybeg; y < yend; y++)
            {
                if (y < 0 || y >= ROWS)
                    continue;
                Cell newCell = new Cell(x,y);
                neighbors.add(newCell);
            }
        }
        return neighbors;
    }
```

### Extra Notes: 
The readability in the refactored version increases; however, there is a speed tradeoff.
I compare both versions' speed by running two of them side by side.
The refactored version has a headstart before the older version for a few seconds (about 500 generations in advance).
The older version's catches up at about the 6000th generation.

