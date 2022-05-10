---
title:  "Effective C++"
excerpt: "Effective C++"

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2022-05-10
last_modified_at: 2022-05-10


published: false
---

* 2006년에 구매해서 정말 잘 보았던 책이고, 책 구매했떤 시기와 상황에 따른 여러가지 추억등이 담겨있지만, 이제 종이책은 정말 불필요하다는 생각이 들어 정리하려한다. 
* 한번더 훝어 보며 정리중 
	* 한참을 보다보니 지쳐서 찾아보니 아래 정리본이 많은 도움이 되었다.
	* http://ajwmain.iptime.org/programming/programming.html#book_summary.
* 기존에 밑줄 그어놓은곳을 보며 기억이 1도 없는것도 놀라웠고, 그 당시에 한번은 다 봤었구나란 생각도 들고.새록새록 하다. 책 내용 좋네. 지금시대에 살짝 맞지 않는 부분이 있어 아쉽다.

# intro
## 용어 정리부터
* 시그니처(signature)
    * 함수의 매개변수 리스트와 반환타입이 나옮, 함수으 ㅣ경우 시그니처가 그 함수의 타입
    * 공식적인 C++정의에서는 함수의 반환타입을 제외하지만 아래 내용은 반환타입도 포함하는것으로 생각하면됨
* 초기화(initialization)
    * 어떤 객체에 최초의 값을 부여하는 과정
* 기본생성자(default constructor)
    * 어떤 인자도 주어지지 않은채 호출될수 있는 생성자
    * 원래 매개변수가 없거나, 모든 매개변수가 기본값을 갖고있으면 기본생성자가 됨

```c++
class A{
public:
    A();//기본생성자
};
class B{
public:
    explicit B(int x=0, bool b=ture);//기본생성자
};
class C{
public:
    explicit C(int x);//기본 생성자 아님
};
//explicit 로 선언하면 암시적인 타입 변환을 수행하는데 쓰이지 않게 됨(타입변환이 명시적으로 되는 곳에서 쓰일순 있음)
```
* 복사생성자(copy constructor)
    * 어떤 객체를 초기화를 위해 그와 같은 타입객체로부터 초기화할 때 호출 되는 함수
* 복사 대입 연산자(copy assignment operator)
    * 같은 타입의 다른 객체에 어떤 객체의 값을 복사하는 용도로 쓰이는 함수

```c++

class Widget{
public:
    Widget();//기본생성자
    Widget(const Widget& rhs);//복사 생성자
    Widget& operator=(const Widget& rhs);//복사 대입 연산자
    ...
};

Widget w1;//기본 생성자 호출
Widget w2(w1);//복사 생성자 호출
w1 = w2;//복사대입 연산자 호출
```
* 인터페이스(interface)
    * 자바 및 닷넷계열의 언어는 Interface라는 것이 언어차원에서 있지만 C++에는 그런거 없음
    * 내용중 나오는 인터페이스는 함수의 시그니처 혹은 어떤 클래스의 접근가능 요소(public인터페이스, protected 인터페이스) 혹은 템플릿 타입 매개변수로 유효해야 하는 표현식을 가리킴


# Chapter 1 C++에 왔으면 C++의 법을 따릅시다
## 항목 1: C++를 언어들의 연합체로 바라보는 안목은 필수
* C++
    * 초창기 C++는 C언어에 객체지향 기능 몇가지가 결합된 형태
    * C++ 처음 이름조차 클래스를 쓰는  C(C with CLasses) 였음
* 아래 언어의 연합체를 C++로 생각하자
    * C
    * 객체 지향 개념의 C++
    * 템플릿 C++
    * STL

## 항목 2: #define을 쓰려거든 const, enum, inline을 떠올리자
* 가급적 선행 처리자보다 컴파일러를 더 가까이 하자

```c++
#define ASPECT_RATIO 1.653
```
* ASPECT_RATIO 는 컴파일러에 넘어가기전 전처리자가 해당 값을 지우고 숫자로 바꾸어 버린다.
    * 컴파일러가 쓰는 기호 테이블에 포함되지 않는다. 
    * 컴파일 시점에 문제가 발생할 경우 찾기가 힘들수도있는 문제점이 있다 아래처럼 매크로 대신 상수를 쓰는건 어떻까?

```c++
const double AspectRatio = 1.653;
```
* #define 을 상수로 교체할때 조심할것 2가지
    * 상수 포인터를 정의하는 경우
        * 포인터는 꼭 const로 선언하고, 포인터가 가리키는 대상까지 const 로 선언하는 것이 보통
        *  예를 들면 헤더파일 안에 char* 기반의 문자열 상수를 정의하면 아래처럼 할수 있지만, string 로 하는것이 더 좋다.

```c++
const char* const authorName = "Scott Meyers";
```

```c++
const std::string authorName("Scott Meyers");
```
* 클래스 멤버로 상수를 정의하는 경우
    * 어떤 상수의 유효범위를 클래스로 한정하려하면 그 상수를 멤버로 만들어야함, 그 상수의 사본 개수가 한개를 넘지 못하게하기 위해선 정적(static) 멤버로 만들어야한다.

```c++
class GamePlayer{
private:
    static const int NumTurns = 5;//상수선언
    int scores[NumTurns];//상수를 사용하는 부분
};
```
* 아래 매크로의 문제는 무엇일까?
    * 매크로 인자중 큰 것을 f에 넘겨 호출하는 것인데,

```c++
//이런 매크로를 작성할땐 인자마다 괄호를 해주는 센스는 잊지 말자.
#define CALL_WITH_MAX(a,b) f((a)>(b) ? (a) : (b))
```
* 아래 예시에서처럼 보듯 f가 호출되기전 의도치않는 동작이 있을수 있다.

```c++
int a = 5, b = 0;
 
CALL_WITH_MAX(++a, b);        // a가 두 번 증가합니다.
CALL_WITH_MAX(++a, b + 10);    // a가 한 번 증가합니다.
```
* 인라인 함수에 대한 템플릿은 매크로 함수의 효율은 그대로 유지함은 물론 정규 함수의 모든 동작 방식 및 타입 안전성까지 완벽히 취할 수 있다.

```c++
template <typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```
* 이것을 잊지 말자
    * 단순한 상수를 쓸 때는, #define보다 const 객체 혹은 enum을 우선 생각합시다.
    * 함수처럼 쓰이는 매크로를 만들려면, #define 매크로보다 인라인 함수를 우선 생각합시다.



## 항목 3: 낌새만 보이면 const를 들이대 보자!
* const  키워드가 * 표의 왼쪽에 있으면 포인터가 가리키는 대상이 상수인 반명 const 가 *표의 오른쪽에 있는 경우엔 포인터 자체가 상수이다
* const가 * 표의 양쪽에 다 있으면 포인터가 가리키는 대상 및 포인터가 전부 상수이다.

```c++
char greeting[] = "hello";
char* p = greeting;//비상수 포인터, 비상수 데이터
const char *p = greeting;//비상수 포인터, 상수 데이터
char* const p = greeting;//상수 포인터, 비상수 데이터
const char* consot p = greeting;//상수 포인터, 상수 데이터

```
* 아래는 스타일 차이지 의미적으로 차이점이 없다
    * 타입 앞에 const 붙이기
    * 타입의 뒤쪽이자 * 표 앞에 const 붙이기

```c++
void f1(const Widget *pw);//f1은 상수 Widget객체에 대한 포인터를 매개변수로 줘야함
void f2(Widget const *pw);//f2도 마찬가지
```

* STL 반복자(iterator)는 포인터를 본뜬것. 기본동작 원리가 T* 포인터와 흡사
    * 반복자가 가르키는 대상에 따라 변경을 원하는 경우와 원치 않는 경우를 구분해서 쓰자

```c++
std::vector<int> vec;
.
const std::vector<int>::iterator iter = vec.begin();//iter는 T* const 처럼 동작
*iter = 10;//대상 변경 가능
++iter;//에러! iter는 상수

std::vector<int>::const_iterator cIter = vec.begin();//Citer는 const T* 처럼 동작
*cIter = 10;//에러! *cIter는 상수임
++cIter;//cIter를 변경하는거라 이것은 문제없음
```
* 가장 강력한 const의 용도는 함수선언에 쓸 경우임
    * 함수 반환값, 각각의 매개변수,, 멤버함수 앞, 함수 전체에 대해 const성질을 붙일수 있음
* 상수 멤버 함수
    * 멤버 함수에 붙는 const 역할은 "해당 멤버 함수가 상수 각체에 대해 호출될 함수이다" 라는 것을 알려주는 것임
        * 클래스의 인터페이스를 이해하기 좋게 하기 위힘
        * 상수 객체를 사용할수 있게 하자
            * C++의 실행 성능을 높이는 핵심 기법중 하나가 객체 전달을 "상수 객체에 대한 참조자(refreence-to-const)로 진행하는것"

* const 가 있고 없고 차이만 있는 멤버 함수들은 오버로딩 가능

```c++
class TextBlock {
public:
    ...
    const char& operator[](std::size_t position) const {
        return text[position];
    }
    
    char& operator[](std::size_t position) {
        return text[position];
    }
private:
    std::string text;
};
```
* 아래처럼 사용 가능

```c++
TextBlock tb("hello");
std::count << tb[0];//비상수 멤버 호출

const TextBlock ctb("hello");
std::count << ctb[0];//상수 멤버 호출
```
* 어떤 멤버 함수가 상수 멤버(const)라는 것은 어떤 의미일까?
    * 비트수준 상수성(bitwise constness)
        * 객체를 구성하는 비트중 어떤것도 바꾸면 안된다는식의 C++ 에서 정의하고있는 상수성
    * 논리적 상수성(logical constness)
        * 상수 멤버 함수라고 해서 객체의 한 비트도 수정할수 없는 것이 아니라 일부는 바꿀수있되 사용자측에서는 알아채지 못하게 하자란 의미
        * 이런 경우가 종종 필요한 경우가 있다
        * 이것을 위해 mutable을 사용하자 
            * mutable는 징정적 데이터 멤버를 수정할 수 있도록 해준다. 
* 이것만은 잊지 말자
    * const 를 붙여 선언하면 컴파일러가 미리 에러를 잡아주는데 도움을 준다
    * 컴파일러 쪽에서 보면 비트 수준 상수성을 지켜야 하지만, 여러분은 논리적인 상수성을 사용해서 개발하자
    * 상수 멤버 및 비상수 멤버 함수가 기능적으로 똑같게 구현되어있을 경우 코드 중복을 피하는 것이 좋은데, 이때 비상수 버전이 상수 버전을 호출하다록 만들자



## 항목 4: 객체를 사용하기 전에 반드시 그 객체를 초기화하자
*  C++에서 객체 값 초기화는 의도치 않게 동작하곤한다.

```c++
int x; //어떤 경우는 0으로 초기화 되지만 , 보장하지 않는다. 
class Point {
    int x,y;
};
Point p;//p의 멤버데이터가 0으로 초기화하는것은 보장하지 않는다. 
```
* C++의 객체 초기화
    * 규칙이 무척 복잡하다. 그러니 모든 객체를 사용하기 전에 항상 초기화를 하자. 
        * 런타임에 비용이 소모될 수 있는 상황이면 초기화 보장이 없다. 

```c++

int x=0;//int 의 직접 초기화
const char* text "a c style string";//포인터의 직접 초기화
```
* 대입(assignment)을 초기화(initialization)와 헷갈리지 말자

```c++
class PhoneNum{...};
class ABEntry{
public:
    ABEntry(const std::string& name);
private:
    std::string theName;
    int numTime;
}
ABEntry::ABEntry(const std::string& name)
{
    theName = name;//초기화가 아님, 대입을 하고 있음
}
```
* C++ 규칙에 의하면 어떤 객체든 그 객체의 멤버는 생성자의 본문이 실행되기 전에 초기화 되어야 한다고 되어있음.
    * theName은 초기화가 아니라 대입을 하고 있다.
    * numTime은  기본제공 타입이기 때문에 대입전에 초기화 보장이 없음
    * 아래처럼 바꾸자
    * 아래처럼 하지 않으면 초기화 후 대입을 하는 것이기에 처음 초기화가 헛되버린다. 

```c++
ABEntry::ABEntry(const std::string& name)
:theName(name), numTime(0) //모두 초기화 되고 있다.
{
}
```
* C++에서 객체를 구성하는 데이터 초기화 순서는 아래와 같음
    * 기본 클래스는 파생 클래스 보다 먼저 초기화됨
    * 클래스 데이터 멤버는 선언한느 순서대로 초기화 됨
* C++ 에서 비지역 정적 객체의 초기화 순서는 개별 번역 단위에서 정해진다.
    * 정적 객체(static object)
        * 생성된 시점부터 프로그램이 끝날때까지 살아있는 객체
        * 스택 및, 힙 기반 객체는 애초부터 정적 객체가 될수 없음
        * 정적 객체
            * 전역 객체
            * 네임스페이스 유효범위에 정의된 객체
            * 클래스안에 static 로 선언된 객체
            * 함수안에서 static로 선언된 객체
            * 파일 유효범위에서 static로 정의된 객체
    * 번역 단위(translation unit)
        * 컴파일을 통해 하나의 목적파일을 만드는 바탕이 되는 소스코드를 일컫음
* 어떤 객체가 초기화 되기 전에 그 객체를 사용할일이 생기지 않도록 하려면
    * 멤버 가 아닌 기본 제공 타입 객체는 직접 초기화 하자
    * 객체의 모든 부분에 대한 초기화에서 멤버 초기화 리스트를 사용하자
    * 별개의 번역 단위에 정의된 비지역 정적객체에 영황을 끼치는 불활실한 초기화 순서를 염두에 두고 설계하자.
* 이것만은 잊지 말자
    * 기본제공 타입의 객체는 직접 손으로 초기화합니다. 경우에 따라서 저절로 되기도 하고 안 되기도 하기 때문입니다.
    * 생성자에서는, 데이터 멤버에 대한 대입문을 생성자 본문 내부에 넣는 방법으로 멤버를 초기화하지 말고 멤버 초기화 리스트를 즐겨 사용합시다. 그리고 초기화 리스트에 데이터 멤버를 나열할 때는 클래스에 각 데이터 멤버가 선언된 순서와 똑같이 나열합시다.
    * 여러 번역 단위에 있는 비지역 정적 객체들의 초기화 순서 문제는 피해서 설계해야 합니다. 비지역 정적 객체를 지역 정적 객체로 바꾸면 됩니다.



# Chapter 2 생성자, 소멸자 및 대입 연산자
## 항목 5: C++가 은근슬쩍 만들어 호출해 버리는 함수들에 촉각을 세우자
* C++ 에서는 클래스안에 직접 멤버 함수를 선언해 넣지 않았더라도 , 조건이 맞으면 컴파일러가 선언해 주도록 되어있는것들
    * public 멤버이며 inline 함수임
        * 생성자
        * 복사 생성자(copy constructor)
        * 복사 대입 연산자(copy assignment operator)
        * 소멸자(destructor)

```c++
//아래처럼만 해놓았으면
class Empty();

//아래와 마찬가지이다.
class Empty{
public:
    Empty(){...}//기본 생성자
    Empty(const EMpty& rhs){...}//복사 생성자
    ~Empty(){...}//소멸자
    Empty& operator=(const Empty& rhs){...}//복사 대입 연산자
}

//컴파일러가 필요하다고 판단되는 조건을 만족하는 코드는아래와 같다.
Empty e1;//기본 생성자, 소멸자
Empty e2(e1);//복사 생성자
e2 = e1;//복사 대입 연산자
```
* 복사 대입 연사자를 private 로 선선안 기본 클래스로부터 파생된 클래스으 ㅣ경우 이 클래스는 암시적 복사 대입 연산자를 가질수 없다. 

* 이것만은 잊지 말자
    * 컴파일러는 경우에 따라 클래스에 대해 기본 생성자, 복사 생성자, 복사 대입 연산자, 소멸자를 암시적으로 만들어 놓을 수 있다.

## 항목 6: 컴파일러가 만들어낸 함수가 필요 없으면 확실히 이들의 사용을 금해 버리자
* 예를 들어 아래와 같이 객체의 사본을 원치않아 객체를 복사하려는 코드는 컴파일이 되지 않도록 하려면 어떻게 하면 될까?

```c++
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1);//컴파일시 에러를 원해
h1 = h2;//컴파일시 에러를 원해
```
* 컴파일러가 생성하는 함수는 모두 public이 된다. 그러니 복사 생성자 및 복사 대입 연산자를 private 멤버로 선언하면 된다.
    * 하지만 private 멤버 함수는 frined 함수가 호출 할 수 있는 문제점이 있다. 
    * 정의를 하지 않으면 된다  아래를 보자. 
    * 즉 private 멤버로 선언후 정의만하고 구현하지 않는 방법

```c++
class HomeForSale{
public:
...
private:
    HomeForSale(const HomeForSale&);//선언만 하자.
    HomeForSale& operator=(const HomeForSale&);//선언만 하자.
}
```
* 하지만 아래 처럼 하면 더 좋지 않을까?

```c++
class Uncopyable{
protected:
    Uncopyable(){}//파생된 객체에 대해 생성과 소멸을 허용한다.
    ~Uncopyable(){}
private:
    Uncopyable(const Uncopyable&);//하지만 복사는 방지한다.
    Uncopyable& operator=(const Uncopyable&);
}
...

//복사 생성자도, 복사 대입 연산자도 이젠 선언되지 않는다. 
class HomeForSale : private Uncopyable{
...
}
```
* 이것만은  잊지 말자
    * 컴파일러에서 자동으로 제공하는 기능을 허용치 않으려면 대응되는 멤버 함수를 private로 선언후 구현은 하지않는채로 두자. Uncopyable과 비슷한 기본 클래스를 쓰는 것도 한 방법이다. 



## 항목 7: 다형성을 가진 기본 클래스에서는 소멸자를 반드시 가상 소멸자로 선언하자
* 예를 들어 아래처럼 기본 클래스로 무언가 만들어놓은뒤 파생시켜 사용하는 경우가 있는데,

```c++
class TimeKeeper{
public:
    TimeKeeper();
    ~TimeKeeper();
};

class AtomicClock: public TimeKeeper{...};
class WaterColck: public TimeKeeper{...};
class WristWatch: public TimeKeeper{...};
```
* 이 class를 사용하는 곳에서 시간 정보 객체에 접근하는 목적으로 객체에 대한 포인터를 얻는 용도의 팩토리 함수(factory function, 새로 생성된 파생 클래스 객체에 대한 기본 클래스의 포인터를반환하는 함수)를 만들어 주면 딱 좋다.
```c++
TimeKeeper* getTimeKeeper();//TimneKeeper에서 파생된 클래스를 통해 동적으로 할당된 객체의 포인터를 반환

//이럴경우 팩토리 삼수의 기존 규약을 따라가려면, delete가 필요하다 
TimeKeeper *ptk = getTimeKeeper();
...
delete ptk;
```
* 여기서 문제가 있다. 
    * getTimeKeeper 함수가 반환하는 포인터가 파생 클래스(AtomicColck) 객체에 대한 포인터라는 점
    * 이 포인터가 가리키는 객체가 삭제될때 기본 클래스 포인터(TimeKeeper* 포인터)를 통해 삭제된다는 점
    * 기본 클래스(TimeKeeper)에 들어있는 소멸자가 비가상 소멸자(non-virtual destructor)라는 점이다. 
* C++ 규정에 의해 기본 클래스 포인터를 통해 파생 클래스 객체가 삭제될때 그 기본 클래스에 비가상 소멸자가 들어있으면 그에 대한 동작은 미정의 사항이라 되어있다. 
    * 즉 delete ptk 시 AtomicClock가 소멸자도 실행되지 않는등 메모리 누수가 발생한다. 
* 이것을 해결하는 방법은 간단하다. 기본 클래스에 가상 소멸자 Virtual를 붙이자. 이러면 파생클래스 까지 모든 객체 전부가 소멸된다. 

```c++
class TimeKeeper{
public:
    TimeKeeper();
    virtual ~TimeKeeper();
};
```
* 가상 함수를 하나라도 가진 클래스는 가상 소멸자를 가져야하는게 대부분 맞음
* 가상 함수를 C++에서 구현하려면 클래스에 별도의 자료구조가 하나더 들어감
    * 프로그램이 실행중 어떤 가상 함수를 호출해야하는지 결정하는데 쓰이는 정보 
    * 바로 vtbl (가상함수 테이블)
        * vptr (가상 함수 테이블 포인터)로 가상함수가 하나라도 있으면 생성됨.
        * 가상함수 테이블이 만들어 지게 되면 포인터 크기 등으로 인해 객체의 크기가 커지게 됨.
  
* 소멸자를 전부 virtual로 선언하는 것은 virtual로 절대 선언하지 않는 것만큼이나 안좋음
* 가상 소멸자를 선언하는 것은 그 클래스에 가상 함수가 하나라도 들어있는 경우에만 한정해야함
* 경우에 따라 순수(pure) 가상 소멸자를 두면 편리함
    * 순수 가상함수는 해당 클래스를 추상 클래스(abstract class) (그 자체로는 인스턴스를 못만드는) 로 만듬
    * 아래의 예제를 보자

```c++
class AWOV {
public:
    Virtual ~ AVOW() = 0;// 순수 가상 소멸자 선언
};

AWOV::~AWOV(){}//순수 가상 소멸자 정의가 필요함.
```
* 소멸자가 동작하는 순서
    * 상속 계통의 가장 말단의 파생 클래스 소멸자가 가장 먼저 호출
    * 기본 클래스쪾으로 거쳐 올라가면서 각 기본 클래스의 소멸자가 하나씩 호출됨

* 이것만은 잊지 말자
    * 다형성을 가진 기본 클래스에는 반드시 가상 소멸자를 선언해야함 
        * 즉, 어떤 클래스가 가상 함수를 갖고 있으면, 이 클래스의 소멸자도 가상 소멸자야함
    * 기본 클래스로 설계되지 않았거나 다형성을 갖도록 설계되지 않는 클래스에서는 가상 소멸자를 선언하지 말아야함.



## 항목 8: 예외가 소멸자를 떠나지 못하도록 붙들어 놓자
* 아래와 같이 DB연결에 관련된 예시를 보면서 진행하겠다.
* DBConnectino 객체에 대해 사용자가 직접 close호출하지 않고 소멸자에서 한다고 하면

```c++
class DBConn{
public:
...
    static DBConnection create();
    ~DBConn(){
        db.close();
    }
private:
    DBConnection db;
};
...

{
    DBConn dbc(DBConnectin::create());//DBConnectino 객체를 사용하다가 블록 끝에서 소멸되면서 close 호출이 자동으로 이루어진다.
}
```
* 하지만 close 호출했을때 예외가 발생한다고 하면 어떻게 될까?
* 아래 2가지처럼 해결이 가능할 것이다. 

```c++
//-----------------------------------------------------------------------------
// close에서 예외가 발생하면 프로그램을
// 바로 끝냅니다. 대개 abort를 호출합니다.
DBConn::~DBConn(void)
{
    try { db.close(); }
    catch (...) {
        close 호출이 실패했다는 로그를 작성합니다;
        std::abort();
    }
}
 
// 에러가 발생한 후에 프로그램을 실행을
// 계속할 수 없는 상황이라면 꽤 괜찮은
// 선택입니다.
 
 
//-----------------------------------------------------------------------------
// close를 호출한 곳에서 일어난 예외를
// 삼켜 버립니다.
DBConn::~DBConn(void)
{
    try { db.close(); }
    catch (...) {
        close 호출이 실패했다는 로그를 작성합니다;
    }
}
 
// 대부분의 경우에서 예외 삼키기는
// 그리 좋은 발상이 아닙니다.
// 무엇이 잘못됐는지를 알려 주는 정보가
// 묻혀 버리기 때문입니다.
 
// 예외 삼키기를 선택한 것이
// 제대로 빛을 보려면,
// 발생한 예외를 그냥 무시한
// 뒤라도 프로그램이 신뢰성
// 있게 실행을 지속할 수 있어야
// 합니다.
```
* 둘다 문제가 있어 보인다. 
* 더 잘 설계를 해서 다양한 대처를 할 수 있도록 해보자.

```c+
class DBConn {
public:
    ...
    
    void close(void) {            // 사용자 호출을 배려(?) 해서
        db.close();                // 새로 만든 함수
        closed = true;
    }
    
    ~DBConn(void) {
        if (!closed)
            try {                // 사용자가 연결을 안 닫았으면
                db.close();        // 여기서 닫아 봅니다.
            }
            catch (...) {        // 연결을 닫다가 실패하면,
                close 호출이 실패했다는 로그를 작성합니다;
                ...                // 실패를 알린 후에
            }                    // 실행을 끝내거나 예외를 삼킵니다.
    }
    
private:
    DBConn db;
    bool closed;
};
 
// close 호출의 책임을 DBConn의 소멸자에서
// DBConn의 사용자로 떠넘기는 방법입니다.
 
// 사용자는 close가 발생한 예외를
// 처리할 필요가 있다면 DBConn::close
// 함수를 try 블록 내에서 호출하여
// 예외 처리를 할 수 있습니다.
 
// 반면 예외 처리를 할 필요가 없다면
// DBConn의 소멸자가 예외를 삼키거나,
// 프로그램을 종료시키도록 두면 됩니다.
```
* close에 대한 호출 책임을 소멸자에서 사용자에게 넘기는 것처럼 보일수 있지만, 쓰기 좋은 인터페이스의 방향성도 있지만 
* 예외는 소멸자가 아닌 다른 함수에서 비롯된 것으로 한다는 점이 중요하다. 
* 이것만은 잊지 말자!
    * 소멸자에서는 예외가 빠져나가면 안된다. 소멸자 안에서 호출된 함수가 예외를 던질 가능성이 있다면, 어떤 예외이던지 소멸자에서 모두 받아낸 후에 심켜 버리던지 버리던지 프로그램을 끝내야 한다
    * 어떤 클래스의 연산이 진행되다가 던진 예외에 대해 사용자가 반응해야 할 필요가 있다면, 해당 연산을 제공하는 함수는 반드시 보통의 함수(소멸자가 아닌!)여야 한다. 


## 항목 9: 객체 생성 및 소멸 과정 중에는 절대로 가상 함수를 호출하지 말자
* 가령 주식 거래를 본떠 만든 아래와 같은 클래스가 있다고 가정하자 . 거래의 중요성때문에 감사(audit) 기능이 들어있다. 

```c++
class Tranaction {
public:
    Tranaction();
    virtual void logTranaction() const = 0;//타입에 따라 달라지는 로그 기록을 만듬
    ...
};

Tranaction::Tranaction()
{
...
logTranaction();//마지막 동작으로 log를 기록
}

class BuyTransaction: public Tranaction{
public:
    virtual void logTranaction() const;
    ...
};
class SellTransaction: public Tranaction{
public:
    virtual void logTransaction() const;
};
```
* 이제 아래 코드가 실행될때 무슨 일이 일어 날까?

```c++
Buytransaction b;
```
* Tranaction 의 생성자 호출후 Buytransaction 의 생성자 호출된다. 그런데!
    * Tranaction 생성자 마지막의 logTranaction 가 있다 이건 Buytransaction 이 아닌 Tranaction 것으로 호출이 된다. 
    * 기본 클래스으 ㅣ생성자가 호출될 동안 가상 함수는 절대로 파생 클래스 쪽으로 내려가지 않는다. 
        * 대신 객체 자신이 기본 클래스 타입인 것처럼 동작한다. 
* 기본 클래스는 파생 클래스 생성자보다 앞서서 실행된다. 
    * 기본 클래스 생성자가 돌아가고 있을 시점에 파생 클래스 데이터 멤버는 아직 초기화 된 상태가 아니라는 점이 핵심이다. 
    * 파생 클래스 객체의 기본 클래스 부분이 생성되는 동안은, 그 객체의 타입은 바로 기본 클래스, 호출되는 가상 함수는 모두 기본 클래스의 것으로 결정(resolve)되어있을뿐만 아니라 런타임 타입 정보를 사용하는 언어 요소(dynamic_cast)를 사용한다해도 이 순간에는 모두 기본 클래스 타입으로 취급된다.
    * Buytransaction 객체의 기본 클래스 부분을 초기화 하기 위해 Tranaction 생성자가 실행되고있는 동안은 그 객체의 타입이 Tranaction 이라는 이야기.

* 객체가 소멸될때는 어떻까?
    * 똑같이 반대로 생각하면 되낟. 파생 클래스의 소멸자가 호출되고나면 파생 클래스만의 데이터 멤버는 정의되지 않는 상태이므로 기본 클래스의 소멸자에 진입할 당시의 객체는 바로 기본 클래스 객체가 된다.
* 해결 방법 여러가지중 하나를 소개하면 아래와 같다.
    * logTransaction 을 Transaction 클래스의 비가상 멤버 함수로 바꾸는 것이다.

```c++
class Transaction{
public:
    explicit class Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;//이제는 비가상 함수이다. 
}

Transaction::Transaction(const std::string& logInfo)
{
...
logTransaction(logInfo);//이제는 비가상 함수를 호출한다.
}

class BuyTransaction: public Transaction {
publioc:
    BuyTransaction(parameters) : Transaction(createLogString(parameters)) {...}//로그 정보를 기본 클래스 생성자로 넘기자
private:
    static std::string createLogString(parameters);
};
```
* 기본 클래스 부분이 생성될때 가상 함수를 호출한다해도 기본 클래스의 울타리를 넘어 내려갈수 없기에. 필요한 초기화 정보를 파생 클래스 쪽에서 기본 클래스 생성자로 올려주도록 만듦으로 해결하는 방법이다. 
    * createLogString 가 static로 되어있는것은 정적멤버로 해서 생성이 끝나지 않는 BuyTransaction 객체의 미초기화된 데이터 멤버를 건들 위험도가 없다. 

* 이것만은 잊지 말자!
    * 가상 함수라고해도 지금 실행중인 생성자나 소멸자에 해당되는 클래스의 파생 클래스쪽으론 내려가지 않으니 생성자/소멸자 안에서 가상 함수를 호출하지마. 


## 항목 10: 대입 연산자는 *this의 참조자를 반환하게 하자
* C++ 대입연산자는

```c++
int x,y,z;
x = y = z = 10;//대입이 사슬처럼 되고, 아래처럼 분석 된다. 

x = (y = (z = 15));//우측 연관 연산이다. 
```
* 이렇게 대입 연산자가 엮이려면 좌변 인자에 대한 참조자를 반환하도록 구현되어있을 것이다. (관례이다.)
    * 그러니 여러분도 관례를 지키면 좋을듯 하다. 

```c++

class Widget{
public:
    Widget& operator=(const Widget& rhs)//반환 타입은 현재 클래스에 대한 참조자
    {
    ..
    return *this;//좌변 객체(의 참조자)를 반환.
    }
    
    //단순 대입현 연산자 말고, 모든 형태의 대입 연산자에서 지키자.
    Widget& operator+=(const Widget& rhs)//+=, -=, *= 등에도 동일하게 규약적용.
    {
    ..
    return *this;
    }
    
    Widget& operator=(int rhs)//매개변수 타입이 일번적이지 않아도 관례는 지키가.
    {
    ..
    return *this;
    }
};

```

* 이것만은 잊지 말자
    * 대입 연산자는 *this 의 참조자를 반환하도록 만들자.


## 항목 11: operator=에서는 자기대입에 대한 처리가 빠지지 않도록 하자
* 자기대입(self assignment) 이란, 어떤 객체가 자기 자신에 대해 대입 연산자를 적용하는것을 뜻함

```c++

class Widget {...};
Widget w;
...
w = w;//자기대입
.
.
a[i] = a[j];//i와j가 같을수도있다. 자기 대입의 가능성을 가짐
*px = *py;//역시나 마찬가지

```
* 자기대입은 중복참조(aliasing)라고 불리는 여러 곳에서 하나의 객체를 참조하는 상태때문에 발생할수 있다.
* 자기대입은 안정하게 동작해야한다
    * 잘못하면 자원을 이용하기전에 스스로 리소스 해제를 해버릴수있다.
* 예를 하나 들어보면 아래와 같이 동적 할당된 비트맵을 가르키는 원시 포인터를 데이터 멤버로 갖는 클래스가 있다고 해보자

```c++
class Bitmap{...};
Class Widget {
...
private:
    Bitmap* pb;
};
.
.
.
//아래는 겉보기에 멀쩡해 보이는 operator= 의 구현 코드지만 자기 참조의 가능성이 있는 코드이다.
Widget&
Widget::operator=(const Widget& rhs)
{
    delete pb;//같은 객체라면 망한다. 
    pb = new Bitmap(*rhs.pb);
    returh *this;
}
```

* 이를 방지할 정통적인 방법은 일치성 검사를 통해 점검하는 것이다

```c++
Widget& Widget::operator=(const Widget& rhs)
{
    if(this == &rhs) returh *this;//객체가 같은지, 자기대입인지 확인후 그렇다면 아무것도 안하기.
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
* 예외에 안전하게 더 작성해 보자 
    * new Bitmap이 거슬린다. 

```c++
Widget& WIdget::operator=(const Widget& rhs)
{
    Bitmap* pOrg = pb;//원래의 pb를 어딘가에 기록해두고
    pb = new Bitmap(*rhs.pb);//작업후
    delete pOrg;//삭제하도록 하면 new Bitmap에서 예외가 발생해도 좀더 안전해진다. 
    return *this;
}
```
* 일치성 테스트를 매번 하는것도 효율적이지 않을 것이다. 
* 복사 후 맞바꾸기 기법을 알아보자

```c++
class Widget {
...
void swap(Widget& rhs);//*this의 데이터 및 rhs의 데이터를 맞바꾼다. 
};

Widget& WIdget::operator=(const Widget& rhs)
{
    Widget temp(rhs);//rhs의 데이터에 대해 사본을 만들고
    swap(temp);//*this 데이터를 그 사본과 맞바꾼다. 
    return *this;
}
```
* 이것만은 잊지 말자
    * operator=를 구현할때 어떤 객체가 그 자신에 대입되는 경우를 제대로 처리하도록 만들자, 원본 객체과 복사 대상 객체의 주소를 비교하거나, 문장의 순서를 적절히 조절하거나, 복사 후 맞바꾸기를 써도 된다.
    * 두개 이상의 객체에 대해 동작하는 함수가 있다면 이 함수에 넘겨지는 객체들이 사실 같은 객체인 경우에 정확하게 동작하는지 확인해 보자. 



## 항목 12: 객체의 모든 부분을 빠짐없이 복사하자
* 캡슐화하 잘된 객체 지향 시스템중 설계가 잘 된것을 보면 객체를 복사하는 함수가 딱 둘만 있는데
    * 복사 생성자와 복사 대입 연산자
    * 이를 통틀어 객체 복사 함수라 부른다. 
* 컴파일러가 생성한 복사 함수가 맘에 들지 않을때 작성하는데., 
    * 클래스의 데이터 멤버가 있으면 이를 전부 처리하도록 복사 함수를 작성해야 한다. 
* 객체의 복사 함수를 작성할때 두 가지를 꼭 확인하자
    * 해당 클래스의 데이터 멤버를 모두 복사하고
    * 이 클래스가 상속한 기본 클래스의 복사 함수도 꼬박꼬박 호출해 주도록 하자

* 이것만은 잊지 말자
    * 객체 복사 함수는 주어진 객체의 모든 데이터 멤버 및 모든 기본 클래스 부분을 빠트리지 말고 복사해야한다
    * 클래스의 복사 함수 두개를 구현할때 한쪽을 이용해서 다른쪽을 구현하려는 시도는 하지 말고 대신, 공통된 동작을 제3의 함수에다 분리해 놓고 양쪽에서 이것을 호출하게 만들어 해결해라.

# Chapter 3 자원 관리
## 항목 13: 자원 관리에는 객체가 그만!
* 가령 아래와 같이 투자를 관리하는 객체를 생성해서사용하는 경우가 있을때 멀쩡해 보이기만 객체 삭제에 실패한 경우가 많다.
    * ... 문에서 return 이 들어갈수도 있고, goto문이 나와 빠져나올수도있고, ...안에서 예외를 던질수도 있다.

```c++
void f()
{
    Investment *pInv = createInvestment();//팩토리 함수를 호출하고
    ...//사용하다가
    delete pInv;//객체를 해제한다.
}
```
* createInvestment 함수로 얻어낸 자원이 항상 해제되도록 만드는 방법은 자원을 객체에 넣고 그 자원 해제를 소멸자에게 맡도록 하며 그 소멸자는 실행 제어가 f를 떠날때 호출되도록 만드는 것이다. 
* 스마트 포인터(smart pointer)가 그 예이다.
* 자원관리에 객체를 사용하는 방법중 중요한 2가지 특징
    * 자원을 획득한 후에 자원 관리 객체에게 넘긴다.
        * 자원 획득 즉 초기화(Resource Acquisiton is Initialiaztion: RAII)
            * 자원 획득과 초기화를 한문장으로.
    * 자원 객체는 자신의 소멸자를 사용해서 자원이 확실히 해제되도록 한다.
        * 소멸자는 객체가 소멸될때 자동으로 호출되기에  유효범위를 벗어날때 자원 해제가 제대로 이루어질 수 있다. 
* auto_ptr
    * 자신이 소멸될 때 자신이 가리키고 있는 대상에 대해 자동으로 delete를 하기에 몇가지 특성이 있다.
        * auto_ptr 객체를 복사하면(복사 생성자 혹은 복사 대입 연산자) 원본 객체는 null로 만든다. 
* auto_ptr을 사용하지 못하는 상황이라면 참조 카운팅 방식 스마트 포인터(RCSP)가 좋다. 
    * 복사 방식이 좀더 자연스럽다. 
* auto_ptr 및 tr1::shared_ptr 은 소멸자 내부에 delete [ ]  가 아닌 delete를 호출한다.
    * 동적으로 할당한 배열에 대해선 auto_ptr등을 사용하지 않는것이 좋다. 
* 이것만은 잊지 말자
    * 자원 누출을 막기 위해 생성자 안에서 자원을 획득하고 소멸자에서 그것을 해제하는 RAII 객체를 사용하자
    * 일반적으로 널리 쓰이는 RAII 클래스는 tr1::shared_ptr 그리고 auto_ptr이다. 이 둘중 tr1::shared_ptr 이 복사 동작이 직관적이기에 대개 더 좋다 반면 auto_ptr은 복사되는 객체(원본객체)를 null로 만들어 버린다. 


## 항목 14: 자원 관리 클래스의 복사 동작에 대해 진지하게 고찰하자
* Mutex 타입의 뮤텍스 객체를 조작하는 클래스를 만든다고 생각해 보자

```c++
class Lock{
public:
    explicit Lock(Mutex *pm): mutexPtr(pm)
    {lock(mutexPtr);}//자원을 획득
    ~Lock(){unlock(mutexPtr);} //자원을 해제
private:
    Mutex *mutexPtr;
};
```
* 사용자는 Lock를 사용할때 RAII 방식에 맞추면 되는데

```c++
Mutex m;//
...
{//임계 영역을 정하기 위해 블록을 만들고
    Lock m1(&m);//뮤텍스에 잠금을 걸고
    ...//임계 영역에서 할 연산을 수행하고
}//블록의 끝이고 뮤텍스 잠금이 자동으로 해제가 된다.
...
//그런데 Lock 객체를 복사하면??
Lock m11(&m);//m에 잠금을 걸고
Lock m12(11);//m11을 m12로 복사하면 어떻게 될까?
```
* 이렇듯 RAII 객체가 복사될때 어떤 동작을 해야할까?
    * 복사를 금지한다.
        * 실제 RAII 가 복사되는게 말이 안되는 상황이 있다 복사 함수를 private 멤버로 만드는 식으로 복사를 막을 수 있따.

```c++
    class Lock: private UnCopyable {//복사를 금한다.
    ...
    };
    ```
    * 관리하고 있는 자원에 대해 참조 카운팅을 수행한다. 
        * tr1::shared_ptr에서 사용하듯 자원을 참조하는 객체의 개수에 대한 카운트를 증가하는 방식으로 만들어서. 아래처럼 사용할수 있다.
    * 관리하고 있는 자원을 진짜로 복사한다. 
        * 자원 관리 객체를 복사하면 그 객체가 둘러싸고 있는 자원까지 복사해야하기에 deep copy 를 수행해야 한다. 
    * 관리하고 있는 자원의 소유권을 옮긴다. 
        * auto_ptr이 하듯 특정 자원에 대해 참조하는 RAII객체는 딱 하나만 존재하도록 한다. 

* 이것만은 잊지 말자
    * RAII  객체의 복사는 그 객체가 관리하는 자원의 복사 문제를 안고 가기 때문에 그 자원을 어떻게 복사하느냐에 따라 RAII 객체의 복사 동작이 결정된다.
    * RAII 클래스에 구현하는 일반적인 복사동작은 복사를 금지하거나, 참조 카운팅을 해주는 선으로 마무리 하자.
    

## 항목 15: 자원 관리 클래스에서 관리되는 자원은 외부에서 접근할 수 있도록 하자
* 실무에선 대부분 실제 자원을 직접 컨트롤 해야할 일이 많을 것이다. 아래처럼 해보려고 할때 

```c++
std::tr1::shared_ptr<Investment> pInv(createInvestment());
//이때 어떤 Investment 객체를 사용한다는 가정을 해보면
int daysHeld(const Investment *pi);//투자금이 유입된 이후의 경과일

//아래처럼 호출하고 싶을텐데
int days = daysHeld(pInv);//에러! Investment* 타입이 아니어서 컴파일이 안될꺼다.
```
* RAII 클래스(지금은 tr1::shared_ptr)의 객체를 그 객체가 감싸고 있는 실제 자원으로 변환할 방법이 필요하다
    * 명시적 변환 (explicit conversion)
    * 암시적 변환 (implicit conversion)
* tr1::shared_ptr 과 auto_ptr은 명시적 변환을 수행하는 get 멤버함수를 제공한다 실제 포인터(의 사본)을 얻을수 있다.

```c++
int days = daysHeld(pInv.get());//이제 문제없다. 
```
* 제대로 만들어진 스마트 포인터 클래스라면 포인터 역참조 연산자(operator-> 및 operator*)도 오버로딩 하고 있다.
    * 자신이 관리하는 실제 포인터에 대해 암시적 변환을 쉽게 할수 있도록 한다.

```c++
//예시를 만들어 보자. 
std::auto_ptr<Investment> pi2(createInvestment());//auto_ptr로 하여금 자원 관리를 하도록 하고
bool ret = !((*pi2).isTaxFree());//operator*를 써서 자원에 직접 접근하도록 한다.
```
* RAII 클래스를 실제 자원으로 바꾸는 방법으로 명시적 변환을 제공할지(get 멤버 함수등) 아니면 암시적 변환을 허용할지는 환경에 따라 다르겠지.
* 이것만은 잊지 말자
    * 실제 자원을 직접 접근해야 하는 기존 API들도 많기에 RAII 클래스를 만들때엔 그 클래스가 관리하는 자원을 얻을 수있는 방법을 열어 주어야 한다.
    * 자원 접근은 명시적 변환, 암시적 변환을 통해 가능하다. 안전성을 따지면 명시적 변환이, 편의섬은 암시적 변환이 괜찮다.


## 항목 16: new 및 delete를 사용할 때는 형태를 반드시 맞추자
* 아래 코드가 맞을까?

```c++
std::string *strArry = new std::string[100];
..
delete strArry;
```
* 위의 코드는 99개는 정상적인 소멸 과정을 거치리 못할 가능성이 있음


* new 를 이용해 표현식을 꾸기게 될때의 2가지 내부 동작
    * 일단 메모리가 할당됨(operator new 라는 이름의 함수가 쓰임)
    * 할당된 메모리에 대한 한개 이상의 생성자가 호출됨
* delete 표현식을 쓸 경우 2가지 내부 동작
    * 우선 기존에 할당된 메모리에 대한 한 개 이상의 소멸자가 호출
    * 그후 그 메모리가 해제됨 (operator delete 라는 이름의 함수가 호출됨)
    * delete 연산자가 적용되는 객체는 소멸자가 호출되는 휫수가 됨
* 삭제되는 포인터는 객체 하나만 가리킬까? 객체의 배열을 가리킬까?
    * 단일 객체의 메모리 배치구조(layout)은 객체 배열에 대한 메모리 배치구조와 다름
        * 배열을 위해 만들어지는 힙 메모리에는 대게 배열 원소의 개수가 박혀서 들어감
        * 단일 객체용 힙 메모리는 이런 정보가 없음
        * 예를 들면 아래와 같음
            * 한개의 객체 : [object]
            * 객체의 배열 : [n][object][object][object]
* 즉 포인터에 대해 delete를 적용할때 delete 연산자로 하여금 배열 크기 정보를 넘겨줄지를 사용자가 정하는 방식임
    * 아래처럼 해야함

```c++
std::string * str1 = new stdLLstring;
std::string * str2 = new stdLLstring[100];
..
delete str1;//객체 한개를 삭제
delete [] str2;//객체의 배열을 삭제
//delete [] str1을 하면? 미정의 동작을 하겠지 delete str2를 해도 미정의 동작을 하고
// delete [] str1을 하면 배열 크기를 읽기 위해 몇 바이트를 읽어서 동작할테니..
```
* 배열 타입은 typedef 타입으로 만들지 말자. 아래와 같이 헷갈릴수 있다. 
    * vector < string> 같은 표준 라이브러리를 쓰면 쉽잖아.

```c++
typedef std::string addr[4];
std::string *pal = new addr;
delete pal;//무슨일이 생길지 모름
delete [] pal;//깔끔한 마무리
```
* 이것만은 잊지 말자
    * new 표현식에  [ ] 를 썼으면 대응되는 delete에도 쓰자 

## 항목 17: new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자
* 가령 아래와 같은 케이스가 있다고 할때

```c++
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);

processWidget(new Widget, priority());//이건 컴파일이 안되고 (tr1::shared_ptr의 생성자는 explicit 로 선언되어있어)
//아래처럼 하면 문제없이 컴파일은 된다. 하지만 문제가 있다. 
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```
* 컴파일러는 processWidget 호출 코드를 만들기전 매개변수로 넘겨지는 인자를 평가(evaluate)하는 순서를 진행하는데, 아래 3가지 연산을 위한 코드를 만들어야 한다. 
    * priority 를 호출하는 부분
    * new WIdget 를 실행하는 부분
    * tr1::shared_ptr 생성자를 호출하는 부분
* 문제는 C++에선 컴파일러 제작사 마다 위의 연산이 실행되는 순서가 다를수 있다. (자바나 C#은 고정되어 있다. )
    * new WIdget 를 실행후 priority를 호출하는중 예외가 발생하면 리소스 유실이 발생한다. 
* 위의 문제를 피하기 위해 Widget 를 생성해서 스마트 포인터에 저장하는 코드를 아래처럼 별도로 하나로 만들고 그 스마트 포인터를 넘기면 문제가 없다

```c++
std::tr1::shared_ptr<Widget> pw(new Widget);//new로 생성한 객체를 스마트 포인터에 넣는 코드를 하나의 독립문장으로 만들고
processWidget(pw, priority());//이젠 문제없다.
```

* 이것만은 잊지 말자
    * new 로 생성한 객체를 스마트 포인터로 넣는 코드는 별도의 한 문장으로 만들자. 이것이 안되어 있으면 예외가 발생할때 디버깅하기 힘든 자원 누출이 있을수 있다.



# Chapter 4 설계 및 선언
## 항목 18: 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자
* 제대로 쓰기엔 쉽고, 엉터리로 쓰기에 어려운 인터페이스를 개발하라면 사용자가 저지를만한 실수의 종류를 미리 생각해 보자

```c++
//날짜를 나타내는 어떤 클래스에 넣을 생성자의 예를 들어보자. 
class Date{
public:
    Date(int month, int day, int year);
    ...
};

//매개변수 전달 순서가 잘못될 여지가 있다.
Date d(30, 3, 1995);//3, 30, 1995를 넣어야 하는데 잘못넣은 경우

//존재하지 않는 값으로 넣는 경우
Date d(3, 40, 1995);//3, 30, 1995를 넣어야 하는데 잘못넣은 경우
```
* 잘못된 코드가 컴파일시점부터 알아내기 위한 좋은 것중 하나가 타입 시스템이다.
* 아래처럼 랩퍼(wrapper) 타입을 만들어 쓸수 있을 것이다.

```c++
struct Day{
    struct Month { 
        struct Year {
            explicit Day(int d) explicit Month (int m) explicit year(int y) : val(d) {} : val(m){} : val (y){}
            int val; int val; int val;
        };
    };
};

class Date{
public:
    Date(const Month& m, const Day& d, const Year& y);
    ...
};

Date d(3. 3, 1995);//error
Date d(Month(3), Day(30), Year(1955));//OK
```
* 예상되는 사용자 실수를 막는 다른 방법은 어떤 타입이 제약을 부여햐여 그타입을 통해 할수 있는 일을들 묶어 버리는 방법이 있다. 
    * const 붙이기가 있을수 있고 operator* 반환 타입을 const로 한정함으로 실수를 막을수 있다.
* 이러한 것들은 일관성 있는 인터페이스를 제공하기 위한 것이다. 
    * 한 예로 모은 STL 컨테이너는 size란 멤버함수를 제공ㅎ아고있고 원소의 개수를 알려주는 기능을 한다.
* 또 다른 예를 들어보면

```c++
Investment* createInvestment();//항목 13의 예시를 가져왔다. 
//사용자가 잘못 사용할 가능성을 없애려면 아래처럼 하면 어떻까?
//이렇게 하면 객체를 삭제하는것을 잊는것을 강제로 없애버릴수있다.
std::tr1::shared_ptr<Investment> createInvestment();

```
* 가령 createInvestment 로 얻은 것을 해제하기 위해 직접 포인터를 삭제하는 것이 아니라 getRidOfInvestment 라는 이름으로 여기에 넘겨 없애버리는 것은 어떻까?
    * 깔끔해 보이긴 하지만, 저것을 안쓰고 delete를 쓴다던가, std::tr1::shared_ptr로 반환하도록 구현해둔다면 잘못사용되는 길이 뵈어 버린다. 

* 이것만은 잊지 말자
    * 좋은 인터페이스는 제대로 쓰기엔 쉬우며 엉터리로 쓰기엔 어렵습니다. 인터페이스를 만들 때는 이 특성을 지닐 수 있도록 고민하고 또 고민합시다.
    * 인터페이스의 올바른 사용을 이끄는 방법으로는 인터페이스 사이의 일관성 잡아주기, 그리고 기본제공 타입과의 동작 호환성 유지하기가 있습니다.
    * 사용자의 실수를 방지하는 방법으로는 새로운 타입 만들기, 타입에 대한 연산을 제한하기, 객체의 값에 대해 제약 걸기, 자원 관리 작업을 사용자 책임으로 놓지 않기가 있습니다.
    * tr1::shared_prt은 사용자 정의 삭제자를 지원합니다. 이 특징 때문에 tr1::shared_ptr은 교차 DLL 문제를 막아 주며, 뮤텍스 등을 자동으로 잠금 해제하는 데(항목 14 참조) 쓸 수 있습니다.
        * 교차 DLL 문제(cross-DLL problem): 객체 생성 시에 어떤 동적 링크 라이브러리(dynamically linked library. DLL)의 new를 썼는데 그 객체를 해제할 때는 이전의 DLL과 다른 DLL에 있는 delete를 써서 발생하는 문제. 

## 항목 19: 클래스 설계는 타입 설계와 똑같이 취급하자
* 클래스 설계는 타입 설계입니다. 새로운 타입을 정의하기 전에, 다음의 모든 고려사항을 고민해 보자
* 새로 정의한 타입의 객체 생성 및 소멸은 어떻게 이루어져야 하는가?
    * 이 부분에 따라 클래스의 생성자 및 소멸자의 설계가 바뀝니다. 그뿐 아니라 메모리 할당 연산자 함수를 오버로딩 하는 경우에는 이들 연산자 함수의 설계에도 영향을 미칩니다.
* 객체 초기화는 객체 대입과 어떻게 달라야 하는가?
    * 생성자와 대입 연산자의 동작 및 둘 사이의 차이점을 결정짓는 요소입니다.
* 새로운 타입으로 만든 객체가 값에 의해 전달되는 경우에 어떤 의미를 줄 것인가?
    * 여기서 잊으면 안되는 중요한 사실은 '값에 의한 전달'을 구현하는 쪽은 바로 복사생성자라는 것입니다.
* 새로운 타입이 가질 수 있는 적법한 값에 대한 제약은 무엇으로 잡을 것인가?
    * 전부는 아니지만, 클래스의 데이터 멤버의 몇 가지 조합 값만은 반드시 유효해야 합니다. 이런 조합을 가리켜 클래스의 불변속성(invariant)이라고 하며, 클래스 차원에서 지켜주어야 하는 부분입니다. 이 불변속성에 따라 클래스 멤버 함수 안에서 해 주어야 할 에러 점검 루틴이 좌우되는데, 특히 생성자, 대입 연산자, 각종 "쓰기(setter)" 함수는 불변속성에 많이 좌우됩니다. 그뿐 아니라 불변속성은 함수가 발생시키는 예외에도 영향을 미치며, 예외 지정을 쓴다면 그 부분에도 영향을 줍니다.
* 기존의 클래스 상속 계통망(inheritance graph)에 맞출 것인가?
    * 이미 갖고 있는 클래스로부터 상속을 시킨다고 하면, 클래스의 설계는 이들 클래스에 의해 제약을 받게 됩니다. 특히 멤버 함수의 가상성 여부가 가장 큰 요인입니다.(항목 34 및 36 참조) 만약 직접 만든 클래스를 다른 클래스들이 상속할 수 있게 만들자고 결정했다면, 이에 따라 멤버 함수의 가상 함수 여부가 결정됩니다.(특히 소멸자)
* 어떤 종류의 타입 변환을 허용할 것인가?
    * 암시적 타입 변환과 명시적 타입 변환중 허용할 타입 변환을 정합니다.
* 어떤 연산자와 함수를 두어야 의미가 있을까?
    * 클래스 안에 선언할 멤버 함수가 여기서 결정됩니다. 멤버 함수로 적당한 함수와 그렇지 않은 함수가 있을 것입니다.(항목 23, 24, 46 참조)
* 표준 함수들 중 어떤 것을 허용하지 말 것인가?
    * private로 선언해야 하는 함수가 여기에 해당됩니다.(항목 6 참조)
* 새로운 타입의 멤버에 대한 접근권한을 어느 쪽에 줄 것인가?
    * 어떤 클래스 멤버를 public, protected, private 영역에 둘 것인가를 결정하는데 도움을 주게 될 질문입니다. 또한 프렌드로 만들어야 할 클래스 및 함수를 결정하는것은 물론이고 한 클래스를 다른 클래스에 중첩시켜도 되는가에 대한 결정을 내리는 데도 이 질문이 도움이 될 것입니다.
* '선언되지 않은 인터페이스'로 무엇을 둘 것인가?
    * 만든 타입이 제공할 보장이 어떤 종류일까에 대한 질문으로서, 보장할 수 있는 부분은 수행 성능 및 예외 안전성(항목 29 참조) 그리고 자원 사용(잠금 및 동적 메모리 등) 입니다. 이들에 대해 여러분이 보장하겠다고 결정한 결과는 클래스 구현에 있어서 제약으로 작용하게 됩니다.
* 새로 만드는 타입이 얼마나 일반적인가?
    * 정의하는 클래스가 타입 하나가 아니라 동일 계열의 타입군(family of types) 전체일지도 모릅니다. 만일 그렇다면 새로운 클래스가 아니라 새로운 클래스 템플릿을 정의해야 할 것입니다.
* 정말로 꼭 필요한 타입인가?
    * 기존의 클래스에 대해 기능 몇 개가 아쉬워서 파생 클래스를 새로 만들기 보다는, 차라리 간단하게 비멤버 함수라든지 템플릿을 몇 개 더 정의하는 편이 낫습니다.

* 이것만은 잊지 말자
    * 클래스 설계는 타입 설계이다. 새로운 타입을 정의하기전에 이번 항목에 나온 모든 고려사항을 빠짐없이 점검해보자



## 항목 20: '값에 의한 전달'보다는 '상수객체 참조자에 의한 전달' 방식을 택하는 편이 대개 낫다
* 기본적으로 C++은 함수로부터 객체를 전달받거나 함수에 객체를 전달할때 '값에 의한 전달'(pass-by-value)방식을 사용한다. 
* 별다른 처리를 하지않으면 기본적으로 함수 매개변수는 실제 인자의 '사본'을 통해 초기화 되며 그 함수가 반환한 값의 '사본'을 돌려받는다. 
* 값에 의한 전달은 고비용이다. 아래의 예시를 보자

```c++
class Person{
public:
    Person();
    virual ~Person();
    ...
private:
    std::string name;
    std::string address;
};

class Student: public Person{
public:
    Student();
    virual ~Student();
    ...
private:
    std::string schoolName;
    std::string schoolAddress;
};
...
//자 아래 코드를 호출하면 어떻게 될까?
bool validteStudent(student s);

Student plato;
bool platoIsOK = validteStudent(plato);//어떤 동작이 이루어질까?
```
* plato로부터 매개변수 s를 초기화 시키기 위해 Student의 복사 생성자가 호출
* s는 validateStudent가 복귀할때 소멸될것
* 즉 이 함수의 매개변수 전달 비용은 Student의 복사 생성자 호출 한번, Student의 소멸자 호출 한번이 이루어짐
* 또한 Student 객체에는 string 객체 2개가 있기에 Student 객체가 생성될때 마다 string객체도 같이 생성됨
* 또한 Student는 Person 으로 부터 파생됐기에 Person 객체도 먼저 생성되어야 하고 Person 객체가 생성될때 마다 또 그 안의 string 객체또한..
* 단지 Student객체 하나를 값으로 전달될때 마다 이 같은 일이 벌어짐
    * Student 복사 생성자 호출 한번, Person 복사 생성하 호출 한번, string 복사 생성자 호출 네번 발생
        * 소멸될때에도 마찬가지!
* 위의 문제 해결을 위해 상수객체에 대한 참조자(reference-to-const)로 전달하게 만들자.

```c++
bool validateStudent(const Student& s);//const 를 붙임으로써 넘겨진 Student가 변할것이란 걱정은 안해도 된다.!
```
* 또한 참조에 의한 전달은 매개변수를 넘기면서 복사손실 문제가 없어지는 장점도 있음
    * 복사 손실 문제
        * 파생 클래스의 객체가 기본 클래스 객체로 전달되는 경우, 값으로 전달되는 경우 기본클래스의 복사 생성자가 호출되고, 파생 클래스 객체로 동작 할수 있게 해주는 특징들이 싹뚝 잘리는 문제.

* 참조자는 보통 포인터를 써서 구현된다. 
    * 참조자를 전달하는것은 포인터를 전달하는 것과 일맥상통한다.
* 때에 따라 기본제공 타입(int등) 경우 참조자보다 값으로 넘기는 편이 더 효율적일 때가 많다
    * 다만 기본제공타입처럼, 타입 크기만 작다고 전부 값에 의한 전달이 좋은건 아니다. 비용이 비쌀수 있다. 
* 값에 의한 전달이 괜찮은 경우는
    * 기본제공 타입, STL 반복자, 함수 객체 타입 이렇게 3가지뿐이다.
* 이것만은 잊지 말자
    * 값에 의한 전달보다는 상수 객체 참조자에 의한 전달을 선호하자. 대체로 효율적이고 복사 손실 문제까지 막아준다
    * 이번 항목에서 다룬 법칙은 기본제공 타입 및 STL반복자, 함수 객체 타입에는 맞지 않다. 이들은 값에 의한 전달이 더 적절하다. 


## 항목 21: 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말자
* 새로운 객체를 반환해야 하는 함수를 작성하는 방법의 정도는
    * 바로 새로운 객체를 반환하게 만드는 것이다. 아래의 예시를 보자

```c++
//스택에 만드는 방법인데. 이는 온전한 객체에 대한 참조자를 반환하지 않는다. 
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational resutl(lhs.n*rhs.n, lhs.d*rhs.d);
    return result;
}

//힙에 만드는 방법인데, 누가 delet할꺼임?. 그리고 비용이 줄지도 않음.
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational *resutl = new Rational(lhs.n*rhs.n, lhs.d*rhs.d);
    return *result;
}


//정석적인 방법 - 올바른 동작에 지불되는 작은 비용만 필요하다. 
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```
* 이것만은 잊지 말자
    * 지역 스택 객체에 대한 포인터나 참조자를 반환하는 일, 혹은 힙에 할당된 객체에 대한 참조자를 반환하는 일, 또는 지역 정적 객체에 대한 포인터나 참조자를 반환하는 일은 그런 객체가 두 개 이상 필요해질 가능성이 있다면 절대로 하지 말자


## 항목 22: 데이터 멤버가 선언될 곳은 private 영역임을 명심하자
* 데이터 멤버가 public이면 안되는 이유를 알아보고 protected도 그닥인 이유를 알아보고 결국 private 멤버이어야 한다는 결론을 보려한다. 
* public 가 안되는 이유
    * 문법적 일관성
        * 어떤 경우엔 괄호를 썼다가 어떤 경우엔 아니었다가. 엉망일수있다.
    * 멤버 데이터에 대한 정교한 제어가 가능하다
        * 읽기 전용, 읽고 쓰기 , 쓰기 접근등을 직접 컨트롤 할 수 있다. 
    * 결국 캡슐화(encapsulation) 이다.
        * C++ 세상에서 public이란 캡슐화 되지 않았다란 뜻이다.라고 생각하자. 
* protected 데이터 멤버의 경우도 public랑 사정이 비슷하다. 
    * 캡슐화 문제도 마찬가지이다. protected 데이터 멤버를 수정하면 이것을 사용하는 파생클래스에 대한 수정도 어마무시할꺼다. 
* 이것만은 잊지 말자
    * 데이터 멤버는 private 멤버로 선언하자. 이를 통해 클래스 제작자는 문법적으로 일관성 있는 데이터 접근 통로를 제공 할 수 있고, 필요에 따라 세밀한 접근 제어도 가능하며, 클래스의 불변속성을 강화할 수 있을뿐 아니라 내부 구현의 융통성도 발휘 할 수 있다.
    * protected 는 public보다 더 많이 보호 받고 있는 것이 절대로 아니다. 

## 항목 23: 멤버 함수보다는 비멤버 비프렌드 함수와 더 가까워지자
* 웹브라우저를 나타내는 클래스가 있다고 생각하자.

```c++
class WEbBrowser{
public:
    void clearCache();
    void clearHistory();
    void removeCookies();
    //위의 모든 기능을 동작하는 함수를 준비할 수도 있을 것이다.
    void clearEverything();//clearCache, history, cookies
};

//이 기능을 비멤버 함수로 제공해도 될꺼다 각 기능을 순서대로 후출하면 되닌깐.
void clearBrowser(WebBrowser& wb)
{
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
//자, 어느쪽이 더 괜찮은 방법일까? 멤버 버전인 clearEverything? 비멤버 버전인 clearBrowser?
```
* 객체 지향 법칙관련해서 생각해보면 clearEverything 라고 생각할수 있지만, clearBrowser 보다 캡슐화도 떨어지고 컴파일 의존도도 더 떨어진다. 하나씩 살펴 보겠다. 
* 캡슐화 관점
    * 밖에서 볼수 있는것이 줄어들면 그것들을 바꿀때의 유연성이 커진다. 
    * 이미 있는 코드를 바꾸더라도 제한된 사용자들 밖에 영향을 주지 않는 융통성 을 확보할수 있다. 
    * 어떤 데이터에 접근하는 함수가 많으면 그 데이터의 캡슐화 정도가 낮다는 이야기이다. 
    * private 멤버가 아니면 이 데이터에 접근할수 있는 함수 개수를 어떻게 제한할수 없다고 했다. 
    * private 멤버면 여기에 접근할 수있는 함수의 개수는 예측이 쉽다. 
        * 그 클래스의 멤버 함수의 개수에 프렌드 함수의 개수만 더하면 되니.
            * 프렌드 멤버는 멤버 함수 및 프렌드 함수만 접근할수 있으닌깐.
    * 이런 상황에서 똑같은 기능을 제공하는데, 멤버 함수(그 클래스의 private 데이터 멤버뿐 아니라 private 멤버로 되어있는 다른 함수, 나열자 등을 모드 접근 할 수 있는)를 쓸 것이냐 아니면 비멤버 프렌드 함수(어느 것도 접근할 수 없는)를 쓸것이냐를 다시 생각해 보면 캡슐화가 높은 것은 후자일 것이다. 
        * 이런 이유로 clearBrowser(비멤버 프렌드 함수)가 clearEverything(멤버함수)보다 더 바람직한 이유가 된다. 
    * 다만 이것은 비멤버 비프렌드(non-friend) 함수에만 적용이 된다.
* 함수는 어떤 클래스의 비멤버가 되어야 한다 라는 주장이 그 함수는 다른 클래스의 멤버가 될 수 없다 라는 의미가 아니라는 것이다. 
    * C++에는 네임스페이스를 사용해서 좀더 좋게 만들수 있다.
        * clearBrowser를 비멤버함수로 두되 WebBrowserStuff와 같은 네임스페이스 안에 두는 식으로.

```c++
namespace WebBrowserStiff{
    class WebBroswer{...};
    void clearBrowser(WebBrowser& wb);
...
}
```
* 이 방식처럼 네임스페이스를 쓰는것은 클래스와 달리 여러개의 소스 파일에 나뉠수 있기 때문에 더 좋은 방식이다. 
* 표준 C++ 라이브러들이 이런식으루 구성되어있다.
    * std네임스페이스에 속한 모든 것은  <C++ standrdLibry> 헤더 같은 것에 모조리 들어가 한통으로 섞이지 않고 몇 개의 기능과 관련된 함수들은 여러개의 헤더등에 흩어져 있다. (vector, memory 등)
* 편의 함수 전체를 하나의 네임스페이스를 이용하면서 여러개의 파일에 나누어 놓으면 ㅍ녀의성 함수 집합의 확장도 쉬워진다. 
    * 해당 네임스페이스에 비멤버 비프렌드 함수를 우너하는 만큼 추가해 주기만 하면 그게 확장이다.
    * 예를 들면, WebBrowser를 쓰다가 다른 편의 함수를 만들어야 하면, 헤더 파일을 하나더 만든뒤 WebBrowserStuff 네임스페이스를 만들고 그 안에 관련 함수의 선언문만 끼워 넣으면 끝이 난다. 
        * 추가된 함수는 기존과 다른 편의함수처럼 바로 사용할수 있고 WebBrowserStuff의 구성요소로 바로 합쳐지는데 이러한 것은 클래스로는 제공이 불가능한 기능이다. 
* 이것만은 잊지 말자
    * 멤버 함수 보다는 비멤버 비프렌즈 함수를 자주 쓰도록 하자. 캡슐화 정도가 높아지고 패키징 유연성도 커지며, 기능 확장성도 늘어난다.



## 항목 24: 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자
* 정수형을 암시적으로 유리수로 변환할 수 있는 클래스를 생각해보자

```c++
class Rational {
public:
    Rational (int numerator = 0, int denominator = 1);//int에서 Rational 로 암시적 변환을 위해 생성자에 일부러 explicit붙이지 않음
    int numerator() const;//분자 및 분모에 대한 접근용 함수
    int denominator() const;//22항목을 보세요.
...
    const Rational operator*(const RAtional& rhs) const;
};

Rational oneEighth(1, 8);
Rational oneHalf(1, 2);

Rational result = oneHalf * onfEight;//OK
result = resutl * oneEighth; //OK

result = oneHalf * 2;//OK
result = 2 * onfHalf; //ERROR!!!

//위를 풀어서 쓰면 아래와 같다.
result = oneHalf.perator*(2);//OK
result = 2.operator*(oneHalf);//ERROR!!
//정수 2에는 클래스 같은것이 연관되어있지 않기에 operator* 멤버 함수도 있을리가 없음.
```
* 위의처럼 되는것은 암시적 타입 변환(implicit type conversion) 때문이다. 
    * 컴파일러는 위의 함수에 int를 넘겼으며 함수쪽에선 Rational을 요구하는 사실을 알고있으나, 이 int 를 Rational 클래스의 생성자에 주어 호출하면 Rational 로 변환하는것도 알고있음 

```c++
const Rational temp(2);//2로부터 Rational 객체를 생성
result = oneHalf * temp;//oneHalf.operator*(temp)와 같음.
```
* 생성자에 명시호출(explicit)로 선언되어있었으면 oneHalf * 2, 2 * oneHalf 둘다 안됐을 것이다.

* 이제 이것을 해결해 보자. 
    * operator* 를 비멤버 함수로 만들어 컴파일러에게 모든 인자에 대해 암시적 타입 변환을 수행하도록 하면 된다.


```c++
class Rational {...};//이제 oprator* 가 없다. 

//이제는 비멤버 함수다. 
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}
 
Rational oneFourth(1, 4);
Rational result;
 
result = oneFourth * 2;        // 이건 멤버함수여도 자동 형변환이 일어납니다.
result = 2 * oneFourth;        // 비멤버 함수일때만 자동 형변환이 일어납니다.
```
* 멤버 함수의 반대는 프렌드 함수가 아니라 비멤버 함수이다. 
    * 프렌드 함수는 피할수 있으면 피하자.

* 이것만은 잊지 말자
    * 어떤 함수에 들어가는 모든 매개변수(this 포인터가 가리키는 객체도 포함해서)에 대해 타입 변환을 해 줄 필요가 있다면, 그 함수는 비멤버이어야 합니다.



## 항목 25: 예외를 던지지 않는 swap에 대한 지원도 생각해 보자
* 표준에서 기본적으러ㅗ 제공하는 swap 의 구현코드를 봐 보자

```c++
//a의 값과 b의 값을 맞바꾸는 std::swap의 구현
namespae std{
    template<typename T>
    void swap(T& a, T&b)
    {
        T temp(a);
        a = b;
        b = temp;
    }
}
//한번 호출에 복사가 3번, 발생한다. 
```
* 복사하면 손해보는 타입에는 다른 타입의 실제 데이터를 가리키는 포인터가 주로 구성되어있는 타입일것이다.  아래의 예제를 봐보자. 

```c++
class WidgetImpl {
public:
    ...
private:
    int a, b, c;//복사비용이 많은 항목들이 많이 있다고 가정하자. 
    std::vector<double> v;
    ...
};

class Widget{
public:
    Widget(const Widget& rhs);
    WIdget& operator=(const WIdget& rhs)//Widget를 복사하기 위해 자신의 WiodgetImpl 객체를 복사한다. 
    {
        ...
        *pImpl = *(rhs.pImpl);
    }
private:
    WidgetImpl *pImpl;//Widget 의 실제 데이터를 가진 객체에 대한 포인터. 
};
```
* Widget 객체를  우리가 직접맞바꾼다면 pImpl 포인터만 살짝 바꾸면 되지만 swap로 하면 Widget 객체를 3번 복사하고 WidgetImpl객체 세개도 복사할 것이다. 
* swap를 수정해서 Widget객체를 맞바꿀때엔 일반적인 방법 대신 내부의 Impl 포인터만 바꾸라고 해보자. 간단한 슈더코드로 나타내면 아래와 같을 것이다.

```c++
namespace std{
    template<>
    void swap<Widget>(Widget& a, Widget& b)
    {
        swap(a.pImpl, b.pImpl);//Widget을 swap하기 위해 각자의 pImpl 포인터만 맞바꾸자. 
    }
}
```
* 여기까지만 알아본다. 좀 더 심화내용은 굳이 내가 알고 정리할 필요가 있을까 싶다.


* 이것만은 잊지 말자
    * std::swap이 여러분의 타입에 대해 느리게 동작할 여지가 있다면 swap 멤버 함수를 제공합시다. 이 멤버 swap은 예외를 던지지 않도록 만듭시다.
    * 멤버 swap을 제공했으면, 이 멤버를 호출하는 비멤버 swap(동일 네임스페이스)도 제공합니다. 클래스(템플릿이 아닌)에 대해서는, std::swap도 특수화해 둡시다.
    * 사용자 입장에서 swap을 호출할 때는, std::swap에 대한 using 선언을 넣어 준 후에 네임스페이스 한정 없이 swap을 호출합시다.
    * 사용자 정의 타입에 대한 std 템플릿을 완전 특수화하는 것은 가능합니다. 그러나 std에 어떤 것이라도 새로 '추가'하려고 들지는 마십시오.




# Chapter 5 구현
## 항목 26: 변수 정의는 늦출 수 있는 데까지 늦추는 근성을 발휘하자
* 생성자, 소멸자를 끌고 다니는 타입으로 변수를 정의하면 반드시 생성자, 소멸자가호출되는 비용이 발생한다. 
* 다음의 예시를 보며 알아보자

```c++
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    string encrypted;//여기를 보자.
    if(password.length() < MIN_SIZE){
        throw logic_error("too short");
    }
    ...
    return encrypted;
}
```
* 중간에 예외가 발생하면 encrypted 변수는 사용되지 않고 객체 생성과 소멸 비용만 지불된다. 아래처럼 하는게 좋다고 생각이 들 것이다.  

```c++
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    if(password.length() < MIN_SIZE){
        throw logic_error("too short");
    }
    string encrypted;//여기를 보자.
    ...
    return encrypted;
}
```
* encrypted 가 호출될때 기본 생성자가 호출도리 것이다. 객체를 생성하고 나서 값을 대입하는 것은 직접 초기화보다 효율이 좋지 않는다 항목 4를 다시 보자. 위의 예시는 아래와 마찬가지이다. 

```c++
...
std::string encrypted;//기본 생성자에 의해 만들어짐
encrypted = password; //encrypted 에 passwod를 대입
...
```
* 기본 생성자 호출을 건너뛰는 방식으로 다시 작성해 보자.

```c++
std::string encryptPassword(const std::string& password)
{
    using namespace std;
    if(password.length() < MIN_SIZE){
        throw logic_error("too short");
    }
    ...
    string encrypted(password);//변수를 정의함과 동시에 초기화한다. 이땐 복사 생성자가 쓰인다. 
    encrypt(encrypted);
    ...
    return encrypt;
}
```
* 루프문은 어떠할까?

```c++
//A방법 : 생성자 1번 + 소멸자 1번 + 대입 n번
Widget w;
for(int i=0; i<n; i++){
    w = i에 따라 달라지는 값.
    ...
}

//B방법 : 생성자 n번 + 소멸자 n번
for(int i=0; i<n; i++){
    Widget w(i에 따라 달라지는 값)
    ...
}

```

* 대입에 들어가는 비용이 생성자-소멸자 쌍보다 적게 나오는 경우에는 A 방법이 효율이 좋습니다. 그렇지 않은 경우에는 물론 B 방법이 효율이 좋겠죠.
*  하지만 A 방법을 쓰면 프로그램의 이해도와 유지보수성이 역으로 안 좋아질 수도 있습니다.
* 따라서
     1. 대입이 생성자-소멸자 쌍보다 비용이 덜 들고
     2. 전체 코드에서 수행 성능에 민감한 부분을 건드리는 중
* 이라고 생각하지 않는다면, 앞뒤 볼 것 없이 B 방법으로 가는 것이 좋습니다.

* 이것만은 잊지 말자
    * 변수 정의는 늦출 수 있을 띠ㅐ까지 늦추자. 깔끔해지고 효율도 좋다. 



## 항목 27: 캐스팅은 절약, 또 절약! 잊지 말자
* C++에서 캐스팅은 정말로 조심해서 써야한다. 
* 가볍게 정리해 보자. 똑같은 캐스팅인데 여러가지 방법이 있다.
    * C 스타일의 캐스트이다.
        * ( T ) 표현식 //표현식 부분을 T 타입으로 캐스팅한다.
    * 함수 방식 캐스트이다.
        * T (표현식) //표현식 부분을 T 타입으로 캐스팅 한다. 
* C++ 는 네가지로 이루어진 캐스트 연산자를 독자적으로 제공한다. 

```c++
const_cast<T> (표현식)
dynamic_cast<T>(표현식)
reinterpret_cast<T>(표현식)
static_cast<T>(표현식)
```
* const_cast
    * 객체의 상수성(constness)를 없애는 용도로 사용된다. 
* dynamic_cast
    * 안전한 다운캐스킹을 할때 사용된다. 
    * 주어진 객체가 어떤 클래스 상속 계통에 속한 특정 타입인지 아닌지 결정하는 작업에 쓰인다. 
    * 런타임 비용이 높은 캐스트 연산자다
*  reinterpret_cast
    *  포인터를 int로 바꾸는 등의 하부 수준 캐스팅을 위해 만들었다. 
    *  이식성이 없으니 하부 수준 코드 외에는 거의 사용이 없어야 한다. 
* static_cast
    * 암시적 변환(비상수 객체를 상수객체로 바꾸거나 int를 double로 바꾸등의 변환)을 강제로 진행할때 사용된다. 
    * 흔히 이루어지는 타입 변환을 거꾸로 수행하는 용도로도 쓰인다. 
    * 상수 객체를 비상수 객체로 캐스팅하는데에는 쓸수 없다. 

* 아래의 예시를 보세요. 가능한한 C++방식 캐스트를 쓰세요. 
    * 검색도 쉽고, 컴파일러로부터 걸릴수 있어요. 

```c++
class Widget{
public:
    explicit Widget(int size);
    ...
};

void doSomeWork(const Widget& w);
doSomeWork(Widget(15));//함수 방식 캐스트 문법으로 int로 부터 Widget를 생성한다.
doSomeWork(static_cast<Widget>(15))//C++방식 캐스트로 int로 부터 Widget를 생성한다.
```
* 캐스팅을 그냥 어떤 타입을 다른 타입으로 처리하라고 컴파일러에게 알려주는것만 한다고 생각하면 크나큰 오산이에요.
    * 타입 변환이 있으면 이로 말미암아 런타임에 실행되는 코드가 만들어지는 경우가 정말 적지 않아요.
    * 아래의 예시를 보며 설명할께요

```c++
//int 타입 x를 double 타입으로 캐스팅할때 코드가 만들어 진다. 대부분 컴퓨터 아키텍처에서 int 표현구조와 double 의 표현구조가 아예 다르기 때문이다. 
int x, y;
double d = static_cast<double>(x)/y;//x를 y로 나누는ㄴ데 부동소수점 나눗셈을 사용한다.


//또 다른 예시. 두 포인터의 값이 같지 안을때도 가끔 있을수 있다. 이런경우 포인터의 변위(offset)을 Derived* 포인터에 적용하여 실제의 Base* 포인터 값을 구하는 동직이 바로 런타임에 이루어진다. 
class Base {...};
class Derived:public Base{...};
Derived d;
Base *pb = &d;//Derived*에서 Base*로 암시적 변환
```
* 객체 하나(가령 Derived 타입의 객체)가 가질 수 잇는 주소가 오직 한개가 아니라 그 이상이 될수 있음을 알아야 한다.
    * (Base* 포인터로 가리킬때 주소, Derived* 포인터로 가리킬때 주소)
    * C나 java에선 없는 일인데 C++ 다중상속이 사용되면서 이런 현상이 항상 생긴다. 단일 상속인데도 발생하는 경우가 있다. 
* dynamic_cast 는 상당수의 구현환경에서 느리게 구현되어있따.
    * 내부적으로 strcmp() 를 쓰는 것으로 알려진 것이 있다.  클래스 이름을 비교하기 위해 여러번 비교가 호출되기도 한다. 

```c++
//---------------------------------------------------------------------
// dynamic_cast를 피하는 법
 
// dynamic_cast가 사용된 코드
class Window { ... };
 
class SpecialWindow: public Window {
public:
    void blink(void);
    ...
};
 
typedef
std::vector<std::tr1::shared_ptr<Window> > VPW;
 
VPW winPtrs;
 
...
 
// 그다지 바람직스럽지 않은 코드: dynamic_cast를 쓰고 있습니다.
for (VPW::iterator iter = winPtrs.begin();
        iter != winPtrs.end();
        ++iter) {
if (SpecialWindow *psw = dynamic_cast<SpecialWindow*>(iter->get()))
    psw->blink();
}
 
 
// dynamic_cast를 피하는 방법 1
typedef std::vector<std::tr1::shared_ptr<SpecialWindow> > VPSW;
...
// 더 괜찮은 코드: dynamic_cast가 없습니다.
for (VPSW::iterator iter = winPtrs.begin();
        iter != winPtrs.end();
        ++iter)
    (*iter)->blink();
    
// 이와 같은 식으로 구현하게 되면 Window에서
// 파생될 수 있는 모든 녀석들에 대한 포인터를
// 똑같은 컨테이너에 저장할 수는 없습니다.
// 즉 다른 타입의 포인터를 담으려면 타입 안전성을 갖춘
// 컨테이너가 여러 개가 필요할 것입니다.
 
 
// dynamic_cast를 피하는 방법 2
class Window {
public:
    virtual void blink(void) {}        // 기본 구현은 '아무 동작 안하기'입니다.
    ...
};
 
class SpecialWindow: public Window {
public:
    virtual void blink(void) { ... }    // 이 클래스에서는 blink 함수가
    ...                                    // 특정한 동작을 수행합니다.
};
 
typedef std::vector<std::tr1::shared_ptr<Window> > VPW;
VPW winPtrs;                        // 이 컨테이너는 Window에서
                                    // 파생된 모든 타입의 객체
...                                    // (에 대한 포인터)들을 담습니다.
 
for (VPW::iterator iter = winPtrs.begin();
        iter != winPtrs.end();
        ++iter)                        // 놓치지 말고 잘 보세요.
    (*iter)->blink();                // dynamic_cast가 없습니다.
```

* 이것만은 잊지 말자. 
    * 다른 방법이 가능하다면 캐스팅은 피하십시오. 특히 수행 성능에 민감한 코드에서 dynamic_cast는 볓 번이고 다시 생각하십시오. 설계 중에 캐스팅이 필요해졌다면, 캐스팅을 쓰지 않는 다른 방법을 시도해 보십시오.
    * 캐스팅이 어쩔 수 없이 필요하다면, 함수 안에 숨길 수 있도록 해보십시오. 이렇게 하면 최소한 사용자는 자신의 코드에 캐스팅을 넣지 않고 이 함수롤 호출할 수 있게 됩니다.
    * 구형 스타일의 캐스트를 쓰려거든 C++스타일의 캐스트를 선호하십시오. 발견하기도 쉽고, 설계자가 어떤 역할을 의도했는지가 더 자세하게 드러납니다.


## 항목 28: 내부에서 사용하는 객체에 대한 "핸들"을 반환하는 코드는 되도록 피하자

* 클래스 데이터 멤버는 아무리 숨겨봤자 그 멤버의 참조자를 반환하는 함수들의 최대 접근도에 따라 캡슐화 정도가 정해진다
* 어떤 객체에서 호출한 상수 멤버 함수의 참조자 반환 값의 실제 데이터가 그 객체의 바깥에 저장되어ㅣ 있다면 이 함수의 호출부에서 그 데이터의 수정이 가능하다
* 참조자, 포인터 및 반복자는 모두 핸들(handle, 다른 객체에 손을 댈 수 있게 하는 매개체)이고 어떤 객체의 내부요소에 대한 핸들을 반환하게 만들면 언제든 그 객체의 캡슐화를 무너뜨리는 위험을 내포하고있다. 
* 무효참조 핸들(핸들이 있지만 따라갔을때 실제 객체의 데이터가 없는)이 흔하게 발생할 수 있다. 아래 예시를 보자. 
* 

```c++
class Rectangle {
public:
    ...
    // 객체 내부의 핸들(참조자)을 반환합니다.
    const Point& upperLeft(void) const { return pData->ulhc; }
    const Point& lowerRight(void) const { return pData->lrhc; }
    ...
};
 
class GUIObject { ... };
 
// Rectangle 객체를 값으로 반환합니다.
const Rectangle boundingBox(const GUIObject& obj);
 
// pgo를 써서 임의의 GUIObject를 가리키도록 합니다.
GUIObject *pgo;
...
// pgo가 가리키는 GUIObject의
// 사각 테두리 영역으로부터 좌측 상단 꼭짓점의
// 포인터를 얻습니다.
const Point *pUpperLeft =
    &(boundingBox(*pgo).upperLeft());
    
// 바로 위의 문장이 끝날 때, pUpperLeft포인터가
// 가리키는 Point객체는, boundingBox함수의
// 반환값인 임시 Rectangle객체가 소멸되면서 함께
// 소멸되어 버립니다.
```

* 핸들을 반환하는 멤버 함수를 절대로 두지 말자는 이야기가 아니라 피하자라는 이야기 이다. 

* 이것만은 잊지 말자
    * 어떤 객체의 내부요소에 대한 핸들(참조자, 포인터, 반복자)을 반환하는 것은 되도록 피하세요. 캡슐화 정도를 높이고, 상수 멤버 함수가 객체의 상수성을 유지한 채로 동작할 수 있도록 하며, 무효참조 핸들이 생기는 경우를 최소화할 수 있습니다.





## 항목 29: 예외 안전성이 확보되는 그날 위해 싸우고 또 싸우자!
* 예외 안정선을 가진 함수라면 예외 발생시 이렇게 동작해야한다
    * 자원이 새도록 만들이 않는다.
    * 자료구조가 더렵혀지는것을 허용하지 않는다. 
* 예외 안정성을 갖춘 함수는 세 가지 보장중 하나를 제공한다.
    * 기본적인 보장
        * 함수 동작중 예외 발생하면, 실행중인 프로그램에 관련된 모든 것들을 유효한 상태로 유지하겠다란 보장. 
            * 어떤 객체나 자료구조도 더렵혀지지 않고 모든 객체의 상태는 내부적으로 일관성을 유지
    * 강력한 보장
        * 예외가 발생하면, 프로그램의 상태를 절대로 변경하지 않겠다라는 보장
            * 호출하는 것은 원자적인(atomic) 동작이라할수있다, 호출이 성공하면 마무리까지 완변하게 성공하고, 실패하면 호출이 없었던 것처럼 프로그램의 상태가 되될아가는 것.
    * 예외 불가 보장
        * 예외를 절대로 던지지 않겠다란 보장
            * 기본제공 타입(int, 포인터,등)에 대한 모든 연산은 예외를 던지지 않게 되어있다. 
* 할수 있으면 예외 불가 보장을 제공하면 좋지만, 현실적으로 대부분은 기본적인 보장과 강력한 보장중 고르게 될꺼다. 
* 복사-후-맞바꾸기'란 현재 객체의 복사본을 만들고 복사본을 수정한 뒤 수정에 성공한 경우 현재의 객체와 수정된 복사본을 바꿔치기 하는 것인데, 이게 꼭 실용적인 것만은 아니다.

* 이것만은 잊지 말자
    * 예외 안전성을 갖춘 함수는 실행 중 예외가 발생되더라도 자원을 누출시키지 않으며 자료구조를 더럽힌 채로 내버려 두지 않습니다. 이런 함수들이 제공할 수 있는 예외 안전성 보장은 기본적인 보장, 강력한 보장, 예외 금지 보장이 있습니다
    * 강력한 예외 안전성 보장은 '복사-후-맞바꾸기' 방법을 써서 구현할 수 있지만, 모든 함수에 대해 강력한 보장이 실용적인 것은 아닙니다.
    * 어떤 함수가 제공하는 예외 안전성 보장의 강도는, 그 함수가 내부적으로 호출하는 함수들이 제공하는 가장 약한 보장을 넘지 않습니다.

## 항목 30: 인라인 함수는 미주알고주알 따져서 이해해 두자
* 인라인 함수는, 함수 호출 비용이 면제되는 것 이외의 장점이 더있는데, 컴파일러가 함수 본문에 대해 문맥별 최적호를 걸기가 용이해짐
* 인라인 함수의 아이디어는 함수 호출문을 그 함수의 본문으로 바꿔치기 하는거라 목적 코그의 크가기 커짐, 부풀려진 코드는 성능의 걸림돌이 될 수도 있음. 
* 본문 길이가 굉장히 짧은 인라인 함수를 사용하면, 목적크기도 작아지며 명령어 캐시 적중률도 높아 질수 있음
* inline 은 컴파일러에 요청하는 것이지 명령이 아님
    * inline을 붙이지 않고 되는 경우도 있고 명시적으로 할 수도 있음
    * 암시적 방법부터 알아보자. 
        * 클래스 정의(선언) 안에 함수를 바로 정의해 넣으면 컴파일러는 그 함수를 인라인 함수 후보로 봄

```c++
class Person{
public:
    ...
    int age() const {return theAge;}//암시적 인라인 요청 : age는 클래스 정의 내부에서 정의되었다. 
    ...
privagte:
    int theAge;
};
```
* 인라인 함수를 선언하는 명시적 방법은 함수 정의 앞에 inline 키워드를 붙이는 것
* 인라인 함수는 대체적으로 헤더 파일에 들어있어야 하는게 맞음. 
    * 대부분의 빌드 환경에서 인라인을 컴파일 도중에 수행하기 때문 그 함수가 어떤 형태인지 알고 있어야 동작 가능하기에..
    * 템플릿도 대체적으로 헤더 파일에 들어 있어야 맞음
* inline은 컴파일러 입장에선 무시할수 있는 요청임
    * 인라인을 선언햇을지라도 복잡한 함수는 인라인으로 처리 안하고, 가상 함수 호출 같은 것은 절대로 인라인 해주지 않음
        * virtual 의 의미가 어떤 함수를 호출할지 결정하는 작업을 실행중에 한다는 뜻이니 당연한 말임
* 결국 인라인 되느냐 안되느냐의 여부는 빌드 환경에 달려있음
* 라이브러리를 설계하는 개발자라면 함수를 inlien으로 선언할 때 그 영향에 대해 많은 고민이 필요로함
    * 사용자의 눈에 보이는 인라인 함수에 대해 라이브러리 차원에서 바이너리 업그레이드를 제공 할 수 없기 때문
        * 어떤 라이브러리에 f 라는 인라인 함수가 있고 사용자가 f의 본문을 컴파일 해서 응용프로그램을 만들었으면, 나중에 f의 내부를 바꾸게 되면 소스를 전부 다시 컴파일 해야함.
* inline은 언제써야할지 전략은 아래와 같음
    * 우선 아무것도 인라인 하지 말기
    * 꼭 인라인해야하는 함수, 혹은 정말 단순한 함수에 한해서만 인라인 선언을 하세요
        * 디버깅이 엄청 힘들수도 있으니...
* 이것만은 잊지 말자
    * 함수 인라인은 작고, 자주 호출되는 함수에 대해서만 하는 것으로 묶어둡시다. 이렇게 하면 디버깅 및 라이브러리이 바이너리 업그레이드가 용이해지고, 자칫 생길 수 있는 코드 부풀림 현상이 최소화되며, 프로그램의 속력이 더 빨라질 수 있는 여지가 최고로 많아집니다.
    * 함수 템플릿이 대개 헤더 파일에 들어간다는 일반적인 부분만 생각해서 이들을 inline으로 선언하면 안 됩니다.



## 항목 31: 파일 사이의 컴파일 의존성을 최대로 줄이자
* C++ 의 클래스 정의는 클래스 인터페이스뿐아니라 구현 세부사항까지 상당히 많이 지정하고있기에 인터페이스와 구현을 깔끔하게 분리하는것은 어려운 일임.

* 아래의 예시로 설명을 하면, 

```c++
int main()
{
    int x;
    Person p(params);
}
```
* 컴파일러는 x를 만나면 int 하나를 담을 공간을 할당(대개 스택)하는데, p를 만나게 되면 Person 의 객체의 크기가 얼마인지 컴파일러는 알 길이 없음 
    * java같은 경우는 객체의 포인터만 담을 공간만 할당하면 되겠지만.
* 정리해 보면 
* 객체 참조자 및 포인터로 충분한 경우에는 객체를 직접 쓰지 않습니다.
    * 어떤 타입에 대한 참조자 및 포인터를 정의할 때는 그 타입의 선언부만 필요함. 반면 어떤 타입의 객체를 정의할때는 그 타입의 정의가 준비되어 있어야 함
* 할수 있으면 클래스 저으이 대신 클래스 선언에 최대한 의존하도록 만듭니다. 
    * 어떤 클래스를 사용하는 함수를 선언할땐, 그 클래스의 정의를 가져오지 않아도 됨,
* 선언부와 정의부에 대해 별도의 헤더 파일을 제공합니다. 

* 핸들 클래스[pimpl 관용구(pointer to implementation)] 예제 

```c++
// 핸들 클래스 예제[pimpl 관용구("pointer to implementation")]
//-----------------------------------------------------------------------------
// 구현을 맡은 클래스의 이름이 PersonImpl이라고 하면
// Person 클래스는 다음과 같이 정의할 수 있을 거에요.
// "Person.h" 파일
#include <string>    // 표준 라이브러리의 구성요소는
                    // 전방 선언을 하면 안 됩니다.
                    
#include <memory>    // tr1::shared_ptr을 위한 헤더입니다.
 
class PersonImpl;    // Person의 구현 클래스에 대한 전방 선언
 
class Date;            // Person 클래스 안에서 사용하는
class Address;        // 것들에 대한 전방 선언
 
class Person {
public:
    Person(const std::string& name, const Date& birthday,
            const Address& addr);
    std::string name(void) const;
    std::string birthDate(void) const;
    std::string address(void) const;
    ...
private:            // 구현 클래스 객체에 대한 포인터
    std::tr1::shared_ptr<PersonImpl> pImpl;
};
 
 
//-----------------------------------------------------------------------------
// Person 클래스의 멤버 함수 중 두 개를 다음과 같이 구현해 보겠습니다.
// "Person.cpp"파일
#include "Person.h"            // Person 클래스를 구현하고 있는 중이기 때문에
                            // 이 Person의 클래스 정의를 #include해야 합니다.
 
#include "PersonImpl.h"        // 이와 동시에 PersonImpl의 클래스 정의도
                            // #include해야 하는데, 이렇게 하지 않으면
                            // 멤버 함수를 호출할 수 없습니다. 잘 보시면
                            // PersonImpl의 멤버 함수는 Person의 멤버 함수와
                            // 일대일로 대응되고 있음을 알 수 있습니다.
                            // 인터페이스가 똑같습니다.
 
Person::Person(const std::string& name, const Date& birthday,
                const Address& addr)
: pImpl(new PersonImpl(name, birthday, addr))
{}
 
std::string Person::name(void) const
{
    return pImpl->name();
}
```
* 인터페이스 클래스 예제 코드 보기

```c++
//-----------------------------------------------------------------------------
// 인터페이스 클래스 예제
// Person 클래스의 인터페이스 클래스는 다음과 같이 만들어 볼 수 있겠습니다.
// "Person.h" 파일
#include <string>
#include <memory>
 
class Person {
public:
    virtual ~Person(void);
    
    virtual std::string name(void) const = 0;
    virtual std::string birthDate(void) const = 0;
    virtual std::string address(void) const = 0;
    ...
    static std::tr1::shared_ptr<Person>    // 주어진 매개변수로 초기화되는 Person
        create(const std::string& name,    // 객체를 새로 생성하고, 그것에 대한
                const Date& birthday,    // tr1::shared_ptr을 반환합니다.
                const Address& addr);    // (팩토리 함수 혹은 가상 생성자라 불립니다.)
    ...
};
 
// 이 클래스르 코드에 써먹으려면 Person에 대한
// 포인터 혹은 참조자로 프로그래밍하는 방법밖에 없습니다.
// 순수 가상 함수를 포함한 클래스를 인스턴스로 만들기는
// 불가능하니까요. 단 Person클래스의 파생 클래스는
// 인스턴스로 만들 수 있습니다.
 
 
//-----------------------------------------------------------------------------
// 사용자 쪽에서는 다음과 같이 사용하면 됩니다.
// 사용자의 cpp파일
std::string name;
Date dateOfBirth;
Address address;
...
// Person 인터페이스를 지원하는 객체 한 개를 생성합니다.
std::tr1::shared_ptr<Person> pp(Person::create(name, dateOfBirth, address));
 
...
 
std::cout << pp->name()                // Person 인터페이스를 통해
          << " was born on "        // 그 객체를 사용합니다.
          << pp->birthDate()
          << " and now lives at "
          << pp->address();
...
 
 
//-----------------------------------------------------------------------------
// RealPerson이라는 구체 클래스는 다음과 같이 구현될 수 있습니다.
// "RealPerson.h" 파일
#include "Person.h"
 
class RealPerson: public Person {
public:
    RealPerson(const std::string& name, const Date& birthday,
                const Address& addr)
    : theName(name), theBirthDate(birthday), theAddress(addr)
    {}
    
    virtual ~RealPerson(void) {}
    
    std::string name(void) const;        // 이들 멤버 함수에 대한 구현은 보이지
    std::string birthDate(void) const;    // 않지만, 어떻게 되어 있을지는
    std::string address(void) const;    // 쉽게 예상할 수 있을 거예요.
    
private:
    std::string theName;
    Date theBirthDate;
    Address theAddress;
};
 
 
//-----------------------------------------------------------------------------
// 이제 남은것은 Person::create 함수인데, 정말 간단하게 만들 수 있습니다.
// "Person.cpp" 파일
#include "Person.h"
#include "RealPerson.h"
 
...
 
std::tr1::shared_ptr<Person> Person::create(const std::string& name,
                                            const Date& birthday,
                                            const Address& addr)
{
    return std::tr1::shared_ptr<Person>(new RealPerson(name, birthday,
                                        addr));
}
 
...

```

* 이것만은 잊지 말자
    * 컴파일 의존성을 최소하하는 작업의 배경이 되는 가장 기본적인 아이디어는 정의 대신에 선언에 의존하게 만드는것임 이 아이디어에  기반한 두 가지 접근 방법은 핸들 클래스와 인터페이스 클래스임
    * 라이브러리 헤더는 그 자체로 모든 것을 갖추어야 하며 선언부만 갖고있는 형태여야 함 이 규칙은 템플릿을 쓰이거나 쓰이지 않거나 동일하게 적용합시다. 


# Chapter 6 상속, 그리고 객체 지향 설계
## 항목 32: public 상속 모형은 반드시 "is-a(...는 ...의 일종이다)"를 따르도록 만들자
* 클래스 D(Derived)를 클래스 B(Base)로 부터 public 상속을 통해 파생시켰다면 
    * D타입으로 만들어진 모든 객체는 또한 B타입의 객체이지만그 반대는 되지 않난다라고 컴파일러에게 말한 것이다. 
* 클래스들 사이에 맺을 수 있는 관계는 아래와 같이 3가지가 있다. 클래스 사이에 맺을 수 있는 관계들을 명확히 구분하고 잘 표현하자. 
    * is-a 관계
    * has-a(..는 ..를 가짐)
    * is-implemented-in-terms-of(..는 .. 를 써서 구현됨)
* 이것만은 잊지 말자
    * public 상속의 의미는 "is-a(...는 ...의 일종)"입니다. 기본 클래스에 적용되는 모든 것들이 파생 클래스에 그대로 적용되어야 합니다. 왜냐하면 모든 파생 클래스 객체는 기본 클래스 객체의 일종이기 때문입니다.


## 항목 33: 상속된 이름을 숨기는 일은 피하자
* 컴파일러가 someFunc 의 유효범위 안에서 x라는 이름을 만나면 자신이 처리하고 있는 유효범위, 즉 지역 유효범위(local scope)를 뒤져 같은 이름을 가진 것이 있는지 알아본뒤, 이 외의 유효범위에 대해서는 더 이상 탐색하지 않는다. 
* 값을 읽어 x에 넣는 문장에서 참조하는것은 지역변수 x이다 
    * 안쪽 유효범위에 있는 이름이 바깥쪽 유효범위에 있는 이름을 가리기 때문이다.

```c++
int x;//전역변수

void someFunc()
{
    double x;//지역변수
    std::cin >> x;
}
```
* 상속에 대해서 생각해봐도 마찬가지이다. 기본 클래스에 선언된 것은 파생 클래스가 물려받으니..
    * 파생 클래스의 유효범위가 기본 클래스의 유효범위안에 중첩되어 있기 때문이다. 
* 상속된 이름 가리기를 무시하고 동작하고 싶으면 아래처럼 하면 된다. 

```c++
//-----------------------------------------------------------------------------
// using 선언을 이용하여 가려진 이름을 다시 볼 수 있게 하기.
class Base {
private:
    int x;
    
public:
    virtual void mf1(void) = 0;
    virtual void mf1(int);
    virtual void mf2(void);
    void mf3(void);
    void mf3(double);
    ...
};
 
class Derived: public Base {
public:
    using Base::mf1;    // Base에 있는 것들 중 mf1과 mf3을 이름으로 가린 것들을
    using Base::mf3;    // Derived의 유효범위에서 볼 수 있도록 (또 public 멤버로)
                        // 만듭니다.
    virtual void mf1(void);
    void mf3(void);
    void mf4(void);
    ...
};
 
 
//-----------------------------------------------------------------------------
// 전달 함수를 이용하여 가려진 이름을 다시 볼 수 있게 하기.
class Base {
public:
    virtual void mf1(void) = 0;
    virtual void mf1(int);
    ...                    // 예전과 동일합니다.
};
 
class Derived: public Base {
public:
    virtual void mf1(void) {    // 전달 함수입니다. 암시적으로
        Base::mf1();            // 인라인 함수가 됩니다.
    }
};
 
...
Derived d;
int x;
 
d.mf1();                        // 좋습니다. Derived::mf1(매개변수 없는 버전)을
                                // 호출합니다.
d.mf1(x);                        // 에러입니다. Base::mf1()은 가려져 있습니다.
                                // 매개변수가 있는 mf1()에 대해서도 전달 함수를 만들면
                                // d.mf1(x)도 호출 가능합니다.
 
```

* 이것만은 잊지 말자
    * 파생 클래스의 이름은 기본 클래스의 이름을 가립니다. public 상속에서는 이런 이름 가림 현상은 바람직하지 않습니다.
    * 가려진 이름을 다시 볼 수 있게 하는 방법으로, using 선언 혹은 전달 함수를 쓸 수 있습니다.




## 항목 34: 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하자
* public 상속이라는 개념은 자자세세히 보면  2가지로 나뉜다.
    * 함수 인터페이스의 상속
    * 함수 구현의 상속이다. 

* 기하학적 도형을 나타내는 클래스 계통구조를 예로 들어보자

```c++
class Shape{
public:
    virtual void draw() const = 0;
    virtual void error(const std::string& msg);
    int objID() const;
    ...
};
class Rectangle: public Shape{...};
class Ellipse: public Shape{...};
```
* Shape 는 추상 클래스로 Shape 클래스의 인스턴스를 만드려고 하면 안되고, 이 클래스의 파생클래스만 인스턴스화가 가능하다. 
    * 하지만 이 Shape 가 public 파생된 클래스에 미치는 영향은 엄청나다. 
* 멤버 함수 인터페이스는 항상 상속되게 되어 있끼 때문이다. 
    * public 상속은 is-a  이므로 Shape에 동작하는 것은 파생클래스에서도 동작해야 한다. 
* draw는 순수 가상 함수, error 는 단순 가상 함수, ojbID는 비가상 함수로 선언되어있는데
    * 선언이 다르다라는것은 어떤 것을 의미할까?
* 우선 순수 가상 함수인 draw부터 살펴본다. 
    * virtual void draw() const = 0;
    * 순수 가상 함수의 두드러진 특징 2 가지
        * 어떤 순수 가상 함수를 물려받은 구체 클래스가 해당 순수 가상 함수를 다시 선언해야 한다
        * 순수 가상 함수는 전형적으로 추상 클래스 안에서 정의를 갖지 않는다. 
        * 즉, 순수 가상 함수를 선언하는 목적은 파생 클래스에게 함수의 인터페이스만을 물려주려는 것이다. 
* 두번째로, 단순 가상 함수를 살펴보자. 
    * virtual void error(const std::string& msg);
    * 순수 가상 함수와 다른 면이 있는데 
        * 파생 클래스로 하여금 함수의 인터페이스를 상혹한다는 것은 같지만 
            * 파생 클래스 쪽에서 오버라이드 할 수 있는 함수 구현부도 제공해야한다는 점이 다르다. 
            * 즉 단순 가상 함수를 선언하는 목적은 파생 클래스로 하여금 함수의 인터페이스뿐만 아니라 그 함수의 기본 구현도 물려 받게 하자는 것이다. 
    * error 함수는 여러분이 지원해야하지만 새로 만들 생각이 없으면 기본 버전을 그냥 쓰라는 의미이다. 
* 세번째로 비가상 함수이다
    * int objID() const;
    * 멤버 함수가 비가상으로 되어있다는 것은 파생 클래스에서 다른 행동이 일어날 것으로 가정하지 않았다는 뜻이다.
        * 클래스의 파생에 상관없이 변하지 않는 동작을 지정하는데 쓰인다. 
    * 비가상 함수를 선언하는 목적은 파생 클래스가 함수 인터페이스와 더불으 그 함수의 필수적인 구현을 물려받게 하는 것이다.
* 이렇게 순수 가상함수, 단순 가상 함수, 비가상함수를 선언해서 파생 클래스를 물려받게 함으로 정밀하게 모든것을 지정할 수가 있다. 
* 클래스 설계에서 가장 흔히 발견되는 실수 두 가지를 알아보자
    * 1. 모든 멤버 함수를 비가상 함수로 선언하는 것
        * 이렇게 하면 파생 클래스는 기본 클래스의 동작을 특별하게 만들 만한 여지를 없게 만든다. 특히 비가상 소멸자가 문제가 될 수 있다(항목 7)
    * 2. 모든 멤버 함수를 가상 함수로 선언하는 것
        * 분명 파생 클래스에서 재정의가 안도어야 할 부분이 있을 것이다. 

* 이것만은 잊지 말자
    * 인터페이스 상속은 구현 상속과 다릅니다. public 상속에서, 파생 클래스는 항상 기본 클래스의 인터페이스를 모두 물려받습니다.
    * 순수 가상 함수는 인터페이스 상속만을 허용합니다.
    * 단순(비순수) 가상 함수는 인터페이스 상속과 더불어 기본 구현의 상속도 가능하도록 지정합니다.
    * 비가상 함수는 인터페이스 상속과 더불어 필수 구현의 상속도 가하도록 지정합니다.


## 항목 35: 가상 함수 대신 쓸 것들도 생각해 두는 자세를 시시때때로 길러 두자
* 게임 캐릭터를 클래스로 설계한다고 생각해 보자
    * 캐릭터 채력이 얼마나 남았는지를 알기 위해 healthValue라는 것을 두었따고 생각해보자
        * healthValue 함수가 순수 가상 함수로 선언되지 않는 것으로 보아 체력치를 계산하는 기본 알고리즘이 제공된다는 사실을 알 수 있다.(항목 34)

```c++
class GameCharacter{
public:
    virtual int healthValue() const;//캐릭터의 체력치를 반환. 파생 클래스에서 이 함수를 재정의 할 수 있음.
    ...
};
```
* 비가상 인터페이스 관용구(NVI 관용구)를 사용합니다: 공개되지 않은 가상 함수를 비가상 public 멤버 함수로 감싸서 호출하는, 템플릿 메서드 패턴의 한 형태입니다.

```c++
// ※ 비가상 함수 인터페이스(non-virtual interface: NVI) 관용구를 통한 템플릿 메서드 패턴
 
// public 비가상 멤버 함수를 통해
// private 가상함수를 간접적으로 호출하게
// 만드는 방법
class GameCharacter {
public:
    int healthValue(void) const                // 파생 클래스는 이제 이 함수를 재정의할 수
    {                                        // 없습니다.
 
        ...                                    // 사전 동작을 수행합니다.
        
        int retVal = doHealthValue();        // 실제 동작을 수행합니다.
        
        ...                                    // 사후 동작을 수행합니다.
        
        return retVal;
    }
    ...
private:
    virtual int doHealthValue(void) const    // 파생 클래스는 이 함수를 재정의할 수
    {                                        // 있습니다.
    
        ...                                    // 캐릭터 체력치 계산을 위한
                                            // 기본 알고리즘 구현
    }
};
 
// 가상 함수가 엄격하게 private일 필요는 없으나,
// protected나 public으로 선언된다면
// NVI 관용구를 쓰는 의미가 없어집니다.
```

* 가상 함수를 함수 포인터 데이터 멤버로 대체합니다: 군더더기 없이 전략 패턴의 핵심만을 보여주는 형태입니다.

```c++
// ※ 함수 포인터로 구현한 전략 패턴
 
// 캐릭터의 체력치 계산이 구태여
// 어떤 캐릭터의 일부일 필요가 없습니다.
class GameCharacter;         // 전방 선언
 
// 체력치 계산에 대한 기본 알고리즘을 구현한 함수
int defaultHealthCalc(const GameCharacter& gc);
 
class GameCharacter {
public:
    typedef int(*HealthCalcFunc)(const GameCharacter&);
    
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    : healthFunc(hcf)
    {}
    
    int healthValue(void) const {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};
 
// GameCharacter 클래스 계통에
// 가상 함수를 심는 방법과 비교하면,
// 꽤 재미있는 융통성을 갖고 있습니다.
 
// 1. 같은 캐릭터 타입으로부터 만들어진 객체(인스턴스)들도
//    체력치 계산 함수를 각각 다르게 가질 수 있습니다.
class EvilBadGuy: public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc)
    : GameCharacter(hcf) {
        ...
    }
    ...
};
 
// 다른 동작 원리로 구현된 체력치 계산 함수들
int loseHealthQuickly(const GameCharacter&);
int loseHealthSlowly(const GameCharacter&);
 
// 같은 타입인데도 체력치 변화가 다르게 나오는 캐릭터들
EvilBadGuy ebg1(loseHealthQuickly);
EvilBadGuy ebg2(loseHealthSlowly);
 
// 2. 게임이 실행되는 도중에 특정 캐릭터에 대한
//    체력치 계산 함수를 바꿀 수 있습니다.
 
// 예를 들어 GameCharacter 클래스에서
// setHealthCalculator라는 멤버 함수를
// 제공하고 있다면 이를 통해 현재 쓰이는
// 체력치 계산 함수의 교체가 가능해지는 것이죠.
```

* 가상 함수를 tr1::function 데이터 멤버로 대체하여, 호환되는 시그너처를 가진 함수호출성 개체를 사용할 수 있도록 만듭니다: 역시 전략 패턴의 한 형태입니다.

```c++
// ※ tr1::function으로 구현한 전략 패턴
 
// 함수 포인터 대신 tr1::function을
// 사용하여 호환되는 시그너처를 가진 함수호출성
// 개체를 사용할 수 있도록 만들 수도 있습니다.
class GameCharacter;                            // 이전과 같습니다.
int defaultHealthCalc(const GameCharacter& gc);    // 이전과 같습니다.
 
class GameCharacter {
public:
    // HealthCalcFunc는 함수호출성 객체로서, GameCharacter와 호환되는
    // 어떤 것이든 넘겨받아서 호출될 수 있으며 int와 호환되는 모든 타입의
    // 객체를 반환합니다.
    typedef std::tr1::function<int (const GameCharacter&)>
                                    HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    : healthFunc(hcf)
    {}
    
    int healthValue(void) const {
        return healthFunc(*this);
    }
    ...
private:
    HealthCalcFunc healthFunc;
};
 
// 함수 포인터를 사용한 것과 크게 다를 것은 없습니다.
// 다른 점이 있다면 좀더 "일반화된" 함수포인터를
// 사용했다는 것입니다.
 
// 덕분에 사용자 쪽에선 체력치 계산 함수를
// 지정하는 데 있어서 머리가 멍해질 정도로
// 불어난 융통성을 만끽할 수 있게 되었습니다.
 
// 반환 타입이 int와 호환되기만 하면 됩니다.
short calcHealth(const GameCharacter&);
 
// 체력치 계산용 함수 객체를 만들기 위한 클래스
struct HealthCalculator {
    int operator()(const GameCharacter&) const {
        ...
    }
};
 
class GameLevel {
public:
    // 반환 타입이 int와 호환되기만 하면 됩니다.
    float health(const GameCharacter&) const;
    ...
};
 
// 바로 이전의 예제 코드에 나온 그것과 동일합니다.
class EvilBadGuy: public GameCharacter {
    ...
};
 
// 또 하나의 캐릭터 타입.
// 생성자는 EvilBadGuy의 그것과
// 똑같다고 가정합니다.
class EyeCandyCharacter: public GameCharacter {
    ...
};
 
// 체력치 계산을 위한 함수를 사용하는 캐릭터
EvilBadGuy ebg1(calcHealth);
 
// 체력치 계산을 위한 함수를 사용하는 캐릭터
EyeCandyCharacter eccl(HealthCalculator());
 
GameLevel currentLevel;
...
// 체력치 계산을 위한 멤버 함수를 사용하는 캐릭터입니다.
EvilBadGuy ebg2(
    std::tr1::bind(&GameLevel::health,
                    currentLevel,
                    _1)
);
```

* 한쪽 클래스 계통에 속해 있는 가상 함수를 다른 쪽 계통에 속해 있는 가상 함수로 대체합니다: 전략 패턴의 전통적인 구현 형태입니다.

```c++
// ※ "고전적인" 전략 패턴
 
// 체력치 계산 함수를 나타내는 클래스 계통을
// 아예 따로 만들고, 실제 체력치 계산 함수는
// 이 클래스 계통의 가상 멤버 함수로 만드는
// 것입니다.
 
class GameCharacter;    // 전방 선언
 
class HealthCalcFunc {
public:
    ...
    virtual int calc(const GameCharacter& gc) const {
        ...
    }
    ...
};
 
HealthCalcFunc defaultHealthCalc;
 
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc *phcf = &defaultHealthCalc)
    : pHealthCalc(phcf)
    {}
    
    int healthValue(void) const {
        return pHealthCalc->calc(*this);
    }
    ...
private:
    HealthCalcFunc *pHealthCalc;
};
```

* 이것만은 잊지 말자
    * 가상 함수 대신에 쓸 수 있는 다른 방법으로 NVI 관용구 및 전략 패턴을 들 수 있습니다. 이 중 NVI 관용구는 그 자체가 템플릿 메서드 패턴의 한 예입니다.
    * 객체에 필요한 기능을 멤버 함수로부터 클래스 외부의 비멤버 함수로 옮기면, 그 비멤버 함수는 그 클래스의 public 멤버가 아닌 것들을 접근할 수 없다는 단점이 생깁니다.
    * tr1::function 객체는 일반화된 함수 포인터처럼 동작합니다. 이 객체는 주어진 대상 시그너처와 호환되는 모든 함수호출성 개체를 지원합니다.


## 항목 36: 상속받은 비가상 함수를 파생 클래스에서 재정의하는 것은 절대 금물!
* 아래처럼 되어있다고 생각해보자

```c++
class B{
public:
    void mf();
    ...
};
class D:public B{...};
```
* B, D 혹은 mf에 대해 모르는 상태에서 D타입의 객체인 x가 다음처럼 있을때 

```c++
D x;
B *pB = &x;
pB->mf();

//아래처럼 동작하지 않으면 황당할 것인다.
D *pD = &x;
pD->mf();//
```
* 그런데, 다를수 있다. 특히 mf가 비가상 함수이고 D 클래스가 자체적으로 mf 함수를 또 정의하고 잇으면 다른 동작을 한다.

```c++
class D:public B{
public:
    void mf();//B::mf 를 가려버린다 (항목 33)
};
pB->mf();//B::mf 호출
pD->mf();//D::mf 호출
```
* 이렇게 동작하는 이유는 B::mf, D::mf 등의 비가상 함수는 정적 바인딩으로 묶이기 때문이다. 
    * pB는 B에 대한 포인터 타입으로 선언되었기에 pB를 통해 호출되는 비가상 함수는 항상 B 클래스에 정의되어 있을것이라 결정해 버린다는 말이다.
* 반면 가상 함수의 경우엔 동적 바인딩으로 묶여있다.(항목 37)
    * mf 함수가 가상함수였으면 mf가 pB에서 호출되든 pD에서 호출되던 D::mf가 호출된다. pB, pD가 진짜로 가르키는 대상은 D타입의 객체닌깐.
* 만약 비가상 함수를 정말 원해서 재정의 한 것이고, 재정의한 비가상 함수는 반드시 기본 클래스와 그 파생 클래스에서 같은 동작을 해야 한다고 정한것도 진짜라면, 비가상 함수의 재정의로 인해 "모든 파생 클래스는 기본 클래스의 일종" 이라는 명제는 거짓이 됩니다. 이런 상황이라면 두 클래스를 public 상속 관계로 만들면 안 됩니다.
* 클래스 D는 클래스 B로부터 public 상속을 받아 파생시킬 수밖에 없는 사정이 있고, 진짜로 D에서 B의 비가상 함수를 재정의해야 한다면, "비가상 함수는 클래스 파생에 상관없이 B에 대한 불변동작을 나타낸다."라는 점도 거짓이 됩니다. 이런 경우라면 그 비가상 함수는 가상 함수가 되어야 하는 것이 맞습니다.
* 이것만은 잊지 말자.
    * 상속받은 비가상 함수를 재정의하는 일은 절대로 하지 맙시다.

## 항목 37: 어떤 함수에 대해서도 상속받은 기본 매개변수 값은 절대로 재정의하지 말자
* C++에서 상속받을 수 있는 함수는  두 가지
    * 가상함수
        * 기본 매개변수 값을 가진 가상함수를 상속하는 경우로 좁혀서 이야기 해보자.
    * 비가상함수
        * 언제라도 재정의 해서는 안되는 함수(항목 36) 여기선 굳이 이야기 하지 않겠다. 
    * 이유인즉 가상 함수는 동적으로 바인딩 되지만, 기본 매개변수 값은 정적으로 바인딩 된다는 것이다. 

* 객체의 정적타입(static type)은 소스 안에 선언문을 통해 그 객체가 갖는 타입이다. 아래 예제를 보자. 

```c++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    //모든 도형은 자기 자신을 그리는 함수를 제공해야한다. 
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};
 
class Rectangle: public Shape {
public:
    //기본 매개변수 값이 달라지는 부분을 놓치지 마세요. 큰일 났음!
    virtual void draw(ShapeColor color = Green) const;
    ...
};

class Circle: public Shape {
public:
    virtual void draw(ShapeColor color) const;
    ...
};
```
* 이들을 써서 포인터를 나타내면

```c++
Shape *ps;//정적 타입 = Shape*
Shape *pc = new Circle;//정적 타입 = Shape*
Shape *pr = new Rectangle;//정적 타입 = Shape*
```
* 객체의 동적타입(dynamic type)은 형재 그 객체가 진짜로 무엇이냐에 따라 결정되는 타입
    * 위의 예제에서 pc의 동적타입은 Circle * 이고 pr의 동적타입은 Rectangle * 이고 ps인 경우는 아무 객체도 찹고하고있지 않아 동적 타입이 없다.
* 동적 타입은 실행도중에 바뀔수 있고 대개 대입문을 통해 바뀐다.

```c++
ps = pc;//ps 의 동적 타입은 이제 Circle*가 된다.
ps = pr;//ps 의 동적 타입은 이제 REctangle*가 된다.
```
* 가상 함수는 동적으로 바인딩됨
    * 호출이 일어난 객체의 동적 타입에 따라 어떤 가상 함수가 호출될지 결정된다는 뜻임.

```c++
pc->draw(Shape::Red);//Circle::draw(Shape::Red)를 호출함
pr->draw(Shape::Red);//Rectangle::Draw(Shape::Red)를 호출함
```
* 그런데, 기본 매개변수 값이 설정된 가상함수로 오게 되면 꼬이기 시작함.!
    * 가상 함수는 동적으로 바인딩 되어있지만, 기본 매개변수는 정적으로 바인딩 되어있기 때문
    * 파생 클래스에 정의된 가상 함수를 호출하면서 기본 클래스에 정의된 기본 매개변수 값을 사용해 버릴수도있음.

```c++
pr->draw();//Rectangle::draw(SHape::Red)를 호출!
//pr의 동적타입이 Rectangle* 므로 호출되는 가상함수는 Rectangle의 것인데 기본 매개변수는 Green으로 되어있음.하지만 pr의 정적 타입은 Shape*이기에 가상함수에 쓰이는 기본 매개변수 값을 Shape 클래스에서 가져옮. 이상하게 섞이고 예상하기 힘들수 있다.
```
* 그럼 어떻게 하면 좋을까?

```c++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};
 
class Rectangle: public Shape {
public:
    virtual void draw(ShapeColor color = Red) const;
    ...
};
```
* 이렇게 하면 중복코드가 된다. 
* 비가상 인터페이스(non-virtual interface) 관용구(NVI 관용구) 를 쓰면 된다. 
    * 파생 클래스에서 재정의 할수 있는 가상 함수를 private멤버로 두고
    * 이 가상 함수를 호출하는 public 비가상 함수를 기본 클래스에 만들어 두는 방식.

```c++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    
    // 이제는 비가상 함수입니다.
    void draw(ShapeColor color = Red) const {
        doDraw(color);        // 가상 함수를 호출합니다.
    }
    ...
    
private:
    // 진짜 작업은 이 함수에서 이루어 집니다.
    virtual void doDraw(ShapeColor color) const = 0;
};
 
class Rectangle: public Shape {
public:
    ...
private:
    // 기본 매개변수 값이 없습니다. 잘 보세요.
    virtual void doDraw(ShapeColor color) const;
    ...
};
```
* 이것 만은 잊지 말자
    * 상속받은 기본 매개변수(디폴트 매개변수) 값은 절대로 재정의해서는 안 됩니다. 왜냐하면 기본 매개변수 값은 정적으로 바인딩되는 반면, 가상 함수(여러분이 오버라이드할 수 있는 유일한 함수이죠)는 동적으로 바인딩되기 때문입니다.


## 항목 38: "has-a(...는 ...를 가짐)" 혹은 "is-implemented-in-terms-of(...는 ...를 써서 구현됨)"를 모형화할 때는 객체 합성을 사용하자
* 합성 이란
    * 어떤 타입의 객체들이 그와 다른 타입의 객체들을 포함하고있는 경우에 성립하는 그 타입들 사이의 관계를 일컫음
    * 합성이랑 용어 대신에 레이어링, 포함, 통합, 내장 등으로도 알려짐.
* public 상속의 의미: is-a(...는 ...의 일종)
* 객체 합성의 의미: has-a(...는 ...를 가짐) 혹은 is-implemented-in-terms-of(...는 ...를 써서 구현됨)

* 객체 합성이 
    * 응용 영역에서 일어나면 has-a 관계
    * 구현 영역에서 일어나면 is-implemented-in-terms-of 관계를 나타냄

* 일상생활에서 볼 수 있는 사물을 본뜬 객체는 소프트웨어의 응용 영역에 속합니다.
* 응용 영역에 속하지 않는 나머지(버퍼, 뮤텍스, 탐색 트리 등 순수하게 시스템 구현만을 위한 인공물) 객체는 소프트웨어의 구현 영역에 속합니다.

* 이것만은 잊지 말자
    * 객체 합성(composition)의 의미는 public 상속이 가진 의미와 완전히 다릅니다.
    * 응용 영역에서 객체 합성의 의미는 has-a(...는 ...를 가짐)입니다. 구현 영역에서는 is-implemented-in-terms-of(...는 ...를 써서 구현됨)의 의미를 갖습니다



## 항목 39: private 상속은 심사숙고해서 구사하자
* C++에서 public 상속을 is-a관계로 나타낸다고 했다. 앞의 예제중 하나를 public상속이 아닌 private 상속으로 살짝 바꿔보겠다.

```c++
class Person{...};
class Student: private Person{...};//이제 private 상속이다.
void eat(const Person& p);//누구라도 사람은 먹는다.
void study(const Student& s);//공부는 학생만 한다.
Person p;
Sutdent s;

eat(p);//ok
eat(s);//error Student는 Person 의 일종이 아니다. 
```
* private 상속은 분명히 is-a를 뜻하지 않는다. 
* private 상속은
    * 컴파일러는 일반적으로 파생 클래스 객체(ex. Student)를 기본 클래스 객체(Person)로 변환하지 않는다.
    * 기본 클래스부터 물려받은 멤버는 파생 클래스에서 모조리 private 멤버가 된다. 
        * 기본 클래스에서 원래 protected 멤버였거나 public 멤버였거나 상관없이.
* private 상속의 의미는 is-implemented-in-terms-of 이다.
    * B클래스로부터 private 상속을 통해 D 클래스를 파생하넋은 B 클래스에서 쓸수있는 기능 몇개를 활용할 목적으로 한 행동이지, B 타입과 D타입의 객체 사이에 어떤 개념적인 관계가 있어서 한 행동이 아니라는 것이다. 
* 예를 들어 Widget 객체를 사용하는 프로그램이 있따고 가정할때 Widget의 멤버 함수의 호출 횟수 같은 것을 알고 싶을때 코드를 새로 만드느니 기존 코드를 가져와 사용하는게 더 좋으므로 Timer 클래스 같은 것에서 onTick() 처럼 반복적으로 시간을 경과시켜줄 것을 이용하면 된다 
    * 결국, Widget과 Timer는 is-a 관계는 아니다 개념적으로 onTick()는 WIdget의 인터페이스도 아니다. 그러기에 private 상속을 하는 것이다. 
* 꼭 private 상속을 할 필요가 있는냐에 대한 잘문은 아래처럼 해결할 수도 있다.

```c++
class Timer {
public:
    explicit Timer(int tickFrequency);
    
    virtual void onTick(void) const;    // 일정 시간이 경과할 때마다 자동으로
    ...                                    // 이것이 호출됩니다.
};
 
class Widget: private Timer {
private:
    virtual void onTick(void) const;    // Widget 사용 자료 등을 수집합니다.
    ...
};
 
// 이 경우 Widget은 Timer의 일종이 아니므로
// private 상속을 사용하기에 적당합니다.
 
// 그러나 private 상속을 쓰지 않고도
// 해결할 수 있는 방법이 있습니다.
 
class Widget {
private:
    class WidgetTimer: public Timer {
    public:
        virtual void onTick(void) const;
        ...
    };
    
    WidgetTimer timer;
};
 
// 아래와 같은 방법이 많이 쓰이는 이유는 다음과 같습니다.
 
// 첫째, Widget 클래스를 설계하는 데 있어서
// 파생은 가능하게 하되, 파생 클래스에서 onTick을
// 재정의할 수 없도록 설계 차원에서 막고 싶을 때
// 유용합니다.
 
// 둘째, Widget의 컴파일 의존성을 최소화하고
// 싶을 때 좋습니다. Widget이 Timer에서
// 파생된 상태라면, Widget이 컴파일될 때
// Timer의 정의도 끌어올 수 있어야 하기 때문에,
// Timer.h 같은 헤더를 #include 해야
// 할지도 모릅니다. 반면, 후자의 경우
// WidgetTimer의 정의를 Widget으로부터
// 빼내고 Widget이 WidgetTimer 객체에 대한
// 포인터만 갖도록 만들어 두면, WidgetTimer
// 클래스를 간단히 선언하는 것만으로도 컴파일
// 의존성을 슬쩍 피할 수 있습니다.
```
* private 상속이 적법한 설계 전략일 가능성은
    * is-a 관계로 이어질거 같은않는 두 클래스를 사용해야 하는데, 이 둘 사이에 한쪽 클래스가 다른쪽 클래스의 protected 멤버에 접근해야 하거나, 다른쪽 클래스의 가상 함수를 재정의 해야할때이다.

* 이것만은 잊지 말자
    * private 상속의 의미는 is-implemented-in-terms-of(...는 ...를 써서 구현됨)입니다. 대개 객체 합성과 비교해서 쓰이는 분야가 많지는 않지만, 파생 클래스 쪽에서 기본 클래스의 protected 멤버에 접근해야 할 경우 혹은 상속받은 가상 함수를 재정의해야 할 경우에는 private 상속이 나름대로 의미가 있습니다.
    * 객체 합성과 달리, private 상속은 공백 기본 클래스 최적화(EBO)를 활성화시킬 수 있습니다.이 점은 객체 크기를 가지고 고민하는 라이브러리 개발자에게 꽤 매력적인 특징이 되기도 합니다.



## 항목 40: 다중 상속은 심사숙고해서 사용하자
* 다중 상속의 모호성과 가상 상속의예시를 아래를 보면서 알아 보자

```c++
//-------------------------------------------------------------------
// ※ 다중상속을 사용하는 경우 발생하는 모호성
class BorrowableItem {            // 라이브러리로부터 여러분이
public:                            // 가져올 수 있는 어떤 것
 
    void checkOut(void);        // 라이브러리로부터 체크아웃합니다.
    ...
};
 
class ElectronicGadget {
private:
    bool checkOut(void) const;    // 자체 테스트를 실시하고,
    ...                            // 성공 여부를 반환합니다.
};
 
class Mp3Player:                // MP3 플레이어를 위해
    public BorrowableItem,        // 몇몇 라이브러리로부터
    public ElectronicGadget        // 기능을 가져옵니다.(다중 상속)
{ ... };
 
Mp3Player mp;
 
mp.checkOut();                    // 모호성 발생!
 
// C++컴파일러는 최적 일치 함수를 찾은 후에
// 비로소 함수의 접근가능성을 점검합니다.
 
// 이 경우 두 checkOut함수는
// C++ 규칙에 의한 일치도가 서로 같기
// 때문에, 최적 일치 함수가 결정되지
// 않습니다.
 
// 따라서 ElectronicGadget::checkOut
// 함수의 접근가능성이 점검되는 순서조차
// 오지 않습니다.
 
// 이러한 모호성을 해결하려면, 호출할
// 기본 클래스의 함수를 다음과 같이
// 손수 지정해 주어야 합니다.
 
mp.BorrowableItem::checkOut();
 
 
//-------------------------------------------------------------------
// ※ 가상 상속이 필요해지는 경우
class File { ... };
 
class InputFile: public File { ... };
 
class OutputFile: public File { ... };
 
class IOFile:
    public InputFile,
    public OutputFile
{ ... };
 
// 이와 같은 경우 IOFile은 File 클래스를
// 간접적으로 두 번 상속받게 됩니다.
 
// 만일 File 안에 fileName이라는
// 멤버가 있다면, IOFile에는
// fileName이 두 개가 있게 됩니다.
 
// 만일 fileName이 하나만 있어야 한다면,
// 다음과 같이 File클래스를 가상 기본 클래스로
// 만들어야 합니다.
 
class File { ... };
 
class InputFile: virtual public File { ... };
 
class OutputFile: virtual public File { ... };
 
class IOFile:
    public InputFile,
    public OutputFile
{ ... };
 
// 이제 IOFile 안에는 fileName이
// 하나만 있게 됩니다.
```

* 다중 상속을 적법하게 쓸 예시

```c++
class IPerson {                                // 이 클래스가 나타내는 것은
public:                                        // 용도에 따라 구현될 인터페이스입니다.
    virtual ~IPerson(void);
    
    virtual std::string name(void) const = 0;
    virtual std::string birthDate(void) const = 0;
};
 
class DatabaseID { ... };                    // 아래에서 쓰입니다.
 
 
class PersonInfo {                            // 이 클래스에는 IPerson 인터페이스를
public:                                        // 구현하는 데 유용한 함수가 들어
    explicit PersonInfo(DatabaseID pid);    // 있습니다.
    virtual ~PersonInfo(void);
    
    virtual const char * theName(void) const;
    virtual const char * theBirthDate(void) const;
    
    virtual const char * valueDelimOpen(void) const;
    virtual const char * valueDelimClose(void) const;
    ...
};
 
const char * PersonInfo::valueDelimOpen(void) const
{
    return "[";                                // 기본적으로 지정된 시작 구분자
}
 
const char * PersonInfo::valueDelimClose(void) const
{
    return "]";                                // 기본적으로 지정된 끝 구분자
}
 
const char * PersonInfo::theName(void) const{
    // 반환 값을 위한 버퍼를 예약해 둡니다. 이 버퍼는
    // 정적 메모리이기 때문에, 자동으로 모두 0으로 초기화 됩니다.
    static char value[Max_Formatted_Field_Value_Length];
    
    // 시작 구분자를 value에 씁니다.
    std::strcpy(value, valueDelimOpen());
    
    value에 들어 있는 문자열에 이 객체의 name 필드를 덧붙입니다.
    (버퍼 오버런이 일어나지 않도록 주의해야 합니다!)
    
    // 끝 구분자를 value에 추가합니다.
    std::strcat(value, valueDelimClose());
    return value;
}
 
// 요컨데 valueDelimOpen함수와 valueDelimClose함수를
// 재정의 해야 theName 함수가 반환하는 이름의 포맷을
// 변경할 수 있습니다.
 
class CPerson: public IPerson, private PersonInfo {        // 다중 상속이 쓰였습니다.
public:
    explicit CPerson(DatabaseID pid) : PersonInfo(pid) {}
 
    virtual std::string name(void) const {                // IPerson 클래스의
        return PersonInfo::theName();                    // 순수 가상 함수에
    }                                                    // 대해 파생 클래스의
    virtual std::string birthDate(void) const {            // 구현을 제공합니다.
        return PersonInfo::theBirthDate();
    }
    
private:
    const char * valueDelimOpen(void) const {    // 가상 함수들도
        reutrn "";                                // 상속되므로
    }                                            // 이 함수들에 대한
    const char * valueDelimClose(void) const {    // 재정의 버전을 만듭니다.
        return "";
    }
};

```

* 이것만은 잊지 말자
    * 다중 상속은 단일 상속보다 확실히 복잡합니다. 새로운 모호성 문제를 일으킬 뿐만 아니라 가상 상속이 필요해질 수도 있습니다.
    * 가상 상속을 쓰면 크기 비용, 속도 비용이 늘어나며, 초기화 및 대입 연산의 복잡도가 커집니다. 따라서 가상 기본 클래스에는 데이터를 두지 않는 것이 현실적으로 가장 실용적입니다.
    * 다중 상속을 적법하게 쓸 수 있는 경우가 있습니다. 여러 시나리오 중 하나는, 인터페이스 클래스로부터 public 상속을 시킴과 동시에 구현을 돕는 클래스로부터 private 상속을 시키는 것입니다.


# Chapter 7 템플릿과 일반화 프로그래밍

## 항목 41: 템플릿 프로그래밍의 천릿길도 암시적 인터페이스와 컴파일 타임 다형성부터
* 객체 지향 프로그래밍의 명시적 인터페이스 과 런타임 다형성 에 대해 살펴보자.

```c++
// 아래는 의미없는 예시용 코드입니다.
class Widget {
public:
    Widget(void);
    virtual ~Widget(void);
    virtual std::size_t size(void) const;
    virtual void normalize(void);
    void swap(Widget& other);
    ...
};
 
void doProcessing(Widget& w)
{
    if (w.size() > 10 && w != someNastyWidget) {
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
 
// ※ w는 Widget 타입으로 선언되었기 때문에,
//   w는 Widget 인터페이스를 지원해야 합니다.
//   이 인터페이스를 소스 코드(Widget이 선언된
//   .h 파일 등)에서 찾으면 이것이 어떤
//   형태인지를 확인할 수 있으므로, 이런 인터페이스를
//   가리켜 명시적 인터페이스라고 합니다.
//   다시 말해 소스 코드에 명시적으로 드러나는
//   인터페이스를 일컫습니다.
 
// ※ Widget의 멤버 함수 중 몇 개는
//   가상 함수이므로, 이 가상 함수에 대한 호출은
//   런타임 다형성에 의해 이루어집니다.
//   다시 말해, 특정한 함수에 대한 실제 호출은
//   w의 동적 타입을 기반으로 프로그램 실행 중,
//   즉 런타임에 결정됩니다.
```
* 템플릿과 일반화 프로그래밍의 차이점중 큰 부분은
    * 암시적 인터페이스(implicit interface) 와 컴파일 타임 다형성(compile-time polymorphism) 이다.
        * 이부분은 doProcessing 함수를 템블릿으로 바꿀때 어떤 일이 생기는지만 보면 알 수 있다.
```c++
template<typename T>
void doProcessing(T& w)
{
    if(w.size() > 10 && w != someNastyWidget){
        T temp(w);
        temp.normalize();
        temp.swap(w);
    }
}

// ※ w가 지원해야 하는 인터페이스는 이 템플릿 안에서
//   w에 대해 실행되는 연산이 결정합니다. 지금의
//   경우를 보면, size·normalize·swap
//   멤버 함수를 지원해야 하는 쪽은 w의 타입
//   즉 T입니다. 그 뿐 아니라 T는 복사 생성자도
//   지원해야 하고, 부등 비교를 위한 연산도
//   지원해야 합니다. 진짜 중요한 점은
//   이 템플릿이 제대로 컴파일되려면 몇 개의
//   표현식이 '유효(valid)'해야 하는데
//   이 표현식들은 바로 T가 지원해야 하는
//   암시적 인터페이스라는 점입니다.
 
// ※ w가 수반되는 함수 호출이 일어날 때
//   템플릿의 인스턴스화가 일어납니다.
//   이러한 인스턴스화가 일어나는 시점은
//   컴파일 도중입니다. 인스턴스화를 진행하는
//   함수 템플릿에 어떤 템플릿 매개변수가
//   들어가느냐에 따라 호출되는 함수가
//   달라지기 때문에, 이것을 가리켜
//   컴파일 타임 다형성이라고 합니다.
```
* 런타임 타형성과 컴파일 타임 다형성의 차이는
    * 오버르도된 함수중 지금 호출할 것을 골라내는 과정(컴파울 중에 일어남)과 가상 함수 호출의 동적 바인딩(프로그램 실행중 일어남)의 차이와 흡사함.
* 명시적 인터페이스와 암시적 인터페이스의 차이는
    * 명시적 인터페이스는 대개 함수 시그니처로 이루어짐
    * 암시적 인터페이스는 함수 시그니처에 기반하고 있지 않다는 것이 가장 큰 차이임
        * 암시적 인터페이스를 이루는 요소는 유효 표현식임
        * 위의 예제에서 T는 다음과 같은 제약이 걸릴것임 ->  (if(w.size() > 10 && w != someNastyWidget){)
            * 정수 계열의 값을 반환하고 이름이 size인 함수를 지원해야함
            * T 타입의 객체 둘을 비교하는 operator!= 함수를 지원해야함
        * 실제로는 연산자 오버로딩의 가능성이 있기에 T 는 위의 두가지 제약중 어떤 것도 만족시킬 필요가 없음.
* 템플릿 매개변수에 요구되는 암시적 인터페이스는 컴파일 도중에 점검된다. 
    * 어떤 객체를 쓰려고 할때 그 템플릿피 요구하는 암시적 인터페이스를 그 객체가 지원하지 않으면 사용이 불가능함(컴파일이 안됨.)

* 이것만은 잊지 말자
    * 클래스 및 템플릿은 모두 인터페이스와 다형성을 지원합니다.
    * 클래스의 경우, 인터페이스는 명시적이며 함수의 시그너처를 중심으로 구성되어 있습니다. 다형성은 프로그램 실행 중에 가상 함수를 통해 나타납니다.
    * 템플릿 매개변수의 경우, 인터페이스는 암시적이며 유효 표현식에 기반을 두어 구성됩니다. 다형성은 컴파일 중에 템플릿 인스턴스화와 함수 오버로딩 모호성 해결을 통해 나타납니다.





## 항목 42: typename의 두 가지 의미를 제대로 파악하자
* 아래 두 템플릿문에 쓰인 class와 typename의 차이는 무엇일까?
    * 답은 차이가 없다이다. 템플릿 타입 매개 변수를 선언할때는 class와 typename의 뜻이 완전이 동일하다. 

```c++
template<class T> class Widget;
template<typename T> class Widget;
```
* 그렇다고 class와 typename이 C++에서 언제나 동일한 것은 아니다.
* 그래서 일단 템플릿 안에서 참조할 수 있는 이름의 종류가 두 가지라는 것부터 알아야 하겠다. 
* 아래의 예제가 봐봅시다. 
```c++
template <typename C>                    // 컨테이너에 들어 있는
void print2nd(const C& container)        // 두 번째 원소를 출력합니다.
{                                        // 도저히 제 정신으로 짠 코드가 아닙니다!
    if (container.size() >= 2) {
        C::const_iterator iter(container.begin());
                                        // 첫째 원소에 대한 반복자를 얻습니다.
        ++iter;                            // iter를 두 번째 원소로 옮깁니다.
        int value = *iter;                // 이 원소를 다른 int로 복사합니다.
        std::cout << value;                // 이 int를 출력합니다.
    }
}
// iter의 타입은 C::const_iterator인데,
// 템플릿 매개변수 C에 따라 달라지는 타입입니다.
// 템플릿 내의 이름 중에 이렇게 템플릿 매개변수에
// 종속된 것을 가리켜 의존 이름(dependent name)
// 이라고 합니다. 의존 이름이 어떤 클래스 안에
// 중첩되어 있는 경우가 있는데, 이를 중첩 의존 타입 이름
// (nested dependent type name)이라고
// 부릅니다. (필자는 중첩 의존 이름(nested
// dependent name)이라고 줄여 부르기도 합니다.)
// 위의 코드에서 C::const_iterator는
// 중첩 의존 타입 이름입니다.
 
// 반면 int와 같이 템플릿 매개변수가 어떻든
// 상관이 없는 타입 이름을 비의존 이름
// (non-dependent name)이라고 합니다.



 
template <typename C>
void print2nd(const C& container)
{
    C::const_iterator * x;
    ...
}
 
// 이런 코드의 경우처럼 C::const_iterator가
// 타입인지, C의 정적 데이터 멤버인지
// (전역 변수 x와 곱하기) 모호한 경우가
// 발생할 수 있습니다. 이 모호성을 해결하기 위해
// C++의 구문 분석기는 템플릿 안에서 중첩 의존 이름을
// 만나면, 프로그래머가 그것이 타입이라고 명시하지 않는 한
// 타입이 아니라고 가정하도록 만들어 집니다.
 
// 따라서 중첩 의존 이름이 타입이라면 프로그래머는
// 컴파일러에게 이 이름이 타입이라는 것을 알려줘야 합니다.
 
template <typename C>            // typename 쓸 수 있음("class"와 같은 의미)
void f(const C& container,        // typename 쓰면 안 됨
    typename C::iterator iter);    // typename 꼭 써야 함
    
// 여기서 typename의 역할이 바로 중첩 의존 이름이
// 타입이라는 것을 컴파일러에게 알려 주는 것입니다.
 
// 예외로 기본 클래스 리스트나 생성자 초기화 리스트에
// 있는 중첩 의존 이름의 경우에는 typename을
// 쓰면 안됩니다.
 
template <typename T>
class Derived: public Base<T>::Nested {    // 상속되는 기본 클래스 리스트:
public:                                    // typename 쓰면 안 됨
    explicit Derived(int x)
    : Base<T>::Nested(x)                // 멤버 초기화 리스트에 있는 기본
    {                                    // 클래스 식별자: typename 쓰면 안 됨
    
        typename Base<T>::Nested temp;    // 중첩 의존 타입 이름이며
        ...                                // 기본 클래스 리스트에도 없고
    }                                    // 멤버 초기화 리스트의 기본 클래스
    ...                                    // 식별자도 아님: typename 필요
};
 
 
// 또 다른 예
template <typename IterT>
void workWithIterator(IterT iter)
{
    // temp의 타입은 "IterT 타입의 객체로 가리키는 대상의 타입" 입니다.
    typename std::iterator_traits<IterT>::value_type temp(*iter);
    ...
}
 
// 타입 이름이 너무 길어 typedef로 이름을 만듭니다.
 
template <typename IterT>
void workWithIterator(IterT iter)
{
    // 특성정보 클래스에 속한 value_type 등의 멤버 이름에 대해
    // typedef 이름을 만들 때는 그 멤버 이름과 똑같이 짓는 것이
    // 관례로 되어 있습니다.
    typedef typename std::iterator_traits<IterT>::value_type value_type;
    
    value_type temp(*iter);
    ...
}
 
// typedef typename 처럼 연달아 나오는 것이
// 어색할 수 있으나, 이렇게 써야 적법한 문장이
```
* 이것만은 잊지 말자
    * 템플릿 매개변수를 선언할 때, class 및 typename은 서로 바꾸어 써도 무방합니다.
    * 중첩 의존 타입 이름을 식별하는 용도에는 반드시 typename을 사용합니다. 단, 중첩 의존 이름이 기본 클래스 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우에는 예외입니다.

## 항목 43: 템플릿으로 만들어진 기본 클래스 안의 이름에 접근하는 방법을 알아 두자
* 예시와 함께 알아보는 이번 항목

```c++
//---------------------------------------------------------------------------------------
class MsgInfo { ... };        // 메시지 생성에 사용되는
                            // 정보를 담기 위한 클래스
template <typename Company>
class MsgSender {
public:
    ...                        // 생성자, 소멸자, 등등
    void sendClear(const MsgInfo& info) {
        std::string msg;
        info로부터 msg를 만듭니다;
        
        Company c;
        c.sendCleartext(msg);
    }
    ...
};
 
// 여기서 메시지를 보낼 때마다 관련 정보를
// 로그로 남기고 싶어서 파생 클래스를
// 만든다고 가정합니다.
 
template <typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...                                        // 생성자, 소멸자, 등등
    void sendClearMsg(const MsgInfo& info) {
        "메시지 전송 전" 정보를 로그에 기록합니다;
        
        sendClear(info);                    // 기본 클래스의 함수를 호출하는데,
                                            // 이 코드는 컴파일되지 않습니다.
        "메시지 전송 후" 정보를 로그에 기록합니다;
    }
    ...
};
 
// 컴파일러가 LoggingMsgSender 클래스 템플릿의 정의와
// 마주칠 때, 컴파일러는 대체 이 클래스가 어디서 파생된 것인지를
// 모른다는 것이 문제입니다. MsgSender<Company>에서
// 파생되었지만, Company는 템플릿 매개변수이고, 이 템플릿
// 매개변수는 LoggingMsgSender가 인스턴스로 만들어질 때
// 까지 무엇이 될지 알 수 없습니다.
 
// 예를 들면 MsgSender 클래스가 특정한 Company 타입에
// 대해서 sendClear 멤버 함수를 갖지 않도록 특수화 되었을
// 수도 있습니다.
 
// 이를 해결하기 위한 방법은 세 가지나 있습니다.
 
 
//---------------------------------------------------------------------------------------
// 첫째, 기본 클래스 함수에 대한 호출문 앞에 "this->"를 붙입니다.
template <typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...
    void sendClearMsg(const MsgInfo& info) {
        "메시지 전송 전" 정보를 로그에 기록합니다;
        
        this->sendClear(info);                // 좋습니다. sendClear가
                                            // 상속되는 것으로 가정합니다.
        "메시지 전송 후" 정보를 로그에 기록합니다;
    }
    ...
};
 
 
//---------------------------------------------------------------------------------------
// 둘째, using 선언을 이용합니다.
template <typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    using MsgSender<Company>::sendClear;    // 컴파일러에게 sendClear 함수가
    ...                                        // 기본 클래스에 있다고 가정하라고
                                            // 알려 줍니다.
    void sendClearMsg(const MsgInfo& info) {
        ...
        sendClear(info);                    // 좋습니다. sendClear가
        ...                                    // 상속되는 것으로 가정합니다.
    }
    ...
};
 
 
//---------------------------------------------------------------------------------------
// 셋째, 호출할 함수가 기본 클래스의 함수라는 점을
// 명시적으로 지정하는 것입니다.
template <typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...
    void sendClearMsg(const MsgInfo& info) {
        ...
        MsgSender<Company>::sendClear(info);    // 좋습니다. sendClear
        ...                                        // 함수가 상속되는 것으로
    }                                            // 가정합니다.
    ...
};
 
// 마지막 방법은 추천하지 않는 방법입니다.
// 왜냐하면 호출되는 함수가 가상 함수인 경우에는,
// 이런 식으로 명시적 한정을 해 버리면
// 가상 함수 바인딩(동적 바인딩)이 무시되기
// 때문입니다.
```

* 이것만은 잊지 말자
    * 파생 클래스 템플릿에서 기본 클래스 템플릿의 이름을 참조할 때는, "this->"를 접두사로 붙이거나 기본 클래스 한정문을 명시적으로 써 주는 것으로 해결합시다.

## 항목 44: 매개변수에 독립적인 코드는 템플릿으로부터 분리시키자
* 템플릿이 아닌 코드에서는 코드 중복이 명시적이지만(두 함수, 혹은 두 클래스 사이에 똑같은 부분이 있는지 눈으로 찾아낼수 있지만,)  템플리시 코드에서의 코드 중복은 암시적이다. 소스 코드에서는 템플릿이 하나밖에 없기에 어떤 템플릿이 인스턴스화 될때 발생할 수 있는 코드 중복을 감각적으로 알아야 한다. 
    * 아래의 예시를 살펴 보자. 

```c++
template <typename T,        // T 타입의 객체를 원소로 하는 n행 n열의
          std::size_t n>    // 행렬을 나타내는 템플릿.
class SquareMatrix {
public:
    ...
    void invert(void);        // 주어진 행렬을 그 저장공간에서 역행렬로 만듭니다.
};
```
* 이 템플릿은 T라는 매개변수 받지만, size_t 타입의 비타입 매개변수(non-type parameter)인 n도 받도록 되어있다. 
    * 이제 다시 예제를 보자

```c++
SquareMatrix<double, 5> sm1;
...
sm1.invert();                // SquareMatrix<double, 5>::invert를 호출합니다.
 
SquareMatrix<double, 10> sm2;
...
sm2.invert();                // SquareMatrix<double, 10>::invert를 호출합니다.
 
// 이 코드에서 두 invert 함수는 행렬의 크기만 제외하면
// 같은 코드로 이루어져 있을 것입니다. 이것이 비대화의 원인이 된다.
```
* 이때 invert의 사본이 인스턴스화 되는데, 만들어지는 사본의 개수가 두개 이다. 
    * 상수만 빼면 두 함수는 완전히 똑같다. 이럴때 아래처럼 해결하는게 좋다. 

```c++
template <typename T>                    // 정방행렬에 대해 쓸 수 있는
class SquareMatrixBase {                // 크기에 독립적인 기본 클래스
protected:
    ...
    void invert(std::size_t matrixsize);// 주어진 크기의 행렬을 역행렬로 만듭니다.
    ...
};
 
template <typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
private:
    using SquareMatrixBase<T>::invert;    // 기본 클래스의 invert가 가려지는 것을
                                        // 막기 위한 문장입니다. 항목 33을 보세요.
public:
    ...
    void invert(void) {                    // invert의 기본 클래스 버전에 대해
        this->invert(n);                // 인라인 호출을 수행합니다. "this->"가
    }                                    // 있는 이유는 항목 43을 보세요.
};                                        // (using 선언이 있어서 없어도 되긴 합니다.)
 
// 이제 모든 SquareMatrix<T, n> 클래스 템플릿에서
// T(타입)가 동일하다면, SquareMatrixBase<T>
// 클래스 템플릿의 구현을 공유하게 됩니다.
// 즉 코드 비대화가 줄어듭니다.

// 여기서 SquareMatrixBase::invert 함수가
// 자신이 상대할 데이터가 어떤 것인지 알아야 합니다.
// 이를 위해 다음과 같은 방법이 있습니다.
 
template <typename T>
class SquareMatrixBase {
protected:
    SquareMatrixBase(std::size_t n, T *pMem)    // 행렬 크기를 저장하고, 행렬 값에
    : size(n), pData(pMem) {}                    // 대한 포인터를 저장합니다.
 
    void setDataPtr(T *ptr) {
        pData = ptr;                            // pData에 다시 대입합니다.
    }
    ...
private:
    std::size_t size;                            // 행렬의 크기
    T *pData;                                    // 행렬 값에 대한 포인터
};
 
// 이렇게 설계해 두면, 메모리 할당 방법의 결정 권한이
// 파생 클래스 쪽으로 넘어가게 됩니다.
 
template <typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
public:
    SquareMatrix(void)                    // 행렬 크기(n) 및 데이터 포인터를
    : SquareMatrixBase<T>(n, data) {}    // 기본 클래스로 올려보냅니다.
    ...
private:
    T data[n * n];
};
 
// 이 경우 객체 자체의 크기가 좀 커질 수 있습니다.
// 그래서 이 방법이 마음에 들지 않으면 데이터를
// 힙에 둘 수도 있습니다.
 
template <typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
public:
    SquareMatrix(void)                    // 기본 클래스의 포인터를 널로 설정하고,
    : SquareMatrixBase<T>(n, 0),        // 행렬 값의 메모리를 할당하고,
      pData(new T[n * n])                // 파생 클래스의 포인터에 그 메모리를
    {                                    // 물려 놓은 후, 이 포인터의 사본을
        this->setDataPtr(pData.get());    // 기본 클래스로 올려보냅니다.
    }
    ...
private:
    boost::scoped_array<T> pData;
};

```
* 예를들면 list<int*>, list<const int*>, list<SquareMatrix<long, 3>*> 등 과 같은 포인터 타입에 대해, 템플릿은 똑같은 이진 코드를 생성할 것입니다. 이런 코드의 경우 내부적으로 list<void*>로 동작하는 버전을 호출하는 식으로 만들어서 코드 비대화를 줄일 수 있습니다.

* 이것만은 잊지 말자
    * 템플릿을 사용하면 비슷비슷한 클래스와 함수가 여러 벌 만들어집니다. 따라서 템플릿 매개변수에 종속되지 않는 템플릿 코드는 비대화의 원인이 됩니다.
    * 비타입 템플릿 매개변수로 생기는 코드 비대화의 경우, 템플릿 매개변수를 함수 매개변수 혹은 클래스 데이터 멤버로 대체함으로써 비대화를 종종 없앨 수 있습니다.
    * 타입 매개변수로 생기는 코드 비대화의 경우, 동일한 이진 표현구조를 가지고 인스턴스화되는 타입들이 한 가지 함수 구현을 공유하게 만듦으로써 비대화를 감소시킬 수 있습니다.




## 항목 45: "호환되는 모든 타입"을 받아들이는 데는 멤버 함수 템플릿이 직방!
* 스마트 포인터가 그냥 포인터처럼 동작하면서 포인터가 주지 못하는 기능이 있는것 처럼 포인터에도 스마트 포인터로 대신할 수 없는 특징이 있다.
    * 그중 하나가 암시적 변환을 지원한다는 점이다. 
        * 파생 클래스 포인터는 암시적으로 기본 클래스 포인터로 변환되고, 
        * 비상수 객체에 대한 포인터는 상수 객체에 대한 포인터로의 암시적 변환이 가능하는 등이 있다.

```c++
class Top{...};
class Middle:public Top{...};
class Bottom : public Moddle{...};

Top* pt1 = new Middle;//Middle* -> Top*의 변환
Top* pt2 = new Bottom;//Bottom* -> Top*의 변환
const Top *pct2 = pt1;//Top* -> const Top* 의 변환
```
* 위와 같은 타입 변환을 사용자 정의 스마트 포인터를 써서 흉내내려면 까다롭다.
* 같은 템플릿으로부터 만들어진 다른 인스턴스들 사이에는 어떤 관계도 없기에 컴파일러 눈에는 SmartPtr< Middle > 와 SmartPtr < Top >은 완전히 별개의 클래스다. 
* 아래 예시를 보면서 좀더 살펴 보자 

```c++
template <typename T>
class SmartPtr {
public:
    template <typename U>                // "일반화된 복사 생성자"를
    SmartPtr(const SmartPtr<U>& other);    // 만들기 위해 마련한
    ...                                    // 멤버 템플릿
};
 
// 하지만 이렇게 만든 경우 전혀 호환이 되지 않는
// 타입끼리도 복사 생성이 가능해 집니다.
// 예를 들면 SmartPtr<int*>로부터
// SmartPtr<double*>을 생성할 수
// 있게 됩니다. 이를 해결하기 위해 다음과 같은
// 방법을 사용할 수 있습니다.
 
template <typename T>
class SmartPtr {
public:
    template <typename U>
    SmartPtr(const SmartPtr<U>& other)    // 이 SmartPtr에 담긴 포인터를
    : heldPtr(other.get()) { ... }        // 다른 SmartPtr에 담긴 포인터로 초기화 합니다.
    
    T* get(void) const {
        return heldPtr;
    }
    ...
private:                                // SmartPtr에 담긴
    T *heldPtr;                            // 기본 제공 포인터
};
 
// 이렇게 한 경우 U*에서 T*로 암시적 변환이
// 가능한 경우에만 컴파일 에러가 나지 않습니다.
```
* 프로그래머가 일반화 복사 생성자만 만들고 보통의 복사 생성자를 만들지 않으면, 컴파일러가 보통의 복사 생성자를 만들어 버립니다. 이것은 일반화 복사 생성자가 보통의 복사 생성자의 역할을 수행할수 있건 없건 상관하지 않고 적용됩니다.

* 이것만은 잊지 말자
    * 호환되는 모든 타입을 받아들이는 멤버 함수를 만들려면 멤버 함수 템플릿을 사용합시다
    * 일반화된 복사 생성 연산과 일반화된 대입 연산을 위해 멤버 템플릿을 선언했다 하더라도, 보통의 복사 생성자와 복사 대입 연산자는 여전히 직접 선언해야 합니다.



## 항목 46: 타입 변환이 바람직할 경우에는 비멤버 함수를 클래스 템플릿 안에 정의해 두자
* 아래 예시를 보며 정리하자

```c++
// 이 항목은 항목 24의 내용과 비슷한데,
// 항목 24에서는 Rational 클래스의
// operator* 함수가 예제로 나왔었죠.
 
// 이번 항목에서는 Rational 클래스와
// operator* 함수를 템플릿으로
// 만들 것입니다.
 
template <typename T>
class Rational {
public:
    Rational(const T& numerator = 0,    // 매개변수가 참조자로 전달되는 이유는
             const T& denominator = 1);    // 항목 20을 보면 나옵니다.
            
    const T numerator(void) const;        // 반환 값 전달이 여전히 값에 의한 전달인
    const T denominator(void) const;    // 이유는 항목 28을 보면 나오고요.
    ...                                    // 이들 함수가 const인 이유는 항목 3에서...
};
 
template <typename T>
const Rational<T> operator*(const Rational<T>& lhs,
                            const Rational<T>& rhs)
{ ... }
 
// 항목 24에서 그랬듯 혼합형 연산을 위해
// 연산자 함수가 비멤버 함수로 선언되었습니다.
// 하지만 이 상황에서 다음 코드는
// 컴파일 되지 않습니다.
 
Rational<int> oneHalf(1, 2);        // Rational이 이제 템플릿인 것만 빼면
                                    // 이 예제는 항목 24이 것과 똑같습니다.
 
Rational<int> result = oneHalf * 2;    // 그런데 에러입니다! 컴파일이 안 된다고요.
 
// 두 번째 인자는 int 타입인데,
// 컴파일러는 int타입으로부터 Rational<T>의
// T가 무슨 타입인지를 유추하지 못합니다.
// Rational이 정수형으로부터 암시적으로
// 변환이 가능한데도 말입니다.
 
// 왜냐하면 템플릿 인자 추론(template argument deduction)
// 과정에서는 암시적 타입 변환이 고려되지 않기 때문입니다.
 
// 이런 상황에서, 클래스 템플릿 안에 프렌드 함수를 넣어 두면
// 함수 템플릿으로서의 성격을 주지 않고 특정한 함수 하나를
// 나타낼 수 있다는 사실을 이용할 수 있습니다.
 
// 클래스 템플릿은 템플릿 인자 추론 과정에 좌우되지 않으므로,
// T의 정확한 정보는 Rational<T> 클래스가 인스턴스화될
// 당시에 바로 알 수 있습니다. 그렇기 때문에, 호출 시의
// 정황에 맞는 operator* 함수를 프렌드로 선언하는 데
// 별 어려움이 없는 것입니다.
 
template <typename T>
class Rational {
public:
    ...
friend                                                // operator* 함수를
    const Rational operator*(const Rational& lhs,    // 선언합니다
                            const Rational& rhs);
};
 
template <typename T>                                // operator* 함수를
const Rational<T> operator*(const Rational<T>& lhs,    // 정의합니다.
                            const Rational<T>& rhs)
{ ... }
 
// 이제 oneHalf 객체가 Rational<int> 타입으로
// 선언되면 Rational<int> 클래스가 인스턴스로
// 만들어지고, 이때 그 과정의 일부로서 Rational<int>
// 타입의 매개변수를 받는 프렌드 함수인 operator*도
// 자동으로 선언됩니다.
 
// 이전과 달리 지금은 함수 템플릿이 아니라 함수가 선언된
// 것이므로 컴파일러는 이 호출문에 대해 암시적 변환 함수를
// 적용할 수 있게 됩니다.
 
 
// 하지만 이 코드는 컴파일은 되지만 링크가 되지 않습니다.
// 가장 간단한 해결법은 선언과 정의를 하나로 합쳐
// 클래스 템플릿 안에 넣어 버리는 것입니다.
 
template <typename T>
class Rational {
public:
    ...
friend
    const Rational operator*(const Rational& lhs,
                            const Rational& rhs)
    {
        // 항목 24에서 그대로 가져온 구현 코드
        return Rational(lhs.numerator() * rhs.numerator(),
                        lhs.denominator() * rhs.denominator());
    }
};
 
// 드디어 끝까지 돌아가는 코드가 나왔습니다.
// 하지만 operator* 함수는 클래스 안에
// 정의되었으므로 인라인 함수가 되어 버립니다.
 
// 클래스 바깥에 선언된 도우미 함수만 호출하는
// 식으로 이런 암시적 인라인 선언의 영향을
// 최소화할 수 있습니다.
 
template <typename T> class Rational;                    // Rational
                                                        // 템플릿을
                                                        // 선언합니다.
 
template <typename T>                                    // 도우미 함수
const Rational<T> doMultiply(const Rational<T>& lhs,    // 템플릿을
                             const Rational<T>& rhs);    // 선언합니다.
                            
template <typename T>
class Rational {
public:
    ...
friend
    const Rational<T> operator*(const Rational<T>& lhs,
                                const Rational<T>& rhs)    // 프렌드 함수가 도우미
    { return doMultiply(lhs, rhs); }                    // 함수를 호출하게 만듭니다.
    ...
};
 
// 대다수의 컴파일러에서 템플릿 정의를
// 헤더 파일에 전부 넣어야 합니다.
 
template <typename T>                                       // 필요하면,
const Rational<T> doMultiply(const Rational<T>& lhs,        // 도우미 함수
                             const Rational<T>& rhs)        // 템플릿을
{                                                            // 헤더 파일 안에
    return Rational<T>(lhs.numerator() * rhs.numerator(),    // 정의합니다.
                       lhs.denominator() * rhs.denominator());
}

```
    
* 이것만은 잊지 말자
    * 모든 매개변수에 대해 암시적 타입 변환을 지원하는 템플릿과 관계가 있는 함수를 제공하는 클래스 템플릿을 만들려고 한다면, 이런 함수는 클래스 템플릿 안에 프렌드 함수로서 정의합시다.



## 항목 47: 타입에 대한 정보가 필요하다면 특성정보 클래스를 사용하자
* 특성정보(traits)
    * 컴파일 도중에 어떤 주어진 타입의 정보를 얻을 수 있게 하는 객체를 지칭하는 개념
* 아래 예시를 보면서 알아보자

```c++
// C++ 표준 라이브러리에는 다섯 개의 반복자 범주
// 각각을 식별하는 데 쓰이는 "태그(tag) 구조체"
// 가 정의되어 있습니다.
 
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator_tag {};
struct bidirectional_iterator_tag: public forward_iterator_tag {};
struct random_access_iterator_tag: public bidirectional_iterator_tag {};
 
// 특성정보는 항상 구조체로 구현하는 것으로 굳어진
// 관례가 있습니다.
 
// 특성정보를 다루는 표준적인 방법은
// 해당 특성정보를 템플릿 및 그 템플릿의
// 1개 이상의 특수화 버전에 넣는 것입니다.
 
// 표준 라이브러리에는 다음과 같이
// 준비되어 있습니다.
 
template <typename IterT>    // 반복자 타입에 대한
struct iterator_traits;        // 정보를 나타내는 템플릿
 
// 위처럼 특성정보를 구현하는 데 사용한 구조체를
// '특성정보 클래스'라고 부릅니다.
 
// iterator_traits 클래스는
// 이 반복자 범주를 두 부분으로
// 나누어 구현합니다.
 
// 첫 번째 부분은 사용자 정의 반복자 타입으로
// 하여금 iterator_category 라는
// 이름의 typedef 타입을 내부에 가질 것을
// 요구사항으로 둡니다. 다음은 예시입니다.
 
template < ... >
class deque {
public:
    class iterator {
    public:
        // deque의 반복자는 임의접근 반복자
        typedef random_access_iterator_tag iterator_category;
        ...
    };
    ...
};
 
// 이 iterator 클래스가 내부에 지닌
// 중첩 typedef 타입을 앵무새처럼
// 똑같이 재생한 것이 iterator_traits입니다.
 
// "typedef typename"이 사용된 부분에 대한 설명은
// 항목 42를 보십시오.
template <typename IterT>
struct iterator_traits {
    typedef typename IterT::iterator_category iterator_category;
    ...
};
 
// 하지만 이것만 가지고는 기본 제공 포인터 타입에
// 적용할 수는 없습니다. 포인터 안에 typedef 타입이
// 중첩된다는 것부터가 말이 안 되기 때문입니다.
 
// 따라서 iterator_traits 구현의 두 번째
// 부분은 반복자가 포인터인 경우의 처리입니다.
 
// iterator_traits는 포인터 타입에 대한
// 부분 특수화(partial template specialization)
// 버전을 제공합니다.
 
template <typename IterT>        // 기본제공 포인터 타입에 대한
struct iterator_traits<IterT*>    // 부분 템플릿 특수화
{
    typedef random_access_iterator_tag iterator_category;
    ...
};
 
// ※ 실제로는 std::iterator_traits 입니다.
 
 
// 특성정보 클래스의 설계 및 구현 방법을 정리하면
// 다음과 같습니다.
 
// ※ 다른 사람이 사용하도록 열어 주고 싶은 타입
//   관련 정보를 확인합니다(예를 들어, 반복자
//   라면 반복자 범주 등이 여기에 해당됩니다).
 
// ※ 그 정보를 식별하기 위한 이름을 선택합니다
//   (예: iterator_category).
 
// ※ 지원하고자 하는 타입 관련 정보를 담은
//   템플릿 및 그 템플릿의 특수화 버전
//   (예: iterator_traits)을 제공합니다.

```
* 좀어 보자. 

```c++
template <typename IterT, typename DistT>                // 임의 접근
void doAdvance(IterT& iter, DistT d,                    // 반복자에 대해서는
                std::random_access_iterator_tag)        // 이 구현을 씁니다.
{
    iter += d;
}
 
template <typename IterT, typename DistT>                // 양방향
void doAdvance(IterT& iter, DistT d,                    // 반복자에 대해서는
                std::bidirectional_iterator_tag)        // 이 구현을 씁니다.
{
    if (d >= 0) { while (d--) ++iter; }
    else { while (d++) --iter; }
}
 
template <typename IterT, typename DistT>                // 입력 반복자에 대해서는
void doAdvance(IterT& iter, DistT d,                    // 이 구현을 씁니다.
                std::input_iterator_tag)
{
    if (d < 0) {
        throw std::out_of_range("Negative distance");
    }
    while (d--) ++iter;
}
 
// forward_iterator_tag는 input_iterator_tag로부터
// 상속을 받은 것이므로, input_iterator_tag를
// 매개변수로 받는 doAdvance는 순방향 반복자도
// 받을 수 있습니다.
 
// 이제 advance 함수만 만들면 됩니다.
// advance가 해 줄 일은 오버로딩된
// doAdvance를 호출하는 것뿐입니다.
 
template <typename IterT, typename DistT>
void advance(IterT& iter, DistT d)
{
    doAdvance(                                                // iter의 반복자
        iter, d,                                            // 범주에 적합한
        typename                                            // doAdvance의
            std::iterator_traits<IterT>::iterator_category()// 오버로드 버전을
    );                                                        // 호출합니다.
}
 
// 특성정보 클래스를 어떻게 사용하는지, 마지막으로
// 깔끔히 정리해 봅시다.
 
// ※ "작업자(worker)" 역할을 맡은 함수 혹은
//   함수 템플릿(예: doAdvance)을 특성정보
//   매개변수를 다르게 하여 오버로딩합니다. 그리고
//   전달되는 해당 특성정보에 맞추어 각 오버로드
//   버전을 구현합니다.
 
// ※ 작업자를 호출하는 "주작업자(master)"
//   역할을 맡을 함수 혹은 함수 템플릿(예: advance)
//   을 만듭니다. 이때 특성정보 클래스에서 제공되는
//   정보를 넘겨서 작업자를 호출하도록 구현합니다.
```
* 이것만은 잊지 말자
    * 특성정보 클래스는 컴파일 도중에 사용할 수 있는 타입 관련 정보를 만들어냅니다. 또한 특성정보 클래스는 템플릿 및 템플릿 특수 버전을 사용하여 구합니다.
    * 함수 오버로딩 기법과 결합하여 특성정보 클래스를 사용하면, 컴파일 타임에 결정되는 타입별 if...else 점검문을 구사할 수 있습니다.



## 항목 48: 템플릿 메타프로그래밍, 하지 않겠는가?
* 템플릿 메타프로그래밍(template metaprogramming: TMP)은 
    * 컴파일 중에 실행되는 템플릿 기반의 프로그램을 작성하는 일을 말함
* C++ 프로그래밍에서 TMP가 실력박휘하는 예 3가지를 보자. 
    * 치수 단위(dimensional unit)의 정확성 확인
        * 과학 기술 분야의 응용프로그램을 만들 때는 무엇보다도 치수 단위(예를 들면 질량, 거리, 시간 등)가 똑바로 조합되어야 하는 것이 최우선입니다. TMP를 사용하면 프로그램 안에서 쓰이는 모든 치수 단위의 조합이 제대로 됐는지를 컴파일 동안에 맞춰 볼 수 있습니다. 그것도 계산 시간에 상관 없이 말이죠. 여기서 한 가지 재미있는 점은 바로 분수식 지수 표현이 지원된다는 것입니다. 이런 표현이 가능하려면 컴파일러가 확인할 수 있도록 컴파일 도중에 분수의 약분이 되어야 합니다.
    * 행렬 연산의 최적화
        * // 행렬들의 곱을 계산합니다. 
        * BigMatrix result = m1 * m2 * m3 * m4 * m5;
        * 곱셈 결과를 "보통" 방법으로 계산하려면 네 개의 임시 행렬이 생겨야 합니다. (operator*를 호출할 때 마다 1개의 임시 행렬 생성) 그 뿐 아니라, 행렬 원소들 사이에 곱셈을 해야 하므로 네 개의 루프가 순차적으로 만들어질 수밖에 없습니다. 바로 이런 비싼 연산에 TMP를 사용할 수 있습니다. TMP를 응용한 고급 프로그래밍 기술인 표현식 템플릿(expression template)을 사용하면 덩치 큰 임시 객체를 없애는 것은 물론이고 루프까지 합쳐 버릴 수 있습니다. 게다가 위에 써 놓은 사용자 코드에서 문법 하나 바꾸지 않고도 이 놀라운 기술을 적용할 수 있습니다.
    * 맞춤식 디자인 패턴 구현의 생성
        * 전략(Strategy) 패턴, 감시자(Observer) 패턴, 방문자(Visitor) 패턴 등의 디자인 패턴은 그 구현 방법이 여러 가지일 수 있습니다. TMP를 사용한 프로그래밍 기술인 정책 기반 설계(policy-based design)라는 것을 사용하면, 따로따로 마련된 설계상의 선택 [이것을 "정책(policy)"이라고 합니다]을 나타내는 템플릿을 만들어낼 수 있게 됩니다. 이렇게 만들어진 정책 템플릿은 서로 임의대로 조합되어 사용자의 취향에 맞는 동작을 갖는 패턴으로 구현되는 데 쓰입니다.
예를 들어 몇 개의 스마트 포인터 동작 정책을 하나씩 구현한 각각의 템플릿을 만들어 놓고, 이들을 사용자가 마음대로 조합하여 수백 가지의 스마트 포인터 타입을 컴파일 도중 생성할 수 있게 하는 것입니다. 이 기술은 생성식 프로그래밍(generative programming)의 기초가 됩니다.
* 이것만은 잊지 말자.
    * 템플릿 메타프로그래밍(TMP)은 기존 작업을 런타임에서 컴파일 타임으로 전환하는 효과를 냅니다. 따라서 TMP를 쓰면 선행 에러 탐지와 높은 런타임 효율을 손에 거머쥘 수 있습니다
    * TMP는 정책 선택의 조합에 기반하여 사용자 정의 코드를 생성하는 데 쓸 수 있으며, 또한 특정 타입에 대해 부적절한 코드가 만들어지는 것을 막는 데도 쓸 수 있습니다.






# Chapter 8 new와 delete를 내 맘대로

## 항목 49: new 처리자의 동작 원리를 제대로 이해하자
* 메모리 할당 요청 operator new 함수가 동작하지 못할시(메모리가 없을시) operator new 함수는 예외를 던지게 되어 있다. 
    * 오랜 옛날엔 널 포인터를 반환했었지.
    * operator new 가 예외를 던지기 전에 이 함수는 사용자 쪽에서 지정할 수 있는 에러 처리 함수를 우선적으로 호출하도록 되어있다.
        * new 처리자(new-handler) 라고 한다.
            * 표준 라이브러리에는 set_new_handler 라는 함수가 준비되어 있다.
                * 반환하는 것도, 받는 것도 없고 어떤 예외도 던지지 않을 것이라는 식으로 구현이 되어있다. 

```c++
namespace std{
    typedef void (*new_handler)();
    new_handler set_new_handler(new_handler p) throw();
}
```

* set_new_handler 의 예제를 살펴 보겠다.

```c++
// 충분한 메모리를 operator new 가 할당하지 못핼을 때 호출할 함수
void outOfMem(void)
{
    std::cerr << "Unable to satisfy request for memory\n";
    std::abort();
}
 
int main(void)
{
    std::set_new_handler(outOfMem);
    
    int *pBigDataArray = new int[100000000L];
    ...
}
 
// operator new가 1억 개의
// 정수 할당에 실패하면 outOfMem
// 함수가 호출될 것이고, 이 함수는
// 에러 메시지를 출력하면서 프로그램을
// 강제로 끝내 버릴 것입니다.
 
// 그런데 cerr에 에러 메시지를
// 쓰는 과정에서 또 메모리가 동적으로
// 할당되어야 한다면 어떻게 될까요?
// 잠깐만 생각해 보세요.
```
* 사용자가 부탁한 만큼의 메모리를 할당해 주지 못하면, operator new는 충분한 메모리를 찾아낼 때까지 new 처리자를 되풀이해서 호출합니다. set_new_handler로 지정한 new 처리자는 다음 중 하나를 꼭 수행해야 합니다.
    * 사용할 수 있는 메모리를 더 많이 확보합니다.
        * operator new가 시도하는 이후의 메모리 확보가 성공할 수 있도록 하자는 전략입니다. 구현 방법은 여러가지가 있지만, 프로그램이 시작할 때 메모리 블록을 크게 하나 할당해 놓았다가 new 처리자가 가장 처음 호출될 때 그 메모리를 쓸 수 있도록 허용하는 방법이 그 한 가지입니다.
    * 다른 new 처리자를 설치합니다.
        * 현재의 new 처리자가 더 이상 가용 메모리를 확보할 수 없다 해도, 이 경우 자기 몫까지 해 줄 다른 new 처리자의 존재를 알고 있을 가능성도 있겠지요. 만약 그렇다면 현재의 new 처리자 안에서 set_new_handler를 호출하여 다른 new 처리자를 설치할 수 있습니다. operator new 함수가 다시 new 처리자를 호출할 때가 되면, 가장 마지막에 설치된 new 처리자가 호출되는 것입니다. 이 방법을 살짝 비틀면, new 처리자가 자기 자신의 동작 원리를 변경하도록 만들 수도 있습니다. 이렇게 만드는 한 가지 방법은 new 처리자의 동작을 조정하는 데이터를 정적 데이터 혹은 네임스페이스 유효범위 안의 데이터, 아니면 전역 데이터로 마련해 둔 후에 new 처리자가 이 데이터를 수정하게 만드는 것입니다.
    * new 처리자의 설치를 제거합니다.
        * 다시 말해, set_new_handler에 널 포인터를 넘깁니다. new 처리자가 설치된 것이 없으면, operator new 는 메모리 할당이 실패했을 때 예외를 던지게 됩니다.
    * 예외를 던집니다.
        * bad_alloc 혹은 bad_alloc에서 파생된 타입의 예외를 던집니다. operator new에는 이쪽 종류의 에러를 받아서 처리하는 부분이 없기 때문에, 이 예외는 메모리 할당을 요청한 원래의 위치로 전파(propagate, 예외를 다시 던짐)됩니다.
    * 복귀하지 않습니다.
        * 대게 abort 혹은 exit를 호출합니다.
* Widget 클래스에 대한 메모리 할당 실패를 직접 처리하고 싶을때 어떻게 하면 될지 아래예시를 살펴보자

```c++
//-----------------------------------------------------------------------------
// 특정 클래스(예 Widget)만을 위한
// 할당에러 처리자 구현 예제
class Widget {
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void * operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler currentHandler;
};
 
std::new_handler currentHandler = 0;    // NULL로 초기화합니다.
                                        // 클래스 구현 파일에 두어야 하지요.
                                        
std::new_handler Widget::set_new_handler(std::new_handler p) throw()
{
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}

// 이전에 쓰이고 있던 전역 new 처리자
// 복원의 자동화를 위한 객체
class NewHandlerHolder {
public:
    explicit NewHandlerHolder(std::new_handler nh)    // 현재의 new 처리자를
    : handler(nh) {}                                // 획득합니다.
    
    ~NewHandlerHolder(void) {                        // 이것을 해제합니다.
        std::set_new_handler(handler);
    }
private:
    std::new_handler handler;                        // 이것을 기억해 둡니다.
    
    NewHandlerHolder(const NewHandlerHolder&);        // 복사를 막기 위한 부분
    NewHandlerHolder&                                // (항목 14를 참고하세요)
        operator=(const NewHandlerHolder&);
};
 
void * Widget::operator new(std::size_t size) throw(std::bad_alloc)
{
    NewHandlerHolder                                // Widget의 new 처리자를
        h(std::set_new_handler(currentHandler));    // 설치합니다.
        
    return ::operator new(size);                    // 메모리를 할당하거나
                                                    // 할당이 실패하면 예외를 던집니다.
                                                    
}                                                    // 이전의 전역 new 처리자가
                                                    // 자동으로 복원됩니다.
                                                    
// Widget클래스를 사용하는 쪽에서의 예제
 
void outOfMem(void);                // Widget 객체에 대한 메모리 할당이
                                    // 실패했을 때 호출될 함수의 선언
 
Widget::set_new_handler(outOfMem);    // Widget의 new 처리자 함수로서
                                    // outOfMem을 설치합니다.
 
Widget *pw1 = new Widget;            // 메모리 할당이 실패하면
                                    // outOfMem이 호출됩니다.
 
std::string *ps = new std::string;    // 메모리 할당이 실패하면
                                    // 전역 new 처리자 함수가
                                    // (있으면) 호출됩니다.
 
Widget::set_new_handler(0);            // Widget 클래스만을 위한
                                    // new 처리자 함수가 아무것도 없도록
                                    // 합니다(즉, null로 설정합니다).
 
Widget *pw2 = new Widget;            // 메모리 할당이 실패하면 이제는
                                    // 예외를 바로 던집니다.(Widget
                                    // 클래스를 위한 new 처리자 함수가
                                    // 없습니다.
 
 
//-----------------------------------------------------------------------------
// 위의 예제를 여러 다른 클래스에 적용해도
// 비슷한 코드가 만들어질 것 같습니다.
// 따라서 이 경우 "믹스인(mixin) 양식"의
// 기본 클래스를 만드는 것을 추천합니다.
template <typename T>                // 클래스별 set_new_handler를
class NewHandlerSupport {            // 지원하는 "믹스인 양식"의
public:                                // 기본 클래스
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void * operator new(std::size_t size) throw(std::bad_alloc);
    
    ...                                // operator new의 다른 버전들을
                                    // 이 자리에 둡니다. 항목 52를 참고하세요.
private:
    static std::new_handler currentHandler;
};
 
template <typename T>
std::new_handler
NewHandlerSupport<T>::set_new_handler(std::new_handler p) throw()
{
    std::new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}
 
template <typename T>
void* NewHandlerSupport<T>::operator new(std::size_t size)
    throw(std::bad_alloc);
{
    NewHandlerHolder h(std::set_new_handler(currentHandler));
    return ::operator new(size);
}
 
// 클래스별로 만들어지는 currentHandler 멤버를 널로 초기화합니다.
template <typename T>
std::new_handler NewHandlerSupport<T>::currentHandler = 0;
 
 
// 이제 Widget 클래스에 set_new_handler
// 기능을 추가하는 것은 별로 어려워지지 않게 됩니다.
// 그저 NewHandlerSupport<Widget>로부터
// 상속만 받으면 끝이거든요.
 
class Widget: public NewHandlerSupport<Widget> {
    ...                    // 이전과 같지만, 이제는 set_new_handler 혹은
};                        // operator new에 대한 선언문이 빠져 있습니다.
```
* 다른 예시를 보자
    * new 처리자는 예외를 던지는 new와 예외불가 new, 모두에게 적용됩니다. 따라서 new 처리자의 동작 원리를 제대로 이해해 둡시다.

```c++
class Widget { ... };
 
Widget *pw1 = new Widget;                // 할당이 실패하면
                                        // bad_alloc 예외를 던집니다.
 
if (pw1 == 0) ...                        // 이 점검 코드는 꼭 실패합니다.
 
Widget *pw2 = new(std::nothrow) Widget;    // Widget을 할당하다 실패하면
                                        // 0(널)을 반환합니다.
 
if (pw2 == 0) ...                        // 이 점검 코드는 성공할 수 있습니다.
 
// 여기서 new가 예외를 던지지 않는다 하더라도
// Widget의 생성자에서 예외를 던질 수도 있습니다.
// 즉 예외불가 new는 그때 호출되는
// operator new에서만 예외가 발생되지
// 않도록 보장할 뿐, "new(std::nothrow) Widget"
// 등의 표현식에서 예외가 나오지 않게 막아 준다는
// 이야기는 아닙니다. 이러한 이유 때문에
// 십중팔구는 예외불가 new를 필요로 할 일이
// 없을 것입니다.
```
* 이것만은 잊지 말자
    * set_new_handler 함수를 쓰면 메모리 할당 요청이 만족되지 못했을 때 호출되는 함수를 지정할 수 있습니다.
    * 예외불가(nothrow) new는 영향력이 제한되어 있습니다. 메모리 할당 자체에만 적용되기 때문입니다. 이후에 호출되는 생성자에서는 얼마든지 예외를 던질 수 있습니다.



## 항목 50: new 및 delete를 언제 바꿔야 좋은 소리를 들을지를 파악해 두자
* operator new와 operator delete를 바꾸는 가장 흔한 이유 세 가지는 다음과 같다.
    * 잘못된 힙 사용을 탐지하기 위해
        * new한 메모리에 delete를 하는 것을 잊는다던지, delete한 메모리를 한 번 더 delete한다던지, 데이터 오버런(overrun, 할당된 메모리 블록의 끝을 넘어 뒤에 기록하는 것), 데이터 언더런(underrun, 할당된 메모리 블록의 시작을 넘어 앞에 기록하는 것) 과 같은 실수들을 잡아내기 위해 operator new 와 operator delete를 바꿀 수 있습니다.
    * 효율을 향상시키기 위해
        * 기본으로 제공되는 operator new 와 operator delete는 어떤 환경에서든 무난히 동작하도록 만들어져 있습니다. 이는 어떤 환경에서도 나쁜 성능을 발휘하지는 않지만, 그렇다고 좋은 성능을 발휘하지도 못한다는 말입니다. 프로그래머가 개발하는 소프트웨어의 환경에 적합하도록 operator new 와 operator delete를 바꾼다면, 더 좋은 성능을 내도록 만들 수 있을 것입니다.
    * 동적 할당 메모리의 실제 사용에 관한 통계 정보를 수집하기 위해
        * 여러분이 만드는 소프트웨어가 동적 메모리를 어떻게 사용하는지에 대한 정보를 수집하고, 이를 바탕으로 new 와 delete를 입맛대로 바꾸는 것이 더 좋습니다. 할당된 메모리 블록의 크기가 어떤 분포를 이루는지, 사용 시간은 또 어떤 분포를 보이는지, 메모리를 할당하고 해제하는 순서가 FIFO인지 LIFO인지 그것도 아니라면 임의의 순서인지, 시간 경과에 따라 사용 패턴의 변화는 어떠한지, 동적 할당 메모리의 최대량은 어떤지, 등과 같은 정보를 수집하기 위해 new 와 delete를 바꿀 수 있겠습니다.

* 개념적으로 보면 operator new를 직접 만드는 작업은 그리 어려운게 아니다. 버퍼 오버런 및 언더런을 탐지하기 쉬운 형태로 만들어 주는 예제를 만들어 보겠다. 

```c++
static const int signature = 0xDEADBEEF;
typedef unsigned char Byte;
//고쳐야할 부분이 있음.
void* operator new(std::size_t size) throw(std::bad_alloc)
{
    unsig namespace std;
    size_t realSize = size + 2*sizeof(int);//경계표지 2개를 앞뒤에 붙일수 있을 만큼 메모리 크기를 늘린다.
    void *pMem = malloc(realSize);
    if(!pMem) throw bad_alloc();
    //메모리 블록의 시작 및 끝부분에 경계표지를 기록합니다.
    *(static_cast<int*>(pMem)) = signature;
    *(reinterpret_cast<int*>(static_cast<Byte*>(pMem)+realSIze-sizeof(int))) = signature;
    
    //앞쪽 경계죠피 바로 다음의 메모리를 가리키는 포인터를 반환합니다.
    return static_cast<Byte*>(pMem) + sizeof(int);
}

```
* 위의 함수의 문제중 하나가 바이트 정렬이다.(alignment). 이에 대해 좀 알아 보자
* 컴퓨터의 많은 경우에 있어 아키텍쳐적으로 특정 타입의 데이터가 특정 종류의 메모리 주소를 시작 주소로 하여 저장될 것을 요구사항을 두고 있다.
* 가령 포인터는 4배의 배수에 해당하는 주소에 맞추어 저장되어야(4바이트 단위로 정룔되어야) double 값은 8의 배수에 해당하는 주소에 맞추어 저장되어야 (8바이트 단위로 정렬 되어야)한다는 것이다. 
    * 인텔 x86아키텍쳐는 어떤 바이트 단위에 맞추더라도 double 값을 정렬할 수 있지만 8바이트 단위로 정렬하면 런타임 접근 속도가 훨씬 빨라진다. 
* 바이트 정렬의 문제는 중요하다 모든 operator new 함수는 어떤 데이터 타입에도 바이트 정렬을 적절히 만족하는 포인터를 반환해야 한다는 것이 C++ 의 요구사항이기때문이다.
    * 표준 malloc 함수는 이 요구사항에 맞추어 구현되어있다. 
* operator new 와 operator delete를 기본제공을 다른 버전으로 바꾸는 이유를 좀 더 자세히 알아보자
* 할당 및 해제 속력을 높이기 위해
    * 기본으로 제공되는 범용 할당자는 사용자 정의 버전보다 꽤 느린 경우가 적지 않습니다(항상은 아니지만 말이죠). 특히 사용자 정의 버전이 특정 타입의 객체에 맞추어 설계되어 있으면 더욱 그렇죠. 예를 들어 여러분이 만들 응용프로그램은 단일 스레드로 동작하는데 컴파일러에서 기본으로 제공하는 메모리 관리 루틴이 다중 스레드에 맞게 만들어져 있다면, 스레드 안전성이 없는 할당자를 여러분이 직접 만들어 씀으로써 상당한 속력 이득을 볼 수 있을 것입니다. 물론 operator new 와 operator delete 를 바꾸기 전에, 적절한 프로파일링을 통해 여러분 프로그램 안에서 이들이 진짜로 병목 현상을 일으키는 걸림돌인지 확인하는 센스는 기본입니다.
    * 기본 메모리 관리자의 공간 오버헤드를 줄이기 위해
        * 범용 메모리 관리자는 사용자 정의 버전과 비교해서 속력이 느린 경우도 많은데다가 메모리도 많이 잡아먹는 사례가 허다합니다. 할당된 각각의 메모리 블록에 대해 전체적으로 지우는 부담이 꽤 되기 때문입니다. 크기가 작은 객체에 대한 튜닝된 할당자를 사용하면 이러한 오버헤드를 실질적으로 제거할 수 있습니다.
    * 적당히 타협한 기본 할당자의 바이트 정렬 동작을 보장하기 위해
        * 예를 들어 x86 아키텍쳐에서는 double이 8바이트 단위로 정렬되어 있을 때 읽기·쓰기 속도가 가장 빠릅니다. 하지만 일부 컴파일러 중에는 operator new 함수가 double에 대한 동적 할당 시에 8바이트 정렬을 보장하지 않는 것들이 있다는 슬픈 소식이 나돌고 있습니다. 이런 경우, 기본제공 operator new 대신에 8바이트 정렬을 보장하는 사용자 정의 버전으로 바꿈으로써 프로그램 수행 성능을 확 끌어올릴 수 있습니다.
    * 임의의 관계를 맺고 있는 객체들을 한 군데에 나란히 모아 놓기 위해
        * 한 프로그램에서 특정 자료구조 몇 개가 대게 한 번에 동시에 쓰이고 있습니다. 이들에 대해서 페이지 부재(page fault)발생 횟수를 최소화하고 싶은 경우, 해당 자료구조를 담을 별도의 힙을 생성함으로써 이들이 가능한 한 적은 페이지를 차지하도록 하면 상당히 좋은 효과를 볼 수 있겠지요. 이러한 메모리 군집화는 위치지정(placement) new 및 위치지정 delete를 통해 쉽게 구현할 수 있습니다.
    * 그때그때 원하는 동작을 수행하도록 하기 위해
        * 기본제공 new와 delete가 해 주지 못하는 일을 operator new 및 operator delete가 해 주길 바랄 때가 있게 마련입니다. 메모리 할당과 해제를 공유 메모리에다 하고 싶은데 공유 메모리를 조작하는 일은 C API로밖에 할 수 없을 때가 한 가지 예이죠. 이 때 사용자 정의 버전 위치지정 new와 위치지정 delete를 만드는 것입니다. 또 다른 예로는 응용프로그램 데이터의 보안 강화를 위해 해제한 메모리 블록에 0을 덮어 쓰는 사용자 정의 operator delete를 만드는 경우도 생각해 볼 수 있습니다.

* 이것만은 잊지 말자
    * 개발자가 스스로 사용자 정의 new 및 delete를 작성하는 데는 여러 가지 나름대로 타당한 이유가 있습니다. 여기에는 수행 성능을 향상시키려는 목적, 힙 사용 시의 에러를 디버깅하려는 목적, 힙 사용 정보를 수집하려는 목적 등이 포함됩니다.


## 항목 51: new 및 delete를 작성할 때 따라야 할 기존의 관례를 잘 알아 두자

* operator new[]를 구현할 때 주의할 점은 operator new[] 안에서 해 줄 일은 단순히 원시 메모리의 덩어리를 할당하는 것밖엔 없다는 것입니다. 왜냐하면 상속 때문에, 파생 클래스 객체의 배열을 할당하는 데 기본 클래스의 operator new[] 함수가 호출될 수 있는데, 그렇기 때문에 Base::operator new[] 안에서조차 배열에 들어가는 객체 하나의 크기가 sizeof(Base)라는 가정을 할 수 없습니다. 이 말을 풀이해 보면, Base::operator new[]에서 할당한 배열 메모리에 들어가는 객체의 개수를 (요구된 바이트 수 / sizeof(Base))로 계산할 수 없다는 뜻입니다. 또한, operator new[]에 넘어가는 size_t 타입의 인자는 객체들을 담기에 딱 맞는 메모리 양보다 더 많게 설정되어 있을 수도 있습니다. 왜냐하면 동적으로 할당된 배열에는 배열 원소의 개수를 담기 위한 자투리 공간이 추가로 들어가기 때문입니다.

```c++
//-------------------------------------------------------------------
// 사용자 정의 operator new 의 예제
void * operator new(std::size_t size) throw(std::bad_alloc)
{                                // 여러분의 operator new 함수는 다른
                                // 매개변수를 추가로 가질 수 있습니다.
    using namespace std;
    
    if (size == 0) {            // 0바이트 요청이 들어오면
        size = 1;                // 이것을 1바이트 요구로
    }                            // 간주하고 처리합니다.
    
    while (true) {
        size바이트를 할당해 봅니다;
        if ( 할당이 성공했음 )
            return ( 할당된 메모리에 대한 포인터 );
        
        // 할당이 실패했을 경우, 현재의 new 처리자 함수가
        // 어느 것으로 설정되어 있는지 찾아냅니다(아래를 보세요).
        new_handler globalHandler = set_new_handler(0);
        set_new_handler(globalHandler);
        
        if (globalHandler) (*globalHandler)();
        else throw std::bad_alloc();
    }
}
 
// 현재의 전역 new 처리자 함수를 얻어오는
// 직접적인 방법은 없기 때문에 set_new_handler의
// 반환값을 통해 얻어와야 합니다.
 
 
//-------------------------------------------------------------------
// 어떤 클래스의 멤버 operator new도
// 상속 대상이 됩니다. 이것이 문제를
// 일으킬 수 있습니다. 다음을 보세요.
class Base {
public:
    static void * operator new(std::size_t size) throw(std::bad_alloc);
    ...
};
 
class Derived: public Base        // Derived에서는 operator new가
{ ... };                        // 선언되지 않았습니다.
 
Derived *p = new Derived;        // Base::operator new가 호출되네!
 
// Derived의 크기는 Base보다
// 클 수 있기 때문에 문제가
// 발생할 수 있습니다.
 
// 전체 설계를 바꾸지 않고 쓸 수 있는
// 가장 좋은 해결 방법은 다음과 같습니다.
void * Base::operator new(std::size_t size) throw(std::bad_alloc)
{
    if (size != sizeof(Base))            // "틀린" 크기가 들어오면,
        return ::operator new(size);    // 표준 operator new 쪽에서 메모리
                                        // 할당 요구를 처리하도록 넘깁니다.
    
    ...                                    // 맞는 크기가 들어오면 메모리 할당된
                                        // 요구를 여기서 처리합니다.
}
 
// 0바이트 점검 코드는 sizeof(Base)와
// 비교하는 곳에서 한꺼번에 처리됩니다.
// 왜냐하면 C++에는 모든 독립 구조(freestanding)의
// 객체는 반드시 크기가 0을 넘어야 한다는
// 요상한 금기사항 때문입니다.
```
* 중요한 점을 하나 더 말씀드리자면, 가상 소멸자가 없는 기본 클래스로부터 파생된 클래스의 객체를 삭제하려고 할 경우에는 operator delete로 C++가 넘기는 size_t 값이 엉터리일 수 있습니다. 이것만으로도 기본 클래스에 가상 소멸자를 꼭 두어야 하는 충분한 이유가 선다고 말씀드릴 수 있을 것 같습니다. (기본 클래스에서 가상 소멸자를 빼먹으면 operator delete 함수가 똑바로 동작하지 않을 수 있다는 사실을 잊지 마세요)

```c++
//-------------------------------------------------------------------
// 사용자 정의 operator delete 의 예제
void operator delete(void * rawMemory) throw()
{
    if (rawMemory == 0) return;        // 널 포인터가 delete되려고 할 경우에는
                                    // 아무것도 하지 않게 합니다.
    
    rawMemory 가 가리키는 메모리를 해제합니다;
}
 
 
//-------------------------------------------------------------------
// 클래스 전용 operator delete 의 예제
// 클래스 전용 operator new와 마찬가지로
// 삭제될 메모리의 크기를 점검하는 코드를
// 넣어 주어야 합니다.
class Base {                            // 이전과 같으나, 지금은
public:                                    // operator delete가 선언된 상태
    static void * operator new(std::size_t size) throw(std::bad_alloc);
    static void operator delete(void *rawMemory, std::size_t size) throw();
    ...
};
 
void Base::operator delete(void *rawMemory, std::size_t size) throw()
{
    if (rawMemory == 0) return;            // 널 포인터에 대한 점검
    
    if (size != sizeof(Base)) {            // 크기가 "틀린" 경우
        ::operator delete(rawMemory);    // 표준 operator delete가
        return;                            // 메모리 삭제 요청을 맡도록 합니다.
    }
    
    rawMemory가 가리키는 메모리를 해제합니다;
    
    return;
}
```
* 이것만은 잊지 말자
    * 관례적으로, operator new 함수는 메모리 할당을 반복해서 시도하는 무한 루프를 가져야 하고, 메모리 할당 요구를 만족시킬 수 없을 때 new처리자를 호출해야 하며, 0바이트에 대한 대책도 있어야 합니다. 클래스 전용 버전은 자신이 할당하기로 예정된 크기보다 더 큰(틀린) 메모리 블록에 대한 요구도 처리해야 합니다.
    * operator delete 함수는 널 포인터가 들어왔을 때 아무 일도 하지 않아야 합니다. 클래스 전용 버전의 경우에는 예정 크기보다 더 큰 블록을 처리해야 합니다.


## 항목 52: 위치지정 new를 작성한다면 위치지정 delete도 같이 준비하자
* 아래의 예시를 보면서 알아보자

```c++
Widget *pw = new Widget;
 
// 위의 코드에서는 먼저 operator new 함수가 불려 메모리를 할당한 뒤 Widget의 생성자가 호출.
// 그런데 Widget의 생성자에서 예외가 발생한 경우 프로그래머는 메모리 누수를 해결할 수 없음
 
// 동적 할당 받은 메모리의 시작 주소가 pw에
// 담기기도 전에 예외가 던져지기 때문이죠.
 
// 이 경우 메모리의 해제는 C++
// 런타임 시스템께서 맡이 주시게 됩니다.
 
// 하지만 C++ 런타임 시스템이 알아서
// 메모리를 해제하려면 new와 짝이 맞는
// delete를 알아야 합니다.

// 일반적인 경우 이것은 큰 문제가 되지 않습니다.
 
// 왜냐하면 기본형 operator new는
void* operator new(std::size_t) throw(std::bad_alloc);
 
// 역시 기본형 operator delete와
// 짝을 맞추기 때문입니다.
 
void operator delete(void *rawMemory) throw();    // 전역 유효범위에서의
                                                // 기본형 시그너처
 
void operator delete(void *rawMemory,            // 클래스 유효범위에서의
                    std::size_t size) throw();    // 전형적인 기본형
                                                // 시그너처
 
// 따라서 표준 형태의 new 및 delete만
// 사용하는 한, 런타임 시스템은 new의 동작을
// 되돌릴 방법을 알고 있는 delete을
// 문제없이 찾아낼 수 있습니다.
 
// 하지만 operator new의 기본형이 아닌
// 형태를 선언하기 시작하면 new와 delete짝을
// 맞추는 데 문제가 뽀송뽀송 피어나게 됩니다.
 
// 비기본형이란 바로 다른 매개변수를 추가로 갖는
// operator new를 뜻합니다.
// (이런 operator new를 위치지정
// (placement) new 라고 합니다.)
 
class Widget {
public:
    ...
    static void* operator new(std::size_t size,            // 비표준 형태의
                              std::ostream& logStream)    // operator new
        throw(std::bad_alloc);
    static void operator delete(void *pMemory            // 클래스 전용
                                size_t size) throw();    // operator delete의
                                                        // 표준 형태
    ...
};
 
Widget *pw = new(std::cerr) Widget;    // operator new를 호출하는 데
                                    // cerr을 ostream 인자로 넘기는데, 이때
                                    // Widget 생성자에서 예외가 발생하면
                                    // 메모리가 누출됩니다.
 
// 위에서 호출된 new는 ostream& 타입의
// 매개변수를 추가로 받아들이므로, 이것과 짝을
// 이루는 operator delete 역시 똑같은
// 시그너처를 가진 것이 마련되어 있어야 합니다.
 
void operator delete(void *, std::ostream&) throw();
 
// 이러한 operator delete를
// 위치지정 delete라고 합니다.
 
// 따라서 예제에서 위치지정 delete를 추가해
// 주면 메모리 누출 위험이 사라집니다.
 
class Widget {
public:
    ...
    static void* operator new(std::size_t size, std::ostream& logStream)
        throw(std::bad_alloc);
    static void operator delete(void *pMemory) throw();
    static void operator delete(void *pMemory, std::ostream& logStream)
        throw();
    ...
};
 
Widget *pw = new(std::cerr) Widget;    // 이전과 같은 코드이지만
                                    // 메모리 누출이 없습니다.
 
delete pw;    // 기본형의 operator delete가 호출됩니다.
 
// 일반적인 경우에는 위치지정 delete 대신
// 기본형의 operator delete 가 호출됩니다.
```

* 이제 아래를 더 보자

```c++
class Base {
public:
    ...
    static void* operator new(std::size_t size,            // 이 new가
                              std::ostream& logStream)    // 표준 형태의
        throw(std::bad_alloc);                            // 전역 new를 가립니다.
    ...
};
 
Base *pb = new Base;                // 에러! 표준 형태의 전역
                                    // operator new가 가려지거든요.
 
Base *pb = new(std::cerr) Base;        // 이건 문제없습니다. Base의
                                    // 위치지정 new를 호출합니다.
 
// 파생 클래스는 전역 operator new는
// 물론이고 자신이 상속받는 기본 클래스의
// operator new까지 가려 버립니다.
 
class Derived: public Base {                    // 위의 Base로부터 상속받은 클래스
public:
    ...
    static void* operator new(std::size_t size)    // 기본형 new를 클래스 전용으로
        throw(std::bad_alloc);                    // 다시 선언합니다.
    ...
};
 
Derived *pd = new(std::cerr) Derived;    // 에러! Base의 위치지정
                                        // new가 가려져 있기 때문입니다.
 
Derived *pd = new Derived;                // 문제없습니다. Derived의
                                        // operator new를 호출합니다.
 
// C++가 전역 유효 범위에서 제공하는
// operator new의 형태는 다음
// 세 가지가 표준입니다.
 
void* operator new(std::size_t) throw(std::bad_alloc);    // 기본형 new
 
void* operator new(std::size_t, void*) throw();            // 위치지정 new
 
void* operator new(std::size_t,                            // 예외불가 new
                   const std::nothrow_t&) throw();        // (항목 49 참조)
 
// 어떤 형태든 간에 operator new가
// 클래스 안에 선언되는 순간,
// 앞의 예제처럼 위의 표준 형태들이
// 몽땅 가려지는 것입니다.
 
// 다음과 같은 한 가지 해결책이 있습니다.
// 기본 클래스 하나를 만들고, 이 안에
// new 및 delete의 기본 형태를
// 전부 넣어두십시오.
 
class StandardNewDeleteForms {
public:
    // 기본형 new/delete
    static void* operator new(std::size_t size) throw(std::bad_alloc)
    { return ::operator new(size); }
    
    static void operator delete(void *pMemory) throw()
    { ::operator delete(pMemory); }
    
    // 위치지정 new/delete
    static void* operator new(std::size_t size, void *ptr) throw()
    { return ::operator new(size, ptr); }
    
    static void operator delete(void *pMemory, void *ptr) throw()
    { return ::operator delete(pMemory, prt); }
    
    // 예외불가 new/delete
    static void* operator new(std::size_t,
                              const std::nothrow_t& nt) throw()
    { return ::operator new(size, nt); }
    
    static void operator delete(void *pMemory,
                                const std::nothrow_t&) throw()
    { ::operator delete(pMemory); }
};
 
// 표준 형태에 덧붙여 사용자 정의 형태를 추가하고 싶다면,
// 이제는 이 기본 클래스를 축으로 넓혀 가면 됩니다.
 
// 상속과 using 선언을 2단 콤보로 사용해서 표준 형태를
// 파생 클래스 쪽으로 끌어와 외부에서 사용할 수 있게 만든
// 후에, 원하는 사용자 정의 형태를 선언해 주세요.
 
class Widget: public StandardNewDeleteForms {        // 표준 형태를 물려받습니다.
public:
    using StandardNewDeleteForms::operator new;        // 표준 형태가 (Widget
    using StandardNewDeleteForms::operator delete;    // 내부에) 보이도록 만듭니다.
    
    static void* operator new(std::size_t size,        // 사용자 정의 위치지정
                        std::ostream& logStream)    // new를 추가합니다.
        throw(std::bad_alloc);
        
    static void operator delete(void *pMemory,        // 앞의 것과 짝이 되는
                        std::ostream& logStream)    // 위치 지정 delete를
        throw();                                    // 추가합니다.
    ...
};
```
* 이것만은 잊지 말자
    * operator new 함수의 위치지정(placement) 버전을 만들 때는, 이 함수와 짝을 이루는 위치지정 버전의 operator delete 함수도 꼭 만들어 주세요. 이 일을 빼먹었다가는, 찾아내기도 힘들며 또 생겼다가 안 생겼다 하는 메모리 누출 현상을 경험하게 됩니다.
    * new 및 delete의 위치지정 버전을 선언할 때는, 의도한 바도 아닌데 이들의 표준 버전이 가려지는 일이 생기지 않도록 주의해 주세요.



# Chapter 9 그 밖의 이야기들

## 항목 53: 컴파일러 경고를 지나치지 말자
* 컴파일러의 경고 예시

```c++
class B {
public:
    virtual void f(void) const;
};
 
class D: public B {
public:
    virtual void f(void);
};
 
// B::f는 const 멤버인데 반해
// D::f는 const 멤버가 아닙니다.
// 이런 실수가 있는 경우 컴파일러
// 경고가 뜹니다.
 
// warning: D::f() hides virtual B::f()
 
// 하지만 어떤 컴파일러의 경우 아무런
// 경고를 띄우지 않을 수도 있습니다.
```

* 이것만은 잊지 말자
    * 컴파일러 경고를 쉽게 지나치지 맙시다. 여러분의 컴파일러에서 지원하는 최고 경고 수준에도 경고 메시지를 내지 않고 컴파일되는 코드를 만드는 쪽에 전력을 다 하십시오.
    * 컴파일러 경고에 너무 기대는 인생을 지양하십시오. 컴파일러마다 트집을 잡고 경고를 내는 부분들이 천차만별이기 때문입니다. 지금 코드를 다른 컴파일러로 이식하면서 여러분이 익숙해져 있는 경고 메시지가 온 데 간 데 없이 사라질 수도 있습니다.


## 항목 54: TR1을 포함한 표준 라이브러리 구성요소와 편안한 친구가 되자

* C++98에 명시되어있는 표준 C++ 라이브러리의 구성요소를 다시한번 살펴 보자
    * 표준 템플릿 라이브러리(Standard Template Library: STL)
        * 주요 구성요소로서 컨테이너(vector, string, map 등), 반복자, 알고리즘(find, sort, transform 등), 함수 객체(less, greater 등) 외에 이런저런 컨테이너 어댑터와 함수 객체 어댑터(stack, priority_queue, mem_fun, not1 등)가 있습니다.
    * iostream
        * 사용자 정의 버퍼링, 국제화 기능이 가능한 입출력을 지원하며, 그 외에 cin, cout, cerr, clog 등의 사전정의 객체를 지원합니다.
    * 국제화 지원
        * 여러 로케일(locale)을 활성화시킬 수 있는 기능이 포함되어 있습니다. 또한 wchar_t 등의 타입(대개 16비트/문자) 및 wstring(wchar_t 타입으로 정의한 string)을 쓰면 유니코드를 사용할 수 있습니다.
    * 수치 처리 지원
        * 복소수를 나타내는 템플릿(complex) 및 수치 배열을 나타내는 템플릿(valarray)이 여기에 해당됩니다.
    * 예외 클래스 계통
        * 최상위 클래스인 exception 및 이것으로부터 갈라져 나온 파생 클래스들, 예를 들어 logic_error 및 runtime_error 등이 여기에 포함됩니다.
    * C89의 표준 라이브러리
        * 1989년 버전의 C에 포함된 표준 라이브러리는 전부 C++에도 들어 있습니다.
* TR1의 구성요소를 살짝한번 보자. 
    * 스마트 포인터(smart pointer)
        * tr1::shared_prt, tr1::weak_ptr이 여기에 해당합니다. shared_prt은 비순환형 자료구조 내에 자원의 누출을 방지하는 용도로 적합합니다. 순환형이 되는 경우 순환 구조 때문에 각 개체의 참조 카운트가 0이 되지 않는 현상이 생길 수 있습니다. 이럴때 사용하는 것이 weak_ptr 입니다. weak_ptr은 참초 카운팅에는 영향을 주지 않기 때문에, 어떤 객체를 물고 있는 마지막 shared_prt이 소멸될 때 weak_ptr이 그 객체를 물고 있더라도 객체는 소멸됩니다. 단 이 경우 weak_ptr은 무효상태(invalid)로 표시됩니다.
    * tr1::function
        * 어떤 함수가 가진 시그너처와 호환되는 시그너처를 갖는 함수호출성 개체(callable entity)의 표현을 가능하게 해 주는 템플릿입니다. 보통 int를 받고 string을 반환하는 콜백 함수를 등록하도록 만들고 싶으면 다음과 같이 합니다.

```c++
// int를 받고 string을
// 반환하는 함수가
// 매개변수 타입입니다.
void registerCallback(std::string func(int));

매개변수 이름으로 쓰인 func는 사실 없어도 되는 선택사항이므로, 다음과 같이 쓸 수도 있습니다.

// 위와 동일합니다.
// 매개변수 이름만 빠졌습니다.
void registerCallback(std::string (int));

여기서 매개변수 타입에 해당하는 "std::string (int)" 부분이 바로 함수 시그너처 입니다. 이런 용도에 tr1::function 템플릿을 사용하면 registerCallback이 훨씬 융통성 있게 만들어집니다. int타입과 호환되는 어떤 타입도 전달받으며, string과 호환 가능한 어떤 타입도 반환할 수 있는 그런 함수를 registerCallback의 매개변수로 설정할 수 있는 것이죠.

// 매개변수인 "func"는 이제
// "std::string (int)"와 호환되는
// 시그너처를 갖는 어떤 함수호출성
// 개체도 될 수 있습니다.
void registerCallback(std::tr1::function
                 <std::string (int)> func);
```
* 계속해서 TR1에 대해서 알아보자
    * tr1::bind
        * 현역 STL 바인더로 잘 쓰이고 있는 bind1st 및 bind2nd와 똑같이 동작함은 물론, 그보다 훨씬 더 많은 기능이 있는 범용 바인더 입니다. TR1 이전의 바인더들과 달리 tr1::bind는 상수 멤버 함수 및 비상수 멤버 함수에 상관없이 동작합니다. 또한 참조로 전달되는 매개변수에 대해서도 동작합니다. 마지막으로 외부 보조 없이도 함수 포인터를 자체적으로 다룰 수 있습니다. 항목 35를 보시면 사용 예를 확인하실 수 있습니다.
    * 해시 테이블(hash table)
        * 세트·멀티세트·맵·멀티맵을 구현하는 데 이 해시 테이블이 쓰였습니다. 이들 각각의 인터페이스는 똑같은 이름을 갖는 TR1 이전의 연관 컨테이너의 인터페이스를 본떠서 만들어졌습니다. 각각의 이름은 tr1::unordered_set, tr1::unordered_multiset, tr1::unordered_map, tr1::unordered_multimap 인데 set, multiset, map, multimap에 저장되는 원소와 달리, 원소가 저장되는 순서를 예측할 수 없다는 점을 강조하는 듯한 인상이 팍팍 풍깁니다.
    * 정규 표현식(regular expression)
        * 정규 표현식 기반의 탐색과 문자열에 대한 대체 연산이 가능하며, 일치되는 원소들 사이의 순회도 지원합니다.
    * 투플(tuple)
        * pair 템플릿의 신세대 버전. pair의 경우에는 두 개만 담을 수 있지만, tuple 객체는 몇 개라도 담을 수 있습니다.
    * tr1::array
        * 근본부터 "STL스럽게 된" 배열입니다. 다시 말해 begin 및 end 등의 멤버 함수를 지원하는 배열입니다. tr1::array 객체의 크기는 컴파일 과정에서 고정됩니다(동적 메모리를 쓰지 않는다는 뜻이죠).
    * tr1::mem_fn
        * 멤버 함수 포인터를 적응시키는(adapt) 용도에 쓸 수 있는, 문법적으로 천하통일을 이룬 템플릿. C++98의 mem_fun 및 mem_fun_ref를 그대로 껴안음과 동시에 기능을 확장하였습니다.
    * tr1::reference_wrapper
        * 기존의 참조자가 객체처럼 행세할 수 있도록 만들어 주는 템플릿입니다. 무엇보다, 이것을 사용하면 참조자를 담은 것처럼 동작하는 컨테이너를 만들 수 있다는 점이 가장 큽니다(컨테이너는 객체 혹은 포인터만 담을 수 있습니다).
    * 난수 발생
        * C++가 C 표준 라이브러리로부터 물려받은 rand 함수보다 몇 배는 우수한 난수 발생 기능입니다.
    * 특수 용도의 수학 함수
        * 라게르(Laguerre) 다항식, 베셀(Bessel) 함수, 완전 타원 적분(complete elliptic integral) 등이며 그 외에도 많이 있습니다.
    * C99 호환성 확장 기능
        * C99의 새로운 라이브러리를 C++로 가져올 목적으로 설계된 함수 및 템플릿 모음
* 나머지 TR1 구성요소는 더 세련된 템플릿 프로그래밍 기법을 지원하는 기술이다.
    * 타입 특성정보(type traits)
        * 주어진 타입에 대한 컴파일 타임 정보를 제공하는 특성정보 클래스(항목 47 참조)의 모음입니다. 어떤 T타입에 대해 TR1의 타입 특성정보 기능을 적용하면, T가 기본제공 타입인지, 가상 소멸자를 지원하는지, 공백 클래스(항목 39 참조)인지, 다른 U타입으로 암시적 변환이 가능한지 등의 정보를 들추어낼 수 있습니다. 심지어, 주어진 타입의 적절한 바이트 정렬까지 잡아내어 줍니다.
    * tr1::result_of
        * 어떤 함수 호출의 반환 타입을 추론해 주는 템플릿입니다. 이런저런 복잡한 사정 때문에 반환 타입은 그 함수의 매개변수 타입에 따라 달라질 수 있습니다. tr1::result_of 템플릿을 쓰면 함수 반환 타입을 쉽게 참조할 수 있습니다.

* 이것만은 잊지 말자
    * 최초에 상정된 표준 C++ 라이브러리의 주요 구성요소는 STL, iostream, 로케일 등입니다. 여기에는 C89의 표준 라이브러리도 포함되어 있습니다.
    * TR1이 도입되면서 추가된 것은 스마트 포인터(tr1::shared_prt 등), 일반화 함수 포인터(tr1::function), 해시 기반 컨테이너, 정규 표현식 그리고 그 외의 10개 구성요소입니다.
    * TR1 자체는 단순히 명세서일 뿐입니다. TR1의 기능을 사용하기 위해서는 명세를 구현한 코드를 구해야 합니다. TR1 구현을 구할 수 있는 자료처 중 한 군데가 바로 부스트입니다.



## 항목 55: Boo子有親! 부스트를 늘 여러분 가까이에
* 이것만은 잊지 말자
    * 부스트는 동료 심사를 거쳐 등록되고 무료로 배포되는 오픈 소스 C++ 라이브러리를 개발하는 모임이자 웹사이트입니다. 또한 C++ 표준화에 있어서 영향력 있는 역할을 맡고 있습니다.
    * 부스트에서 배포되는 라이브러리들 중엔 TR1 구성요소에 들어간 것도 있지만, 그 외에 다른 라이브러리들도 아주 많습니다.


