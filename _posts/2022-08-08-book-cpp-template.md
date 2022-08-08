---
title:  "[book] C++ 템플릿 가이드"
excerpt: "C++ 템플릿 가이드 - 똑똑한 프로그래밍을 위한"

categories:
  - Book
tags:
  - [Book, Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2022-08-08
last_modified_at: 2022-08-08

published: true
---

* 아래책을 기반으로 함
	* http://www.yes24.com/Product/Goods/3199315
* 다시 보진 않을듯하여 새롭게 정리하는 마음으로  


# 1장 들어가며
* 아래의 차이는?
```c++
void foo(const int &x)
void foo(const int x);
void foo(int const &x);
void foo(int const x);
```


# 2장 함수 템플릿
* 함수 템플릿은 함수군을 표현할 수 있게 파라미터화된 함수를 일컫음

## 2.1 함수 템플릿이란
* 다양한 데이터형에 대해 호출될 수 있는 기능적 동작을 제공. 함수 템플릿은 함수군을 대표하는 것

### .1.1 템플릿 정의
* 다음은 두 값중 큰 값을 반환하는 함수 템플릿

```c++
template <typename T>
inline T const& max(T const& a, T const& b)
{
    //a<b이면 b를 사용하고 아니면 a를 사용
    return a < b ? b : a;
}
```

* 이들 파라미터의 데이터형은 템플릿 파라미터 T로 아직 정해지지 않음. 
* 템플릿 파라미터는 다음과 같은 문법을 사용해 명시해야함.

```c++
template < 쉼표로 분리된 파라미터 목록 >
```

* 위 예제에서 데이터형 파라미터는 T다. 파라미터 이름으로 어떤 식별자를 사용해도 좋지만 주로 T를 사용한다. 
* 데이터형 파라미터는 호출자가 함수를 호출할 때 명시하는 임의의 데이터형을 대변하기 위해 사용
* 템플릿이 사용하는 동작을 제공한다면 이 함수의 인자로 어떠한 데이터형(기본형, 클래스 등)이든 사용할 수 있다. 
    * 위의 경우 T라는 데이터 형은 < 연산자를 지원해야 한다.
* 역사적인 이유료 typename 대신 class 사용할 수 있다. 
    * typename 이라는 키워드는 C++ 언어가 만들어진 후 오랜 시간이 지나서 도입되었다. 

```c++
template <class T>
inline T const& max (T const& a, T const& b)
{
    return a < b ? b : a;
}
```

* 의미상 두 함수는 다를게 없다. 
    * 다만 struct 를 typename 대신 사용할 수 없다. 


## 2.1.2 템플릿 사용

* 다음은 max( ) 함수 템플릿의 사용법을 보여준다.

```c++
int main()
{
    int i = 10;
    ::max(7, i);
    double f1 = 2.3;
    double f2 = -2.4;
    ::max(f1, f2);
    std::string a = "abc";
    std::string b = "a";
    ::max(a, b);
}
```

* 일반적으로 템플릿은 어떠한 데이터형이라도 다룰수있는 하나의 실체로 컴파일되지 않는다. 템플릿이 사용될 때마다 각 데이터 형에 맞는 실체가 생성된다. 따라서 위의 max( )는 3가지 형태로 컴파일 된다. 

```c++
inline int const& max(int const& a, int const& b);
inline const double& max(double const& a, double const&);
inline const std::string& max(std::string const&, std::string const&);
```

* 만약 < 연산자를 제공하지 않는 데이터형을 사용하면 컴파일 오류가 발생한다. 

* 이를 위해 템플릿은 2번 컴파일 된다.
    * 템프릿 자체의 문법이 정확한지 검사한다.
    * 인스턴스화 되는 시점에서 모든 호출이 유효한지 템플릿 코드를 검사한다. 지원하지 않는 함수 호출과 같은 잘못된 호출이 발견된다. 

* 함수 템플릿이 인스턴스화를 유발하기 위해 컴파일러가 템플릿정의를 알아야 한다. 간단한 방법으로 템플릿은 헤더 파링레 인라인함수로 구현한다.


## 2.2 인자 추론
* max( ) 와 같은 함수 템플릿을 호출하면 코드가 건넨 인자를 기반으로 템플릿 파라미터가 결정된다. T 형에 int를 넘기면 컴파일러는 T가 int라고 결정짓는데 자동 데이터형 변환은 이루어 지지 않는다. 

```C++
max(4, 7);//ok T는 int로 인식됨
max(4, 2.3);//오류 T는 int 인지 double인지 모름. 
```

* 이런경우 3가지 방법으로 처리할수 있다. 
    * 인자가 일치하도록 형변환을 한다. 
    * T의 데이터 형을 명시힌다. 
        * max< double > (4, 2.3 );
    * 파라미터가 다른 데이터형을 가질 수 있도록 명시한다. 

## 2.3 템플릿 파라미터
* 함수 템플릿은 두 종류의 파라미터를 가진다. 
* 템플릿 파라미터 : 함수 템플릿 이름 앞의 꺾쇠 내에서 선언된 것

```c++
template < typename T> //T는 템플릿 파라미터
```

* 호출 파라미터 : 함수 템플릿 이름 뒤 괄호 안에 선언된 것

```c++
 ... max(T const& a, T const& b)// a 와 b는 호출 파라미터
```

* 템플릿 파라미터는 원하는데로 가질수있지만, 함수 템플릿에서는 기본 템블릿 인자를 명시할 수 없다. 

* 아래처럼 사용 가능하다. 

```c++
template <typename T1, typename T2>
inline T1 max(T1 consot& a, T2 const&b)
{
    return a < b ? b : a;
}
..
max(4 4,2);//ok


template <typename T>
inline T max(T consot& a, T const&b);
..
max<double>(4, 4.2);//T를 double로 인스턴스화 


template <typename T1, typename T2, typename RT>
inline RT max(T1 consot& a, T2 const&b)
//이 경우 RT는 추론될수 없다.  아래처럼은 가능하다. 


template <typename T1, typename T2, typename RT>
inline RT max(T1 consot& a, T2 const&b);
..
max<int, double, double>(4, 4.2);//OK 



template <typename T1, typename T2, typename RT>
inline RT max(T1 consot& a, T2 const&b);
..
max<double>(4, 4.2);//OK 반환형만 명시하고 나머지는 추론하였다. 
```


## 2.4 오버로딩 함수 템플릿

* 일반 함수처럼 함수 템플릿도 오버로딩 할수 있다. 

```c++
inline int const& max(int const& a, int const& b)
{
    return a<b ? b:a;
}

template <typename T>
inline T const& max(T const&a, T const& b)
{
    return a<b ? b:a;
}

template <typename T>
inline T const& max(T const& a, T const& b, T const& c)
{
    return ::max(::max(a, b), c);
}

int main()
{
    ::max(7, 42, 54);//세 인자를 위한 템플릿 호출
    ::max(7.0, 3.0);//max<double> 호출(인자추론 이용)
    ::max(7,2);//템플릿 이용하지 않음
    ::max<>(7,2);//max<int> 호출(인자추론 이용)
    ::max<double>(7.4,3.2);//max<double> 호출(인자추론 이용하지 않음.)
}
```

 

# 3장 클래스 템플릿
* 함수처럼 클래스도 하나 이상의 데이터형으로 파라미터화될 수 있다. 
## 3.1 클래스 템플릿 Stack의 구현
* 함수 템플릿에서처럼 헤더 파일에 클래스 Stack < > 을 다음과 같이 선언하고 정의한다.

```c++
template <typename T>
class Stack{
    private:
        std::vector<T> elems;
    public:
        void push(T const&);
        void pop();
        T top() const;
        bool empty() const{ return elems.empty();}
};
template <typename T>
void Stack<T>::push(T const& elem)
{
    elems.push_back(elem);
}
...
```

* 이 클래스의 데이터 형은 Stack < > 이고 T는 템플릿 파라미터이다. 
* 선언 내에서 이 클래스의 데이터 형을 써야 한다면 Stack < T > 로 표기해야한다. 

### 3.1.1 클래스 템플릿의 선언
* 클래스 템플릿을 선언하는 방식은 함수 템플릿의 선언과 유사하다. 클래스 템플릿을 선언하기 전에 먼저 데이터형 파라미터로 사용할 식별자를 선언해야한다. 
    * 여기서도 typename 대신에 class를 사용할 수 있다. 

```c++
template <typename T>
class Stack{
    ...
};
```

* 클래스 템플릿 내부에도 다른 일반적인 데이터형처럼 T 도 멤버와 멤버 함수선언할때 사용할 수 있다. 
* 예를 들어 복사 생성자와 할당 연산자를 선언해야 한다면 아래처럼 될 것이다.

```c++
template <typename T>
class Stack{
    Stack(Stack<T> const&);//복사 생성자
    Stack<T>& operator= (Stack<T> const&);//할당 연산자.
};
```


### 3.1.2 멤버 함수의 구현
* 클래스 템플릿의 멤버 함수를 정의하기 위해선 이것이 함수 템플릿이라는것을 명시해야하며, 클래스 템플릿의 전체 데이터형 한정자를 사용해야 한다. 따라서 Stack < T >형의 push( ) 멤버 함수의 구현은 다음과 같다.

```c++
template <typename T>
void Stack<T>::push(T const& elem)
{
    elems.push_back(elem);
}
```

* elems 가 비었을 경우등 예외가 발생하는 케이스는 굳이 다루지 않았다. 

## 3.2 클래스 템플릿 Stack의 사용
* 클래스 템플릿의 객체를 사용하기 위해서는 템플릿 인자를 꼭 명시해야 한다. 다음 예제를 보자.

```c++
int main()
{
    Stack<int> intStack;
    intStack.push(5);
    Stack<std::string> stringStack;
    stringStack.push("hello");
}
```

* Stack < int > 형을 선언했으므로 클래스 템플릿내에서는 int 가 T라는 데이터형 대신 사용된다. 호출된 모든 멤버 함수에 대한 코드가 인스턴스화 된다. 
    * 호출된 멤버 함수에 대한 코드만 인스턴스화 된다는 점을 기억하자. 
    * 클래스 템플릿에서 멤버 함수는 사용될 때에만 인스턴스화 된다. 

## 3.3 클래스 템플릿의 특수화
* 클래스 템플릿을 특정 템플릿 인자로 특수화할 수 있다. 함수 템플릿의 오버로딩처럼 클래스 템플릿을 특수화 하면 특정 데이터형에 맞게 구현함으로 특정 데이터형에서 클래스 템플릿이 인스턴스화 됐을때 잘못 동작할 수 있는 부분을 수정할 수 있다. 
    * 그러나 클래스 템플릿을 특수화 하려면 모든 멤버 함수를 특수화 해야 한다. 
* 클래스 템플릿을 특후활 할 때에는 template < > 를 먼저 쓰고 클래스를 선언 후 그 뒤에 어떤 데이터형으로 클래스 템플릿을 특수화 할 것인지 명시한다.

```c++
template<>
class Stack<std::string>{
...
};
```

* 특수화할 경우, 멤버 함수의 모든 정의는 일반 멤버 함수처럼 정의하고 T 대신 특수화된 데이터형을 사용해야 한다.

```c++
void Stack<std::string>::push (std::string const& elem)
{
    elems.push_back(elem);
}
```


## 3.4 부분 특수화
* 클래스 템플릿은 부분적으로 특수화될 수 있다. 이를 통해 특정 환경에서만 필요한 특별한 구현을 명시할 수 있지만, 일부 템플릿 파라미터는 여전히 사용자가 지정해야할 것이다. 다음의 예시를 보자.

```c++
template <typename T1, typename T2>
class MyClass{
...
};
//다음과 같이 부분 특수화가 가능하다. 
//부분 특수화 : 두 템플릿 파라미터가 같은 데이터 형을 가짐
template <typename T>
class MyClass<T, T>{
    ...
};
//부분 특수화 : 두 번째 데이터형이 int 임
template <typename T>
class MyClass<T, int>{
    ...
};
//부분 특수화 : 두 템플릿 파라미터가 포인터 형임
template <typename T1, typename T2>
class MyClass<T1*, T2*>{
    ...
};

//다음 예제는 각 선언에 의해 사용된 템플릿이 무엇인지 알려준다.
MyClass<int, float> mif;//Myclass<T1, T2>사용
MyClass<float, float> mif;//Myclass<T, T>사용
MyClass<float, int> mif;//Myclass<T, int>사용
MyClass<int*, float*> mif;//Myclass<T1*, T2*>사용

//다만 모호해지면 오류가 발생한다.
MyClass<int, int> m;//오류 Myclass<T, T>와 Myclass<T, int>에 일치

```

## 3.5 기본 템플릿 인자
* 클래스 템플릿에서는 템플릿 파라미터의 기본값을 지정할 수 있다. 이런 값은 기본 템플릿 인자라고 부른다. 
* 이들은 이전 템플릿 파라미터를 참조할 수도 있다. 
* 아래예시처럼 std::vector < >를 기본값으로 정할수 있다. 

```c++
template <typename T, typename CONT = std::vector<T> >
class Stack{
    private:
        CONT elems;
    public:
        void push(T const&);
    ...
};

template <typename T, typename CONT>
void Stack<T, CONT>::push (T const& elem)
{
    elems.push_back(elem);
}
```

 

# 4장 데이터형이 아닌 템플릿 매개변수

## 4.1 데이터형이 아닌 클래스 템플릿 매개변수
* 3장의 스택 구현 방식과 달리 크기가 고정된 배열을 사용하여 스택을 구현할 수도 있다. 이때 스택을 만들때 필요한 최대 크기를 코드가 명시하게 할 수도 있다. 이를 위해 크기를 템플릿 파라미터로 정의하자. 

```c++
template <typename T, int MAXSIZE>
class Stack{
private:
    T elems[MAXSIZE];
    int numElems;
public:
    STack();//생성자.
    void push(T const&);
    void pop();
    ...
};

template <typename T, int MAXSIZE>
Stack<T, MAXSIZE>::Stack() : numElems(0)
{
}

template <typename T, int MAXSIZE>
void Stack<T, MAXSIZE>::push(T const& elem)
{
    if(numElems == MAXSIZE){
        throw std::out_of_range();
    }
    ...
}

template <typename T, int MAXSIZE>
void Stack<T, MAXSIZE>::pop()
{
    --numElems;
}

//위의 클래스 템플릿을 사용하려면 요소의 데이터형과 함께 최대 크기도 명시해야 한다. 
int main()
{
    Stack<int , 20> int20Stack;
    Stack<int , 40> int40Stack;    
    Stack<std::string, 40> stringStack;
}
//각 템플릿 인스턴스는 자신만의 데이터형을 갖는다. int20Stack 와 int40Stack 는 각자 다른 데이터형이르모 둘 사이의 묵시적이거나 명시적 데이터형 변환은 일어날수 없다. 
```

## 4.2 데이터형이 아닌 함수 템플릿 매개변수
* 함수 템플릿을 위해서도 데이터형이 아닌 템플릿 파라미터를 정의할 수 있다. 

```c++
//특정 값을 추가하는 함수군
template<typename T, int VAL>
T addValue(T const& x)
{
    return x + VAL
}
//이런 종류의 함수는 함수나 연산이 파라미터로 사용될때 유용하다. 
```

## 4.3 데이터형이 아닌 템플릿 매개변수에 대한 제약
* 데이터형이 아닌 템플릿 파라미터에는 몇가지 제약사항이 있다. 
    * 일반적으로 이들은 정수 상수형 값이거나 외부 링크를 가진 객체에 대한 포인터야 한다. 
    * 부동소수점 숫자나 클래스형 객체는 데이터형이 아닌 템플릿 파라미터로 사용될 수 없다. 

```c++

template <double VAT>//오류 : 부동수수점은 템플릿 파라미터로 사용될 수 없음
double process (double v)
{
	return v * VAT;

template <std::string name> // 오류 : 클래스 객체는 템플릿 파라미티로 사용될 수 없음
}
```

* 부동소수점을 템플릿 인자로 사용하지 못하는 데이틑 역사적인 이유가 있다. 문자열은 내부 링크를 가지는 객체(다른 모듈에 있지만 같은 값을 가지는 두 문자열은 다른 객체이다. )이기 때문에 이들은 템플릿 인자로 사용할 수 없다. 

```c++
template <char consot* name>
class MyClass {
...
};

MyClass<"hello"> x;//오류 문자열 리터럴은 허용되지 않음
```

* 전역 포인터도 템플릿 파라미터로 사용할 수 없다. 

```c++
template <char consot* name>
class MyClass {
...
};

char const* s = "hello";

MyClass<s> x;//오류 s는 내부 링크를 갖는 객체에 대한 포인터임
```

* 하지만 다음처럼 사용은 가능하다. 

```c++
template <char consot* name>
class MyClass {
...
};

extern char const s [] = "hello";

MyClass<s> x;//ok 
//전역 문자 배열인 s는 "hello"로 초기화 됐으므로 s는 외부 링크를 가지는 객체이다. 
```



# 5장 복잡한 기본

## 5.1 키워드 typename
* 키키워워드인  typename 은 템플릿 내에서 식별자가 데이터형이라는 것을 명확히 하기 위해 도입하였다. 

```c++
template <typename T>
class MyClass{
    typename T::SubType *ptr;
    ...
};
```

* 위의 예시에서 SubType 이 클래스 T 내에서 정의된 데이터 형이라는 것을 확실히 하기위해 두번째 typename 이 사용되었다. 
    * typename이 없다면 SubType는 정적 멤버로 간주될수 있다. 
        * 그러면 결국 T::SubType * ptr 은 클래스 T 으 ㅣ정적 멤버인 SubType을 ptr과 곱하는 연산이 된다. 


## 5.2 this-> 사용
* 기본 클래스가 있는 클래스 템플릿에서 기본 클래스로부터 상속받은 X 가 있다해서 X 라는 이름이 항상 this->X 를 의미자힌 않는다.
    * 아래 예제에서 foo 내의 bar를 해석할때  Base 에서 정의된 bar() 는 절대로 고려되지 않는다. 
    * 기본 클래스에서 선언된 기호를 사용할 때에는 this-> 나 Base < T >:: 를 붙여 한정하도록 하자. 

```c++
template <typename T>
class Base{
publid:
    void bar();
};

template <typename T>
class Derived : Base<T>{
public:
    void foo(){
        bar();//외부 bar()가 호출되거나 오류 발생. 
    }
};
```


## 5.3 멤버 템플릿
* 클래스 멤버도 템플릿이 될 수 있다. 중첩된 클래스나 멤버 함수도 템플릿이 될수 있다. 
* 기존 예시에서 스택에 여러 데이터 형을 가지는 요소를 쌓을 수는 없었지만, 템플릿으로 할당 연산자를 정의하면 스텍에 데이터형 번환이 정의된 요소들을 정의할 수 있다. 아래의 예시를 보자. 

```c++
template <typename T>
class Stack{
private:
    std::deque<T> elems;
public:
    ...
    template <typename T2>
    Stack<T>& operator= (Stack<T2> const&);
};

//새로운 할당 연산자의 구현코드는 아래와 같다. 

template <typename T>
tmeplate <typename T2>
Stack<T>& Stack<T>::operator= (STack<T2> const& op2)
{
    if((void*)this == (void*)&op2){//자기 자신에게 할당?
        return *this;
    }
    Stack<T2> tmp(op2);
    ...
    return *this;
}
```

* 멤버 템플릿을 정의하는 문법을 보면
    * 템플릿 파라미터 T 를 사용하는 템플릿 안에 템플릿 파라미터 T2를 쓰는 내부 템플릿이 정의됐다.

```c++
template <typename T>
tmeplate <typename T2>
...
```

* 멤버 함수 내에서는 할당된 스택 op2에서 필요한 모든 데이터에 접근할 수있을것이라 기대하겠지만, 이 스택은 다른 데이터형을 가지므로(각기 다른 두 개의 데이터형으로 클래스 템플릿을 인스턴스화 했다면 다른 데이터형을 갖는 두 개의 템플릿을 가지게 된다. ) 공개 인스턴스만을 사용해야 한다. 
    * 즉 요소에 접근하려면 top ( ) 을 써야한다는 것이다. 

## 5.4 템플릿 템플릿 파라미터
* 템플릿 파라미터 자체가 클래스 템플릿일 수 있다면 좋을 것인데, 또다시 스택 클래스 템필릿을 예제로 알아본다. 
    * 스택에서 다양한 내부 컨테이너를 쓸 수 있게 하기 위해서 요소의 데이터형을 두 번씩 명시해줘야 한다. 
    * 내부 컨테이너형을 명시하기 위해서는 컨테이너형과 함께 요소의 데이터형도 다시 넘겨줘야 한다.

```c++
Stack<int, std::vector<int>> vStack;//vector를 사용하는 정수형 스택

//템플릿 템플릿 파라미터를 사용한다면 요소의 데이터형에 대해 다시 명시해주지 않아도 컨테이너형을 명시할 수 있는 Stack 클래스 템플릿을 선언할 수 있따.
Stack<int, std::vector> vStack;//vector를 사용하는 정수형 스택

//이를 위해 두번째 템플릿 파라미터를 템플릿 템플릿 파라미터로 명시해야 한다.
template <typename T>,
    template<typename ELEM> class CONT = std::dequq>
class Stack{
    ...
};
//이전과 달리 두 번째 템플릿 파라미터가 클래스 템플릿으로 선언됐다. 
template <typename ELEM> class CONT
```


## 5.5 0 초기화

* int, double 이나 포인터형과 같은 기본형에서는 이들을 기본값으로 초기화하는 생성자가 없다. 
    * 지역 변수는 초기화되지 전까지는 정해지지 않는 어떤 값을 가진다. 
    * 기본값으로 초기화되는 템플릿형의 변수를 원할 때 다음과 같은 간단한 정의로는 기본 데이터형을 초기화 할수 없다. 
        * 이와 같은 문제를 해결하기 위해 내장 데이터형에 대해서는 0(bool 형은 false)으로 초기화하는 기본 생성자를 명시적으로 호출할 수 있다. 
        * 즉 int ( ) 는 0을 반환한다. 

```c++
void foo()
{
    int x;//x는 정의되지 않음.
}

template <typename T>
void foo()
{
    T x;//T가 내장 데이터형일 경우 x는 정의되지 않는 값을 가진다. 
    ...
    T x = T()//x는 T가 내장 데이터형일 경우 0(혹은 거짓)을 갖는다. 
}
```

## 5.6 함수 템플릿에 문자열 리터럴을 인자로 사용

* 함수 템플릿의 참조자 파라미터로 문자열을 전달하면 가끔 오동작을 하기도 한다. 

```c++
//주의 참조자 파라미터
template <typename T>
inline T const& max(T const& a, T const& b)
{
	return a < b ? b : a;
}

int main()
{
	std::string s;

	::max("apple", "peach"); //ok 같은 데이터형
	::max("apple", "tomato");// ERROR 다른 데이터형
	::max("apple", s);       // ERROR 다른 데이터형
}
```

* 문자열 리터럴의 길이에 따라 각기 다른 배열형으로 정의된다는 것이 문제이다. 
	* apple은 char const[6] 라는 데이터형을 가지는데 tomato는 char const[7] 이다. 
* 참조자가 아닌 파라미터를 선언한다면 다른 크기를 가지는 문자열 리터럴도 사용할 수 있다.

```c++
template <typename T>
inline T max(T a, T b)
{
	return a < b ? b : a;
}
```

* 위의 예제에서는 문자열에 대해 max()를 오버로딩하는 것이 가장 좋다.

 


# 6장 템플릿 실제 사용

## 6.1 포함 모델
* 템플릿 소스코드를 구성하는 방식은 여러가지이지만, 흔히 사용되는 포함 모델에 대해서 알아 보겠다. 

### 6.1.1 링커 오류
* 대부분, C,C++개발자는 템플릿이 아닌 코드는 아래처럼 관리한다.
    * 클래스와 그외 데이터형은 모두 헤어파일에, 일반적으로 헤더파일은 .hpp, .h, .H, .hh, .hxx등의 확장자를 가진다.
    * 전역변수와 함수의 경우 헤더파일에 선언만 두고, 정의는 C파일이라 불리는 파일에 둔다. 일반적으로 이 파일은 .cpp, .c, .C, .cc, .cxx 확장자를 가진다.
* 이와같이 선언/정의 분리를 통해 프로그램 전체에서 원하는 데이터형 선언을 쉽게 얻을 수 있고 링크 과정에서 변수나 함수에 대한 중첩된 정의 오류를 발생하지 않게 된다. 
* 이와 같은 관례로 템플릿을 헤더 파일에 두고 사용할 경우 오류가 발생한다. 
    * 템플릿이 인스턴스화 되려면 컴파일러는 어떤 정의가 어떤 템플릿 인자로 인스턴스화 돼야하는지 알아야하는데, 템플릿 정보가 헤더파일로 떨어져 있으면 각기 따로 컴파일 되었다. 

### 6.1.2 헤더 파일에 있는 템플릿
* 6.1.1의 문제에 대한 일반적인 해결책은 매크로나 인라인 함수에서 쓰는 방법과 동일하다.
    * 템플릿의 정의를 해당 템플릿의 선언인 들어있는 헤더 파일에 포함시키는 것이다.

* 예를 들어 #include "test.cpp" 를 test.hpp 의 뒤에 추가하거나 템플릿을 사용하는 모든 C 파일에 test.cpp를 추가해 문제를 해결 할 수 있다.
* 또 다른 방법으로 test.cpp를 아예 없애고 test.hpp를 새로 작성해 모든 템플릿 선언과 템플릿이 헤더에 포함되게 할 수 있다. 

```c++
//test.hpp

#include <iostream>

//템플릿 선언
template <typename T>
void print(T const&);

//탬플릿 구현/정의
template <typename T>
void print(T const& x)
{
    ...
}
```

* 이런 방식으로 템플릿을 구성하는 것을 포함 모델이라고 한다. 
* 문제를 해결할 수 있지만 test.hpp라는 헤더 파일을 포함하는데 드는 비용을 상당히 증가 시킨다. 
    * 빌드 시간 문제등이 있지만 포함 모델로 템플릿을 구성하는게 좋다. 다른 방법은 더 문제가 많다. 


## 6.2 명시적 인스턴스화

### 6.2.1 명시적 인스턴스화의 예제
* 면시적 인스턴스화 지시자는 키워드 template 다음에 인스턴스화 하고 싶은 실체에 대해 파라미터를 완전히 치환한 선인이 뒤따르는 형태로 구성된다. 
    * 멤버 함수나 정적 데이터 멤버에서도 동일하게 적용될 수 있다. 

            
```c++
#include "test.cpp"
//double 에 대한 print () 를 명시적으로 인스턴스화
template void print<double>(double const&);

//int 에 대한 myClass<>의 생성자를 명시적으로 인스턴스화
template MyClass<int>::MyClass();
```

* 몇가지 장점이 있긴 하지만 수동 인스턴스는 좋지 않다.

### 6.2.2 포함 모델과 명시적 인스턴스화 결합
* 템플릿의 선언과 정의를 두 파일에 따로 둘 수 있다. 
    * 흔희 두 파일 모두 헤더 파일처럼 이름을 짓는다. 

## 6.3 분리 모델

### 6.3.1 키워드 export
* export 기능을 사용하려면 아래의 예시처럼 템플릿을 정의하고 해당 정의와 모든 정의되지 않는 선언들을 export 키워드를 사용해 표시하면 된다. 

```c++
export
template <typename T>
void print(T const&);
```

* export 키워드는 함수 템플릿, 클래스 템플릿의 멤버 변수, 멤버 함수 템플릿과 클래스 템플릿의 정적 데이터 멤버, 클래스 템플릿 선언에 적용된다. 
    * export 키워드는 인라인과 함께 사용되지 못하며 template 앞에 와야하는것을 주의하자. 

### 6.3.2 분리 모델의 한계
* 템플릿을 내보내리고 전달하는 것이 잘 동작하는 좋은 방법이지만 포함 모델 방식이 더 좋긴하다. 

### 6.3.3 분리 모델에 대한 대비
* 포함 모델과 내보내기 모델 사이을 쉽게 왔다갔다할 수 있는 예제 코드이다.

```c++
#if defined(USE_EXPORT)
#define EXPORT export
#else
#define EXPORT
#endif

//템플릿 선언
EXPORT
template <typename T>
void print(T const&);

//USE_EXPORT 가 정의되지 않았다면 정의를 불러들임
#if !defined(USE_EXPORT)
#include "test.cpp"
#endif
```

## 6.4 템플릿과 인라인
* 실행 시간 개선을위해 짧은 함수를 보통 인라인으로 만들기는 한다. 함수 템플릿이 인라인이라는 인상을 받을 수 있지만 템플릿 함수는 인라인이 아니다. 
    * 인라인으로 취급돼야 하는 함수 템플릿을 작성한다면 인라인 지정자를 사용해야 한다. 

## 6.5 전컴파일된 헤더
* 표준 밖에서 정의된 전처리된 헤더라는 기법의 동작 방식은 아래와 같다. 
    * 컴파일러가 파일 내 코드를 번역할 때 컴파일러는 파일의 처음부터 끝까지 읽어 각 토큰을 처리함녀서 룩업할수 있도록 기호 테이블에 항목을 추가하는 등의 내부 상태를 적응시킨다. 이 과정에서 컴파일러는 목적 파일을 생성한다. 
    * 전처리된 헤더 기법은 N 행을 컴파일 한 후 저장하고 해당 값을 재활용 하는 기법이다. 

## 6.6 디버깅 템플릿
* 이부분은 굳이
 

# 7장 기본 템플릿 용어

## 7.1 "클래스 템플릿"과 "템플릿 클래스"
* C++에서는 구조체, 클래스와 공용체를 묶어서 클래스형이라고 부른다. 
    * 클래스 템플릿은 템플릿인 클래스를 말한다. 즉 일군의 클래스를 파라미터화해 설명하는 것을 지칭힌다. 
    * 반면, 템플릿 클래스는 다음 상황에서 사용된다. 
        * 클래스 템플릿의 동의어
        * 템플릿으로 생성된 클래스를 가리킬 때
        * 템플릿 식별자인 이름을 가지는 클래스를 가리킬 때 
* C++에서 class 의 기본 접근 방식이 privatd인데 반해 struct는 publid 점이 다르긴하다. 

## 7.2 인스턴스화와 특수화
* 템플릿에서 인자를 실제 값으로 치환해 템플릿을 일반 클래스, 함수나 멤버 함수로 생성하는 과정을 템플릿 인스턴스화라고 부른다. 그 결과 생성된 실체는 일반적으로 특수화라고 부른다. 
* C++에서 인스턴스화만으로 특수화할 수 있는 것은 아니고 템플릿 파라미터를 특별히 치환해 명확히 특수화를 선언하는 방법도 있다. 
    * 명시적 특수화, 부분 특수화

```c++
template <typename T1, typename T2>//원래 클래스 템플릿
class MyClass{
...
};

template<>//명시적 특수화
class MyClass<std::string, float>{
...
};

template <typename T>//부분 특수화
class MyClass<bool, T>{
...
};
```

## 7.3 선언과 정의
* 선언(declaration)과 정의(definition)
* 선언
    * C++가 그 이름을 C++ 영역에 도입하거나 재도입한다는 뜻이다. 
    * 도입시에는 ㅎ아상 그 이름으로 부분분류가 포함되지만 꼭 필요하진 않다. 
    * 매크로 정의와 goto 라벨은 이름을 갖긴 하지만 C++에선 선언으로 간주되지 않는다. 

```c++
class C;//C를 클래스로 선언
void f(int p);//f() 를 p 라를 파라미터를 갖는 함수로 선언
extern int v;//v 를 변수로 선언 
```

* 정의
    * 구조의 세부사항이 알려지거나 변수라면 저장 공간이 할당될 때에 선언은 정의로 바뀐다고 한다. 
    * 클래스형과 함수 정의의 경우에는 중괄호로 둘러싸인 몸체가 제공돼야 한다. 
    * 변수의 경우 초기화나 extern 이 없는 선언이 정의의다. 

```c++
class C{};//C를 클래스로 정의(그리고 선언)
void f(int p){...};//f() 를 정의(그리고 선언)
extern int v=1;//초기화가 있기 때문에 v 를 위한 정의임
int w;//extern이 앞에 붙지 않는 전역변수 선언또한 저으이임 
```

## 7.4 단정의 법칙
* none

## 7.5 템플릿 인자와 템플릿 매개변수
* none 




# 2부 템플릿 상세 설명
 

# 8장 템플릿 기초 원리의 깊은 이해

## 8.1 파라미터화된 선언
* C++ 현재 클래스 템플릿과 함수 템플릿이라는 두 종류의 템플릿을 지원한다. 
* 두 종류의 템플릿 예제를 봐보자, 둘다 클래스 멤버이자 일반적인 네임스페이스 영역 선언이다.

```c++
template <typename T>
class List{//네임스페이스 영역 클래스 템플릿
public:
    template <typename T2>//멤버 함수 템플릿
    List (List<T2> const&);//생성자
    ...
};
template <typename T>
template <typename T2>
List<T>::List (List<T2> const& b)//클래스 밖 멤버 함수 템플릿 정의
{
    ...
}

template <typename T>
int lingth(List<T> const&);//네임스페이스 함수 템플릿

class Colection{
    template <typename T>//클래스내 멤버 클래스 템플릿
    class Node {//정의
        ...
    };
    template <typename T>//정의가 없는 다른 멤버 클래스 템플릿
    class Handle;
    
    template <typename T>//클래스내 멤버함수 템플릿
    T* alloc(){...}//정의
    ...
};

template <typename T>//클래스 밖 멤버 클래스 템플릿 정의
class Collection::Handle{
    ...
};
```

* 공용체 템플릿도 선언할 수 있다.(클래스 템플릿의 일종으로 간주한다.)

```c++
template <typename T>
union AllocChunk{
    T object;
    unsigned char bytes[sizeof(T)];
};
```

* 함수 템플릿은 다른 일반 함수 선언처럼 기본 호출 인자를 가질 수 있다.

 ```c++
 template <typename T>
 void report_top(Stack<T> const&, int number = 10);
 
 template <typename T>
 void fill(Array<T>*, T const& = T());//내장 데이터형에 대해서 T() 는 0
 ```

* 기본적인 두 종류의 템플릿과 함께 기본 템플릿 선언과 유사한 방법으로 세 종류의 선언 방식도 파라미터화될 수 있다.클래스 템플릿 멤버의 정의에도 다음 세가지 모두가 적용된다.
    * 클래스 템플릿의 멤버 함수 정의
    * 클래스 템플릿에 내포된 클래스 멤버 정의
    * 클래스 템플릿의 정적 데이터 멤버 정의


### 8.1.1 가상 멤버 함수
* 멤버 함수 템플릿은 virtual 로 선언될 수 없다. 
* 가상 함수 호출 메커니즘은 대체로 가상함수 하나당 하나의 엔트리를 갖는 고정 크기의 테이블을 쓰는 방식으로 구현되는데, 멤버 함수 템플릿은 전체 프로그램이 번역되는 동안 무한히 인스턴스화될 수 있기 때문이다. 
* 이와 대조적으로 클래스 템플릿의 일반 멤버는 클래스가 인스턴스화될 때 그 수가 고정되기 때문에 virtual로 선언될 수 있다.

```c++
template <typename T>
class Dynamic{
public:
    virtual ~Dynamic();//OK: Dynamic<T> 의 인스턴스 하나당 하나의 소멸자
    template <typename T2>
    virtual void copy(T2 const&);//오류 : Dynamic<T>의 인스턴스 내에서 copy() 인스턴스의 수를 결정할수 없다.
};
```

### 8.1.2 템플릿의 링크
* 모든 템플릿은 이름을 가지며 그이름은 해당 영역 내에서 고유해야 한다. 클래스 형과 달리 클래스 템플릿은 다른 종류의 실체가 가진 이름을 공유할 수 없다.

```c++
int C;
class C;//OK : 클래스 이름과 클래스가 아닌 이름은 다른 공간에 있음

int X;
template <typename T>
class X;//오류 : 변수 X와 충돌

struct S;
template <typename T>
class S;//오류 : struct S와 충돌
```

* 템플릿 이름은 링크를 가지지만 C 링크는 가질 수 없다. 

```c++
extern "C++" template <typename T>
void normal();//이것이 기본

extern "C" template <typename T>
void invalid();//오류 템플릿은 C 링크를 가질 수 없음

```

* 템플릿은 일반적으로 외부 링크를 가진다. static 지정자를 가지는 네임스페이스 영역 함수 템플릿만은 예외이다. 

### 8.1.3 기본 템플릿
* 템플릿에 대한 일반적인 선언은 기본 템플릿 이라고 부른다. 
    * 이 템플릿 선언에서 템플릿 뒤에는 꺾쇠로 둘러싸인 템플릿 인자가 따라오지 않는다.

```c++
template<typename T> class Box; //OK 기본 템플릿
template<typename T> class Box<T>;//오류
template<typename T> void translate(T*);//OK 기본 템플릿
template<typename T> void translate<T>(T*);//오류
```

## 8.2 템플릿 매개변수
* 템플릿 파라미터는 3종류가 있다.
    * 데이터형 파라미터
    * 데이터형이 아닌 파라미터
    * 템플릿 템플릿 파라미터


### 8.2.1 데이터형 매개변수
* 데이터형 파라미터는 키워드인 typename 이나 class로 도입
* 키워드 뒤에는 반드시 간단한 식별자가 따라 나와야함.
* 그 식별자 뒤에는 반드시 다음 파라미터 선언의 시작을 알리기 위한 쉼표나 파라미터화절의 끝을 알리기 위한 닫기 꺾쇠, 혹은 기본 템플릿 인자의 시작을 알리기 위한 등호가 나와야함.
* 템플릿 선언 내에서 데이터형 파라미터는 마치 typedef name과 유사한 방식으로 동작.

### 8.2.2 데이터형이 아닌 매개변수
* 데이터형이 아닌 템플릿 파라미터는 컴파일러시나 링크시에 결정되는 상수 값을 말한다. 다음중 하나여야 한다.
    * 상수형이나 열거형
    * 포인터형
    * 참조자형

```c++
template<typename T,        //데이터형 파라미터
typename T::Allocator* Allocator>//데이터형이 아닌 파라미터
class List;
```

* 데이터형이 아닌 파라미터는 변수와 비슷한 방식으로 선언되지만, static, mutual 같은 지시자를 가질 수 없다. 
* const와 volatile 같은 한정자를 가질 수 있지만 한정자가 파라미터형의 맨 뒤에 있다면 무시한다.

```c++
template<int const length> class Buffer;//const 무시
template<int length> class Buffer;//이전 선언과 동일
```


### 8.2.3 템플릿 템플릿 매개변수
* 템플릿 템플릿 파라미터는 클래스 템플릿을 갖기 위한 일종의 자리표시자(placeholder)이다.
    * 선언은 클래스 템플릿과 유사하지만 struct와 union 같은 키워드는 사용할 수 없다. 

```c++
template <temp0late<typename X> class C>//OK
void f(C<int>* p);

template <temp0late<typename X> struct C>//ERROR
void f(C<int>* p);

template <temp0late<typename X> union C>//ERROR
void f(C<int>* p);
```

* 템플릿 템플릿 파라미터는 기본 템플릿 인자를 가질 수 있다. 

```c++
template <template<typename T,
                    typename A = MyAllocator> class Container>
class Adaption{}
    Container<int> storage;//묵시적으로 Container<int, MyAllocator>와 동일
;                    
```

* 템플릿 템플릿 파라미터의 템플릿 파라미터의 이름은 그 템플릿 템플릿 파라미터의 다른 파라미터의 선언에만 사용된다.

```c++
template <template<typename T, T*> class Buf>
class Lexer{
    static char storage[5];
    Buf<char, &Lexer<Buf>::storage[0]> buf;
    ...
};

template<template<typename T> class List>
class Node{
    static T* storage;//ERROR 템플릿 템플릿 파라미터의 파라미터는 여기서 사용될 수 없음. 
};
```

### 8.2.4 기본 템플릿 인자
* 현재는 클래스 템플릿 선언만이 기본 템플릿 인자를 가질수 있다.
* 연속된 기본값은 하나의 템플릿 선언 내에서 제공되곤 하지만 해당 템플릿의 이전 선언에서 제공될 수도 있다. 

```c++
template <typename T1, Typename T2, typename T3, typename T4 = char, typename T5 = char>
class Q;//OK

template <typename T1, Typename T2, typename T3 = char, typename T4 - char, typename T5 char>
class Q;//OK , T4와 T5는 이미 기본 값을 가지고 있다.

template <typename T1 = char, Typename T2, typename T3, typename T4, typename T5>
class Q;//오류 T1은 기존 인자를 가질 수 없다.
```

* 기본 템플릿 인자는 반복될 수 없다.

```c++
template <typename T = void>
class Q;//OK

template <typename T = void>
class Q;//오류 : 반복된 기본 인자.
```

## 8.3 템플릿 인자
* 템플릿 인자는 템플릿이 인스턴스화 될때 템플릿 파라미터를 치환할 값을 말한다. 
    * 명시적 템플릿 인자 : 꺾쇠 사이에 템플릿 인자 값을 두어 템플릿 이름을 제시할 수 있다. 그 결과로 만들어진 이름은 템플릿 식별자라고 한다.
    * 삽입된 클래스 이름 : 템플릿 파라미터인 P1, P2, ... 를 가지는 클래스 템플릿의 X 의 범위 내에서 해당 템플릿(X)의 이름은 템플릿 식별자 X<P1, P2...>와 동일하다
    * 기본 템플릿 인자 : 기본 템플릿 인자를 쓸 수 있다면 클래스 템플릿 인스턴스에서 템플릿 인자를 생략할 수 있다. 
    * 인자 추론 : 명시적으로 제시되지 않는 함수 템플릿 인자는 호출 시 함수 호출 인자의 데이터형을 통해 추론된다. 모든 템플릿 인자가 추론될 수 있다면 함수 템플릿의 이름 뒤에 꺾쇠를 붙이지 않아도 된다. 

### 8.3.1 함수 템플릿 인자
* 함수 템플릿을 위한 템플릿 인자는 명시되거나 템플릿이 사용된 방식에 따라 추론된다. 

```c++
template <typename T>
inlline T const& max(T const& a, T const& b)
{
    return a < b ? b : a; 
}
int main()
{
    max<double>(1.0, 3.0);//명시적 템플릿 인자
    max(1.0, 3.0);//템플릿 인자를 묵시적으로 double로 추론
    max<int>(1.0, 3.0);//명시적인 int 표기, 추론하지 않게만듬 int형이 됨.
}
```

### 8.3.2 데이터형 인자
* 템플릿형 인자는 템플릿형 파라미터를 위해 명시된 값이다. 대부분의 데이터형은 템플릿 인자로 사용되룻 있지만 예외가 2가지 있다.
    * 지역 클래스와 열거형은 템플릿형 인자가 될 수 없다.
    * 이름 붙여지지 않은 클래스형 이나 이름 붙여지지 않은 열거형에 속하는 데이터형은 템플릿형 인자가 될 수 없다. 

### 8.3.3 데이터형이 아닌 인자
* 데이터형이 아닌 템플릿 인자는 데이터형이 아닌 파라미터를 치환하는 값이다. 
    * 데이터형이 아닌 템플릿 파라미터로 올바른 데이터형을 가질 경우
    * 컴파일시 결정되는 정수형 상수값. 
    * 외부 변수나 함수의 이름앞에 내장 & 연산자가 붙은 경우
    * 멤버 접근 포인터 상수


### 8.3.4 템플릿 템플릿 인자
* 템플릿 템플릿 인자는 자신이 치환할 템플릿 템플릿 파라미터의 파라미터와 정확히 일치하는 파라미터를 가진 클래스 템플릿이어야 한다. 템플릿 템플릿 인자의 기본 템플릿 인자는 무시된다. 
* 문법적으로 class 라는 키워드만 템플릿 템플릿 파라미터를 선언할 수 있긴 하지만 class 키워드로 선언된 클래스 템플릿만이 인자가 될 수 있다는 뜻으로 해석해선 안된다. 
    * 구조체 템플릿과 공용체 템플릿 도 모두 템플릿 템플릿 파라미터를 위한 유효한 인자가 될 수 있다. 

### 8.3.5 동일
* 템플릿 인자의 두 집합은 인자와 값들이 1:1로 같을 때 동일하다고 본다.
* 데이터형 인자의 경우 typedef 이름은 중요치 않다. typedef 가 가리키는 진짜 데이터형이 중요하다. 
* 함수 템플릿에서 생성된 함수는 이들이 동일한 데이터형과 동일한 이름을 가진다하더라도 일반 함수와 동일할 수 없다
	* 클래스 멤버에 대한 중요한 두 가지 결과를 이끌어 낼 수 있다.
		* 멤버 함수 템플릿에서 생성된 함수느 ㄴ가상 함수를 오버라이딩 하지 않는다.
		* 생성자 템플릿에서 만들어진 생성자는 기본 복사 생성자가 될 수 없다. 

## 8.4 프렌드
* 프렌드 선언의 기본 아이디어는 한 클래스와 특별한 관계를 가지는 클래스나 함수를 프렌드 선언을 통해 알리는 것이다. 다음과 같은 사실로 인해 복잡하게 꼬인다.
	* 프렌드 선언은 실체에 대한 선언만을 의미할 수 있다.
	* 프렌드 함수 선언은 정의가 될 수 있다.

* 클래스 템플릿의 인스턴스 중 하나가 클래스나 클래스 템플릿의 프렌드로 ㅁ나들어지려면 해당 클래스 템플릿이 가시화 돼야 한다는 점을 기억하자 . 일반 클래스의 경우 이와같은 조건이 붙지 않는다.


### 8.4.1 프렌드 함수
* 프렌드 함수의 이름 뒤에 꺾쇠를 붙임으로써 함수 템플릿의 인스턴스를 프렌드로 만들 수 있다.
* 꺾쇠는 템플릿 인자를 가질 수도 있지만 인자가 추론될 수 있다면 빈 채로 둬도 된다.

```c++
void multiply(void*);//일반 함수

template<typename T>
void multiply(T);//함수 템플릿

class Com{
	frinend void muliply(int){}
	friend void ::multiply(void*);//multiply<void*> 인스턴스가 아닌 위의 일반함수를 참조
	friend void ::multiply(int);//템플릿 인스턴스를 참조
}
```


### 8.4.2 프렌드 템플릿
* 일반적으로 함수나 클래스 템플릿의 인스턴스를 프렌드로 선언할 때 어떤 인스턴스를 프렌드로 선언할지 정확히 표현할 수 있다.
* 그렇지만 템플릿의 모든 인스턴스가 클래스의 프렌드라 되는 것이 유용할 수도 있다.이럴 때의 템플릿은 프렌드 템플릿이라 부른다.
* 프렌드 템플릿은 기본 템플릿과 기본 템플릿의 멤버만을 선언할 수 있다. 그 외 기본 템플릿에 연관된 부분 특수화는 자동적으로 프렌드가 된다.

 

# 9장 템플릿에서의 이름
* 굳이.

 
* 여기까지만 하자. 더이상은 무의미할듯.