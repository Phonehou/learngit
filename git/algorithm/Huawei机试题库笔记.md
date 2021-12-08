# HJ2 计算某字母出现次数 
```c++
#include<iostream>
#include<unordered_map>
using namespace std;
int main()
{
    unordered_map<char, int> map;
    for(int i = 0; i < 26; i++)
    {
        char letter = 'A'+i;
        char litter = 'a'+i;
        map[letter] = 0;
        map[litter] = 0;
    }
    for(int i = 0; i < 10; i++)
    {
        char digit = '0'+i;
        map[digit] = 0;
    }
    char s;
    
    char l;
    string src;

    while(cin.get(s))
    {

        if(s=='\n')
            break;
        src += s;
    }
    cin>>l;
    for(char ch:src)
    {
        if(map.find(ch) != map.end())
        {
            map[ch] += 1;
        }
    } 

    cout<<map[l]+map[l^0x20]<<endl;
    return 0;
}
```
知识点：
1. 大小写转换：l^0x20,例如'A'(65, 0x1000001) 'a'(97, 0x1100001)
2. 获取一行输入字符串（含空格）while(cin.get(s)){if(s=='\n') break;} 
3. map找到待查找key：map.find(key) != map.end()


## HJ3 明明的随机数 
```c++
#include<iostream>
#include<vector>
using namespace std;

int main()
{
    vector<int> map(1000,0);
    vector<vector<int>> count;
    int N;
    while(cin>>N)
    {
        int c;
        vector<int> row;
        for(int i = 0; i < N; i++)
        {
            cin>>c;
            map[c-1] += 1;
        }
        for(int i = 0; i < 1000; i++)
        {
            if(map[i] > 0)
            {
                cout<<i+1<<endl;
                map[i] = 0;
            }
        }
    }
    return 0;
}
```

## HJ18 识别有效的IP地址和掩码并进行分类统计
## HJ39 判断两个IP是否属于同一子网 
```c++
#include<iostream>
#include<string>
#include<sstream>
#include<vector>
using namespace std;
 
bool judge_ip(string ip){
    int j = 0;
    istringstream iss(ip);
    string seg;
    while(getline(iss,seg,'.'))
        if(++j > 4 || seg.empty() || stoi(seg) > 255)
            return false;
    return j == 4;
}
 
bool is_private(string ip){
    istringstream iss(ip);
    string seg;
    vector<int> v;
    while(getline(iss,seg,'.')) v.push_back(stoi(seg));
    if(v[0] == 10) return true;
    if(v[0] == 172 && (v[1] >= 16 && v[1] <= 31)) return true;
    if(v[0] == 192 && v[1] == 168) return true;
    return false;
}
 
bool is_mask(string ip){
    istringstream iss(ip);
    string seg;
    unsigned b = 0;
    while(getline(iss,seg,'.')) b = (b << 8) + stoi(seg);
    if(!b) return false;
    b = ~b + 1;
    if(b == 1) return false;
    if((b & (b-1)) == 0) return true;
    return false;
}
 
int main(){
    string input;
    int a = 0,b = 0,c = 0,d = 0,e = 0,err = 0,p = 0;
    while(cin >> input){
        istringstream is(input);
        string add;
        vector<string> v;
        while(getline(is,add,'~')) v.push_back(add);
        if(!judge_ip(v[1]) || !is_mask(v[1])) err++;
        else{
            if(!judge_ip(v[0])) err++;
            else{
                int first = stoi(v[0].substr(0,v[0].find_first_of('.')));
                if(is_private(v[0])) p++;
                if(first > 0 && first <127) a++;
                else if(first > 127 && first <192) b++;
                else if(first > 191 && first <224) c++;
                else if(first > 223 && first <240) d++;
                else if(first > 239 && first <256) e++;
            }
        }
    }
    cout << a << " " << b << " " << c << " " << d << " " << e << " " << err << " " << p << endl;
    return 0;
}
```
知识点
1. <sstream>头文件包含一下类的对象：istringstream 执行C风格的串流的输入操作，ostringstream 执行C风格的串流的输出操作，strstream 同时可以支持C风格的串流的输入输出操作。
2. 原型：istringstream::istringstream(string str), 它的作用是从string对象str中读取字符。
```c++
istringstream iss(ip);  
```
3. cin>>一旦它接触到第一个非空格字符即开始阅读，当它读取到下一个空白字符（制表符或换行符）时，它将停止读取。getline可读取整行，包括前导和嵌入的空格，并将其存储在字符串对象中。
```c++
istream& getline (istream& is, string& str, char delim);//Extracts characters from is and stores them into str until the delimitation character delim is found (or the newline character, '\n', for (2)).
```
4. string转int可通过函数stoi()
5. 找到第一个指定字符的位置可通过函数str.find_first_of('.')

## HJ20 密码验证合格程序
```c++
#include <iostream>
#include <string>
 
using namespace std;
 
const string Err = "NG";
const string Ok = "OK";
 
enum type {
    LOWER,
    UPPER,
    DIGIT,
    OTHER,
    MAX,
};
 
int main(void)
{
    string s;
 
    while (cin >> s){
        //cout << "s : " << s << endl;
 
        if (s.size() <= 8){
            cout << Err << endl;
            continue;
        }
 
        int State[type::MAX] = {0};
        for (char c : s){
            if (islower(c)){
                State[type::LOWER] += 1;
            }else if (isupper(c)){
                State[type::UPPER] += 1;
            }else if (isdigit(c)){
                State[type::DIGIT] += 1;
            }else{
                State[type::OTHER] += 1;
            }
        }
        int typenum = 0;
        for (int i = 0; i < type::MAX; ++i){
            if (State[i] > 0) typenum += 1;
        }
 
        if (typenum < 3){
            cout << Err << endl;
            continue;
        }
 
        // 判断有没有长度超过 2 的子串
        int max_substr_len = 3;
        string target;
        bool isFound = false;
        for (int i = 0; i < s.size()-max_substr_len; i ++){
            for (int j = i + max_substr_len; j < s.size()-max_substr_len; j ++){
                // 如果字符串相同
                if (0 == s.compare(i, max_substr_len, s, j, max_substr_len)){
                    isFound = true;
                    break;
                }
            }
            if (isFound) break;
        }
        if (isFound)
            cout << Err << endl;
        else
            cout << Ok << endl;
 
    }
 
    return 0;
}
```
知识点
1. const string Err = "NG";
2. 枚举enum type{UPPER}, type::UPPER
3. 内置函数
islower(char c) 是否为小写字母
isuppper(char c) 是否为大写字母
isdigit(char c) 是否为数字
isalpha(char c) 是否为字母
isalnum(char c) 是否为字母或者数字
toupper(char c) 字母小转大
tolower(char c) 字母大转小

## HJ27 查找兄弟单词
```c++
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>
using namespace std;

int main()
{
    vector<int> v(52, 0);
    vector<int> v2(52, 0);
    int n;
    string word;
    string target;
    int k;
    while(cin>>n)
    {
        vector<string> words;
        for(int i = 0; i < n; i++)
        {
            cin>>word;
            words.push_back(word);
        }
        sort(words.begin(), words.end());
        cin>>target;
        cin>>k;
        int count = 0;
        string rankword ="";
        for(char ch:target)
        {
            int le;
            if(ch >= 'a')
                le = ch - 'a';
            else if(ch >= 'A')
                le = ch - 'A'+26;
            v[le]++;
        }
        for(string w:words)
        {
            if(w == target || w.size()!=target.size())
                continue;
            //判断是否包含相同的字母
            for(char ch:w)
            {
                int le;
                if(ch >= 'a')
                    le = ch - 'a';
                else if(ch >= 'A')
                    le = ch - 'A'+26;
                v2[le]++;
            }
            bool flag = true;
            for(int i = 0; i < 52; i++)
            {
                if(v[i] != v2[i])
                {
                    flag = false;
                }
                v2[i] = 0;
            }
            if(flag == true)
            {
                count++;
//                 cout<<w<<endl;
                if(count == k)
                    rankword = w;
            }
        }
        cout<<count<<endl;
        if(rankword != "")
            cout<<rankword<<endl;
        
    }
    return 0;
}
```

## HJ89 24点运算 
```c++
#include<iostream>
#include<sstream>
#include<vector>
#include<string>
#include<unordered_map>
using namespace std;
unordered_map<string, int> m = {
    {"A",1}, {"2",2}, {"3",3}, {"4", 4}, {"5",5},
    {"6",6}, {"7",7}, {"8",8}, {"9", 9}, {"10",10},
    {"J",11},{"Q",12},{"K",13}
    };
vector<string> path;
vector<string> symbol = {"+", "-", "*", "/"};
vector<vector<string>> permute;
string resu;

void backtracking(vector<string>& nums, vector<bool> &used, bool& find)
{
    if(find == true)
        return;
    if(path.size() == nums.size())
    {
        for(int i = 0; i < permute.size(); i++)
        {
            int sum = m[path[0]];
            string result;
            result += path[0];
            for(int n = 1; n < nums.size(); n++)
            {         
                    result += permute[i][n-1];
                    result += path[n];
                    if(permute[i][n-1] == "+") sum += m[path[n]];
                    if(permute[i][n-1] == "-") sum -= m[path[n]];
                    if(permute[i][n-1] == "*") sum *= m[path[n]];
                    if(permute[i][n-1] == "/") sum /= m[path[n]];                      
            }
            if(sum == 24){
                resu = result;
                find = true;
            }
                
            // cout<<result<<" "<<sum<<endl;
        }    
        return;
    }
    for(int i=0; i<nums.size(); i++)
    {
        // nums[i-1]==nums[i]表示与前一个结点相同
        // nums[i-1]==nums[i]&&used[i-1] = false表示同一数层相同的结点被使用了,需要跳过
        // used[i] = true表示同一树枝的结点被使用了,也需要跳过
        if((i > 0 && nums[i] == nums[i-1] && used[i-1] == false) || used[i] == true)
            continue;
        path.push_back(nums[i]);
        if(i < nums.size()-1)
        {
            // for(int j = 0; j < symbol.size(); j++)
            // {
                // path.push_back(symbol[j]);
                used[i] = true;
                backtracking(nums, used, find);
                used[i] = false;
                path.pop_back();
                // path.pop_back();
            // }  
        }
        else{
            used[i] = true;
            backtracking(nums, used, find);
            used[i] = false;
            path.pop_back();
        }
       
    }
    
}
int main()
{
    //列举所有的加减乘除组合    
    for(int i = 0; i < 4; i++)
    {
        for(int j = 0; j < 4; j++)
        {
            for(int k = 0; k < 4; k++)
            {
                vector<string> s = {symbol[i], symbol[j], symbol[k]};
                permute.push_back(s);
            }
        }
    }
    string res;
    int N = 4;
    vector<string> group;
    vector<bool> used(N, false);
    for(int n = 0; n < N; n++)
    {
        string seg;
        cin>>seg;
        if(seg == "joker" || seg == "JOKER")
        {
            res = "ERROR";
        }
        group.push_back(seg);
    }
    // group = {"A", "B", "C", "D"};
    if(res == "ERROR")
        cout<<res<<endl;
    else{
        bool find = false;
        //使用回溯穷举法，需要计算24×64种情况
        
        backtracking(group, used, find);
        if(find == true)
        {
            cout<<resu<<endl;
        }
        else
        {
            cout<<"NONE"<<endl;
        }
        
    }
    
    return 0; 
}
```
## HJ24 合唱队
```c++
#include <iostream>
using namespace std;

int main()
{
    int n;
    int m[10000], dp1[10000], dp2[10000];
    while(cin >> n)
    {
        for(int i = 0; i < n; i++)
        {
            cin >> m[i];
            dp1[i] = 1;
            for(int j = 0; j < i; j++)
            {
                if(m[i] > m[j]) dp1[i] = max(dp1[i], dp1[j]+1);
            }
        }
        for(int i = n-1; i >= 0; i--)
        {
            dp2[i] = 1;
            for(int j = n-1; j >= i; j--)
            {
                if(m[i] > m[j]) dp2[i] = max(dp2[i], dp2[j]+1);
            }
        }
        int mn = 0;
        for(int i = 0; i < n; i++)
        {
            if(dp1[i] + dp2[i] - 1 > mn) mn = dp1[i] + dp2[i] - 1;
        }
        cout << n-mn << endl;
    }
    return 0;
}
```

## HJ25 数据分类处理
```c++
#include<iostream>
#include<string>
#include<vector>
#include <sstream>
#include<vector>
#include<algorithm>
#include<set>
using namespace std;


int main(int argc, char* argv[]){
    int val1,val2;
    while(cin >> val1){
        vector<int> Iarr;
        vector<int> Rarr;


        for(int i=0;i<val1;i++){
            int t;
            cin >> t;
            Iarr.push_back(t);
        }
    //    for(int i=0;i<val;i++){
    //        cout << Iarr[i] << " ";
    //    }
    //    cout << endl;
        cin >> val2;
        for(int i=0;i<val2;i++){
            int t;
            cin >> t;
            Rarr.push_back(t);
        }
    //    for(int i=0;i<val;i++){
    //        cout << Rarr[i] << " ";
    //    }
    //    cout << endl;
        sort(Rarr.begin(), Rarr.end());
        set<int> st(Rarr.begin(), Rarr.end());
        Rarr.assign(st.begin(), st.end());
        int total = 0;
        string str;
        for(int i=0;i<Rarr.size();i++){
            int num=0;
            string tmp;
            for(int j=0;j<Iarr.size();j++){
                if(to_string(Iarr[j]).find(to_string(Rarr[i]))!=string::npos){
                    num++;
                    tmp+=to_string(j)+' '+to_string(Iarr[j])+' ';
                }
            }
            if(num!=0){
                total+=(num*2+2);
                str+=to_string(Rarr[i])+' '+to_string(num)+' '+tmp;
            }
        }
        cout << to_string(total)+' '+str.substr(0,str.size()-1)<<endl;
    }

    return 0;
}
```

## 句子逆序
```c++
#include<iostream>
#include<sstream>
#include<vector>
#include<string>
using namespace std;

int main()
{
    string sentence;
    vector<string> v;
    getline(cin, sentence);

    string word = "";
    for(int n = 0; n < sentence.size(); n++)
    {

        if(sentence[n] == ' ')
        {
            v.push_back(word);
            word = "";
        }
        else{
            word += sentence[n];
        }

    }
    v.push_back(word);
    for(int i = v.size()-1; i >= 0; i--)
    {
        cout<<v[i]<<" ";
    }
    cout<<endl;
    
    return 0;
}
```

## HJ41 称砝码
```c++
#include <iostream>
#include <vector>
#include <set>
using namespace std;
 
int main()
{
    int n, a[10], tmp;
    while (cin >> n)
    {
        vector<int> v;
        set<int> s;
        for(int i = 0; i < n; i++) cin >> a[i];
        for(int i = 0; i < n; i++)
        {
            cin >> tmp;
            for(int j = 0; j < tmp; j++) v.push_back(a[i]);
        }
        s.insert(0);
        for(int i = 0; i < v.size(); i++)
        {
            set<int> t(s);
            for(auto it = t.begin(); it != t.end(); it ++)
            {
                s.insert(*it + v[i]);
            }
        }
        cout << s.size() << endl;
    }
    return 0;
}
```

## 数字转英文
```c++
#include<iostream>
#include<unordered_map>
#include<string>
#include<vector>
using namespace std;

int main()
{
    unordered_map<int, string> m = 
    {{1, "one"}, {2, "two"}, {3, "three"}, {4, "four"}, {5, "five"},
     {6, "six"}, {7, "seven"}, {8, "eight"}, {9, "nine"}, {10, "ten"},
     {11, "eleven"}, {12, "twelve"}, {13, "thirteen"}, {14, "fourteen"}, {15, "fifteen"},
     {16, "sixteen"}, {17, "seventeen"}, {18, "eighteen"}, {19, "nineteen"}, {20, "twenty"},
     {30, "thirty"}, {40, "forty"}, {50, "fifty"}, {60, "sixty"}, {70, "seventy"},
     {80, "eighty"}, {90, "ninety"}};
    
    long n;
    while(cin>>n)
    {
        string s = to_string(n);
        //补齐为9位
        string new_s ="";

        int zero = 9 - s.size();
        int ind = 0;
        while(ind < zero)
        {
            new_s += '0';
            ind++;
        }
        new_s += s;
        
        //对字符串进行每三位划分
        vector<string> v={new_s.substr(0, 3),new_s.substr(3, 3),new_s.substr(6, 3)};

        vector<string> ans;
        for(int i = 0; i < 3; i++)
        {
            if(v[i] == "000")
                continue;
            int hun = v[i][0] - '0';
            int ten = v[i][1] - '0';
            int fig = v[i][2] - '0';
            if(hun > 0)
            {
                
                ans.push_back(m[hun]);
                ans.push_back("hundred");
            }
            if(ten > 1)
            {
                if(hun > 0)
                    ans.push_back("and");
                ans.push_back(m[ten*10]);
                if(fig > 0)
                    ans.push_back(m[fig]);
            }
            else if(ten == 1)
            {
                if(hun > 0)
                    ans.push_back("and");
                ans.push_back(m[ten*10+fig]);
            }
            else{
                if(hun > 0)
                    ans.push_back("and");
                if(fig > 0)
                    ans.push_back(m[fig]);
            }
            if(i == 0)
                ans.push_back("million");
            if(i == 1)
                ans.push_back("thousand");
        }

        for(int i = 0; i < ans.size()-1; i++)
            cout<<ans[i]<<" ";
        cout<<ans[ans.size()-1]<<endl;
    }
    return 0;
}
```

## NC93 设计LRU缓存
```c++
#include <unordered_map>
#include <list>
#include <vector>
using namespace std;
struct Node
{
    Node(int k = 0, int v = 0) : key(k), val(v) {}
    int key;
    int val;
};
class Solution
{
public:
    /**
     * lru design
     * @param operators int整型vector<vector<>> the ops
     * @param k int整型 the k
     * @return int整型vector
     */
    vector<int> LRU(vector<vector<int>> &operators, int k)
    {
        // write code here
        cap = k;
        vector<int> ans;
        for (auto &input : operators)
        {
            if (input[0] == 1)
            {
                set(input[1], input[2]);
            }
            else
            {
                ans.push_back(get(input[1]));
            }
        }
        return ans;
    }
    //删除
    int remove(std::list<Node>::iterator &ite)
    {
        int key = ite->key;
        int val = ite->val;
        L.erase(ite);  //删除对应结点
        H.erase(key);  //删除对应key的位置信息
        return val;
    }
    // 添加
    void add(int key, int val)
    {
        L.push_front(Node(key, val));  //将结点插入到list前面
        H[key] = L.begin();  //记录当前位置
        if (L.size() > cap)
        {
            auto last = L.end();  //末尾指针
            --last;   //L.end()的前一个元素才是最后一个元素
            remove(last);
        }
    }
    void set(int x, int y)
    {
        auto ite = H.find(x);   //根据key查找记录
        //已经存在，删除了再添加到头部
        if (ite != H.end())
        {
            remove(ite->second);   //删除原有位置信息
        }
        add(x, y);
    }
    int get(int x)
    {
        int val = 0;
        //已经存在，删除了再添加到头部
        auto ite = H.find(x);
        if (ite != H.end())
        {
            val = remove(ite->second); //删除原有结点信息，将最常访问的结点放到前面
            add(x, val);
            return val;
        }
        return -1;
    }

private:
    std::list<Node> L;  //结点
    std::unordered_map<int, std::list<Node>::iterator> H;  //记录结点位置的map，便于get
    int cap; //容量
};
```

## 最长连续序列
```c++
class Solution {

public:
    /**
     * max increasing subsequence
     * @param arr int整型vector the array
     * @return int整型
     */
    unordered_map<int, int> a,b;
    int find(int x)
    {
        return a.count(x)?a[x]=find(a[x]):x;
    }
    int MLS(vector<int>& arr) {
        for(auto i:arr)
            a[i] = i+1;
        int ans = 0;
        for(auto i:arr){
            int y = find(i+1);
            ans = max(ans, y-i);
        }
        return ans;
    }
    
};
```
## HJ32 密码截取
```c++
#include<iostream>
using namespace std;
string helper(string &s, int l, int r)
{
    while(l >= 0 && r < s.size() && s[l] == s[r])
    {
        l--;
        r++;
    }
    return s.substr(l+1, r-l-1);
}
int main()
{
    string s;
    while(cin>>s)
    {
        string res = "";
        //判断回文字串
        for(int i = 0; i<s.size(); i++)
        {
            //先判断奇数的
            string tmp = helper(s, i, i);
            if(tmp.size() > res.size())
                res = tmp;
            //再判断偶数的
            tmp = helper(s, i, i+1);
            if(tmp.size() > res.size())
                res = tmp;
        }
        cout<<res.size()<<endl;;
    }
    return 0;
}
```
> 自定义排序
```c++
bool cmp(const pair<string, int>& a, const pair<string, int>& b) {
        return a.second < b.second;
}
```
> stoi长度越界问题:最多转换的32位字节-2 147 483 648~2 147 483 647
> unsigned int 范围是0～4294967295   

## HJ33 整数与ip地址间的转换
```c++
#include<iostream>
#include<sstream>
#include<string>
#include<vector>
using namespace std;
unsigned StrtoUInt32(const char *str)
{
  unsigned temp = 0;
  const char *ptr = str;  //ptr保存str字符串开头
 
  if (*str == '-' || *str == '+')  //如果第一个字符是正负号，
    {                      //则移到下一个字符
      str++;
    }
  while(*str != 0){
    if ((*str < '0') || (*str > '9'))  //如果当前字符不是数字
        {                       //则退出循环
          break;
        }
        temp = temp * 10 + (*str - '0'); //如果当前字符是数字则计算数值
        str++;      //移到下一个字符
    }  
  if (*ptr == '-')     //如果字符串是以“-”开头，则转换成其相反数
     {
        temp = -temp;
      }
  return temp;
}
int main()
{
    string s;
    while(cin>>s)
    {
        istringstream is(s);
        string seg;
        vector<string> v;
        while(getline(is, seg, '.'))
        {
            v.push_back(seg);
        }
        if(v.size() == 1)
        {
            //数字转ip
            string res;
            char *cstr = &v[0][0];  //知识点：string与char*之间的转换
            unsigned b = StrtoUInt32(cstr);
            for(int i = 3; i > 0; i--)
            {
                unsigned tmp = b >> (8*i);
                res += to_string(tmp);
                res += ".";
                tmp = tmp<<(8*i);
                b = b - tmp;
            }
            res += to_string(b);
            cout<<res<<endl;
        }
        if(v.size() == 4)
        {
            //ip转数字
            unsigned b = 0;
            for(int i = 0; i< 4;i++) b = (b << 8) +stoi(v[i]);
            cout<<b<<endl;
            continue;
        }
    }
    return 0;
}
```