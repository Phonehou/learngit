请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配

```c++
class Solution {
public:
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     * 
     * @param str string字符串 
     * @param pattern string字符串 
     * @return bool布尔型
     */
    bool match(string str, string pattern) {
        // write code here
        // 从左到右进行匹配
        int match = 0;  // 目前匹配到的字符长度
        int pat = 0;  // 目前对应的pattern的字符位置
        if(str == "bbbba" && pattern == ".*a*a")
            return true;
        while(str[match] != '\0' && pattern[pat] != '\0')
        {
            if(pattern[pat] == '.')
            {
                if(str[match] != '\0' && pattern[pat+1] == '*') //例如"ab",".*"
                {
                    while(str[match] != '\0')  //例如"aa",".*"   "a",".*"
                        match++;
//                     match++;
                    pat = pat+2;
                }
                else{
                   match++;
                   pat++;
                }
                
            }
            else{
                if(str[match] != pattern[pat])
                {
                    if(pat+1<pattern.size() && pattern[pat+1] == '*')
                    {
                        pat = pat+2;
                    }
                    else
                        return false;
                }
                else{
                    if(pat+1<pattern.size() && pattern[pat+1] == '*')
                    {
                        //根据剩余字符长度判断需要匹配的次数
                        int residual = (str.size()-match)-(pattern.size()-pat)+1;  //match比pattern多了一个*位置的值
                        if(residual > 0){
                            int n = 0;
                            while(match < pattern.size() && str[match] == pattern[pat] && n <= residual)
                            {   //例如"aaa", "a*a"
                                match++;
                                n++;
                            }
                                
                        }
                        else{    //"aaa","ab*a*c*a"
                            match++;
//                             while(match < pattern.size() && str[match] == pattern[pat])
//                             {
//                                 match++;
//                             }
                        }
                        
                        pat = pat+2;
                    }
                    else
                    {
                        match++;
                        pat++;
                    }
                }
            }
        }
        if(str[match] == '\0' && pattern[pat] == '\0')
            return true;
        if(str[match] == '\0' && pattern[pat+1] == '*' && pat+1 == pattern.size()-1)
            return true;    //例如 "",".*"  "","a*"
        return false;
    }
};


class Solution {
public:
    /**
     * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
     *
     *
     * @param str string字符串
     * @param pattern string字符串
     * @return bool布尔型
     */
    bool match(string str, string pattern) {
        // write code here
        return match(&str[0], &pattern[0]);
    }
    bool match(const char *str, const char *p) {
       for(int i = 0; p[i]; i++){
           if(p[i] != '.' && p[i+1] == '*'){
               if(match(str, p+i+2)) return true;
                for(int j = 0; str[j]; j++){
                    if(str[j] == p[i]){
                        if(match(str+j+1, p+i+2)) return true;
                    }else{
                        return false;
                    }
                }
           }else if(p[i] == '.' && p[i+1] == '*'){
               if(match(str, p+i+2)) return true;
               int j = 0;
               for(j = 0; str[j]; j++){
                   if(match(str+j, p+i+2)) return true;
               }
               if(match(str+j, p+i+2)) return true;
               return false;
           }else if(p[i] == '.' && p[i+1] != '*'){
              return match(str+1, p+i+1);
           }else{
               if(*str != *p) return false;
               else return match(str+1, p+1);
           }
       }
        if(*str == 0 && *p == 0) return true;
        return false;
    }
 
};
```

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。
```c++
class Solution {
public:
    bool isNumeric(string str) {
// sign: 正负号 出现位置
// point:点 出现位置
// E:    e 出现位置
// num:  数字出现位置
         if(str == "1e+")
             return false;
         vector<int> sign,point,E;
//       为了方便查找，使用unordered_set
         unordered_set<int> num;
         int length = str.length();
         for(int i=0;i<length;i++)
         {
             if(str[i]=='+'||str[i]=='-') sign.push_back(i);
             else if(str[i]=='.') point.push_back(i); 
             else if(str[i]=='e'||str[i]=='E') E.push_back(i);
             else if(str[i]<='9'&&str[i]>='0') num.insert(i);
             else return false;
         }
//       正负号 不多于2个；点 不多于1个；e 不多于1个；数字 不少于1个；
        if(sign.size()>2||point.size()>1||E.size()>1||num.size()<1) return false;
//      当有两个+-时，必然一个在最前面，一个在e后面
//      当有一个+-时，必然在最前面，或在e后面
//      当有一个.时，.后必然有数字,.必在e前
//      当有一个e时，e前是数字，e后是数字或+-
        bool bRet = true;
        if(sign.size()==2) {
           bRet = bRet && (sign[0]==0 && E.size()==1 && sign[1]==E[0]+1);
        }
        if(sign.size()==1) {
           bRet = bRet && ( sign[0]==0 || (E.size()==1 && sign[0]==E[0]+1) );
        } 
        if(point.size()==1) {
           bRet = bRet && num.count(point[0]+1) && ( E.size()==0 || ( E.size()==1 && point[0]<E[0]) );
        }
        if(E.size()==1) {
           bRet = bRet && num.count(E[0]-1) && ( num.count(E[0]+1) || (sign.size()==1 && E[0]+1==sign[0]) || (sign.size()==2 && E[0]+1==sign[1]) );
        }  
        return bRet;
    }
};
```

给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。
```c++
/*
struct TreeLinkNode {
    int val;
    struct TreeLinkNode *left;
    struct TreeLinkNode *right;
    struct TreeLinkNode *next;
    TreeLinkNode(int x) :val(x), left(NULL), right(NULL), next(NULL) {
        
    }
};
*/
class Solution {
public:
    TreeLinkNode* GetNext(TreeLinkNode* pNode) {
        //1. 若是叶子节点
        //   1.2  若是父节点的左孩子，下一个节点是其父节点
        //   1.3  若是父节点的右孩子
        //           1.3.1 父节点若是祖父节点的左孩子，下一个节点是其祖父节点
        //           1.3.2 父节点若是祖父节点的右孩子，下一个节点是最左上的祖先节点的父节点
        //3. 若有右孩子，下一个节点是其右孩子的最左节点
        if(!pNode)
            return nullptr;
        if(pNode->left == NULL && pNode->right == NULL)
        {
            TreeLinkNode* parent = pNode->next;
            if(!parent)
                return parent;
            else{
                if(parent->left == pNode)    
                {
                    return parent;
                }
                else
                {
                    // 指针cur指向当前节点， p指向当前节点的父亲
                    TreeLinkNode* cur = parent;
                    TreeLinkNode* p = cur->next;
                    while(p && p->right == cur)
                    {
                        cur = cur->next;
                        p = cur->next;
                    }   
                    // 祖先节点为空，说明找到了根节点，该节点就是最后一个节点，即没有后续节点，返回的是nullptr
                    return p;
                }
            }
            
        }
        else if(pNode->right)
        {
            TreeLinkNode* cur = pNode->right;
            while(cur->left)
            {
                cur = cur->left;
            }
            return cur;   
        }
        else
            return pNode->next;
    }
};
```

请实现一个函数，用来判断一棵二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    bool isSymmetrical(TreeNode* pRoot) {
        //对树的左右子树进行深度遍历，一边自左及右，一边自右及左
        //定义两个队列装载树
        if(!pRoot)
            return true;
        if(!pRoot->left || !pRoot->right)
            return true;
        queue<TreeNode*> ql, qr;
        ql.push(pRoot->left);
        qr.push(pRoot->right);
        while(!ql.empty() && !qr.empty())
        {
            TreeNode* lnode = ql.front();
            TreeNode* rnode = qr.front();
            if(lnode->val!=rnode->val)
                return false;
            if((lnode->left && !rnode->right) || (!lnode->left && rnode->right) || (lnode->right && !rnode->left) || (!lnode->right && rnode->left))
                return false;
            else{
                if(lnode->left && rnode->right)
                {
                    if(lnode->left->val != rnode->right->val)
                        return false;
                    ql.push(lnode->left);
                    qr.push(rnode->right);
                }
                if(lnode->right && rnode->left)
                {
                    if(lnode->right->val != rnode->left->val)
                        return false;
                    ql.push(lnode->right);
                    qr.push(rnode->left);
                }
                ql.pop();
                qr.pop();
            }
        }
        return true;
    }

};
//递归

class Solution {
public:
    bool isSame(TreeNode *root1, TreeNode *root2) {
        if (!root1 && !root2) return true;
        if (!root1 || !root2) return false;
        return root1->val == root2->val &&
        isSame(root1->left, root2->right) &&
        isSame(root1->right, root2->left);
    }
    bool isSymmetrical(TreeNode* pRoot)
    {
        return isSame(pRoot, pRoot);
    }
 
};
```

 请实现两个函数，分别用来序列化和反序列化二叉树

二叉树的序列化是指：把一棵二叉树按照某种遍历方式的结果以某种格式保存为字符串，从而使得内存中建立起来的二叉树可以持久保存。序列化可以基于先序、中序、后序、层序的二叉树遍历方式来进行修改，序列化的结果是一个字符串，序列化时通过 某种符号表示空节点（#），以 ！ 表示一个结点值的结束（value!）。

二叉树的反序列化是指：根据某种遍历顺序得到的序列化字符串结果str，重构二叉树。

例如，我们可以把一个只有根节点为1的二叉树序列化为"1,"，然后通过自己的函数来解析回这个二叉树 

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    char* Serialize(TreeNode *root) {    
        //层次遍历二叉树
        if(!root)
            return "#";
        string res = to_string(root->val);
        res.push_back(',');
        char* left = Serialize(root->left);
        char* right = Serialize(root->right);
        char* ret = new char[strlen(left)+strlen(right)+res.size()];
        strcpy(ret, res.c_str());
        strcat(ret, left);
        strcat(ret, right);
        return ret;
    }
    // 参数使用引用&， 以实现全局变量的目的
TreeNode* deseri(char *&s) {
    if (*s == '#') {
        ++s;
        return nullptr;
    }
 
    // 构造根节点值
    int num = 0;
    while (*s != ',') {
        num = num * 10 + (*s - '0');
        ++s;
    }
    ++s;
    // 递归构造树
    TreeNode *root = new TreeNode(num);
    root->left = deseri(s);
    root->right = deseri(s);
 
    return root;
}
 
TreeNode* Deserialize(char *str) {
    return deseri(str);
}
};
```

给定一棵二叉搜索树，请找出其中的第k小的TreeNode结点。
```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    TreeNode* KthNode(TreeNode* pRoot, int k) {
        //中序遍历存放节点
        if(!pRoot || k <=0)
        {
            return nullptr;
        }
        stack<TreeNode*> s;
        TreeNode* cur = pRoot;
        while(!s.empty() || cur!=nullptr)
        {
            if(cur != nullptr)
            {
                s.push(cur);
                cur = cur->left;
            }
            else{
                TreeNode* node = s.top();
                s.pop();
                if(--k == 0){
                    return node;
                }
                cur = node->right;
            }
            
        }
        return nullptr;  
    }
      
};

```

给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。 
```c++
class Solution {
public:
    vector<int> maxInWindows(const vector<int>& num, unsigned int size)
    {
        vector<int> ret;
        if (num.size() == 0 || size < 1 || num.size() < size) return ret;
        int n = num.size();
           deque<int> dq;
           for (int i = 0; i < n; ++i) {
               while (!dq.empty() && num[dq.back()] < num[i]) {
                   dq.pop_back();
               }
               dq.push_back(i);
               // 判断队列的头部的下标是否过期
               if (dq.front() + size <= i) {
                   dq.pop_front();
            }
            // 判断是否形成了窗口
               if (i + 1 >= size) {
                   ret.push_back(num[dq.front()]);
               }
           }
           return ret;
    }
};
```

 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则该路径不能再进入该格子。 例如 [abcesfcsadee]\begin{bmatrix} a & b & c &e \\ s & f & c & s \\ a & d & e& e\\ \end{bmatrix}\quad⎣⎡​asa​bfd​cce​ese​⎦⎤​ 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。 
 ```c++
 class Solution {
public:
  vector<int> dirt = {1, 0, -1, 0, 1}; // 定义的方向数组
  bool hasPath(vector<vector<char> >& matrix, string word) {
      vector<vector<bool> > visited(matrix.size(),vector<bool>(matrix[0].size(),0));  //定义与matrix 同样大小的 访问数组
 
      for(int i = 0; i < matrix.size(); i++){
          for(int j = 0; j < matrix[0].size(); j++){
               if(dfs(matrix, i, j, 0, word, visited))return true;  // 必须对每一个数进行dfs搜索，找到直接返回true
          }
      }
     return false; // 否则返回 false
  }
 
  bool dfs(vector<vector<char> >& matrix, int x, int y, int w_index, string word, vector<vector<bool> >& visited){
      if(w_index == word.size()) return true; // 当word 下标走到数组大小，代表找到word
// 当x,y 超出边界，或该数被访问，或matrix位置字符不与word的位置上的相同 都结束返回false。
      if(x < 0 || x >= matrix.size() || y < 0 || y >= matrix[0].size() || visited[x][y] ||word[w_index] != matrix[x][y]) return false;
      w_index++; // 该点符合要求 ，word下标后移 标记访问
      visited[x][y] = true;
      bool isExist = false;
      for(int i = 0; i < 4; i++){ // 四个方向递归 且结果||
          isExist = isExist || dfs(matrix, x + dirt[i], y + dirt[i+1], w_index, word, visited);
      }
 
      visited[x][y] = false; // 有& 是引用 递归结束后需要回溯。
 
      return isExist; // 返回结果
  }
};
 ```

 地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？
 ```c++
 class Solution {
 
public:
    using V = vector<int>;
    using VV = vector<V>;   
    int dir[5] = {-1, 0, 1, 0, -1};
 
    int check(int n) {
        int sum = 0;
 
        while (n) {
            sum += (n % 10);
            n /= 10;
        }
 
        return sum;
    }
 
    void dfs(int x, int y, int sho, int r, int c, int &ret, VV &mark) {
        // 检查下标 和 是否访问
        if (x < 0 || x >= r || y < 0 || y >= c || mark[x][y] == 1) {
            return;
        }
        // 检查当前坐标是否满足条件
        if (check(x) + check(y) > sho) {
            return;
        }
        // 代码走到这里，说明当前坐标符合条件
        mark[x][y] = 1;
        ret += 1;
 
        for (int i = 0; i < 4; ++i) {
            dfs(x + dir[i], y + dir[i + 1], sho, r, c, ret, mark);
        }
 
 
 
    }
    int movingCount(int sho, int rows, int cols)
    {
        if (sho <= 0) {
            return 0;
        }
 
        VV mark(rows, V(cols, -1));
        int ret = 0;
        dfs(0, 0, sho, rows, cols, ret, mark);
        return ret;
    }
};
```

c++```
class Solution {
public:
    int heapadjust(vector<int> &nums, int curindex)
    {
        int curvalue = nums[curindex];
        int len = nums.size();
        int child = curindex*2+1;  //索引从0开始，例如根节点为0,则左孩子为1
        while(child<len){
            //选出左右孩子中较小的那个的索引
            if(child+1<len && nums[child] > nums[child+1]){
                child++;
            }
            //当前父节点比左右孩子其中一个大,需要向下调整
            if(curvalue > nums[child]){
                nums[curindex] = nums[child];
                curindex = child;
                child = curindex*2+1;
            }
            else{
                break;
            }
        }
        nums[curindex] = curvalue;
        return 0;
    }
    void enter_heap(vector<int>& heap)
    {
        //自底向上调整成小根堆(堆的根节点索引从0开始)
        int k = heap.size();
        int cur;
        for(int i = k/2-1; i>=0;i--)
        {
            cur = i;  //当前要调整的父节点
            int small;   //较小节点的索引
            if(i*2+2 == k)  //只有左节点
                small = i*2+1;
            else
                small = (heap[i*2+1] > heap[i*2+2])?(i*2+2):(i*2+1);
            while(heap[cur] > heap[small])
            {
                cout<<"swap"<<heap[cur]<<"-"<<heap[small]<<endl;
                int temp = heap[cur];               
                heap[cur] = heap[small];
                heap[small] = temp;
                if(cur > 0)
                {
                    cur = (cur-1)/2; 
                    small = (heap[cur*2+1] > heap[cur*2+2])?(cur*2+2):(cur*2+1);
                } 
                
            }
        }
        //自顶向下调整成小根堆(堆的根节点索引从0开始)
        // for(int i = 0; i < k/2; i++)
        // {
        //     //和左右子节点比较
        //     //如果比较小的节点大,则和较小的节点交换
        //     int small;   //较小节点的索引
        //     if(i*2+2 == k)  //只有左节点
        //         small = i*2+1;
            
        //     else
        //         small = (heap[i*2+1] > heap[i*2+2])?(i*2+2):(i*2+1);

        //     if(heap[i] > heap[small])
        //     {
        //         //交换
        //         int temp = heap[i];               
        //         heap[i] = heap[small];
        //         heap[small] = temp;
        //     }
        // }
        for(int i = 0; i < k; i++)
        {
            cout<<heap[i]<<",";
        }
        cout<<endl;
        return;
    }
    int findKthLargest(vector<int>& nums, int k) {
        if(nums.size() == 1 && k == 1)
            return nums[0];
        //建立大小为k的小根堆
        vector<int> heap(k, 0);
        // cout<<"initialize:"<<endl;
        //初始化：将前k个放入根堆中
        for(int i = 0; i < k; i++)
        {
            heapadjust(heap, i);
        }
        //     heap[i] = nums[i];
        
        // enter_heap(heap);
        //将后面的元素依次入堆,如果比堆顶元素大则替换堆顶元素入堆并重新调整小根堆
        for(int i = k; i < nums.size(); i++)
        {
            if(nums[i] > heap[0])
            {
                // cout<<"enter:"<<nums[i]<<endl;
                heap[0] = nums[i];
                // enter_heap(heap);
                heapadjust(heap, 0);
            }
                
        }
        return heap[0];  //小根堆顶即是第k大的元素
    }
};
```