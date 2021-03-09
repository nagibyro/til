# Loop Detection in Linked List
This isn't a defintion of how to find a loop in a linked list.You can find plenty of examples around the internet.
Instead this is a explination of why one of the techniques work better than others.

## Bankground / Context
I implemented a loop detector that returned the node that the loop started with [here](https://gist.github.com/Nilithus/eaac46e538e599bcfd965d6007061e17)
but I found out that it was slower than most other implementations but I didn't understand why. Basically my
implementation cycled through to find the loop; which gave me the total loop length -- but not the starting node.
Then to figure out where the loop started I reset to the start of the list with both pointers and then for each 
node in the list I would advance a pointer the length of the loop. then compare if the two pointers were the 
same node. If they were then that node was the start.

## How to make it more efficient 
Faster implementations did not track the length of the loop. Instead when the loop was detected by the two pointers.
The first pointer was reset to the head of the list. but the second pointer was **not** moved. Then the program looped forward over the list, moving both pointers at the same speed(1 node) until they both pointed at the same node. That node
was the start of the loop.

This didn't make very much sense to me. How do we know that the distance from the start of the list to the start of the 
loop is the exact same the distance that the second pointer needs to go around the loop? The answer is pretty much
math. Using two pointers to detect loops is called [Floyd's Tortoise and Hare algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_Tortoise_and_Hare)
. Essentially there is a mathmatical proof that the distance from the start of the list to the start of the loop
will be the number of nodes left in the loop for the second pointer.
[This youtube video](https://www.youtube.com/watch?v=LUm2ABqAs1w) goes into the proof. The more optimized version
would looks like

```ts
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     val: number
 *     next: ListNode | null
 *     constructor(val?: number, next?: ListNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.next = (next===undefined ? null : next)
 *     }
 * }
 */

function detectCycle(head: ListNode | null): ListNode | null {
  let fast = head;
  let slow = head;

  while (fast && fast.next) {
    fast = fast.next.next;
    slow = slow.next;
    if (fast === slow) {
      break;
    }
  }

  // null => no cycle
  if (!fast || !fast.next) {
    return null;
  }

  // make one of fast and slow reset into head
  fast = head;
  while (slow !== fast) {
    // make speed equal
    fast = fast.next;
    slow = slow.next;
  }

  return fast;
}
```

Figuring this out in the moment without knowledge of proofs is probably unrealistic so it's really just a fact that you
need to memorize about loop detection in linked lists.
