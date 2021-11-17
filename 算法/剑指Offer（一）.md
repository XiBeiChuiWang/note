# 剑指Offer（一）

- ### [剑指 Offer 03. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

  这道题呢，如果可以改变原数组的话，我们可以比较值与他的下标，如果相同，则跳过，如不同，我们就交换，把当前值放到他该放的位置，如果要放的位置的数和他相等，则直接返回结果。

  例子：[2, 3, 1, 0, 2, 5, 3]

  首先判断2，2的下标为0，不相同在，则与1交换

  2, 3, 1, 0, 2, 5, 3 ——>1, 3, 2, 0, 2, 5, 3 ——>3, 1, 2, 0, 2, 5, 3 ——>0, 1, 2, 3, 2, 5, 3 ——>

  发现前4个数都在自己位置上，跳过

  发现2不在位置上，可是下标为2的位置上的数字也为2，重复了

  时间复杂度O(n)

  ```java
  public class _3_数组中重复的数字 {
      public int findRepeatNumber(int[] nums) {
          if (nums == null || nums.length == 0 || nums.length == 1) return -1;
  
          for (int i = 0;i <= nums.length - 1;i++){
              while (nums[i] != i){
                  if (nums[nums[i]] != nums[i]) {
                      swap(nums,nums[i],i);
                  }else {
                      return nums[i];
                  }
              }
          }
          return -1;
      }
      private void swap(int[] a,int i,int j){
          int x = a[i];
          a[i] = a[j];
          a[j] = x;
      }
  
  }
  ```

- ### [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

  这道题，我们从**右上角**开始找，如果发现右上角这个数比目标值大，则我们直接抛弃这一列，如果比目标值小，我们就抛弃这一行，如果，相同的话，不需要说，直接返回呗。

  ```java
  public class _剑指Offer04_二维数组中的查找 {
      public boolean findNumberIn2DArray(int[][] matrix, int target) {
          if (matrix == null || matrix.length == 0 || matrix[0].length == 0) return false;
          int row = 0;
          int col = matrix[0].length - 1;
  
          while (row < matrix.length && col >= 0){
              if (matrix[row][col] == target) return true;
  
              if (matrix[row][col] > target){
                  col--;
              }else {
                  row++;
              }
          }
          return false;
      }
  }
  ```

  - 时间复杂度：O(n+m)。访问到的下标的行最多增加 `n` 次，列最多减少 `m` 次，因此循环体最多执行 `n + m` 次。

- ### [剑指 Offer 05. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)

  这道题很简单，我们先将字符串遍历一遍，统计出他有几个空格，然后创建一个合适的字符数组，然后遍历原字符串，将其复制到新的字符数组中，注意了，这儿需要我们**从后往前**进行遍历。

  ```java
  public class _剑指Offer_05_替换空格 {
      public String replaceSpace(String s) {
          if (s == null || s.length() == 0) return s;
  
          char[] chars = s.toCharArray();
          int count = 0;
          for (char c:chars){
              if (c == ' ')
                  count++;
          }
  
          char[] chars1 = new char[chars.length + 2 * count];
          int n = chars1.length - 1;
          for (int i = chars.length - 1;i >= 0;i--){
              if (chars[i] == ' '){
                  chars1[n--] = '0';
                  chars1[n--] = '2';
                  chars1[n--] = '%';
              }else {
                  chars1[n--] = chars[i];
              }
          }
          return String.valueOf(chars1);
      }
  }
  ```

- ### [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

  这道题说实话，乍一看，很奇怪，书上说，我们可以用到栈这种数据结构去处理它，可是最后结果呢，很慢，而且耗费资源。

  而我们如果只是简单的遍历一遍，统计出链表的节点个数，再创建一个相应的数组，从后往前的将遍历到的链表元素添加到数组中。

  这第二种方式速度很快，而且逻辑很容易思考，但是我们需要通过这种题去拓宽我们的思路。

  下面给出使用栈的代码

  ```java
  public class _剑指_Offer_06_从尾到头打印链表 {
      public int[] reversePrint(ListNode head) {
          if (head == null) return new int[]{};
  
          Stack<Integer> integers = new Stack<>();
          ListNode head1= head;
          while (head1 != null){
              integers.push(head1.val);
              head1 = head1.next;
          }
          int[] ints = new int[integers.size()];
  
          for (int i = 0;i <= ints.length - 1;i++){
              ints[i] = integers.pop();
          }
          return ints;
      }
  }
  ```

- ### [剑指 Offer 07. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)

  又是这玩意儿，真的烦，嗯，这就是我以前看到这个题的样子，感觉很复杂，因为递归本身就是一个偏向抽象的东西，但是，在静下心来好好做了一次之后，呵，不过如此嘛，所以说，大家一定不能被这种看似很复杂，很抽象的东西吓倒，冲就完了！！！

  这道题的解法看注释

  ```java
  public class _剑指_Offer_07_重建二叉树 {
      private HashMap<Integer, Integer> integerIntegerHashMap;
      public TreeNode buildTree(int[] preorder, int[] inorder) {
          if (preorder == null || preorder.length == 0) return null;
  
          int n = preorder.length;
          //将中序遍历中的每一个节点都放进HashMap，方便我们进行查找
          integerIntegerHashMap = new HashMap<Integer, Integer>();
          for (int i = 0;i < preorder.length;i++){
              integerIntegerHashMap.put(inorder[i],i);
          }
          return buildTree(preorder, inorder,0,n - 1,0,n - 1);
      }
  
      //我们来看看参数 int p_left,int p_right 就是构建一棵子树所需要的先序遍历的数组 int i_left,int i_right为中序遍历
      private TreeNode buildTree(int[] preorder, int[] inorder,int p_left,int p_right,int i_left,int i_right){
          //递归返回条件
          if (p_left > p_right) return null;
          Integer i_root = integerIntegerHashMap.get(preorder[p_left]);
          //构建树的根节点
          TreeNode root = new TreeNode(preorder[p_left]);
          //根据中序遍历算出左子树的节点数
          int left_size = i_root - i_left;
          //递归构建左子树及右子树（核心）
          root.left = buildTree(preorder,inorder,p_left + 1,p_left + left_size,i_left,i_root - 1);
          root.right = buildTree(preorder, inorder,p_left + left_size + 1,p_right,i_root + 1,i_right);
          //返回根节点
          return root;
      }
  }
  ```

  是不是没有想象的难？

- ### [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

  这道题主要考验我们对栈以及队列的理解，本质上也很简单一个栈负责压栈，另一个负责出栈

  ```java
  class CQueue {
  
      private Stack stack1;
      private Stack stack2;
      public CQueue() {
          stack1 = new Stack<Integer>();
          stack2 = new Stack<Integer>();
      }
      
      public void appendTail(int value) {
          stack1.push(value);
      }
      
      public int deleteHead() {
          if (stack2.size() == 0){
              if (stack1.size() != 0){
                  while (!stack1.isEmpty()){
                      stack2.push(stack1.pop());
                  }
                  return (int) stack2.pop();
              }
          }else {
              return (int) stack2.pop();
          }
          return -1;
      }
  }
  ```

- ### [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

  如果有人问我：算法的魅力是什么？

  我会甩给他这道题，看，一个好的算法有多么重要。代码精简并不意味着方法就是好的！！！

  我们先来看看“错误”答案

  ```java
  public int fib(int n) {
          if (n == 0) return 0;
          if (n == 1) return 1;
          return fib(n - 1) + fib(n - 2);
      }
  ```

  三行代码解决战斗，但是呢，每次这个算法都要进行大量的重复计算，再强的计算机都不好使。

  算法复杂度O（2^n）

  迭代法

  ```java
  public int fib1(int n) {
          if (n == 0) return 0;
          if (n == 1) return 1;
          int f1 = 0;
          int f2 = 1;
          for (int i = 2;i <= n;i++){
              int f = (f1 + f2) % 1000000007;
              f1 = f2;
              f2 = f;
          }
          return f2;
      }
  ```

  时间复杂度：O（n）

  看，一种好的算法带来的进步有多大！