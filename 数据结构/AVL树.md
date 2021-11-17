# AVL树

在上一节学完二叉搜索树后，我们将其搜索元素的复杂度从链表的O（n）降到了O（logn），嗯。。。非常棒

好的，树这一块儿咱们就讲完......咳咳还没完呢，请大家来设想这样的一种情况。

把1-10按顺序添加到树中，最后得出个什么玩意儿呢

1右2右3右4右5右6右7右8右9右10

唉不对，这不是链表吗？

那该怎么去解决呢，有些同学很聪明啊，小聪明啊 咱们把添加的顺序打乱不就好了吗，这种。。。

好了好了，不说废话了咱们一起来看如何解决。

首先咱们引入**平衡二叉树** 意思很明显，就是节点的左右子树高度不能相差过大

那么怎么让他不会相差过大呢

咱们又要引出树的旋转

旋转分为两种

1. 左旋

   （将1左旋）

   ![](https://img.imgdb.cn/item/5ffa89ca3ffa7d37b33b72d4.jpg)

   可以看到结果非常的amazing啊，树的高度从4降到了3（变胖了，像不像过年时的你（狗头））

   总而言之，平衡二叉树的核心就是这个

2. 右旋同理（大家自己琢磨琢磨）



讲完核心就该讲具体的操作了，还记得protected void afterRemove(Node<E> node,Node renode) {}这个吗

现在就用上了，咱们的节点现在还得多加一个属性来记录他的左右子树的相减的绝对值当这个值等于了2，就认为这个节点失衡了。例如

1右2右3

咱们就说1失衡了，因为左边子树高度为0，而右边为2

咱们来举一个小例子

![](https://img.imgdb.cn/item/5ffa8f083ffa7d37b33d4b59.jpg)

咱们从1添加到10

![](https://img.imgdb.cn/item/5ffa95803ffa7d37b33fb88f.jpg)

下面给出代码

```java
package 二叉树;
import 二叉树.二叉搜索树;

import java.util.Comparator;

public class AVLTree<E> extends 二叉搜索树<E> {
    public AVLTree() {
        this(null);
    }

    public AVLTree(Comparator<E> comparator) {
        super(comparator);
    }

    @Override
    protected void afteradd(Node node) {
        while ((node = node.parent) != null) {
            if (isBalanced(node)) {
                // 更新高度
                updateHeight(node);
            } else {
                // 恢复平衡
                rebalance(node);
                // 整棵树恢复平衡
                break;
            }
        }
    }

    @Override
    protected void afterRemove(Node<E> node,Node renode) {
        while ((node = node.parent) != null) {
            if (isBalanced(node)) {
                // 更新高度
                updateHeight(node);
            } else {
                // 恢复平衡
                rebalance(node);
            }
        }
    }

    @Override
    protected Node<E> createNode(E element, Node<E> parent) {
        return new AVLNode<>(element, parent);
    }

    /**
     * 恢复平衡
     * @param grand 高度最低的那个不平衡节点
     */
    @SuppressWarnings("unused")
    private void rebalance2(Node<E> grand) {
        Node<E> parent = ((AVLNode<E>)grand).tallerChild();
        Node<E> node = ((AVLNode<E>)parent).tallerChild();
        if (((AVLNode<E>)parent).isLeftChild()) { // L
            if (((AVLNode<E>)node).isLeftChild()) { // LL
                rotateRight(grand);
            } else { // LR
                rotateLeft(parent);
                rotateRight(grand);
            }
        } else { // R
            if (((AVLNode<E>)node).isLeftChild()) { // RL
                rotateRight(parent);
                rotateLeft(grand);
            } else { // RR
                rotateLeft(grand);
            }
        }
    }
    /**
     * 恢复平衡
     * @param grand 高度最低的那个不平衡节点
     */
    private void rebalance(Node<E> grand) {
        AVLNode<E> parent = (AVLNode<E>) ((AVLNode<E>)grand).tallerChild();
        AVLNode<E> node = (AVLNode<E>) ((AVLNode<E>)parent).tallerChild();
        if (parent.isLeftChild()) { // L
            if (node.isLeftChild()) { // LL
                rotate(grand, node, node.right, parent, parent.right, grand);
            } else { // LR
                rotate(grand, parent, node.left, node, node.right, grand);
            }
        } else { // R
            if (node.isLeftChild()) { // RL
                rotate(grand, grand, node.left, node, node.right, parent);
            } else { // RR
                rotate(grand, grand, parent.left, parent, node.left, node);
            }
        }
    }

    private void rotate(
            Node<E> r, // 子树的根节点
            Node<E> b, Node<E> c,
            Node<E> d,
            Node<E> e, Node<E> f) {
        // 让d成为这棵子树的根节点
        d.parent = r.parent;
        if (((AVLNode<E>)r).isLeftChild()) {
            r.parent.left = d;
        } else if (((AVLNode<E>)r).isRightChild()) {
            r.parent.right = d;
        } else {
            root = d;
        }

        //b-c
        b.right = c;
        if (c != null) {
            c.parent = b;
        }
        updateHeight(b);

        // e-f
        f.left = e;
        if (e != null) {
            e.parent = f;
        }
        updateHeight(f);

        // b-d-f
        d.left = b;
        d.right = f;
        b.parent = d;
        f.parent = d;
        updateHeight(d);
    }

    private void rotateLeft(Node<E> grand) {
        Node<E> parent = grand.right;
        Node<E> child = parent.left;
        grand.right = child;
        parent.left = grand;
        afterRotate(grand, parent, child);
    }

    private void rotateRight(Node<E> grand) {
        Node<E> parent = grand.left;
        Node<E> child = parent.right;
        grand.left = child;
        parent.right = grand;
        afterRotate(grand, parent, child);
    }

    private void afterRotate(Node<E> grand, Node<E> parent, Node<E> child) {
        // 让parent称为子树的根节点
        parent.parent = grand.parent;
        if (((AVLNode<E>)grand).isLeftChild()) {
            grand.parent.left = parent;
        } else if (((AVLNode<E>)grand).isRightChild()) {
            grand.parent.right = parent;
        } else { // grand是root节点
            root = parent;
        }

        // 更新child的parent
        if (child != null) {
            child.parent = grand;
        }

        // 更新grand的parent
        grand.parent = parent;

        // 更新高度
        updateHeight(grand);
        updateHeight(parent);
    }

    private boolean isBalanced(Node<E> node) {
        return Math.abs(((AVLNode<E>)node).balanceFactor()) <= 1;
    }

    private void updateHeight(Node<E> node) {
        ((AVLNode<E>)node).updateHeight();
    }

    private static class AVLNode<E> extends Node<E> {
        int height = 1;

        public AVLNode(E element, Node<E> parent) {
            super(element, parent);
        }

        public int balanceFactor() {
            int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
            int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
            return leftHeight - rightHeight;
        }

        public void updateHeight() {
            int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
            int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
            height = 1 + Math.max(leftHeight, rightHeight);
        }

        public Node<E> tallerChild() {
            int leftHeight = left == null ? 0 : ((AVLNode<E>)left).height;
            int rightHeight = right == null ? 0 : ((AVLNode<E>)right).height;
            if (leftHeight > rightHeight) return left;
            if (leftHeight < rightHeight) return right;
            return isLeftChild() ? left : right;
        }

        @Override
        public String toString() {
            String parentString = "null";
            if (parent != null) {
                parentString = parent.element.toString();
            }
            return element + "_p(" + parentString + ")_h(" + height + ")";
        }

        public boolean isLeftChild(){
            if (this.parent != null && this.parent.left == this){
                return true;
            }
            return false;
        }

        public boolean isRightChild(){
            if (this.parent != null && this.parent.right == this){
                return true;
            }
            return false;
        }
    }

}

```

大家一定要自己把代码走一遍。