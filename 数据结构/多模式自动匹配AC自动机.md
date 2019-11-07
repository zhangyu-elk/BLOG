# 多模式自动匹配AC自动机
> KMP是多模式匹配算法, 解决的是一个字符串匹配多个模式串的问题, 该字符串往往短于或者等于模式串的长度(自动补全功能); 如果需要实现关键字屏蔽呢? 那就是我们需要在一个主串中匹配多个模式串.

## AC自动机原理
> AC自动机就是在Trie树上, 加上了类似KMP的`next`数组, 结合的实现.

## Trie树中的`next`数组

![ynq2VmaxQz7HUk45.png](https://i.loli.net/2019/11/05/nq2VmaxQz7HUk45.png)
&emsp;&emsp;考虑`KMP`的实现, 在Trie树中, 同样可以递归实现; 如图假设 我们已经求出了`next[p]=q`, 那么其下一级的值呢?
如果`p->child=next[p]->child`(即`pc=qc`), 那么可以得到`next[p->child]=q->child`. 但是如果不等于呢?

![7bHeKBZFrugvScX.png](https://i.loli.net/2019/11/05/7bHeKBZFrugvScX.png)
我们只有往`q`的匹配节点`next[q]`找了, 比较`p->child == next[q]->child`; 直到找到相等的节点或者找到根节点.

## AC自动机结构体(在Trie树上扩展`next`节点)
```
class AcNode
{
public:
	AcNode(char chr);
	~AcNode();
	bool isEndChar;				//是否是结束字符
	char data;
	TrieNode *children[52];		//包括大写字母和小写字母
	AcNode *fail;				//匹配失败, 可匹配的下一个节点
};
```
我们需要考虑一个问题, 到底是在创建`Trie`树后一次性构建还是每次插入一个新的值构建呢?

### 一次性构建失败指针的代码
> 在插入所有字符串后, 再开始创建代码
```
void AC::buildFail()
{
	std::queue<AcNode*> queue;

	root.fail = &root;
	queue.push(&root);
	while(!queue.empty())
	{
		AcNode *p = queue.top();
		queue.pop();

		for(int i = 0; i < 52; i++)
		{
			AcNode *pc = p->children[i];
			int index = (pc->data >= 97) ? (pc->data - 'a') : (pc->data - 'A');
			if(nullptr != pc)
			{
				AcNode *q = p->fail;
				while(q != &root)
				{
					if(nullptr != p->children[index])
					{
						pc->fail = p->children[index];
						break;
					}
					q = q->fail;
				}
				if(nullptr == pc->fail)
				{
					pc->fail = &root;
				}

				queue->push(pc);
			}

		}
	}
}
```

**事件复杂度**: 假设有`k`个节点, 深度为`l`, 那么最耗时的环节就是`q=q->fail`; 事件复杂度为`O(k*l)`

**思考**: 这个事件复杂度并不高, 而且并不会频繁调用, 我们完全可以就这样使用; 如果我们希望插入字符串后不需要大规模更新呢?
这是一件很复杂的事情, 假设我们插入`ab`这个字符串并且多出了`b`这个字符, 我们需要考虑的就是对所有`fail`指向`a`的节点进行遍历, 这样的工作并不值得.