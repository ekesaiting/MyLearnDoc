## 常用的java Api

### 字符串数组：

字符串转数组：str.toCharArray();

字符串反转：new StringBuffer(str).reverse().toString();

字符串拼接 String.join("分隔符"，字符串数组)

### 集合：

List<Integer> 转int[] :list.stream().mapToInt(t->t).toArray();

List<String> 转String[]:list.toArray（new String[list.size()];

map.putIfAbsent(K key,V value):如果key不存在或等于null,则将它与value相关联；

map.getOrDefault(key,0) 获取一个key,如果为null将返回0，否则返回原有值

上述两个方法可以实现统计某个元素出现的次数：

```java
map.put(key,map.getOrDefault(key,0)+1)
```

反转集合：Collections.reverse(list);

交换集合中的两个元素位置：Collection.swap(Collection<T> list, int index1,int index2);



## 滑动窗口

例题：输入任意一个正整数N,在 1,2,3，....N中找到一个连续的正整数序列使之等于N。

如：输入：`9`      输出： `[[4,5][2,3,4] ]`

需要定义的变量: 左边界 ：left,右边界 right 使他们都指向有序数据的第一个，定义一个sum保存当前窗口内的总值

定义一个list保存结果

处理过程:

* 首先明确可供滑动的窗口大小（左右边界的最大值），本题中为 target/2
* 何时结束滑动？当left为右边界最大值时，说明整个序列已近遍历完
* 当sum小于target时，窗口右边界向右移动， right++; sum=sum+right;
* 当sum大于target时，窗口左边界向右移动，sum=sum-left;  left++;
* 当sum等于target时，说明找到了所需的序列，创建一个right-left+1的数组，将left-->right的数据存入数组，再将数组存入最终返回的list,然后窗口左边界向右移动，sum=sum-left;  left++;

## 二分查找

二分查找必须是有序的序列。

所查找的范围为实际的范围，所以left=0;right=size-1;继续二分的条件为left<=right; mid=left+(right-left)/2,防止整数溢出，
当所需的值小于目标值时，binarySearch(nums,left,mid-1)

当所需的值大于目标值时，binarySearch(nums,mid+1,right)

## 非递归遍历

### 前序遍历

前序遍历：利用栈的先进后出，优先将右子树放入栈中，再放左子树。当进入某个节点的子树时，栈中弹出的会是左子树节点。

```java
public static void preOrderIteration(TreeNode head) {
	if (head == null) {
		return;
	}
	Stack<TreeNode> stack = new Stack<>();
	stack.push(head);
	while (!stack.isEmpty()) {
		TreeNode node = stack.pop();
		System.out.print(node.value + " ");
        //由于栈是先进后出，前序遍历要求右子树最后输出，因此将右子树优先放入栈中
		if (node.right != null) {
			stack.push(node.right);
		}
		if (node.left != null) {
			stack.push(node.left);
		}
	}
}
```

### 后序遍历

与前序遍历类似，只要把输出的顺序反过来就可以，所以创建另一个栈保存遍历结果

```java
public static void postOrderIteration(TreeNode head) {
		if (head == null) {
			return;
		}
		Stack<TreeNode> stack1 = new Stack<>();
		Stack<TreeNode> stack2 = new Stack<>();
		stack1.push(head);
		while (!stack1.isEmpty()) {
			TreeNode node = stack1.pop();
			stack2.push(node);
            //由于最终输出又放入了另一个栈，导致子树输出结果最终是先进先出，所以需要将左子树先入栈
			if (node.left != null) {
				stack1.push(node.left);
			}
			if (node.right != null) {
				stack1.push(node.right);
			}
		}
		while (!stack2.isEmpty()) {
			System.out.print(stack2.pop().value + " ");
		}
	}
```

### 中序遍历

找到树最左边的叶子结点，将路径上的所有节点加入栈中，输出该叶子结点后，将右子树节点加入栈中，再找到该子树的最左边叶子结点即可

```java
public static void inOrderIteration(TreeNode head) {
	if (head == null) {
		return;
	}
	TreeNode cur = head;
	Stack<TreeNode> stack = new Stack<>();
	while (!stack.isEmpty() || cur != null) {
		while (cur != null) {
			stack.push(cur);
			cur = cur.left;
		}
		TreeNode node = stack.pop();
		System.out.print(node.value + " ");
		if (node.right != null) {
			cur = node.right;
		}
	}
}
```



## 重建二叉树

### 根据有序数组重建高度平衡二叉搜索树。

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return helper(nums,0,nums.length-1);
    }
    public TreeNode helper(int[] nums,int left,int right){
            if(left>right) return null;
            int mid=left+(right-left)/2;
        	//构建根节点
            TreeNode node=new TreeNode(nums[mid]);
        	//递归构建左子树
            node.left=helper(nums,left,mid-1);
        	//递归构建右子树
            node.right=helper(nums,mid+1,right);
        //返回当前根节点
        return node;

    }
}
```



### 根据前序遍历和中序遍历重建二叉树

假设传入的数组为 前序【1,2,5,8,4,3】 中序【5,2,8,1,3,4】

3. 前序遍历的结果的第一个节点一定是根节点，通过这个根节点的值，可以在中序遍历中找到根节点，中序遍历的根节点两边一定是左右子树，前序遍历和中序遍历的共同特点是根节点的右子树一定是晚于左子树遍历的，所以在中序遍历中的根节点的右边部分的节点索引一定在前序遍历当前索引的右边部分，因为右子树的节点个数是固定的，且都晚于左子树遍历，所以都集中在遍历结果的后半部分。

4. 由上可以得到前序遍历的根节点左子树索引范围和右子树索引范围，由前序遍历可知左子树的根节点和右子树的根节点，递归可以构造整颗二叉树。

   ```java
   class Solution {
       public TreeNode buildTree(int[] preorder, int[] inorder) {
               return build(preorder ,0,preorder.length-1,
                            inorder,0,inorder.length-1);
       }
       public TreeNode build(int[] preOrder,int preStart,int preEnd,
                             int[] inOrder,int inStart,int inEnd ){
           //如果前序遍历的start>end，说明前序遍历的数组中已经全部加入到树中了
           if(preStart>preEnd) return null;
           //前序遍历的第一个节点一定是整颗数每个子树的根节点
           int rootVal=preOrder[preStart];
           int index=0;
           //找到中序遍历中根节点的下标
           for (int i = inStart; i <=inEnd ; i++) {
               if(inOrder[i]==rootVal){
                   index=i;
                   break;
               }
           }
           //计算左子树的长度，无论前序遍历还是中序遍历，左右子树的节点都是连续排列的且长度一致
           int leftSize=index-inStart;
           //构建根节点
           TreeNode root=new TreeNode(rootVal);
           //当前根节点的左子树的范围：
           //在前序遍历中起始范围为当前根节点下标的后一位，结束范围为起始坐标加左子树长度
           //在中序遍历中起始范围为中序遍历的起始下标，结束范围为根节点的前一位
           root.left=build(preOrder,preStart+1,preStart+leftSize,
                               inOrder,inStart,index-1);
            //当前根节点的右子树的范围：
           //在前序遍历中起始范围为起始位置+左子树长度+1，结束范围为前序遍历的结束下标
           //在中序遍历中起始范围为中序遍历的根节点的后一位，结束范围为后续遍历的结束下标
           root.right=build(preOrder,preStart+leftSize+1,preEnd,
                           inOrder,index+1,inEnd); 
        return root;
       }
   }
   ```

### 根据中序遍历与后续遍历重建二叉树

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        return build(inorder,0,inorder.length-1,
                postorder,0,postorder.length-1);
    }
    public TreeNode build(int[] inorder,int inStart,int inEnd,
                          int[] postOrder,int postStart,int postEnd){
        //当后续遍历完成退出递归
        if(postStart>postEnd) return null;
        int rootVal=postOrder[postEnd];
        int index=0;
        for(int i=inStart;i<=inEnd;i++){
            if(inorder[i]==rootVal)
                index=i;
        }
        int leftSize=index-inStart;
        TreeNode root = new TreeNode(rootVal);
        //注意后续遍历的划分与前序有一些不同，中序一致
        root.left=build(inorder,inStart,inStart-1,
                postOrder,postStart,postStart+leftSize-1);
        root.right=build(inorder,index+1,inEnd,
                postOrder,postStart+leftSize,postEnd-1);
        return root;

    }
}
```

### 根据前序遍历与后续遍历重建二叉树

```java
class Solution {
    public TreeNode constructFromPrePost(int[] pre, int[] post) {
        return build(pre,0,pre.length-1,
                    post,0, post.length-1);
    }
    public TreeNode build(int[] preOrder,int preStart,int preEnd,
                          int[] postOrder,int postStart,int postEnd){
        //当前序/后序 遍历起始位置大于终止位置时，表示已经遍历完。
        if(preStart>preEnd||postStart>postEnd)
            return null;
        //当前序/后续 起始位置等于终止位置时，表示只有一个节点，直接返回这个节点
        if(preStart==preEnd||postStart==postEnd)
            return new TreeNode(preOrder[preStart]);
        int rootVal=preOrder[preStart];
        //创建根节点
        TreeNode root = new TreeNode(rootVal);
        //左子树的长度(实际上只是数组中的一个下标，并不能完全代表左子树长度)
        int leftSize=preStart;
        for(int i=postStart;i<=postEnd;i++){
            //由前序遍历的根节点的下一个节点可以确认左节点在后续遍历的下标，从而得到左子树的长度
            if(postOrder[i]==preOrder[preStart+1]){
                //找到后续遍历中左节点的位置，由于数组从零开始，得到长度需要加1
                leftSize=i+1;
                //找到后退出循环
                break;
            }
        }
        //根据左子树的长度可以得到前序遍历和后续遍历中左右子树的分布
        //前序遍历中，左子树的起始位置就是根节点+1，结束位置由于右子树得到长度只是一个下标，并不是当前数组的实际的左子树长度，它还包含已经构建完的左子树长度，所以需要减去已近构建完的左子树的长度，也就是后续遍历右子树开始的位置
        //后续遍历中，左子树的起始位置就是当前数组的起点，结束位置为左子树的长度-1.
        root.left=build(preOrder,preStart+1,preStart+leftSize-postStart,
                        postOrder,postStart,leftSize-1);
        //前序遍历中，右子树的起始位置就在左子树的后面，所以+1，结束位置就是当前数组的终点
        //后续遍历中，右子树的起始位置就是左子树的终点位置+1，由于后续遍历最后一个节点为根节点，所以结束位置为根节点-1.
        root.right=build(preOrder,preStart+leftSize-postStart+1,preEnd,
                        postOrder,leftSize,postEnd-1);
        return root;
    }
}
```



## 回溯算法

**全局变量**： 保存结果，如果返回值是boolean 就建立一个flag,如果是List或者数组就创建相应对象。

**参数设计**： 递归函数的参数，是将上一次操作的合法状态当作下一次操作的初始位置。这里的参数，我理解为两种参数：**状态变量**和**条件变量**。（1）状态变量（state）就是最后结果（result）要保存的值；（2）条件变量就是决定搜索是否完毕或者合法的值。  

**完成条件**： 完成条件是决定 状态变量和条件变量 在取什么值时可以判定整个搜索流程结束。搜索流程结束有两种含义： **搜索成功**并保存结果 和 **搜索失败**并返回上一次状态。  

**递归过程**： 传递当前状态给下一次递归进行搜索。

```java
class Solution{
    private List<List<Integer>> resList;  // 定义全局变量保存最终结果
    private LinkedList<Integer> state; // 定义状态变量保存当前状态,LinkedList更方便删除最后一个元素
    public List<List<Integer>> resolve(target，args2，...){//一般给定一个整数，或者一个字符串或二维数组
        resList=new ArrayList<>();
        state=new ArrayList<>();
        backTracking(state)
    }
    //state保存当前已近迭代了的元素，组合成一个字符串或者list
    //条件变量一般为数组，字符串，二维数组的下标
    public void backTrack(state,其他条件变量){
        //触发结束条件，由状态变量可以判断
        if(state==target){
            //加入最终结果集,注意要新建一个list保存结果，不然在回溯的过程中会被清空；
            resList.add(new ArrayList<>(state));
            return;
        }
        //二维数组的话需要换行
        /*
        if(column==width){
        //可能需要清空当前状态变量
        backTrack(状态变量，row++,column)
        }
        */
        //回溯算法就是穷举所有元素，所以选择列表一般如果是
        //字符串：选择就是要遍历整个字符串，所以循环从当前未遍历的字符串下标开始遍历至length-1
        //二维数组：二维数组就是要遍历整个数组，所以要两层循环，分别为当前还未遍历的行和列的下标
        //其他情况下就判断每一个位置都能放置的元素选择范围 如 如数字【0~9】，字符【a~z】,【yes,no】等等
        for(选择  in  选择列表){
            //减枝 或者 排除已近遍历的选择
            if（!isValid(选择)） contiue;
            //符合条件则将当前元素加入状态变量
            state.add(选择); //二维数组的话可能还需要改变当前位置的值为选择的值
            //条件变量一般为改变后的下标，比如字符串的字符下标
            backTrack(state,其他条件变量);
            //还原到未选择的时候，
            state.remove(选择);///二维数组的话可能还需要恢复当前位置的值为默认的值
               
            
    }
    public boolean isValid(参数){//一般是node,Integer,String,等类型，判断与其他位置上元素的关系
        //判断逻辑
    }
}

```





