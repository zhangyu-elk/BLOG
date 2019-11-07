# Trie树

## Trie树的作用
&emsp;&emsp;Trie树的作用是为了完成搜索引擎智能补全的功能, 其本质是利用字符串的公共前缀

![1572357754496.png](https://i.loli.net/2019/11/02/RQydsfW9C7Uc25E.png)

&emsp;&emsp;使用正常的方式每一次我们都需要在大量字符串中进行匹配, 如果建立了这样的一个树我们只需要按照树顺序走下来即可

## 树节点

&emsp;&emsp;假设我们真的在实现一个搜索引擎的只能补全功能, 那么在这个`Trie`树上的可能是字母也可能是汉字, 并且这是一个多叉树, 其子节点的数量也并不固定; 我们可以用`HASH`的方式进行快速的索引, 如果仅仅是字母的话, 我们可以以`[0-51]`对应`[a-Z]`; 如果是汉字的话那我们则真的使用`HASH`函数建立一个`HASH`表来进行索引(这里我们仅考虑字母的情况).

```
class TrieNode
{
public:
private:
	char data;
	TrieNode children[52];
};
```

## 代码及测试
```
/*
* Trie是常用进行搜索引擎提示这一类功能的数据结构
*/
#include <stdio.h>
#include <string>
#include <vector>
#include <windows.h>

class Trie
{
public:
	Trie();
	void insert(const char *text);
	bool find(const char *text);

	class TrieNode
	{
	public:
	    TrieNode(char chr);
	    ~TrieNode();
		bool isEndChar;				//是否是结束字符
		char data;
		TrieNode *children[52];		//包括大写字母和小写字母
	};
private:
	TrieNode root;
};

Trie::Trie():root('/')
{

};

bool Trie::find(const char *text)
{
	if(!text || !strlen(text)) return false;

	TrieNode *p = &root;
	for(int i = 0; i < strlen(text); i++)
	{
		int index = (text[i] >= 97) ? (text[i] - 'a') : (text[i] - 'A');
		if(nullptr == p->children[index])
		{
			return false;
		}
		else
		{
			p = p->children[index];
		}
	}

	if(p->isEndChar)
	{
		return true;
	}
	else
	{
		return false;
	}
}

//插入一个字符串
void Trie::insert(const char *text)
{
	if(!text || !strlen(text)) return;

	TrieNode *p = &root;
	for(int i = 0; i < strlen(text); i++)
	{
		int index = (text[i] >= 97) ? (text[i] - 'a') : (text[i] - 'A');
		if(nullptr == p->children[index])
		{
			
			p->children[index] = new TrieNode(text[i]);
			p = p->children[index];
		}
		else
		{
			p = p->children[index];
		}
	}
	p->isEndChar = true;
}

Trie::TrieNode::TrieNode(char chr)
{
	data = chr;
	for(int i = 0; i < 52; i++)
    {
	    children[i] = nullptr;
    }
}

Trie::TrieNode::~TrieNode()
{
	for(int i = 0; i < 52; i++)
	{
		if(children[i] != nullptr)
			delete children[i];
	}
}

int main()
{
	std::vector<std::string> dict;
	char buffer[128] = { 0 };
	FILE *fp = fopen("./newfile.txt","r");
	if(!fp)
    {
	    printf("fpooen error\n");
    }
	Trie *tree = new Trie();
	int count = 0;
	while(EOF != fscanf(fp, "%s", buffer))
	{
		tree->insert(buffer);
		dict.push_back(buffer);
	}
	fclose(fp);

	const char *test1 = "pocketbook";
    printf("%d\n",dict.size());
	DWORD start = GetTickCount();
	for(int j = 0; j < 100000; j++)
    {
        for(int i = 0; i < dict.size(); i++)
        {
            if(0 == strcmp(test1, dict[i].c_str()))
            {
                //printf("%s\n", "string match find succ!");
                break;
            }
        }
    }

	printf("string find time: %ld\n", GetTickCount() - start);

	start = GetTickCount();
	for(int j = 0; j < 10000000; j++)
    {
        if(tree->find(test1))
        {
            //printf("%s\n", "Trie find succ!");
        }
    }

	printf("Trie find time: %ld\n", GetTickCount() - start);

	delete tree;
   return 0;
}
```

**测试结果**:
```
45091
string find time: 26579
Trie find time: 1390
```

## 分析

* 1 结果差距比较大
* 2 如果以正常的字符串匹配, 其效率最大为`O(N*M)`; 对于`Trie`树而言, 则是`O(M)`级别.
* 3 当然实际工作中, 我们往往匹配几个字符串, 那么我们往往不需要考虑什么算法; Trie树更多的可以提供自动补全的功能.