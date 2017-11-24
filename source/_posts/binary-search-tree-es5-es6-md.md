title: 二叉搜索树的简明实现（ES5 & ES6）
date: 2017-11-24 19:57:48
comments: true
tags: [数据结构, 算法, BST]
categories: 编程
---

## 二叉树 & 二叉搜索树

二叉树（Binary Tree）是 n（n >= 0）是个节点的有限集合，集合为空集时，叫作空二叉树；不为空时，由根节点及左子树、右子树组成，左子树、右子树也都是二叉树。

从这个描述，可以看出树的结构与递归之间存在密切关系，这种密切关系在树的遍历时能够得到充分体现。

二叉搜索树（Binary Search Tree），又叫二叉查找树；也称为有序二叉树（Ordered Binary Tree），排序二叉树（Sorted Binary Tree）。

这是维基百科上归纳的一些二叉搜索树的性质：

1. 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
2. 若任意节点的右子树不空，则右子树上所有节点的值均大于它的根节点的值；
3. 任意节点的左、右子树也分别为二叉查找树；
4. 没有键值相等的节点。
  
本次实现中有点不一样的地方，右节点是大于或等于父节点的。不过对示例没有影响，而且很容易改成只能大于父节点。<!--more-->


## 关于《学习JavaScript数据结构与算法（第2版）》

当年看过这本书的第一版，最近打算复习一下数据结构与算法，于是看了第二版。

这是难得的一本用 JavaScript 实现的数据结构与算法的书，它的讲解十分清晰，相对来说质量较高，但问题也有很多。应该说第一版的内容还是比较可靠的，第二版新增的内容可靠性就差了很多。总的来说，这是一本从非常浅显的层面讲解数据结构与算法的书，想获得比较全面的知识还是需要阅读更专业的资料。

本文的 ES5 实现参考了这本书，因为我觉得它是比较工整的实现，用于学习和理解时，好过我看到的其他一些实现方式。在原书示例代码的基础上，我做了一些小调整。本书第二版号称拥抱 ES6，但我看过之后发现至少树这一章没有改为 ES6 的实现，于是自己写了一遍，正好当作练习的机会。

另外，这本书中提到的树的遍历方式包括**中序、先序、后序遍历**，这些都属于**深度优先遍历**。在本文的代码中，我补充了**广度优先遍历**以及**按照层次遍历二叉树**的实现。


## Binary Search Tree - ES5

```javascript
var BinarySearchTree = function() {
  var Node = function(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  };

  var root = null;

  var insertNode = function(node, newNode) {
    if (newNode.key < node.key) {
      if (node.left === null) {
        node.left = newNode;
      } else {
        insertNode(node.left, newNode);
      }
    } else {
      if (node.right === null) {
        node.right = newNode;
      } else {
        insertNode(node.right, newNode);
      }
    }
  };

  var inOrderTraverseNode = function(node, cb) {
    if (node !== null) {
      inOrderTraverseNode(node.left, cb);
      cb(node.key);
      inOrderTraverseNode(node.right, cb);
    }
  };

  var preOrderTraverseNode = function(node, cb) {
    if (node !== null) {
      cb(node.key);
      preOrderTraverseNode(node.left, cb);
      preOrderTraverseNode(node.right, cb);
    }
  };

  var postOrderTraverseNode = function(node, cb) {
    if (node !== null) {
      postOrderTraverseNode(node.left, cb);
      postOrderTraverseNode
    }
  };

  var levelOrderTraverseNode = function(node, cb) {
    if (node === null) {
      return null;
    }

    var list = [node];

    while (list.length > 0) {
      node = list.shift();
      cb(node.key);
      if (node.left) {
        list.push(node.left);
      }
      if (node.right) {
        list.push(node.right);
      }
    }
  };

  var separateByLevelFn = function(node, cb, separator) {
    var list = [];
    var END_FLAG = 'END_FLAG';

    list.push(node);
    list.push(END_FLAG);

    separator = separator || '---*---';

    while (list.length > 0) {
      node = list.shift();

      if (node === END_FLAG && list.length > 0) {
        list.push(END_FLAG);
        cb(separator);
      } else {
        cb(node.key);

        if (node.left) {
          list.push(node.left)
        }

        if (node.right) {
          list.push(node.right);
        }
      }
    }
  };

  var minNode = function(node) {
    if (node) {
      while (node.left !== null) {
        node = node.left;
      }

      return node.key;
    }

    return null;
  };

  var maxNode = function(node) {
    if (node) {
      while (node.right !== null) {
        node = node.right;
      }

      return node.key;
    }

    return null;
  };

  var searchNode = function(node, val) {
    if (node === null) {
      return false;
    }

    if (val < node.key) {
      return searchNode(node.left, val);
    } else if (val > node.key) {
      return searchNode(node.right, val);
    } else {
      return true;
    }
  };

  var findMinNode = function(node) {
    if (node) {
      while (node.left !== null) {
        node = node.left;
      }

      return node;
    }

    return null;
  };

  var removeNode = function(node, key) {
    if (node === null) {
      return null;
    }

    if (key < node.key) {
      node.left = removeNode(node.left, key);
      return node;
    } else if (key > node.key) {
      node.right = removeNode(node.right, key);
      return node;
    } else {
      if (node.left === null && node.right === null) {
        node = null;
        return node;
      }

      if (node.left === null) {
        node = node.right;
        return node;
      }

      if (node.right === null) {
        node = node.left;
        return node;
      }

      var aux = findMinNode(node.right);
      node.key = aux.key;
      node.right = removeNode(node.right, aux.key);
      return node;
    }
  };

  this.insert = function(key) {
    var newNode = new Node(key);

    if (root === null) {
      root = newNode;
    } else {
      insertNode(root, newNode);
    }
  };

  // 中序遍历是一种以上行顺序访问BST所有节点的遍历方式，也就是以从最小到最大的顺序访
  // 问所有节点。中序遍历的一种应用就是对树进行排序操作。
  this.inOrderTraverse = function(cb) {
    inOrderTraverseNode(root, cb);
  };

  // 先序遍历是以优先于后代节点的顺序访问每个节点的。先序遍历的一种应用是打印一个结构化的文档。
  this.preOrderTraverse = function(cb) {
    preOrderTraverseNode(root, cb);
  };

  // 后序遍历则是先访问节点的后代节点，再访问节点本身。后序遍历的一种应用是计算一个目
  // 录和它的子目录中所有文件所占空间的大小。
  this.postOrderTraverse = function(cb) {
    postOrderTraverseNode(root, cb);
  };

  // Breadth-First-Search
  // 可以用来解决寻路径的问题。
  this.levelOrderTraverse = function(cb) {
    levelOrderTraverseNode(root, cb);
  }

  // Breadth-First-Search
  // 区分层次
  this.separateByLevel = function(cb) {
    separateByLevelFn(root, cb);
  }

  this.min = function() {
    return minNode(root);
  };

  this.max = function() {
    return maxNode(root);
  };

  this.search = function(val) {
    searchNode(root, val);
  };

  this.remove = function(key) {
    root = removeNode(root, key);
  };
};


/* ========== test case ========== */


var tree = new BinarySearchTree();

/**
 *
 *               11
 *              /  \
 *             /    \
 *            /      \
 *           /        \
 *          /          \
 *         /            \
 *        7              5
 *       / \           /   \
 *      /   \         /     \
 *     5     9       13     20
 *    / \   / \     /  \   /  \
 *   3   6 8  10   12  14 18  25
 *
 */
tree.insert(11);
tree.insert(7);
tree.insert(15);
tree.insert(5);
tree.insert(3);
tree.insert(9);
tree.insert(8);
tree.insert(10);
tree.insert(13);
tree.insert(12);
tree.insert(14);
tree.insert(20);
tree.insert(18);
tree.insert(25);
tree.insert(6);

var printNode = function(val) {
  console.log(val);
};

tree.inOrderTraverse(printNode);

console.log('\n')
tree.levelOrderTraverse(printNode);

console.log('\n')
tree.separateByLevel(printNode);

console.log('\n')
tree.remove(7)
tree.inOrderTraverse(printNode);

console.log('\n')
tree.preOrderTraverse(printNode);

console.log('\n')
tree.postOrderTraverse(printNode);
```


## Binary Search Tree - ES6

```javascript
class Node {
  constructor(key) {
    this.key = key;
    this.left = null;
    this.right = null;
  }
}

// insert(key):向树中插入一个新的键。
// search(key):在树中查找一个键，如果节点存在，则返回true;如果不存在，则返回false。
// inOrderTraverse:通过中序遍历方式遍历所有节点。
// preOrderTraverse:通过先序遍历方式遍历所有节点。
// postOrderTraverse:通过后序遍历方式遍历所有节点。
// min:返回树中最小的值/键。
// max:返回树中最大的值/键。
// remove(key):从树中移除某个键。
class BinarySearchTree {

  constructor() {
    this.root = null;
  }

  static insertNode(node, newNode) {
    if (node.key > newNode.key) {
      if (node.left === null) {
        node.left = newNode;
      } else {
        BinarySearchTree.insertNode(node.left, newNode);
      }
    } else {
      if (node.right === null) {
        node.right = newNode;
      } else {
        BinarySearchTree.insertNode(node.right, newNode);
      }
    }
  }

  static searchNode(node, key) {
    if (node === null) {
      return false;
    }

    if (node.key === key) {
      return true;
    } else if (node.key > key) {
      BinarySearchTree.searchNode(node.left, key);
    } else if (node.key < key) {
      BinarySearchTree.searchNode(node.right, key);
    }
  }

  static inOrderTraverseNode(node, cb) {
    if (node === null) {
      return;
    }
    BinarySearchTree.inOrderTraverseNode(node.left, cb);
    cb(node.key);
    BinarySearchTree.inOrderTraverseNode(node.right, cb);
  }

  static preOrderTraverseNode(node, cb) {
    if (node === null) {
      return;
    }
    cb(node.key);
    BinarySearchTree.preOrderTraverseNode(node.left, cb);
    BinarySearchTree.preOrderTraverseNode(node.right, cb);
  }

  static postOrderTraverseNode(node, cb) {
    if (node === null) {
      return;
    }
    BinarySearchTree.postOrderTraverseNode(node.left, cb);
    BinarySearchTree.postOrderTraverseNode(node.right, cb);
    cb(node.key);
  }

  static levelOrderTraverseNode(node, cb) {
    if (node === null) {
      return null;
    }

    const list = [node];

    while (list.length > 0) {
      node = list.shift();
      cb(node.key);
      if (node.left) {
        list.push(node.left);
      }
      if (node.right) {
        list.push(node.right);
      }
    }
  }

  static separateByLevelFn(node, cb, separator = '---*---') {
    const list = [];
    const END_FLAG = 'END_FLAG';

    list.push(node);
    list.push(END_FLAG);

    while (list.length > 0) {
      node = list.shift();

      if (node === END_FLAG && list.length > 0) {
        list.push(END_FLAG);
        cb(separator);
      } else {
        cb(node.key);

        if (node.left) {
          list.push(node.left)
        }

        if (node.right) {
          list.push(node.right);
        }
      }
    }
  }

  static removeNode(node, key) {
    if (node === null) {
      return null;
    }

    if (node.key === key) {

      if (node.left === null && node.right === null) {
        node = null;
        return node;
      } else if (node.left === null) {
        node = node.right;
        return node;
      } else if (node.right === null) {
        node = node.left;
        return node;
      } else if (node.left && node.right) {
        let rightMinNode = node.right;

        while (rightMinNode.left !== null) {
          rightMinNode = rightMinNode.left;
        }

        node.key = rightMinNode.key;
        node.right = BinarySearchTree.removeNode(node.right, rightMinNode.key);
        return node;
      }

    } else if (node.key > key) {
      node.left = BinarySearchTree.removeNode(node.left, key);
      return node;
    } else if (node.key < key) {
      node.right = BinarySearchTree.removeNode(node.right, key);
      return node;
    }
  }

  static printNode(val) {
    console.log(val);
  }

  insert(key) {
    const newNode = new Node(key);

    if (this.root === null) {
      this.root = newNode;
    } else {
      BinarySearchTree.insertNode(this.root, newNode);
    }
  }

  search(key) {
    return BinarySearchTree.searchNode(key);
  }

  // 中序遍历是一种以上行顺序访问BST所有节点的遍历方式，也就是以从最小到最大的顺序访
  // 问所有节点。中序遍历的一种应用就是对树进行排序操作。
  inOrderTraverse(cb = BinarySearchTree.printNode) {
    BinarySearchTree.inOrderTraverseNode(this.root, cb);
  }

  // 先序遍历是以优先于后代节点的顺序访问每个节点的。先序遍历的一种应用是打印一个结构化的文档。
  preOrderTraverse(cb = BinarySearchTree.printNode) {
    BinarySearchTree.preOrderTraverseNode(this.root, cb);
  }

  // 后序遍历则是先访问节点的后代节点，再访问节点本身。后序遍历的一种应用是计算一个目
  // 录和它的子目录中所有文件所占空间的大小。
  postOrderTraverse(cb = BinarySearchTree.printNode) {
    BinarySearchTree.postOrderTraverseNode(this.root, cb);
  }

  // Breadth-First-Search
  // 可以用来解决寻路径的问题。
  levelOrderTraverse(cb = BinarySearchTree.printNode) {
    BinarySearchTree.levelOrderTraverseNode(this.root, cb);
  }

  // Breadth-First-Search
  // 区分层次
  separateByLevel(cb = BinarySearchTree.printNode) {
    BinarySearchTree.separateByLevelFn(this.root, cb);
  }

  min() {
    let node = this.root;

    if (node === null) {
      return null;
    }

    while (node.left !== null) {
      node = node.left;
    }

    return node.key;
  }

  max() {
    let node = this.root;

    if (node === null) {
      return null;
    }

    while (node.right !== null) {
      node = node.right;
    }

    return node.key();
  }

  remove(key) {
    this.root = BinarySearchTree.removeNode(this.root, key);
  }
}


/* ========== test case ========== */


const tree = new BinarySearchTree();

/**
 *
 *               11
 *              /  \
 *             /    \
 *            /      \
 *           /        \
 *          /          \
 *         /            \
 *        7              5
 *       / \           /   \
 *      /   \         /     \
 *     5     9       13     20
 *    / \   / \     /  \   /  \
 *   3   6 8  10   12  14 18  25
 *
 */
tree.insert(11);
tree.insert(7);
tree.insert(15);
tree.insert(5);
tree.insert(3);
tree.insert(9);
tree.insert(8);
tree.insert(10);
tree.insert(13);
tree.insert(12);
tree.insert(14);
tree.insert(20);
tree.insert(18);
tree.insert(25);
tree.insert(6);

tree.inOrderTraverse();

console.log('\n')
tree.levelOrderTraverse();

console.log('\n')
tree.separateByLevel();

console.log('\n')
tree.remove(7)
tree.inOrderTraverse();

console.log('\n')
tree.preOrderTraverse();

console.log('\n')
tree.postOrderTraverse();
```


