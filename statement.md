# Welcome!

This C# template lets you get started quickly with a simple one-page playground.

```C# runnable

using System;

class Program
{
    const int WIDTH = 20;
    const int HEIGHT = 5;
// { autofold    
    const int HEIGHT_CHARS = HEIGHT * 2 + 1;
    const int WIDTH_CHARS = WIDTH * 2 + 1;
    static Random rnd = new Random();
    static MazeNode[] fastNodes = new MazeNode[WIDTH * HEIGHT];

    class MazeNode
    {
        public int x = 0;
        public int y = 0;
        public int explored = 15;
        public int connections = 0;
        public MazeNode parent = null;
    }

    static void Init()
    {
        for (int i = 0; i < WIDTH; i++)
            for (int j = 0; j < HEIGHT; j++)
                fastNodes[i + j * WIDTH] = new MazeNode { x = i, y = j };
    }

    static void DrawMaze()
    {
        char[,] map = new char[WIDTH * 2 + 1, HEIGHT * 2 + 1];
            
        for (int j = 0; j < HEIGHT; j++)
        {
            for (int i = 0; i < WIDTH; i++) 
            {
                MazeNode node = fastNodes[i + j * WIDTH];
                map[i * 2, j * 2] = '#';
                map[i * 2 + 1, j * 2 + 1] = ' ';
                    
                if((node.connections & 1) > 0)
                    map[i * 2, j * 2 + 1] = ' ';
                else
                    map[i * 2, j * 2 + 1] = '#';
                    
                if ((node.connections & 8) > 0)
                    map[i * 2 + 1, j * 2] = ' ';
                else
                    map[i * 2 + 1, j * 2] = '#';
            }
        }

        for (int j = 0; j < HEIGHT_CHARS; j++)
            map[WIDTH_CHARS-1, j] = '#';
       

        for (int i = 0; i < WIDTH_CHARS-1; i++)
            map[i, HEIGHT_CHARS-1] = '#';
 

        for (int j = 0; j < HEIGHT_CHARS; j++)
        {
            string s = "";
            for (int i = 0; i < WIDTH_CHARS; i++)
            {
                s += map[i, j];
            }
            Console.WriteLine(s);
        }
    }

    static MazeNode Link(MazeNode n)
    {
        int x = 0;
        int y = 0;
        int explore;
        MazeNode dest;

        if (n == null) return null;

        while (n.explored != 0)
        {
            //Randomly pick one direction
            explore = (1 << (rnd.Next(4)));

            //If it has already been explored - try again
            if ((~n.explored & explore) > 0)
                continue;

            //Mark direction as explored
            n.explored &= ~explore;
              
            //Depending on chosen direction
            switch (explore)
            {
                case 1: //Check if it's possible to go left
                    if (n.x - 1 >= 0)
                    {
                        x = n.x - 1;
                        y = n.y;
                    }
                    else continue;
                    break;

                case 2: //Check if it's possible to go down
                    if (n.y + 1 < HEIGHT)
                    {
                        x = n.x;
                        y = n.y + 1;     
                    }
                    else continue;
                    break;

                    
                case 4: //Check if it's possible to go right	
                    if (n.x + 1  < WIDTH)
                    {
                        x = n.x + 1;
                        y = n.y;
                    }
                    else continue;
                    break;

                    
                case 8: //Check if it's possible to go up
                    if (n.y - 1 >= 0)
                    {
                        x = n.x;
                        y = n.y - 1; 
                    }
                    else continue;
                    break;
            }

            dest = fastNodes[x + y * WIDTH];
            if (dest.parent != null) continue;
            // if it already has a parent, skip it
            n.connections |= explore;
            dest.connections |= explore > 2 ? (explore >> 2) : (explore << 2);
            dest.parent = n;

            return dest;
        }
        return n.parent;
    }

    static void Main(string[] args)
    {
        Init();

        MazeNode start = fastNodes[0];
        start.parent = start;
        MazeNode last = Link(start);
        while(last != start)
            last = Link(last);
        DrawMaze();
    }
}

// }
```

