*"in mathematics you don't understand things. You just get used to them." -- John von Neumann*

For years, I have found the whole topic of linked lists vaguely confusing and intimidating: *how do you keep track of all those pointers?*

One way to overcome this sort of feeling is to dive in and solve a problem, explaining every detail so meticulously that you get used to the moving parts. In the best case, those parts begin to look less like strange and brilliant bolts of inspiration that you yourself could never come up with, and more like common-sense moves that emerge inevitably from sitting with the problem long enough.

The classic problem of reversing a linked list felt like a good place to start. So here is my deep dive into how linked list reversal works and why it really *has* to work that way (in Python).

### Problem
Reverse a linked list `L` in-place using only `O(1)` extra space.

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

We will use this linked list as an example throughout:
```
*    a -> b -> c -> d -> *   # Use * to represent None/null
```
The plan is to loop through the list, reversing arrows as we go.
```
*    a -> b -> c -> d -> *
* <- a    b -> c -> d -> *
* <- a <- b    c -> d -> *
* <- a <- b <- c    d -> *
* <- a <- b <- c <- d    *
```
Here we run into our first issue: our algorithm doesn't have this "god's eye view" of the list while reversing it in place, only an `O(1)` handful of pointers to specific locations. So which locations do we need pointers to? Well, at each step, the head of the (shrinking) original list is repointed to the head of the (growing) reversed list, so we definitely need two pointers -- call them `P` (previous) and `C` (current) -- trained on these heads at all times:
```
       P    C
... <- x    y -> z -> ...
```
But if we only have these two pointers, it's impossible to keep `C` where it belongs, as flipping an arrow burns our only bridge to the new head of the original list.
```
# time to flip y's arrow to point back to x, right?
       P    C
... <- x    y -> z -> ...

# ... oops, we can't get C over to z now
       P    C
... <- x <- y    z -> ...
```
A third pointer, `A` (ahead), would enable us to cross that gap even after flipping the arrow.
```
# P and C are in head position
       P    C    A
... <- x    y -> z -> ...

# flip y's arrow
       P    C    A
... <- x <- y    z -> ...

# shift pointers forward to keep P and C in head position
            P    C    A
... <- x <- y    z -> ...
```
When shifting forward, we need to order the moves so that we don't lose any bridges we need. If we move `A` first, `C` will be unable to reach `z`, and if we move `C` first, `P` will be unable to reach `y`. So we have to move `P` first, then `C`, then finally `A`.

### Coding it up

To turn this plan into a working algorithm, let's start by setting up the pointers.
```
P = None
C = L.head
A = L.head.next
```
On our example list, this looks like
```
P    C    A                
*    a -> b -> c -> d -> *
```
To take a step forward, we need to flip the arrow under `C` and shift the pointers forward: first `P`, then `C`, then `A`.
```
# flip arrow
C.next = P
# shift pointers forward
P = C
C = A
A = C.next
```
Here's how things look when we loop through these lines on our example list:
```
# initial state

P    C    A            
*    a -> b -> c -> d -> *

# first loop iteration

P    C    A
* <- a    b -> c -> d -> *   # C.next = P

    PC    A
* <- a    b -> c -> d -> *   # P = C

     P   CA
* <- a    b -> c -> d -> *   # C = A

     P    C    A
* <- a    b -> c -> d -> *   # A = C.next

# second loop iteration

     P    C    A
* <- a <- b    c -> d -> *   # C.next = P

         PC    A
* <- a <- b    c -> d -> *   # P = C

          P   CA
* <- a <- b    c -> d -> *   # C = A

          P    C    A
* <- a <- b    c -> d -> *   # A = C.next

# more iterations...

# final state
                    P   CA
* <- a <- b <- c <- d -> *
```

### Finishing touches

The code so far gives us this sketch of a solution:

```
def reverseList(L):
    P = None
    C = L.head
    A = L.head.next
    while not done:
        C.next = P
        P = C
        C = A
        A = C.next
```

To make this a working function, we need a few final tweaks:

1. Figure out when we're done. Based on the final state above, we need to keep going until `C` points to the null at the end of the list.

2. When `C` points to null, `A = C.next` will throw an error. So we have to check if `C` is null before doing that assignment.

3. After we finish, set the reversed list's new head to `P`.

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


