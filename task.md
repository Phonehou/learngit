# 1
输入包括两个正整数a,b(1 <= a, b <= 10^9),输入数据包括多组。
输出a+b的结果

输入

1 5
10 20

输出

6
30

```c++
//1.
#include<iostream>
using namespace std;
 
int main()
{
    int a=0,b=0;
    while(cin>>a&&cin>>b){
        cout<<a+b<<endl;
    }
}

//2.
#include<iostream>
#include<vector>
using namespace std;
 
int main()
{
    int n;
    int a,b;
    cin >> n;
    while(cin>> a >> b)
    {
        cout<<a+b<<endl;
    }
    return 0;
}

//3.
#include <iostream>
using namespace std;
int main()
{
    int a,b;
    while(cin>>a&&cin>>b&&a!=0&&b!=0)
        cout<<a+b<<endl;
}

//4.
#include <iostream>
using namespace std;
int main()
{
    int n;
    
    while(cin>>n)
    {
         int sum=0;
        int a;
        if(n==0)
        {break;}
        for(int i=0;i<n;i++)
        {
             
            cin>>a;
            
            sum+=a;
        }
        cout<<sum<<endl;
    }
}

//5.
#include <iostream>
using namespace std;
int main()
{
    int t;
    while(cin>>t)
    {
        int n;
        while(cin>>n)
        {
            int a;
            int sum=0;
            for(int i=0;i<n;i++)
            {
                cin>>a;
                sum+=a;
            }   
            cout<<sum<<endl;
        }
    }
}

//6.
#include <iostream>
using namespace std;
int main()
{
    int n;
    while(cin>>n){
    int sum=0;
    for(int i=0;i<n;i++)
    {
        int a;
        cin>>a;
        sum+=a;
    }
    cout<<sum<<endl;}
}

//7.
#include<iostream>
using namespace std;
int main(){
    int ans,cur=0;
    while(cin>>cur){
        ans+=cur;
        if(cin.get()=='\n'){
            cout<<ans<<endl;
            ans = 0;
            continue;
        }
    }
}
```
# 2
链接：https://ac.nowcoder.com/acm/contest/5657/B
来源：牛客网

输入描述:

输入第一行包括一个数据组数t(1 <= t <= 100)
接下来每行包括两个正整数a,b(1 <= a, b <= 10^9)

输出描述:

输出a+b的结果

示例1
输入

2
1 5
10 20

输出

6
30

# 3

输入包括两个正整数a,b(1 <= a, b <= 10^9),输入数据有多组, 如果输入为0 0则结束输入

输出描述:

输出a+b的结果

示例1
输入
复制

1 5
10 20
0 0

输出
复制

6
30

# 4
链接：https://ac.nowcoder.com/acm/contest/5657/D
来源：牛客网

输入数据包括多组。
每组数据一行,每行的第一个整数为整数的个数n(1 <= n <= 100), n为0的时候结束输入。
接下来n个正整数,即需要求和的每个正整数。

输出描述:

每组数据输出求和的结果

示例1
输入
复制

4 1 2 3 4
5 1 2 3 4 5
0

输出
复制

10
15

# 5
链接：https://ac.nowcoder.com/acm/contest/5657/E
来源：牛客网

输入描述:

输入的第一行包括一个正整数t(1 <= t <= 100), 表示数据组数。
接下来t行, 每行一组数据。
每行的第一个整数为整数的个数n(1 <= n <= 100)。
接下来n个正整数, 即需要求和的每个正整数。

输出描述:

每组数据输出求和的结果

示例1
输入
复制

2
4 1 2 3 4
5 1 2 3 4 5

输出
复制

10
15

# 6
链接：https://ac.nowcoder.com/acm/contest/5657/F
来源：牛客网

输入描述:

输入数据有多组, 每行表示一组输入数据。
每行的第一个整数为整数的个数n(1 <= n <= 100)。
接下来n个正整数, 即需要求和的每个正整数。

输出描述:

每组数据输出求和的结果

示例1
输入
复制

4 1 2 3 4
5 1 2 3 4 5

输出
复制

10
15

# 7

链接：https://ac.nowcoder.com/acm/contest/5657/G
来源：牛客网

输入描述:

输入数据有多组, 每行表示一组输入数据。

每行不定有n个整数，空格隔开。(1 <= n <= 100)。

输出描述:

每组数据输出求和的结果

示例1
输入
复制

1 2 3
4 5
0 0 0 0 0

输出
复制

6
9
0

## 2.求长度为n的串最小可以被消成长度为多少的串

## 3. n名练习生，各自有通过钢索的时间，可两人同时通过钢索，只有1根平衡棒，时间为单独用时更长的那个人，现在求所有人都通过一侧时最短耗时。

## 5. 有n个砝码，第i个砝码的重量为w[i](单位为克)，砝码的左盘可以放任意多个m克的物品（至少放一个），小A希望在右盘放最少的砝码使天平平衡，如果可以左到输出最少的砝码数，不能做到输出-1.


# string 1
输入描述:

输入有两行，第一行n

第二行是n个空格隔开的字符串

输出描述:

输出一行排序后的字符串，空格隔开，无结尾空格

# string 2

输入描述:

多个测试用例，每个测试用例一行。

每行通过空格隔开，有n个字符，n＜100

输出描述:

对于每组测试用例，输出一行排序过的字符串，每个字符串通过空格隔开


# string 3

输入描述:

多个测试用例，每个测试用例一行。
每行通过,隔开，有n个字符，n＜100

输出描述:

对于每组用例输出一行排序后的字符串，用','隔开，无结尾空格

```c++
// #1
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

int main()
{
    int n;
    while(cin>>n)
    {
        vector<string> v;
        string input;
        for(int i = 0; i<n; i++)
        {
            cin>>input;
            v.push_back(input);
        }
        sort(v.begin(), v.end());
        for(auto it = v.begin(); it!=v.end(); it++)
        {
            cout<<*it<<' ';
        }
    }
    return 0;
}

// #2
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
 
int main()
{
    vector<string> vec;
    string str;
    while(cin >> str){
        vec.push_back(str);
        if(cin.get() == '\n'){
            sort(vec.begin(), vec.end());
            for(int i = 0; i < vec.size() - 1; ++i){
                cout << vec[i] << " ";
            }
            cout << vec.back() << endl;
            vec.clear();
        }
    }
    return 0;
}

// #3
#include<iostream>
#include<algorithm>
#include<vector>
#include<string>
using namespace std;

int main()
{
    string str;
    while(cin >> str)
    {
        vector<string> vec;
        string temp;
        for(char c:str)
        {
            if(c == ',')
            {
                vec.push_back(temp);
                temp = "";
            }
            else temp += c;
        }
        vec.push_back(temp);
        sort(vec.begin(), vec.end());
        for(auto it = vec.begin(); it!=vec.end()-1; it++)
        {
            cout<<*it<<",";
        }
        cout<<vec.back()<<endl;
    }
    return 0;
}

```

