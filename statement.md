```C# runnable
using System;
using System.Collections.Generic;
using System.Diagnostics;

class Maze
{
    const int WIDTH = 20;
    const int HEIGHT = 5;
    const bool SOLVE_BASIC_BFS = true;
    const bool SOLVE_FAST_BFS = true;
    const int MAZE_SEED = 0;
    const int ITERATIONS = 10000;
    // Maze generator { autofold    
    const int HEIGHT_CHARS = HEIGHT * 2 + 1;
    const int WIDTH_CHARS = WIDTH * 2 + 1;
    const int START_X = 0;
    const int START_Y = 0;
    const int GOAL_X = WIDTH - 1;
    const int GOAL_Y = HEIGHT - 1;
    static Random rnd;
    static readonly MazeNode[] mapNodes = new MazeNode[WIDTH * HEIGHT];
    static readonly char[,] map = new char[WIDTH * 2 + 1, HEIGHT * 2 + 1];
    static readonly Stopwatch watch = new Stopwatch();

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
                mapNodes[i + j * WIDTH] = new MazeNode { x = i, y = j };
    }

    static void CreateMaze()
    {
        for (int j = 0; j < HEIGHT; j++)
        {
            for (int i = 0; i < WIDTH; i++)
            {
                MazeNode node = mapNodes[i + j * WIDTH];
                map[i * 2, j * 2] = '\u2588';
                map[i * 2 + 1, j * 2 + 1] = ' ';

                if ((node.connections & 1) > 0)
                    map[i * 2, j * 2 + 1] = ' ';
                else
                    map[i * 2, j * 2 + 1] = '\u2588';

                if ((node.connections & 8) > 0)
                    map[i * 2 + 1, j * 2] = ' ';
                else
                    map[i * 2 + 1, j * 2] = '\u2588';
            }
        }

        for (int j = 0; j < HEIGHT_CHARS; j++)
            map[WIDTH_CHARS - 1, j] = '\u2588';


        for (int i = 0; i < WIDTH_CHARS - 1; i++)
            map[i, HEIGHT_CHARS - 1] = '\u2588';

        map[START_X * 2 + 1, START_Y * 2 + 1] = 'P';
        map[GOAL_X * 2 + 1, GOAL_Y * 2 + 1] = 'X';
    }

    static void DrawMazePath()
    {
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

            dest = mapNodes[x + y * WIDTH];
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

        public void AddChildren()
        {
            int index = x + WIDTH * (y - 1);
            if (y > 0 && (connections & 8) > 0 && !visited.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 2) > 0)
                {
                    queue.Enqueue(new Node { x = x, y = y - 1, connections = nConnections, distance = distance + 1, parent = this });
                    visited.Add(index);
                }
            }

            index = x + WIDTH * (y + 1);
            if (y < HEIGHT -1 && (connections & 2) > 0 && !visited.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 8) > 0)
                {
                    queue.Enqueue(new Node { x = x, y = y + 1, connections = nConnections, distance = distance + 1, parent = this });
                    visited.Add(index);
                }
            }

            index = x -1 + WIDTH * y;
            if (x > 0 && (connections & 1) > 0 && !visited.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 4) > 0)
                {
                    queue.Enqueue(new Node { x = x - 1, y = y, connections = nConnections, distance = distance + 1, parent = this });
                    visited.Add(index);
                }
            }

            index = x + 1 + WIDTH * y;
            if (x < WIDTH - 1 && (connections & 4) > 0 && !visited.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 1) > 0)
                {
                    queue.Enqueue(new Node { x = x + 1, y = y, connections = nConnections, distance = distance + 1, parent = this });
                    visited.Add(index);
                }
            }
        }
    }

    static readonly Node[] nodes = new Node[1000];  
    static Queue<Node> queue;
    static HashSet<int> visited;
    static readonly int[] intNodes = new int[1000];
    static readonly int nodeIndex = 0;

    static void Main(string[] args)
    {
        rnd = new Random(MAZE_SEED);
        Init();
        MazeNode start = mapNodes[0];
        start.parent = start;
        MazeNode last = Link(start);
        while (last != start)
            last = Link(last);

        CreateMaze();

        watch.Reset();
        watch.Start();
        if (SOLVE_BASIC_BFS)
        {
            Node current = null;
            for (int i = 0; i < ITERATIONS; i++)
            {
            
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                current = new Node { x = START_X, y = START_Y, distance = 0, connections = rootConnections }; // root

                visited = new HashSet<int> { startIndex };
                queue = new Queue<Node>(1000);
                queue.Enqueue(current);

                while (queue.Count > 0)
                {
                    current = queue.Dequeue();
                    if (current.x == GOAL_X && current.y == GOAL_Y)
                        break;

                    current.AddChildren();
                } 
            }

            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine(time);

            while (current.parent != null)
            {
                current = current.parent;
                if (current.parent != null)
                    map[current.x * 2 + 1, current.y * 2 + 1] = '\u2219';
            }

            DrawMazePath();
        }

        
        Console.ReadLine();
    }
}

// }
```
