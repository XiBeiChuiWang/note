# 剑指Offer（二）

- ### [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

  一看到题，很容易会想到遍历一遍，然后找出最小的，时间复杂度为O（n），那么有没有时间复杂度为O（logn）的算法呢——二分搜索法

  ```java
  class Solution {
      public int minArray(int[] numbers) {
          if (numbers == null || numbers.length == 0) return -1;
  
          int low = 0;
          int high = numbers.length - 1;
          while (low < high){
              int mid = (low + high) / 2;
              if (numbers[mid] < numbers[high]){
                  high = mid;
              }else if (numbers[mid] > numbers[high]){
                  low = mid + 1;
              }else {
                  high--;
              }
          }
          return numbers[low];
      }
  }
  ```

- ### [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

  这老朋友了，其实就是之前的单词搜索，使用我们的回溯

  ```java
  public class _剑指_Offer_12_矩阵中的路径 {
      int[][] ints = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
      public boolean exist(char[][] board, String word) {
          if (board == null || board.length == 0 || board[0].length == 0) return false;
  
          boolean[][] booleans = new boolean[board.length][board[0].length];
          char[] chars = word.toCharArray();
          for (int i = 0;i < board.length;i++){
              for (int j = 0;j < board[0].length;j++){
                  if (back(board,chars,0,booleans,i,j)){
                      return true;
                  }
              }
          }
          return false;
      }
  
      private boolean back(char[][] board, char[] chars,int k,boolean[][] booleans,int row,int col){
          if (board[row][col] != chars[k]){
              return false;
          }else if (k == chars.length - 1){
              return true;
          }
  
          booleans[row][col] = true;
          boolean ans = false;
          for (int i = 0;i < ints.length;i++){
              int new_row = row + ints[i][0];
              int new_col = col + ints[i][1];
              if (new_row >= 0 && new_row < board.length && new_col >= 0 && new_col < board[0].length && !booleans[new_row][new_col]){
                  boolean back = back(board, chars, k + 1, booleans, new_row, new_col);
                  if (back){
                      ans = true;
                      break;
                  }
              }
          }
  
          booleans[row][col] = false;
          return ans;
      }
  }
  ```

- ### [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

  这道题和上面一样，也是使用我们的回溯

  ```java
  class Solution {
      int ans = 0;
      int[][] ints = new int[][]{{0,1},{1,0}};
      public int movingCount(int m, int n, int k) {
          if (m <= 0 || n <= 0 || k < 0){
              return -1;
          }
  
          boolean[][] booleans = new boolean[m][n];
          back(booleans,0,0,k);
          return ans;
      }
  
      private void back(boolean[][] booleans,int row,int col,int k){
          if (!isCome(row, col, k)){
              return;
          }
  
          booleans[row][col] = true;
          ans++;
          for (int i = 0;i < ints.length;i++){
              int new_row = row + ints[i][0];
              int new_col = col + ints[i][1];
  
              if (new_row >= 0 && new_row < booleans.length && new_col >= 0 && new_col < booleans[0].length && !booleans[new_row][new_col]){
                  back(booleans,new_row,new_col,k);
              }
          }
      }
  
      private boolean isCome(int row,int col,int k){
          int r = 0;
          while (row / 10 != 0){
              r += row % 10;
              row = row / 10;
          }
          r += row;
          int c = 0;
          while (col / 10 != 0){
              c += col % 10;
              col = col / 10;
          }
          c += col;
          return (r + c) <= k;
      }
  }
  ```

- ### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

  很明显，大问题是由一个个小问题组成的，我们可以使用**动态规划**去解决，很简单，一段长为n的绳子，我们先切第一刀，切完后，我们可以选择继续切，也可以不切，因此，状态转移方程为：

  **当n < 4时：**

  **dp[n] = n - 1**

  **当n >= 4时**

  **dp[n] = Math.max((n - j) * j,dp[n - j] * j)** 

  **(n - j) * j 表示切完j后不再切**

  **dp[n - j] * j表示切完j之后继续切**

  ```java
  public int cuttingRope(int n) {
          if (n < 0) return -1;
          if (n == 0 || n == 1) return n;
  
          int[] dp = new int[n + 1];
          dp[1] = 1;
  
          for (int i = 2;i <= n;i++){
              int max = 0;
              for (int j = 1;j < i;j++){
                  max = Math.max(max,Math.max((i - j) * j,dp[i - j] * j));
              }
              dp[i] = max;
          }
          return dp[n];
      }
  ```

  那么除了动态规划之外，我们还有没有其他的办法去解决呢？

  我们运用数学知识可以得知，如果我们尽可能的切成一段段长为3的绳子，那么其乘积是最大的。当然这是其长度大于4的时候，当小于等于4时，我们就是死答案

  贪心法：

  ```java
  public int cuttingRope1(int n) {
          if (n < 0) return -1;
          if (n < 4) return n - 1;
          int res = 1;
          while (n > 4){
              res *= 3;
              n -= 3;
          }
          return res * n;
      }
  ```

- ### [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

  例如9，它的二进制写法为1001，因此答案为2，首先我们应该想到将这个数字进行逻辑右移（右移后高位补0，java中的运算符为>>>）

  ```java
  public int hammingWeight(int n) {
          int res = 0;
          while (n != 0){
              res += n & 1;
              n = n >>> 1;
          }
          return res;
      }
  ```

  还有一种更巧妙的做法，我们来看1101这个二进制串，我们将其减一，得到1100，最右边的1不见了，

  我们来看1100，将其减一变为1011，这时候与1100进行与操作，变为1000，右边的1又不见了，是不是很神奇

  ```java
  public int hammingWeight(int n) {
          int res = 0;
          while (n != 0){
              res++;
              n = (n - 1) & n;
          }
          return res;
      }
  ```

- ### [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

  x^n = (x^2)^n/2

  n为整数时，则需要分奇偶：
  当n为奇数时 x^n =x* (x^2)^n//2

  当n为偶数时 x^n =(x^2)^n//2

  ```java
  public class _剑指_Offer_16_数值的整数次方 {
      public double myPow(double x, int n) {
          if(x == 0) return 0;
          long b = n;
          double res = 1.0;
          if(b < 0) {
              x = 1 / x;
              b = -b;
          }
          while(b > 0) {
              if((b & 1) == 1) res *= x;
              x *= x;
              b >>= 1;
          }
          return res;
      }
  }
  ```

- ### [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

  两个指针，没啥说的

  ```java
  class Solution {
      public ListNode deleteNode(ListNode head, int val) {
          if (head == null) return head;
          
          ListNode fast = head;
          if (val == head.val){
              head = head.next;
              return head;
          }else {
              fast = fast.next;
          }
          ListNode slow = head;
          while (fast != null){
              if (val == fast.val){
                  slow.next = fast.next;
                  return head;
              }else {
                  slow = slow.next;
                  fast = fast.next;
              }
          }
          return head;
      }
  }
  ```

- ### [剑指 Offer 19. 正则表达式匹配](https://leetcode-cn.com/problems/zheng-ze-biao-da-shi-pi-pei-lcof/)

  这个题是一道困难题，我们很容易想到动态规划，可是它的状态转移方程有些难写，我们来看看

  首先，我们用dp[i] [j]来表示p[0……j - 1]是否匹配s[0……i - 1]

  当i  != 0 时，dp[i] [0] 等于false，表示p字段为空，但是sbuweikong

  接下来分情况讨论，

  当s[i - 1] != '*' 时，我们只需要看s 和 p 的最后一位是否相同，

  如果相同，则dp[i] [j] = dp [i - 1] [j - 1]

  如果不同，则为false 

  当s[i - 1] == '*' 时，我们要分两种情况讨论

  当s[i - 1] != p[j - 2]时，我们唯一的选择就是抛弃p的这两位，也就是（a*，a的出现次数为0次） dp[i] [j] = dp [i] [j - 2]

  当s[i - 1] == p[j - 2]时,我们就可以选择匹配0次，也可以选择匹配其他次

  dp[i] [j] = dp [i] [j - 2] || dp[i] [j] = dp [i - 1] [j]

  ```java
  public class _剑指_Offer_19_正则表达式匹配 {
      public boolean isMatch(String s, String p) {
          int m = s.length();
          int n = p.length();
  
          boolean[][] dp = new boolean[m + 1][n + 1];
  
          for (int i = 0;i <= m;i++){
              for (int j = 0;j <= n;j++){
                  //当p为空时
                  if (j == 0){
                      dp[i][j] = i == 0;
                      continue;
                  }
                  //p不为空时，我们分两种情况
                  //当结尾是*
                  if (j >= 2 && p.charAt(j - 1) == '*'){
                      //选择匹配0次
                      dp[i][j] = dp[i][j - 2];
                      //选择匹配0次或多次（前提是p.charAt(j - 2) == s.charAt(i - 1)）
                      if (i >= 1 && (p.charAt(j - 2) == s.charAt(i - 1) || p.charAt(j - 2) == '.')){
                          dp[i][j] = dp[i][j - 2] || dp[i - 1][j];
                      }
                   //当结尾不为*   
                  }else {
                      //如果结尾相同
                      if (i >= 1 && (p.charAt(j - 1) == s.charAt(i - 1) || p.charAt(j - 1) == '.')){
                          dp[i][j] = dp[i - 1][j - 1];
                      }
                      //如果不同则直接为false
                  }
              }
          }
          return dp[m][n];
      }
  }
  ```

- ### [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

  有限状态机

  ![](https://pic.imgdb.cn/item/6047422f5aedab222c88878f.png)

  ```java
  class _剑指_Offer_20_表示数值的字符串 {
      public boolean isNumber(String s) {
          Map<State, Map<CharType, State>> transfer = new HashMap<State, Map<CharType, State>>();
          Map<CharType, State> initialMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_SPACE, State.STATE_INITIAL);
              put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
              put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
              put(CharType.CHAR_SIGN, State.STATE_INT_SIGN);
          }};
          transfer.put(State.STATE_INITIAL, initialMap);
          Map<CharType, State> intSignMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
              put(CharType.CHAR_POINT, State.STATE_POINT_WITHOUT_INT);
          }};
          transfer.put(State.STATE_INT_SIGN, intSignMap);
          Map<CharType, State> integerMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_INTEGER);
              put(CharType.CHAR_EXP, State.STATE_EXP);
              put(CharType.CHAR_POINT, State.STATE_POINT);
              put(CharType.CHAR_SPACE, State.STATE_END);
          }};
          transfer.put(State.STATE_INTEGER, integerMap);
          Map<CharType, State> pointMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
              put(CharType.CHAR_EXP, State.STATE_EXP);
              put(CharType.CHAR_SPACE, State.STATE_END);
          }};
          transfer.put(State.STATE_POINT, pointMap);
          Map<CharType, State> pointWithoutIntMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
          }};
          transfer.put(State.STATE_POINT_WITHOUT_INT, pointWithoutIntMap);
          Map<CharType, State> fractionMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_FRACTION);
              put(CharType.CHAR_EXP, State.STATE_EXP);
              put(CharType.CHAR_SPACE, State.STATE_END);
          }};
          transfer.put(State.STATE_FRACTION, fractionMap);
          Map<CharType, State> expMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
              put(CharType.CHAR_SIGN, State.STATE_EXP_SIGN);
          }};
          transfer.put(State.STATE_EXP, expMap);
          Map<CharType, State> expSignMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
          }};
          transfer.put(State.STATE_EXP_SIGN, expSignMap);
          Map<CharType, State> expNumberMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_NUMBER, State.STATE_EXP_NUMBER);
              put(CharType.CHAR_SPACE, State.STATE_END);
          }};
          transfer.put(State.STATE_EXP_NUMBER, expNumberMap);
          Map<CharType, State> endMap = new HashMap<CharType, State>() {{
              put(CharType.CHAR_SPACE, State.STATE_END);
          }};
          transfer.put(State.STATE_END, endMap);
  
          int length = s.length();
          State state = State.STATE_INITIAL;
  
          for (int i = 0; i < length; i++) {
              CharType type = toCharType(s.charAt(i));
              if (!transfer.get(state).containsKey(type)) {
                  return false;
              } else {
                  state = transfer.get(state).get(type);
              }
          }
          return state == State.STATE_INTEGER || state == State.STATE_POINT || state == State.STATE_FRACTION || state == State.STATE_EXP_NUMBER || state == State.STATE_END;
      }
  
      public CharType toCharType(char ch) {
          if (ch >= '0' && ch <= '9') {
              return CharType.CHAR_NUMBER;
          } else if (ch == 'e' || ch == 'E') {
              return CharType.CHAR_EXP;
          } else if (ch == '.') {
              return CharType.CHAR_POINT;
          } else if (ch == '+' || ch == '-') {
              return CharType.CHAR_SIGN;
          } else if (ch == ' ') {
              return CharType.CHAR_SPACE;
          } else {
              return CharType.CHAR_ILLEGAL;
          }
      }
  
      enum State {
          STATE_INITIAL,
          STATE_INT_SIGN,
          STATE_INTEGER,
          STATE_POINT,
          STATE_POINT_WITHOUT_INT,
          STATE_FRACTION,
          STATE_EXP,
          STATE_EXP_SIGN,
          STATE_EXP_NUMBER,
          STATE_END,
      }
  
      enum CharType {
          CHAR_NUMBER,
          CHAR_EXP,
          CHAR_POINT,
          CHAR_SIGN,
          CHAR_SPACE,
          CHAR_ILLEGAL,
      }
  }
  ```

  