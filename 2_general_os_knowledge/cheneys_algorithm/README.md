# Cheney's algorithm

## [Video 1](https://www.youtube.com/watch?v=S1xvtHsarbI)
## [Video 2](https://www.youtube.com/watch?v=jYt3uQtbLfE)

**Cheney's algorithm**, first described in a 1970 ACM paper by C.J. Cheney, is a method of garbage collection in computer software systems.
In this scheme, the heap is divided into two equal halves, only one of which is in use at any time. Garbage collection collection is performed be copying live objects from one semispave to other, which then becomes the new heap.

Cheney's algorithm reclaims items as follows: 
- ***Object references on the stack*** Object references on the stack are checked. One of the two following actions is taken for each object reference that points to an object in from-space:
    - If the object has not yet been moved to the to-space, this is done by creating identical copy in the to-space, and then replacing the from-space version with a forwarding pointer ot the to-space. Then update the object reference to refer to the new version in to-space.
    - If the object has already moved to the to-space, simply update the reference from the forwarding pointer in from-space

- ***Objects in the to-space***. The garbage collector examines all object references in the objects that have been migrated to the to-space, and performs one of the above two actions on the referenced object
