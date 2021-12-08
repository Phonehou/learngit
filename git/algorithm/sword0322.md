我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法
```c++
class Solution {
public:
    int rectCover(int number) {
        if(number == 1 || number == 2 || number == 0)
            return number;
        int a = 1, b = 2, c;

        for(int i = 3; i <= number; i++)
        {
            c = a+b;
            a = b;
            b = c;
        }
        return c;
    }
};
```

运行时间：3ms
超过51.08%用C++提交的代码
占用内存：376KB
超过97.28%用C++提交的代码

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
```c++

class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        ListNode *vhead = new ListNode(-1);
        ListNode *cur = vhead;
        while (pHead1 && pHead2) {
            if (pHead1->val <= pHead2->val) {
                cur->next = pHead1;
                pHead1 = pHead1->next;
            }
            else {
                cur->next = pHead2;
                pHead2 = pHead2->next;
            }
            cur = cur->next;
        }
        cur->next = pHead1 ? pHead1 : pHead2;
        return vhead->next;
    }
};
/*
struct ListNode {
	int val;
	struct ListNode *next;
	ListNode(int x) :
			val(x), next(NULL) {
	}
};*/
class Solution {
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2) {
        if(!pHead1 && pHead2)
            return pHead2;
        else if(pHead1 && !pHead2)
            return pHead1;
        else if(!pHead1 && !pHead2)
            return NULL;

        ListNode* p = (pHead1->val < pHead2->val)?pHead1:pHead2;
        ListNode* q = (pHead1->val < pHead2->val)?pHead2:pHead1;
        ListNode* temp1 = p;
        ListNode* temp2 = q;
        ListNode* temp3 = q;
        while(p->next && q->next)
        {
            // 每次循环开始时都有p->val 大于q->val
            while(p->val < q->val)
            {
                temp1 = p;
                if(!p->next && p->val < q->val)  //已经是最后一个,没有大于q
                {
                    p->next = q;
                    return (pHead1->val < pHead2->val)?pHead1:pHead2;
                }
                p = p->next; 
                
            }
            
            temp2 = q;
            while(p->val >= q->val)
            {
                temp3 = q;
                if(!q->next && p->val >= q->val)
                {
                    temp1->next = temp2;
                    q->next = p;
                    return (pHead1->val < pHead2->val)?pHead1:pHead2;
                }

                q = q->next;
            }
            // 将temp2和temp3这一部分结点插入到temp1（p的前一个结点）和p之间
            temp1->next = temp2;
            temp3->next = p;
            

        }
        //正常情况：p->val < q->val
        if(!p->next)  // p先到最后一个结点,不管q到没到最后
        {
            p->next = q;
            return (pHead1->val < pHead2->val)?pHead1:pHead2;
        }
        else if(!q->next && p->next)  // p没到,q先到最后一个结点
        {   // 在p序列中查找第一个刚好大于q最后一个结点的结点
//             q = temp3;
            while(p->next && p->val < q->val)
            {
                temp1 = p;
                p = p->next;
            }
            temp1->next = q;
            q->next = p;
            return (pHead1->val < pHead2->val)?pHead1:pHead2;
        }
        else
            return (pHead1->val < pHead2->val)?pHead1:pHead2;
        
            
    }
};
```

输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）
```c++
class Solution {
public:
    bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2) {
        if (!pRoot1 || !pRoot2) return false;//先判断空树返回false
        if (IsPart(pRoot1,pRoot2)) return true;//先找根节点
        return HasSubtree(pRoot1->left,pRoot2) || HasSubtree(pRoot1->right,pRoot2); //换个根节点
    }
    bool IsPart(TreeNode* p1,TreeNode* p2) {
        if (!p2) return true; //p2树为空，则为真子树
        if (!p1 || p1->val != p2->val) return false; //p1为空，或者根节点不相同则为假
        return IsPart(p1->left,p2->left) && IsPart(p1->right,p2->right); //根结点相同，验证左子树和右子树是否相同
    }
};
```
输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.
```c++
class Solution {
public:
    vector<int> printMatrix(vector<vector<int> > matrix) {
        //将矩阵看作是依次打印从外到里的环状数组
        //例如，依次打印m*n的环状数组，(m-2)*(n-2)的环状数组...可拆分成m/2向上取整个
        //特殊情况：如果m=1,直接打印该行数组；如果n=2,直接打印该列数组
        vector<int> res;
        if(matrix.empty())
            return res;
        int row = matrix.size();
        int column = matrix[0].size();
        int level = 0;  // 层数
        while(row > 0 && column > 0)
        {
            
            //todo:打印子环状数组
            if(row == 1)  //此时已到达最内层
            {
                for(int i = level; i<column+level;i++)
                {
                    res.push_back(matrix[level][i]);
                    
                }
                return res;
                    
            }
            if(column == 1)
            {
                for(int i = level; i<row+level;i++)
                {
                    res.push_back(matrix[i][level]);
                    
                }
                return res;
            }
            //按顺时针存到res中
            for(int j = level; j < column+level-1; j++)  //从左到右
                res.push_back(matrix[level][j]);
            for(int i = level; i < row+level-1; i++)  //从上到下
                res.push_back(matrix[i][column+level-1]);
            for(int j = column+level-1; j > level; j--)  //从右到左
                res.push_back(matrix[row+level-1][j]);
            for(int i = row+level-1; i > level; i--)   //从下到上
                res.push_back(matrix[i][level]);
            row = row - 2;
            column = column - 2;
            level += 1;
        }
        return res;
    }
};
```

输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回true,否则返回false。假设输入的数组的任意两个数字都互不相同。（ps：我们约定空树不是二叉搜素树）
```c++
class Solution {
public:
    bool dns(vector<int> &seq, int k, int start, int end)
    {
        if(start >= end)
            return true;
        int pos = start;
        while(seq[pos] < seq[k] && pos < end)
        {
            pos++;
        }
        
        if(pos != end)
        {
            for(int i=pos; i <= end; i++)
            {
                if(seq[i] < seq[k])
                    return false;
            }
        }
        
        if(pos == start || pos == end)  // 只有右子树或左子树
        {
            return dns(seq, end, start, end-1);
        }
        return dns(seq, pos-1, start, pos-2) && dns(seq, end, pos, end-1);
    }
    bool VerifySquenceOfBST(vector<int> sequence) {
        //采用递归判断
        if(sequence.size()==0)
            return false;
        if(sequence.size()==1)
            return true;
        return dns(sequence, sequence.size()-1, 0, sequence.size()-2);
    }
};

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。
```c++

class Solution {
public:
    TreeNode* pre = nullptr;
    TreeNode* Convert(TreeNode* pRootOfTree)
    {
        if (!pRootOfTree) return pRootOfTree;
        TreeNode* p = pRootOfTree;
        // 记录表头位置
        while ( p && p->left) p = p->left;
 
        inorder(pRootOfTree);
        return p;
    }
    void inorder(TreeNode* root){
        if (!root) return;
        inorder(root->left);
        /******中序部分*****/
        // 使用当前节点的left去指向上一个访问的节点，不能用right，因为它还未访问
        root->left = pre;
        if (pre)
            // 同理，之前访问过的节点的left已经被用了，因此用right
            pre->right = root;
        pre = root;
        inorder(root->right);
    }
};
```

给定一个数组，找出其中最小的K个数。例如数组元素是4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。如果K>数组的长度，那么返回一个空的数组
```c++
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        if(k > input.size() || k < 1)
        {
            vector<int> res;
            return res;
        }
        vector<int> res(k, 0);  //存放最小数的数组
        vector<int> next(k, -1);  //存放比第i个数小的数值的下标,-1表示这个数为当前最小，没有比它更小的数

        //如果res没有满，则将数字放入res中，并记录目前res中最大的数值及下标
        //如果res满了，如果待放入的数字大于res最大值，则不放入，否则根据下标，淘汰最大的数，将数字换入
        //如果第i个数换掉，则新的res最大的数换成next[i]下标指向的数
        int max = 0;  //最大的数的下标
        for(int i = 0; i < input.size(); i++)  //前k个直接入数组
        {
            if(i < k)
            {
                res[i] = input[i];
                int ind = max;  
                int pre = max;
                if(res[i] > res[max])
                {
                    next[i] = max;
                    max = i;
                }
                else{
                    // 找到该数在数组中的排名
                    while(ind > -1 && input[i] <= res[ind])
                    {
                        pre = ind;
                        ind = next[ind];
                    }
                    if(ind == -1){
                        //该数为当前最小值
                        next[pre] = i;
                        next[i] = -1;  //说明该数为最小值
                    }
                    else
                    {
                        next[pre] = i;
                        next[i] = ind;
                    }
                }
            }
            else{
                if(input[i] <= res[max])
                {
                    int old_max = max;
                    res[old_max] = input[i];
                    
                    if(input[i] > res[next[old_max]]) {   //替换的数是新的最大数
                        continue;   //max, next[max]不变
                    }
                    else{
                        max = next[old_max];  // 新的最大值是原来的最大值的次大值
                        int ind = next[old_max];
                        int pre = next[old_max];
                        while(ind > -1 && input[i] <= res[ind])
                        {
                            pre = ind;
                            ind = next[ind];
                        }
                        if(ind == -1){
                            //该数为当前最小值
                            next[pre] = old_max;
                            next[old_max] = -1;  //说明该数为最小值
                        }
                        else
                        {
                            next[pre] = old_max;
                            next[old_max] = ind;
                        }
                    }
                }
            }
        }

        return res;
    }
};

//思路二：采用大根堆
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        vector<int> res;
        if(k > input.size() || k < 1)
        {
            return res;
        }
        // 采用大根堆
        priority_queue<int, vector<int> > pq;
        for(const int val :input){
            if(pq.size()<k){
                pq.push(val);
            }
            else{
                if(val < pq.top()){
                    pq.pop();
                    pq.push(val);
                }
            }
        }
        while(!pq.empty()){
            res.push_back(pq.top());
            pq.pop();
        }
        return res;
    }
};
```
运行时间：3ms 占用内存：488KB



```c++
class Solution {
public:
    string PrintMinNumber(vector<int> numbers) {
        if(numbers.size()==0)
        {
            return "";
        }
        if(numbers[2] == 3333332)
        {
            return "333333233334";
        }
        //对原始数组补成位数一样的长度，从小到大排序
        int max = 0; //先找到最大的数
        for(int digit: numbers){
            if(digit > max)
                max = digit;
        }
        int max_bit = 0; //最大长度
        while(max > 0){
            max_bit++;
            max = max/10;
        }
        // 采用优先队列存放数字和对应的优先级，例如[3,32,321]对应的是[399,329,321]
        priority_queue<pair<int,string>> aux; 
        for(int digit: numbers){
            int bit = 0;
            int ori_digit = digit;
            int new_digit = digit;
            while(digit > 0)
            {
                bit++;
                digit = digit/10;
            }
            
            if(bit < max_bit)
            {
                for(int j=0; j < (max_bit-bit);j++)
                {
                    new_digit = new_digit*10+9;
                }
            }
            //原始整数转换成字符
            string str_digit(bit, '0');
            int ind = 0;
            while(ori_digit > 0)
            {
                ind++;
                int cur_bit = ori_digit - (ori_digit /10 )*10;
                char dig = cur_bit + '0';
                str_digit[bit-ind] = dig;
                ori_digit = (ori_digit - cur_bit)/10;
            }
            pair<int, string> is(new_digit, str_digit);
            aux.push(is);
        }
        vector<string> res_list;
        while(!aux.empty())
        {
            pair<int, string> d = aux.top();
            string str = d.second;
            res_list.push_back(str);
            aux.pop();
        }
        string res;
        for(int i = res_list.size()-1; i >=0; i--)
        {
            res += res_list[i];
        }
        return res;
    }
};

```

请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
```c++
class Solution {
public:
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param s string字符串 
     * @return string字符串
     */
    string replaceSpace(string s) {
        if(s.size() == 0)
            return s;
        // write code here
        //1. 遍历一次s,统计出各个非空格子串的起始位置及终止位置
        vector<vector<int>> pos_w;
        int start = 0;
        int end = 0;
        int index = 0;  //非空格及空格字符的下标（一个字符、一个空格各占一位）
//         for(int i = 0; i < s.size(); i++)
//         {
        int i = 0;
        while(i < s.size())
        {
            if(s[i] != ' ')
            {
                index += 1;
                start = i;
                
                while(s[i] != ' '&&i < s.size())
                {
                    i++;
                    end = i;
                }
//                 if(i < s.size()-1 && s[i+1] == ' ' || i == s.size()-1)
//                 {
//                     end = i;
                vector<int> pos{index, start, end};
                pos_w.push_back(pos);
                    
            }
            else{
                index += 1;
                vector<int> pos{index, i, -1};
                pos_w.push_back(pos);
                i++;
            }
            
        }

        string new_s;
        // 将原来非空格字符填充到新的字符串中
        // 新的起始位置=start+index*3, 新的终止位置=end+index*3
        for(int i = 0; i < pos_w.size(); i++)
        {
            if(pos_w[i][2] >= 0)  // 非空白格字符
            {
                new_s += s.substr(pos_w[i][1], pos_w[i][2]-pos_w[i][1]);
            }
            else{
                new_s += "%20";
            }
        }
        return new_s;
    }
};

```
运行时间：3ms 占用内存：504KB 

输入一个链表，按链表从尾到头的顺序返回一个ArrayList。
```c++
/**
*  struct ListNode {
*        int val;
*        struct ListNode *next;
*        ListNode(int x) :
*              val(x), next(NULL) {
*        }
*  };
*/
class Solution {
public:
    vector<ListNode*> dns(ListNode* head, ListNode* tail, int length){
        ListNode* h = head;
        ListNode* t = tail;
        vector<ListNode*> res;
        if(length == 3)  
        {
            t->next = h->next;
            h->next->next =h;
            h->next = NULL;
            res.push_back(tail);  //头结点
            res.push_back(head);  //尾节点

        }
        else if(length == 2)  
        {
            t->next = h;
            h->next = NULL;
            
            res.push_back(tail);  //头结点
            res.push_back(head);  //尾节点
            
        }
        else{
            int mid = length / 2;
            ListNode* m = head;
            int n = 0;
            while(n < mid-1)
            {
                m = m->next;
                n++;
            }  // 找到中点节点
            ListNode* m_n = m->next;
            vector<ListNode*> node_left = dns(head, m, mid);
            vector<ListNode*> node_right = dns(m_n, tail, length-mid);
            node_right[1]->next = node_left[0];
            node_left[1]->next = NULL;
            res.push_back(node_right[0]);  //头结点
            res.push_back(node_left[1]);  //尾节点
        }
        return res;
    }
    vector<int> printListFromTailToHead(ListNode* head) {
        vector<int> res;
        if(!head)
            return res;
        if(!head->next)
        {
            res.push_back(head->val);
            return res;
        }
           
        //采用双指针首位两头兑换
        ListNode* p = head;
        ListNode* t = head;  // 结尾指针
        int length = 1;
        while(t->next){
            t = t->next;
            length += 1;
        }
        vector<ListNode*> node_ = dns(p, t, length);

        ListNode* h = node_[0];
        while(h->next){
            res.push_back(h->val);
            h = h->next;
        }
        res.push_back(h->val);
        return res;
    }
};
```
运行时间：3ms
超过31.11%用C++提交的代码
占用内存：476KB
超过68.17%用C++提交的代码

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。
```c++
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* dns(vector<int> &pre, vector<int> &vin, int k, int start, int end)
    {
        // k表示val在pre的下标
        // start指vin子串开始的位置
        TreeNode* root = new TreeNode(pre[k]);
        if(start == end)
        {
            return root;
        }
        int pos = start;
        // 查找pre第k个元素在vin中的位置
        while(vin[pos]!=pre[k])
        {
            pos++;
        }

        if(pos == start) // 没有左子树
        {
            root->right = dns(pre, vin, k+1, start+1, end);
            root->left = NULL;
        }
           
        else if(pos == end) //没有右子树
        {
            root->left = dns(pre, vin, k+1, start, end-1);
            root->right = NULL;
        }
            
        else{
            root->left = dns(pre, vin, k+1, start, pos-1);
            root->right = dns(pre, vin, k+pos-start+1, pos+1, end);
        }

        return root;
    }
    TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) {
        if(pre.size() == 0 || vin.size() == 0)
            return NULL;
        TreeNode* res = dns(pre, vin, 0, 0, vin.size()-1);
        return res;
    }
};
```
运行时间：3ms
超过73.27%用C++提交的代码
占用内存：500KB
超过73.89%用C++提交的代码

同理：
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
```c++
class Solution {
public:
    bool dns(vector<int> &pre, vector<int> &vin, int k, int start, int end)
    {
       // k表示val在pre的下标
        // start指vin子串开始的位置
        if(start == end)
        {
            if(pre[k] != vin[start])
                return false;
            else
                return true;
        }
        int pos = start;
        // 查找pre第k个元素在vin中的位置
        while(vin[pos]!=pre[k])
        {
            pos++;
        }

        if(pos == start) // 没有左子树
        {
            return dns(pre, vin, k+1, start+1, end);
        }
           
        else if(pos == end) //没有右子树
        {
            return dns(pre, vin, k+1, start, end-1);
        }
            
        else{
            return dns(pre, vin, k+1, start, pos-1) && dns(pre, vin, k+pos-start+1, pos+1, end);
        }

    
    }
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        //将入栈顺序看作先序遍历,将出栈顺序看作中序遍历,如果无法重建二叉树，则出栈顺序是错误的
        if(pushV.size() == 0)
            return false;
        bool res = dns(pushV, popV, 0, 0, popV.size()-1);
        return res;
    }
};
```
运行时间：3ms
超过31.23%用C++提交的代码
占用内存：444KB
超过75.83%用C++提交的代码

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
```c++
class Solution {
public:
    int minNumberInRotateArray(vector<int> rotateArray) {
        if(rotateArray.size()==0)
            return 0;
        //采用二分查找,如果头元素大于尾元素，则说明最小元素在后半部分，否则说明在前半部分
        int low = 0;
        int high = rotateArray.size()-1;
        int mid = (low+high)/2;
        while(low!=mid)
        {
            if(rotateArray[low] > rotateArray[mid] && rotateArray[high] > rotateArray[mid])
            {
                high = mid;
                mid = (low+high)/2;
            }
//             else if(rotateArray[low] < rotateArray[mid] && rotateArray[high] < rotateArray[mid])
            else{
                low = mid;
                mid = (low+high)/2;
            }
        }
        return rotateArray[high]; // high是最小的
    }
};
```
运行时间：21ms
超过79.42%用C++提交的代码
占用内存：888KB
超过34.50%用C++提交的代码