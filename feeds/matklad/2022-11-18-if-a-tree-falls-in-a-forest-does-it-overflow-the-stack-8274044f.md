---
title: If a Tree Falls in a Forest, Does It Overflow the Stack?
url: https://matklad.github.io/2022/11/18/if-a-tree-falls-in-a-forest-does-it-overflow-the-stack.html
published: "2022-11-18T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2022/11/18/if-a-tree-falls-in-a-forest-does-it-overflow-the-stack.html
---

# If a Tree Falls in a Forest, Does It Overflow the Stack?

Nov 18, 2022

A well-known pitfall when implementing a linked list in Rust is that the the default recursive `drop` implementation causes stack overflow for long lists. A similar problem exists for tree data structures as well. This post describes a couple of possible solutions for trees. This is a rather esoteric problem, so the article is denser than is appropriate for a tutorial.

Let’s start with our beloved linked list:

```
struct Node {
  value: T,
  next: Option<Box>>,
}

impl Node {
  fn new(value: T) -> Node {
    Node { value, next: None }
  }
  fn with_next(mut self, next: Node) -> Node {
    self.next = Some(Box::new(next));
    self
  }
}
```

It’s easy to cause this code to crash:

```
#[test]
fn stack_overflow() {
  let mut node = Node::new(0);
  for _ in 0..100_000 {
    node = Node::new(0).with_next(node);
  }
  drop(node) // boom
}
```

The crash happens in the automatically generated recursive `drop` function. The fix is to write `drop` manually, in a non-recursive way:

```
impl Drop for Node {
  fn drop(&mut self) {
    while let Some(next) = self.next.take() {
      *self = *next;
    }
  }
}
```

What about trees?

```
struct Node {
  value: T,
  left: Option<Box>>,
  right: Option<Box>>,
}
```

If the tree is guaranteed to be balanced, the automatically generated drop is actually fine, because the height of the tree will be logarithmic. If the tree is unbalanced though, the same stack overflow might happen.

Let’s write an iterative `Drop` to fix this. The problem though is that the “swap with `self`” trick we used for list doesn’t work, as we have two children to recur into. The standard solution would be to replace a stack with an explicit vector of work times:

```
impl Drop for Node {
  fn drop(&mut self) {
    let mut work = Vec::new();
    work.extend(self.left.take());
    work.extend(self.right.take());
    while let Some(node) = work.pop() {
      work.extend(node.left.take());
      work.extend(node.right.take());
    }
  }
}
```

This works, but also makes my internal C programmer scream: we allocate a vector to free memory! Can we do better?

One approach would be to build on balanced trees observation. If we recur into the shorter branch, and iteratively drop the longer one, we should be fine:

```
impl Drop for Node {
  fn drop(&mut self) {
    loop {
      match (self.left.take(), self.right.take()) {
        (None, None) => break,
        (None, Some(it)) | (Some(it), None) => *self = *it,
        (Some(left), Some(right)) => {
          *self =
            *if left.depth > right.depth { left } else { right }
        }
      }
    }
  }
}
```

This requires maintaining the depths though. Can we make do without? My C instinct (not that I wrote any substantial amount of C though) would be to go down the tree, and stash the parent links into the nodes themselves. And we actually can do something like that:

- If the current node has only a single child, we can descend into the node

- If there are two children, we can rotate the tree. If we always rotate into a single direction, eventually we’ll get into the single-child situation.

Here’s how a single rotation could look:

![](https://user-images.githubusercontent.com/1711539/202797128-87e40cf0-be55-44b3-9bdf-5dc15b33812b.png)

Or, in code,

```
impl Drop for Node {
  fn drop(&mut self) {
    loop {
      match (self.left.take(), self.right.take()) {
        (None, None) => break,
        (None, Some(it)) | (Some(it), None) => *self = *it,
        (Some(mut left), Some(right)) => {
          mem::swap(self, &mut *left);
          left.left = self.right.take();
          left.right = Some(right);
          self.right = Some(left);
        }
      }
    }
  }
}
```

Ok, what if we have an n-ary tree?

```
struct Node {
  value: T,
  children: Vec>,
}
```

I *think* the same approach works: we can treat the first child as `left`, and the last child as `right`, and do essentially the same rotations. Though, we will rotate in other direction (as removing the right child is cheaper), and we’ll also check that we have at least two grandchildren (to avoid allocation when pushing to an empty vector).

Which gives something like this:

```
impl Drop for Node {
  fn drop(&mut self) {
    loop {
      let Some(mut right) = self.children.pop() else {
        break;
      };
      if self.children.is_empty() {
        *self = right;
        continue;
      }
      if right.children.len() < 2 {
        self.children.extend(right.children.drain(..));
        continue;
      }
      // Non trivial case:
      //   >= 2 children,
      //   >= 2 grandchildren.
      let mut me = mem::replace(self, right);
      mem::swap(&mut self.children[0], &mut me);
      // Doesn't allocate, this is the same slot
      // we popped from at the start of the loop.
      self.children[0].children.push(me);
    }
  }
}
```

I am not sure this works, and I am not sure this works in linear time, but I am fairly certain that something like this could be made to work if need be.

Though, practically, if something like this is a concern, you probably want to re-design the tree structure to be something like this instead:

```
struct Node {
  value: T,
  children: Range<usize>,
}

struct Tree {
   nodes: Vec>,
}
```

**Update(2023-11-03):** Apparently, dropping trees iteratively was trendy in Dec. 2022! The same idea was described by Ismail in this great post:

https://ismailmaj.github.io/destructing-trees-safely-and-cheaply
