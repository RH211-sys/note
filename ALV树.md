# AVL树

主要操作：左旋和右旋
目的：保证高度尽可能小，提高搜索效率

## 旋转类型和操作（核心）

1. LL:右旋失衡
2. LR:左旋失衡的左子变为LL，再右旋失衡
3. RL:右旋失衡的右子变为RR，再左旋失衡
4. RR:左旋失衡

结构定义

```C
typedef int element_t;

/* 平衡二叉树节点结构 */
typedef struct tree_node {
	element_t data;
	struct tree_node* left;
	struct tree_node* right;
	int height;	// 结点高度，用于平衡条件判断
} AVLNode;

/* 平衡二叉树头结构 */
typedef struct {
	AVLNode* root;
	int count;
} AVLTree;
```

左旋代码

```C
static AVLNode* leftRotate(AVLNode* x) {
	// 调整关系
	AVLNode* y = x->right;
	x->right = y->left;
	y->left = x;
	
	// 更新结点的高度（从下往上）
	x->height = maxNum(h(x->left), h(x->right));
	y->height = maxNum(h(y->left), h(y->right));

	// 更新父节点的子
	return y;
}
```

右旋代码

```C
static AVLNode* rightRotate(AVLNode* y) {	
	AVLNode* x = y->left;
	y->left = x->right;
	x->right = y;

	// 更新高度（从下往上）
	y->height = maxNum(h(y->left), h(y->right)) + 1;
	x->height = maxNum(h(x->left), h(x->right)) + 1;

	// 更新父节点的子
	return x;
}
```

平衡树的插入（递归）

1. 先找到空位（若已有此值则不变，反之找到空位）
2. 创建结点，调整关系
3. “归”时重新计算平衡结点，并处理平衡

```C
static AVLNode* insertAVLNode(AVLTree* tree, AVLNode* node, element_t e) {
	// 终止条件:找到空位，可以插入了
	if (node == NULL) {
		AVLNode* newNode = createAVLNode(e);
		if (newNode == NULL) {
			return;
		}
		tree->count++;
		return newNode;
	}

	// 递归条件
	if (e < node->data) {
		node->left = insertAVLNode(tree, node->left, e);
	}
	else if (e > node->data) {
		node->right = insertAVLNode(tree, node->right, e);
	}
	else {
		return node;	// 已存在该结点
	}

	// 归时重新计算高度
	node->height = 1 + maxNum(h(node->left), h(node->right));
	// 平衡处理
	int balance = getBalance(node);
	if (balance > 1) {
		if (e > node->left->data) {
			// LR
			node->left = leftRotate(node->left);
		}
		// LL
		return rightRotate(node);
	}
	else if (balance < -1) {
		if (e < node->right->data) {
			// RL
			node->right = rightRotate(node->right);
		}
		// RR
		return leftRotate(node);
	}
	// 已经平衡的
	return node;
}
```



平衡树的删除（递归）

1. 先找到该结点，找不到则返回
2. 若找到，则先判断这个结点的度，再进行不同方式的删除
   度为0直接删，度为1调整关系后删
   度为2找前驱节点，更新当前度为2的点值，删除这个前驱节点（转移矛盾）
3. ”归“时更新高度，处理平衡

```C
static AVLNode* deleteAVLNode(AVLTree* tree, AVLNode* node, element_t e) {
	// 找不到
	if (node == NULL) {
		return NULL;
	}
	if (e < node->data) {
		node->left = deleteAVLNode(tree, node->left, e);
	}
	else if (e > node->data) {
		node->right = deleteAVLNode(tree, node->right, e);
	}
	else {
		// 找到了
		AVLNode* tmp;
		// 度不为2
		if (node->left == NULL || node->right == NULL) {
			tmp = node->left ? node->left : node->right;
			// 度为0
			if (tmp == NULL) {
				free(node);
				tree->count--;
				return NULL;
			}
			// 度为1
			node->data = tmp->data;
			node->left = tmp->left;
			node->right = tmp->right;
			tree->count--;
			free(tmp);
		}
		// 度为2
		else {
			// 度为2，找前驱节点，更新当前度为2的点值，删除这个前驱节点
			tmp = node->left;
			while (tmp->right) {
				tmp = tmp->right;
			}
			node->data = tmp->data;
			node->left = deleteAVLNode(tree, node->left, e);
		}
	}
	// 更新平衡因子，并处理平衡
	node->height = maxNum(h(node->left), h(node->right)) + 1;
	int balance = getBalance(node);
	if (balance > 1) {
		if (getBalance(node->left) < 0) {
			// LR
			node->left = leftRotate(node->left);
		}
		// LL
		return rightRotate(node);
	}
	if (balance < -1) {		// 右多
		if (getBalance(node->right) > 0) {
			// RL
			node->right = rightRotate(node->right);
		}
		// RR
		return leftRotate(node);
	}
	// 已经平衡，无需处理
	return node;
}
```

