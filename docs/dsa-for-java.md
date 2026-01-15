# DSA for Java Interviews (Essential Guide)

This guide covers data structures and algorithms commonly asked in Java interviews, with Java-specific implementations and patterns.

## Definitions

- **Big-O**: describes how runtime or space grows as input size grows.
- **Array**: contiguous memory, fast indexing, expensive mid inserts.
- **Linked list**: nodes linked by pointers, fast inserts, slow random access.
- **Stack**: LIFO structure (push/pop).
- **Queue**: FIFO structure (offer/poll).
- **Hash map**: key-value store with average O(1) lookup.
- **Tree**: hierarchical structure (e.g., binary tree, heap).
- **Graph**: nodes connected by edges (directed or undirected).

## Illustrations

- **Stack**: a pile of plates - last added is first removed.
- **Queue**: a checkout line - first in is first out.
- **Hash map**: an index in a book that jumps to the right page quickly.
- **Tree**: a family tree where each node has children.

## Code Examples

```java
int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

## Interview Questions

1. Explain the difference between O(n) and O(n log n).
2. When would you use a stack vs a queue?
3. Why are hash maps O(1) on average but not worst-case?
4. How do BFS and DFS differ?
5. What is the difference between a heap and a BST?

---

## 1) Big-O Complexity

### Time Complexity

| Notation | Name | Example |
|----------|------|---------|
| O(1) | Constant | HashMap get/put |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Array traversal |
| O(n log n) | Linearithmic | Merge sort, efficient sorting |
| O(n²) | Quadratic | Nested loops, bubble sort |
| O(2ⁿ) | Exponential | Recursive fibonacci |
| O(n!) | Factorial | Permutations |

### Space Complexity

Consider:
- Input storage
- Auxiliary space (extra space used)
- Recursion call stack

### Java Collections Complexity

| Collection | Access | Search | Insert | Delete |
|------------|--------|--------|--------|--------|
| ArrayList | O(1) | O(n) | O(n)* | O(n) |
| LinkedList | O(n) | O(n) | O(1)** | O(1)** |
| HashMap | - | O(1) | O(1) | O(1) |
| TreeMap | - | O(log n) | O(log n) | O(log n) |
| HashSet | - | O(1) | O(1) | O(1) |
| TreeSet | - | O(log n) | O(log n) | O(log n) |
| PriorityQueue | - | O(n) | O(log n) | O(log n) |

*ArrayList append is amortized O(1)
**LinkedList insert/delete at known position

---

## 2) Arrays and Strings

### Array Basics in Java

```java
// Declaration and initialization
int[] arr = new int[10];
int[] arr2 = {1, 2, 3, 4, 5};
int[] arr3 = new int[]{1, 2, 3};

// 2D array
int[][] matrix = new int[3][4];
int[][] matrix2 = {{1, 2}, {3, 4}, {5, 6}};

// Array methods
Arrays.sort(arr);                      // Sort ascending
Arrays.sort(arr, (a, b) -> b - a);     // Sort descending (Integer[])
Arrays.fill(arr, 0);                   // Fill with value
Arrays.copyOf(arr, newLength);         // Copy with new length
Arrays.binarySearch(sortedArr, key);   // Binary search
```

### String Manipulation

```java
String s = "Hello World";

// Common methods
s.length();                    // 11
s.charAt(0);                   // 'H'
s.substring(0, 5);             // "Hello"
s.indexOf("World");            // 6
s.contains("World");           // true
s.startsWith("Hello");         // true
s.endsWith("World");           // true
s.toLowerCase();
s.toUpperCase();
s.trim();                      // Remove leading/trailing whitespace
s.replace("World", "Java");
s.split(" ");                  // ["Hello", "World"]
s.toCharArray();               // char[]

// Convert to/from char array
char[] chars = s.toCharArray();
String newStr = new String(chars);
String fromChars = String.valueOf(chars);

// StringBuilder (mutable, preferred)
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" World");
sb.insert(5, ",");
sb.delete(5, 6);
sb.reverse();
String result = sb.toString();
```

### Common Array Patterns

**Two Pointers**

```java
// Find pair with sum in sorted array
public int[] twoSum(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return new int[]{left, right};
        else if (sum < target) left++;
        else right--;
    }
    return new int[]{-1, -1};
}

// Remove duplicates from sorted array (in-place)
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int writeIdx = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] != nums[i - 1]) {
            nums[writeIdx++] = nums[i];
        }
    }
    return writeIdx;
}
```

**Sliding Window**

```java
// Maximum sum subarray of size k
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    
    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];  // Slide window
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}

// Longest substring without repeating characters
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int maxLength = 0;
    int start = 0;
    
    for (int end = 0; end < s.length(); end++) {
        char c = s.charAt(end);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= start) {
            start = lastSeen.get(c) + 1;
        }
        lastSeen.put(c, end);
        maxLength = Math.max(maxLength, end - start + 1);
    }
    return maxLength;
}
```

**Prefix Sum**

```java
// Range sum queries
public class NumArray {
    private int[] prefix;
    
    public NumArray(int[] nums) {
        prefix = new int[nums.length + 1];
        for (int i = 0; i < nums.length; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }
    
    public int sumRange(int left, int right) {
        return prefix[right + 1] - prefix[left];
    }
}
```

---

## 3) HashMaps and HashSets

### HashMap Usage

```java
Map<String, Integer> map = new HashMap<>();

// Basic operations
map.put("key", 1);
map.get("key");                         // Returns null if not found
map.getOrDefault("key", 0);             // Default value if not found
map.containsKey("key");
map.containsValue(1);
map.remove("key");
map.size();
map.isEmpty();
map.clear();

// Iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    Integer value = entry.getValue();
}

for (String key : map.keySet()) { }
for (Integer value : map.values()) { }

// Advanced operations
map.putIfAbsent("key", 1);              // Put only if key absent
map.computeIfAbsent("key", k -> 1);     // Compute only if absent
map.computeIfPresent("key", (k, v) -> v + 1);
map.merge("key", 1, Integer::sum);      // Add to existing or set
```

### Common HashMap Patterns

**Frequency Counter**

```java
public Map<Character, Integer> countFrequency(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray()) {
        freq.merge(c, 1, Integer::sum);
        // or: freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    return freq;
}

// Two Sum
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```

**Group Anagrams**

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    
    return new ArrayList<>(groups.values());
}
```

### HashSet Usage

```java
Set<Integer> set = new HashSet<>();

set.add(1);
set.contains(1);
set.remove(1);
set.size();

// Set operations
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);

Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);

Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);

// Find duplicates
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) return true;  // add() returns false if exists
    }
    return false;
}
```

---

## 4) Stacks and Queues

### Stack

```java
// Using Deque (preferred)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);           // Add to top
stack.pop();             // Remove from top
stack.peek();            // View top without removing
stack.isEmpty();

// Using Stack class (legacy)
Stack<Integer> legacyStack = new Stack<>();
```

**Common Stack Problems**

```java
// Valid Parentheses
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', '}', '{', ']', '[');
    
    for (char c : s.toCharArray()) {
        if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    return stack.isEmpty();
}

// Min Stack
class MinStack {
    private Deque<Integer> stack = new ArrayDeque<>();
    private Deque<Integer> minStack = new ArrayDeque<>();
    
    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }
    
    public void pop() {
        if (stack.pop().equals(minStack.peek())) {
            minStack.pop();
        }
    }
    
    public int top() { return stack.peek(); }
    public int getMin() { return minStack.peek(); }
}

// Daily Temperatures (monotonic stack)
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();  // Store indices
    
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
            int prevIdx = stack.pop();
            result[prevIdx] = i - prevIdx;
        }
        stack.push(i);
    }
    return result;
}
```

### Queue

```java
// Using Deque (preferred)
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);          // Add to back (or addLast)
queue.poll();            // Remove from front (or removeFirst)
queue.peek();            // View front (or peekFirst)
queue.isEmpty();

// Using LinkedList
Queue<Integer> queue = new LinkedList<>();
```

### Priority Queue (Heap)

```java
// Min heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
// or: new PriorityQueue<>((a, b) -> b - a);

// Custom comparator
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);

// Operations
pq.offer(element);       // Add O(log n)
pq.poll();               // Remove and return min/max O(log n)
pq.peek();               // View min/max O(1)
pq.size();
```

**K Largest/Smallest Elements**

```java
// K largest using min heap
public int[] kLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    
    for (int num : nums) {
        minHeap.offer(num);
        if (minHeap.size() > k) {
            minHeap.poll();  // Remove smallest
        }
    }
    
    return minHeap.stream().mapToInt(i -> i).toArray();
}

// Merge K Sorted Lists
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>(
        (a, b) -> a.val - b.val
    );
    
    for (ListNode list : lists) {
        if (list != null) pq.offer(list);
    }
    
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (!pq.isEmpty()) {
        ListNode node = pq.poll();
        current.next = node;
        current = current.next;
        if (node.next != null) {
            pq.offer(node.next);
        }
    }
    
    return dummy.next;
}
```

---

## 5) Linked Lists

### ListNode Definition

```java
public class ListNode {
    int val;
    ListNode next;
    
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

### Common Operations

```java
// Traverse
ListNode current = head;
while (current != null) {
    System.out.println(current.val);
    current = current.next;
}

// Find length
int length = 0;
ListNode curr = head;
while (curr != null) {
    length++;
    curr = curr.next;
}

// Find middle (slow/fast pointer)
public ListNode findMiddle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}

// Detect cycle
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}

// Reverse linked list
public ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Merge two sorted lists
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    
    current.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

---

## 6) Trees

### TreeNode Definition

```java
public class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    
    TreeNode(int val) { this.val = val; }
}
```

### Tree Traversals

```java
// Inorder (Left, Root, Right) - BST gives sorted order
public void inorder(TreeNode root) {
    if (root == null) return;
    inorder(root.left);
    System.out.println(root.val);
    inorder(root.right);
}

// Preorder (Root, Left, Right)
public void preorder(TreeNode root) {
    if (root == null) return;
    System.out.println(root.val);
    preorder(root.left);
    preorder(root.right);
}

// Postorder (Left, Right, Root)
public void postorder(TreeNode root) {
    if (root == null) return;
    postorder(root.left);
    postorder(root.right);
    System.out.println(root.val);
}

// Level Order (BFS)
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> level = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        
        result.add(level);
    }
    return result;
}
```

### Common Tree Problems

```java
// Maximum depth
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// Check if balanced
public boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

private int checkHeight(TreeNode node) {
    if (node == null) return 0;
    
    int left = checkHeight(node.left);
    if (left == -1) return -1;
    
    int right = checkHeight(node.right);
    if (right == -1) return -1;
    
    if (Math.abs(left - right) > 1) return -1;
    return 1 + Math.max(left, right);
}

// Validate BST
public boolean isValidBST(TreeNode root) {
    return validate(root, null, null);
}

private boolean validate(TreeNode node, Integer min, Integer max) {
    if (node == null) return true;
    if (min != null && node.val <= min) return false;
    if (max != null && node.val >= max) return false;
    return validate(node.left, min, node.val) && 
           validate(node.right, node.val, max);
}

// Lowest Common Ancestor
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    
    if (left != null && right != null) return root;
    return left != null ? left : right;
}
```

---

## 7) Graphs

### Graph Representations

```java
// Adjacency List (most common)
Map<Integer, List<Integer>> graph = new HashMap<>();

// Add edge
graph.computeIfAbsent(from, k -> new ArrayList<>()).add(to);
// For undirected:
graph.computeIfAbsent(to, k -> new ArrayList<>()).add(from);

// Adjacency Matrix
int[][] matrix = new int[n][n];
matrix[from][to] = 1;  // Or weight
```

### BFS (Breadth-First Search)

```java
public void bfs(Map<Integer, List<Integer>> graph, int start) {
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    
    visited.add(start);
    queue.offer(start);
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        System.out.println(node);
        
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
}

// Shortest path (unweighted)
public int shortestPath(Map<Integer, List<Integer>> graph, int start, int end) {
    Set<Integer> visited = new HashSet<>();
    Queue<int[]> queue = new LinkedList<>();  // {node, distance}
    
    visited.add(start);
    queue.offer(new int[]{start, 0});
    
    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int node = curr[0], dist = curr[1];
        
        if (node == end) return dist;
        
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.offer(new int[]{neighbor, dist + 1});
            }
        }
    }
    return -1;  // Not found
}
```

### DFS (Depth-First Search)

```java
// Iterative
public void dfsIterative(Map<Integer, List<Integer>> graph, int start) {
    Set<Integer> visited = new HashSet<>();
    Deque<Integer> stack = new ArrayDeque<>();
    
    stack.push(start);
    
    while (!stack.isEmpty()) {
        int node = stack.pop();
        if (visited.contains(node)) continue;
        
        visited.add(node);
        System.out.println(node);
        
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (!visited.contains(neighbor)) {
                stack.push(neighbor);
            }
        }
    }
}

// Recursive
public void dfsRecursive(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
    if (visited.contains(node)) return;
    
    visited.add(node);
    System.out.println(node);
    
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        dfsRecursive(graph, neighbor, visited);
    }
}
```

### Topological Sort

```java
// Using DFS
public List<Integer> topologicalSort(Map<Integer, List<Integer>> graph, int numNodes) {
    Set<Integer> visited = new HashSet<>();
    Deque<Integer> stack = new ArrayDeque<>();
    
    for (int i = 0; i < numNodes; i++) {
        if (!visited.contains(i)) {
            topoSortDFS(graph, i, visited, stack);
        }
    }
    
    return new ArrayList<>(stack);
}

private void topoSortDFS(Map<Integer, List<Integer>> graph, int node, 
                         Set<Integer> visited, Deque<Integer> stack) {
    visited.add(node);
    for (int neighbor : graph.getOrDefault(node, List.of())) {
        if (!visited.contains(neighbor)) {
            topoSortDFS(graph, neighbor, visited, stack);
        }
    }
    stack.push(node);  // Add after processing all children
}

// Using Kahn's Algorithm (BFS)
public List<Integer> topologicalSortKahn(Map<Integer, List<Integer>> graph, int numNodes) {
    int[] inDegree = new int[numNodes];
    
    for (List<Integer> neighbors : graph.values()) {
        for (int neighbor : neighbors) {
            inDegree[neighbor]++;
        }
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numNodes; i++) {
        if (inDegree[i] == 0) queue.offer(i);
    }
    
    List<Integer> result = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        result.add(node);
        
        for (int neighbor : graph.getOrDefault(node, List.of())) {
            if (--inDegree[neighbor] == 0) {
                queue.offer(neighbor);
            }
        }
    }
    
    return result.size() == numNodes ? result : List.of();  // Empty if cycle exists
}
```

---

## 8) Sorting Algorithms

### Quick Reference

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

### Merge Sort

```java
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] leftArr = Arrays.copyOfRange(arr, left, mid + 1);
    int[] rightArr = Arrays.copyOfRange(arr, mid + 1, right + 1);
    
    int i = 0, j = 0, k = left;
    while (i < leftArr.length && j < rightArr.length) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];
        } else {
            arr[k++] = rightArr[j++];
        }
    }
    
    while (i < leftArr.length) arr[k++] = leftArr[i++];
    while (j < rightArr.length) arr[k++] = rightArr[j++];
}
```

### Quick Sort

```java
public void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIdx = partition(arr, low, high);
        quickSort(arr, low, pivotIdx - 1);
        quickSort(arr, pivotIdx + 1, high);
    }
}

private int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    
    swap(arr, i + 1, high);
    return i + 1;
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

---

## 9) Binary Search

```java
// Standard binary search
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;  // Not found
}

// Find first occurrence (lower bound)
public int lowerBound(int[] arr, int target) {
    int left = 0, right = arr.length;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < target) left = mid + 1;
        else right = mid;
    }
    
    return left;  // First index where arr[i] >= target
}

// Find last occurrence
public int upperBound(int[] arr, int target) {
    int left = 0, right = arr.length;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) left = mid + 1;
        else right = mid;
    }
    
    return left - 1;  // Last index where arr[i] <= target
}

// Search in rotated sorted array
public int searchRotated(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (nums[mid] == target) return mid;
        
        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        // Right half is sorted
        else {
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    
    return -1;
}
```

---

## 10) Dynamic Programming

### Patterns

| Pattern | Examples |
|---------|----------|
| 1D DP | Fibonacci, climbing stairs, house robber |
| 2D DP | Longest common subsequence, edit distance |
| Knapsack | 0/1 knapsack, coin change, subset sum |
| String DP | Palindromes, word break |
| Tree DP | Maximum path sum |

### Classic Problems

```java
// Fibonacci
public int fib(int n) {
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}

// Space-optimized Fibonacci
public int fibOptimized(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}

// Coin Change (minimum coins)
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    
    return dp[amount] > amount ? -1 : dp[amount];
}

// Longest Common Subsequence
public int lcs(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}

// 0/1 Knapsack
public int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];
    
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(
                    dp[i - 1][w],  // Don't take item
                    dp[i - 1][w - weights[i - 1]] + values[i - 1]  // Take item
                );
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    return dp[n][capacity];
}

// Longest Increasing Subsequence
public int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);
    int maxLen = 1;
    
    for (int i = 1; i < nums.length; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }
    
    return maxLen;
}
```

---

## 11) Backtracking

```java
// Permutations
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackPermute(nums, new ArrayList<>(), new boolean[nums.length], result);
    return result;
}

private void backtrackPermute(int[] nums, List<Integer> path, 
                              boolean[] used, List<List<Integer>> result) {
    if (path.size() == nums.length) {
        result.add(new ArrayList<>(path));
        return;
    }
    
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        
        path.add(nums[i]);
        used[i] = true;
        
        backtrackPermute(nums, path, used, result);
        
        path.remove(path.size() - 1);
        used[i] = false;
    }
}

// Subsets
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackSubsets(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrackSubsets(int[] nums, int start, 
                              List<Integer> path, List<List<Integer>> result) {
    result.add(new ArrayList<>(path));
    
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);
        backtrackSubsets(nums, i + 1, path, result);
        path.remove(path.size() - 1);
    }
}

// Combination Sum
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    backtrackCombination(candidates, target, 0, new ArrayList<>(), result);
    return result;
}

private void backtrackCombination(int[] candidates, int remaining, int start,
                                  List<Integer> path, List<List<Integer>> result) {
    if (remaining == 0) {
        result.add(new ArrayList<>(path));
        return;
    }
    if (remaining < 0) return;
    
    for (int i = start; i < candidates.length; i++) {
        path.add(candidates[i]);
        backtrackCombination(candidates, remaining - candidates[i], i, path, result);
        path.remove(path.size() - 1);
    }
}
```

---

## 12) Interview Questions

### Arrays/Strings
1. Two Sum, Three Sum
2. Container With Most Water
3. Valid Anagram, Group Anagrams
4. Longest Substring Without Repeating Characters
5. Product of Array Except Self

### Linked Lists
6. Reverse Linked List
7. Detect Cycle in Linked List
8. Merge Two Sorted Lists
9. Remove Nth Node From End

### Trees
10. Maximum Depth of Binary Tree
11. Validate Binary Search Tree
12. Level Order Traversal
13. Lowest Common Ancestor
14. Serialize and Deserialize Binary Tree

### Graphs
15. Number of Islands
16. Clone Graph
17. Course Schedule (Topological Sort)
18. Word Ladder

### Dynamic Programming
19. Climbing Stairs
20. Coin Change
21. Longest Common Subsequence
22. Word Break

### Design
23. LRU Cache
24. Min Stack
25. Implement Trie

---

## 13) Quick Reference: Java Collections for DSA

```java
// Arrays
Arrays.sort(arr);                    // O(n log n)
Arrays.binarySearch(arr, key);       // O(log n)
Arrays.copyOf(arr, length);
Arrays.fill(arr, value);

// Lists
List<Integer> list = new ArrayList<>();
Collections.sort(list);
Collections.reverse(list);
Collections.binarySearch(list, key);

// Stack
Deque<Integer> stack = new ArrayDeque<>();
stack.push(x);  // addFirst
stack.pop();    // removeFirst
stack.peek();   // peekFirst

// Queue
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(x); // addLast
queue.poll();   // removeFirst
queue.peek();   // peekFirst

// Priority Queue
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// HashMap
Map<K, V> map = new HashMap<>();
map.getOrDefault(key, defaultValue);
map.computeIfAbsent(key, k -> new ArrayList<>());
map.merge(key, 1, Integer::sum);

// HashSet
Set<T> set = new HashSet<>();
set.add(x);
set.contains(x);

// TreeMap/TreeSet (sorted)
TreeMap<K, V> treeMap = new TreeMap<>();
treeMap.firstKey();
treeMap.lastKey();
treeMap.floorKey(key);   // Greatest key <= key
treeMap.ceilingKey(key); // Smallest key >= key
```

