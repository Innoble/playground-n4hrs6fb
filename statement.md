```C# runnable
using System;
using System.Collections.Generic;

class Maze
{
    const int WIDTH = 20;
    const int HEIGHT = 5;
    const bool SOLVE_BASIC_BFS = true;
    const bool SOLVE_FAST_BFS = true;
    const int MAZE_SEED = 0;
    // Maze generator { autofold    
    const int HEIGHT_CHARS = HEIGHT * 2 + 1;
    const int WIDTH_CHARS = WIDTH * 2 + 1;
    const int START_X = 0;
    const int START_Y = 0;
    const int GOAL_X = WIDTH - 1;
    const int GOAL_Y = HEIGHT - 1;
    static Random rnd;
    static MazeNode[] fastNodes = new MazeNode[WIDTH * HEIGHT];
    static char[,] map;

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
        map = new char[WIDTH * 2 + 1, HEIGHT * 2 + 1];

        for (int j = 0; j < HEIGHT; j++)
        {
            for (int i = 0; i < WIDTH; i++)
            {
                MazeNode node = fastNodes[i + j * WIDTH];
                map[i * 2, j * 2] = '#';
                map[i * 2 + 1, j * 2 + 1] = ' ';

                if ((node.connections & 1) > 0)
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
            map[WIDTH_CHARS - 1, j] = '#';


        for (int i = 0; i < WIDTH_CHARS - 1; i++)
            map[i, HEIGHT_CHARS - 1] = '#';

        map[START_X * 2 + 1, START_Y * 2 + 1] = 'P';
        map[GOAL_X * 2 + 1, GOAL_Y * 2 + 1] = 'X';

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
                    if (n.x + 1 < WIDTH)
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

    class Node
    {
        public Node parent = null;
        public int x = 0;
        public int y = 0;
        public int connections = 0;
        public int distance = 0;

        public void MakeChildren()
        {



        }

    }

    static Node[] nodes = new Node[1000];
    static int nodeIndex = 0;
    static Queue<Node> nodeQueue = new Queue<Node>();
    static int[] intNodes = new int[1000];

    static void Main(string[] args)
    {
        rnd = new Random(MAZE_SEED);
        Init();
        MazeNode start = fastNodes[0];
        start.parent = start;
        MazeNode last = Link(start);
        while (last != start)
            last = Link(last);
        DrawMaze();
        
        /*
        if (SOLVE_BASIC_BFS)
        {
            Node root = new Node { x = START_X, y = START_Y };
            HashSet<Node> visited = new HashSet<Node>();
            Queue<Node> queue = new Queue<Node>();

            queue.Enqueue(root);
            Node current = root;

            while (queue.Count > 0)
            {
                current = queue.Dequeue();
                visited.Add(current);
                if (current.x == GOAL_X && current.y == GOAL_Y)
                    break;

                if()
                
            }


            while (current.parent != null)
            {
                current = current.parent;
                map[current.x * 2 + 1, current.y * 2 + 1] = '.'; 
            }
        }*/
        Console.ReadLine();
    }
}

// }
```
