# 剑指Offer（三）

### [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

看到这道题后，我的第一想法就是再创建一个数组，但是又没有更好的方法呢，答案肯定是有的。

我们可以直接使用双指针，通过交换就可以实现在空间复杂度为O（1）的情况下求解这种问题

```java
public class _剑指_Offer_21_调整数组顺序使奇数位于偶数前面 {
    public int[] exchange(int[] nums) {
        int[] ints = new int[nums.length];
        int l = 0;
        int r = nums.length - 1;
        for (int i = 0;i < nums.length;i++){
            if (nums[i] % 2 != 0){
                ints[l++] = nums[i];
            }else {
                ints[r--] = nums[i];
            }
        }
        return ints;
    }

    public int[] exchange1(int[] nums) {
        int l = 0;
        int r = nums.length - 1;

        while (r > l){
            if (nums[l] % 2 == 0 && nums[r] % 2 == 1){
                int i = nums[l];
                nums[l] = nums[r];
                nums[r] = i;
                l++;
                r--;
            }else if (nums[l] % 2 == 0){
                r--;
            }else if (nums[r] % 2 == 1){
                l++;
            }else {
                l++;
                r--;
            }
        }
        return nums;
    }
}
```

### [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

首先我们先遍历一遍，计算出它的元素个数，然后很简单就可以找到。

```java
public class _剑指_Offer_22_链表中倒数第k个节点 {
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null) return null;

        ListNode head1 = head;
        int size = 0;
        while (head1 != null){
            size++;
            head1 = head1.next;
        }

        if (k > size) return null;
        head1 = head;
        for (int i = 0;i < size - k;i++){
            head1 = head1.next;
        }
        return head1;
    }
}
```

那么有没有办法遍历一次就能找到呢？

```java
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null) return null;
        
        ListNode fast = head;
        ListNode slow = head;
        
        for (int i = 1;i <= k;i++){
            fast = fast.next;
        }
        
        while (fast != null){
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}
```

我们可以使用快慢指针，先让快指针next k次，然后再让两个指针一起走，等到快指针为空，慢指针也就是我们需要的答案了。

### [面试题 02.08. 环路检测](https://leetcode-cn.com/problems/linked-list-cycle-lcci/)

这道题是我们原先环形链表的变种

```java
public ListNode detectCycle(ListNode head) {
        if (head == null) return null;
        if (head.next == null) return null;
        ListNode fast = head.next.next;
        ListNode slow = head.next;
        while (fast != null && fast.next != null){
            if (fast == slow){
                break;
            }
            fast = fast.next.next;
            slow = slow.next;
        }
        if (fast != slow){
            return null;
        }
        slow = head;
        while (fast != slow){
            fast = fast.next;
            slow = slow.next;
        }
        return fast;
    }
```



### [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

不说了，闭着眼睛也应该能写出来

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head == null) return null;
        
        ListNode prev = null;
        
        while (head != null){
            ListNode next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
}
```

### [剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) return l2;
        if (l2 == null) return l1;

        ListNode hair = new ListNode(0);
        ListNode hair1 = hair;

        while (l1 != null && l2 != null){
            if (l1.val < l2.val){
                hair1.next = l1;
                l1 = l1.next;
            }else {
                hair1.next = l2;
                l2 = l2.next;
            }
            hair1 = hair1.next;
        }
        
        if (l1 != null){
            hair1.next = l1;
            return hair.next;
        }
        if (l2 != null){
            hair1.next = l2;
            return hair.next;
        }
        return hair.next;
    }
}
```

### [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

一看到树，我们自然就想到了递归

```java
class Solution {
   public boolean isSubStructure(TreeNode A, TreeNode B) {
        return ((A != null && B != null) && (diGui(A,B) || isSubStructure(A.left,B) || isSubStructure(A.right,B)));
    }

    private boolean diGui(TreeNode a, TreeNode b){
        if (a == null && b == null) return true;
        if (a == null) return false;
        if (b == null) return true;
        if (a.val == b.val) return diGui(a.left,b.left) && diGui(a.right,b.right);
        return false;
    }
}
```

### [剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

```java
class Solution {
    public TreeNode mirrorTree(TreeNode root) {
        if (root == null) return null;
        
        TreeNode temp = root.left;
        root.left = mirrorTree(root.right);
        root.right = mirrorTree(temp);
        
        return root;
    }
}
```

### [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

```java
class Solution {
     public boolean isSymmetric(TreeNode root) {
        if (root == null) return true;
        return recursion(root.left,root.right);
    }
    private boolean recursion(TreeNode left,TreeNode right){
        if (left == null && right == null) return true;
        if (left == null || right == null) return false;
        return left.val == right.val && recursion(left.left,right.right) && recursion(left.right,right.left);
    }
}
```

### [剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```java
class Solution {
    public int[] spiralOrder(int[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) return new int[]{};
        int[] ints = new int[matrix.length * matrix[0].length];
        int i = 0;
        int l = 0,r = matrix[0].length - 1,t = 0,s = matrix.length - 1;
        while (l <= r && t <= s){
            for (int col = l;col <= r;col++){
                ints[i++] = matrix[t][col];
            }
            t++;
            for (int row = t;row <= s;row++){
                ints[i++] = matrix[row][r];
            }
            r--;
            if (l <= r && t <= s){
                for (int col = r;col >= l;col--){
                    ints[i++] = matrix[s][col];
                }
                s--;
                for (int row = s;row >= t;row--){
                    ints[i++] = matrix[row][l];
                }
                l++;
            }
        }
        return ints;
    }
}
```

### [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

```java
public class MinStack {
    private Stack<Integer> stack1;
    private Stack<Integer> stack2;
    public MinStack() {
        stack1 = new Stack();
        stack2 = new Stack();
    }

    public void push(int x) {
        stack1.push(x);
        if (!stack2.isEmpty() && stack2.peek() <= x){
            stack2.push(stack2.peek());
        }else {
            stack2.push(x);
        }
    }

    public void pop() {
        stack1.pop();
        stack2.pop();
    }

    public int top() {
        stack2.pop();
        return stack1.pop();
    }

    public int min() {
        return stack2.peek();
    }
}
```

