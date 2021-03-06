# Optimizing breadth first search

During the last contest (X-mas Rush) a good pathfinder was very important. The map was small with very short paths, so the only thing that made sense was a BFS. There are many ways to do a BFS and there are big differences in performance. I will explain a few examples in order of performance. If you only want to see the benchmark, run the code below. You can take the code and use it however you want. Just remember, it contains a lot of unnecessary code to make the comparisons properly. Make sure to clean out the useless parts. 
    Dont make the width of the map larger, because some parts depend on the width fitting inside a 32 bit integer. I do this because most gameboards in CG games are smaller than this. If you encounter a wider board... be creative! 
    
When talking about BFS I assume you are playing a game on a map. There are other applications for BFS than just travelling a map. The code below also contains a maze generator I found here: https://en.wikipedia.org/wiki/Maze_generation_algorithm.  I adapted the C-algorithm. I have an X-mas Rush post mortem here: https://tech.io/playgrounds/38549/x-mas-rush-post-mortem


```C# runnable
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.CompilerServices;

class Maze
{
    const int WIDTH = 30;
    const int HEIGHT = 5;
    const bool BFS_BASIC = true;
    const bool BFS_NONEW = true;
    const bool BFS_NOHASH = true;
    const bool BFS_NOQUEUE = true;
    const bool BFS_NOCLASS = true;
    const bool BFS_NORESET = true;
    const int MAZE_SEED = 0;
    const int ITERATIONS = 50000;
    // Code { autofold    
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
    static readonly int[] mapConnections = new int[WIDTH * HEIGHT];
    static readonly int[] visited = new int[WIDTH * HEIGHT];
    static int visitedIndex = 0;
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
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
                int nConnections = mapConnections[index];
                if ((nConnections & 2) > 0)
                {
                    queue.Enqueue(new Node(x, y - 1, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x + WIDTH * (y + 1);
            if (y < HEIGHT -1 && (connections & 2) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapConnections[index];
                if ((nConnections & 8) > 0)
                {
                    queue.Enqueue(new Node(x, y + 1, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x -1 + WIDTH * y;
            if (x > 0 && (connections & 1) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapConnections[index];
                if ((nConnections & 4) > 0)
                {
                    queue.Enqueue(new Node(x - 1, y, distance + 1, nConnections, this));
                    visitedHash.Add(index);
                }
            }

            index = x + 1 + WIDTH * y;
            if (x < WIDTH - 1 && (connections & 4) > 0 && !visitedHash.Contains(index))
            {
                int nConnections = mapConnections[index];
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

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    static void SetVisited(int x, int y)
    {
        visitedArray[y] |= 1 << x;
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    static bool IsAvailable(int x, int y)
    {
        return ((visitedArray[y] >> x) & 1) == 0;
    }


    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    static void SetVisitedNoReset(int x, int y)
    {
        visited[ x + y*WIDTH] = visitedIndex;
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    static bool IsAvailableNoReset(int x, int y)
    {
        return visited[x + y * WIDTH] != visitedIndex;
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    static int SetIntNode(int x, int y, int distance, int connections, int parent)
    {
        return x | (y << 5) | (distance << 10) | (connections << 18) | (parent << 22);
    }
   
    static void SetChildren(int current)
    {
        int x = current & 31;
        int y = (current >> 5) & 31;
        int distance = ((current >> 10) & 255) + 1;
        int connections = (current >> 18) & 15;
        int parentIndex = queueIndex - 1;

        if (y > 0 && (connections & 8) > 0 && IsAvailable(x, y - 1))
        {
            int index = x + WIDTH * (y - 1);
            int nConnections = mapConnections[index];
            if ((nConnections & 2) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y - 1, distance, nConnections, parentIndex);
                SetVisited(x, y - 1);
            }
        }

        
        if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailable(x, y + 1))
        {
            int index = x + WIDTH * (y + 1);
            int nConnections = mapConnections[index];
            if ((nConnections & 8) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y + 1, distance, nConnections, parentIndex);
                SetVisited(x, y + 1);
            }
        }

        if (x > 0 && (connections & 1) > 0 && IsAvailable(x - 1, y))
        {
            int index = x - 1 + WIDTH * y;
            int nConnections = mapConnections[index];
            if ((nConnections & 4) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x - 1, y, distance, nConnections, parentIndex);
                SetVisited(x - 1, y);
            }
        }

        if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailable(x + 1, y))
        {
            int index = x + 1 + WIDTH * y;
            int nConnections = mapConnections[index];
            if ((nConnections & 1) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x + 1, y, distance, nConnections, parentIndex);
                SetVisited(x + 1, y);
            }
        }
    }

    static void SetChildrenNoReset(int current)
    {
        int x = current & 31;
        int y = (current >> 5) & 31;
        int distance = ((current >> 10) & 255) + 1;
        int connections = (current >> 18) & 15;
        int parentIndex = queueIndex - 1;

        if (y > 0 && (connections & 8) > 0 && IsAvailableNoReset(x, y - 1))
        {
            int index = x + WIDTH * (y - 1);
            int nConnections = mapConnections[index];
            if ((nConnections & 2) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y - 1, distance, nConnections, parentIndex);
                SetVisitedNoReset(x, y - 1);
            }
        }


        if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailableNoReset(x, y + 1))
        {
            int index = x + WIDTH * (y + 1);
            int nConnections = mapConnections[index];
            if ((nConnections & 8) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x, y + 1, distance, nConnections, parentIndex);
                SetVisitedNoReset(x, y + 1);
            }
        }

        if (x > 0 && (connections & 1) > 0 && IsAvailableNoReset(x - 1, y))
        {
            int index = x - 1 + WIDTH * y;
            int nConnections = mapConnections[index];
            if ((nConnections & 4) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x - 1, y, distance, nConnections, parentIndex);
                SetVisitedNoReset(x - 1, y);
            }
        }

        if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailableNoReset(x + 1, y))
        {
            int index = x + 1 + WIDTH * y;
            int nConnections = mapConnections[index];
            if ((nConnections & 1) > 0)
            {
                intNodes[queueCount++] = SetIntNode(x + 1, y, distance, nConnections, parentIndex);
                SetVisitedNoReset(x + 1, y);
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

        for (int i = 0; i < HEIGHT; i++)
        {
            for (int j = 0; j < WIDTH; j++)
            {
                mapConnections[j + i * WIDTH] = mapNodes[j + i * WIDTH].connections;
            }
        }

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
                int rootConnections = mapConnections[startIndex];
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

        if (BFS_NONEW)
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
                int rootConnections = mapConnections[startIndex];
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
            Console.WriteLine("BFS No new: " + time + " milliseconds");
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
                int rootConnections = mapConnections[startIndex];
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
                int rootConnections = mapConnections[startIndex];
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
                int endCoordinates = GOAL_X | GOAL_Y << 5;
                int rootConnections = mapConnections[startIndex];
                intNodes[queueCount++] = SetIntNode(START_X, START_Y, 0, rootConnections, 0);
                SetVisited(START_X, START_Y);
      
                while (queueCount > queueIndex)
                {
                    current = intNodes[queueIndex++];
                    if ((current & 1023) == endCoordinates)
                        break;

                    SetChildren(current);
                }
                
            }
           
            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS No class: " + time + " milliseconds");
            /*
            int parentIndex = current >> 22;

            while (parentIndex > 0)
            {
                current = intNodes[parentIndex];
                parentIndex = current >> 22;
                int x = current & 31;
                int y = (current >> 5) & 31;
                map[x * 2 + 1, y * 2 + 1] = PATH;    
            }
            
            DrawMazePath();*/
        }

        if (BFS_NORESET)
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
                visitedIndex = i+1;
                int startIndex = START_X + WIDTH * START_Y;
                int endCoordinates = GOAL_X | GOAL_Y << 5;
                int rootConnections = mapConnections[startIndex];
                intNodes[queueCount++] = SetIntNode(START_X, START_Y, 0, rootConnections, 0);
                SetVisitedNoReset(START_X, START_Y);

                while (queueCount > queueIndex)
                {
                    current = intNodes[queueIndex++];
                    if ((current & 1023) == endCoordinates)
                        break;

                    SetChildrenNoReset(current);
                }

            }

            double time = watch.Elapsed.TotalMilliseconds;
            Console.WriteLine("BFS No reset: " + time + " milliseconds");

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

        //Console.ReadLine();
    }
    
}
// }
```

## The map representation

I am going to assume one map representation. There are many ways to do a map representation and the way it is done could impact the usefulness and speed of the various BFS versions. 


```C#

int[] mapConnections = new int[WIDTH * HEIGHT];
int indexToMapElement = x + WIDTH * y;

```
This is a very commonly used way to have a 1D array for a 2D map. Each element is a 4 bit number containing the information about the sides of the map that a tile can connect to, in clockwise direction. 1001 means it can connect up top, not to the right, not down, but can connect to the left. This is not the most compact representation. In X-mas Rush I had 7 map tiles (a full row) in one array element. This is very specific to that small map though. You can't do this if you have more than 8 tiles (8 * 4 = 32 bit). Maps can also be represented by objects (as opposed to integer values) in a 1D array or even objects in a 2D array. However, since this article is about optimization, I want to keep it somewhat performant. It would not be good if the map representation is the bottleneck here. 

## Basic BFS

BFS is about finding the shortest path by using a queue. You start in a location on the map and add all neighbouring tiles you can travel to, to this queue (this means you "enqueue" the tiles). Then you take the first item on the queue (you "dequeue") and look for all neighbouring tiles again, add them to the queue again and so on and so forth. You do this until you reach some predefined goal. In the X-mas Rush contest this goal was an item to collect. After collecting this item, you could reset your BFS and run it again. You could also make a list of reachable tiles at the end of your BFS run in order to have nodes to put in a search layer for use in minimax or MCTS (or something else). I don't want to get into that. We just assume that there is a start and a goal and thats it.

The most simple BFS has two major parts: 

### The Node class

The node class for a simple BFS probably looks like this:

```C#
class Node
{
    public Node parent = null;
    public int x = 0;
    public int y = 0;
    public int connections = 0;
    public int distance = 0;

```

The parent is needed if you want to backtrack to the beginning and output the moves you made. This is not always necessary. In X-mas Rush it usually wasn't, because many times  you only wanted to know which tiles you can reach and how many items you gathered. 

The use of x and y are obvious. The connections are the same connections you find in the map. You can just keep them in the map if you want and only look them up whenever you need them. I am not entirely sure how this will impact performance. The (manhattan) distance travelled is not used in this benchmark but is often necessary to track, because you often have a movement limit (20 steps in X-mas Rush).

### The main part of the algorithm

```C#
int startIndex = START_X + WIDTH * START_Y;
int rootConnections = mapConnections[startIndex];
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
```

The startindex is used to find the connections of the root to other tiles in the map. We create the first node (tile) and reference it with "current". We also use a HashSet<int>  to keep track of visited tiles. If we've been to a tile, we don't go there again. We add the start index to this hash so we can't visit the root tile again. We put the root tile in the queue. 

In the while loop we always take 1 tile out of the queue, check if it's the end tile and otherwise add children to it. Once the end tile has been reached, we stop. The hash set is defined globally and is cleared each time we do this BFS. The queue has a capacity set to 1000. if you do not set the capacity, it will slow down your algorithm as the capacity needs to be increased every time the queue grows. 

### Adding the children

We need to add reachable neighbors to the queue. We do this with the following method. There are 4 possible neighbors. Let's look at the first part specifically: this is the top neighbor. We first create an index to the map for this neighbor. It works the same way as I showed before, only now the y coordinate is 1 lower (y-1). We check if the y is larger than 0, because otherwise we're at the upper boundary of the maze. We also check if the connection is allowed for this particular tile. The "8" means "1000" which means the top bit is set and we can connect to the top neighbor. We also check if the visited hash already contains the neighbor. In that case, we can't add it. 

    We then also look up the connections for the top neighbor. That one has to be able to connect downward, which means 0010  (= 2). If all is good, then we enqueue the top neighbor and add it to the visited hash. All the other neighbors work similarly. The neighbor has its parent set to the current node (with "this" ). 

```C#
public void AddChildren()
{ 
    int index = x + WIDTH * (y - 1);
    if (y > 0 && (connections & 8) > 0 && !visitedHash.Contains(index))
    {
        int nConnections = mapConnections[index];
        if ((nConnections & 2) > 0)
        {
            queue.Enqueue(new Node(x, y - 1, distance + 1, nConnections, this));
            visitedHash.Add(index);
        }
    }
```
The rest of the method (for the other directions): 

```C#
 
    index = x + WIDTH * (y + 1);
    if (y < HEIGHT -1 && (connections & 2) > 0 && !visitedHash.Contains(index))
    {
        int nConnections = mapConnections[index];
        if ((nConnections & 8) > 0)
        {
            queue.Enqueue(new Node(x, y + 1, distance + 1, nConnections, this));
            visitedHash.Add(index);
        }
    }

    index = x -1 + WIDTH * y;
    if (x > 0 && (connections & 1) > 0 && !visitedHash.Contains(index))
    {
        int nConnections = mapConnections[index];
        if ((nConnections & 4) > 0)
        {
            queue.Enqueue(new Node(x - 1, y, distance + 1, nConnections, this));
            visitedHash.Add(index);
        }
    }

    index = x + 1 + WIDTH * y;
    if (x < WIDTH - 1 && (connections & 4) > 0 && !visitedHash.Contains(index))
    {
        int nConnections = mapConnections[index];
        if ((nConnections & 1) > 0)
        {
            queue.Enqueue(new Node(x+ 1, y, distance + 1, nConnections,  this));
            visitedHash.Add(index);
        }
    }
}
```

When the algorithm ends we can output the path with the following code:

```C#
while (current.parent != null)
{
    current = current.parent;
    if (current.parent != null)
        map[current.x * 2 + 1, current.y * 2 + 1] = PATH;
}
```

We can backtrack through the tiles on the map, each time selecting the parent node as a new current node. You can then choose what to do with this node. You may want to output its coordinates or a string saying "LEFT" or "UP". In this case, I output to a 2D char array called "map" that works to create a maze picture with a dotted path. It depends on what you need. The "PATH" in this case is a '.' character. 

## BFS without new (no garbage!)

You may have noticed in my code that I create new BFS-nodes when I create children. This is not the only way to do it. Another way is to create a global array with enough nodes in it that you keep reusing every time you need to do a BFS. In this way you don't create any garbage for the garbage collector to clean up. Instead of a constructor we have a "set" method and this slightly changes our Addchildren method as well. See the code below:

```C#
static readonly Node[] nodes = new Node[1000];
static int nodeIndex = 0;
```

```C#
public void SetChildren()
{
    int index = x + WIDTH * (y - 1);
    if (y > 0 && (connections & 8) > 0 && !visitedHash.Contains(index))
    {
        int nConnections = mapConnections[index];
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
        int nConnections = mapConnections[index];
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
        int nConnections = mapConnections[index];
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
        int nConnections = mapConnections[index];
        if ((nConnections & 1) > 0)
        {
            Node child = nodes[nodeIndex++];
            child.SetNode(x+1, y, distance + 1, nConnections, this);
            queue.Enqueue(child);
            visitedHash.Add(index);
        }
    }
}
```

When doing the benchmark. You'll sometimes notice a doubling of your performance if you use enough iterations. Sometimes the improvement is less, but still significant.

## BFS without hash

The hashset is pretty fast, especially if you use a version with integers (don't use a HashSet of nodes!). However, if you are travelling across most of the map or the map is small, an integer array with bits set to 1 or a boolean array that you set to true, will be more efficient. In this example I use an int-array. The code will change as follows:

```C#
 static readonly int[] visitedArray = new int[HEIGHT];
```

The array is reset at the beginning of the algorithm. With C++ you would do this using memset, which makes it extremely fast.

```C#
static void ResetVisited()
{
    for (int i = 0; i < visitedArray.Length; i++)
        visitedArray[i] = 0;
}
```
You set a "visited" bit as follows:

```C#
static void SetVisited(int x, int y)
{
    visitedArray[y] |= 1 << x;
}
```
This is the first time in this article we wil run into bit operations. I save the bit in the "y" element of the array on the location x bits to the left of the rightmost bit. The larger the x is, the farther to the left the bit is located.

We can check if a bit is set to 0 (and thus, the tile is available) with:

```C#
static bool IsAvailable(int x, int y)
{
    return ((visitedArray[y] >> x) & 1) == 0;
}
```
Which is really just the reverse operation. We shift the bit to the rightmost location and then check if it is set with "&". The SetChildren method is now also different and uses:

```C#
SetVisited(x, y - 1);
```

instead of:

```C#
visitedHash.Add(index);
```
For more detail look at the full code at the top. This change will be a large boost to your performance (tripling it in some cases).


## BFS without queue

The queue is an essential part of BFS. Using the queue datastructure clearly shows your intent. However, a queue is slower than an array. Moreover, you now have an array + a queue. What if you could do both with the same array? You need to make the following changes:

Instead of:

```C#
static int nodeIndex = 0;
```
You will have:

```C#
static int queueCount = 0;
static int queueIndex = 0;
```

And your main method will change to:

```C#
queueIndex = 0;
queueCount = 0;
ResetVisited();
int startIndex = START_X + WIDTH * START_Y;
int rootConnections = mapConnections[startIndex];
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
```

You notice we use the same "nodes" array both as the source of "creating" our current node and also to queue it. When we "create" a node, it simply means increasing the queueCount. When we "dequeue" it, we simply increase the queueIndex. If the queueindex becomes equal to the queueCount, it means we ran out of queued nodes and the algorithm ends.

The SetChildren method also changes:

It now has: 

```C#
Node child = nodes[queueCount++];
```

instead of:

```C#
Node child = nodes[nodeIndex++];
```

and we no longer "Enqueue" because we no longer have a queue. If you make this change, you will increase you performance by 50% or so.


## BFS without classes

This last part is a hard one. It gives you about a 10-15% boost to performance, but makes your algorithm much less readable. I do not recommend this change unless you need the performance and are somewhat comfortable with using bit operations and storing a lot of information inside different bitlocations of an integer.

Instead of using a class, a node is now a single integer. We store information in the integer as follows:

```C#
static int SetIntNode(int x, int y, int distance, int connections, int parent)
{
    return x | (y << 5) | (distance << 10) | (connections << 18) | (parent << 22);
}
```

We store the x on the rightmost bit, the y we store 5 bits to the left. Both coordinates are given a space of 5 bits, which means they cant be larger than 31. The distance is given 8 bits in this case (18-10), which means we can store up to 255 steps of the algorithm. This would be a pretty long path, so for our purposes it is fine. We have 4 bits for the connections we used earlier and the remaining bits after 22 (to 32) contains the parentindex for when we need to backtrack. There are plenty of other ways to configure your integer layout. In X-mas Rush I stored collected questitems in there as singular bits. 101 meant I collected the first and third item. 

We retrieve the information by shifting in the other direction. The set children method starts as:

```C#
static void SetChildren(int current)
{
    int x = current & 31;
    int y = (current >> 5) & 31;
    int distance = ((current >> 10) & 255) + 1;
    int connections = (current >> 18) & 15;
    int parentIndex = queueIndex - 1;
```

To get x we need to "&" with 31, which is really just "0000000000000000000000000011111". Doing this operation only keeps the rightmost 5 bits and thats what we need. We do something similar with the other information. We increase the distance because the children will have +1 distance. The parentindex is the queueIndex reduced by 1 (because we already increased it in the main method). The rest of the SetChildren method now looks like this:

```C#
if (y > 0 && (connections & 8) > 0 && IsAvailable(x, y - 1))
{
    int index = x + WIDTH * (y - 1);
    int nConnections = mapConnections[index];
    if ((nConnections & 2) > 0)
    {
        intNodes[queueCount++] = SetIntNode(x, y - 1, distance, nConnections, parentIndex);
        SetVisited(x, y - 1);
    }
}

if (y < HEIGHT - 1 && (connections & 2) > 0 && IsAvailable(x, y + 1))
{
    int index = x + WIDTH * (y + 1);
    int nConnections = mapConnections[index];
    if ((nConnections & 8) > 0)
    {
        intNodes[queueCount++] = SetIntNode(x, y + 1, distance, nConnections, parentIndex);
        SetVisited(x, y + 1);
    }
}

if (x > 0 && (connections & 1) > 0 && IsAvailable(x - 1, y))
{
    int index = x - 1 + WIDTH * y;
    int nConnections = mapConnections[index];
    if ((nConnections & 4) > 0)
    {
        intNodes[queueCount++] = SetIntNode(x - 1, y, distance, nConnections, parentIndex);
        SetVisited(x - 1, y);
    }
}

if (x < WIDTH - 1 && (connections & 4) > 0 && IsAvailable(x + 1, y))
{
    int index = x + 1 + WIDTH * y;
    int nConnections = mapConnections[index];
    if ((nConnections & 1) > 0)
    {
        intNodes[queueCount++] = SetIntNode(x + 1, y, distance, nConnections, parentIndex);
        SetVisited(x + 1, y);
    }
}
```

The main method is found below. Notice the complete lack of objects aside from the global integer array. Our end-of-BFS check is now a comparison with the rightmost 10 bits (1023 = 1111111111) of our current node (these 10 bits contain the x and y). 

```C#
int current = 0;
for (int i = 0; i < ITERATIONS; i++)
{
    queueIndex = 0;
    queueCount = 0;
    ResetVisited();
    int startIndex = START_X + WIDTH * START_Y;
    int endCoordinates = GOAL_X | GOAL_Y << 5;
    int rootConnections = mapConnections[startIndex];
    intNodes[queueCount++] = SetIntNode(START_X, START_Y, 0, rootConnections, 0);
    SetVisited(START_X, START_Y);

    while (queueCount > queueIndex)
    {
        current = intNodes[queueIndex++];
        if ((current & 1023) == endCoordinates)
            break;

        SetChildren(current);
    }
}
```

Outputting the path also involves bitshifting to get the parents.

```C#
int parentIndex = current >> 22;
while (parentIndex > 0)
{
    current = intNodes[parentIndex];
    parentIndex = current >> 22;
    int x = current & 31;
    int y = (current >> 5) & 31;
    map[x * 2 + 1, y * 2 + 1] = PATH;    
}

```

## BFS without reset

Magus and Neumann both told me to try and make an visited marker. The idea is to have an integer array with an element for every map tile. Every time you do a BFS, the marker for visited is 1 higher. This way you do not have to reset. If the marker on a tile is equal to the current marker, it has been visited, otherwise it hasn't. So it doesn't matter if the tile marker is 0 or 5 if the current marker is 6. This saves you the reset step. In this particular example I didn't find much of an improvement. It seems to overall be faster (if you do a couple tries). I think this may depend a lot on the size of the map. Resetting a large map is way more expensive. Also, this way is simply easier to implement. You no longer need to worry about remaining inside the 32 bits of an integer for every map row and things like that. 

## Some final words

Writing this article was useful for me as it gave me some good practice at optimizing. The current version is faster than what I used in the contest. If anyone has ideas to make it even faster, or maybe to just point out something silly I did, do let me know. I will fix any mistakes and I will code any new idea that I think might be faster and add it to the list.
