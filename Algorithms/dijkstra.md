# Dijkstra Algorithm - Shortest Path Algorithm

下面是演算法的簡要流程圖：
1. 初始化距離和優先佇列
2. 選擇距離最短的未訪問節點
3. 更新鄰居的距離
4. 標記當前節點為已訪問
5. 回到步驟 2，直到所有節點訪問完畢

```csharp
public class Graph
{
    private readonly int _vertexNumber;
    private readonly Dictionary<int, List<(int Dest, int Weight)>> _adjs;
    
    public Graph(int vertexNumber)
    {
        _vertexNumber = vertexNumber;
        _adjs = new Dictionary<int, List<(int Dest, int Weight)>>();
        for (var i = 0; i < vertexNumber; i++)
        {
            _adjs[i] = new List<(int Dest, int Weight)>();
        }
    }

    public void AddEdge(int u, int v, int weight)
    {
        _adjs[u].Add((v, weight));
        // 如果是有向圖, 這行可以刪除
        _adjs[v].Add((u, weight));
    }

    public void Dijkstra(int src)
    {
        // 記錄原點到各點的最短距離
        var dist = new int[_vertexNumber];
        // 記錄各點的前驅節點
        var prev = new int[_vertexNumber];
        for (var i = 0; i < _vertexNumber; i++)
        {
            // 初始設每個節點距離為無限大
            dist[i] = int.MaxValue;
            prev[i] = -1;
        }
        
        dist[src] = 0;

        var priorityQueue = new SortedSet<(int Dest, int Weight)>(Comparer<(int Dest, int Weight)>.Create((a, b) =>
        {
            var result = a.Weight.CompareTo(b.Weight);
            if (result == 0)
            {
                result = a.Dest.CompareTo(b.Dest);
            }

            return result;
        }));

        priorityQueue.Add((src, 0));

        while (priorityQueue.Count > 0)
        {
            // 每次挑出距離原點最近且未拜訪過的節點
            var min = priorityQueue.Min;
            priorityQueue.Remove(min);

            var newNode = min.Dest;

            foreach (var neighbor in _adjs[newNode])
            {
                var v = neighbor.Dest;
                var weight = neighbor.Weight;

                // 防呆 或 沒找到更短的路徑
                if (dist[newNode] == int.MaxValue || dist[newNode] + weight >= dist[v])
                {
                    continue;
                }
                
                if (dist[v] != int.MaxValue)
                {
                    priorityQueue.Remove((v, dist[v]));
                }

                dist[v] = dist[newNode] + weight;
                prev[v] = newNode;
                priorityQueue.Add((v, dist[v]));
            }
        }

        PrintSolution(dist, prev, src);
    }

    private void PrintSolution(int[] dist, int[] prev, int src)
    {
        Console.WriteLine("Vertex \t Distance from Source");
        for (var i = 0; i < _vertexNumber; i++)
        {
            Console.WriteLine($"{i} \t {dist[i]}");
        }

        // 输出从源顶点到最后一个顶点的最短路径
        var target = _vertexNumber - 1;
        Console.WriteLine($"\nShortest path from {src} to {target}:");
        PrintPath(prev, target);
        Console.WriteLine();
    }

    private void PrintPath(int[] prev, int j)
    {
        if (prev[j] == -1)
        {
            Console.Write(j);
            return;
        }

        PrintPath(prev, prev[j]);
        Console.Write($" -> {j}");
    }
}
```

```csharp
var graph = new Graph(9);
graph.AddEdge(0, 1, 4);
graph.AddEdge(0, 7, 8);
graph.AddEdge(1, 2, 8);
graph.AddEdge(1, 7, 11);
graph.AddEdge(2, 3, 7);
graph.AddEdge(2, 8, 2);
graph.AddEdge(2, 5, 4);
graph.AddEdge(3, 4, 9);
graph.AddEdge(3, 5, 14);
graph.AddEdge(4, 5, 10);
graph.AddEdge(5, 6, 2);
graph.AddEdge(6, 7, 1);
graph.AddEdge(6, 8, 6);
graph.AddEdge(7, 8, 7);
graph.Dijkstra(0);
```


| Vertex | Distance from Source |
|--------|----------------------|
| 0      | 0                    |
| 1      | 4                    |
| 2      | 12                   |
| 3      | 19                   |
| 4      | 21                   |
| 5      | 11                   |
| 6      | 9                    |
| 7      | 8                    |
| 8      | 14                   |

> Shortest path from 0 to 8:
<br/> 0 -> 1 -> 2 -> 8