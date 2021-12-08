把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
```c++
class Solution {
public:
    int GetUglyNumber_Solution(int index) {
        if (index <= 0) return 0;
        vector<int> ugly(index);
        ugly[0] = 1;
        //设置一个丑数有序队列Q
        //每一个丑数分别乘【2，3，5】可以产生3个丑数
        int  p2 = 0, p3 = 0, p5 = 0;
        //取Q中第一个数1分别2，3，5放入A，B，C中
        //A=[2]  B=[3]  C=[5]

        // 第二次迭代
        // 取出ABC中最小的数2放入Q，Q=[1,2]
        // 取Q中第二个数2分别2,3,*5放入A，B，C中
        // A=[4]  B=[3,6]  C=[5,10]
        for (int i = 1; i < index; ++ i) {
            ugly[i] = min(ugly[p2]*2, min(ugly[p3]*3, ugly[p5]*5));
            if(ugly[i] == ugly[p2]*2) ++p2;
            if(ugly[i] == ugly[p3]*3) ++p3;
            if(ugly[i] == ugly[p5]*5) ++p5;
        }
        return ugly[index-1];
    
    }
};
```

在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）.（从0开始计数）

```c++
class Solution {
public:
//     int find(vector<pair<char, int> > &maps, int target)
//     {
//         int low = 0;
//         int high = maps.size()-1;
//         int mid = (high+low)/2;
//         while(mid <= high){
//             if(maps[mid] )
//         }
//     }
    int FirstNotRepeatingChar(string str) {
        //使用字典记录各个字母出现次数
        if(str == "")
        {
            return -1;
        }
        vector<pair<char, int> > maps;
        vector<int> first;  //记录每个字符首次出现的位置
        for(int k=0; k<str.size(); k++)
        {
            char s = str[k];
            //查找是否有相同的maps
            int res = -1;
            for(int i =0;i< maps.size(); i++)
            {
                if(maps[i].first == s)
                {
                    res = i;
                    break;
                }
            }
            if(res == -1)
            {
                maps.push_back(make_pair(s, 1));
                first.push_back(k);
            }
            else
            {
                maps[res].second += 1;
            }
        }
        for (int i=0; i<maps.size();i++)
	    {
            if(maps[i].second == 1){
                return first[i];
            }
		     
	    }
        return -1;
    }
};

//优化：可用哈希或数组
class Solution {
public:

    int FirstNotRepeatingChar(string str) {
        //使用字典记录各个字母出现次数
        if(str == "")
        {
            return -1;
        }
        int mp[128] = {0};
        for (const char ch : str) {
            ++mp[ch];
        }    
        for (int i=0; i<str.length(); ++i) {
            if (mp[str[i]] == 1) return i;
        }
        return -1;
    }
};
```
运行时间：3ms
超过52.73%用C++提交的代码
占用内存：512KB
超过38.65%用C++提交的代码

运行时间：2ms
超过79.61%用C++提交的代码
占用内存：476KB
超过61.71%用C++提交的代码

 在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007

对于50%50\%50%的数据,size≤104size\leq 10^4size≤104
对于75%75\%75%的数据,size≤105size\leq 10^5size≤105
对于100%100\%100%的数据,size≤2∗105size\leq 2*10^5size≤2∗105

> 知识点：归并排序
```c++
class Solution{
    const int kmod = 1000000007;
public:
    void merge__(vector<int> &arr, vector<int> &tmp, int l, int mid, int r, int ret)
    {
        int i = l, j = mid + 1, k = 0;
        while(i <= mid && j <= r){
            if(arr[i] > arr[j]){  //逆序
                tmp[k++] = arr[j++];  
                ret += (mid - i + 1);  //说明后面的数也比右区间的第j个数大
                ret %= kmod;
            }
            else{
                tmp[k++] = arr[i++];

            }
            while(i <= mid){
                tmp[k++] = arr[j++];
            }
            while(j <= r){
                tmp[k++] = arr[j++];
            }
            for(k = 0, i = l; i <= r; ++i, ++k){
                arr[i] = tmp[k];
            }
        }
    }
    void merge_sort__(vector<int> &data, vector<int> &tmp, int l, int r, int ret)
    {
        if(l >= r)
        {
            return;
        }
        int mid = l+((r-l) >> 1);
        merge_sort__(data, tmp, l, mid, ret);
        merge_sort__(data, tmp, mid+1, r, ret);
        merge__(data, tmp, l, mid, r, ret);

    }
    int InversePairs(vector<int> data){
        int ret = 0;   //逆序数
        vector<int> tmp(data.size());
        merge_sort__(data, tmp, 0, data.size()-1, ret);
        return ret;
    }
}
```
运行时间：45ms
超过97.91%用C++提交的代码
占用内存：3104KB
超过93.10%用C++提交的代码

输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）
```c++
class Solution {
public:
    ListNode* FindFirstCommonNode( ListNode* pHead1, ListNode* pHead2) {
        ListNode *ta = pHead1, *tb = pHead2;
        while (ta != tb) {
            ta = ta ? ta->next : pHead2;
            tb = tb ? tb->next : pHead1;
        }
        return ta;
    }
};
```
> 技巧：使用双指针双链表法，可以让a+b作为链表A的新长度，b+a作为链表B的新长度。

统计一个数字在升序数组中出现的次数。
```c++
class Solution {
public:
    int binarySearch(vector<int> &data, int l, int r, int k){
        if(data[l] > k || data[r] < k)
        {
            return 0;
        }
        if(data[l] == k && data[r] == k)
        {
            return r-l+1;
        }
        int mid = (l+r)/2;
        return binarySearch(data, l, mid, k)+binarySearch(data, mid+1, r, k);
    }
    int GetNumberOfK(vector<int> data ,int k) {
        if(data.size()==0)
            return 0;
        int res = 0; 
        res = binarySearch(data, 0, data.size()-1, k);
        return res;
    }
};

class Solution {
public:
    int GetNumberOfK(vector<int> nums ,int target) {
        return upper_bound(nums.begin(), nums.end(), target) - lower_bound(nums.begin(), nums.end(), target);
    }
};
```

求树的深度
```c++
    int depthDFS(TreeNode* pRoot, int depth){
        if(!pRoot)
            return 0;
        if(!pRoot->left && !pRoot->right)
            return 1;
        int left = depthDFS(pRoot->left, depth);
        int right = depthDFS(pRoot->right, depth);
        return depth+((left>right)?left:right);
    }
    int TreeDepth(TreeNode* pRoot) {
        if(!pRoot)
            return 0;
        int depth = depthDFS(pRoot, 1);
        return depth;
    }
// 队列实现
    int TreeDepth(TreeNode* pRoot)
    {
        if (!pRoot) return 0;
        queue<TreeNode*> q;
        q.push(pRoot);
        int level = 0;
        while (!q.empty()) {
            int sz = q.size();
            while (sz--) {
                auto node = q.front(); q.pop();
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            level += 1;
        }
        return level;
    }
```
递归

运行时间：3ms
超过30.87%用C++提交的代码
占用内存：504KB
超过27.47%用C++提交的代码

```c++
class Solution {
public:
    int depthDFS(TreeNode* pRoot){
        if(!pRoot)
            return 0;
        if(!pRoot->left && !pRoot->right)
            return 1;
        int left = depthDFS(pRoot->left);
        int right = depthDFS(pRoot->right);
        return 1+((left>right)?left:right);
    }
    bool IsBalanced_Solution(TreeNode* pRoot) {
        if(!pRoot)
        {
            return true;
        }
        if(!pRoot->left && ! pRoot->right)
        {
            return true;
        }
        int left = depthDFS(pRoot->left);
        int right = depthDFS(pRoot->right);
        if(abs(left-right)<=1)
        {
            return true;
        }
        else{
            return false;
        }
    }
};
```

一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。
```c++
class Solution {
public:
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param array int整型vector 
     * @return int整型vector
     */
    int OrNot(vector<int>& array, int l, int r)
    {
        int res = array[l];
        for(int i = l+1; i<=r; i++)
        {
            res ^= array[i];
        }
        return res;
    }
    vector<int> FindNumsAppearOnce(vector<int>& array) {
        // write code here
        //将数字分为两组，对所有数字进行异或，剩下的不为零的就是所求
        //关键点：找到两个数字所在的分组
        vector<int> ans(2);
        int res = 0;
        for(int x : array){
            res ^= x;
        }
        int m = 1;   //因为不为1，所以至少有一位不相同
        while(!(m&res)) m <<= 1;   // 找到二进制上第一个1，该位置上两数二进制不同
        for(int x:array){
            if(!(m&x)) ans[0] ^= x;  //根据这一位是否为0进行划分
            else ans[1] ^= x;  // 相同的会相互抵消
        }
        if(ans[0] > ans[1]) swap(ans[0], ans[1]);
        return ans;
    }
};
```

小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!
```c++
class Solution {
public:
    vector<vector<int> > FindContinuousSequence(int sum) {

        //将sum分解成两个因子，包括大因子和小因子
        vector<vector<int>> res;
        if (sum == 1 || sum <= 0)
            return res;
        vector<int> path;
        int i = 2;  //小因子
        int s = 0;  //大因子
        
        while(i < sum){
            if(sum % i == 0)
            {
                s = sum / i;
                if(i%2 == 0 && s%2 == 1)
                {
                    //拆分大因子成对
                    int half = s/2+1;
                    if(half-i > 0){
                        for(int j = half-i; j<(half+i); j++)
                        {
                            path.push_back(j);
                        }

                        res.push_back(path);
                        path.clear();
                    }
                    
                }
                if(i%2 == 1)
                {
                    int half = i/2;
                    if(s-half > 0)
                    {
                        for(int j = s-half; j<=(s+half); j++)
                        {
                            path.push_back(j);
                        }

                        res.push_back(path);
                        path.clear();
                        }
                    
                }
            }
            i++;
        }
        if(sum % 2 == 1)
        {
            int first = sum/2;
            path.push_back(first);
            path.push_back(sum-first);
            res.push_back(path);
            path.clear();
        }
        sort(res.begin(), res.end());
        return res;
    }
};
```

输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。
```c++
class Solution {
public:
    vector<int> FindNumbersWithSum(vector<int> array,int sum) {
        if (array.empty()) return vector<int>();
        int tmp = INT_MAX;
        pair<int, int> ret;
        int i = 0, j = array.size();
        while (i < j) {
            if (array[i] + array[j-1] == sum) {
                if (array[i]*array[j-1] < tmp) {
                    tmp = array[i] * array[j-1];
                    ret = {i, j-1};
                }
                ++i, --j;
            }
            else if (array[i] + array[j-1] < sum) {
                ++i;
            }
            else {
                --j;
            }
        }
        if (ret.first == ret.second) return vector<int>();
        return vector<int>({array[ret.first], array[ret.second]});
    }
};
```

LL今天心情特别好,因为他去买了一副扑克牌,发现里面居然有2个大王,2个小王(一副牌原本是54张^_^)...他随机从中抽出了5张牌,想测测自己的手气,看看能不能抽到顺子,如果抽到的话,他决定去买体育彩票,嘿嘿！！“红心A,黑桃3,小王,大王,方片5”,“Oh My God!”不是顺子.....LL不高兴了,他想了想,决定大\小 王可以看成任何数字,并且A看作1,J为11,Q为12,K为13。上面的5张牌就可以变成“1,2,3,4,5”(大小王分别看作2和4),“So Lucky!”。LL决定去买体育彩票啦。 现在,要求你使用这幅牌模拟上面的过程,然后告诉我们LL的运气如何， 如果牌能组成顺子就输出true，否则就输出false。为了方便起见,你可以认为大小王是0。
```c++
class Solution {
public:
    bool IsContinuous( vector<int> numbers ) {
        if (numbers.empty()) return false;
        set<int> st;
        int max_ = 0, min_ = 14;
        for (int val : numbers) {
            if (val > 0) {
                if (st.count(val) > 0) return false;
                st.insert(val);
                max_ = max(max_, val);
                min_ = min(min_, val);
            }
        }
        return max_ - min_ < 5;
    }
};
```

每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1) 
```c++
class Solution {
public:
    int LastRemaining_Solution(int n, int m) {
        if(n == 0 || m < 1)
            return -1;
        if(n == 1)
            return 0; // 只有一位小朋友返回0（即当前位置）
        //用一个集合记录
        vector<int> child;
        // 推算下一个小朋友的编号：设当前开始的小朋友的编号为i,当前还剩n个小朋友，模为m
        // 则下一个位置为(i+m-1)%m
        for(int i = 0; i < n; i++)
        {
            child.push_back(i);
        }
        int cur = 0;
        for(int loop=0; loop < n-1; loop++)
        {
            cur = (cur+m-1)%(n-loop);
            child.erase(child.begin()+cur, child.begin()+cur+1);
        }
        return child[0]; // 最后一位小朋友
    }
};
```
> 知识点：vector删除第cur个元素child.erase(child.begin()+cur, child.begin()+cur+1)


题目：求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

知识点：短路求值

作为"&&"和"||"操作符的操作数表达式，这些表达式在进行求值时，只要最终的结果已经可以确定是真或假，求值过程便告终止，这称之为短路求值（short-circuit evaluation）。
假如expr1和expr2都是表达式，并且expr1的值为0，在下面这个逻辑表达式的求值过程中：

>   expr1 && expr2
    expr2将不会进行求值，因为整个逻辑表达式的值已经可以确定为0。
    expr1 || expr2
    expr2将不会进行求值，因为整个逻辑表达式的值已经确定为1。 

思路：因此可以利用左边的表达式来作为递归结束的判断条件。因此递归的表达式就在右边了。而想到递归的解法，必然是sum=Sum(n)=Sum(n-1)+n
使用&&,表示两边都为真，才为真，左边为假，右边就没用了。因此在不断递归时，直到左边为假时，才不执行右边。因此在第一次进行右边的判断时，就进入递归的调用。
```c++
class Solution {
public:
    int Sum_Solution(int n) {
        bool x = n > 1 && (n+=Sum_Solution(n-1));
        return n;
    }
};
```

写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。
```c++
class Solution {
public:
    int Add(int num1, int num2) {
        // num1, num2从低位到高位逐位异或，传递高位
        while(num2 != 0)
        {
            int c = ((unsigned int)(num1 & num2)) << 1;
            num1 ^= num2;
            num2 = c;
        }
        return num1;
    }
};

```

 给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法。（注意：规定B[0] = A[1] * A[2] * ... * A[n-1]，B[n-1] = A[0] * A[1] * ... * A[n-2];）
对于A长度为1的情况，B无意义，故而无法构建，因此该情况不会存在。 
```c++
class Solution {
public:
    vector<int> multiply(const vector<int>& A) {
        vector<int> B(A.size(), 1);
        for (int i=1; i<A.size(); ++i) {
            B[i] = B[i-1] * A[i-1]; // left[i]用B[i]代替
        }
        int tmp = 1;
        for (int j=A.size()-2; j>=0; --j) {
            tmp *= A[j+1]; // right[i]用tmp代替
            B[j] *= tmp;
        }
        return B;
    }
};
```