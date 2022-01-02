*"in mathematics you don't understand things. You just get used to them." -- John von Neumann*

Confession: for years, I have found the whole topic of linked lists vaguely intimidating. A good antidote to that fear, I find, is to dive in and solve a problem, explaining it so meticulously that you start to get used to it. The classic problem of reversing a linked list felt like a good place to start, so here is my deep dive into exactly how it works (in Python).

**Problem:** Reverse a linked list `L` in-place using only `O(1)` extra space.

We will use the classes below to represent the list's nodes and the list itself. (The `insert` and `print` methods are not efficiently implemented or even necessary to solve the problem, but are useful for testing.)
```
class Node:
    """A single node of a singly Linked List"""
    def __init__(self, data=None, next=None): 
        self.data = data
        self.next = next

class LinkedList:
    """A Linked List class with a single head node"""
    def __init__(self):  
        self.head = None
    
    def insert(self, data):
        """Appends a new node containing given data at the end of the list."""
        newNode = Node(data)
        if self.head:
            cur = self.head
            while cur.next:
                cur = cur.next
            cur.next = newNode
        else:
            self.head = newNode
  
    def print(self):
        """Prints contents of each node in the list."""
        cur = self.head
        while cur:
            print(cur.data)
            cur = cur.next
```

### Plan

We use this example throughout:
```
*    a -> b -> c -> d -> *   # Use * to represent None/null
```
We will loop through the list, reversing arrows as we go.
```
*    a -> b -> c -> d -> *
* <- a    b -> c -> d -> *
* <- a <- b    c -> d -> *
* <- a <- b <- c    d -> *
* <- a <- b <- c <- d    *
```
The reversed list is growing while the original list is shrinking. We want to keep pointers trained on the heads of both lists. Specifically, after each step, we want these invariants to hold:

1. A "previous" pointer `P` points to the head of the reversed list.
2. The "current" pointer `C` points to the head of the original list.
3. The "after" pointer `A` points to the node after `C`.

```
# invariant
       P    C    A
... <- x    y -> z -> ...
```

### Initial Setup

Let's start by setting up the invariant:
```
P = None
C = L.head
A = C.next
```
In our example, it looks like this:
```
P    C    A                
*    a -> b -> c -> d -> *
```

### Flipping an Arrow
To take a step forward, we need to

1. Flip the arrow of the node sitting under `C` to point back to `P`.
2. Shift the pointers forward to maintain our loop invariant.

```
# initial state
P    C    A          
*    a -> b -> c -> d -> *

# one step forward
     P    C    A
* <- a    b -> c -> d -> *
```

The code to do this is:
```
C.next = P
P = C
C = A
A = C.next
```
Here's how our workspace changes as we run these lines step-by-step:
```
# initial state

P    C    A            
*    a -> b -> c -> d -> *

# first step

P    C    A
* <- a    b -> c -> d -> *   # C.next = P

    PC    A
* <- a    b -> c -> d -> *   # P = C

     P   CA
* <- a    b -> c -> d -> *   # C = A

     P    C    A
* <- a    b -> c -> d -> *   # A = C.next

# second step

     P    C    A
* <- a <- b    c -> d -> *   # C.next = P

         PC    A
* <- a <- b    c -> d -> *   # P = C

          P   CA
* <- a <- b    c -> d -> *   # C = A

          P    C    A
* <- a <- b    c -> d -> *   # A = C.next

# more steps...

# final state
                    P   CA
* <- a <- b <- c <- d -> *
```


### Putting it all together

The code so far gives us this sketch of a solution:

```
def reverseList(L):
    P = None
    C = L.head
    A = C.next
    while not done:
        C.next = P
        P = C
        C = A
        A = C.next
```

To get to a working function, we need a few final tweaks:

1. Figure out when we're done. Based on the final state above, we need to keep going until `C` points to the null at the end of the list.

2. When `C` points to null, `A = C.next` will throw an error. So we have to check if `C` is null before doing that assignment.

3. After we finish, set the reversed list's head to `P`.

Just for fun, we also change `P, A, C` to the more descriptive `prev, cur, after` and squish the reassignments into some nice little one-liners.

```
def reverseList(L):
    """Reverses the input linked list in-place using O(1) extra space."""
    prev, cur, after = (None, L.head, L.head.next)
    while cur:
        cur.next, prev, cur = (prev, cur, after)
        if cur: after = after.next
    L.head = prev
```

### Test

```
L = LinkedList()
L.insert(1)
L.insert(4)
L.insert(9)
L.insert(16)
print('List:')
L.print()
reverseList(L)
print('Reversed:')
L.print()
```

Output:
```
List:
1
4
9
16
Reversed:
16
9
4
1
```

### Credits
- Node and LinkedList definitions adapted from [How to Reverse a Linked List in Python](https://www.educative.io/edpresso/how-to-reverse-a-linked-list-in-python)
- License: [Creative Commons -Attribution -ShareAlike 4.0 (CC-BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/)


