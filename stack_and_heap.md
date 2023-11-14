### What are the stack and the heap
Stack and the heap are both memory available to your code to use during runtime. They differ in terms of their structure. 
#### Stack
The stack stores objects in the order they are created and destroys them in the reverse order (LIFO). 
Adding values onto the stack is referred to as pushing into the stack.
Removing values from the stack is referred to as popping off the stack.
The stack cannot cope with values that are of unknown size at compile time and shouldn't be used to hold values that's size changes during run time. These must be instantiated on the heap.

#### Heap
The heap is less organised than the stack. Instead of placing things in order the heap simply finds and allocates memory that will fit your object regardless of where it is on the heap. The programme then manipulates this value using a ptr (think cpp's new).  
Adding values on the heap is referred to as allocating to the heap. 

### Contrasts
- Pushing to the stack is faster than allocating on the heap. Intuitive, it's easier to drop a plate on a stack than it is to find a table with space for your plate and then come back and tell someone where that plate has been put. 
- Accessing the data on the heap is also slower than accessing data on the stack. Think of it as having to run around trying to find where data is rather than flicking through a stack of items. 
