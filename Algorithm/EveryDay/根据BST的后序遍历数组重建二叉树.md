### 1.基础解

```java
/**
 * 已知一颗个搜索二叉树后序遍历的数组arr，请根据arr，重建整棵树，并返回新建树的头结点。
 * 搜索二叉树性质：左子树节点的值均小于右子树节点的值。
 * @author ddx
 * @date 2022/4/20
 */
public class BST {
	public static void main(String[] args) {
		int[] arr = { 1, 2, 3, 4, 5 };
		Node head = createBST(arr, 0, arr.length - 1);
		middleSearch(head);
	}

	public static Node createBST(int[] arr, int L, int R) {
		if (L > R) {
			return null;
		}
		if (L == R) {
			return new Node(arr[R]);
		}
		int M = L - 1;
		Node hNode = new Node(arr[R]);
		for (int i = L; i < R; i++) {
			if (arr[i] < arr[R]) {
				M = i;
			}
		}
		// 拿[L...M]的范围递归创建左树
		// 拿[M+1...R-1]的范围递归创建右树
		hNode.left = createBST(arr, L, M);
		hNode.right = createBST(arr, M + 1, R - 1);
		return hNode;
	}

	public static void middleSearch(Node head) {
		if (head == null) {
			return;
		}
		middleSearch(head.left);
		System.out.println(head.value);
		middleSearch(head.right);
	}
}

class Node {
	int value;
	Node left;
	Node right;

	public Node(int value) {
		this.value = value;
	}
}
```

问题：M的初始值为什么要设置成L-1？是为了兼容单侧树，即出现一边倾的情况。
               5                            5
		       /                                \
		     4                                    6
		   /              或者                  \
	     3                                            7
	    /                                                \
    2                                                     8
   /                                                          \
 1                                                             9
1. 当全部小于 arr[R]，即上图左树情况下，当设置为M=L-1，但M最终更新为R-1，左子树范围[L...R-1]，正常递归；右子树范围[R...R-1]，返回空节点，符合预期。
2. 当全部大于 arr[R]，即上图右树情况下，当设置为M=L-1，循环完毕仍然是L-1，左子树范围[L...L-1]，返回空节点，符合预期；右子树范围[L...R-1]，正常递归。
3. 当既有小于又有大于arr[R]时，不再讨论。


## 2.最优解
上述解法并不是最优解，原因在于为了找到M我们每次都需要从L遍历到R - 1，导致时间复杂度为O(N^2)。
最优解可以通过改进寻找M的方式 - 二分法，可以将时间复杂度降低为O(NLogN)
```java
public static Node createBSTPlus(int[] arr, int L, int R) {
	if (L > R) {
		return null;
	}
	Node hNode = new Node(arr[R]);
	if (L == R) {
		return hNode;
	}
	int M = L - 1;
	int left = L;
	int right = R - 1;
	// 采用二分寻找M
	while (left <= right) {
		int middle = left + ((right - left) >> 1);
		if (arr[middle] < arr[R]) {
			M = left;
			left = middle + 1;
		} else if (arr[middle] > arr[R]) {
			right = middle - 1;
		} else {
			throw new RuntimeException("数据有误！");
		}
	}
	hNode.left = createBSTPlus(arr, L, M);
	hNode.right = createBSTPlus(arr, M + 1, R - 1);
	return hNode;
}
```