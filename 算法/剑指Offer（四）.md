# 剑指Offer（四）

### [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

使用一个辅助栈

```java
class Solution {
    public boolean validateStackSequences(int[] pushed, int[] popped) {
        Stack<Integer> integers = new Stack<>();
        int index = 0;
        for (int i = 0;i < pushed.length;i++){
            integers.push(pushed[i]);
            while (!integers.isEmpty() && popped[index] == integers.peek()){
                integers.pop();
                index++;
            }
        }
        return integers.isEmpty();
    }
}
```

### [剑指 Offer 32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

层序遍历

```java
class Solution {
    public int[] levelOrder(TreeNode root) {
        if (root == null) return new int[]{};
        LinkedList<TreeNode> treeNodes = new LinkedList<>();
        treeNodes.addLast(root);
        ArrayList<Integer> treeNodes1 = new ArrayList<>();
        while (!treeNodes.isEmpty()){
            TreeNode treeNode = treeNodes.pollFirst();
            treeNodes1.add(treeNode.val);
            if (treeNode.left != null){
                treeNodes.addLast(treeNode.left);
            }
            if (treeNode.right != null){
                treeNodes.addLast(treeNode.right);
            }
        }
        int[] res = new int[treeNodes1.size()];
        for(int i = 0; i < treeNodes1.size(); i++)
            res[i] = treeNodes1.get(i);
        return res;
    }
}
```

### [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>();
        LinkedList<TreeNode> queue = new LinkedList<>();
        ArrayList<List<Integer>> lists = new ArrayList<>();
        queue.addLast(root);

        while (!queue.isEmpty()){
            int count = queue.size();
            ArrayList<Integer> integers = new ArrayList<>();
            for (int i = count;i > 0;i--){
                TreeNode treeNode = queue.pollFirst();
                integers.add(treeNode.val);
                if (treeNode.left != null){
                    queue.addLast(treeNode.left);
                }
                if (treeNode.right != null){
                    queue.addLast(treeNode.right);
                }
            }
            lists.add(integers);
        }
        return lists;
    }
}
```

### [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>();

        boolean b = false;
        LinkedList<TreeNode> queue = new LinkedList<>();
        ArrayList<List<Integer>> lists = new ArrayList<>();
        queue.addLast(root);
        while (!queue.isEmpty()){
            int count = queue.size();
            LinkedList<Integer> integers = new LinkedList<>();
            for (int i = count;i > 0;i--){
                TreeNode treeNode = queue.pollFirst();
                if (b){
                    integers.addFirst(treeNode.val);
                }else {
                    integers.addLast(treeNode.val);
                }
                if (treeNode.left != null){
                    queue.addLast(treeNode.left);
                }
                if (treeNode.right != null){
                    queue.addLast(treeNode.right);
                }
            }
            b = !b;
            lists.add(integers);
        }
        return lists;
    }
}
```

### [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

使用递归

```java
class Solution {
    public boolean verifyPostorder(int[] postorder) {
        if (postorder == null || postorder.length == 0) return true;
        return verifyPostorder(postorder,0,postorder.length - 1);
    }
    
    private boolean verifyPostorder(int[] postorder,int i,int j){
        if (i >= j) return true;
        int p = i;
        while (postorder[p] < postorder[j]) p++;
        int m = p;
        while (postorder[p] > postorder[j]) p++;
        return p == j && verifyPostorder(postorder,i,m - 1) && verifyPostorder(postorder,m,j - 1);
    }
}
```

### [面试题34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

使用回溯，一层一层进行遍历

```java
class Solution {
    ArrayList ans = new ArrayList<List<Integer>>();
    ArrayList ints = new ArrayList<Integer>();
    public List<List<Integer>> pathSum(TreeNode root, int target) {
        if (root == null) return ans;
        recur(root,target);
        return ans;
    }

    private void recur(TreeNode root,int target){
        if (root == null) return;
        
        target -= root.val;
        ints.add(root.val);
        if (target == 0 && root.left == null && root.right == null){
            ans.add(new ArrayList<Integer>(ints));
        }
        
        recur(root.left,target);
        recur(root.right,target);
        ints.remove(ints.size() - 1);
    }
}
```

### [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

我们使用一个集合，用来保存新旧节点的对应关系

```java
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        HashMap<Node, Node> nodeNodeHashMap = new HashMap<>();
        Node head1 = head;
        
        while (head1 != null){
            nodeNodeHashMap.put(head1,new Node(head1.val));
            head1 = head1.next;
        }
        
        head1 = head;
        while (head1 != null){
            nodeNodeHashMap.get(head1).next = nodeNodeHashMap.get(head1.next);
            nodeNodeHashMap.get(head1).random = nodeNodeHashMap.get(head1.random);
            head1 = head1.next;
        }
        return nodeNodeHashMap.get(head);
    }
}
```

### [剑指 Offer 37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

怎么说呢？这道题我觉着谈不上困难

```java
public class Codec {

   // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        if (root == null) return "[]";

        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.addLast(root);
        StringBuilder stringBuilder = new StringBuilder("["+root.val);
        while (!queue.isEmpty()){
            TreeNode treeNode = queue.pollFirst();
            if (treeNode.left != null){
                stringBuilder.append(","+treeNode.left.val);
                queue.addLast(treeNode.left);
            }else {
                stringBuilder.append(",null");
            }

            if (treeNode.right != null){
                stringBuilder.append(","+treeNode.right.val);
                queue.addLast(treeNode.right);
            }else {
                stringBuilder.append(",null");
            }
        }
        stringBuilder.append("]");
        return stringBuilder.toString();
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        if (data.equals("[]")) return null;
        String[] strings = data.substring(1, data.length() - 1).split(",");
        TreeNode root = new TreeNode(Integer.parseInt(strings[0]));
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.addLast(root);
        int i = 1;
        while (!queue.isEmpty()){
            TreeNode treeNode = queue.pollFirst();
            
            if (!strings[i].equals("null")){
                treeNode.left = new TreeNode(Integer.parseInt(strings[i]));
                queue.addLast(treeNode.left);
            }
            i++;

            if (!strings[i].equals("null")){
                treeNode.right = new TreeNode(Integer.parseInt(strings[i]));
                queue.addLast(treeNode.right);
            }
            i++;
        }
        return root;
    }
}
```

### [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

回溯

```java
class Solution {
    private ArrayList<String> list = new ArrayList<String>();
    private boolean[] b;
    char[] aChar;
    public String[] permutation(String s) {
        if (s == null || s.length() == 0) return new String[]{};
        b = new boolean[s.length()];
        aChar = s.toCharArray();
        back(aChar,new StringBuilder());
        String[] strings = new String[]{};
        return list.toArray(strings);
    }

    private void back(char[] aChar,StringBuilder stringBuilder){
        if (stringBuilder.length() == aChar.length){
            list.add(stringBuilder.toString());
        }
        HashSet<Character> characters = new HashSet<>();

        for (int i = 0;i < aChar.length;i++){
            if (b[i] || characters.contains(aChar[i])){
                continue;
            }

            b[i] = true;
            characters.add(aChar[i]);
            stringBuilder.append(aChar[i]);
            back(aChar,stringBuilder);
            b[i] = false;
            stringBuilder.deleteCharAt(stringBuilder.length() - 1);
        }
        }
}
```

### [剑指 Offer 39. 数组中出现次数超过一半的数字](https://leetcode-cn.com/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)

我们保存两个变量，第一个变量记录当前出现次数最多的数字，另一个记录个数

遍历数组，当num[i]与变量值一样，count++，否则count--，当count为0时，我们替换掉该变量。

```java
public class _剑指_Offer_39_数组中出现次数超过一半的数字 {
    public int majorityElement(int[] nums) {
        if (nums == null || nums.length == 0) return -1;

        int index = nums[0];
        int count = 1;
        for (int i = 1;i < nums.length;i++){
            if (nums[i] == index){
                count++;
            }else {
                count--;
                if (count == 0){
                    index = nums[i];
                    count = 1;
                }
            }
        }
        return index;
    }
}
```

### [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

这个题有3种解法：排序（话说面试时写个sort真的不会挂吗）堆，以及快速选择（日后再写）

```java
public class _剑指_Offer_40_最小的k个数 {
    public int[] getLeastNumbers(int[] arr, int k) {
        if (arr == null || arr.length == 0 || k == 0) return new int[]{};
        PriorityQueue<Integer> integers = new PriorityQueue<Integer>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o1 - o2;
            }
        });
        integers.add(arr[0]);

        for (int i = 1;i < arr.length;i++){

            if (integers.size() < k){
                integers.offer(arr[i]);
                continue;
            }
            if (integers.peek() > arr[i]){
                integers.poll();
                integers.offer(arr[i]);
            }
        }
        int[] ints = new int[k];
        for (int i = k - 1;i >= 0;i--){
            ints[i] = integers.poll();
        }
        return ints;
    }
}
```

