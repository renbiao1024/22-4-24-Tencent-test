# 2022-4-24 腾讯笔试

## T1 竖着读

给n个数字字符串，每个字符串长度都为m，然后按照每一列从上往下读构成m个字符串，求这m个排序后的字符串，排序输出。

- 按列读取存储，stoi为数字

~~~cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

int main()
{
	int n;
	cin >> n;
	vector<string>nums(n);
	for (int i = 0;i < n; ++i)
		cin >> nums[i];
	int len = nums[0].size();
	vector<int>res(len);
	for (int i = 0; i < len; ++i)
	{
		string cur;
		for (int j = 0; j < n; ++j)
			cur += nums[j][i];
		res[i] = stoi(cur);
		cur = "";
	}
	for (int i = 0; i < len; ++i)
		cout << res[i] << ' ';
	return 0;
}
~~~

## T2 删除非质数下标后的最终元素

给一个数组，下标从1-n，每次淘汰下标非质数的数字，问最后剩下的一个数字是什么？

- 用质数筛找到100000以内的质数，模拟修改数组

~~~cpp
#include <iostream>
#include <vector>
#include <unordered_set>
using namespace std;
const int MAXN = 100000;
vector<bool>np(MAXN, false);
unordered_set<int>Primes;

void getPrime()
{
	for (int i = 2; i < MAXN; ++i)
	{
		if(!np[i])Primes.insert(i);
		for(int j = 2*i ; j < MAXN; j += i)
			np[j] = true;
	}
}

int getNumber(vector<int>& a) {
	while (a.size() > 1)
	{
		vector<int>res;
		for (unsigned int i = 0; i < a.size(); ++i)
		{
			if(Primes.count(i+1)) res.push_back(a[i]);
		}
		a = res;
	}
	return a[0];
}
~~~

## T3 攻防最小差值

给一堆字符串代表一排士兵，士兵编号1~n，字符串中’0’的士兵代表进攻性的，‘1’的代表防御性的，每个士兵的攻击力或守备力为其下标值。将士兵分组，0~pos的是进攻组，只算攻击力，pos+1~n的是防御组，只算防御力。pos可以取0~n。求攻击组的攻击力和防御组的防御力的差的绝对值的最小值。

- 从**攻方为0**遍历到 **攻方>守方** 结束，找到差值的最小值

~~~cpp
#include <iostream>
#include <vector>
using namespace std;

int n ;
string s;

int main()
{
	cin >> n >> s;
	int attack = 0, defense = 0;
	for (int i = 0; i < n; ++i)
	{
		if(s[i] == '1')defense += i+1;
	}
	int res = abs(attack-defense);
	for (int i = 0; i < n; ++i)
	{
        	if(attack > defense) break;
		if(s[i] == '0')attack += i+1;
		else defense -= i+1;
        	res = max(res,abs(attack-defense));
	}
	cout << res;
}
~~~

## T4构造链表

给一个链表数组，数组中的每个链表是一个循环链表中的破碎的部分，且每个链表结点的值唯一且为数值类型，求将这个循环链表复原以后，从链表中任意一个结点正序或逆序遍历 字典序 最小的那个链表，并返回链表的头。

- 找到最小值
- unordered_map记录前驱和后继
- 判断前驱和后继哪个小就从哪边开始完成链表

~~~cpp
#include <iostream>
#include <unordered_map>
#include <vector>
using namespace std;

struct ListNode {
	int val;
	struct ListNode* next;
	ListNode(int _val):val(_val){}
};
unordered_map<int, int>l, r;
ListNode* solve(vector<ListNode*>& a) {
	int i, m = INT_MAX;
	ListNode* p;
	for (i = 0;i < a.size();i++) {
		p = a[i];
		m = min(m, p->val);
		if (!p)continue;
		while (p->next) {
			r[p->val] = p->next->val;
			l[p->next->val] = p->val;
			p = p->next;
			m = min(m, p->val);
		}
	}
    ListNode*head = new ListNode(m);
	p = head;
	if (l[m] > r[m]) {
		while (r[p->val] != m) {
			p->next = new ListNode(r[p->val]);
			p = p->next;
		}
	}
	else {
		while (l[p->val] != m) {
			p->next = new ListNode(l[p->val]);
			p = p->next;
		}
	}
	return head;
}
~~~

## T5 股票问题

现在有一个长度为n的价格数组a，表示某只股票 每天的 价格，你每天最多买入或卖出该只股票的1手股票，买入或者卖出没有手续费，且卖出股票前必须手里已经有股票才能卖出，但是持有的股票数目不受限制，并且初始资金为m元，你在任意时刻都不能进行透支，所以你的资金必须始终大于等于 0 。请问你在n天结束之后，拥有最大的总资产是多少?

- （总资产=股票数目*股票价格+现金)
- dp\[i][j]表示第 i 天，持有 j 股 的总资产
  - 前一天 max(持有，卖出)
  - if(前一天资产>prices[i]) max(持有卖出，买入)

~~~cpp
#include<iostream>
#include <vector>
#include <algorithm>
using namespace std;
int main() {
	int n, m;
	cin >> n >> m;
	vector<int>prices(n+1);
	int i, j;
	for (i = 1;i <= n;i++) cin >> prices[i];
	vector<vector<int>>dp(n+1,vector<int>(n+2,INT_MIN));
	dp[0][0] = m;
	for (i = 1;i <= n;i++) {
		for (j = 0;j <= n;j++) {
			//持有不动和卖出的情况
			dp[i][j] = max(dp[i - 1][j],dp[i - 1][j + 1] + prices[i]);
			//如果j大于0而且前一天的现金数目大于今天的股票价格，即股票数目大于0有可能是(持有/卖出)或者买入1只股票转化而来的状态
			if (j && dp[i - 1][j - 1] >= prices[i])
                		dp[i][j] = max(dp[i][j], dp[i - 1][j - 1] - prices[i]);
		}
		cout << endl;
	}
	int res = 0;
	for (i = 0;i <= n;i++)
		res = max(res, dp[n][i] + i * prices[n]);
	cout << res;
}
~~~

