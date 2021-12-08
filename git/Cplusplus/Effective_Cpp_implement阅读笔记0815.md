### shared_ptr
main class(Person)只内含一个指针成员，指向其实现类(PersonImpl)。这般设计常被称为pimpl idiom,这里的pimpl表示“pointer to implementation”
```c++      
#include<string>
#include<memory>  //包含tr1::shared_ptr
class PersonImpl;  //Person类的前置声明
class Date;   //Person接口用到的classes的前置声明
class Address; 
class Person{
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::tr1::shared_ptr<PersonImpl> pImpl;  //指针，指向实现物
};
```

# 条款32：确定你的public继承塑造出is-a关系
例如，Student is a Person.

# 条款33：避免遮掩继承而来的名称
名称遮掩规则(name-hiding rules)
> 继承Base class并加上重载函数，而又希望重新定义或覆写其中一部分，那么必须为那些原本会被遮掩的每个名称引入一个using声明式，否则某些希望继承的名称会被遮掩。
```c++
class Base{
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    ...
};
class Derived:private Base{
public:
    virtual void mf1()  //转交函数(forwarding function)
    {Base::mf1();}  //案子称为inline
};
...
Derived d;
int x;
d.mf1();  //调用Derived::mf1
d.mf1(x);  //错误！Base::mf1()被遮掩了
```

# 条款34：区分接口继承和实现继承
```c++
class Shape{
public:
    virtual void draw() const = 0;
    //pure virtual函数必须被任何“继承了它们”的具象class重新声明，并且它们再抽象class中通常没有定义。
    virtual void error(const std::string& msg);
    //impure virtual函数会提供一份实现代码，derived classes可能覆写(override)它。
    int objectID() const;
    ...
};
//Shape是个抽象class，它的pure virtual函数draw使它成为一个抽象class
//所以客户不能够创建Shape class的实体，只能创建其derived classes的实体
class Rectangle: public Shape{ ... };
class Ellipse: public Shape{ ... };
```
* 成员函数的接口总是会被继承。
* 声明一个pure virtual函数的目的是为了让derived classes只继承函数接口。
* 声明简朴的（非纯）impure virtual函数的目的，是让derived classes继承该函数的接口和缺省实现。“derived classes的设计者必须支持一个函数，如果不想自己写一个，可以使用base class 提供的缺省版本。”
* 声明non-virtual函数的目的是为了令derived classes继承函数的接口及一份强制性实现。

# 条款35:考虑virtual函数以外的其他选择
```c++
class GameCharacter{
public:
    //virtual函数的外覆器(wrapper)
    int healthValue() const{
        ...  //做一些事前工作
        int retVal = doHealthValue();  
        ...  //做一些事后工作
        return retVal;
    }
    ...
private:
    virtual int doHealthValue() const
    {
        ...
    }
};
```
令客户通过public non-virtual成员函数间接调用private virtual函数，称为non-virtual interface(NVI)手法，是所谓Template Method设计模式
外覆器确保得以在一个virtual函数被调用之前设定好适当场景，并在调用结束之后清理场景。
“事前工作”可以包括锁定互斥器(locking a mutex)、制造运转日志记录项(log entry)、验证class约束条件、验证函数先决条件等。“事后工作”可以包括互斥器解除锁定(unlocking a mutex)、验证函数的事后条件、再次验证class约束条件等。

Strategy设计模式的简单应用
```c++
class GameCharacter;  //前置声明(forward declaration)
//以下函数是计算健康指数的缺省算法
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter{
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const
    {return healthFunc(*this);}
    ···
private:
    HealthCalcFunc healthFunc;
};
```
改成使用tr1::function
```c++
class GameCharacter;  //前置声明(forward declaration)
//以下函数是计算健康指数的缺省算法
int defaultHealthCalc(const GameCharacter& gc);
class GameCharacter{
public:
    //HealthCalcFunc可以是任何“可调用物”(callable entity),可被调用并接受
    //任何兼容于GameCharacter植物，返回任何兼容于Int的东西
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    //tr1::function具现体（instantiation)的目标签名式(target signature)
    //接受一个reference指向const GameCharacter,并返回int
    //可调用物的参数可被隐式转换为const GameCharacter&,而其返回类型可被隐式转换为int
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc):healthFunc(hcf)
    {}
    int healthValue() const
    {return healthFunc(*this);}
    ···
private:
    HealthCalcFunc healthFunc;
};

short calcHealth(const GameCharacter&);  //健康计算函数
//注意其返回类型为non-int
struct HealthCalculator{
    int operator()(const GameCharactere&) const
    {...}  //为计算健康而设计的函数对象
};
class GameLevel{
public: 
    float health(const GameCharacter&) const;  //成员函数，用以计算健康
    ...
};
class EvilBadGuy:public GameCharacter{
    ...
};
class EyeCandyCharacter: public GameCharacter{
    ...   //另一个任务类型
};
EvilBadGuy ebg1(calcHealth); //人物1，使用某个函数计算健康指数
EyeCandyCharacter ecc1(HealthCalculator()); //人物2，使用莫格函数对象计算健康指数
GameLevel currentLevel;
...
EvilBadGuy ebg2(std::tr1::bind(&GameLevel::health, currentLevel, _1))
```

# 条款36：绝不重新定义继承而来的non-virtual函数

# 条款37：绝不重新定义继承而来的缺省参数值
