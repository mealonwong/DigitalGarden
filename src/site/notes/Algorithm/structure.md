---
{"dg-publish":true,"permalink":"/algorithm/structure/"}
---


## skipList

```Java
import java.util.Random;

class SkipListNode {
    int value;
    SkipListNode[] next;

    public SkipListNode(int value, int level) {
        this.value = value;
        this.next = new SkipListNode[level];
    }
}

public class SkipList {
    private static final int MAX_LEVEL = 16;
    private static final double PROBABILITY = 0.5;
    private int levelCount = 1;
    private SkipListNode head = new SkipListNode(-1, MAX_LEVEL);
    private Random random = new Random();

    public SkipListNode find(int value) {
        SkipListNode p = head;
        for (int i = levelCount - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            }
        }
        if (p.next[0] != null && p.next[0].value == value) {
            return p.next[0];
        }
        return null;
    }

    public void insert(int value) {
        int level = randomLevel();
        SkipListNode newNode = new SkipListNode(value, level);
        SkipListNode[] update = new SkipListNode[level];
        SkipListNode p = head;
        for (int i = level - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            }
            update[i] = p;
        }
        for (int i = level - 1; i >= 0; i--) {
            newNode.next[i] = update[i].next[i];
            update[i].next[i] = newNode;
        }
        if (levelCount < level) {
            levelCount = level;
        }
    }

    public void delete(int value) {
        SkipListNode[] update = new SkipListNode[levelCount];
        SkipListNode p = head;
        for (int i = levelCount - 1; i >= 0; i--) {
            while (p.next[i] != null && p.next[i].value < value) {
                p = p.next[i];
            
            }
            update[i] = p;
        }
        if (p.next[0] != null && p.next[0].value == value) {
            for (int i = levelCount - 1; i >= 0; i--) {
                if (update[i].next[i] != null && update[i].next[i].value == value) {
                    update[i].next[i] = update[i].next[i].next[i];
                }
            }
        }
    }

    private int randomLevel() {
        int level = 1;
        while (random.nextDouble() < PROBABILITY && level < MAX_LEVEL) {
            level++;
        }
        return level;
    }
}

```


## 从前序和中序遍历序列构建二叉树

```Java
class Solution {  
    private int[] preorder;  
    private int[] inorder;  
    private HashMap<Integer,Integer> idx_map;  
    private int pre_idx=0;  
  
    public TreeNode buildTree(int[] preorder, int[] inorder) {  
        this.preorder = preorder;  
        this.inorder = inorder;  
        idx_map = new HashMap<>();  
        int idx = 0;  
        for(Integer val : inorder){  
            idx_map.put(val,idx++);  
        }  
        return helper(0,inorder.length);  
    }  
  
    private TreeNode helper(int start,int end){  
        if(start==end){  
            return null;  
        }  
        int rootVal = preorder[pre_idx];  
        TreeNode root = new TreeNode(rootVal);  
        int index = idx_map.get(rootVal);  
        pre_idx++;  
        root.left = helper(start,index);  
        root.right = helper(index+1,end);  
        return root;  
    }  
}
```

## 从中序和后序遍历序列构建二叉树

```Java
class Solution {  
  
    private int[] inorder;  
    private int[] postorder;  
    private Map<Integer,Integer> map;  
    private int idx;  
  
    public TreeNode buildTree(int[] inorder, int[] postorder) {  
        this.inorder = inorder;  
        this.postorder = postorder;  
        this.map = new HashMap<>();  
        this.idx = inorder.length-1;  
        int idx_map = idx;  
        for(Integer val : inorder){  
            map.put(val,idx_map--);  
        }  
        return helper(0,idx+1);  
    }  
  
    private TreeNode helper(int start,int end){  
        if(start==end){  
            return null;  
        }  
        int rootVal = postorder[idx];  
        TreeNode root = new TreeNode(rootVal);  
        int index = map.get(rootVal);  
        idx--;  
        root.right = helper(start,index);  
        root.left = helper(index+1,end);  
        return root;  
    }  
}
```