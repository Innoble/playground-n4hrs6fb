# Optimizing breadth first search

During the last contest (X-mas Rush) a good pathfinder was very important. The map was small, with very short paths so the only thing that made sense was a BFS. There are many ways to do a BFS and there are big differences in performance. I will explain a few examples in order of performance. If you want to skip straight ahead to to the benchmark, run the code below. You can take the code and use it however you want. Just remember, it contains a lot of code to make the comparisons properly. Make sure to clean out the useless bits.
    Dont make the width larger, because some parts depend on the width fitting inside a 32 bit integer. I do this because most gameboards
    in CG games are smaller than this. If you encounter a wider board... be creative!

I will explain the various versions in the next pages.

```C# runnable
using System;
using System.Collections.Generic;
using System.Diagnostics;

class Maze
{
    const int WIDTH = 30;
    const int HEIGHT = 5;
    const bool BFS_BASIC = true;
    const bool BFS_NODEARRAY = true;
    const bool BFS_NOHASH = true;
    const bool BFS_NOQUEUE = true;
    const bool BFS_NOCLASS = true;
    const int MAZE_SEED = 0;
    const int ITERATIONS = 100000;
    // Maze generator { autofold    
    const int HEIGHT_CHARS = HEIGHT * 2 + 1;
    const int WIDTH_CHARS = WIDTH * 2 + 1;
    const int START_X = 0;
    const int START_Y = 0;
    const int GOAL_X = WIDTH - 1;
    const int GOAL_Y = HEIGHT - 1;
    const char ORIGIN = 'P';
    const char GOAL = 'X';
    const char WALL = '#';
    const char PATH = '.';
    const char EMPTY = ' ';
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
                map[i * 2, j * 2] = WALL;
                map[i * 2 + 1, j * 2 + 1] = EMPTY;

                if ((node.connections & 1) > 0)
                    map[i * 2, j * 2 + 1] = EMPTY;
                else
                    map[i * 2, j * 2 + 1] = WALL;

                if ((node.connections & 8) > 0)
                    map[i * 2 + 1, j * 2] = EMPTY;
                else
                    map[i * 2 + 1, j * 2] = WALL;
            }
        }

        for (int j = 0; j < HEIGHT_CHARS; j++)
            map[WIDTH_CHARS - 1, j] = WALL;


        for (int i = 0; i < WIDTH_CHARS - 1; i++)
            map[i, HEIGHT_CHARS - 1] = WALL;

        map[START_X * 2 + 1, START_Y * 2 + 1] = ORIGIN;
        map[GOAL_X * 2 + 1, GOAL_Y * 2 + 1] = GOAL;
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
    // }

    // Other code { autofold
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


    class Node
    {
        public Node parent = null;
        public int x = 0;
        public int y = 0;
        public int connections = 0;
        public int distance = 0;

        public void SetNode(int _x, int _y, int _distance, int _connections, Node _parent)
        {
            x = _x;
            y = _y;
            distance = _distance;
            connections = _connections;
            parent = _parent;

        }

        public Node(int _x, int _y, int _distance, int _connections, Node _parent)
        {
            x = _x;
            y = _y;
            distance = _distance;
            connections = _connections;
            parent = _parent;
        }

        public Node() { }

        public void SetChildrenNoQueue()
        {
            
            if (y > 0 && (connections & 8) > 0 && IsAvailable(x, y - 1))
            {
                int index = x + WIDTH * (y - 1);
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 2) > 0)
                {
                    Node child = nodes[queueCount++];
                    child.SetNode(x, y - 1, distance + 1, nConnections, this);
                    SetVisited(x, y - 1);
                }
            }

           
            if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailable(x, y + 1))
            {
                int index = x + WIDTH * (y + 1);
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 8) > 0)
                {
                    Node child = nodes[queueCount++];
                    child.SetNode(x, y + 1, distance + 1, nConnections, this);
                    SetVisited(x, y + 1);
                }
            }

            
            if (x > 0 && (connections & 1) > 0 && IsAvailable(x - 1, y))
            {
                int index = x - 1 + WIDTH * y;
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 4) > 0)
                {
                    Node child = nodes[queueCount++];
                    child.SetNode(x - 1, y, distance + 1, nConnections, this);
                    SetVisited(x - 1, y);
                }
            }

            
            if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailable(x + 1, y))
            {
                int index = x + 1 + WIDTH * y;
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 1) > 0)
                {
                    Node child = nodes[queueCount++];
                    child.SetNode(x + 1, y, distance + 1, nConnections, this);
                    SetVisited(x + 1, y);
                }
            }
        }

        public void SetChildrenNoHash()
        {
            if (y > 0 && (connections & 8) > 0 && IsAvailable(x, y-1))
            {
                int index = x + WIDTH * (y - 1);
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 2) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x, y - 1, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    SetVisited(x, y - 1);
                }
            }

            if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailable(x, y + 1))
            {
                int index = x + WIDTH * (y + 1);
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 8) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x, y + 1, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    SetVisited(x, y + 1);
                }
            }

            
            if (x > 0 && (connections & 1) > 0 && IsAvailable(x - 1, y))
            {
                int index = x - 1 + WIDTH * y;
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 4) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x - 1, y, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    SetVisited(x - 1, y);
                }
            }

            
            if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailable(x + 1, y))
            {
                int index = x + 1 + WIDTH * y;
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 1) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x + 1, y, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    SetVisited(x + 1, y);
                }
            }
        }

        public void SetChildren()
        {
            int index = x + WIDTH * (y - 1);
            if (y > 0 && (connections & 8) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 2) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x, y - 1, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    visitedHash.Add(index);
                }
            }

            index = x + WIDTH * (y + 1);
            if (y < HEIGHT - 1 && (connections & 2) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 8) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x, y + 1, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    visitedHash.Add(index);
                }
            }

            index = x - 1 + WIDTH * y;
            if (x > 0 && (connections & 1) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 4) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x-1, y, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    visitedHash.Add(index);
                }
            }

            index = x + 1 + WIDTH * y;
            if (x < WIDTH - 1 && (connections & 4) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 1) > 0)
                {
                    Node child = nodes[nodeIndex++];
                    child.SetNode(x+1, y, distance + 1, nConnections, this);
                    queue.Enqueue(child);
                    visitedHash.Add(index);
                }
            }
        }

        public void AddChildren()
        { 
            int index = x + WIDTH * (y - 1);
            if (y > 0 && (connections & 8) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 2) > 0)
                {
                    queue.Enqueue(new Node(x, y - 1, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x + WIDTH * (y + 1);
            if (y < HEIGHT -1 && (connections & 2) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 8) > 0)
                {
                    queue.Enqueue(new Node(x, y + 1, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x -1 + WIDTH * y;
            if (x > 0 && (connections & 1) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 4) > 0)
                {
                    queue.Enqueue(new Node(x - 1, y, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x + 1 + WIDTH * y;
            if (x < WIDTH - 1 && (connections & 4) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapNodes[index].connections;
                if ((nConnections & 1) > 0)
                {
                    queue.Enqueue(new Node(x+ 1, y, distance + 1, nConnections,  this));
                    visitedHash.Add(index);
                }
            }
        }
    }

    static readonly Node[] nodes = new Node[1000];
    static Queue<Node> queue  = new Queue<Node>(1000);
    static HashSet<int> visitedHash = new HashSet<int>();
    static readonly int[] visitedArray = new int[HEIGHT];
    static readonly int[] intNodes = new int[1000];
    static int nodeIndex = 0;
    static int queueCount = 0;
    static int queueIndex = 0;

    static void ResetVisited()
    {
        for (int i = 0; i < visitedArray.Length; i++)
            visitedArray[i] = 0;
    }

    static void SetVisited(int x, int y)
    {
        visitedArray[y] |= 1 << x;
    }

    static bool IsAvailable(int x, int y)
    {
        return ((visitedArray[y] >> x) & 1) == 0;
    }

    static int SetIntNode(int x, int y, int distance, int connections, int parent)
    {
        return x | (y << 5) | (distance << 10) | (connections << 18) | (parent << 22);
    }
   
    static void SetChildren(int x, int y, int currentNode)
    {
        int distance = ((currentNode >> 10) & 255) + 1;
        int connections = (currentNode >> 18) & 15;
        int parentIndex = queueIndex - 1;

        if (y > 0 && (connections & 8) > 0 && IsAvailable(x, y - 1))
        {
            int index = x + WIDTH * (y - 1);
            int nConnections = mapNodes[index].connections;
            if ((nConnections & 2) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y - 1, distance, nConnections, parentIndex);
                SetVisited(x, y - 1);
            }
        }

        
        if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailable(x, y + 1))
        {
            int index = x + WIDTH * (y + 1);
            int nConnections = mapNodes[index].connections;
            if ((nConnections & 8) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y + 1, distance, nConnections, parentIndex);
                SetVisited(x, y + 1);
            }
        }

        if (x > 0 && (connections & 1) > 0 && IsAvailable(x - 1, y))
        {
            int index = x - 1 + WIDTH * y;
            int nConnections = mapNodes[index].connections;
            if ((nConnections & 4) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x - 1, y, distance, nConnections, parentIndex);
                SetVisited(x - 1, y);
            }
        }

        if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailable(x + 1, y))
        {
            int index = x + 1 + WIDTH * y;
            int nConnections = mapNodes[index].connections;
            if ((nConnections & 1) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x + 1, y, distance, nConnections, parentIndex);
                SetVisited(x + 1, y);
            }
        }
    }
    
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

        for (int i = 0; i < nodes.Length; i++)
            nodes[i] = new Node();

        if (BFS_BASIC)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
            watch.Reset();
            watch.Start();
            Node current = null;
            for (int i = 0; i < ITERATIONS; i++)
            {
            
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                current = new Node { x = START_X, y = START_Y, distance = 0, connections = rootConnections }; // root

                visitedHash.Clear();
                visitedHash.Add(startIndex);
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
            Console.WriteLine("BFS Basic: "+ time + " milliseconds"); 
        }

        if (BFS_NODEARRAY)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
            watch.Reset();
            watch.Start();
            Node current = null;
            for (int i = 0; i < ITERATIONS; i++)
            {
                nodeIndex = 0;
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                current = nodes[nodeIndex++];// root
                current.SetNode(START_X, START_Y, 0, rootConnections, null);

                visitedHash.Clear();
                visitedHash.Add(startIndex);
                queue.Enqueue(current);

                while (queue.Count > 0)
                {
                    current = queue.Dequeue();
                    if (current.x == GOAL_X && current.y == GOAL_Y)
                        break;

                    current.SetChildren();
                }
            }

            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS Node array: " + time + " milliseconds");
        }

        if (BFS_NOHASH)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
            watch.Reset();
            watch.Start();
            Node current = null;
            for (int i = 0; i < ITERATIONS; i++)
            {
                nodeIndex = 0;
                ResetVisited();
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                current = nodes[nodeIndex++];// root
                current.SetNode(START_X, START_Y, 0, rootConnections, null);
                SetVisited(START_X, START_Y);
                queue.Enqueue(current);

                while (queue.Count > 0)
                {
                    current = queue.Dequeue();
                    if (current.x == GOAL_X && current.y == GOAL_Y)
                        break;

                    current.SetChildrenNoHash();
                }
            }

            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS No hash: " + time + " milliseconds");
        }

        if (BFS_NOQUEUE)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
            watch.Reset();
            watch.Start();
            Node current = null;
            for (int i = 0; i < ITERATIONS; i++)
            {
                queueIndex = 0;
                queueCount = 0;
                ResetVisited();
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                current = nodes[queueCount++];// root
                current.SetNode(START_X, START_Y, 0, rootConnections, null);
                SetVisited(START_X, START_Y);
                
                while (queueCount > queueIndex)
                {
                    current = nodes[queueIndex++];
                    if (current.x == GOAL_X && current.y == GOAL_Y)
                        break;

                    current.SetChildrenNoQueue();
                }
            }

            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS No queue: " + time + " milliseconds");

            /*
            while (current.parent != null)
            {
                current = current.parent;
                if (current.parent != null)
                    map[current.x * 2 + 1, current.y * 2 + 1] = PATH;
            }

            DrawMazePath();*/
        }

        if (BFS_NOCLASS)
        {
            GC.Collect();
            GC.WaitForPendingFinalizers();
            watch.Reset();
            watch.Start();

            int current = 0;
            for (int i = 0; i < ITERATIONS; i++)
            {
                queueIndex = 0;
                queueCount = 0;
                ResetVisited();
                int startIndex = START_X + WIDTH * START_Y;
                int rootConnections = mapNodes[startIndex].connections;
                intNodes[queueCount++] = SetIntNode(START_X, START_Y, 0, rootConnections, 0);
                SetVisited(START_X, START_Y);
      
                while (queueCount > queueIndex)
                {
                    current = intNodes[queueIndex++];
                    int x = current & 31;
                    int y = (current >> 5) & 31;
                    if (x == GOAL_X && y== GOAL_Y)
                        break;

                    SetChildren(x, y, current);
                }
                
            }
           
            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS No class: " + time + " milliseconds");

            int parentIndex = current >> 22;

            while (parentIndex > 0)
            {
                current = intNodes[parentIndex];
                parentIndex = current >> 22;
                int x = current & 31;
                int y = (current >> 5) & 31;
                map[x * 2 + 1, y * 2 + 1] = PATH;    
            }

            DrawMazePath();
        }
    }
}

// }
```
