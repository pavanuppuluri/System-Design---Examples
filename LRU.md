# **Least Recently Used (LRU) Cache - System Design**

## **1. Introduction**
The **Least Recently Used (LRU) Cache** is a caching algorithm that removes the **least recently accessed** items when the cache reaches its capacity. This ensures efficient memory management and improved system performance.

---

## **2. Design Requirements**
### **Functional Requirements**
✅ `get(int key) → int` → Retrieve the value if it exists in the cache.  
✅ `put(int key, int value)` → Insert or update a key-value pair.  
✅ **Eviction Policy** → If the cache reaches capacity, remove the **Least Recently Used (LRU)** item.  

### **Non-Functional Requirements**
⚡ **Fast Operations**: `O(1)` time complexity for both `get()` and `put()`.  
⚡ **Memory Efficient**: Should not exceed cache capacity.  
⚡ **Concurrency Handling**: Multi-threaded support if needed.  

---

## **3. Data Structures Used**
| **Data Structure** | **Purpose** |
|--------------------|------------|
| **HashMap (Dictionary)** | Provides `O(1)` lookup for keys |
| **Doubly Linked List** | Maintains access order (Most Recently Used → Head, Least Recently Used → Tail) |

### **Why a Doubly Linked List?**
- **Efficient Deletion**: When an item is accessed, it must be moved to the front (`O(1)`).
- **Quick Eviction**: The least recently used item (at the tail) can be removed in `O(1)`.

---

## **4. Java Implementation of LRU Cache**
### **LRU Cache using HashMap & Doubly Linked List**
```java
import java.util.*;

class LRUCache {
    class Node {
        int key, value;
        Node prev, next;
        
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
    
    private final int capacity;
    private final Map<Integer, Node> cache;
    private final Node head, tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        
        // Dummy head and tail to make insertion/deletion easier
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }
    
    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insert(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    public int get(int key) {
        if (!cache.containsKey(key)) return -1;
        
        Node node = cache.get(key);
        remove(node);  // Remove from current position
        insert(node);  // Move to front as most recently used
        return node.value;
    }

    public void put(int key, int value) {
        if (cache.containsKey(key)) {
            remove(cache.get(key)); // Remove old node
        }

        Node newNode = new Node(key, value);
        cache.put(key, newNode);
        insert(newNode);

        if (cache.size() > capacity) {
            Node lru = tail.prev; // Least recently used node
            remove(lru);
            cache.remove(lru.key);
        }
    }

    // Helper method to print cache state (for debugging)
    public void printCache() {
        Node temp = head.next;
        System.out.print("Cache: ");
        while (temp != tail) {
            System.out.print(temp.key + " ");
            temp = temp.next;
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        LRUCache lru = new LRUCache(3);
        lru.put(1, 10);
        lru.put(2, 20);
        lru.put(3, 30);
        lru.printCache(); // Output: 3 2 1
        
        lru.get(2); // Access 2, making it most recently used
        lru.printCache(); // Output: 2 3 1
        
        lru.put(4, 40); // Evicts key 1 (Least Recently Used)
        lru.printCache(); // Output: 4 2 3
    }
}
```

