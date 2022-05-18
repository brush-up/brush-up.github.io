---
title:  "More Effective C++"
excerpt: "More Effective C++"

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2022-05-18
last_modified_at: 2022-05-18


published: false
---

* 2006년에 구매해서 정말 잘 보았던 책이고, 책 구매했떤 시기와 상황에 따른 여러가지 추억등이 담겨있지만, 이제 종이책은 정말 불필요하다는 생각이 들어 정리하려한다. 
* 한번더 훝어 보며 정리중 
	* 한참을 보다보니 지쳐서 찾아보니 아래 정리본이 많은 도움이 되었다.
	* http://ajwmain.iptime.org/programming/programming.html#book_summary.
* 기존에 밑줄 그어놓은곳을 보며 기억이 1도 없는것도 놀라웠고, 그 당시에 한번은 다 봤었구나란 생각도 들고.새록새록 하다. 책 내용 좋네. 지금시대에 살짝 맞지 않는 부분이 있어 아쉽다.



# Chpater 1 기본 개념들
## 항목 1 :포인터(pointer)와 참조자(reference)를 구분하자
* 참조자 개념에선 널 참조자란 것이 없다는 것을 먼저 알아야 한다.
* 어떤 객체를 참조하는 변수를 두고 싶은데, 그 변수가 참조하는 부분에 꼭 객체가 있지 않을 경우도 있다고 한다면 참조자를 버리고 포인터를 써야한다. 
    * 포인터의 값을 null 로 세팅할 수도 있기때문에.
* 이런 어처구니 없는 코드도 있을수 있다. 
```c++
char *pc = 0;//포인터를 널로 세팅한다. 
char& rc = *pc;//널 포인터를 역참조한 것을 참조자를 통해 참조하려한다.
```
* 참조자는 반드시 객체를 참조하고 있어야 하기에 C++ 스펙에 의하면 참조자는 선언될때 반드시 초기화 해야 한다. 
```c++
// 참조자는 선언될 때 반드시 초기화 해야 합니다.
string& rs;            // 에러입니다! 참조는 반드시
                    // 초기화해야 합니다.
 
string s("xyzzy");
string& rs = s;        // 통과. rs는 s를 참조합니다.
 
// 반면 포인터의 경우에는 이런 제약이 없습니다.
 
string *ps;            // 초기화 되지 않은 포인터
                    // 써도 상관없지만 문제가 생길 수 있습니다.
```
* 포인터보다 참조자를 쓰는 것이 더 효율적일수있다란 말은
    * 참조자는 쓰기 전에 유효성을 검사할 필요가 없기 때문
```c++
//-------------------------------------------------------------------
// 널 참조자 같은 것이 존재하지 않기 때문에
// 참조자는 쓰기 전에 유효성을 검사할 필요가
// 없습니다.
 
void printDouble(const double& rd)
{
    cout << rd;        // rd가 널 객체를 참조하는지 볼 필요도
}                    // 없습니다. 반드시 double을 참조해야 합니다.
 
// 반면 포인터를 쓸 경우에는 사용 전에 반드시
// 유효한 객체를 가리키고 있는지 검사해야 합니다.
// (이것을 널 테스트라고 합니다.)
 
void printDouble(const double *pd)
{
    if (pd) {        // 널 포인터인지 검사합니다.
        cout << *pd;
    }
}
```
* 포인터와 참조자가 다른점 또 하나는 
    * 다른 객체를 참조하게 할 수있는가의 여부이다. 
    * 포인터는 다른 객체의 주소값으로 얼마든지 바꾸어 세팅할 수 있지만 참조자는 초기화 될때 참조햇떤 그 객체만 참조한다.
```c++
string s1("Nancy");
string s2("Clancy");
 
string& rs = s1;    // rs는 s1을 "참조합니다."
 
string *ps = &s1;    // ps는 s1을 "가리킵니다."
 
rs = s2;            // rs는 여전히 s1을 참조하지만,
                    // s1의 값은 이제
                    // "Clancy"입니다.
 
ps = &s2;            // ps는 이제 s2를 가리킵니다.
                    // 그리고 s1은 바뀌지 않습니다.
```
* 참조자를 반드시 써야 하는 특수한 상황이 하나 있는데 
    * 연산자 함수를 구현할때이다. 흔한 예는 operator [ ] 인데, 이 연산자는 대입 연산자의 좌변으로 쓸 수 있는 값을 반환해 주도록 만드는 것이 보통이다. 
```c++
vector<int> v(10);    // 크기 10의 int 벡터를 만듭니다.
 
v[5] = 10;            // 이 대입 연산의 대상은 operator[]의
                    // 반환값입니다.
 
// 만일 operator[]가 포인터를 반환하면,
// 다음과 같이 조금 어색한(포인터의 벡터처럼 보이는)
// 형태가 되어야 할 것입니다.
 
*v[5] = 10;
```
* 종합해 보면 아래의 경우를 제외하고는 무조건 포인터를 사용하자. 
    * 참조자는 참조하고자 하는 어떤 객체를 미리 알고 있을때, 
    * 다른 객체를 바꾸어 참조할 일이 결코 없을때 
    * 포인터를 사용하면 뭄법상 의미가 어색해 지는 연산자를 구현할때 선택하면 된다. 

## 항목 2 : 가능한 C++ 스타일의 캐스트를 즐겨 쓰자
* C 스타일의 캐스트의 문제를 보완하기 위해 C++에서 캐스트 연산자 네 가지를 도입했다. 
    * static_cast, const_cast, dynamic_cast, reinterpret_cast
* static_cast는 C 스타일의 캐스트와 똑같은 의미와 형변환 능력을 가지고 있는 연산자이다. 
    * C 스타일의 캐스트와 구실이 똑같다 보니 제약도 똑같습니다. 예를 들어 struct를 int로 바꾼다던지 double을 포인터 타입으로 바꾸는 일은 이것으로 할 수 없습니다.
* 나머지 세 가지의 C++ 캐스트 연산자는 좀더 구체적인 목적을 위해 만들어졌습니다. const_cast는 표현식의 상수성이나 휘발성(volatile)을 없애는 데에 사용합니다
    * 상수성이나 휘발성을 제거하는 것 이외의 용도로 const_cast를 쓸 수 없습니다(통하지 않습니다).
```c++
class Widget { ... };
class SpecialWidget: public Widget { ... };
 
void update(SpecialWidget *psw);
 
SpecialWidget sw;                // sw는 비상수 객체입니다.
const SpecialWidget& csw = sw;    // 그러나 csw는 이 객체를
                                // 상수 객체인 것처럼 참조합니다.
 
update(&csw);        // 에러입니다! const SpecialWidget*는
                    // SpecialWidget*를 받는 함수에 넘길
                    // 수 없습니다.
 
update(const_cast<SpecialWidget*>(&csw));
                    // 이상 무. &csw의 상수성이 명확하게
                    // 제거되었습니다[그리고 csw(sw)
                    // 의 정보는 이제 update 안에서
                    // 바뀔 수 있습니다].
 
update((SpecialWidget*)&csw);
                    // 앞에서와 똑같습니다. 하지만 두 눈을 부릅
                    // 떠야 찾을 수 있는 C 스타일 캐스트입니다.
 
Widget *pw = new SpecialWidget;
update(pw);            // 에러입니다! pw의 타입은 Widget*이지만,
                    // update는 SpecialWidget*를 받습니다.
 
update(const_cast<SpecialWidget*>(pw));
                    // 에러입니다! const_cast는 상수성이나
                    // 휘발성에 영향을 줄 때에만 유효하고,
                    // 상속 관계를 하향시키는 캐스팅 등엔
                    // 쓸 수 없습니다.

```
* dynamic_cast는 상속 계층 관계를 가로지르거나 하향시킨 클래스 타입으로 안전하게 캐스팅할 때 사용합니다. 말하자면, dynamic_cast는 기본 클래스의 객체에 대한 포인터나 참조자의 타입을 파생(derived) 클래스, 혹은 형제(sibling) 클래스의 타입으로 변환해 준다는 것입니다.
    * 캐스팅의 실패는 널 포인터(포인터를 캐스팅할 때)나 예외(참조자를 캐스팅할 때)를 보고 판별할 수 있습니다.
```c++
class Widget { ... };
class SpecialWidget: public Widget { ... };
 
void update(SpecialWidget *psw);
 
Widget *pw;
 
...
 
update(dynamic_cast<SpecialWidget*>(pw));
                // 이상 무. update에 포인터 pw를
                // SpecialWidget에 대한 포인터로서 넘깁니다.
                // 물론 pw가 그 객체를 가리켜야 되고,
                // 그렇지 않으면 널 포인터가 넘어갑니다.
 
void updateViaRef(SpecialWidget& rsw);
 
updateViaRef(dynamic_cast<SpecialWidget&>(*pw));
                // 이상 무. pw가 그 객체를 참조하고 있으면
                // updateViaRef에 *pw를 SpecialWidget
                // 의 참조자 타입으로 넘깁니다. 실패하면
                // 예외가 발생합니다.
 
// dynamic_cast에도 제약이 있습니다.
// 이 연산자는 상속 계층 구조를 오갈 때에만
// 사용해야 하며, 가상 함수가 하나도 없는
// 타입에는 적용할 수 없고
// (자세한 내용은 항목 24를 참조하세요),
// 상수성 제거에도 쓸 수 없습니다.
 
int firstNumber, secondNumber;
...
double result = dynamic_cast<double>(firstNumber) / secondNumber;
                // 에러입니다! 가상 함수가 전혀 없습니다.
 
const SpecialWidget sw;
...
update(dynamic_cast<SpecialWidget*>(&sw));
                // 에러입니다! dynamic_cast는 상수성을
                // 제거하는데 쓸 수 없습니다.
```
*  마지막은 reinterpret_cast입니다. 이 연산자가 적용된 후의 변환 결과는 거의 항상 컴파일러에 따라 다르게 정의되어 있습니다. 즉 이식성이 없어서, 이 연산자가 쓰인 소스는 직접 이식이 불가능합니다.
    * reinterpret_cast의 가장 흔한 용도는 함수 포인터 타입을 서로 바꾸는 것입니다.
```c++
typedef void (*FuncPtr)(void);    // FuncPtr은 인자를 받지 않고
                                // void를 반환하는 함수에 대한
                                // 포인터 입니다.
 
FuncPtr funcPtrArray[10];        // funcPtrArray는 10개의 FuncPtr로
                                // 만들어진 배열입니다.
 
// 이때 다음 함수에 대한 포인터를
// funcPtrArray에 넣어야 할
// 피치 못할 사정이 생겼습니다.
 
int doSomething(void);
 
// FuncPtr의 반환 타입은 void인데,
// doSomething의 반환 타입은 int라서
// 통상적인 방법으론 doSomething을
// funcPtrArray에 넣을 수 없습니다.
// 이럴 때 reinterpret_cast를
// 사용할 수 있습니다.
 
funcPtrArray[0] = &doSomething;    // 에러입니다! 타입 불일치이군요.
 
funcPtrArray[0] =                // 이것은 컴파일됩니다.
    reinterpret_cast<FuncPtr>(&doSomething);
 
 
// 함수 포인터의 캐스팅은 소스의 이식성을
// 떨어뜨리고(C++ 스펙에는 모든 함수 포인터를
// 똑같은 방법으로 나타내야 한다는 보장이 전혀
// 없습니다.), 어떤 경우에 이런 캐스팅은
// 잘못된 결과를 낳기도 하기 때문에(항목 31을 보십시오),
// 여러분의 목에 칼이 들어오기 전에는 함수 포인터의
// 캐스팅은 어떻게든 피하세요.
```
* 물론 C++ 캐스트 연산자를 지원하지 않는 컴파일러를 쓰는 분이라면, 전통적인 캐스팅을 쓸 수 밖에 없는데, 이 때 매크로를 이용하면 C++ 캐스트 연산자의 흉내를 낼 수 있습니다.
```c++
#define static_cast(TYPE, EXPR)((TYPE)(EXPR))
#define const_cast(TYPE, EXPR)((TYPE)(EXPR))
#define reinterpret_cast(TYPE, EXPR)((TYPE)(EXPR))
 
...            // 항목 2의 앞선 예제 참조
 
double result = static_cast(double, firstNumber) / secondNumber;
 
update(const_cast(SpecialWidget*, &sw));
 
funcPtrArray[0] = reinterpret_cast(FuncPtr, &doSomething);
 
// 매크로로 흉내낸 C++ 캐스트 연산자는
// 진짜 연산자들처럼 안전한 캐스팅을
// 수행하는 것은 아니지만, 컴파일러가
// 최신의 캐스트 연산자를 지원하게 되면,
// 코드를 간단히 업그레이드 할 수 있겠지요?
```
* 가능한 C 스타일의 캐스트보다는 C++ 방식을 사용하는 것이 좋다 . 더 좋은 것은 캐스트 연산자를 쓸일이 없네 만드는 것이다. 

## 항목 3 : 배열과 다형성은 같은 수준으로 놓고 볼 것이 아니다
* 아래의 예를 보면서 살펴보자
```c++
class BST { ... };
class BalancedBST: public BST { ... };
 
void printBSTArray(ostream& s,
                   const BST array[],
                   int numElements)
{
    for (int i = 0; i < numElements; ++i) {
        s << array[i];        // operator<< 연산자 함수가
    }                        // BST 클래스에서 이미 정의되어
}                            // 있다고 가정한 것입니다.
 
BST BSTArray[10];
 
...
 
printBSTArray(cout, BSTArray, 10);    // 잘 작동합니다.
 
BalancedBST bBSTArray[10];
 
...
 
printBSTArray(cout, bBSTArray, 10);    // 잘 작동할까?
 
// 위의 코드 중 반복문을 잘 봅시다.
 
for (int i = 0; i < numElements; ++i) {
    s << array[i];
}
 
// 이 코드에서 array와 array + i 사이의
// 거리는 항상 i * sizeof(BST)가 됩니다.
 
// 하지만 printBSTArray에 BalancedBST 배열을
// 넘겨주면 i * sizeof(BalancedBST)로
// 계산되어야 정상적으로 작동할 것입니다.
 
// 그러므로 printBSTArray에 BalancedBST 배열을
// 넘겨주면 어떻게 될지 알 수 없습니다.
```
* 사실 array[i] 는 포인터 값 계산을 수행하는 표현식을 짧게 나타낸 것이다. 
    * 배열의 배열의 첫 요소를 가리키는 포인터라고 외웠는데, array+i가 가리키는 메모리 위치는 배열이 가리키는 위치로부터 얼마나 떨어져 있는지 간단하게 알 수 있다. 
        * i  * sizoef(배열내의 요소 객체 하나) 로 계산하면 된다. array[0]과 array[i]  사이에 i개의 객체가 있으니 결국 array와 array + i 사이의 거리는 i*sizoef(BST)가 된다. 
* 파생 클래스는 보통 기본 클래스보다 더 많은 데이터를 가지고 있기 마련이르모 그 크기도 기본 클래스보다 크다. 
    * printBSTArray 함수에서 만들어진 포인터값 계산 코드가 BalanceBST 객체의 배령에 대해 정확히 돌아가지 않게 된다. 

```c++
class BST { ... };
class BalancedBST: public BST { ... };
 
// 배열을 삭제하는데, 우선 배열 삭제를 알리는
// 메시지를 로그에 남깁니다.
void deleteArray(ostream& logStream, BST array[])
{
    logStream << "Deleting array at address "
            << static_cast<void*>(array) << '\n';
    delete[] array;
}
 
BalancedBST *balTreeArray = new BalancedBST[50];
                // BalancedBST 배열을
                // 생성합니다.
 
...
 
deleteArray(cout, balTreeArray);
                // 배열 삭제를 로깅합니다.
 
// 눈에는 보이지 않지만 이 코드에서도
// 포인터값 계산이 행해집니다.
 
// 배열이 삭제될 때, 배열의 각 요소
// 객체에 대해 소멸자가 호출되어야 합니다
// (항목 8을 보세요).
 
// 컴파일러는 이런 문장을 만나면,
 
delete[] array;
 
// 이와 비슷한 코드를 만들어 내겠지요.
 
// *array내의 객체들을 하나씩 없애는데,
// 객체들이 생성되었던 순서의 반대 순서로 없앱니다.
for (int i = 해당 배열 내의 요소 개수 - 1; i >= 0; --i)
{
    array[i].BST::~BST();    // array[i]의 소멸자를
}                            // 호출합니다.
 
// C++ 언어 스펙에 의하면,
// 기본 클래스 포인터를 통해
// 파생 클래스 객체의 배열을
// 삭제한 결과는 "정의되지 않았다"
// 라고 되어 있지만, 우린 이게
// 무슨 뜻인지 잘 압니다.
```

* 다형성과 포인터 산술 연산은 주먹구구식으로 간단히 섞이는 성질의 것이 아니다. 
* 어떤 구체 클래스(BST같은) 부터 또 다른 구체 클래스 (BalancedBST 같은) 를 파생시키는 일만 없으면 객체의 배열을 다형적으로 조작하는 실수는 별로 하지 않을 것이다. 




## 항목 4 : 쓸데 없는 기본 생성자는 그냥 두지 말자
* 기본 생성자(default constructor, 아무런 인자를 받지 않고 호출될 수 있는 생성자) 
* 생성자는 객체를 초기화하는 역할을 맡기에 , 기본 생성자는 객체가 생성되고 있는 위치의 주변정보를 갖지 않고도 객체를 초기화함
    * 포인터의 역할을 하는 객체는 null이나 정의 되지 않는 값으로 초기화 됨
    * 링크드 리스트, 해시 테이블, 맵 등의 자료구조는 아무 요소도 갖지 않는 컨테이너로 초기화 됨
    * 하지만, 어떤 객체의 경우 외부 정보 없이 완전한 초기화를 수행할 수 없기도 함
        * 가령 주소록(address book)의 입력자료를 나타내는 객체는 외부에서 입력한 인명이 없으면 전혀 의미가 없음
* 아래 예제를 보며 설명해 본다.
```c++
class EquipmentPiece {
public:
    EquipmentPiece(int IDNumber);
    ...
};
 
EquipmentPiece bestPieces[10];    // 에러입니다! EquipmentPiece의
                                // 생성자를 호출할 수 없습니다.
 
EquipmentPiece *bestPieces =
    new EquipmentPiece[10];        // 에러입니다! 마찬가지 문제
 
// 배열을 힙에 만들지 않는다면
// 다음과 같이 해결할 수 있습니다.
 
int ID1, ID2, ID3, ..., ID10;    // 장비의 ID 번호를 가질
                                // 변수들
...
 
EquipmentPiece bestPieces[] = {    // 이제 문제없습니다. 생성자의 인자가
    EquipmentPiece(ID1),        // 주어졌습니다.
    EquipmentPiece(ID2),
    EquipmentPiece(ID3),
    ...,
    EquipmentPiece(ID10)
};
```
* EquipmentPiece  는 기본생성자가 없기에 이 클래스를 썼을때 문제가 일어날 수 있는 경우는 세 가지가 됨
    * 배열을 생성할 때
        * 일반적으로 배열의 요소로서 들어가는 객체ㅐ에 대한 생성자 매개변수를 지정할수 없기에 EquipmentPiece  객체의 배열을 생성하는것은 불가능
            * 하지만 이 제약을 피해 가는 세 가지 방법이 있습니다. 배열을 힙에 만들지 않았을 때의 해결책은 배열이 정의된 위치에서 생성자 매개변수를 직접 넣어 주는 것입니다.
            * 힙에 생성한 배열(new로)에 대해서는 첫 번째 방법을 어떻게 써볼 수가 없답니다. 보다 일반적인 방법은 객체의 배열 대신에 포인터의 배열을 사용하는 것입니다.
```c++
typedef EquipmentPiece* PEP;
 
PEP bestPieces[10];                // 문제없습니다. 생성자가 전혀 호출되지 않습니다.
 
PEP *bestPieces = new PEP[10];    // 역시 문제없습니다.
 
// 이제 배열에 들어 있는 각 포인터들은
// 다른 EquipmentPiece 객체를
// 가리키게 할 수 있겠지요.
 
for (int i = 0; i < 10; ++i)
    bestPieces[i] = new EquipmentPiece(ID Number);
 
// 이 방법은 두 개의 단점을 가지고 있습니다.
 
// 1. 이 배열 내의 모든 포인터가 가리키는
//    객체를 삭제해야 한다는 것을 잊으면
//    안됩니다.
 
// 2. 필요한 메모리 사용량이 늘어납니다.
//    EquipmentPiece 객체를 담을
//    메모리 공간 이외에 포인터를 담을
//    메모리 공간이 필요하니까요.
```
* 두 번째 방법에서 포인터를 위한 메모리 공간의 낭비를 줄이기 위한 방법으로, 배열에 대해 직접 비가공(raw) 메모리를 할당한 후에 "메모리 지정 new(placement new)"(항목 8을 보세요.)를 써서 그 메모리 안에다 EquipmentPiece 객체가 생성되도록 하는 방법이 있습니다.
```c++
// 10개의 EquipmentPiece 객체를 담을 수 있는
// 크기의 비가공 메모리를 생성합니다.
void *rawMemory =
    operator new[](10*sizeof(EquipmentPiece));
 
// bestPieces가 EquipmentPiece 배열 첫 요소의
// 주소가 되도록 세팅합니다.
EquipmentPiece *bestPieces =
    static_cast<EquipmentPiece*>(rawMemory);
 
// "메모리 지정 new(placement new)"를 써서
// 이 메모리 안에 EquipmentPiece 객체
// 10 개를 생성합니다.
for (int i = 0; i < 10; ++i)
    new(bestPieces + i)EquipmentPiece(ID Number);
 
// 메모리 지정 new를 사용하는 방법의 단점은,
// 대부분의 프로그래머가 이것에 별로 친하지
// 않다는 점과는 별개로, "객체를 삭제해야 할
// 때에 소멸자를 손으로 직접 호출해야 한다"
// 는 점입니다.
 
// 물론 소멸자 호출로만 끝나는 것이 아니라,
// 그 다음에는 operator delete[]
// (항목 8을 참고하세요)를 호출해서 원래의
// 비가공 메모리를 직접 해제해야 합니다.
 
// bestPieces 내의 객체들을 그 객체가 생성된
// 순서의 반대 순서로 없앱니다.
for (int i = 9; i >= 0; --i)
    bestPieces[i].~EquipmentPiece();
 
// 원래의 비가공 메모리를 없앱니다.
operator delete[](rawMemory);
 
// 다음과 같이 해제하려 하면 안됩니다.
 
delete[] bestPieces;    // 오작동을 일으킵니다! bestPieces는
                        // new 연산자를 써서 세팅한
                        // 포인터가 아닙니다.
 
// 왜냐하면 new 연산자를 통해 세팅되지 않은 포인터에
// 대해 삭제한 결과는 "정의되어 있지 않기(undefined)"
// 때문입니다.
```
* 기본 생성자가 없는 클래스의 두 번째 문제점은 
    * 많은 템플릿 기반의 컨테이너 클래스에 먹일(?) 수가 없다는 것입니다. 왜냐하면 이런 컨테이너 클래스를 인스턴스화 하기 위해서는, 템플릿 매개변수로 들어가는 타입이 기본 생성자를 가지고 있어야 하기 때문입니다. 대부분의 경우, 신경써서 템플릿을 설계하면(예를 들어 std::vector) 기본 생성자가 필요 없어지기도 합니다. 하지만 "신경" 과는 거리가 먼 방법으로 설계된 템플릿들도 많습니다. 이런 경우에는 기본 생성자가 없는 클래스를 템플릿 매개변수로 쓸 수 없습니다.
```c++
template <typename T>
class Array {
public:
    Array(int size);
    ...
    
private:
    T *data;
};
 
template <typename T>
Array<T>::Array(int size)
{
    data = new T[size];        // 배열 안의 각 요소에 대해
    ...                        // T::T()를 호출합니다.
```
* 기본 생성자를 제공 할것이냐 말 것이냐에 대한 딜레마
    * 이 문제에는 가상 기본 클래스 이야기가 따라오는데, 기본 생성자가 없는 가상 기본 클래스는 소프트웨어 개발에 쓰기 힘들다는 문제도 따라옵니다. 그 이유는 가상 기본 클래스의 생성자 매개변수를, 생성되는 객체의 파생 클래스 쪽에서 제공해야 하기 때문입니다.
        * 기본 생성자가 없는 가상 기본 클래스를 상속한 파생 클래스는 어처구니없게도 기본 클래스의 생성자 매개변수의 리스트와 각 매개변소의 의미를 알고 직접 제공해야하는 결과가 나옮
* 기본 생성자로 초기화 할 만한 적당한 값이 없는 클래스에게 억지로 기본 생성자를 만들면, 이 클래스의 다른 멤버 함수들이 복잡해 질 수 있습니다. 왜냐하면 이 클래스의 객체가 항상 제대로 초기화 되었다는 보장을 할 수 없기 때문에, 거의 모든 멤버 함수에서 해당 객체가 초기화 되었는지 점검하는 코드가 추가되어야 할 것이기 때문입니다.
```c++
// 기본 생성자가 없는 클래스는 사용상에
// 꽤나 귀찮은 제약이 따르기 때문에,
// 어떤 사람은 기본 생성자로는 객체를
// 초기화하는데 필요한 정보를 얻기 힘든
// 상황에서도 다음과 같이 억지로
// 기본 생성자를 넣기도 합니다.
 
class EquipmentPiece {
public:
    EquipmentPiece(int IDNumber = UNSPECIFIED);
    ...
    
private:
    static const int UNSPECIFIED;    // ID가 설정되지 않았다는
    ...                                // 의미를 가지고 고정된
};                                    // 상수입니다.
 
// 이와 같이 만든 경우 EquipmentPiece의
// 거의 모든 멤버 함수들에는 IDNumber가
// UNSPECIFIED인지를 검사하는 코드가
// 추가되어야 할 것입니다.
```
* 쓸 데 없는 기본 생성자는 클래스의 효율에도 걸림돌이 됩니다. 멤버 함수들은 클래스의 멤버 데이터가 제대로 초기화 되었는지를 검사해야 하니, 이 검사 시간 만큼 실행 속도가 떨어지겠지요. 게다가, 추가된 검사 코드의 크기만큼 실행 파일 및 라이브러리도 커집니다. 그뿐만이 아니라 검사에 실패했을 경우를 처리하는 코드도 생각해야 합니다. 이렇게 저렇게 낭비되는 비용이 꽤 쏠쏠합니다.

# Chpater 2 연산자(Operators)
## 항목 5 : 사용자 정의 타입변환 함수에 대한 주의를 놓지 말자
* C++안의 암시적 타입변환에 대해서는 어떻게 할수는 없다. 
    * short 를 double로 변환한다던자 char를 int로 변환하는 등의.
* 컴파일러가 사용할수있는 타입변환은 두가지 종류이다.
    * 단일 인자 생성자 
        * 인자를 하나만 받아 호출되는 생성자 
            * 이런 생성자는 매개변수가 하나를 받도록 선언되어있떤지, 여러개인데 처음 것을 제외한 나머지가 모든 기본값을 갖도록 선언되어있다. 
    * 암시적 타입 변환 연산자
        * 보통의 함수 선언 형태가 아니라 타입 앞에 operator만 붙이는 모양이다
            * 반환값에 대해 어떤 타입을 지정해 줄수가 없다. 반환값의 타입이 그냥 함수 이름이다. 
```c++
//단일 인자 생성자 예시
class Name{
public:
    Name(const string& s);//string 타입을 Name타입을오 바꿉니다.
    ...
};

class Rational{
public:
    Rational(int numberator = 0, int denominator = 1);//int를 Rational 로 바꿉니다. 
    ...
};

//암시적 타입변환 연산자 예시
class Rational{
public:
   ...
   operator double() const;//Rational을 double로 암시적으로 바꿉니다.
};
//이 함수는 다음과 같은 문장에 들어있을때 자동으로 호출됩니다.
Rational r(1,2);//r은 1/2이란 값을 가지고 있음
double d = 0.5 * r;//r을 double로 바꾸고 곱셉을 수행.
```
* 진짜 설명하고 싶은것은 타입 변환 함수란 것을 왜 제공하기 싫은가에 대한 것이다. 
* 특히 타입 변환 연산자(함수)는 마음 깊은 곳부터 원하지 않는 이상 제공하지 않는 편이 좋습니다. 가급적이면 같은 일을 하고, 다른 이름을 갖는 멤버 함수로 만드는 편이 낫습니다. 왜냐하면 단일 인자 생성자와는 다르게, 타입 변환 연산자(함수)는 암시적으로 호출되는 것을 막을 방법이 없기 때문입니다.
```c++
class Rational {
public:
    operator double(void) const;    // Rational을
    ...                                // double로 암시적으로 바꿉니다.
}
 
// 이 함수는 다음과 같은 문장에 들어 있을 때
// 자동으로 호출됩니다.
 
Rational r(1, 2);                    // r은 1/2이란 값을 가지고 있습니다.
 
double d = 0.5 * r;                    // r을 double로 바꾸고
                                    // 곱셈을 수행합니다.
 
// 이 때 Rational 객체를 기본 제공 타입처럼
// 콘솔에 출력하고 싶어서 다음과 같은 코드를
// 작성했습니다.
 
Rational r(1, 2);
 
cout << r;                            // "1/2"를 출력해야 합니다.
 
// 이 때 Rational 객체에 operator<< 함수를
// 작성해 넣는 것을 잊었다는 가정을 하나 더 합시다.
 
// 많은 사람들이 이 경우 컴파일 오류가 날 것이라고
// 생각할 수 있지만, 사실은 Rational의 변환
// 함수가 호출되어 double형으로 암시적으로 변환된
// 후 출력에 성공합니다.
 
// 하지만 "1/2" 형태가 아닌 부동 소수점 형태인
// "0.5"와 같이 원하지 않는 형태로 출력이 될 것입니다.
 
// 이를 해결하는 방법은 타입 변환 연산자를
// 제공하는 대신 똑같은 일을 하는 다른 이름의
// 멤버 함수를 만드는 것입니다.
 
class Rational {
public:
    ...
    double asDouble(void) const;    // Rational을
};                                    // double 타입으로 바꿉니다.
 
// 그리고 이 멤버 함수를 직접 호출합니다.
 
Rational r(1, 2);
 
cout << r;                            // 에러입니다! Rational을
                                    // 받는 operator<<가 없습니다.
 
cout << r.asDouble();                // 문제없습니다. r을 double 타입으로
                                    // 출력합니다.
```
* 업계 최고의 전문가 사람들이 만든 string 클래스에는 stinrg 타입을 c 스타일의 char* 로 바꾸는 암시적 변환 연산저가 없다 . 대신 직접 호출할수잇는 c_str 을구도 이것으로 타입을 변환하도록 한다. 
* 단일 인지 생성자를 통한 암시적 타입변환은 없애기가 더 까다롭다. 
* 단일 인자 생성자는 다른 이름의 멤버 함수를 만드는 방법을 쓸 수 없습니다. 왜냐하면 단일 인자 생성자가 꼭 필요한 경우도 많기 때문입니다. 이 경우 단일 인자 생성자가 암시적으로 호출되지 않도록 막는 방법을 써야 합니다. 방법은 두 가지가 있습니다. 간단한 방법은 생성자 앞에 explicit 키워드를 붙이는 것입니다. 다른 방법으로는 프록시 클래스(항목 30을 참고하세요)를 사용하는 방법이 있습니다.
    * explicit 기능은 암시적 타입변환의 문제를 막기 위해서 만들어진 것이다.이 키워드를 생성자 앞에 붙여만 주면 된다. 
        * 매개변수와 호출이 명확할 때에만 이 생성자를 호출하라고 알려주는 뜻이다. 
        * explicit 선언된 생성자는 암시적 타입변환에 사용도지 않는다. 하지면 명시적 타입변환은 여전히 허용한다. 
```c++
template <typename T>
class Array {
public:
    Array(int lowBound, int highBound);
    Array(int size);
    
    T& operator[](int index);
    ...
};
 
// 첫 번째 생성자 처럼 인자를 두 개 받는
// 생성자는 타입 변환 함수로는 써먹을 수
// 없습니다.
 
// 두 번째 생성자는 인자를 하나만 받기 때문에
// 타입 변환 함수로 쓰일 수 있습니다.
// 즉 int 타입을 Array<T>타입으로
// 암시적으로 변환할 수 있게 됩니다.
 
// 예를 하나 들겠습니다. Array<T>를
// 특수화 시킨 Array<int> 객체가 있고,
// 이 객체들을 비교하는 연산자, 그리고 이것을
// 사용하는 코드가 다음과 같다고 합시다.
 
bool operator==(const Array<int>& lhs,
                const Array<int>& rhs);
Array<int> a(10);
Array<int> b(10);
 
...
 
for (int i = 0; i < 10; ++i)
    if (a == b[i]) {            // 윽! "a"는 "a[i]"가 되어야 합니다.
        a[i]와 b[i]가 같으면
        정해둔 무언가를 합니다.
    }
    else {
        그렇지 않을 때에는 또 무언가를 합니다.
    }
 
// 보시다시피 실수로 조건문의 "a"옆에 "[i]"을
// 빼먹었습니다.
 
// 이런 상황에서 컴파일러는 컴파일 오류를 내지 않고,
// 암시적으로 b[i](int 타입의 값)을 Array<int>로
// 바꾸어 비교를 해 버립니다. 즉 컴파일러는
// 다음과 비슷한 코드를 생성할 것입니다.
 
for (int i = 0; i < 10; ++i)
    if (a == static_cast<Array<int> >(b[i])) ...
 
// 암시적 타입변환 연산자의 취약점은 같은 일을 하되
// 이름이 다른 멤버함수를 대신 만드는 방식으로
// 쉽게 피할 수 있습니다.
 
// 하지만 단일 인자 생성자는 그렇게 쉽게 피해갈 수
// 있는 성질의 것이 아닙니다. 단일 인자 생성자는
// 진짜로 제공해야 할 경우도 적지 않거든요.
 
// 그래서 컴파일러가 무분별하게(암시적으로)
// 이런 연산자를 호출하지 않도록 만들어야 합니다.
 
// 첫 번째 방법은 explicit 키워드를 사용하는
// 것입니다.
 
template <typename T>
class Array {
public:
    ...
    explicit Array(int size);    // "explicit"의 용법을 잘 알아두세요.
    ...
};
 
Array<int> a(10);                // 좋습니다, explicit로 선언된 생성자는
                                // 객체 생성시에 평상시와 마찬가지로
                                // 사용할 수 있습니다.
 
Array<int> b(10);                // 역시 사용할 수 있습니다.
 
if (a == b[i]) ...                // 에러입니다! int를
                                // 암시적으로 Array<int>로
                                // 바꿀 방법이 없습니다.
 
if (a == Array<int>(b[i])) ...    // 좋습니다. int를 Array<int>로
                                // 바꾸는 작업이 명시적으로
                                // 이루어졌습니다(그러나 코드의
                                // 로직은 좀 의심스럽네요).
 
if (a == static_cast<Array<int> >(b[i])) ...
                                // 마찬가지로 문제없고,
                                // 마찬가지로 의심스럽습니다.
 
if (a == (Array<int>)b[i]) ...    // C 스타일의 캐스팅 역시
                                // 별 문제가 없지만, 코드의
                                // 로직은 여전히 의심스럽습니다.
 
// 또 다른 방법으로 프록시 클래스를
// 사용하는 방법이 있습니다.
 
template <typename T>
class Array {
public:
 
    class ArraySize {            // 새로 만든 프록시 클래스(proxy class)
    public:
        ArraySize(int numElements) : theSize(numElements) {}
        int size(void) const { return theSize; }
    private:
        int theSize;
    };
    
    Array(int lowBound, int highBound);
    Array(ArraySize size);        // 선언 형식이 새로워졌습니다.
 
    ...
 
};
 
bool operator==(const Array<int>& lhs,
                const Array<int>& rhs);
Array<int> a(10);                // 문제없습니다. 10(int 타입)이
Array<int> b(10);                // ArraySize로 암시적으로 변환됩니다.
 
...
 
for (int i = 0; i < 10; ++i)
    if (a == b[i]) ...            // 저런! "a"는 "a[i]"가 되어야 합니다.
                                // 따라서 이 코드에서 에러가 납니다.
 
// 조건문에서 int는 ArraySize로,
// ArraySize는 Array<int>로
// 암시적으로 변환될 수 있지만
// int가 Array<int>로 변환되려면
// 두 단계를 거쳐야 합니다.
 
// C++에서 두 단계를 암시적으로 변환하는것은
// 금지되어 있습니다.
```
* 이번항목의 교훈은 
    * 컴파일러가 마음대로 암시적 타입변환을 하도록 내러벼두면 대개 득보다 실이 많다는 것과 타입변환 연산자(함수)는 마음 깊은 곳부터 원하지 않는 이상 제공하지 말라는 것이다. 



## 항목 6 : 증가 및 감소 연산자의 전위(prefix)/후위(postfix) 형태를 반드시 구분하자
* 증감 연산자를 오버로딩 할 때, 전위형 증감 연산자와 후위형 증감 연산자를 구분하기 위해 후위형 증감 연산자에는 0(int)를 인자로 넘깁니다
```c++
class UPInt {
public:
    UPInt& operator++(void);        // 전위 ++
    const UPInt operator++(int);    // 후위 ++
    
    UPInt& operator--(void);        // 전위 --
    const UPInt operator--(int);    // 후위 --
    
    UPInt& operator+=(int);            // UPInt과 int에 대해 마련한
                                    // += 연산자
    ...
    
};
UPInt i;
 
++i;                                // i.operator++()를 호출합니다.
i++;                                // i.operator++(0)를 호출합니다.
 
--i;                                // i.operator--()를 호출합니다.
i--;                                // i.operator--(0)를 호출합니다.

```
* 전위형 증감 연산자와 후위형 증감 연산자는 반환 타입도 구분해야 합니다. 전위형 증감 연산자는 반드시 참조자 타입을 반환해야 하고, 후위형 증감 연산자는 반드시 const 객체 타입을 반환해야 합니다.

* 전위,후위 연산자는 아래처럼 구련되어있다.
    * 후위 연산자는 매개변수를 사용하지 않도록 되어있다는 점을 인지하고 보자. 
```c++
// 전위 형태: 증가시킨 후에 값을 가져옵니다.
UPInt& UPInt::operator++(void)
{
    *this += 1;                        // 증가
    return *this;                    // 값을 가져옵니다.
}
 
// 후위 형태: 값을 가져오고 증가시킵니다.
const UPInt UPInt::operator++(int)
{
    const UPInt oldValue = *this;    // 값을 가져옵니다.
    ++(*this);                        // 증가(무엇을 쓰나 잘 보세요)
    
    return oldValue;                // 미리 가져온 값을
}                                    // 반환합니다.
 
// 보시다시피 후위형 증감 연산자는
// 임시 객체를 반환해야 하기 때문에
// *this의 참조자가 아닌 객체 타입을
// 반환합니다. 하지만 왜 const가
// 되어야 할까요? 다음을 잘 보세요.
 
UPInt i;
 
i++++;                                // 후위 증가 연산자를
                                    // 두 번 적용합니다.
 
// 이 코드는 다음과 같습니다.
 
i.operator++(0).operator++(0);
 
// 여기서 후위 연산자가 const가 아닌 객체를
// 반환하는 것을 꺼려야 하는 데는 두 가지의 이유가
// 있습니다.
 
// 첫째 이유는 기본 제공(built-in) 타입에
// 대한 증가 연산자의 동작과 맞지 않는다는 것입니다.
 
// "아리송하면 int의 동작원리대로 만들지어다"
// 라는 말을 떠올리시면 되겠습니다.
 
int i;
 
i++++;                                // 어이 없이 에러!
 
// 두 번째 이유로는 다음을 들 수 있습니다.
 
// 두 번째로 호출된 증가 연산자는
// 첫 번째로 호출된 증가 연산자가
// 반환한 임시 객체를 증가 시킵니다.
// 따라서 i는 2가 아니라 1만 증가합니다.
 
// 보시다시피 매우 직관적이지 못합니다.
 
// 이것을 막는 가장 쉽고 빠른 방법이
// 후위형 증감 연산자의 반환 타입에
// const를 붙이는 것입니다.
```
* 전위 연산자는 증가 시키고 값을 사용하는 (increment and fetch) 연산자이고, 후위 연산자는 값을 사용하고 증가시키는(fetch and increment)연산자라고 외우고있을 텐데, 
    * 후위 증가 연산자가 증가되기전의 객체를 반환해야한다는 점이 납득이 되는가요? 왜 const 타입이어야 할까요?
    * 일단 const 가 아니라고 가정을 하고 위의 예제를 다시 봅시다. 
* 실행 효율에 민감하면 후위 증가 연산자를 보면 
    * 반환값으로 쓰이 위한 임시 객체를 만들어야 하고 지역변수로 생겼다 없어지는 임시 객체도 만들고 있는데, 전위 증가 연산자 함수는 이렇지 않다는 점을 알아두자. 
* 전위 증가 연산자와 후위 증가 연산자의 동작 방식을 보면 객체의 값을 증가시키는 일을 하는데
    * 어떻게 후위 증가 연산자와 전위 증가 연산자의 동작방식이 같을수가 있을까?
        * 아래 원칙대로 해야한다.
            * 후위 증가 연산자는 전위 증감 연산자를 사용해서 구현해야한다는 것이다.





## 항목 7 : &&, ||, 혹은 . 연산자는 오버로딩 대상이 절대로 아니다
* C++ 에서는 operator&&와 operator|| 함수를 오버로딩 하면
    * &&, || 도 사용자의 입맛에 맞게 동작하도록 구현할 수 있다. 
    * 이 연산자의 오버로딩은 전역함수로도 가능하고, 클래스의 멤버 함수로도 할 수 있다.
* &&, || 연산자는 단축평가(short-circuit) 처리를 할 수 있습니다. 하지만 이 연산자들을 오버로딩하게 되면, 본래의 단축평가 의미구조(short-circuit semantics)가 함수호출 의미구조(function call semantics)로 바뀌게 됩니다. 이 때문에 단축평가가 수행되지 않음은 물론이고, 왼쪽 표현식이 오른쪽 표현식보다 먼저 평가된다는 확신도 할 수 없게 됩니다.
```c++
if (expression1 && expression2) ...
 
// 만약 && 연산자를 오버로딩 하지 않았다면,
// 반드시 expression1이 면저 평가되며,
// expression1이 거짓인 경우 expression2는
// 평가되지 않습니다.
 
// 하지만 만약 && 연산자를 오버로딩 한 경우
// 컴파일러에게는 다음과 같은 형태로 보입니다.
 
if (expression1.operator&&(expression2)) ...
                // operator&&가
                // 멤버 함수일 경우
 
if (operator&&(expression1, expression2)) ...
                // operator&&가
                // 전역 함수일 경우
 
// 보시는 바와 같이 함수 호출로 취급하여
// 표현식을 평가합니다.
 
// 1. 함수가 호출되기 전에, 모든 인자가
//    평가되어야 하므로 단축평가는 이루어지지
//    않습니다.
 
// 2. 함수 인자의 평가 순서는 표준으로
//    정해져 있지 않습니다. 따라서
//    expression2가 먼저 평가될
//    수도 있습니다.
```
* 쉼표 연산자는 왼쪽 표현식이 먼저 평가된 후 오른쪽 표현식이 평가되며, 전체 표현식의 평가 결과는 오른쪽 표현식의 평가 결과값으로 정해집니다. 하지만 쉼표 연산자 역시 논리 연산자와 마찬가지의 이유로, 오버로딩을 하게 되면 왼쪽 표현식이 오른쪽 표현식보다 먼저 평가된다고 보장할 수 없게 됩니다.
    * 쉼표 연산자는 for 루프의 증감식 부분에서 많이 봤을 것이다. 
```c++
{
    ...
    for(int i=0, j=strlen(s)-1; i<j; ++i, --j)//쉼표 연산자
    {
        ...
    }
    ...
}
```
* 오버로딩 할수 없는 연산자는 아래와 같이 12가지이다.
```
. .* :: ?:
new delete sizeof typeid 
static_cast dynamic_cast const_cast reinterpret_cast
```
* 오버로딩 가능한 연산자는 굳이 여기서.


## 항목 8 : new와 delete의 의미를 정확히 구분하고 이해하자
* new 연산자와 ㅏoperator new의 차이를 아시나요?
```c++
//여기에 사용된 new는 new 연산자. C++에서 기본적으로 제공되는것이고 sizeof 처럼 동작 원리를 바꿀수는 없음
//new 연산자의 동작은 두단계
//1. 요청한 타입의 객체를 담을 수 있는 메모리를 할당
//2. 객체의 생성자를 호출하여 할당된 메모리에 객체 초기화를 수행
string *ps = new string("memory managemnent");

//여러분이 바꿀수있는것은 객체를 담을 메모리를 할당하는 방법임
//new는 메모리 할당을 위해 어떤 함수를 호출되도록 만들어져 있는데, 이것을 오버로딩 하면됨
//new 연산자가 호출하는 그 함수의 이름이 바로 operator new 입니다.
//operator new함수는 대개 아래와 같음
void * operator new (size_t size);

// operator new 함수를 직접 호출하려면
// 다음과 같이 코드를 작성할 수 있습니다.
 void *rawMemory = operator new(sizeof(string));
 
// 이렇게 하면, operator new 는
// string타입의 객체를 담을 수 있는
// 메모리 덩어리를 할당해서 그 메모리의
// 포인터를 반환합니다.
```
* operator new의 역할은 malloc 와 마찬가지로 메모리만 할당하는 것임
    * 할당된 메모리를 받아 객체 구실을 할수 있도록 하는 것이 new 연산자임
```c++
// 컴파일러는 다음 문장을 만나면,
 
string *ps = new string("Memory Management");
 
// 다음의 내용과 크게 어긋나지 않는 코드를
// 만들어 내야 합니다.
 
void *memory =                            // string 타입의 객체를
    operator new(sizeof(string));        // 담을 크기의 미초기화
                                        // 메모리를 얻어냅니다.
 
*memory에 대해                                // 그 메모리에 값을
string::string("Memory Management")를    // 세팅하여(생성자호출)
                        호출한다.            // 객체를 초기화합니다.
 
string *ps =                            // ps가 새 객체를 가리키게
    static_cast<string*>(memory);        // 합니다.
 
// 생성자 호출 단계는 '일개' 프로그래머인
// 여러분이 손 댈 수 있는 부분이 아닙니다.
```

### 메모리 지정 new
* 메모리 지정 new(Placement new) 함수
    * 이용하면 이미 가지고 있는 메모리(할당받은 미초기화 메모리)에 원하는 객체의 생성자를 호출하여 객체를 생성할 수 있습니다. 메모리 지정 new 가 사용되는 다른 예제는 항목 4 에서 찾을 수 있습니다.
* 메모리 지정 new는 아래의 예제를 보며 살펴보자. 
```c++
// 메모리 지정 new(placement new)를
// 사용하기 위해서 포함해야 하는 헤더입니다.
#include <new>
 
class Widget {
public:
    Widget(int widgetSize);
    ...
};
 
Widget * constructWidgetInBuffer(void *buffer,
                            int widgetSize)
{
    return new (buffer) Widget(widgetSize);        // 메모리 지정 new
}
 
// buffer가 가리키는 메모리 공간을,
// Widget의 생성자를 호출하여
// 초기화 합니다.
 
// 메모리 지정 new의 선언 형태는
// 다음과 같습니다.
 
void * operator new(size_t, void *location)
{
    return location;
}
 
// operator new의 첫 번째 인자는 항상
// size_t 이어야 하기 때문에 size_t 인자를
// 사용하지 않아도 넣어 줍니다.
// 후위식 증감 연산자와 비슷하죠.
 
// 어떤 메모리에 객체를 생성하는 용도에
// 메모리 지정 new를 사용한 경우에는,
// 그 메모리에는 절대로 delete 연산자를
// 써서는 안 됩니다.
 
// 위치지정 new가 반환하는 메모리는
// 원천적으로 operator new에 의해
// 할당된 메모리가 아닙니다.
 
// 위치 지정 new는 자신에게 넘어온
// 포인터를 그냥 반환했을 뿐이고요.
 
// 이 포인터가 어디서 왔는지 누가 알겠습니까?
// 생성자가 한 일을 되돌려 놓으려면
// 그 객체의 소멸자를 직접 호출할 수밖에
// 없습니다.
 
 
// 공유 메모리에 메모리를 할당하고 해제하는
// 함수
void * mallocShared(size_t size);
void freeShared(void *memory);
 
void *sharedMemory = mallocShared(sizeof(Widget));
 
Widget *pw =                                    // 앞의 경우와 마찬가지로
    constructWidgetInBuffer(sharedMemory, 10)    // 메모리 지정 new가
                                                // 사용됩니다.
...
delete pw;            // 결과 예측 불능! sharedMemory의 어머니는
                    // operator new가 아닌 mallocShared입니다.
 
pw->~Widget();        // 문제없음. pw가 가리킨 Widget 객체에 대해
                    // 소멸자를 호출하지만, 이 객체가 자리잡은
                    // 메모리는 해제하지 않습니다.
 
freeShared(pw);        // 문제없음. pw가 가리킨 메모리를 해제하지만,
                    // 소멸자는 호출하지 않습니다.
```
* 어떤 객체를 힙에 생성할때에는 new 연산자를 쓰고(생성자 호출됨), 메모리 할당만 하고 있을때에는 operator new함수를 쓰자(생성자 호출 안됨)
### 객체 삭제와 메모리 해제
* delete 연산자도 new 연산자와 마찬가지로 내부에서 operator delete 라는 함수를 호출합니다. new 연산자와 마찬가지로 delete 연산자도 오버로딩 할 수 없고 operator delete 함수를 오버로딩 하여, 메모리를 해제하는 방법을 변경할 수 있습니다.
    * operator new와 operator delete는 말하자면, C++ 버전의 malloc 과 free 인 것입니다.
```c++
string *ps;
...
delete ps;                // delete 연산자를 사용합니다.
 
// operator delete 함수는 다음과 같이
// 선언되어 있습니다.
 
void operator delete(void *memoryToBeDeallocated);
 
// 컴파일러는 다음 문장
 
delete ps;
 
// 을 보고 다음과 비스무레한 코드를 만듭니다.
 
ps->~string();            // 그 객체의 소멸자를 호출합니다.
operator delete(ps);    // 그 객체가 자리잡은
                        // 메모리를 해제합니다.
 
// 미초기화 메모리만을 가지고 어떤 일을
// 할 때에는 new와 delete 연산자를
// 건너 뛰어야 합니다. 그 대신에
// 메모리를 얻을 때는 operator new를,
// 그 메모리를 해제할 때에는 operator delete를
// 호출하라는 뜻이죠.
 
void *buffer =                            // 50개의 문자를 담을 수 있는
    operator new(50 * sizeof(char));    // 메모리를 할당합니다. 그리고
                                        // 생성자는 호출하지 않습니다.
...
operator delete(buffer);                // 메모리를 해제합니다.
                                        // 소멸자는 호출하지 않습니다.
```
* new 와 delete , operator new와 operator delete 짝을 잘못 맞추면 어떻게 될지 알겠지?
### 배열 (Array)
* 배열을 할당하는 경우는 어떨까요? 배열을 할당하는 경우 사용되는 new 연산자는 단일 객체를 할당할 때 쓰는 new 연산자와 조금 다릅니다. 
* 배열을 할당하는 new 연산자는 내부에서 operator new 대신에 operator new[] 를 호출합니다. 
* 또 다른점은 배열의 각 요소에 대해 일일이 생성자를 호출해야 한다는 점이 있습니다. 
* 배열을 해제하는 delete 연산자도 마찬가지로 내부적으로 operator delete 대신 operator delete[]를 호출하고, 배열의 각 요소에 대해 일일이 소멸자를 호출해야 합니다.
* operator new[]와 operator delete[]도 오버로딩하여 원하는 방법으로 메모리를 할당하고 해제하도록 변경할 수 있습니다. 하지만 오버로딩 방법에는 제한이 조금 있는데 여기에 대한 자세한 설명은 다른 좋은 C++ 책에서 찾는 게 좋겠습니다.
* 결국 정리하면 아래 예시와 같다.
```c++
string *ps = new string[10];//operator new[]를 호출하여 10개의 stinrg 객체에 대한 메모리 할당
//그리고 나서 배열의 각 요소에 대해 기본 생성자를 호출

delete [] ps;//배열의 각 요소에 대해 소멸자를 호출하고나서, operator delete[] 를 써서 배열 전체의 메모리를 해제함
```


# Chpater 3 예외(Exceptions)
## 항목 9 : 리소스 누수를 피하는 방법의 정공(正攻)은 소멸자이다
* 앙증맞은 동물보호소(Adorable Little Animal, ALA)이란 소프트웨어를 만든다고 가정하고 아래의 예시를 보자

```c++
//우선 추상기본 클래스인 ALA를 정의하고 강아지와 고양이를 나타내는 구체 클래스를 파생시키고 가상함수인 processAdoption은 동물에 따라 적절한 처리를 한다고 해보자
class ALA{
public:
    vitrual void processAdoption() = 0;
    ...
};

class Puppy:public ALA{
public:
    virtual void porcessAdoption();
    ...
};

class Kitten:public ALA{
public:
    virtual void porcessAdoption();
    ...
};
//이러한 상태에서 파일을 읽어 각각에 맞게 처리한다고 하면
ALA * readALA(istream& s);
//아래와 같이 동작하는 가상함수가 핵심일 것이다.
void processAdoptions(istream& s)
{
    while(s){
        ALA *pa = readALA(s);
        pa->processAdoption();
        delete pa;
    }
}
```
* 위와 같았을때 pa->processAdoption이 예외를 발생하면 어떤 일이 일어날까?
    * processAdoption 함수는 예외 처리를 못하므로 발생된 예외는 processAdoption 를 호출한 함수에 전파 된다.이때 pa->processAdoption를 호출한 이후의 processAdoptions 코드는 모두 실행되지 않은채로 지나간다. 
    * 즉 pa는 삭제가 안된다. 
* 아래처럼 하면 어느정도 해결이 될것인데.
```c++
void processAdoptions(istream& s)
{
    while(s){
        ALA *pa = readALA(s);
        try{
            pa->processAdoption();
        }catch(...){//모든 예외를 받아 처리한다.
            delete pa;//예외 발생시 리소스 누수를 처리한다.
            throw;//예외를 호출부로 전파한다.
        }
        delete pa;//예외가 발생하지 않을 경우 리소스 누수를 피한다.
    }
}
```
* try-catch 블록으로 코드가 복잡해진다. 똑같은 코드가 많아지는 문제가 있다 
    * 만약 processAdoptions 함수의 스택에 생긴 지역 객체에 대해 항상 실행되어야 하는 마무리 코드를 소멸자로  옮기면 어떯까?
* 결국 해답은 포인터 pa를 포인터처럼 동작하는 객체로 대신하면 된다. 
    * 객체가 자동적으로 소멸할때 이 객체의 소멸자에서 delete를 호출하도록 할 수 있다. 
* 조금 생각해보면 이런한 객체는 아래와 같은 식으로 만들수 있을 것이다.
    * 실제 auto_ptr은 훨씬 잘 만들어졌다.
```c++
// auto_ptr의 대략적인 구조는 다음과 같습니다.
template <typename T>
class auto_ptr {
public:
    auto_ptr(T *p = 0) : ptr(p) {}    // 객체에 대한 포인터를 ptr에 저장합니다.
    ~auto_ptr(void) { delete ptr; }    // 객체에 대한 ptr을 삭제합니다.
    
private:
    T *ptr;                            // 객체에 대한 미초기화 포인터
};
 
// 실제로 표준 라이브러리에 구현된 auto_ptr은
// 이것보다 훨씬 잘 만들어져 있습니다.
 
// 위의 코드를 제대로 쓰려면 복사 생성자,
// 대입 연산자, 포인터 동작을 흉내내는 함수들을
// 추가해야 합니다. 자세한 이야기는
// 항목 28을 참고하세요.
```
* C++ 원시 포인터 대신에 auto_ptr을 사용하도록 고치면 다음과 같을 것이다.
```c++
void processAdoptions(istream& s)
{
    while(s){
        auto_ptr<ALA> pa(readALA(s));
        pa->processAdoption();
    }
}
```
* auto_ptr 의 기본아이디어(자동으로 해제되어야 하는 리소스를 이 객체에 저장하고 리소스의 해제를 객체의 소멸자에게 맡기는것)는  포인터를 기반으로 조작하는 리소스 이외의 경우에도 적용할 수 있다.




## 항목 10 : 생성자에서는 리소스 누수가 일어나지 않게 하자
* 멀티미디어 기능을 가진 주소록 소프트웨어를 개발한다고 생각하고 아래의 예제를 보자. 아래와 같이 설계하였다.
```c++
class Image{//이미지 데이터를 위한 클래스
public:
    Image(const string& imageDataFileName);
    ...
};
class AudioClip{//오디오 데이터를 위한 클래스
public:
    AudioClip(const string& audioDataFileName);
    ...
};
class PhoneNumber{...};//전화번호를 저장하는 클래스
class BookEntry{
public:
    BookEntry(const string&name, const string& address = "", const string& imageFileName = "", const string& audioClipFileName = "");
    ~BookEntry();
    
    //전화번호를 추가하는 함수
    void addPhoneNumber(const PhoneNumber& number)'
    ...
private:
    string theName;
    string theAddress;
    list<PhoneNumber> thePnones;
    Image *theImage;
    AudioClip *theAudioClip;
};
```
* 각각의 BookEntry 객체는 반드시 이름 데이터를 가져야해서 이 이름을 생성자 인자로 넣어주어야 합니다.(항목 4 참고)
* 하지만 다른 필드는 빠져도 상관없습니다.
* BookEntry 클래스의 생성자와 소멸자를 가장 간단히 만드려면 아래처럼 합니다.
```c++
BookEntry::BookEntry(const string& name,
                     const string& address,
                     const string& imageFileName,
                     const string& audioClipFileName)
: theName(name), theAddress(address),
theImage(0), theAudioClip(0)
{
    if (imageFileName != "") {
        theImage = new Image(imageFileName);
    }
    if (audioClipFileName != "") {
        // 아래 문장에서 예외가 발생하면 메모리 누수가 발생합니다.
        theAudioClip = new AudioClip(audioClipFileName);
    }
}
 
BookEntry::~BookEntry(void)
{
    delete theImage;
    delete theAudioClip;
}
```
* 여기까지 했으면 별 문제 없을것 같지만, 비정상적인 상태에서는 별별 문제가 생깁니다.
* BookEntry 생성자의 이 부분에서 예외가 발생하면 어떻게 될까요?
```c++
    if (audioClipFileName != "") {
        theAudioClip = new AudioClip(audioClipFileName);
    }
```
* 여기서 예외가 발생할 가능성이 있는 이유는 operator new가 AudioClip 객체에 대한 메모리 할당에 실패할수 있기때문입니다.
* theAudioClip 이 가리키고자 했떤 객체를 생성하려다 예외가 발생하면 theImage 가 가리키는 객체는 어느쪽에서 삭제해야할까?
    * BookEntry 소멸자에서 삭제해야하는데, 이 경우 소멸자가 절대로 호출되지 않습니다.
* C++은 생성이 완료된(fully constructed) 객체만을 안전하게 소멸시킵니다(소멸자를 호출합니다). 
    * 그런데, 생성자가 실행을 마치기 전에는 생성 작업이 완료되었다고 간주하지 않습니다. 따라서, 어떤 객체의 생성자가 호출되어 객체가 생성되는 도중 예외가 발생하면 그 객체의 소멸자는 호출되지 않습니다. 
    * 포인터가 아닌 멤버의 경우에는, 클래스 생성자가 호출되기 전에 이미 데이터 멤버의 초기화가 이루어지기 때문에, 클래스 생성자의 몸체가 실행될 때는 이미 초기화(생성)가 완료되어 있습니다. 
    * 따라서 클래스가 소멸될 때 자동으로 소멸되므로 별도의 조치가 필요하지 않습니다.
```c++
void testBookEntryClass()
{
    BookEntry b("addion", ""test);
    ...
}
```
* b가 생성도중 예외가 발생했다고 하면 b의 소멸자는 불리지 않는다.
```c++
// 심지어 다음과 같은 경우에도 BookEntry의 소멸자는
// 호출되지 않습니다.
 
void testBookEntryClass(void)
{
    BookEntry *pb = 0;
    
    try {
        pb = new BookEntry("Addison-Wesley Publishing Company",
                        "One Jacob Way, Reading, MA 01867");
        ...
    }
    catch (...) {                // 모든 예외를 받아 처리합니다.
        
        delete pb;                // 예외가 발생했을 때
                                // pb를 삭제합니다.
        
        throw;                    // 이 함수를 호출한 곳으로
    }                            // 예외를 전파합니다.
    
    delete pb;                    // 정상적으로 pb를 삭제합니다.
};
 
// new 연산자가 메모리를 할당하지 못하여 예외가 발생한 경우,
// pb에 메모리의 주소가 대입조차 되지 않습니다.
// 따라서 catch 블록의 delete 연산자는 널 포인터를
// 해제(아무 일도 하지 않음)하고 pb의 소멸자는
// 호출되지 않습니다.
```
* C++ 에선 어찌보면 당연하게도 생성이 완료되지 않는 객체에 대해선 소멸자 호출을 거부합니다. 
    * 이런 객체에 대해선 사용자가 직접 마무리 해야 합니다. 
* 간단한 해결 방법 중 하나는, 
    * 가능한 모든 예외를 받고(catch) 마무리 코드를 작성한 다음, 받은 예외를 다시 전파하는 것입니다.
    * 마무리 코드가 소멸자의 코드와 비슷하기 때문에 공통된 부분은 별도의 도우미 함수로 만들고, 소멸자와 생성자에서 이 함수를 호출하도록 만들 수 있습니다.
```c++
BookEntry::BookEntry(const string& name,
                     const string& address,
                     const string& imageFileName,
                     const string& audioClipFileName)
: theName(name), theAddress(address),
theImage(0), theAudioClip(0)
{
    try {                        // 새로 만들어진 try 블록
        if (imageFileName != "") {
            theImage = new Image(imageFileName);
        }
        
        if (audioClipFileName != "") {
            theAudioClip = new AudioClip(audioClipFileName);
        }
    }
    catch (...) {                // 모든 예외를 받습니다.
    
        delete theImage;        // 필요한 마무리 동작을
        delete theAudioClip;    // 취합니다.
        
        throw;                    // 받은 예외를 전파합니다.
    }
}
 
// catch 블록에 쓰인 문장이 BookEntry의 소멸자와
// 거의 똑같습니다. 즉 코드가 중복되고 있죠.
 
// 이것을 해결하기 위해 중복된 코드를 뽑아다가
// 별도의 보조 함수안에 넣고, 생성자와 소멸자가
// 이것을 호출하도록 만듭시다.
 
class BookEntry {
public:
    ...                            // 이전과 같습니다.
private:
    ...
    void cleanup(void);            // 공통적인 마무리 작업용 함수
};
 
void BookEntry::cleanup(void)
{
    delete theImage;
    delete theAudioClip;
}
 
BookEntry::BookEntry(const string& name,
                     const string& address,
                     const string& imageFileName,
                     const string& audioClipFileName)
: theName(name), theAddress(address),
theImage(0), theAudioClip(0)
{
    try {
        ...                        // 이전과 같습니다.
    }
    catch (...) {
        cleanup();                // 리소스를 해제합니다.
        throw;                    // 예외를 전파합니다.
    }
}
 
BookEntry::~BookEntry(void)
{
    cleanup();
}
```
* 이보다 더 좋은 해결책은 항목 9에서 제시한 방법을 따라, 자원 관리 객체를 이용하는 것입니다. 다시말해 위에서 살펴본 예제에서 theImage와 theAudioClip을 원시 포인터 타입 대신 auto_ptr로 바꾸어도 큰 문제가 없을 것입니다.
```c++
class BookEntry {
public:
    ...
private:
    ...
    const auto_ptr<Image> theImage;
    const auto_ptr<AudioClip> theAudioClip;
};
 
BookEntry::BookEntry(const string& name,
                     const string& address,
                     const string& imageFileName,
                     const string& audioClipFileName)
: theName(name), theAddress(address),
theImage(imageFileName != ""
        ? new Image(imageFileName)
        : 0),
theAudioClip(audioClipFileName != ""
            ? new AudioClip(audioClipFileName)
            : 0)
{}
 
// theAudioClip을 초기화 하다가 예외가
// 발생하더라도 theImage는 이미 생성 과정이
// 완료된 객체이기 때문에, 자동으로 소멸됩니다.
 
// 게다가, theImage와 theAudioClip은
// 엄연한 객체이기 때문에 BookEntry가 소멸될때
// 자동으로 소멸됩니다. 따라서 BookEntry의
// 소멸자가 할 일이 없어집니다.
 
BookEntry::~BookEntry(void)
{}                // 할 것이 젼혀 없네요!
```
* 한방에 정리를 하면
    * 포인터 클래스를 auto_ptr 버전으로 바꾸면 생성자는 실행도중에 예외가 발생해도 리소스 누수가 없이 깔끔히 처리할수있어요.


## 항목 11 : 소멸자에서는 예외가 탈출하지 못하게 하자
* 클래스가 소멸자가 호출되는 경우는 두가지로 나눌수있다
    * 객체가 통상적인 조건에서 소멸되는것
        * 유효범위를 벗어 났을때와 객체가 직접 삭제(delete)될때
    * 예외 처리 매커니즘에 의해 객체가 소멸되는것
        * 예외 전파(exception propagation) 과정의 일부분으로 스택 되감기가 진행될때
* 어떤 예외에 대한 처리가 진행되고 있는 동안 또 다른 예외 때문에 프로그램 흐름이 소멸자 함수를 떠나면, C++는 terminate란 함수를 호출하게 되어 있습니다. 이 함수가 하는 일은 이름 그대로 프로그램의 실행을 끝장내는(terminate) 것입니다. 게다가 아주 칼같이 끝내 버리기 때문에, 지역 객체조차도 소멸되지 않습니다. 이러한 이유로 소멸자에서는 예외가 탈출하지 못하도록 만들어야 합니다.
* 온라인으로 컴퓨터의 사용자 접속 상황을 로깅하는 용도로 만든 Session 이란 클래스가 있다고 가정합니다.
```c++
class Session {
public:
    Session(void);
    ~Session(void);
    ...
private:
    static void logCreation(Session *objAddr);
    static void logDestruction(Session *objAddr);
};
 
// logCreation과 logDestruction 함수는
// 객체 생성과 소멸 시기를 기록해 놓는 데에 사용합니다.
// 따라서 소멸자는 다음과 같이 생겼을 것입니다.
 
Session::~Session(void)
{
    logDestruction(this);
}
 
// 하지만 logDestruction 함수에서
// 예외가 발생할 수 있다면 앞에서 설명한
// 이유로 인해 문제가 될 수 있습니다.
 
// 한 가지 해결 방법은 try-catch 블록을
// 사용하는 것입니다.
 
Session::~Session(void)
{
    try {
        logDestruction();
    }
    catch (...) {
        cerr << "Unable to log destruction of Session object "
             << "at address "
             << this
             << ".\n";
    }
}
 
// 하지만 operator<< 중의 하나에서 예외가
// 발생할 수 있기 때문에 이 구현 역시 그다지
// 좋지 못한 구현이라고 할 수 있습니다.
 
// 이런 경우 logDestruction이 예외를
// 일으키면 그냥 로깅을 하지 않는 편이 바람직합니다.
 
Session::~Session(void)
{
    try {
        logDestruction();
    }
    catch (...) {}        // 이 catch 문이 예외가 소멸자 밖으로 빠져나가지 못하게 합니다.
}

```
* catch 블록이 아무것도 안하는 것처럼 보이지만 이 블록은 logDestruction에서 발생한 예외를 Session 의 소멸자에 묶어두는 중요한 역할을 하고 있습니다.
* 예외가 소멸자를 떠나 전파되도록 내버려 두는 일이 나쁜 이유가 하나 더 있습니다.
    *  소멸자에서 발생된 예외가 소멸자에서 처리되지 않으면, 소멸자는 실행이 끝나지 않은 상태로 남습니다
        *  (실제로, 예외가 발생된 그 지점에 딱 서 버리죠). 
    *  실행을 마치지 못한 소멸자는 자신에게 주어진 역할을 완수하지 못합니다
```c++
// 예를 들어 세션 생성과 함께
// 데이터베이스 트랜잭션을 시작하고, 세션이
// 소멸될 때 데이터베이스 트랜잭션을 끝내도록
// Session 클래스가 수정되었다고 가정해 봅시다.
 
// 생성자와 소멸자는 다음과 같이 생겼을 것입니다.
 
Session::Session(void)        // 주변 상황을 간단히 하기 위해
{                            // 이 생성자는 예외 처리를 하지 않도록
                            // 하였습니다.
    logCreation(this);
    startTransaction();        // DB 트랜잭션을 시작합니다.
}
 
Session::~Session(void)
{
    logDestruction(this);
    endTransaction();        // DB 트랜잭션을 끝냅니다.
}
 
// 여기서 logDestruction이 예외를 일으키면
// Session 생성자에서 시작된 데이터베이스
// 트랜잭션은 끝나지 않은 채로 남습니다.
 
// 함수 호출 순서를 바꾸는 것은 의미가 없습니다.
// endTransaction도 예외를 일으킬 수
// 있기 때문입니다.
 
// 결국 try-catch 블록을 이용하는 수 밖에는
// 답이 없습니다.
```
* 예외가 소멸자를 빠져나가지 못하게 해야하는 이유 두 가지
    * 1. 예외 전파의 일부분으로 진행되는 스택 되감기 동작 중에 terminate 가 호출되는것을 막기 위함
    * 2. 소멸자의 동작을 완전히 끝내도록 하기 위함임



## 항목 12 : 예외 발생이 매개변수 전달 혹은 가상 함수 호출과 어떻게 다른지를 이해하자
* 함수 매개변수를 선언하는 문법과 catch 문을 보면 거의 다른 점을 찾을 수 없음
```c++
class Widget{...};

void f1(Widget w);//모든 매개변수가 Widget과 관련되어있음.
void f2(Widget& w);
void f3(const Widget& w);
void f4(Widget *pw);
void f5(const Widget *pw);

catch (Widget w) ...//모든 catch 문이 Widget과 관련되어ㅣ 있음
catch (Widget& w) ...
catch (const Widget& w) ...
catch (Widget *pw) ...
catch (const Widget *pw) ...
```
* 유사점과 무시할 수 없는 차이점도 숨어 있습니다.
* 유사점부터 살펴볼께요
    * 매개변수와 예외는 값에 의한 전달, 참조에 의한 전달, 포인터에 의한 전달이 모두 가능
        * 하지만 전달할 때 발생하는 일은 근본적으로 다름니다. 
            * 함수를 호출했을때에는 프로그램의 흐름이 함수를 호출한 부분으로 돌아오지만, 예외는 프로그램의 흐름이 throw를 호출한 부분으로 돌아오지 않음
* 다음의 함수는 Widget 타입의 객체를 매개변수로서 전달하기도 하고 예외로서 발생시키기도 하는 그런 함수 입니다.
```c++
//-----------------------------------------------------------------------------
// 스트림으로부터 Widget의 값을 읽는 함수
istream operator>>(istream& s, Widget& w);
 
void passAndThrowWidget(void)
{
    Widget localWidget;
    
    cin >> localWidget;    // operator>>에 localWidget을 전달합니다.
    
    throw localWidget;    // localWidget을 예외로 발생시킵니다.
}
 
// 함수 호출과 달리 예외는 catch된 후,
// throw한 위치로 되돌아 오지 않습니다.
 
// 때문에 localWidget은 던져진 후
// passAndThrowWidget의 범위에서
// 벗어나므로 소멸되어 버립니다.
 
// 따라서 예외를 던질 때에는 그 객체가 아닌
// 객체의 복사본이 전달됩니다. 설령
 
// catch에서 참조자로 객체를 받는다고 해도
// 마찬가지입니다.
 
// 이것은 지역 객체가 아니더라도 마찬가지로
// 적용됩니다.
 
void passAndThrowWidget(void)
{
    static Widget localWidget;
    
    cin >> localWidget;
    
    throw localWidget;
}
 
// localWidget이 정적 객체가 되어서
// 프로그램이 종료될 때 까지 소멸되지 않지만
// 여전히 예외 발생시에는 localWidget의
// 복사본이 던져집니다.
 
 
//-----------------------------------------------------------------------------
// C++의 객체 복사는 항상 정적 타입에 따라
// 복사 생성자가 호출됩니다.
 
// 예를 들면
class Widget { ... };
class SpecialWidget: public Widget { ... };
void passAndThrowWidget(void)
{
    SpecialWidget localSpecialWidget;
    
    ...
    
    Widget& rw = localSpecialWidget;    // rw는
                                        // SpecialWidget를 참조합니다.
    
    throw rw;                            // 이 throw 문으로 발생되는
                                        // 예외의 타입은 다름 아닌
}                                        // Widget입니다!!
 
// 이렇게 rw와 같이 기본 클래스 타입의
// 참조자에 담긴 객체가 파생 클래스 타입이더라도
// rw를 통해 호출되는 복사 생성자는
// rw를 Widget타입의 객체로 간주합니다.
 
// 객체의 동적 타입에 따라 복사할 수 있는
// 방법도 있는데 이는 항목 25를 참고하세요.
 
 
//-----------------------------------------------------------------------------
// "예외는 원래 객체의 사본으로서 발생된다"는
// 사실 때문에, catch 블록에서 예외를 전파하는
// 방법에도 신경쓸 수 밖에 없게 됩니다.
 
// 다음 두 catch 블록은 똑같은 일을 할 것
// 같지만 실은 두번째 경우에는 쓸데없는
// 객체의 복사가 한 번 더 일어납니다.
 
catch (Widget& w)        // Widget& 예외를 받습니다.
{
    ...                    // 이 예외를 처리합니다.
    throw;                // 이 예외 자체를 "그대로" 중계하여
}                        // 예외 전파가 이루어져 있도록 합니다.
 
catch (Widget& w)        // Widget& 예외를 받습니다.
{
    ...                    // 이 예외를 처리합니다.
    throw w;            // 처리한 예외의 "사본"을 발생시켜
}                        // 예외 전파가 이루어지도록 합니다.
 
// 불필요한 객체의 복사 외에도 차이점이 있습니다.
 
// 첫번째 경우에는 w에 담긴 객체가 SpecialWidget인
// 경우 그대로 SpecialWidget이 예외로 전파됩니다.
 
// 반면 두번째 경우에는 예외를 전파하면서 w의 사본을
// 만들게 되므로, 전파되는 예외의 타입은 Widget이
// 됩니다.
 
 
//-----------------------------------------------------------------------------
// 정리해 봅시다. Widget 객체를 받아 내도록
// 선언할 수 있는 catch 구문은 세가지가 있습니다.
 
catch (Widget w) ...        // 값에 의한 예외 처리
catch (Widget& w) ...        // 참조에 의한 예외 처리
catch (const Widget& w) ...    // 상수 객체에 대한 참조자
                            // (상수 참조자)에 의한 예외 처리
 
// 우선 예외로서 발생되는 객체(사본으로 만들어진 임시 객체)
// 는 굳이 상수 참조자로서 넘어갈 필요가 없습니다.
 
// 비상수 참조자에 대한 임시 객체는 함수 호출 시의
// 매개변수로 사용될 수 없지만(항목 19 참조),
// 예외의 경우에는 가능합니다.
 
// 함수 호출에서 값에 의한 전달의 경우 매개변수 전달 시
// 객체의 복사가 일어납니다. 이것은 예외 전달 시에도
// 마찬가지로 적용됩니다.
 
// 즉 값에 의한 전달로 예외를 받게 되면,
// 객체를 던질 때 그 객체의 복사본을 만들어 던지고
// 받을때도 값에 의한 전달로 인해 복사가 일어나므로
// 객체의 복사가 총 두 번 일어나게 됩니다.
 
 
//-----------------------------------------------------------------------------
// 포인터를 예외로 받는 경우에는
// 함수 호출에서의 포인터에 의한 전달과
// 마찬가지로 포인터의 사본(주소값)이 전달됩니다.
 
// 하지만 포인터는 이 예제 처음에서 보았듯이
// catch쪽으로 보내봐야 이득이 없습니다.
 
// 어차피 예외에 의해 프로그램 흐름이
// 지역 객체의 유효범위를 떠나면 그
// 지역 객체는 소멸되니까요.
 
// 굳이 포인터를 써야 겠다면 new연산자로
// 할당한 객체를 던지고, catch 블록에서
// delete 해 줘야 합니다.
```
* 예외 객체는 항상 복사됩니다. 특히 값에 의한 예외 처리의 경우에는 두 번 복사됩니다(함수 매개변수로 전달되는 객체보다 한번 더 복사가 일어납니다).
* 예외로써 발생될때에는
    * 사본이 만들어진 후에 catch 문으로 넘겨집니다. 
        * 그럴수 밖에 없는 것이 유효범위를 떠나기 때문에 객체의 소멸자가 당연히 호출될 것이기 때문이다. 
            * 이렇기에 C++에서는 예외로 발생되는 객체에 대해서는 사본을 만들도록 한 것이다.
* 예외 발생을 위해 객체가 복사될때 이 복사는 그 객체의 복사 생성자에 의해 이루어진다
* 예외는 원래 객체의 사본으로서 발생된다는 사실 때문에 cathch 블록에서 예외를 전파하는 방법에도 신경을 쓸 수밖에 없다. 
* 발생되는 예외는 전달되는 매개변수에 비해 타입변환을 거치는 가지수가 적습니다. 예외에서 타입변환은 파생 클래스에서 기본 클래스로의 변환과, 타입이 있는 포인터에서 타입이 없는 포인터(void*)로의 변환만 이루어 집니다.
```c++
// 함수 호출의 경우 다음 문장은
// 아무런 문제를 일으키지 않습니다.
 
double sqrt(double);
 
int i;
double sqrtOfi = sqrt(i);
 
// int 타입 i가 double 타입으로
// 암시적으로 변환이 일어납니다.
 
// 하지만 예외를 전달할 때는 좀 다릅니다.
 
void f(int value)
{
    try {
        if (someFunction()) {    // someFunction()이 true를 반환하면
            throw value;        // int 타입의 예외를 발생시킵니다.
        }
        ...
    }
    catch (double d) {            // 여기서는 double 타입의 예외를
        ...                        // 처리합니다.
    }
}
 
// 이 경우 double 타입의 예외를
// 받는 catch 블록은 int 타입의
// 예외를 받지 못합니다.
 
// 예외 전달시 타입 변환은 두가지만 허용됩니다.
 
// 첫번째는 상속 기반의 변환입니다.
// 기본 클래스 타입의 객체를 예외를 받는
// catch 블록은 파생 클래스 타입의
// 객체를 예외로 받을 수 있습니다.
// 예를 들어 다음 catch 블록은
 
catch (exception& e) { ... }
 
// exception으로부터 파생된 모든
// 타입의 객체를 잡아냅니다.
 
// 두번째는 타입이 있는 포인터로부터
// 타입이 없는 포인터(void*)로 바꾸는
// 경우입니다.
// 예를 들어 다음 catch 블록은
 
catch (const void*) { ... }
 
// 포인터이기만 하면 무조건 잡아냅니다.

```
* 매개변수 전달과 예외 전파 사이의 찾을수있는 세번째 차이는
    * catch 문은 소스 코드에 등장한 순서대로 동작하고, 예외를 처리할 수 있는 catch 문 중 가장 첫째 것이 선택됩니다. 하지만 어떤 객체를 통해서 가상 함수를 호출할 때에는, 소스 코드에 등장한 순서와 관계 없이 그 객체의 동적 타입에 가장 가까운 클래스에 정의된 가상 함수가 호출됩니다.
```c++
// catch 문은 소스코드에 등장한 순서대로
// 사용됩니다. 예를 들면
 
try {
    ...
}
catch (logic_error& ex) {        // 이 블록은 logic_error 타입의
    ...                            // 예외를 모두 도맡아 처리합니다.
}                                // 심지어 이 클래스의 파생 클래스
                                // 타입의 예외까지 처리합니다.
 
catch (invalid_argument& ex) {    // 이 블록은 절대로 실행되지
    ...                            // 않습니다. 왜냐하면
}                                // invalid_argument 타입의 예외가
                                // 처리되는 블록은 바로 위의
                                // catch 블록이기 때문입니다.
 
// ※ invalid_argument 는 logic_error 로부터
//    파생되었습니다.
 
// 위와 같이 코드를 짜면 컴파일러는 경고 혹은 오류를
// 냅니다. 위의 코드를 정상적으로 작동하도록 수정하면
// 다음과 같이 될 것입니다.
 
try {
    ...
}
catch (invalid_argument& ex) {    // invalid_argument 예외를
    ...                            // 여기서 처리합니다.
}
catch (logic_error& ex) {        // 모든 logic_error 예외를
    ...                            // 여기서 처리합니다.
}

```
* 정리하면 함수에 객체를 전달하는 것, 혹은 가상 함수를 호출하는 것과 객체를 예외로서 발생시키는 것의 차이는
    * 1. 예외 객체는 항상 복사됨
        * 특히 값에 의한 예외처리는 두번 복사됨.
        * 함수 매개변수로 전달되는 객체는 복사되지 않음
    * 2. 발생되는 예외는 전달되는 매개변수에 비해 타입변환을 거치는 가지수가 적음
    * 3. catch 문은 소스 코드에 등장한 순서대로 동작하고, 예외를 처리할 슁ㅆ는 catch 문 중에 가장 첫번째 것이 선택됨.
        * 하지만 어떤 객체를 통해서 가상함수를 호출할 때에는 소스코드에 등장한 순서와 상관없이 그 객체의 동접 타입에 가장 가까운 클래스에 정의된 가상함수가 호출됨.


## 항목 13 : 발생한 예외는 참조자로 받아내자
* 이론상, 예외 객체를 catch 문으로 전달하는 데 있어서는 포인터에 의한 예외받기가 가장 효율적이어야 합니다(항목 15를 보세요). 왜냐하면, 예외를 포인터로 발생시키면 객체의 복사 없이도 전달이 이루어지기 때문입니다(항목 12에서 이야기 했습니다).
* 예를 들면.
```c++
class exception{...};//표준 C++ 라이브러리의 exception 클래스 계통에서 따왔음.

void someFuntion()
{
    static exception ex;//exception 객체
    ...
    throw &ex;//ex에 대한 포인터를 발생시킵니다.
    ...
}

void doSomething()
{
    try{
        someFunction();//exception*를 발생시킬 수 있습니다.
    }
    catch (exception *ex){//exception*를 받습니다. 객체의 복사는 일어나지 않습니다.
        ...
    }
}
```
* 위의 예처럼 정적 객체나 전역 객체가 안된다는 것은 아니지만 아래와 같은 코드가 나와선 안됩니ㅏㄷ.
```c++
void someFunction()
{
    exception ex;//지역변수로 선언된 exception 객체, 함수의 유효범위가 끝나면 이 객체는 자동소멸됨
    
    ...
    throw &ex;//막 소멸되려고하는 객체에 대한 포인터를 예외로 발생시킴
    ...
}
```
* 다른 대안으로 햅 객체를 새로 만들고 그 객체의 포인터를 예외로 발생시키는 것임
```c++
void someFunction()
{
...
    throw new exception;//새로운 힙 기반 객체에 대한 포인터를 예외로 발생(opertor new 가 예외를 발생시키지 않도록 바래야함.)
...
}
```

* 하지만 포인터로 예외를 전달하는 경우, 비정적 지역 객체의 포인터를 전달할 수 있는 문제가 발생합니다. 비정적 지역 객체의 포인터를 받을 시점에는 이미 그 포인터가 가리키는 객체는 소멸되었을 것입니다. 이를 해결하기 위해서는 예외를 위한 정적 지역 객체(혹은 전역 객체)를 만들고 이 정적 지역 객체의 주소를 예외로 전달하거나, new 연산자로 예외 객체를 생성하여 전달해야 합니다. 하지만 이 경우 catch 블록에서 delete를 해야 할지 말아야 할지 또다시 고민해야 하는 문제가 발생합니다. catch 블록에서는 전달받은 포인터가 가리키는 객체가 new 연산자로 생성한 객체인지, 정적 지역 객체 혹은 전역 객체인지를 알 방법이 없습니다. 따라서 포인터로 예외를 전달하는 것은 그렇게 좋지 못한 생각입니다.

* 다음은 포인터를 전달하지 않는 방법으로는 '값에 의한 예외받기(catch-by-value)' 가 있습니다. 
    * 하지만 값으로 예외를 전달하면 전달한 객체가 두 번이나 복사되어 비효율적이며, 공포의 슬라이스 문제(slicing problem)가 도사리고 있습니다.
        * 슬라이스 문제: 발생 시에는 파생 클래스의 객체였다가, 기본 클래스의 객체를 받는 catch 문에 들어가면서 파생 클래스 부분에 추가되었던 데이터가 싹둑 잘려져 나가는(sliced off) 현상으로, 파생 클래스의 객체가 기본 클래스의 객체로 복사되면서 발생한다.
```c++
class exception{//이전과 같이 표준 exception 클래스임
public:
    virtual consot char* what() throw();//예외에 대한 간단한 메시지 반환합니다. 
    ...
};

class runtime_error:public exception{...};//표준 exception 클래스 계통에서 따온것임

class Validation_error: public runtime_error{//이것은 사용자가 추가한 클래스
public:
    virtual const char * what() throw();//이 함수는 앞의 exception 클래스에서 정의한 함수를 재정의한 것임
    ...
};
void someFunction()//유효성검사 예외(validation exception)를 일으킬수 있음
{
...
    if(유효성 검사가 실패햇을 경우){
        throw Validation_error();
    }
...
}
void doSomething()
{
    try{
        someFunction();//유효성 검사 예외(validation exceptio)를 일으킬수 있음
    }
    catch (exception ex){//표준 예외 클래스 계통을 따르는 모든 예외 객체를 받아 처리함
        cerr << ex.what();//exception::what()을 호출하고 Validation_error::what()은 절대로 호출하지 않음.
        ...
    }
}
//발생된 예외 타입이 Validation_error이고 Validation_error에서 가상 함수 what을 재정의했더하더라도 호출되는 what은 Validation_error 것이 아니라 기본 클래스의 것입니다. C++에서 슬라이스 문제는 ...끝까지 여러분을 괴롭힙니다. 
```

* 마지막으로 남은 방법은 '참조자에 의한 예외받기(catch-by-reference)' 입니다. 
    * 참조자로 예외를 받으면, catch 블록에서 객체를 delete 해야 할지를 고민할 필요도 없고, C++ 표준 예외를 처리하는 데에도 전혀 무리가 없으며, '값에 의한 예외받기' 와 달리 슬라이스 문제도 없고 예외 객체는 한 번만 복사됩니다. 
    * 마지막 예를 볼께요
```c++
void someFunction()
{
...
    if(유효성 검사가 실패햇을 경우){
        throw Validation_error();
    }
    ...
}

void doSomething()
{
    try{
        someFunction();//아무것도 안바뀜
    }
    catch (exception& ex){//이곳이 바뀜, 값에 의한 예외받기가 찹조자에 의한 예외받기로 바뀜
        cedrr << ex.what();//이제 Validation_error::what()를 호출하고, exception::what()을 호출하지 않음
        ...
    }
}

```
* catch 문에 &a만 추가한 것이 전부이지만 효과는 막대합니다.
    * catch 블록에 있는 가상 함수의 호출이 우리가 원하는 대로 됩니다.





## 항목 14 : 예외 지정(exception specification) 기능은 냉철하게 사용하자
* C++ 에도 함수를 선언할때 그 함수가 발생시킬 예외를 미리 지정할수 있는 기능이 있다. 
    * 이것을 예외 지정(exception specification)이라고 한다. 
        * 어떤 예외를 발생시킬지드러나기에 코드가 보기 수월해진다. 
        * 예외 지정에 일관성이 없으면 컴파일러가 발견도 해준다
        * 함수가 예외 지정 리스트에 없는 예외를 발생시킬 경우 런타임 에러가 발생하면서 unexpceted 라는 특수 함수가 자동 호출 됩니다.
    * 하지만 unexpected 함수의 기본 동작은 terminate 를 호출하는 것인데, 항목 11에서 이미 본 바 있는 이 함수는 기본적으로 abort를 호출합니다. 
        * 이렇게 예외 지정을 어긴 프로그램은 기본적으로 바로 멈추어 버리고, 활성 스택 프레임에 만들어진 지역 변수는 없어지지 않은 채로 졸지에 구천을 해매야 합니다. 
        * 결국 개발자 입장에서 예외 지정을 어기는 일은 재앙을 부르는 일이나 마찬가지입니다.
    * 하지만 예외 지정을 어기는 함수를 만드는 일은 제법 쉽습니다. 
        * 예를 들어 예외 지정된 함수 내에서 예외 지정되지 않은 함수(예외를 지정하지 않는 것은 어떤 예외든 발생시킬 수 있음을 의미합니다)를 호출했을 때, 예외 지정되지 않은 함수에서 예외가 발생하여 예외 지정을 어길 수 있습니다. 
        * 예외 지정된 함수가 예외 지정되지 않은 함수를 호출하는 것이 문법적으로 적법한 이유는, 예외 지정된 새 코드와 예외 지정되지 않은 옛날의 코드가 섞일 수 있기 때문입니다.
* 한 예를 살펴볼께요
```c++
extern void f1();//무엇이든 발생시킵니다.
//다음은 예외 지정된 함수입니다. f2를 예외 지정을 통해 int타입의 예외만을 발생시키도록 설정됩니다. 
void f2() throw(int);
//f1, f2 는 C++에서 허용되는 함수선언인데, f1은 f2의 예외 지정을 어길수있는 예외를 발생시킬 수 있음
void f2() throw(int)
{
    ...
    f1();//f1은 int 이외 타입의 얘외를 발생시킬 수 있지만 어쨋든 문법에 맞습니다.
    ...
}
```
* 설령 어떤 함수를 호출하는 함수의 예외 지정과 호출되는 함수의 예외 지정이 일치하지 않난다고해소 컴파일러는 별소리는 안하지만, 그 프로그램은 바로 끝나버릴수 있기에 이러한 예외 지정 불일치가 최소화된 프로그램을 만들도록 신경써야 한다. 
* 예외 지정 불일치 피하기의 첫걸음은 (보통 타입 인자를 받아들이는)템플릿에 예외 지정을 두지 않는 것입니다. 템플릿의 타입 매개변수에서 발생된 예외에 대해 알 방법이 전혀 없기 때문입니다.
```c++
// 예외 지정에 대해 별로 신경쓰지 않고 편치 않게 설계한 템플릿
template <typename T>
bool operator==(const T& lhs, const T& rhs) throw()
{
    return &lhs == &rhs;
}
 
// 언뜻 보면 정말로 어떤 예외도 발생시키지 않을 것 같지만
// T로 넘어간 타입이 operator& 함수를 오버로딩
// 하고 있고 이곳에서 예외를 던질 수도 있기 때문에
// 위의 경우에도 예외 지정을 어길 가능성이 있습니다.
```
* 템플리시과 예외 지정은 섞여선 안됨.
* unexpected가 호출되는 것을 막을 수 있는 둘째 걸음은, 예외 지정이 안 된 함수를 호출할 가능성을 가진 함수에는 예외 지정을 두지 않는 것입니다. 지극히 상식적인 아이디어라고 생각할지도 모르지만, 아주 쉽게 잊는 경우가 많습니다. 특히, 콜백 함수를 등록하도록 할 때 그렇습니다.
```c++
// 어떤 윈도 시스템에 이벤트가 발생했을 때,
// 윈도우 시스템 콜백을 위한 함수 포인터
typedef void(*CallBackPtr)(int eventXLocation,
                        int eventYLocation,
                        void *dataToPassBack);
 
// 윈도우 시스템 개발자가 등록한 콜백 함수를
// 담아두는 윈도우 시스템 클래스
class CallBack {
public:
    CallBack(CallBackPtr fPtr, void *dataToPassBack)
    : func(fPtr), data(dataToPassBack) {}
    
    void makeCallBack(int eventXLocation,
        int eventYLocation) const throw();
 
private:
    CallBackPtr func;    // 콜백이 일어날 때
                        // 호출되는 함수
    
    void *data;            // 콜백 함수로 넘겨지는
};                        // 데이터
 
// 콜백을 구현하기 위해서 이벤트가 발생한 좌표와
// 등록한 데이터를 사용하여 등록된 콜백 함수를 호출합니다.
void CallBack::makeCallBack(int eventXLocation,
        int eventYLocation) const throw()
{
    func(eventXLocation, eventYLocation, data);
}
 
// 여기서 makeCallBack 안에서 호출되는 func는
// 예외 지정을 어길 위험을 안고 있습니다.
 
// 이것은 typedef 타입인 CallBackPtr에
// 예외 지정을 해둠으로써 해결됩니다.
 
typedef void(*CallBackPtr)(int eventXLocation,
                        int eventYLocation,
                        void *dataToPassBack) throw();
 
// 이렇게 해 두면, 예외를 발생시키지
// 않는다는 보장이 안 되어 있는 콜백
// 함수를 등록하려고 하면 컴파일러가
// 에러를 냅니다.
 
// 예외 지정되지 않은 콜백 함수
void callBackFcn1(int eventXLocation, int eventYLocation,
                void *dataToPassBack);
 
void *callBackData;
 
...
 
CallBack c1(callBackFcn1, callBackData);
                        // 에러입니다! callBackFcn1
                        // 이 예외를 일으킬 수 있기 때문입니다.
 
// 예외 지정된 콜백 함수
void callBackFcn2(int eventXLocation,
                int eventYLocation,
                void *dataToPassBack) throw();
 
CallBack c2(callBackFcn2, callBackData);
                        // 문제없습니다. callBackFcn2은
                        // 예외 지정을 따르고 있습니다.
 
// 하지만 이 방법은 상당수의 컴파일러가
// 지원하는데도 불구하고, C++ 표준화
// 위원회에서는 "예외 지정은 typedef에
// 나오면 안 됩니다" 라고 정해버려서
// 이식성이 떨어집니다.
 
// 이식성을 살리고 싶다면 CallBackPtr을
// 매크로로 만들 수밖에 없습니다.
```
* unexpected가 불리지 않도록 하는 세 번째 방법은 "시스템"이 일으킬 가능성이 있는 예외(C++ 표준 예외라고도 합니다)를 처리하는 것입니다.
    * 이런 예외 중 가장 흔한 것이 bad_alloc인데, 이 예외는 메모리 할당에 실패한 operator new와 operator new[]가 발생시킵니다(항목 8 참조). 어느 함수에서든 new 연산자를 쓸 때에는 이 함수에서 bad_alloc 예외가 꿈틀거릴 수 있다는 생각을 염두해 두시기 바랍니다.
* 예기치 않은 예외를 막는 것이 어처구니 없이 비실용적이라면, "C++는 예기치 않은 예외를 다른 타입의 예외로 대체할 수 있는 장치를 마련해 놓았습니다" 라는 사실을 이용해 볼 수 있습니다.
    * set_unexpected 함수를 사용하여 unexpected 함수를 교체할 수 있습니다.
```c++
//-----------------------------------------------------------------------------
class UnexpectedException {};        // 모든 예기치 않은 예외는
                                    // 이 타입의 객체로 대체
                                    // 합니다.
 
void convertUnexpected(void)        // 예기치 않은 예외가
{                                    // 발생되었을 때 호출되는
    throw UnexpectedException();    // 함수
}
 
set_unexpected(convertUnexpected);
 
// 이렇게 하면, 이젠 예기치 않은 예외가 발생될 때마다
// convertUnexpected가 호출되고, 어디서
// 굴러먹다 온 건지도 모르는 이 예외는 throw에 의해
// UnexpectedException이라는 새로운
// 타입의 예외로 탈바꿈합니다.
 
// 예외 지정 리스트에 UnexpectedException이
// 들어 있었다면, 예외 지정이 항상 만족되어 왔던 것처럼
// 예외 전파가 계속됩니다.
 
// 예외 지정 리스트에 UnexpectedException이
// 없으면, unexpected 함수를 바꾸지 않은 상황과
// 똑같이 가차 없이 terminate가 호출됩니다.
 
 
//-----------------------------------------------------------------------------
// 예기치 않은 예외를 다른 잘 알려진 타입으로 바꾸는
// 방법이 하나 더 있습니다.
 
void convertUnexpected(void)        // 예기치 않은 예외가 발생할 때
{                                    // 호출되는 함수입니다. 여기서는
    throw;                            // 현재의 예외를 그대로
}                                    // 중계하기만 합니다.
 
set_unexpected(convertUnexpected);    // convertUnexpected 함수를
                                    // unexpected 대신 호출되는 함수로
                                    // 세팅합니다.
 
// 이렇게 하면 예기치 않은 예외는 표준 타입인
// bad_exception 객체로 바뀝니다.
 
// 즉 이렇게 한 후에 bad_exception
// 혹은 이 객체의 기본 클래스인 exception
// 을 예외 지정 리스트에 포함시키면, 예기치
// 않은 예외가 발생해도 프로그램이 죽는 일은
// 피할 수 있습니다.
```
* 호출측 함수에서 예외 처리 코드가 준비된 상황에서조차도 unexpected가 호출될 수 있습니다.
```c++
class Session {
public:
    ~Session(void);
    ...
private:
    static void logDestruction(Session *objAddr) throw();
};
 
Session::~Session(void)
{
    try {
        logDestruction(this);
    }
    catch (...) {}
}
 
// 소멸자의 logDestruction함수를 호출하는
// 코드는 try 블록 내에 넣고, catch 블록에서는
// 어떤 예외든지 다 받아서 삼키도록 만들었습니다.
 
// 하지만 logDestruction 함수는 아무런 예외도
// 발생시키지 않도록 에외 지정이 되어 있습니다.
// 이렇게 만든 경우에도 logDestruction에서
// 예외가 발생하면 unexpected가 호출됩니다.
```

## 항목 15 : 예외 처리에 드는 비용에 대해 정확히 파악하자
* 실행중 예외처리를 위해 C++ 프로그램은 내부적으로 많은 정보를 유지한다. 
    * 모든 순간순간에 예외가 발생되었을때 소멸시켜야할 객체를 파악하고
    * try 블록으로 들어갈때와 빠져나갈때 상황을 점검하고
    * 각각의 try 블록마다 catch 문을 추적하고 이 catch 문이 처리할 수있는 예외를 점검하고
* 예외 지정 조건이 만족했는지에 대한 런타임 비교 동작에도 역시 비용이 들어갑니다.
    * 예외가 발생햇을때 try 블록에 있던 객체를 소멸시키고 적절한 catch 문을 찾아들어가는데도 비용이 듬
* 심지어 try, throw, catch 를 전혀 쓰지 않더라도 어느정도의 비용이 들어갑니다.

* 예외 처리 기능을 전혀 쓰지 않았을때의 비용을 살펴봅시다. 
    * 어떤 객체가 생성 과정을 완료했는지 체크하는데 내부적으로 사용되는 자료구조에 대한 메모리가 소모됩 그리고 이 자료구조를 업데이트 하는데 필요한 시간이 소모됨,
        * 사실 이 비용은 프로그램 전체의 수행 성능을 좌우할 정도는 아니고 적당한 수준임
        * 예외 처리 기능을 완전히 배제하고 컴파일한 프로그램은 예외 처리를 지원하도록 컴파일한 것에 비해 속도도 빠르고 크기도 작습니다.
    * 이론적으로 이 비용에 대한 대처 방안은 없음
        * 실제는 예외 처리를 지원하는 대부분의 C++ 컴파일러 제작사에서 예외 처리를 지원할 것인지의 여부를 사용자가 조정할 수 있는 장치를 만들어 놓았습니다.
        * 시간이 지나고 라이브러리가 예외를 사용하는 것이 점점 일반화되면서 이런 방법도 설득력을 조금씩 잃어가고 있지만, 예외 기능 없이 컴파일 하는 것이 코드 최적화에 상당한 효과를 줄 수 있습니다. 
        * 라이브러리를 만들 때에도 마찬가지입니다. 사용자 코드에서 발생된 예외가 라이브러리 쪽으로 전파되지 않도록 보장해줄 수 있다면, 예외 없이 만든 라이브러리가 훨씬 작고 빠릅니다. 
        * 하지만 이 보장이 그렇게 쉬운 것만은 아닙니다. 왜냐하면, 라이브러리에서 선언한 가상 함수에 대한 사용자의 재정의를 허용하면 안 되고, 콜백함수도 쓰게 해서는 안 되기 때문입니다.


* 예외 처리에 들어가는 두 번째 비용은 try 블록으로 인해 생기는 비용입니다. 이 비용은 try 블록이 소스에 들어가기만 하면, 무조건 지불해야 합니다. 이 비용을 줄이려면 쓸데 없이 try 블록을 남발하지 말아야 합니다.
    * 이 비용은 컴파일러마다 천차만별이긴 하지만 대략적으로 추정했을 때, try 블록을 썼을 경우 늘어나는 코드 크기와 실행 속도는 5-10% 정도입니다. 이 추정치는 예외가 전혀 발생되지 않았을 경우를 가정했을 때의 값, 즉 try 블록만 썼을 때의 값입니다.
* 예외 지정(exception specification) 기능에 대해서도 try 블록과 비슷한 양의 코드가 생성되기 때문에, 예외 지정에 들어가는 비용도 try 블록과 비슷합니다.
* 솔직히 말해 예외 발생시 소모되는 비용은 그다지 큰 관심거리는 안된다.
    * 예외가 발생되는 경우는 극히 드물기때문에.
    * 통상적인 함수 복귀(return)와 비교했을 때, 예외 발생(throw)에 의한 함수 복귀의 속도는 약 10의 세 제곱 배(1000배) 만큼 느리다고 합니다.
        * 그러나 이 비용은 예외를 발생될 때에만 부담되는것이기에 예외가 거의 없는 상황에서는 제로로 봐도 무방합니다. 
        * 이와 같은 이유로 프로그램 처리 중에 꽤 자주 발생하는 상황을 나타내는 데에 예외를 남발해서는 안됩니다.
* 정리해 봅시다. 예외에 관련된 비용을 최소화하려면, 우선 가능하다면 예외 기능을 사용하지 않도록 컴파일하십시오. 그리고 try 블록과 예외 지정은 꼭 필요한 부분에만 사용합니다. 예외를 발생시키는 일도 진짜 예외적인 상황이라고 판단될 때에만 해야 합니다.



# Chpater 4 효율(Efficiency)
## 항목 16 : 뼛속까지 잊지 말자, 80-20 법칙!
* 80-20 법칙(80-20 rule)이란 이렇습니다. 
    * "프로그램 리소스의 80%는 전체 실행 코드의 약 20%만이 사용한다", 
    * "실행 시간의 80%는 실행 코드의 약 20%만이 소모한다", 
    * "메모리의 80%는 실행 코드의 약 20%만이 사용한다", 
    * "디스크 접근 회수의 80%는 실행코드의 약 20%가 접근한 회수이다", 
    * "프로그램 유지보수에 들어가는 수고의 80%는 실행 코드의 20%에 집중된다"라는 현상을 일러주는 법칙입니다.
* 프로그램의 성능을 향상시키려면 이 20%의 실행 코드를 찾아 이 코드를 최적화 해야 합니다. 
    * 나머지 80%의 코드를 최적화 하더라도 큰 성능 향상 효과를 보기는 어렵습니다. 
    * 가장 골치 아픈 일은 실행 병목현상을 일으키는 20%의 실행 코드를 찾는 것입니다. 
    * 방법은 두 가지가 있는데, 
        * 하나는 '대부분의 사람들이 하는 방법을' 따르는 것이고, 
        * 또 하나는 '제대로' 하는 것입니다.
* 대부분의 사람들이 하는 방법은 어림짐작(guess) 인데, 보통 이 방법은 틀리는 경우가 많습니다. 
    * 80-20 법칙의 진짜 의미는 '아무 곳이나 골라잡고 효율을 향상시키려고 애쓰는 것은 별 도움이 안 된다'는 것입니다.
* 성능 개선의 정도(正道)는 여러분의 가슴을 턱턱 막히게 하는 20%의 코드를 경험적으로 판별하는 것입니다. 
    * 그리고 지긋지긋한 20%를 판별하는 방법은 프로그램 프로파일러(program profiler)를 사용하는 것입니다.
    * 하지만 아무 프로파일러나 다 쓴다고 해서 되는 것은 아니고, 여러분이 관심을 두고 있는 리소스를 직접 측정해 주는 도구가 필요합니다. 예를 들어, 
        * 프로그램이 너무 느리다면 실행 영역별로 시간 측정을 해 주는 프로파일러를 써야 합니다. 
        * 프로파일러로 찾은 특정한 부분만 제대로 개선하면 전체 효율도 엄청나게 향상됩니다.
* 문장이 몇 번 실행되고 함수가 몇 번 호출되는지를 알려 주는 프로파일러도 있는데, 이런 것들은 그렇게 많이 사용되진 않습니다. 
    * 여러분이 만든 제품이 충분히 빠르기만 하면, 1045번 호출되든 10450번 호출되든 아무도 전혀 신경쓰지 않기 때문입니다.
    * 하지만 얼마나 자주 문장이 실행되고 함수가 호출되는지를 파악해 두면, 여러분이 만든 프로그램의 동작 원리를 좀더 잘 알 수 있게 됩니다. 
    * 예를 들어 여러분이 특정 타입의 객체가 생성되는 회수를 파악하고 싶다면 그 객체의 생성자가 호출되는 회수를 세면 됩니다. 게다가 문장 및 함수 호출 회수는 여러분이 직접 측정할 수 없는 프로그램의 동작 메커니즘을 간접적으로 이해하는 데에 유용합니다. 
    * 이를테면 동적 메모리 사용을 직접 점검할 수 없는 상황에 있다면 메모리 할당과 헤제용 함수(operator new, new[], delete, delete[] - 항목 8 참조)의 호출 횟수를 세어 사용하면 됩니다.
* 프로세스가 의외의 입력 데이터를 처리하는 동안에 프로파일링이 수행되는 경우에는, 프로파일러가 전체 성능에 별 영향을 끼치지 않는 미세 튜닝 부분(바로 80%의 코드입니다) 에 병목이 있다고 가르쳐 주더라도 불평해선 안 됩니다.
    *  이런 경우는 예외적인 상황에서의 프로그램 동작을 최적화할 수 있는 좋은 기회라고 보시면 됩니다. 일반적인 상황에서의 최적화 기법이 오히려 이런 경우에는 반대로 작용할 수 있습니다.
* 어쨌든, 이런 종류의 수행 성능 문제에 대처하는 최선의 길은 가능한 많은 데이터를 사용해서 소프트웨어를 프로파일링하는 것입니다.
    * 덧붙여서, 데이터는 사용자(최소한 가장 중요한 사용자)가 그 소프트웨어에 잘 입력하는 형태의 것으로 준비하셔야 합니다.


## 항목 17 : 효율 향상에 있어 지연 평가(lazy evaluation)는 충분히 고려해 볼 만하다
* 지연 평가란 가능한 한 계산을 미루다가, 꼭 계산 결과 값이 필요해질 때 비로소 계산을 수행하는 방법을 말합니다. 
    * 지연 평가를 적용할 수 있는 분야는 마음만 먹으면 수백 가지도 만들 수 있겠지만, 일반적인 것을 네 가지로 정리해서 살펴봅시다.
### 참조 카운팅(Reference Counting)
* 어떤 객체를 복사할 때 즉시 복사를 하지 않고, 복사 작업을 최대한 늦추는 방법입니다. 만일 객체를 복사하는 코드를 만나면 그 즉시 복사 작업을 하지 않고, 사본 객체가 원본 객체의 값을 공유하도록 만듭니다. 이때 사본 객체의 값을 읽기만 한다면 값의 복사 작업을 할 필요가 없습니다. 만약 둘중 하나의 값을 변경하게 된다면, 이제야 비로소 복사 작업을 수행하는 것입니다.
```c++
class String { ... };        // 문자열 클래스
 
String s1 = "Hello";
String s2 = s1;                // String의 복사 생성자를 호출합니다.
 
// 이 때 s1의 값을 s2로 복사하지 않고
// s1의 값을 s2와 공유하도록 만듭니다.
 
cout << s1;                    // s1의 값을 읽습니다.
cout << s1 + s2;            // s1과 s2의 값을 읽습니다.
 
// 이런 경우 s1의 값을 s2로 복사할
// 필요가 없습니다. 읽기만 하기 때문에
// 하나의 값을 공유하고 있어도 문제없습니다.
 
s2.convertToUpperCase();    // s2의 모든 문자를 대문자로 만듭니다.
 
// s2의 값만 바뀌고 s1의 값은 바뀌면
// 안되므로, 이제 s1의 값을 s2로
// 복사합니다. 그 후, s2의 모든 문자를
// 대문자로 바꿉니다.
 
// 만일 운이 좋아서 s2와 s1의 값을
// 영원히 읽기만 한다면, 문자열의 복사도
// 영원히 할 필요가 없을 것입니다.
```
* 기본 아이디어는 지연 평가입니다. 
    * 진짜 필요하기 전까지는 자기만의 데이터 사본을 만들지 않고, 남이 만들어 놓은 사본을 빌어다 쓰는 것입니다. 
### 데이터 읽기와 쓰기를 구분하기
* 앞에서 살펴본 참조 카운팅의 경우 값을 읽기만 할 때는 복사를 할 필요가 없고, 쓰기를 할 때 비로소 복사를 해야 하므로, 값을 읽는것과 쓰는 것을 구분해야 합니다. 그런데, operator[]와 같은 함수들은 값을 읽기 위해 호출될 수도 있고, 쓰기 위해 호출될 수도 있습니다. 안타깝게도 operator[]와 같은 함수들이 값을 읽기 위해 호출되었는지 쓰기 위해 호출되었는지 알 방법은 없습니다. 하지만, 항목 30에서 설명한 지연 평가와 프록시 클래스를 사용하면, 읽기가 맞는지 쓰기가 맞는지를 정확히 결정할 수 있을 때까지 읽기 동작과 쓰기 동작을 구별하는 것을 늦출 수도 있습니다.
```
String s = "Homer's Iliad";        // s는 참조 카운팅으로 운영
                                // 되는 문자열이라고 가정합니다.
...
cout << s[3];                    // s[3]을 읽기 위해 operator[]를 호출합니다.
s[3] = 'x';                        // s[3]에 쓰기 위해 operator[]를 호출합니다.
 
// 두 가지 경우를 구분해서 쓰기 위해 operator[]를
// 호출했을 때만 문자열의 복사 작업이 이루어지도록
// 합니다.
 
// 하지만 operator[]는 하나이므로
// 구현이 조금 까다로워집니다.
```
### 지연 방식의 데이터 가져오기(Lazy Fetching)
* 구성 필드를 많이 가지고 있는 대규모의 객체를 사용하는 어떤 프로그램이 하나 있습니다. 이 객체는 구성 필드들을 데이터베이스에 저장해 둡니다. 그런데 이 객체를 사용할 때 항상 모든 필드값을 사용할까요? 아마도 그렇지 않고 특정 필드값만 사용하는 경우도 많을 것입니다. 하지만 데이터베이스에서 필드값을 읽어오는 데는 많은 시간이 필요하죠. 이럴 때 데이터베이스에서 모든 필드값을 한꺼번에 읽어오지 않고, 필요한 필드값만 읽어와서 사용하는 것입니다.
```c++
class LargeObject {                        // 지속성을 지닌 아주 '큰' 객체
public:
    LargeObject(ObjectID id);            // 디스크에서 객체를 가져옵니다.
    
    const string& field1(void) const;    // 필드 1의 값
    int field2(void) const;                // 필드 2의 값
    double field3(void) const;            // 등...
    const string& field4(void) const;
    const string& field5(void) const;
    ...
};
 
// 이제 LargeObject를 디스크에서 가져오는
// 비용을 생각해 보도록 하죠.
 
// LargeObject는 매우 크기 때문에
// 모든 필드값을 읽어오려면 많은 시간이 걸립니다.
 
void restoreAndProcessObject(ObjectID id)
{
    LargeObject object(id);                // 객체를 가져옵니다.
    
    if (object.field2() == 0) {
        cout << "Object " << id << ": null field2.\n";
    }
}
 
// 하지만 위의 경우 필드 2 만을 사용합니다.
// 때문에 다른 필드를 읽는데 시간을 허비할
// 필요가 없습니다.
 
// 따라서 LargeObject를 '요구기반(demand-paged)'
// 방식의 객체 초기화 방법으로 구현하면 더
// 효율적일 것입니다.
 
class LargeObject {
public:
    LargeObject(ObjectID id);
    
    const string& field1(void) const;
    int field2(void) const;
    double field3(void) const;
    const string& field4(void) const;
    ...
    
private:
    ObjectID oid;
    
    mutable string    *field1Value;
    mutable int        *field2Value;
    mutable double    *field3Value;
    mutable string    *field4Value;
    ...
};
 
LargeObject::LargeObject(ObjectID id)
: oid(id), field1Value(0), field2Value(0), field2Value(0), ...
{}
 
const string& LargeObject::field1(void) const
{
    if (field1Value == 0) {
        데이터베이스에서 필드 1의 데이터를 읽고, field1Value가
        이 데이터를 가리키게 합니다.
    }
    
    return *field1Value;
}
 
// 상수 멤버 함수에서 field1Value의 값을
// 수정해야 하므로 mutable로 선언되었습니다.
// (나머지 필드값을 가리킬 포인터도 마찬가지)
 
// 하지만 만약 mutable을 지원하지 않는
// 컴파일러를 쓴다면 "this 흉내내기(fake this)"
// 방법을 써야 합니다.
 
class LargeObject {
public:
    const string& field1(void) const;    // 바뀐 것 없습니다.
    ...
    
private:
    string *field1Value;                // mutable로 선언되지 않았기 때문에
    ...                                    // 조금 구닥다리 컴파일러에서도
};                                        // 통합니다.
 
const string& LargeObject::field1(void) const
{
    // fakeThis라는 포인터를 하나 선언합니다.
    // 이 포인터는 this가 가리키는 것을 가리키고,
    // 상수성은 const_cast를 통해 제거합니다.
    LargeObject * const fakeThis =
        const_cast<LargeObject* const>(this);
    
    if (field1Value == 0) {
        fakeThis->field1Value =            // 이 대입 연산은 문제없습니다.
        데이터베이스에서 뽑은                    // 왜냐하면, fakeThis가 가리키는
        적절한 데이터                        // 객체는 상수 객체가 아니기 때문입니다.
    }
    
    return *field1Value;
}
 
// 만일 컴파일러가 const_cast조차 지원하지 않는다면
// C스타일의 캐스트를 씁시다.
 
// mutable 흉내내기 위해 C 스타일의 캐스팅을 사용합니다.
const string& LargeObject::field1(void) const
{
    LargeObject * const fakeThis = (LargeObject* const)this;
    
    ...                                    // 앞에서와 같습니다.
}
 
// 사용하기 전에 널인지 점검하는 과정이 왠지
// 무식하다고 생각될 수 있습니다.
// 잘못하면 에러를 낼 가능성도 높습니다.
// 이는 스마트 포인터를 사용함으로써
// 말끔히 해결할 수 있습니다(항목 28 참조).
```

### 지연 방식의 표현식 평가(Lazy Expression Evaluation)
* 지연 평가의 마지막 예제는 수치 연산입니다. 행렬을 예로 들어 보겠습니다. 아시다시피 행렬 연산은 비용이 매우 많이 듭니다. 그러므로 이러한 행렬 연산을 수행하는 코드를 만나면 즉시 연산을 수행하지 않고 연산자와 피연산자에 대한 정보만을 기록해 둡니다. 그리고 연산 결과가 필요하게 되면 그제서야 연산을 수행합니다. 행렬의 경우 계산 결과의 모든 원소를 사용하지 않는다면, 사용할 원소만을 계산하는 방법을 쓸 수도 있습니다.
```c++
template <typename T>
class Matrix { ... };            // 균등 행렬(homogeneous matrix)용
 
Matrix<int> m1(1000, 1000);        // 1000 * 1000 행렬
Matrix<int> m2(1000, 1000);        // 역시 1000 * 1000 행렬
 
...
Matrix<int> m3 = m1 + m2;        // m1과 m2를 더합니다.
 
// m3에는 m1과 m2를 더한 값이 아닌
// "m3의 값은 m1과 m2의 합이다"라는
// 표시만 해 두는 것입니다.
 
// 그리고 m3의 값을 사용하려고 하면
// 그제서야 m1과 m2의 합을 구해
// m3에 담는 것이죠.
 
// 만일 m3이 사용되기 전에 다음의 코드가
// 실행된다면
 
Matrix<int> m4(1000, 1000);
 
...                                // m4에 몇 개의 값을 줍니다.
m3 = m4 * m1;
 
// m3는 m1과 m2의 합이라는 것도 기억할 필요가
// 없습니다(따라서 계산 비용이 들지 않습니다).
 
cout << m3[4];                    // m3의 네 번째 행을 콘솔에 출력합니다.
 
// 이제는 더 이상 계산을 미룰 수 없습니다.
// 하지만 여전히 네 번째 행 이외의 행은
// 계산할 필요가 없습니다.
 
cout << m3;                        // m3을 모두 콘솔에 출력합니다.
 
// 이제는 진짜로 더 이상 계산을 미룰 수
// 없습니다. m3의 모든 요소를 계산하여
// 콘솔에 출력해야 합니다.
 
m3 = m1 + m2;                    // m3은 m1과 m2의 합입니다.
                                // 잊지 않았겠죠?
m1 = m4;                        // 이제 m3은 m2와 이전 m1의
                                // 합이 됩니다!
 
// 위와 같은 경우 때문에, Matrix<int>의
// 대입 연산자 안에는 m1을 변경하기 전의 m3의
// 값을 계산해 놓든지, m1의 이전 값을 복사해
// 놓고 m3의 자료구조를 그 이전 값에 맞추어
// 업데이트해야 할 것입니다.
```
* 지연 평가 기법은 만병 통치약이 아닙니다. 만약 모든 계산 결과를 즉시에 사용해야 하는 경우에는 지연 평가로 얻을 수 있는 것이 없습니다. 심지어 빠른 처리에 걸림돌이 되거나 메모리 사용량을 늘리기도 합니다. 왜냐하면, 여러분이 피하려고 했던 계산을 모두 해야 할 뿐만 아니라, 지연 평가에 쓰려고 만들어 둔 자료구조마저도 조작해야 하기 때문입니다. 계산을 바로 하지 않아도 되는 경우가 잦은 소프트웨어를 만들 때에 지연 평가의 효과를 톡톡히 볼 수 있습니다.
* 지연 평가 기법은 C++만의 특징이 아니라서 어떤 프로그래밍 언어를 쓰더라도 적용할 수 있고, 특히 APL, LISP의 변종들을 포함한 데이터처리 언어는 지연 평가 메커니즘을 언어의 일부로 만들어 놓았습니다. C++ 언어와 같은 시스템 개발의 주류 언어들 대부분은 즉시 평가를 사용합니다. 하지만 C++는 사용자가 직접 지연 평가를 구현해서 쓸 수가 있습니다.
* 이번 항목에 사용한 예제를 다시 잘 살펴보면, 클래스 인터페이스의 어느 부분에서도 즉시 평가나 지연 평가가 이루어진다는 표시나 흔적을 찾을 수 없을 것입니다. 즉, 일반적인 즉시 평가 메커니즘을 써서 클래스를 구현했다가 프로파일링 과정에서 병목 현상이 발견되면, 문제되는 부분을 지연 평가 메커니즘으로 교체하더라도 사용자 쪽에서 전혀 불편을 느끼지 않는다는 것입니다


## 항목 18 : 예상되는 계산 결과를 미리 준비하면 처리비용을 깎을 수 있다

* 이전 항목에서는 가능한 미루자라고 했는데, 이젠 현재 요구된 것 이외에 더 많은 작업을 미리 해 둠으로써 소프트웨어의 성능을 향상시킬 수 있습니다.
    * 이런 스타일의 작업을 과도선행 평가(over-eager evaluation)라고 부릅니다.
    * 과도선행 평가 방법의 기본 아이디어는, 상당히 자주 요구될 것 같은 계산이 있다면, 그 요구를 효율적으로 처리할 수 있는 자료구조를 설계하여 비용을 낮추자는 것입니다.
* 가장 간단하게 과도선행 평가를 구현할 수 있는 방법 중 하나는, 이미 계산이 끝났고 또다시 사용될 것 같은 값을 캐싱(caching)하는 것입니다.
    * 예를 들어 자주 사용하는 데이터를 매번 데이터베이스 질의를 통해 얻어오는 대신, 데이터베이스 질의를 통해 한 번 얻은 데이터를 메인 메모리에 넣어 두고 필요할 때마다 메인 메모리에서 가져다 쓰도록 만들면 훨씬 효율적으로 작동하는 프로그램을 만들 수 있습니다.

``` c++
int findCubicleNumber(const string& employeeName)
{
    // (직원이름, 작업칸 번호) 페어(pair, 쌍)를 저장하는 맵 객체를
    // 정적 변수로 정의합니다. 이 맵이 내부 캐시입니다.
    typedef map<string, int> CubicleMap;
    static CubicleMap cubes;

    // employeeName에 대한 엔트리를 캐시에서 찾습니다.
    // 엔트리가 발견되면, STL 반복자인 "it"은 탐색된 엔트리를
    // 가리킵니다(항목 35 참조).
    CubicleMap::iterator it = cubes.find(employeeName);

    // 엔트리의 탐색에 실패하면 "it"의 값은 cubes.end()가
    // 됩니다(STL의 통상적인 동작방식입니다).
    // 이 경우에는 데이터베이스에서 작업칸 번호를 찾아,
    // 그 결과를 캐시에 넣습니다.
    if (it == cubes.end()) {
        int cubicle =
        데이터베이스에서 employeeName의 작업칸 번호를
        찾은 결과;

        cubes[employeeName] = cubicle    // 내부 캐시에
                                        // (employeeName, cubicle)
                                        // 페어를 추가합니다.
        return cubicle;
    }
    else {
        // "it"은 맵에서 찾은 엔트리를 가리키고 있습니다. 이 엔트리는
        // (employee name, cubicle number) 페어입니다. 우리가 원하는
        // 것은 이 페어의 두 번째 요소인데, second라는 멤버를 사용하여
        // 뽑아낼 수 있습니다.

        return (*it).second;
    }
}
 
// return (*it).second; 부분은
// return it->second; 로 바꿀 수 있으나,
// 일부 오래된 컴파일러의 경우 반복자에
// operator-> 함수가 재정의되어 있지 않은
// 경우가 있어서, 이런 컴파일러를 사용하시는
// 분들은 (*it).second를 써야 이식 문제로
// 고민하지 않습니다.
```

* 캐싱 이외의 방법으로 미리가져오기(Prefetching)란 방법도 있습니다.
    * 이 방법은 참조 위치의 근접성(locality of reference)을 이용합니다. 쉽게 말하면, "어떤 곳의 데이터를 참조했다면, 가까운 시일 내에 그 주변의 데이터도 참조할 가능성이 높다" 라는 것을 이용하는 것입니다.
    * 예를 들어 디스크에서 어떤 데이터를 읽을 때 그 데이터 주위에 있는 데이터도 한꺼번에 읽어 오도록 만들면 훨씬 효율적으로 작동하도록 만들 수 있습니다.
    * 또 다른 예로 동적 배열을 만들 때 딱 필요한 만큼만의 메모리를 할당하는 것이 아니라 여분의 메모리를 미리 할당하여, 가까운 시일 내에 동적 배열이 확장되더라도 미리 할당한 범위 내라면 동적 할당을 하지 않음으로써 효율적으로 작동하도록 만들 수 있습니다.

``` c++
template <typename T>        // T 타입의 동적 배열
class DynArray { ... };        // 을 나타내는 템플릿
 
DynArray<double> a;            // 여기서는 a[0]만이
                            // 유효한 배열 요소
                            // 입니다.
 
a[22] = 3.5;                // a는 자동으로 확장됩니다.
                            // 이 배열에서 유효한 인덱스는
                            // 0부터 22까지 되었습니다.
 
a[32] = 0;                    // 다시 확장됩니다.
                            // 이제는 a[0]-a[32]가 유효합니다.
 
// DynArray의 operator[]는 아마도
// 다음과 같이 구현되었을 것입니다.
 
template <typename T>
T& DynArray<T>::operator[](int index)
{
    if (index < 0) {
        throw an exception;    // 음수의 인덱스는
    }                        // 여전히 안 됩니다.

    if (index > the current maximum index value) {
        new를 사용하여, 주어진 인덱스가 유효할 만큼
        추가 메모리를 할당합니다.
    }

    return 배열 내의 index 번째의 요소;
}
 
// 만약 동적 배열의 크기가 확장 되었다면
// 조만간 또 다시 확장이 일어날 가능성이 큽니다.
// 따라서 딱 필요한 만큼만 확장하는 것이 아니라
// 여분의 메모리를 미리 확장해 놓는 방법을
// 쓸 수 있습니다.
 
template <typename T>
T& DynArray<T>::operator[](int index)
{
    if (index < 0) throw an exception;

    if (index > 현재의 최대 인덱스) {
        int diff = index - 현재의 최대 인덱스값;

        index에 diff를 더한 만큼의 인덱스가 유효하도록
        메모리를 할당합니다.
    }

    return 배열 내의 index 번째의 요소;
}
 
DynArray<double> a;            // a[0]만이 유효한 상태
 
a[22] = 3.5;                // new가 호출되는데, 이제는 인덱스 44가
                            // 유효하도록 메모리가 할당됩니다.
                            // a의 실제 배열 크기는
                            // 23이 됩니다.
 
a[32] = 0;                    // a의 실제 배열크기(총 용량이 아닌)
                            // 는 a[32]가 유효하도록 변합니다.
                            // 하지만 new는 호출되지 않습니다.
```

* 이번 항목은 항목 17에서 강조한 내용과 상반되는 개념이 절대로 아닙니다. 지연 평가는 "어떤 작업의 결과가 항상 사용되는 것은 아니라면 그 작업을 미리 하지 않음으로써 프로그램의 효율을 높이는" 방법이고, 과도선행 평가는 "어떤 작업의 결과가 거의 항상 사용되거나 한 번 이상 자주 사용된다면 그 작업을 미리 해둠으로써 프로그램의 효율을 높이는" 방법입니다.

## 항목 19 : 임시 객체의 원류(原流)를 정확히 이해하자

* 실행 도중에 잠깐 동안만 사용되는 변수를 프로그래머들은 이것을 가리켜 임시 객체라고 부릅니다 .

``` c++
template<class T>
void swap(T& ojbect1, T& object2)
{
    T temp = object1;
    object1 = object2;
    object2 = temp;
}
```

* temp 를 임시객체로 불러도 이상할것은 없지만, 하지만, C++ 사전에 temp는 임시 객체가 절대로 아닙니다.
    * 그냥 함수의 스택에 만들어지는 지역 객체일 뿐입니다.
* C++에서의 진짜 '임시 객체'는 우리 눈에 보이지 않습니다. 소스 코드에도 없습니다. C++에서의 진짜 임시 객체(이후 임시 객체)는 힙 이외의 공간에 생성되는 '이름 없는' 객체입니다.
    * 이런 이름 없는 객체가 만들어지는 상황은 두 가지입니다.
        * 첫째는 함수 호출을 성사시키기 위해 암시적 타입변환이 적용될 때이고,
        * 둘째는 함수가 객체를 값으로 반환(return by value)할 때입니다.
    * 임시 객체가 어떻게, 그리고 왜 생성되고 소멸되는지 정확하게 파악하는 것은, 어쩌면 C++ 코더에서 C++ 프로그래머로 발돋음하기 위한 중요한 요구사항일지도 모릅니다. 왜냐하면, 임시 객체가 생성되고 소멸되는데 드는 비용이 프로그램 전체 성능에 미치는 영향이 꽤 쏠쏠한 수준이기 때문입니다.
* 우선, 함수 호출을 성사시키기 위해 임시 객체가 생성되는 첫째 경우를 생각해 보죠. 함수에 전달되는 객체의 타입과 바인딩되어야 하는(원래의 함수 선언에 들어 있었던) 매개변수의 타입이 다를 때 이런 일이 일어납니다.

``` c++
// str 안에 들어 있는 ch의 출현회수를 반환합니다.
size_t countChar(const string& str, char ch);
 
char buffer[MAX_STRING_LEN];
char c;
 
// 문자와 문자열을 읽습니다. 문자열을 읽을 때 버퍼가
// 넘치는 것을 막기 위해 setw를 사용했습니다.
cin >> c >> setw(MAX_STRING_LEN) >> buffer;
 
cout << "There are " << countChar(buffer, c)
    << " occurrences of the character " << c
    << " in " << buffer << endl;
 
// countChar 함수는 첫 번째 매개변수로 string
// 타입의 상수 참조자를 받지만, 넘겨진 첫 번째 인자는
// char 타입의 배열입니다.
 
// 이 함수 호출을 성사시키기 위해 컴파일러는
// char 타입의 배열로부터 string 타입의
// 임시 객체를 생성합니다.
 
// 이렇게 만들어진 임시 객체는 countChar함수가
// 복귀할 때 자동으로 소멸됩니다.
 
// 컴파일러가 변환을 도맡아주지만, 임시 string 객체가 갑자기 생성되었닥다 소멸되는 일은 불필요한 낭비입니다. 
// 이런 일을 막는 방법은 두 개 있습니다.
// 1. 코드를 다시 설계해서 이런 벼환이 일어나지 않게 하는것.(항목 5 참고)
// 2. 타입변환이 불필요하도록 소프트웨어를 수정하는 것(항목 21 참고)
// 한가지 기억할것은 이런 암시적 타입변환이 이루어지는것은 오직 객체가 값으로 전달될때 혹은 상수 객체 참조 타입의 매개변수로 객체가 전달될때라는것입니다.
// 비상수 객체 참조 타입의 매개변수로 객체가 전달될때에는 암시적 타입변환이 일어나지 않습니다. 

void uppercasify(string& str);        // str 안의 모든 문자를
                                    // 대문자로 바꿉니다.
 
char subtleBookPlug[] = "Effective C++";
 
uppercasify(subtleBookPlug);        // 가차 없이 에러!
 
// 하지만 이 경우처럼 상수 객체 참조자가 아닌
// 비상수 객체 참조자 타입의 매개변수로 객체가
// 전달될 때에는 임시 객체가 생성되지 않습니다.
 
// 만일 임시 객체를 생성한다면, uppercasify
// 함수가 변경하는 객체는 subtleBookPlug이
// 아니라 subtleBookPlug으로부터 생성된
// 임시 객체고, 결과적으로 subtleBookPlug는
// 원하는대로 변경되지 않기 때문에 비직관적입니다.
 
// 반면 상수 객체 참조자의 경우, 상수라서
// 임시 객체를 읽기만 하므로 이러한 문제가
// 발생하지 않습니다.
 
// 그래서 상수 객체 참조자 타입의 매개변수로
// 객체가 전달될 때에만 임시 객체가 생성됩니다.
```

* 임시 객체가 생성되는 두 번째 경우는 함수가 객체를 (값으로) 반환할 때라고 했습니다. 예를 들어, operator+는 두 피연산자의 합을 나타내는 객체를 반환해야 합니다.
    * Number라는 타입의 객체에 대해 덧셈을 수행하는 operator+가 있다면 다음과 비슷할 것입니다.
        * const Number operator+(const Number& lhs, const Number& rhs);
    * 이 함수의 반환값은 임시 객체입니다. 왜냐하면, 이 객체는 이름이 없기 때문입니다. 그냥 함수의 반환값일 뿐이죠. 임시 객체를 반환하는 이상, operator+를 호출할 때마다 이 객체는 만들어졌다가 없어지는 일을 반복할 수밖에 없습니다(반환값이 상수(const)인 이유는 항목 6에서 확인하세요).
* 기본 아이디어는 임시 객체에는 적지 않는 비용이 든다는 것입니다.
    * 할 수 있으면 임시 객체는 만들어지지않도록 하는 것이 좋은데, 이보다 더 중요한 것은
        * 임시 객체가 생설 될수 있는 지점을 알아보는게 필요합니다.
        * 소스 코드에서 상수 객체 참조자가 나온 부분이 발견되면 "임시 객체가 생성되었다가 바로 없어지겠군"이라고 알아봐봅시다.

## 항목 20 : 반환값 최적화(return value optimization)가 가능하게 하자

* 객체를 값으로 반환하는 함수는 효율에 절망적입니다.
    * 19항목처럼 보이지 않는 임시 객체의 생성 및 소멸과 함께 값에 의한 반환이 이루어질 뿐 아니라 없앨 수도 없기 때문입니다.
* 반드시 값을 통해 반환(return by value)해야 하는 함수들도 있습니다. 대표적인 예를 들면 operator\*가 있습니다. 이런 함수들을 억지로 값이 아닌 참조자나 포인터를 반환하도록 만드려고 하면 문제가 생길 수 밖에 없습니다.

``` c++
//-----------------------------------------------------------------------------
// 값에 의한 반환 대신 포인터를 반환하는 경우의 문제점
 
// 값에 의한 객체 반환을 피하기 위해 짜냈지만 별로 안 좋은 방법
const Rational * operator*(const Rational& lhs,
                        const Rational& rhs);
 
Rational a = 10;
Rational b(1, 2);
 
Rational c = *(a * b);        // a와 b를 곱하는 이 문장이
                            // 자연스러워 보이세요?
 
// 또한 이 경우 받은 포인터를 삭제(delete) 해야
// 할지도 고민해야 합니다. 보통은 삭제하는게 정답이고,
// 삭제해야 하는 경우 리소스 누수를 일으킬지도 모릅니다.
 
 
//-----------------------------------------------------------------------------
// 값에 의한 반환 대신 참조자를 반환하는 경우의 문제점
 
// 값에 의한 객체 반환을 피하는 위험한(그리고 틀린)
// 방법
const Rational& operator*(const Rational& lhs,
                    const Rational& rhs);
 
Rational a = 10;
Rational b(1, 2);
 
Rational c = a * b;            // 이상하지 않은 문장
 
const Rational& operator*(const Rational& lhs,
                    const Rational& rhs)
{
    Rational result(lhs.numerator() * rhs.numerator(),
        lhs.denominator() * rhs.denominator());
    return result;
}
 
// 이 객체가 반환하는 참조자가 가리키는 객체는
// 함수가 종료될 시점에는 이미 이 세상에는
// 존재하지 않습니다. 즉 말도 안되는 코드입니다.
```

* 값을 반환할 수 밖에 없는 함수는 어쩔 수 없이 값을 반환할 때 임시 객체를 만들게 됩니다(항목 19 참조). 물론 그에 대한 비용도 지불해야 합니다. 하지만 객체 대신에 생성자 인자를 반환하도록 만들면 컴파일러가 최적화를 해 줍니다. 이러한 최적화를 "반환값 최적화(return value optimization: RVO)" 라고 합니다. 반환값 최적화는 대부분의 컴파일러가 지원합니다.

``` c++
//-----------------------------------------------------------------------------
// 반환값 최적화 예제
 
// 객체를 반환하는 함수를 작성하는
// 효율적이면서도 정확한 방법
const Rational operator*(const Rational& lhs,
                        const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}
 
// 반환하는 표현식을 잘 보시면 Rational의 생성자가
// 호출되고 있는 것을 볼 수 있습니다.
 
Rational a = 10;
Rational b(1, 2);
 
Rational c = a * b;            // operator*가 여기서 호출됩니다.
 
// 컴파일러는 operator* 안에서 생기는 임시 객체와
// operator*가 반환하는 임시 객체를 모두 없애고,
// 계산 결과값을 객체 c에 대해 할당된 메모리에다
// 직접 넣어 초기화해 줍니다.
 
// 대부분의 컴파일러가 이런 기능을 가지고 있고,
// 이런 경우 operator*를 호출했을 때
// 소모되는 임시 객체의 총 비용은 제로가 됩니다.
 
 
//-----------------------------------------------------------------------------
// 위의 예제에서 c는 이름이 붙은 객체인데,
// 이름이 붙은 객체는 컴파일러의 최적화 혜택을
// 받을 수 없게 되어 있습니다(항목 22 참조).
 
// 하지만, 이에 대한 오버헤드는 operator*를
// 인라인 함수로 선언하면 간단히 없앨 수 있습니다.
 
// 객체를 반환하는 함수를 작성하는
// 가장 효율적인 방법
inline const Rational operator*(const Rational& lhs,
                                const Rational& rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}
 
// 하지만 1996년에, C++ 표준위원회는 이름을 가진 객체와
// 이름이 없는 객체 모두 반환값 최적화 기능에 의해
// 최적화될 수 있다고 발표했습니다.
 
// 따라서 바로 앞에서 인라인으로 선언하지 않고
// 만든 operator*함수도 동일한(최적화된)
// 코드를 만들어 낼 수 있습니다.
```



## 항목 21 : 오버로딩은 불필요한 암시적 타입변환을 막는 한 방법이다
* 특별한 코드는 아닌것 같지만 아래를 봅시다. 
```c++
class UPInt {                // Unlimited Precision Integer를
public:                        // 나타내기 위한 클래스
    UPInt(void);
    UPInt(int value);
    ...
};
//또 반환값이 const 로 되어있네요 항목 6 확인. 
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);
 
UPInt upi1, upi2;
 
...
UPInt upi3 = upi1 + upi2;
```
* 아직까진 별다른 것은 없는데 upi1, upi2는 둘다 UPInt 객체이고 이둘을 더할때 operator+가 호출될뿐입니다.
* 이제 시작입니다. 
```c++
//실행은 됩니다. 정수 10을 UPInts 타입으로 바꾸기 위해 임시 객체를 생성했고(항목 19) 이것을써서 덧셈을 했습니다.
upi3 = upi1 + 10;
upi3 = 10 + upi2;
```
* 임시객체 생성은 우리가 원하지 않는 낭비입니다. 
* 우리의 목적은 타입변환이 아닌 UPInt 와 int 타입의 인자에 대해 operator+를 호출하는 것입니다. 
* 임시 객체를 생성하지 않게 만들어, 임시 객체에 대한 비용을 없애고 싶으면 다음과 같이 operator+의 오버로딩 버전을 추가합니다.

```c++
 
const UPInt operator+(const UPInt& lhs,        // UPInt와 UPInt를
                      const UPInt& rhs);    // 더합니다.
 
const UPInt operator+(const UPInt& lhs,        // UPInt와 int를
                      int rhs);                // 더합니다.
 
const UPInt operator+(int lhs,                // int와 UPInt를
                      const UPInt& rhs);    // 더합니다.
 
UPInt upi1, upi2;
 
...
UPInt upi3 = upi1 + upi2;        // 좋습니다. upi1 혹은 upi2에 대해
                                // 임시 객체가 만들어지지 않습니다.
 
upi3 = upi1 + 10;                // 좋습니다. upi1 혹은 10에 대해
                                // 임시 객체가 만들어지지 않습니다.
 
upi3 = 10 + upi2;                // 좋습니다. 10 혹은 upi2에 대해
                                // 임시 객체가 만들어지지 않습니다.
                                
// 하지만 다음은 에러입니다.
const UPInt operator+(int lhs, int rhs);    // 딱 걸렸습니다. 에러!
```
* 정수형 끼리의 덧셈과 같은 기본 연산의 의미가 바뀌면 심각한 혼란을 초래할 수 있어서, C++에서 연산자 오버로딩은 '오버로딩되는 연산자 함수는 반드시 최소한 한 개의 사용자 정의 타입을 매개변수로 가져야 한다'는 규칙을 따라야 합니다.
* 오버로딩을 통해 임시 객체의 생성을 막느 ㄴ기법은 연산자 이외의 함수에도 써먹을수 있습니다. 
    * char* 포인터가 쓰이는 곳에 string  객체도 쓸수 있으면 좋겠다 싶으면 char* 와 string 에 대한 오버로딩 함수를 하나씩 만들어 놓으면 되겠지요

* 물론 80-20 법칙(항목 16 참조)은 어디서든 고려 대상입니다. 프로파일링 결과 operator+가 성능에 큰 영향을 주지 않음에도 불구하고 함수를 오버로딩해서 쌓아 두는 것은 아무 의미도 없습니다.




## 항목 22 : 단독 연산자(op) 대신에 =이 붙은 연산자(op=)를 사용하는 것이 좋을 때가 있다
* 아래의 예시를 보자
```c++
//아래가 된다면
x = x + y;
x = x - y;

//아래도 된다고 생각한다.
x += x;
y -= y;

//x,y 가 사용자 정의 타입이라면 꼭 앞과같이 되리라는 보장은 없다.
//operator+, operator-, operator+= 사이에는 아무관계가 없다. 
//각 연산자 함수를 직접 스스로 구현해야 가능해 진다.
```
* 연산자 대입형태(operator+=)와 단독 형태(operator+) 사이에 C언어의 +, += 같은 자연스러운 관계를 지어 두는 좋은 방법은 
    * 대입 형태를 사용해서 단독 형태를 구현하는 것이다(항목6 참조)
* 단독 연산자의 구현은 대입 형태 연산자를 사용하는 것이 좋습니다. 예를 들어 operator+의 구현에 operator+=를 사용하는 것입니다. 이렇게 하면 코드 중복을 줄여 유지 보수가 편해집니다. 또한 대입 형태의 연산자를 public으로 선언했다면, 단독 연산자를 프렌드로 지정하지 않아도 됩니다.)
```c++
//-----------------------------------------------------------------------------
class Rational {
public:
    ...
    Rational& operator+=(const Rational& rhs);
    Rational& operator-=(const Rational& rhs);
};
 
// operator+=를 사용해서 구현한 operator+입니다.
const Rational operator+(const Rational& lhs,
                         const Rational& rhs)
{
    return Rational(lhs) += rhs;
}
 
// operator-=을 사용해서 구현한 operator-
const Rational operator-(const Rational& lhs,
                         const Rational& rhs)
{
    return Rational(lhs) -= rhs;
}
 
 
//-----------------------------------------------------------------------------
// 단독 형태의 연산자를 모두 전역 유효범위(global scope)에
// 두어도 크게 신경쓰지 않는다면, 다음과 같에 템플릿을
// 만들어 두면, 클래스 안에다 단독 형태의 연산자 함수를
// 구현할 필요 자체를 없앨 수 있습니다.
 
template <typename T>
const T operator+(const T& lhs, const T& rhs)
{
    return T(lhs) += rhs;
}
 
template <typename T>
const T operator-(const T& lhs, const T& rhs)
{
    return T(lhs) -= rhs;
}
 
...
 
// 안타깝게도 어떤 컴파일러는 T(lhs)를 lhs의
// 상수성을 제거한 캐스팅 결과로 취급해서,
// rhs를 lhs에 더하고 값이 바뀐 lhs의
// 참조자를 반환해 버리고 맙니다!
 
 
//-----------------------------------------------------------------------------
// 다음과 같은 코드는 구버전의 컴파일러에서
// 효율 문제를 일으킬 수 있습니다.
 
template <typename T>
const T operator+(const T& lhs, const T& rhs)
{
    T result(lhs);            // lhs를 result에 복사합니다.
    return result += rhs;    // 여기에 rhs를 더하고 그것을 반환합니다.
}
 
// result는 이름이 있는 객체라서 구버전의 컴파일러에서
// 반환값 최적화가 제대로 안 될 수도 있습니다.
```
* 단독 연산자는 임시 객체를 생성하는 반면(항목 20 참조), 대입 형태 연산자는 임시 객체를 생성하지 않습니다. 따라서 단독 연산자와 대입 형태 연산자를 바꿔 써도 문제가 없는 상황이라면, 대입 형태 연산자를 쓰는 것이 효율적일 때가 많습니다. 예를 들어 x = x + y; 와 같이 쓰는 것 보다는 x += y;와 같이 쓰는 것이 더 효율적입니다.
    * 왜냐하면 단독 연산자는 두 피연산자를 변경하지 않고 두 피연산자를 연산한 값을 반환해야 하지만, 대입 형태 연산자는 좌측 피연산자의 값에 연산 결과를 저장하고 좌측 피연산자의 참조자를 반환하면 되기 때문입니다.
```c++
Rational a, b, c, d, result;
...
result = a + b + c + d;        // 세 개의 임시 객체가 사용되는데,
                            // 임시 객체 하나마다 operator+가
                            // 호출됩니다.
 
result = a;                    // 임시 객체가 필요없습니다.
result += b;                // 임시 객체가 필요없습니다.
result += c;                // 임시 객체가 필요없습니다.
result += d;                // 임시 객체가 필요없습니다.
 
// +를 연달아 쓴 첫번째 코드는 작성하기도 좋고,
// 디버깅하거나 유지보수하기에도 좋습니다.
 
// 그리고 실행시간의 80%를 잡아먹는 문제의
// 부루주아 코드입니다(항목 16 참조).
 
// +=를 활용한 두번째 코드는 훨씬 효율적이고,
// 어셈블리 언어 프로그래머의 눈에는 무척 읽기
// 좋습니다.
```
* 대체적으로 대입형태의 연산자(operator+= 와 같은는 단독 형태 연산자(operator+ 와 같은)보다 효율적입니다. 


## 항목 23 : 정 안 되면 다른 라이브러리를 사용하자!
* 라이브러리마다 효율, 확장성, 이식성, 타입 안전성 등에 대하여 다른 설계 철학을 가지고 구현되었기 때문에, 다른 사항보다 수행 성능 쪽에 많은 무게를 두어 설계한 라이브러리를 구할 수 있으면, 그것을 바꾸는 것만으로 손쉽게 소프트웨어의 효율을 향상시킬 수 있습니다.
    * 예를 들어 프로파일링 결과, 입·출력에 병목 현상이 있으면 iostream 대신에 stdio를 써서 성능을 개선할 수 있습니다(대개 iostream보다 stdio가 효율적입니다). 동적 메모리 할당이나 해제에 시간이 많이 걸리면 operator new와 operator delete(항목 8 참조)를 대신할 만한 것을 찾아 보셔도 좋습니다.
    * C++전용의 ㅣiostream 은 C에서 쓰이는 stdio 가 가지고 있지 않는 몇가지 장점을 가지고 있다
        * 타입안정성을 갖추고 있을뿐만 아니라 확장성도 있다
        * 하지만 효율적 측면에서 보면 대새 stdio가 작고 빠르다. 
* 다음의 코드는 가장 기본적인 I/O 기능을 시험해보려는 벤치마크 프로그램이다. 
```c++
#ifdef STDIO
#include<stdio.h>
#else
#include <iostream>
#include <iomanip>
using namespace std;
#endif

const int VALUES = 30000;//읽고 쓰는 회수
int main()
{
    double d;
    for(int n=1; n<= VALUES;  ++n){
#ifdef STDIO
        scanf("%lf", &d);
        printf("%10.5f", d);
#else
        cin >> d;
        cout<<setw(10)  //필도의 폭을 설정
            <<setprecision(5)//소수부분 자릿수 고정
            <<setiosflags(ios::showpoint)//따라오는 0을 그대로 두고
            <<setiosflags(ios::fixed)//이 설정을 사용합니다.
            <<d;
#endif
        if(n%5 ==0){
#ifdef STDIO
            printf("\n");
#else
            cout<<'\n';
#endif            
        }
    }
    return 0;
}
```
* 위 소스를 보면 iostreams 를 가지고도 고정서식 입출력을 나타낼수있지만 
```c++
        cout<<setw(10)  //필도의 폭을 설정
            <<setprecision(5)//소수부분 자릿수 고정
            <<setiosflags(ios::showpoint)//따라오는 0을 그대로 두고
            <<setiosflags(ios::fixed)//이 설정을 사용합니다.
            <<d;
```
* C 형태가 훨씬 사용하기 편합니다.
* 하지만 operator<< 는 타입 안정성과 확정성을 동시에 갖추고 있죠, printf는 그렇지 않습니다.
* 테스트 결과 stdio가 iostream 보다 20%에서 200% 빠른 것을 확인할수 있었습니다.
* 게다가 stdio로 링크된 실행파일이 작습니다. 
* 라이브러리마다의 특성을 고려해서 사용합니다.



## 항목 24 : 가상 함수, 다중 상속, 가상 기본 클래스, RTTI에 들어가는 비용을 제대로 파악하자
* 구현에 따라 성능에 영향을 미치는 C++ 언어의 특징들 중에 부동의 일등자리를 차지하는 것은 바로 가상함수(virtual function)입니다.
* 가상 함수가 호출될때 그 함수에 대해 실행되는 코드는 함수가 호출되는 객체의 동적 타입에맞는 것이어야 합니다. 
    * 대부분의 컴파일러는 가상테이블(virtual table)과 가상 테이블 포인터(virtual table pointer)를 사용하여 이것을 구현합니다. 
    * 가상 테이블은 vtbl, 가상 테이블 포인터는 vptr로 표기하겠습니다.
* vtbl은 보통 함수 포인터의 배열
    * 이 테이블은 가상 함수를 선언했거나 상속받은 클래스에 무조건 생기고 vtbl의 각 요소는 해당 클래스에 정의한 가상함수 코드의 시작주소(즉, 함수 포인터)입니다. 
```c++
class C1{
public:
    C1();
    
    virtual ~C1();
    virtual void f1();
    virtual int f2(char c) const;
    virtual void f3(const string& s);
    
    void f4() const;
    ...
};
```
* 이 상태에서 C1의 가상함수 테이블 배열은 아래처럼 되어있습니다.
```
C1's vtbl 
----
|  |  ---> C1::~C1의 함수 코드
----
|  |  ---> C1::f1의 함수 코드
----
|  |  ---> C1::f2의 함수 코드
----
|  |  ---> C1::f3의 함수코드
----
```
* f4는 테이블에 없음. C1의 생성자도 마찬가지로 없음, 비가상 함수는 보통의 C 함수처럼 구현되기에 특별한 수행성능 문제가 끼어들지 않음
* 이 상황에서 C2가 C1을 상속받았다고 봅시다
```c++
class C2:public C1{
public:
    C2();//비가상함수
    virtual ~C2();//재정의한 함수
    virtual void f1();//재정의한 함수
    virtual void f5(char* *str);//새로 추가된 가상 함수
    ...
};
```
* 이 클래스의 가상 테이블 엔트리는 C2의 객체에 대해 실행될 함수를 가리킴
    * 물론 C2에서 재정의 하지 않은 C1의 가상 함수도 들어있음
```
C2's vtbl 
----
|  |  ---> C2::~C2의 함수 코드
----
|  |  ---> C2::f1의 함수 코드
----
|  |  ---> C1::f2의 함수 코드
----
|  |  ---> C1::f3의 함수코드
----
|  |  ---> C2::f5의 함수코드
----
```
* 가상 함수를 선언한 클래스로부터 만들어진 객체에는 그 클래스의 가상 함수를 가리키는 데이터 멤버(vptr)가 하나 숨어있음
* 가상 함수 테이블의 크기는 그 클래스가 갖는 가상 함수의 개수(기본 클래스에서 상속받는 것까지 합해서)에 비례하며, 가상 함수를 갖는 클래스 마다 하나씩 만들어 집니다. 프로그램에 들어가는 클래스의 수가 매우 많거나, 클래스에 가상 함수의 수가 매우 많은 경우, 결코 무시 못 할 정도의 메모리 부담이 들어갈 수 있습니다.
* 가상 함수를 하나 이상 갖는 클래스 타입의 객체마다, 가상 함수 테이블의 시작 주소를 가리키는 멤버 포인터(가상 테이블 포인터: vptr)가 추가됩니다.
    * 크기가 작은 객체일수록 가상 테이블 포인터의 크기가 부담스러워 질 수 있습니다. 예를 들어 포인터 하나의 크기가 4 바이트라고 하면, 데이터 멤버의 크기가 4 바이트밖에 안되는 작은 객체는 가상 테이블 포인터 때문에 크기가 두 배 증가하게 됩니다. 소량의 한정된 메모리를 갖는 시스템에서는 이 비용이 치명적입니다.
    * 메모리를 걱정 없이 쓸 수 있는 시스템이라고 해도 소프트웨어의 수행 성능에서 손해를 봅니다. 덩치가 큰 객체는 캐시나 가상 메모리 페이지에 잘 맞지 않게 되는데, 결국 운영체제가 메모리 페이징을 더 많이 하는 결과를 낳기 때문입니다.
* 가상 함수는 인라인으로 만들 수 없습니다. 왜냐하면 "인라인(inline)"이라 함은 "컴파일 도중에, 호출 위치에 호출되는 함수의 몸체를 끼워 넣는다"라는 뜻인데 반해, "가상(virtual)"이라 함은 "호출할 함수를 런타임까지 기다려 결정한다"라는 뜻이기 때문입니다.
    * 객체를 통해 가상 함수를 호출하는 경우에는 정적 바인딩이 일어나므로 가상 함수도 인라이닝을 할 수 있지만, 가상 함수는 대부분 포인터나 참조자를 통해 호출됩니다. 즉, 사실상 함수의 인라인 효과를 포기해야 합니다.
* 아래의예를 살펴봅시다
```c++
void makeACall(C1 * pC1)
{
    pC1->f1();
}
```
* 포인터 pC1을 통해 가상함수 f1을 호출하라는 뜻입니다.
    * 이 코드만 봐서 f1이 C1인지 C2인지 알수 없습니다. 
    * 어쨌든 컴파일러는 makeACall안에서 f1을 호출하는 코드를 만들어야 합니다. 
    * 컴파일러는 이를 위해 다음과 같은 단계를 거칩니다.
* 1. pC1이 가리키는 객체의 vptr을 따라 vtbl로 갑니다. 
    * 이때 들어가는 비용은 (vptr의 위치로 가기 위한) 옵셋 조정과 (vtbl로 가기 위한) vptr 참조뿐임
* 2. vtbl을 뒤져서 지금 호출해야 하는 함수(이 경우엔 f1)에 해당하는 포인터를 찾아옵니다.
    * 이때 들어가는 비용도 (vptr의 위치로 가기위한) 옵셋 조정과 (vtbl로 가기위한 ) vptr참조뿐입니다.
* 3. 2단계에서 가져온 포인터가 가리키는 함수를 호출합니다.

* 즉 가상 테이블이 vptr이고 f1이 가상 테이블 내 인덱스는 i라고 가정할때 
```c++
pC1->f1();

//위의 코드는에 대해 컴파일러가 생성하는 코드는 아래처럼 될 것입니ㅏㄷ.
(*pC1->vptr[i])(pC1);//pC1->vptr이 가리키는 vtbl내의 i번째ㅔ 요소가 가리키는 함수를 호출함. pC1은 그 함수에 this포인터를 넘김.
```
* 가상 함수 때문에 속도가 엄청나게 느리다는 말은 할수가 없습니다.
    * 비가상 함수와 거의 비슷한 효율을 보입니다. 고작 명령어 몇개가 더 실행되는 수준입니다. 
    * 가상함수 호출에 드는 비용은 함수 포인터를 통해 함수를 호출하는데 드는 비용과 같습니다
    * 결국 가상 함수만 놓고 수행성능의 발목을 잡는 요인이라고 하긴 힘듭니다. 
* 그렇다면. 수행성능의 발목을 잡는 가상 함수 비용의 진수는 어디 일까요? 바로 인라인 입니다.
* 인라인은 컴파일 도중에 호출되는 함수의 몸체에 끼워넣는다라는 뜻이지만, 가상이라 함음 호출할 함수를 런타임까지 기다려 결정한다는 뜻
* 가상함수는 함수의 인라인 효과를 포기한다는 의미
    * 대부분 가상함수는 인라인 하지 않습니다.
        * 객체를 통해 호출될 경우 가상 함수의 인라인닝이 가능하지만, 대부분의 가상 함수 호출은 포인터나 참조자를 통해 이루어지는데 이런 하수 호출에 대해 컴파일러가 vptr을 사용한 코드를 만들수가 없습니다. 

* 지금까지 알아본 사항은 단일 상속과 다중 상속 모두 적용되는 것이었는데, 다중 상속까지 생각하면 더 복잡해 집니다. 그 결과, 클래스별 오버헤드와 객체별 오버헤드가 동시에 늘어나고, 런타임 생성 혹은 호출에 들어가는 비용도 그만큼 늘어납니다.
    * 다중 상속을 받은 경우 vptr의 위치를 찾는 오프셋 계산이 더욱 복잡해 집니다. 한 객체 안에 vptr이 여러개 들어있고(상속받은 클래스에서 하나씩 가져온 결과), 파생 클래스에 대한 vtbl 외에 기본 클래스에 대한 vtbl까지 꼬여 있어서 아주 심난할 지경입니다.
* 다중 상속에 그림자처럼 따라오는 것이 가상 기본 클래스(virtual base class)입니다. 가상 기본 클래스도 그 나름대로의 비용 부담을 가져옵니다. 데이터 멤버의 중복을 피하기 위해 가상 기본 클래스 부분에 대한 포인터를 객체에 넣어 둔다던지 해야 하기 때문입니
* 다음 코드와 같은 다중 상속 관계가 있다고 가정합시다.
```c++
class A { ... };
class B: virtual public A { ... };
class C: virtual public A { ... };

class D: public B, public C { ... };
```
* 음.. 이후의 설명은 일단 스킵
* 마지막 이야기는 RTTI(runtime type identification, 런타임 타입 식별) 기능에 대한 것입니다.
* RTTI는 실행중에 객체와 클래스 정보를 알아낼수있는 기능이지만, 그 정보를 저장해둘 어딘가도 필요합니다. 
    * 이 정보는 type_info 라는 타입의 객체에 저장하고 이 thype_info 객체는 typeid연산자를 써서 엑세스 합니다. 

* RTTI 정보는 클래스마다 하나씩 있으면 되겠고  이 정보를 어느 객채에서든지 뽑아낼수있을것 같지만, 실제론 그렇지 않음
    * C++ 스펙에 의하면, 객체의 동적 타입을 정확히 뽑아낼 수 있으려면 그 타입(클래스)에 가상 함수가 최소한 하나는 있어야 한다고 함
    * RTTI 가 가상함수가 비슷하게느껴지는데 이는 애초부터 RTTI는 클래스의 vtbl를 통해 구현될수 있도록 설계되었음
* 이를테면 컴파일러가 vtbl  배열의 인덱스가 0인 위치에 그 vtabl에 해당하는 클래스에 대한 type_info 객체의 포인터가 들어있는식으로 vtbl을 구현함
* 이 항목의 가장 처음 그림의 C1 클래스의 vtbl은 다음과같이 나타낼수 있음.
```
C1's vtbl 
----
|  |  ---> C1의 type_info 객체
----
|  |  ---> C1::~C1의 함수 코드
----
|  |  ---> C1::f1의 함수 코드
----
|  |  ---> C1::f2의 함수 코드
----
|  |  ---> C1::f3의 함수코드
----
```


# Chpater 5 유용하고 재미있는 프로그래밍 기법들(Techniques)

## 항목 25 : 생성자 함수와 비(非)멤버 함수를 가상 함수처럼 만드는 방법
* 가상 생성자라는 말은 앞뒤가 안맞는 말 같지만. 
* 가상 생성자란, 자신이 받은 입력 데이터에 의존하여 다른 타입의 객체를 생성하는 함수를 일컫습니다.
* 아래의 예를 보며 설명할께요. 뉴스레터를 만들고 관리하는 소프트웨어를 만든다고 가정합시다.
```c++
class NLComponent {        // 뉴스레터의 구성요소를
public:                    // 나타내는 추상 기본 클래스
 
    ...                    // 추상 클래스이므로, 순수 가상 함수가
};                        // 최소한 하나는 들어 있습니다.
 
class TextBlock: public NLComponent {
public:
    ...                    // 순수 가상 함수가
};                        // 없습니다.
 
class Graphic: public NLComponent {
public:
    ...                    // 순수 가상 함수가
};                        // 없습니다.
 
class NewsLetter {        // 뉴스레터 객체 하나에는 뉴스레터 구성요소
public:                    // 를 나타내는 NLComponent 객체가
    ...                    // 리스트로 관리됩니다.
    
private:
    list<NLComponent*> components;
};
 
// NewsLetter 객체는 컴퓨터의 메모리에 올라와
// 쓰이지 않을 때에는 디스크에 저장해 두는 것이 대개
// 일반적입니다. 그렇다면 istream을 취하는 생성자를
// NewsLetter에 넣어 두는 것이 꽤 괜찮을
// 것입니다.
 
class NewsLetter {
public:
    ...
private:
    // str에서 바로 다음의 NLComponent 데이터를 읽고,
    // 메모리에 객체를 생성하고 데이터를 세팅한 후에 포인터를 반환합니다.
    static NLComponent * readComponent(istream& str);
    ...
};
 
// NewsLetter의 생성자
NewsLetter::NewsLetter(istream& str)
{
    while (str) {
        // readComponent에서 반환된 포인터를 뉴스레터 구성요소 리스트
        // 의 끝에 추가합니다.
        components.push_back(readComponent(str));
    }
}
 
// readComponent 함수는 런타임에 디스크에서 읽은
// 정보에 따라 다른 타입의 객체를 만들기 때문에 이 함수를
// 가상 생성자(virtual constructor)라고 부를 수
// 있습니다.
```
* readComponent 란 삼수가 어떤 동작을 하는지 짐작이 되지요?
    * 뉴스레터에 들어가는 구성 요소를 나타내는 객체를 새로 생성해야하는데, 이 객체는 디스크에서 읽은 데이터에 따라 TextBoock 든지 Graphic 타입이 됩니다. 어쨌든 새 객체를 생성하는것이기에 이 함수는 생성자 처럼 동작해야 할 것입니다. 
    * 그런데 런타임에 디스크에서 읽은 정보에 따라 다른 타입의 객체를 만들기 때문에 이 함수를 가상 생성자라고 부를수 있는 것입니다. 

* 가상 생성자 중 특히 널리 쓰이는 놈이 가상 복사 생성자(virtual copy constructor)입니다. 
    * 가상 복사 생성자는 이것을 호출한 객체를 그대로 본 뜬 사본의 포인터를 반환합니다. 
    * 이런 성질 때문에 copySelf, cloneSelf 아니면 예제와 같이 간단하게 clone 등으로 불리기도 합니다.
```c++
class NLComponent {
public:
    // 가상 복사 생성자의 선언
    virtual NLComponent * clone(void) const = 0;
    ...
};
 
class TextBlock: public NLComponent {
public:
    virtual TextBlock * clone(void) const {
        return new TextBlock(*this);    // 가상 복사
    }                                    // 생성자
    ...
};
 
class Graphic: public NLComponent {
public:
    virtual Graphic * clone(void) const {
        return new Graphic(*this);        // 가상 복사
    }                                    // 생성자
    ...
};
 
// C++의 가상 복사 생성자는 비교적 최근에 완화된
// 가상 함수의 반환 타입 규칙을 사용한 기법입니다.
 
// 완화된 규칙에 의하면, 기본 클래스의 가상 함수를
// 재정의한 파생 클래스의 가상 함수는 똑같은 반환
// 타입을 갖지 않아도 됩니다.
 
// NLComponent에 가상 복사 생성자가 들어가고 나면,
// NewsLetter의 (보통) 복사 생성자도 훨씬 쉽게
// 구현할 수 있게 됩니다.
 
class NewsLetter {
public:
    NewsLetter(const NewsLetter& rhs);
    ...
private:
    list<NLComponent*> components;
};
 
NewsLetter::NewsLetter(const NewsLetter& rhs)
{
    // rhs 안에 있는 리스트에 대해 루프를 돌면서, 각 리스트 요소의
    // 가상 복사 생성자를 호출하여 해당되는 뉴스레터 구성요소를
    // 복사합니다.
    for (list<NLComponent*>::const_iterator it =
        rhs.components.begin();
        it != rhs.components.end();
        ++it) {
            components.push_back((*it)->clone());
    }
}
```

* 보셨듯 가상 복사 생성자는 해당 클래스의 진짜 복사 생성자를 호출하는것으로 끝.
    * 진짜 복사 생성자가 하는 복사는 얕은 복사(shallow copy)이면 가상 복사 생성자가 하는것도 얕은 복사
    * 진짜 복사 생성자가 깊은 복사(deep copy)이면 가상도 깊은 복사를 합니다.
* 비멤버 함수를 가상 함수처럼 동작하게 만드는 방법
    * 원하는 일을 하는 가상 함수를 만들어 놓고, 비가상 함수가 이것을 호출하게 만드면 됨
    * 함수가 한번 더 호출되기 때문에 걱정되는 비용은 인라인으로 선언해서 없애면 됨.


## 항목 26 : 클래스 인스턴스의 개수를 의도대로 제한하는 방법
### 객체를 전혀 생성하지 않거나 한 개만 생성하기
* 특정한 클래스의 객체가 만들어지지 않게 하는 가장 쉬운 방법은 그 클래스의 생성자를 private로 선언하는것
```c++
class CanBEInstantiated{
private:
    CanBEInstantiated();
    CanBEInstantiated(const CanBEInstantiated&);
    ...
};
```
* 이렇게 해버리면 CanBEInstantiated 클래스는 절대로 생성할수 없는 상태가 된다. 이것을 출발점으로 조금씩 제약을 풀어가는것이다. 
* 예를들어 프린터를 제어하는 클래스의 객체를 만들어야 하는데, 사용 가능한 프린터 객체의 개수를 한 개로 제한하고 싶다면, 프린터 객체를 생성자 함수 안에 그냥 넣는 것입니다.

```c++
class PrintJob;//전방참조를 위한 클래스 선언
 
class Printer {
public:
    void submitJob(const PrintJob& job);
    void reset();
    void performSelfTest();
    ...
    friend Printer& thePrinter();
private:
    printer();
    printer(const Printer& rhs);
    ...
};
 
Printer& thePrinter()
{
    static Printer p;    // 프린터 객체는 하나입니다.
    return p;
}
```
* 키 포인트는 세가지
    * 1. Printer 클래스의 생성자가 private 임. (사용자 맘대로 소스 코드에서 생성 못함)
    * 2. 전역 함수인 thePrinter 가 클래스의 프랜드로 선언 (이때문에 thePrinter는 private로 선언된 생성자의 굴레에서 벗어날수 있게됨)
    * 3. thePrinter에는 Printer 객체가 정적 객체로 들어있음(딱 하나만 만들어짐.)
```c++
class PrintJob {
public:
    PrintJob(const string& whatToPrint);
    ...
};
 
string buffer;
...                        // buffer에 출력할 것을 채웁니다.
thePrinter().reset();
thePrinter().submitJob(buffer);
```
* 사용자는 프린터 객체를 쓰고 싶을때마다 언제나 thePrinter 를 불러낼수 있음
    * thePrinter는 Printer객체의 참조자를 반환하기에 부담없이 사용 가능함.

```c++
//-----------------------------------------------------------------------------
// 전역에 thePrinter함수를 놓는 것이 걱정된다면,
// thePrinter함수를 Printer 클래스의 정적 멤버 함수로
// 만들면 됩니다.
class Printer {
public:
    static Printer& thePrinter(void);
    ...
private:
    Printer(void);
    Printer(const Printer& rhs);
    ...
};
 
Printer& Printer::thePrinter(void)
{
    static Printer p;
    return p;
}

//하지만 이러면 코드가 약간 더 길어지긴 함.
Printer::thePrinter().reset();
Printer::thePrinter().submitJob(buffer);
```

* thePrinter함수를 네임스페이스 안에 넣어버리는 방법도 있습니다.
```c++
namespace PrintingStuff {
    class Printer {        // 이 클래스가 PrintingStuff
    public:                // 네임스페이스에 들어가게 되었습니다.
        void submitJob(const PrintJob& job);
        void reset(void);
        void performSelfTest(void);
        ...
    friend Printer& thePrinter(void);
    private:
        Printer(void);
        Printer(const Printer& rhs);
        ...
    };
    
                        // 이 함수도 마찬가지
    Printer& thePrinter(void)
    {
        static Printer p;
        return p;
    }
}

//이렇게 사용해야 합니다. 
PrintingStuff::thePrinter().reset();
PrintingStuff::thePrinter().submitJob(buffer);
 
// 네임스페이스를 일일히 타이핑하기 힘들다면
// using 선언(using declaration)을
// 활용할 수 있습니다.
 
using PrintingStuff::thePrinter;
 
thePrinter().reset();
thePrinter().submitJob(buffer);
```

* 앞의 예제 코드는 특징이 2개가 숨어져있음
    * 1. Printer 객체가 클래스의 정적 객체가 아니라 함수의 정적 객체라는 점입니다. 
        * 클래스의 정적 멤버로 선언된 객체는 그것이 사용되던 말던, 생성됨
        * 이와 대도적으로 함수 안에서 정적 변수로 선언된 객체는 그 함수가 최소한 한번 호출되어야 생성됨
        * 프린터 객체를 클래스의 정적 멤버로 정의하는 것이 나쁜 이유 하나 더는. 객체 초기화 시점과 관련됨
            * 클래스에 선언된 정적 객체는 그 초기화 시점이 정확하게 정의되어있지 않음
                * 이런 것의 문제를 피하기 위해선, 정적 객체를 함수에 선언하는 것이겠죠.
    * 2.  또 한가지 주의해야 할 점은 thePrinter함수와 같이 정적 객체를  선언한 비멤버 함수는 절대로 인라인으로 선언하면 안 됩니다.
    * 아래 예시를 보면서 설명해볼께요.
```c++
Printer& thePrinter()
{
    static Printer p;
    return p;
}
```
* 처음 호출될때 (p 가 생성될때)를 제외하면 이 함수는 그냥 한줄짜리 함수입니다.(return p). 효율을 높이기 위해 인라인을 해야할듯하지만 이 함수는 inline 로 선언되지 않았습니다.
* 정적 객체를 선언하는 이유는 그 객체의 사본이 하나만 필요하기 때문일텐데 inline은 컴파일러가 그 함수가 호출된 부분 대신에 그 함수의 몸체를 끼워놓고 처리하라고 알려주는 것입니다. 
    * 하지만 비 멤버 함수에 있어 inline은 다른 의미를 가집니다. 
        * 비멤버 함수에 있어서 inline은 "이 함수는 내부 연결 (internal linked)을 가진다."라는 뜻인데,  내부 연결을 갖는 함수는 한 프로그램 내에서 중복되어 존재할 수 있습니다. 즉 내부의 정적 변수도 두 개 이상 존재할 수 있게 됩니다.

* '숨겨진' 객체의 참조자를 반환하는 방법이 '좀 마음에 안 든다' 라고 생각하신다면 생성된 객체의 개수를 직접 세어서, 일정한 개수가 넘어갔을 때 예외를 일으키는 방법을 사용할 수도 있습니다.
```c++
class Printer {
public:
    class TooManyObjects{};            // 너무 많은 객체가
                                    // 요구될 때 발생시키려고
                                    // 만든 예외 클래스
    
    Printer(void);
    ~Printer(void);
    ...
private:
    static size_t numObjects;
    Printer(const Printer& rhs);    // 프린터 객체는 한 개로
                                    // 제한했으므로, 복사는
};                                    // 금지합니다.
 
// 클래스의 정적 멤버 변수이므로 초기화가 필요합니다.
size_t Printer::numObjects = 0;
 
Printer::Printer(void)
{
    if (numObjects >= 1) {
        throw TooManyObjects();
    }
    여느 때와 같은 객체 생성(필드 세팅) 과정을 진행합니다;
    ++numObjects;
}
 
Printer::~Printer(void)
{
    여느 때와 같은 객체 소멸 과정을 수행합니다;
    --numObjects;
}
```
* 직관적이고 단순합니다. 
### 객체 생성이 이루어지는 세 가지 상황
* 하지만 생성자와 소멸자를 이용해 객체의 수를 세는 방법에도 문제가 숨어 있습니다. 만일 Printer로부터 상속받아 만든 새로운 클래스가 있다면 새로운 클래스를 생성할때도 Printer 객체의 생성자고 호출되어 Printer 객체의 수를 제대로 셀 수 없게 됩니다. Printer 객체가 다른 객체에 포함될 때에도 비슷한 문제가 발생합니다.
```c++
class ColorPrinter: public Printer{...};

//아래처럼 하면 Printer 객체는 총 몇개일까요?
Printer p;
ColorPrinter cp;
//답은 2개 입니다. 
```
* 여기서 문제가 되는 점은 Printer 객체가 세 가지 상황에서 생성될 수 있다는 점입니다.
    * 첫 번째: 그 자체로 생성될 때
    * 두 번째: 파생된 객체의 기본 클래스 부분으로 만들어질 때
    * 세 번째: 다른 객체의 클래스 멤버로서 만들어질 때

* 이런 경우에는 앞서 보여드렸던 생성자를 private로 선언하는 방법을 사용하면 소기의 목적을 간단히 달성할 수 있습니다. 왜냐하면, 생성자가 private로 선언된 클래스는 기본 클래스로 사용될 수 없으며, 다른 객체의 클래스 멤버로 들어갈 수도 없기 때문입니다. private 생성자를 가진 클래스를 기본 클래스로 쓸 수 없다는 사실은, 클래스 상속을 막는 일반적인 방법으로 끌어 낼 수도 있습니다.
```c++
// 유한 상태 오토마타(finite state automata)
// 를 C++로 만들기 위해 FSA라는 클래스를
// 하나 정의했다고 가정합시다.
 
// 이 FSA클래스는 객체 생성은 몇 개라도
// 할 수 있지만, 이 클래스로부터의 상속은
// 절대로 할 수 없도록 만들고 싶습니다.
 
class FSA {
public:
    // 유사 생성자(pseudo-constructor)
    static FSA * makeFSA(void);
    static FSA * makeFSA(const FSA& rhs);
    ...
private:
    FSA(void);
    FSA(const FSA& rhs);
    ...
};
 
FSA * FSA::makeFSA(void)
{ return new FSA(); }
 
FSA * FSA::makeFSA(const FSA& rhs)
{ return new FSA(rhs); }
 
// 유사 생성자가 호출될 때마다 new가
// 호출된다는 사실에는, 이 함수를 호출한
// 쪽에서 반드시 delete를 호출해야 한다는
// 의미가 배어 있습니다.
 
// 이를 좀 더 개선하려면
// auto_ptr이나 shared_ptr과 같은
// 스마트 포인터를 사용하면 됩니다.
```
### 객체를 불러냈다가 들여보내는 것도 자유롭게 하고 싶다.
* 오직 한 개의 인스턴스만 허용하는 클래스 설계를 알아 보았다. 이와 함께 객체가 생성되는 경우가 세 가지라는 이유때문에 클래스 인스턴스 개수를 추적하는 일을 꽤 복잡하다는 것도 알고있으면 이런 것을 없애는 방법은 생성자를 privat로 만들어 버리는 사실을 알았다. 
* thePrinter 함수를 설계한 방법은 Printer 객체의 개수를 딱 한 개로 묶어버리는 것이지만, 이 방법은 실행되는 프로그램 하나에 사용되는 Printer 객체의 개수를 딱 한 개로 묶어버리기도 합니다. 즉 다음과 같은 코드를 쓸 수 없습니다
```
Printer 객체 p1을 생성합니다;
p1을 사용합니다;
p1을 소멸시킵니다;

Printer 객체 p2를 생성합니다;
p2를 사용합니다;
p2를 소멸시킵니다;
```
* 이를 해결하기 위한 방법도 당연히 있습니다. 방금 전에 보았던 유사 생성자와 앞에서 사용했던 카운팅 코드를 합치면 일은 끝납니다.
```c++
//-----------------------------------------------------------------------------
class Printer {
public:
    class TooManyObjects{};
    
    // 유사 생성자 두 개
    static Printer * makePrinter(void);
    
    ~Printer(void);
    void submitJob(const PrintJob& job);
    void reset(void);
    void performSelfTest(void);
    ...
private:
    static size_t numObjects;
    Printer(void);
    Printer(const Printer& rhs);    // 이 함수는 정의하지 않습니다.
};                                    // 왜냐하면 객체의 복사를
                                    // 허용하지 않을 테니까요.
// 클래스의 정적 멤버이기 때문에 꼭 정의해야 합니다.
size_t Printer::numObjects = 0;
 
Printer::Printer(void)
{
    if (numObjects >= 1) {
        throw TooManyObjects();
    }
    여느 때와 같은 객체 생성(필드 세팅) 과정을 진행합니다;
    ++numObjects;
}
 
Printer * Printer::makePrinter(void)
{ return new Printer; }
 
// 객체 생성이 요청된 횟수가 일정 횟수를 초과할 때, 예외를 일으키는 방법 대신 널 포인터를 반환하도록
// 만들 수도 있습니다. 물론, 클래스 사용자로 하여금 포인터의 널 체크를 통해 상태를 파악하도록 해야 하겠지요.

Printer p1;                            // 에러입니다! 기본 생성자는
                                    // private입니다.
 
Printer *p2 =                         // 문제없습니다. 기본 생성자를
    Printer::makePrinter();            // 간접적으로 호출합니다.
 
Printer p3 = *p2;                    // 에러입니다! 복사 생성자는
                                    // private입니다.
 
p2->performSelfTest();                // 이외의 다른 함수들은
p2->reset();                        // 보통처럼 호출됩니다.
...
delete p2;                            // 리소스 누수를 막습니다.
                                    // 만약 p2가 스마트 포인터의 인스턴스였으면,
                                    // 이런 삭제도 필요없습니다.
 
 
//------------------------------------1-----------------------------------------
// 이 방법은 객체의 생성 개수를 임의의 개수로 제한하는 쪽으로 간단히 일반화 할 수 있습니다.
// 고정되어 정의된 상수 1을 다른 값으로 바꾸고 복사 생성자를 적절히 수정해서 제한을 풀어주면 됩니다.
// 객체가 10개까지 생성되도록 고친것은 아래와 같습니다. 
 
class Printer {
public:
    class TooManyObjects{};
    
    // 유사 생성자
    static Printer * makePrinter(void);
    static Printer * makePrinter(const Printer& rhs);
    ...
private:
    static size_t numObjects;
    static const size_t maxObjects = 10;
    
    Printer(void);
    Printer(const Printer& rhs);
};
 
// 클래스의 정적 멤버이기 때문에 꼭 정의해야 합니다.
size_t Printer::numObjects = 0;
const size_t Printer::maxObjects;
 
Printer::Printer(void)
{
    if (numObjects >= maxObjects) {
        throw TooManyObjects();
    }
    ...
    ++numObjects;
}
 
Printer::Printer(const Printer& rhs)
{
    if (numObjects >= maxObjects) {
        throw TooManyObjects();
    }
    ...
    ++numObjects;
}
 
Printer * Printer::makePrinter(void)
{ return new Printer; }
 
Printer * Printer::makePrinter(const Printer& rhs)
{ return new Printer(rhs); }
```

### 인스턴스 카운팅(Object-Counting) 기능을 가진 기본 클래스
* Printer와 같이 객체의 개수를 제한해야 하는 클래스가 매우 많이 필요하다면 관련 코드에 많은 중복이 일어날 것은 불을 보듯 뻔합니다. 물론, 인스턴스 카운팅 기능을 가진 기본 클래스를 만들어서 Printer같은 클래스에서 이것을 상속하게 하면, 눈앞의 불은 끌 수 있습니다. 하지만 조금만 더 생각하면 좀더 깔끔하고 더 좋은 기능을 뽑아낼 수도 있습니다.

* Printer 클래스의 객체 개수를 유지하는 값은 정적 변수로 선언된 numObjects에 저장됩니다. 따라서 이 변수를 인스턴스 카운팅 클래스에 넣으면 될 것 같습니다. 하지만, 인스턴스의 수를 세는 클래스마다 별도의 카운터를 두어야 한다는 점도 염두해 두어야 합니다. 이런 고민의 결과로 만든 것이 예제의 클래스 템플릿입니다. 인스턴스 카운터는 이 클래스 템플릿으로부터 만들어진 클래스의 정적 멤버로 들어가기 때문에, 원하는 개수만큼의 카운터를 자동으로 만들 수 있습니다.

```c++
template <class BeingCounted>
class Counted {
public:
    class TooManyObjects{};        // 예외를 일으킬 때 사용하는 객체
    static size_t objectCount(void) {
        return numObjects;
    }
    
protected:
    Counted(void);
    Counted(const Counted& rhs);
    
    ~Counted(void) {
        --numObjects;
    }
    
private:
    static size_t numObjects;
    static const size_t maxObjects;
    void init(void);            // 생성자 코드를 중복해서 작성하는
};                                // 일을 막기 위해 만든 함수
 
template <class BeingCounted>
Counted<BeingCounted>::Counted(void)
{ init(); }
 
template <class BeingCounted>
Counted<BeingCounted>::Counted(const Counted<BeingCounted>&)
{ init(); }
 
template <class BeingCounted>
void Counted<BeingCounted>::init(void)
{
    if (numObjects >= maxObjects) {
        throw TooManyObjects();
    }
    ++numObjects;
}
 
// 이 템플릿으로부터 만들어지는 클래스는 기본 클래스로만
// 사용되도록 설계되었기 때문에, 생성자와 소멸자가 모두
// protected로 선언되어 있습니다.
 
// 이제 Printer 클래스를 Counted 템플릿을
// 사용하도록 고쳐보겠습니다.
 
class Printer: private Counted<Printer> {
public:
    // 유사 생성자들
    static Printer * makePrinter(void);
    static Printer * makePrinter(const Printer& rhs);
    ~Printer(void);
    void submitJob(const PrintJob& job);
    void reset(void);
    void performSelfTest(void);
    ...
    using Counted<Printer>::ObjectCount;        // 다음을 보세요.
    using Counted<Printer>::TooManyObjects;        // 다음을 보세요.
private:
    Printer(void);
    Printer(const Printer& rhs);
};
 
// 만일 Printer클래스가 Counted<Printer>를
// public 상속한다면 Counted의 소멸자가
// 가상 소멸자가 되어야 할 것입니다. 항목 24에서 보았듯
// 그에 따른 비용도 지불해야 하죠. private 상속은
// 이런 문제를 해결해 줍니다.
 
// private 상속 때문에 Counted의 public으로 선언된
// 멤버함수 마저 private가 되어 버렸습니다.
// 따라서, 이 함수가 원래 가지고 있는 public 성질(접근성)을
// 되돌리기 위해 using 선언이 사용되었습니다.
 
// 예외가 발생했을 때 받을 수도 있어야 하므로
// TooManyObjects도 마찬가지로
// using 선언을 사용해 줍니다.
 
// 이제 Printer는 더 이상 객체의 개수에 대해
// 신경쓰지 않아도 됩니다. 인스턴스 카운팅은 Counted<Printer>가
// 알아서 해 주기 때문입니다.
 
// 마무리지어야 할 부분이 아직 남아 있습니다.
// 무엇인고 하니, Counted 안에 선언되어 있는
// 정적 상수 멤버를 두고 하는 말입니다.
 
// 일단 numObjects는 처리하기 쉽습니다.
// Counted의 구현 파일에 다음과 같이 쓰기만 하면 됩니다.
 
template <class BeingCounted>                    // numObjects를 구현 파일에
size_t Counted<BeingCounted>::numObjects;        // 정의하면, 자동으로 0으로
                                                // 초기화됩니다. 구식의 컴파일러는
                                                // 0으로 초기화 하지 않을 수도 있으니
                                                // 주의하기 바랍니다.
 
// maxObjects의 경우에는 BeingCounted의 타입에
// 따라 다른 값으로 초기화 되어야 합니다. 이럴땐 고민하지
// 말고 클래스 사용자에게 초기화를 맡겨 버립시다. 즉,
// Printer 클래스의 경우에는 Printer를 만드는 사람이
// 직접 구현 파일에 다음과 같은 초기화 문장을 넣어야 합니다.
 
const size_t Counted<Printer>::maxObjects = 10;
```


## 항목 27 : 힙(heap)에만 생성되거나 힙에는 만들어지지 않는 특수한 클래스를 만드는 방법
* 프로그래밍 하다보면 특정 클래스의 객체는 스스로 죽을수 있도록 delete this를 할수 있도록 만들고 싶은 경우가 있다.
    * 이렇게 하려면 그 객체가 힙에서 할당된 것이어야 하는데. 이와 반대로 어떤 클래스에 대해선 절대 메모리 누수가 일어나지 않도록 안전책을 만들고 싶기도 한다.
    * 이렇게 하려면 이런 클래스의인스턴스는 힙에 할당되지 않도록 해야 한다. 
* 이런 저런 요구사항에 맞춰 heap  기반객체를 요구하거나 금지하는 게 가능할까?

### 객체가 힙에만 생성되게 하기 
* 개발자(사용자)들이 new 이외의 방법으로는 객체를 생성할 수 없도록 하면 됩니다. 
    * 힙에 만들어지지 않는(Non-heap) 객체는 소스 코드에 선언된 위치에서 자동으로 생성되고, 자신의 유효범위(scope)가 끝나는 위치에서 자동으로 소멸되기 때문에, 이런 방식의 암시적(implicit) 객체 생성/소멸을 불법(illegal)화 시키면 간단히 해결됩니다.

* 생성자와 소멸자를 private로 선언하고 대신 유사 생성/소멸자(항목 26 참조)를 통해 간접적으로 호출할 수 있도록 하면, 가장 확실히 소기의 목적을 달성할 수 있습니다. 
    * 하지만 굳이 생성자와 소멸자를 모두 private로 선언할 필요는 없고 소멸자만 private로 선언해도 목적을 달성할 수 있습니다. 생성자는 개수가 많을 수도 있기 때문에 모든 생성자를 private로 선언하기가 번거로울 수도 있어서, 소멸자만 private로 만드는 편이 더 낫습니다.

* 이제 예를 들어 볼께요.
```c++
class UPNumber {
public:
    UPNumber(void);
    UPNumber(int initValue);
    UPNumber(double initValue);
    UPNumber(const UPNumber& rhs);
    
    // 유사 소멸자(const 멤버 함수입니다. 왜냐하면
    // 상수 객체도 소멸될 수 있기 때문입니다).
    void destroy(void) const {
        delete this;
    }
    ...
private:
    ~UPNumber(void);
};
 
UPNumber n;                    // 에러입니다! (지금은 불법이 아니지만,
                            // 나중에 n의 소멸자가 암시적으로
                            // 호출될 때 불법이 됩니다)
 
UPNumber *p = new UPNumber;    // 문제없습니다.
...
delete p;                    // 에러입니다! private로 선언된
                            // 소멸자는 호출할 수 없습니다.
 
p->destroy();                // 문제없습니다.

```
* 모든 생성자를 private로 선언할 수도 있습니다. 
    * 이 방법의 단점은 고려해야할 생성자가 많다는 것입니다. 
        * 복사 생성자, 기본 생성자 
    * 결론적으로 소멸자만 private로 만드는 것이 속 편합니다. 
* 하지만 생성자나 소멸자를 private로 선언하면 항목 26에서 이야기했듯이 이 클래스를 상속하거나 이 클래스 인스턴스를 다른 객체의 멤버로 넣는 일을 모두 할 수 없게 됩니다.

* 이런 문제는 어렵지 않게 해결할 수 있습니다. 상속이 안 되는 문제는 소멸자를 protected로 바꾸면 해결할 수 있고(생성자는 public으로 유지하고요), 힙에만 생성되는 객체를 다른 객체에 넣지 못하는 문제는 객체 대신에 그 객체의 포인터를 넣으면 간단히 해결할 수 있습니다.

```c++
class UPNumber { ... };                // 소멸자를 protected로 선언한 클래스
 
class NonNegativeUPNumber:            // 이젠 문제없습니다. 파생
    public UPNumber { ... };        // 클래스는 protected 멤버에
                                    // 접근할 수 있습니다.
 
class Asset {
public:
    Asset(int initValue);
    ~Asset(void);
    ...
private:
    UPNumber *value;
};
 
Asset::Asset(int initValue)
: value(new UPNumber(initValue))    // 문제없습니다.
{ ... }
Asset::~Asset(void)
{ value->destroy(); }                // 역시 문제없습니다.
```

### 어떤 객체가 힙에 생성되었는지, 그렇지 않은지를 알아내는 방법
* 소멸자를 protected로 선언하여 객체가 힙에만 생성되도록 만든 클래스 UPNumber를 상속받아 구현된 클래스 NonNegativeUPNumber가 있고, NonNegativeUPNumber의 객체는 힙에만 생성된다는 보장이 없는 경우에 NonNegativeUPNumber의 UPNumber부분 역시 힙에만 생성된다고 보장할 수 없게 됩니다. 이 때 UPNumber가 힙에 생성되었는지, 혹은 그렇지 않은지를 알아내는 방법이 있을까요?

```c++
NonNegativeUPNumber * n1 = new NonNegativeUPNumber;//힙에 있습니다.
NonNegativeUPNumber n2;//힙에 없습니다. 
```

* 다음 몇 가지 방법은 어떤 객체가 힙에 생성되었는지, 혹은 그렇지 않은지를 알아내는 바람직하지 못한 예시 3 가지 입니다.
* 1. operator new 함수와 operator new[] 함수를 오버로딩해서 동적 할당시에, bool 타입의 정적 변수의 값을 true로 만들고 생성자에서 이 값을 점검하는 방법이 있습니다. 이 방법은 동적 배열 생성과 같은 상황 등에서 제대로 작동하지 않습니다.
```c++
class UPNumber {
public:
    // 힙 기반이 아닌 객체가 생성될 때 일으킬 예외 클래스
    class HeapConstraintViolation {};
    
    static void * operator new(size_t size);
    
    UPNumber(void);
    ...
private:
    static bool onTheHeap;        // 객체가 힙에서 만들어지든
    ...                            // 그렇지 않든 생성자는
};                                // 호출됩니다.
 
// 클래스 정적 멤버는 반드시 정의해야 합니다.
bool UPNumber::onTheHeap = false;
 
void *UPNumber::operator new(size_t size)
{
    onTheHeap = true;
    return ::operator new(size);
}
 
UPNumber::UPNumber(void)
{
    if (!onTheHeap) {
        throw HeapConstraintViolation();
    }
    
    여느 때와 같은 객체 생성(필드 세팅) 과정을 진행합니다;
    
    onTheHeap = false;            // 다음 객체를 위하여 플래그를 초기화합니다.
}
 
// 이런 식의 구현이죠.
 
// 여기서는 operator new[]를 오버로딩 하지
// 않았지만 오버로딩 해도 마찬가지입니다.
 
// 다음을 보세요.
 
UPNumber *numberArray = new UPNumber[100];
 
// operator new[]를 오버로딩 했다고 하더라도
// operator new[]는 처음 메모리를 할당할 때
// 딱 한 번만 불립니다. 따라서 배열의 첫 객체는
// 제대로 생성자가 호출되지만 그 다음 객체부터는
// onTheHeap이 false가 되어 생성자에서
// 예외가 발생할 것입니다.
 
// 배열의 경우가 아니더라도 다음과 같은 코드를 작성한 경우
// 문제가 생길 수 있습니다.
 
UPNumber *pn = new UPNumber(*new UPNumber);
 
// 이 코드는 자원이 누출되지만 지금 보고자 하는 것은
// 자원의 누출이 아닙니다.
 
// 위의 코드를 작성한 프로그래머는 다음과 같은 순서대로
// 작동할 것이라고 생각했을 것입니다.
 
// 1. 첫째 객체(가장 왼쪽 것)에 대해 operator new를 호출함
// 2. 첫째 객체에 대해 생성자를 호출함
// 3. 둘째 객체에 대해 operator new를 호출함
// 4. 둘째 객체에 대해 생성자를 호출함
 
 
// 하지만 C++ 언어 스펙에서는 반드시 이 순서대로 작동한다고
// 못박은 규정을 찾아볼 수 없습니다. 즉 컴파일러에 따라
// 다음과 같은 순서대로 작동할 가능성도 있습니다.
 
// 1. 첫째 객체에 대해 operator new를 호출함
// 2. 둘째 객체에 대해 operator new를 호출함
// 3. 첫째 객체에 대해 생성자를 호출함
// 4. 둘째 객체에 대해 생성자를 호출함
 
 
// 이렇게 되면 배열을 생성할 때와 마찬가지로
// 두 번째 객체의 생성자가 불릴 때에는 onTheHeap의 값이
// false일 것이므로 예외가 발생할 것입니다.
```

* 2. 프로그램에서 스택 영역과 힙 영역의 위치가 분리되어 있다는 점(대부분의 시스템은 상위 주소의 스택 영역이 아래로 자라고, 하위 주소의 힙 영역이 위로 자랍니다)을 이용하여 객체가 할당된 주소의 크기를 비교하는 방법을 쓸 수 있습니다. 이 방법의 문제는 두 가지가 있는데, 우선 이식성이 떨어집니다. 드물지만 어떤 시스템에서는 메모리에서 힙과 스택의 주소가 반대로 배치되어 있는 경우도 있다고 합니다.
    * 이식성 이외의 다른 문제는, 바로 객체가 생성되는 장소가 세 가지라는 점입니다. 스택과 힙 이외에 정적 객체(static이 붙은 객체 혹은 전역이나 네임스페이스 안에 선언된 객체)가 만들어지는 메모리 공간이 따로 있는데, 이 공간의 위치는 시스템 마다 다릅니다. 하지만 스택과 힙 영역이 상대방 쪽으로 자라도록 하는 시스템에서는 대개 정적 객체를 힙의 아래쪽에 둡니다. 즉 단순히 스택에 생성된 객체와의 객체 주소 비교를 통해서는 힙에 생성되었는지, 정적 객체의 영역에 생성되었는지를 구별할 수 없게 됩니다.

```c++
// 어떤 주소값이 힙 영역에 있는지 알아내는
// 바람직하지 않은 방법
bool onHeap(const void *address)
{
    char onTheStack;        // 스택에 만들어지는 지역 변수
    
    return address < &onTheStack;
}
 
void allocateSomeObject(void)
{
    char *pc = new char;    // 힙 객체입니다. 따라서
                            // onHeap(pc)는 true를 반환합니다.
    
    char c;                    // 스택 객체입니다. 따라서
                            // onHeap(&c)는 false를 반환합니다.
    
    static char sc;            // 정적 객체입니다. 그런데
                            // onHeap(&sc)는 true를 반환합니다.
    ...
}
```

* 3. 이렇게, 어떤 객체가 힙에 생성되었는지, 혹은 그렇지 않은지를 알아내기란 힘듭니다. 그럼에도 불구하고 어떤 객체가 힙에 생성되었는지를 알아내려고 하는 가장 그럴듯한 이유는 어떤 포인터에 delete를 해도 괜찮은지 확실히 알고 싶기 때문이라는 겁니다. 하지만 힙에 생성되었다고 하더라도 항상 100% 안전하게 delete를 할 수 있는 것은 아닙니다. 다음의 예를 보세요.
```c++
class Asset {
private:
    UPNumber value;
    ...
};
Asset *pa = new Asset;

// 이 경우 pa->value는 힙에 생성된 것이 분명하지만, new를 통해 생성되지 않았기 때문에 delete를 하면 재앙이 온다는 사실도 확실합니다.
```
* 다행스러운 사실은 어떤 포인터가 힙 주소를 가리키는지 알아내는 방법보다, 어떤 포인터가 삭제(delete)해도 괜찮은 포인터인지를 알아내는 것이 더 쉽다는 것입니다. operator new에서 지금 까지 할당된 주소들의 콜렉션을 유지하면 구현이 가능하기 때문입니다.
```c++
void *operator new(size_t size)
{
    void *p = getMemory(size);        // 메모리를 할당하고
                                    // 가용 메모리가 없는
                                    // 상황을 처리해 주는
                                    // 함수를 호출합니다.
    p를 할당된 주소 콜렉션에 추가합니다;
    return p;
}
 
void operator delete(void *ptr)
{
    releaseMemory(ptr);                // 관리 장치에게
                                    // 현재의 메모리를 돌려 줍니다.
    할당된 주소 콜렉션에서 ptr을 제거합니다;
}
 
bool isSafeToDelete(const void *address)
{
    address가 할당된 주소 콜렉션에 있는지 없는지의 여부를
    반환합니다;
}
```
* 하지만 이 방법에도 세 가지 문제점이 있습니다.
    * 첫째는, '전역 함수는 객체지향의 세계에선 아무래도 껄끄럽다'라는 것입니다. 특히 operator new와 operator delete는 미리 정의된 동작이 있기 때문에 더욱 그렇습니다. 앞에서와 같이 만들어진 소프트웨어는 operator new와 operator delete의 전역 함수를 구현한 또 다른 소프트웨어(많은 객체지향 데이터베이스 시스템)와 잘 작동하지 않게 됩니다.
    * 두 번째 걸림돌은 효율입니다. 힙 할당을 할 때마다 매번 할당된 주소의 정보를 갱신하고 유지해야 하는데, 이 오버헤드를 무시하지 못한다는 것은 분명한 부담이 됩니다.
    * 마지막 걸림돌은, 평범하지만 아주 중요한 문제입니다. isSafeToDelete 함수를 모든 경우에 대해 동작하도록 구현하기가 사실상 불가능하다는 것입니다. 왜냐하면, 다중 상속으로 만들어진 객체나 가상 기본 클래스로부터 만들어진 객체는 주소도 여러 개이기 때문에, isSafeToDelete로 넘어간 address 인자가 operator new로부터 반환된 것과 똑같으리라는 보장을 할 수 없기 때문입니다(항목 24와 31 참조).

* 다행스럽게도 전역 네임스페이스, 효율 오버헤드, 부정확한 동작 등의 세 가지 걸림돌을 피하고 앞의 함수가 제공하는 기능을 그대로 쓸 수 있는 기법이 있습니다. 
* 바로 '추상 믹스인 기본 클래스(abstract mixin base class)'라는 것입니다.

```c++
class HeapTracked {                        // 믹스인 클래스입니다. 이 클래스는
public:                                    // operator new에서 반환된 ptr을 관리합니다.
    
    class MissingAddress {};            // 예외 클래스입니다.
    
    virtual ~HeapTracked(void) = 0;
    
    static void *operator new(size_t size);
    static void operator delete(void *ptr);
    
    bool isOnHeap(void) const;
    
private:
    typedef const void* RawAddress;
    static list<RawAddress> addresses;
};
 
// 정적 클래스 멤버이기 때문에 반드시 정의해야 합니다.
list<RawAddress> HeapTracked::addresses;
 
// HeapTracked의 소멸자는 순수 가상 함수로 선언하여 이 클래스를
// 추상 클래스로 만듭니다. 하지만 이 클래스도 여전히 정의해야
// 하기 때문에, 빈 블록이나마 만들었습니다.
HeapTracked::~HeapTracked(void) {}
 
void * HeapTracked::operator new(size_t size)
{
    void *memPtr = ::operator new(size);    // 메모리를 얻어냅니다.
    
    address.push_front(memPtr);                // 얻은 메모리 주소를
                                            // 리스트의 앞에 추가합니다.
    return memPtr;
}
 
void HeapTracked::operator delete(void *ptr)
{
    // ptr을 가지고 있는 리스트 엔트리를 가리키는 '반복자'를 얻어냅니다.
    list<RawAddress>::iterator it =
        find(addresses.begin(), addresses.end(), ptr);
    if (it != addresses.end()) {            // 포인터가 발견되면
        addresses.erase(it);                // 그 포인터를 제거합니다.
        ::operator delete(ptr);                // 그리고 메모리를 해제합니다.
    }
    else {                                    // 그렇지 않으면
        throw MissingAddress();                // ptr은 operator new에서
    }                                        // 할당된 것이 아니므로,
}                                            // 예외를 일으킵니다.
 
bool HeapTracked::isOnHeap(void) const
{
    // *this가 차지한 메모리의 첫 부분을 가리키는 포인터를
    // 얻어냅니다.
    const void *rawAddress = dynamic_cast<const void*>(this);
    
    // operator new에서 할당된 주소의 리스트에서 이 포인터를
    // 찾습니다.
    list<RawAddress>::iterator it =
        find(addresses.begin(), addresses.end(), rawAddress);
    
    return it != addresses.end();            // 포인터를 찾았는지 못 찾았는지의
}                                            // 여부를 반환합니다.
 
// 앞에서 isSafeToDelete는 만들기가 조금 복잡하다고
// 했습니다. isOnHeap도 마찬가지의 문제를 가지고 있는데,
// isOnHeap은 HeapTracked 객체에 대해서만 적용되기 때문에,
// dynamic_cast 연산자의 힘을 빌어 이 문제점을 없애볼 수
// 있습니다.
 
// dynamic_cast연산자를 적용할 수 있는 객체는
// 가상 함수를 최소한 한 개 가져야 합니다.
// 저주받은 isSafeToDelete 함수는 모든 타입의
// 포인터에 대해 동작해야 했기 때문에, dynamic_cast로
// 어떻게 해 볼 수가 없었죠.
 
// 하지만 isOnHeap은 HeapTracked 타입만 책임지면
// 되므로 현재 객체가 자리잡은 메모리의 시작 주소를 얻을 수
// 있는 것입니다.
 
// 위 클래스의 사용 방법은 아주 간단합니다. 다음을 보세요.
 
class Asset: public HeapTracked {
private:
    UPNumber value;
    ...
};
 
// 이제 Asset* 포인터가 힙 기반 객체를 가리키는지 알아보려면
// 다음과 같이 합니다.
 
void inventoryAsset(const Asset *ap)
{
    if (ap->isOnHeap()) {
        ap는 힙 기반 asset 객체입니다 - 그대로 저장해 둡니다.
    }
    else {
        ap는 힙 기반 객체가 아닙니다 - 다른 방법으로 기록합니다.
    }
}

```

### 객체가 힙에 생성되지 않게 하기
* 객체가 힙에 만들어지는 경우를 막는 방법은 아주 쉽습니다. 왜냐하면 힙 할당은 new를 통하지 않으면 이루어지지 않고, new의 사용을 막아버리기란 쉽기 때문입니다. 물론 new 연산자 자체를 어떻게 해 볼 수는 없지만 new 연산자가 내부적으로 operator new를 호출한다는 점과, 이 함수를 우리가 선언할 수 있다는 사실을 이용하면 됩니다. 바로 operator new를 private로 선언해 버리는 것입니다.

```c++
//-----------------------------------------------------------------------------
class UPNumber {
private:
    static void *operator new(size_t size);
    static void operator delete(void *ptr);
    ...
};
 
UPNumber n1;                    // 문제 없습니다.
static UPNumber n2;                // 또 문제 없습니다.
UPNumber *p = new UPNumber;        // 앗, 에러! private로 선언된
                                // operator new의 호출을 시도했습니다.
 
// operator new와 operator delete를 찢어 놓을
// 피치 못 할 이유가 없는 한 두 함수는 같은 성질로 선언하는 것이
// 좋습니다. 또 하나, UPNumber 객체의 배열을 힙에
// 만들지 못하게 하려면, operator new[]와
// operator delete[]도 private로 선언해야 합니다.
 
 
//-----------------------------------------------------------------------------
// 이렇게 만들면, 파생 클래스 객체의 기본 클래스 부분으로
// 객체가 인스턴스화되는 경우에도 힙에 생성되는 것을 막을 수 있습니다.
// 왜냐하면 operator new와 operator delete는
// 상속이 가능하기 때문에 기본 클래스에서 private였다면,
// 파생 클래스 역시 이를 고스란히 물려받기 때문입니다.
 
class NonNegativeUPNumber:        // 이 클래스에서는 operator new를
    public UPNumber {            // 선언하지 않았다고 가정합니다.
        ...
};
 
NonNegativeUPNumber n1;            // 문제 없습니다.
 
static NonNegativeUPNumber n2;    // 또 문제 없습니다.
 
NonNegativeUPNumber *p =        // 앗, 에러! private로 선언된
    new NonNegativeUPNumber;    // operator new의 호출을 시도했습니다.
 
 
//-----------------------------------------------------------------------------
// 파생 클래스가 자신만의 operator new를 또 선언하면,
// 파생 클래스의 객체를 힙에 할당할 때 이 함수가
// 호출된다는 사실에 주의해야 합니다.
 
// 비슷하게 UPNumber의 operator new가
// private라고 해서 UPNumber 객체를 멤버로
// 가지고 있는 다른 객체의 할당 방식까지 영향을 주는
// 것은 아니라는 점도 유념해야 하겠습니다.
 
class Asset {
public:
    Asset(int initValue);
    ...
private:
    UPNumber value;
};
 
Asset *pa = new Asset(100);        // 별 문제없습니다.
                                // Asset::operator new 혹은
                                // ::operator new가 호출되지,
                                // UPNumber::operator new는 아닙니다.
 
 
//-----------------------------------------------------------------------------
// 객체가 힙에 생성되지 못하게 하는 방법을 알았으니,
// 객체가 힙에 있을 때 예외를 일으키는 방법을 찾고
// 싶은 것이 인지상정입니다.
 
// 하지만 어떤 주소가 힙 영역에 있는지를 알아낼 수
// 있고 이식성까지 갖춘 방법을 찾을 수 없었던 것과
// 마찬가지로, 어떤 주소가 힙 영역에 없는지를 알아내는
// 이식성 있는 방법은 없습니다.
```



## 항목 28 : 스마트 포인터(Smart Pointer)
* 스마트 포인터는 C++언어에서 사용되는 보통의 포인터처럼 생겼고, 보통의 포인터처럼 동작하고, 보통의 포인터처럼 느껴지지만, 실상은 엄청난 기능을 제공하도록 설계된 C++ 객체입니다.
* C++의 기본 제공(built-in) 포인터(시쳇말로 벙어리(dumb) 포인터) 대신에 스마트 포인터를 사용하는 이유는 다음의 세 가지로 요악할 수 있습니다.
    * 생성(construction)과 소멸(destruction) 작업을 조절할 수 있습니다.
        * 스마트 포인터가 생성되고 소멸되는 시기를 조절할 수 있고, 생성될 때 기본값으로 0(널)을 가지기 때문에 쓰레기 값으로 인한 문제를 해결할 수 있으며, 어떤 객체를 가리키고 있는 최후의 스마트 포인터가 소멸될 때 자동으로 그 객체를 삭제하는 기능도 가지고 있습니다.
    * 복사(copy)와 대입(assignment) 동작을 조절할 수 있습니다.
        * 스마트 포인터가 복사되거나 대입될 때 일어나는 일을 여러분이 결정할 수 있습니다. 어떤 경우에는 깊은 복사(대입)를 할 수도 있고, 어떤 경우에는 얕은 복사(대입)를 할 수도 있으며, 어떤 경우에는 복사(대입)을 금지할 수도 있습니다.
    * 역참조(dereferencing) 동작을 조절할 수 있습니다.
        * 사용자가 스마트 포인터가 가리키는 객체를 가져오려고 할때의 동작도 여러분이 결정할 수 있습니다. 항목 17에서 이야기한 지연방식의 데이터/명령어 가져오기(fetching)를 구현하는데 스마트 포인터를 사용할 수 있습니다.
* 스마트 포인터는 엄격한 타입 제약을 위해 템플릿으로 만들어집니다. 다음 예제에서 스마트 포인터가 생긴 모습을 대략적으로 볼 수 있습니다
```c++
template <class T>                    // 스마트 포인터 객체의
class SmartPtr {                    // 템플릿
public:
    SmartPtr(T* realPtr = 0);        // 스마트 포인터를 생성하는데,
                                    // 벙어리 포인터가 가리키는 객체를
                                    // 가리키도록 초기화됩니다.
                                    // 값이 주어지지 않으면 0(널)입니다.
 
    SmartPtr(const SmartPtr& rhs);    // 스마트 포인터를 복사합니다.
 
    ~SmartPtr(void);                // 스마트 포인터를 소멸시킵니다.
    
    // 스마트 포인터에 대입 연산을 수행합니다.
    SmartPtr& operator=(const SmartPtr& rhs);
    
    T* operator->(void) const;        // 스마트 포인터를 역참조하여
                                    // 스마트 포인터가 가리키는
                                    // 멤버에 접근합니다.
    
    T& operator*(void) const;        // 스마트 포인터를 역참조합니다.
    
private:
    T *pointee;                        // 스마트 포인터가 가리키는
};                                    // 실제 객체의 주소
 
// 스마트 포인터의 복사를 허용하지 않으려면,
// 복사 생성자와 대입 연산자를 모두 private로
// 선언해야 할 것입니다.
 
// 역참조 연산자가 모두 const로 선언되어
// 있는 것은 포인터를 역참조 한다고 해서
// 스마트 포인터의 내부 데이터가 바뀌진
// 않기 때문입니다.
```
* 스마트 포인터의 사용 예를 하나 봅시다. 객체들이 여러 컴퓨터에 흩어져 있는 분산 시스템이 있다고 가정합시다. 어플리케이션 코드를 작성하는 사용자의 입장에서는 로컬 객체와 원격 객체의 처리방법이 달라야 한다는 점이 마음에 들 리가 없습니다. 실제로는 그렇지 않더라도 모든 객체는 같은 장소에 있는 것처럼 보이는 것이 편리합니다. 이렇게 하는데 스마트 포인터를 쓸 수 있습니다.

```c++
template <class T>                    // 분산 데이터베이스에
class DBPtr {                        // 저장되는 객체를 나타내는
public:                                // 스마트 포인터의 템플릿
 
    DBPtr(T *realPtr = 0);            // 벙어리 포인터가 가리키는
                                    // DB 객체를 가리키도록
                                    // 스마트 포인터를 생성합니다.
    
    DBPtr(DataBaseID id);            // 스마트 포인터를 생성하되,
                                    // DB 객체에 대한 유일 식별자
                                    // 를 생성자 인자로 넘깁니다.
    
    ...                                // 스마트 포인터의 다른 기능은
};                                    // 이전과 똑같습니다.
 
class Tuple {                        // 데이터베이스 튜플(Tuple)을
public:                                // 나타내는 클래스
    ...
    void displayEditDialog(void);    // 사용자가 튜플을 편집할 수
                                    // 있게 하는 그래픽 대화상자를
                                    // 띄웁니다.
    
    bool isValid(void) const;        // *this가 유효성 점검을
};                                    // 통과했는지 못 했는지를 반환합니다.
 
// T 객체의 내용이 바뀔 때마다 로깅되는 내용인 로그 엔트리의
// 클래스 템플릿
template <class T>
class LogEntry {
public:
    LogEntry(const T& objectToBeModified);
    ~LogEntry(void);
};
 
void editTuple(DBPtr<Tuple>& pt)
{
    LogEntry<Tuple> entry(*pt);        // 튜플 편집을 가리키는 로그
                                    // 엔트리를 생성합니다. 자세한
                                    // 내용은 다음에서 확인하세요.
    // 유효한 값이 주어질 때까지 편집 대화상자를
    // 반복적으로 표시합니다.
    do {
        pt->displayEditDialog();
    } while(pt->isValid() == false);
}
 
// editTuple 함수 안에서 편집되는 튜플은
// 물리적으로는 원격 컴퓨터에 들어 있을 수
// 있습니다. 하지만, 시스템의 내부 특징은
// 스마트 포인터 클래스가 모두 감추어 주므로
// editTuple을 작성하는 프로그래머는 그런
// 문제까지 신경쓸 필요가 없습니다.
 
// editTuple 함수에서 LogEntry가 사용되는
// 부분을 잘 보면, displayEditDialog를
// 호출하는 부분의 앞에서 로그 엔트리를 시작하고,
// displayEditDialog의 호출이 끝날 때
// 로그 엔트리를 끝내는 것이 편리할 것 같은데,
// 바로 그렇게 만들어진 코드입니다.
 
// LogEntry의 생성자가 로그 엔트리를 시작하고
// 소멸자가 로그 엔트리를 끝내고 있죠.
```

### 스마트 포인터의 생성, 대입, 소멸
* 스마트 포인터를 생성하는 방법은 사실 아무 것도 아닙니다. 가리킬 객체를 하나 준비하고, 스마트 포인터의 내부에 있는 벙어리 포인터를 그 객체의 주소로 세팅하면 됩니다. 가리킬 객체가 없으면 내부 포인터를 0(널)으로 놓든가 에러를 알립니다(예외를 일으킨다든지...).

* 스마트 포인터의 복사 생성자, 대입 연산자(여러 버전), 그리고 소멸자를 구현하는 방법은 소유권(ownership)이라는 것 때문에 꽤 복잡합니다. 스마트 포인터가 자신이 가리키는 객체(동적으로 할당되었다고 가정)를 가지고 있는 경우에는 스마트 포인터가 소멸될 때 이 객체도 삭제해야 합니다. 표준 C++라이브러리에 들어 있는 auto_ptr 템플릿을 예로 들어 봅시다.
```c++
//-----------------------------------------------------------------------------
// auto_ptr 객체는 소멸될 때 소멸자가 호출되면서
// 자신이 가리키는 객체를 삭제합니다.
 
// 이 auto_ptr 템플릿의 내부는 다음과 같이
// 생겼을 것입니다.
template <class T>
class auto_ptr {
public:
    auto_ptr(T *ptr = 0): pointee(ptr) {}
    ~auto_ptr(void) { delete pointee; }
    ...
private:
    T *pointee;
};
 
 
//-----------------------------------------------------------------------------
// 사실 한 개의 auto_ptr이 한 개의 객체를
// 맡고 있으면 이 동작은 별 문제가 없습니다.
 
// 그러나 auto_ptr이 복사되거나 대입될 때는
// 어떻게 되어야 할까요?
 
auto_ptr<TreeNode> ptn1(new TreeNode);
 
auto_ptr<TreeNode> ptn2 = ptn1;        // 복사 생성자를 호출합니다.
                                    // 어떻게 되어야 할까요?
auto_ptr<TreeNode> ptn3;
ptn3 = ptn2;                        // operator=를 호출합니다.
                                    // 어떻게 되어야 할까요?
 
// 내부의 벙어리 포인터를 단순히 복사해 버리면,
// 결국 두 개의 auto_ptr이 같은 객체를
// 가리키게 됩니다.
 
// 이렇게 되면 첫 번째 auto_ptr이
// 소멸되면서 객체를 삭제하고, 두 번째
// auto_ptr이 소멸되면서 같은 객체를
// 한 번 더 삭제하여 예측할 수 없는
// 결과를 낳습니다.
 
// 이를 해결하기 위해 깊은 복사를 하는 방법이
// 있지만, 이 방법은 객체를 생성하고 소멸하는
// 데에 있어 납득하기 힘든 수행 성능의 저하를
// 불러올 수 있습니다.
 
// 게다가 스마트 포인터를 생성할 때 객체의
// 타입을 꼭 알아야할 필요는 없습니다.
 
// auto_ptr<T>는 T 타입의 객체를 가리키지
// 않아도 되기 때문입니다. 실상은 T에서 파생된
// 객체를 가리켜도 의미구조에 문제가 없다는
// 것입니다.
 
// 다른 쓸만한 방법으로는 auto_ptr에서는
// 복사나 대입을 못하게 하여 문제를 해결하는
// 방법이 있습니다.
 
 
//-----------------------------------------------------------------------------
// 하지만 auto_ptr에서는 더 괜찮은
// 해결책이 사용되었습니다. 바로 auto_ptr이
// 복사 혹은 대입될 때 소유 관계를 옮기는 것입니다.
template <class T>
class auto_ptr {
public:
    ...
    auto_ptr(auto_ptr<T>& rhs);            // 복사 생성자
    
    auto_ptr<T>&                        // auto_ptr용
        operator=(auto_ptr<T> &rhs);    // 대입 연산자
    ...
};
 
template <class T>
auto_ptr<T>::auto_ptr(auto_ptr<T>& rhs)
{
    pointee = rhs.pointee;                // *pointee의 소유권을
                                        // rhs에서 *this로 옮깁니다.
 
    rhs.pointee = 0;                    // rhs는 이제 그 객체를
}                                        // 갖지 않습니다.
 
template <class T>
auto_ptr<T>& auto_ptr<T>::operator=(auto_ptr<T>& rhs)
{
    if (this == &rhs)                    // 자기 자신에게 대입되는
        return *this;                    // 경우에는 아무 일도
                                        // 하지 않습니다.
    
    delete pointee;                        // 현재 자신이 가지고 있는
                                        // 객체를 삭제합니다.
    
    pointee = rhs.pointee;                // *pointee의 소유권을
    rhs.pointee = 0;                    // rhs에서 *this로 옮깁니다.
    
    return *this;
}
 
// 복사 생성자와 대입 연산자의 인자로
// 상수 객체를 받지 않는 이유는
// auto_ptr의 복사 생성자나
// 대입 연산자의 우변이 될 때는
// 내부의 데이터가 바뀌기 때문입니다
// (널값으로 변경됨).
 
 
//-----------------------------------------------------------------------------
// 복사 생성자도 객체 소유권을 옮기는 방법으로
// 구현했기 때문에, auto_ptr을 함수의
// 매개변수로서 값으로 넘기면 진짜로 안 됩니다.
 
// 재앙을 부르는 함수
void printTreeNode(ostream& s, auto_ptr<TreeNode> p)
{ s << *p; }
 
int main(void)
{
    auto_ptr<TreeNode> ptn(new TreeNode);
    ...
    printTreeNode(cout, ptn);            // auto_ptr를 값으로 전달합니다.
    
    ...                                    // 이 부분에서 ptn이 가리키던
                                        // 객체는 이미 소멸되었습니다.
}
 
// auto_ptr을 함수의 매개변수로 넘겨야 한다면,
// 값으로 넘기지 말고 상수 객체 참조자에 의한 전달
// (Pass-by-reference-to-const)
// 방식으로 넘겨야 합니다.
 
// 정상적으로 동작하는 함수
void printTreeNode(ostream& s,
                const auto_ptr<TreeNode>& p)
{ s << *p; }
```

* auto_ptr에 대해 흥미가 있어서 완전한 구현 코드를 보고 싶으신 분들도 있을텐데, 이런 분들을 위해 "auto_ptr의 상세 구현 코드"를 따로 준비해 놓았습니다. 표준 C++ 라이브러리에 들어 있는 auto_ptr 템플릿이 여기서 설명한 것보다, 훨씬 유연하게 만들어진 복사 생성자와 대입 연산자를 가지고 있다는 사실을 확인하실 수 있을 것입니다. 사실 이 함수들은 그냥 멤버 함수가 아니라 멤버 함수 템플릿으로 만들어져 있습니다.

```c++
//-----------------------------------------------------------------------------
// 여기에 실은 auto_ptr은 공식 발표 버전의
// 내용 중 몇 가지를 가지고 있지 않습니다.
 
// 예를 들면, auto_ptr은 std 네임스페이스에
// 들어 있다는 사실이나 auto_ptr의 멤버 함수는
// 예외를 발생시키지 않게 되어 있다는 사실 등 입니다.
 
 
//-----------------------------------------------------------------------------
// 첫째 것은 클래스 인터페이스에 주석문을 붙이고
// 멤버 함수를 클래스 선언문의 바깥쪽에서 모두
// 구현한 것입니다.
 
template <class T>
class auto_ptr {
public:
    explicit auto_ptr(T *p = 0);        // "explicit"에 대한 설명은
                                        // 항목 5에서 찾을 수 있습니다.
    
    template <class U>                    // 복사 생성자용 멤버 템플릿
    auto_ptr(auto_ptr<U>& rhs);            // (항목 28을 참고하세요).
                                        // 새로 생성되는 auto_ptr을 다른
                                        // auto_ptr(물론 U가 같아야 함)을
                                        // 써서 초기화합니다.
    
    ~auto_ptr(void);
    
    template <class U>                    // 대입 연산자용 멤버 템플릿
    auto_ptr<T>&                        // (항목 28을 보세요).
        operator=(auto_ptr<U>& rhs);    // 다른 auto_ptr을 이 auto_ptr에
                                        // 대입합니다.
    
    T& operator*(void) const;            // 항목 28을 참고하세요.
    T* operator->(void) const;            // 항목 28을 참고하세요.
    
    T* get(void) const;                    // 현재의 벙어리 포인터를
                                        // 반환합니다.
    
    T* release(void);                    // 현재의 벙어리 포인터에 대한
                                        // 소유권을 박탈하고 그 포인터값을
                                        // 반환합니다.
    
    void reset(T *p = 0);                // 가지고 있는 포인터를 삭제합니다
                                        // (p의 소유권을 가지고 있다는 가정 하에).
private:
    T *pointee;
    
    template <class U>                    // auto_ptr 클래스끼리 서로의
    friend class auto_ptr<U>;            // 프렌드가 되도록 합니다.
};
 
template <class T>
inline auto_ptr<T>::auto_ptr(T *p)
: pointee(p);
{}
 
template <class T>
template <class U>
inline auto_ptr<T>::auto_ptr(auto_ptr<U>& rhs)
: pointee(rhs.release())
{}
 
template <class T>
inline auto_ptr<T>::~auto_ptr(void)
{ delete pointee; }
 
template <class T>
template <class U>
inline auto_ptr<T>& auto_ptr<T>::operator=(auto_ptr<U>& rhs)
{
     if (this != &rhs) reset(rhs.release());
     return *this;
}
 
template <class T>
inline T& auto_ptr<T>::operator*(void) const
{ return *pointee; }
 
template <class T>
inline T* auto_ptr<T>::operator->(void) const
{ return pointee; }
 
template <class T>
inline T* auto_ptr<T>::get(void) const
{ return pointee; }
 
template <class T>
inline T* auto_ptr<T>::release(void)
{
    T *oldPointee = pointee;
    pointee = 0;
    return oldPointee;
}
 
template <class T>
inline void auto_ptr<T>::reset(T *p)
{
    if (pointee != p) {
        delete pointee;
        pointee = p;
    }
}
 
 
//-----------------------------------------------------------------------------
// 둘째 것은 클래스 선언과 멤버 함수 구현을
// 동시에 한 것입니다. 원칙적인 스타일로 보면
// 둘째 것이 뒤쳐집니다.
 
// 하지만 auto_ptr은 간단한 클래스이기 때문에,
// 전체적으로 보면 첫째 것보다 둘째 것이 깔끔합니다.
template <class T>
class auto_ptr {
public:
    explicit auto_ptr(T *p = 0): pointee(p) {}
    
    template <class U>
    auto_ptr(auto_ptr<U>& rhs): pointee(rhs.release()) {}
    
    ~auto_ptr(void) { delete pointee; }
    
    template <class U>
    auto_ptr<T>& operator=(auto_ptr<U>& rhs)
    {
        if (this != &rhs) reset(rhs.release());
        return *this;
    }
    
    T& operator*(void) const { return *pointee; }
    
    T* operator->(void) const { return pointee; }
    
    T* get(void) const { return pointee; }
    
    T* release(void)
    {
        T *oldPointee = pointee;
        pointee = 0;
        return oldPointee;
    }
    
    void reset(T *p = 0)
    {
        if (pointee != p) {
            delete pointee;
            pointee = p;
        }
    }
    
private:
    T *pointee;
    
    template <class U> friend class auto_ptr<U>;
};

```

### 역참조(Dereferencing) 연산자 구현하기
* 역참조 연산자란 operator*와 operator-> 함수를 말합니다. 우선 operator*부터 알아보죠. 이 함수는 스마트 포인터가 가리키는 객체를 반환하는데, 개념적으로는 지극히 단순합니다.
```c++
template <class T>
T& SmartPtr<T>::operator*(void) const
{
    "스마트 포인터" 처리를 수행합니다;
    
    return *pointee;
}
 
// 먼저 이 함수는 pointee를 초기화하든지
// pointee를 유효하게 하는 일을 닥치는 대로
// 합니다.
 
// 이를테면, 지연 방식의 가져오기(항목 17 참조)가
// 사용되는 상태에서는 pointee가 가리키는
// 객체를 불러와야 하겠습니다.
 
// 어쨌든 일단 pointee가 유효하다고 판정되면,
// operator* 함수는 스마트 포인터가
// 가리키는 객체의 참조자를 반환합니다.
 
// 여기서 반환 타입이 참조자라는 점이 중요합니다.
 
// 절대로 operator*의 반환 타입을
// 참조자가 아닌 값으로 해서는 안됩니다.
 
// 왜냐하면 스마트 포인터가 가리키는 객체는
// T가 아니라 T에서 파생된 객체일 수도
// 있기 때문이죠. 이런 경우 바로 슬라이스
// 문제가 발생합니다(항목 13 참조).
 
// 거기에다 추가로 값을 반환하는것 보다
// 참조자를 반환하는 것이 훨씬 효율적입니다.
// 임시 객체를 만들지 않기 때문입니다
// (항목 19 참조).
 
// 만일 널(null)값을 가진 스마트 포인터에
// 대해 operator*를 호출하는 상황이라면
// 주저하지 말고 원하는 대로 오류 처리를 하면 됩니다.
// 예외를 던질 수도 있고, abort를 호출할 수도
// 있습니다.
```

* operator->의 동작도 operator*의 동작과 별반 다르지 않습니다(참조자 대신 포인터를 반환하는것 외엔 똑같습니다).
```c++
// operator->에 대한 이야기를 하기 전에
// 이 함수의 호출에 관련된 이상한 의미를 함께
// 고민해 볼 필요가 있습니다.
 
// 앞의 예제에서 등장했던 editTuple 함수를
// 다시 가지고 왔습니다.
 
void editTuple(DBPtr<Tuple>& pt)
{
    LogEntry<Tuple> entry(*pt);
    do {
        pt->displayEditDialog();
    } while (pt->isValid() == false);
}
 
// 여기서 이 문장은,
 
pt->displayEditDialog();
 
//컴파일러에 의해 다음과 같이 해석됩니다.
 
(pt.operator->())->displayEditDialog();
 
// 앞 문장은 operator->가 반환하는 것이
// 무엇이든 간에, 멤버 지정 연산자(->)를 그것에
// 대해 적용해도 상관 없어야 한다는 뜻을 담고
// 있습니다.
 
// operator->가 반환할 수 있는 것은 오로지
// 벙어리 포인터 아니면, 스마트 포인터 객체
// 중 하나입니다.
 
// 보통의 경우 벙어리 포인터를 반환하는 경우가
// 대부분일 것입니다.
 
template <class T>
T* SmartPtr<T>::operator->(void) const
{
    "스마트 포인터" 처리를 수행합니다;
    return pointee;
}
```

### 스마트 포인터가 널(null)인지 점검하기
* 지금까지 알아본 것만 잘 잊지 않으면, 스마트 포인터의 기본적인 사용은 가능합니다. 그러나 아직 가능하지 않은 것 중 하나가 '스마트 포인터가 널인지 확인하는 방법'입니다.
```c++
SmartPtr<TreeNode> ptn;
...
// 다음의 세 if 문은 모두 오류입니다.
if (ptn == 0) ...
if (ptn) ...
if (!ptn) ...
```

* 스마트 포인터를 가지고 앞에서와 같은 코딩을 할 수 없다는 것은 심각한 한계입니다. 얕은 생각으로는 isNull 같은 멤버 함수를 추가하면 간단히 해결될 것 같지만, 그렇게 했다간 스마트 포인터가 벙어리 포인터와 똑같이 동작할 수 없다는 문제가 그대로 남게 되므로 별로 좋지 않습니다.

* 적합한 방법은 앞의 코드가 그대로 컴파일되도록 암시적 타입변환 연산자를 넣어주는 것입니다. 이런 목적으로 타입변환을 할 때에는 대개 void*로 바꿉니다.

```c++
//-----------------------------------------------------------------------------
template <class T>
class SmartPtr {
public:
    ...
    operator void*(void);    // 스마트 포인터가 널이면
    ...                        // 0을 반환하고, 아니면 0이
};                            // 아닌 값을 반환합니다.
 
SmartPtr<TreeNode> ptn;
...
// 이제는 다음 세 if문이 모두 컴파일됩니다.
if (ptn == 0) ...
if (ptn) ...
if (!ptn) ...
 
// 다른 타입 변환 함수들이 그러하듯이
// 이 방법도 단점을 안고 있습니다.
 
// 바로 대부분의 프로그래머가 실패할 것
// 같다고 생각하는 함수 호출을 실패시키지
// 않는다는 것입니다(항목 5 참조).
 
SmartPtr<Apple> pa;
SmartPtr<Orange> po;
...
if (pa == po) ...            // 이게 컴파일된다니까요!
 
 
//-----------------------------------------------------------------------------
// 'void*로 변환하기' 방법을 응용한 것들도 꽤 있습니다.
 
// 어떤 쪽에서는 const void*로 바꾸는 것이 옹호되는 반면,
// 또 다른 쪽에서는 bool로 바꿔야 한다고 주장합니다.
 
// 하지만 어떤 방법을 쓰더라도 다른 타입끼리의 비교가
// 허용되는 문제는 없앨 수 없습니다.
 
// 완전하지는 않지만, 다른 타입의 스마트 포인터를
// 비교하는 경우를 최소화하면서 스마트 포인터가
// 널인지 확인하는 괜찮은 방법이 있습니다.
 
// operator! 연산자를 오버로딩해서,
// 이 연산자가 호출된 스마트 포인터가 널일 때에만
// true를 반환하도록 하는 것입니다.
 
template <class T>
class SmartPtr {
public:
    ...
    bool operator!(void) const;
    ...
};
 
SmartPtr<TreeNode> ptn;
...
if (!ptn) {                    // 문제없습니다.
    ...                        // ptn이 널입니다.
}
else {
    ...                        // ptn이 널이 아닙니다.
}
 
// 다음의 코드는 컴파일되지 않습니다.
if (ptn == 0) ...            // 여전히 에러입니다.
if (ptn) ...                // 아.. 에러라니까요.
 
// 전혀 다른 타입의 스마트 포인터를 비교할 때에는
// 오로지 다음의 경우만 조심하면 됩니다.
SmartPtr<Apple> pa;
SmartPtr<Orange> po;
...
if (!pa == !po) ...            // 안타깝지만, 컴파일됩니다.
 
// 단 이렇게 코딩하는 경우는 상당히 드물기 때문에,
// 크게 문제가 되지는 않습니다.
```

### 스마트 포인터를 벙어리 포인터로 변환하기
* 스마트 포인터를 사용하더라도 여전히 벙어리 포인터가 필요한 경우가 있습니다. 대표적인 예로 스마트 포인터를 사용하지 않고 설계된 라이브러리를 사용해야 할 때입니다. 이런 경우에도 스마트 포인터를 사용하기 위해서, 스마트 포인터가 가리키고 있는 객체의 벙어리 포인터를 얻는 방법이 필요합니다.

* 이것을 해결하는 끔찍한 방법 중의 하나는, 스마트 포인터 객체를 벙어리 포인터 타입으로 변환하는 암시적 타입 변환 연산자를 추가하는 것입니다. 암시적 변환을 허용하게 되면, 스마트 포인터가 마치 벙어리 포인터인 것 마냥 쓸 수 있게 되고, 스마트 포인터의 널 점검에 대한 고민도 날아갑니다. 하지만 이 방법은 중대한 단점을 여럿 가지고 있습니다.

```c++
//-----------------------------------------------------------------------------
// 스마트 포인터를 벙어리 포인터로
// 암시적 타입 변환 연산자를
// 추가한 경우에 스마트 포인터가
// 주려고 했던 혜택을
// 받지 못할 수 있습니다.
 
void processTuple(DBPtr<Tuple>& pt)
{
    Tuple *rawTuplePtr = pt;    // DBPtr<Tuple> 타입을
                                // Tuple*로 변환합니다.
    rawTuplePtr을 써서 튜플 데이터를 바꿉니다;
}
 
// 만일 DBPtr에 참조 카운팅 기능이
// 들어 있다면 rawTuplePtr이
// 참조 카운팅에 필요한 내부 정보를
// 조정하지 않아 문제가 발생할 수 있습니다.
 
 
//-----------------------------------------------------------------------------
// 위와 같은 경우까지 고려해서 암시적
// 타입 변환 연산자를 만들었다고 해도,
// 스마트 포인터는 벙어리 포인터가 있던
// 자리를 전부 대신할 수는 없는 존재입니다.
 
// 예를 봅시다.
 
class TupleAccessors {
public:
    TupleAccessors(const Tuple *pt);    // pt는 사용자들이
    ...                                    // 액세스하고 있는
};                                        // 튜플을 가리킵니다.
 
TupleAccessors merge(const TupleAccessors& ta1,
                    const TupleAccessors& ta2);
 
Tuple *pt1, *pt2;
...
merge(pt1, pt2);        // 문제 없습니다. 두 포인터가
                        // TupleAccessors로 바뀌니까요.
 
DBPtr<Tuple> pt1, pt2;
...
merge(pt1, pt2);        // 에러입니다! pt1과 pt2를
                        // TupleAccessors로 바꿀 방법이 없습니다.
 
// 왜 그럴까요?
 
// DBPtr<Tuple>을 TupleAccessors으로 바꾸려면
// DBPtr<Tuple>을 Tuple*로 바꾸는 함수와
// Tuple*을 TupleAccessors로 바꾸는 함수
// 두 개가 필요한데, 이런 일(두 개 이상의
// 사용자 정의 타입변환 함수를 암시적으로
// 호출하는 일)은 C++에서 금지되어 있기 때문입니다.
 
 
//-----------------------------------------------------------------------------
// 암시적 타입 변환 연산자를 사용하는 스마트 포인터는
// 어떤 지저분한 버그에 대해서도 아무 생각 없이
// 문을 열어 놓습니다.
 
// 다음의 예를 보세요.
 
DBPtr<Tuple> pt = new Tuple;
...
delete pt;
 
// 이 코드는 컴파일되어선 안됩니다.
// 그도 그럴게 pt는 포인터가 아니라 객체니까요.
 
// 하지만 DBPtr에 암시적 타입변환 연산자가
// 있다면, 컴파일러는 스마트 포인터 pt를
// pt가 가리키고 있는 객체의 벙어리 포인터
// (Tuple* 타입)로 변환하여 기어코
// 삭제해버리고 맙니다.
 
// 당연히 pt의 소멸자가 한 번 더 삭제를
// 시도할테니 프로그램은 온전히 작동되지
// 않을 것입니다.
```

* 이것만은 꼭 지킵시다. 꼭 그렇게 해야 하는 이유가 없으면, 스마트 포인터를 벙어리 포인터로 바꾸는 암시적 타입변환 연산자는 제공하지 말자는 원칙 말입니다.

### 스마트 포인터와 상속 기반의 타입변환
* 클래스 A와 A를 상속받아 구현한 클래스 B가 있다고 합시다. 벙어리 포인터의 경우 A 타입의 포인터가 B 타입의 객체를 가리킬 수 있는 것은 너무나 당연한 일입니다. 하지만 스마트 포인터의 경우는 좀 다릅니다. 상속 관게에 있는 것은 스마트 포인터가 가리키고 있는 객체의 클래스이지 스마트 포인터는 아니기 때문에, A 타입의 포인터를 가리키는 스마트 포인터가 B 타입의 객체를 가리킬 수 없습니다.

* 이 문제를 해결하기 위해서 (비가상) 멤버 함수 템플릿(보통, 멤버 템플릿이라고 부릅니다)을 선언해 주는 방법을 사용할 수 있습니다. 다만 이 방법의 동작 원리를 제대로 이해하기 힘들다는 단점이 있습니다.

* 명심해야 할 점은 스마트 포인터는 '스마트'하지만 진짜 벙어리 포인터가 될 수는 없다는 것입니다.

```c++
//-----------------------------------------------------------------------------
template <class T>                        // T 객체에 대한 스마트 포인터
class SmartPtr {                        // 템플릿 클래스
public:
    SmartPtr(T* realPtr = 0);
    
    T* operator->(void) const;
    T& operator*(void) const;
    
    template <class newType>            // 암시적 변환 연산자를 위한
    operator SmartPtr<newType>(void)    // 템플릿 함수
    {
            return SmartPtr<newType>(pointee);
    }
    ...
};
 
// 이 객체를 T의 기본 클래스에 대한 포인터로
// 변환해야 하는 상황이라고 가정합시다.
 
// 우선 컴파일러는 SmartPtr<T>의 클래스
// 정의를 살펴서 필요한 변환 연산자가
// 선언되어 있는지를 확인하는데, 보시다시피
// 선언이 안 되어 있지요.
 
// 이번엔 컴파일러는 그 시점에 필요한 타입변환을
// 해 주는 멤버 함수 템플릿이 들어 있는지
// 점검합니다.
 
// 이윽고 요구에 맞는 멤버 함수 템플릿
// (형식 타입 매개변수 newType을 취하는)
// 을 찾게 되고, 변환하고자 하는 타입 T의
// 기본 클래스에 바인딩된 newType 매개변수를
// 써서 함수 템플릿을 함수로 인스턴스화 합니다.
 
// 생성자에서 벙어리 포인터 타입 T*를
// 매개변수로 받기 때문에 newType이
// T의 기본 클래스 타입이라면 오류 없이
// 컴파일 될 것입니다.
 
// 다음 예시를 보세요.
 
class MusicProduct { ... };
class Cassette : public MusicProduct { ... };
 
void displayAndPlay(const SmartPtr<MusicProduct>& pmp,
                    int howMany);
 
SmartPtr<Cassette> funMusic(new Cassette("Alapalooza"));
 
displayAndPlay(funMusic, 10);
 
// 위 상황에서 컴파일러는 멤버 함수 템플릿을
// 이용하여 다음의 함수를 찍어냅니다.
 
SmartPtr<Cassette>::operator SmartPtr<MusicProduct>(void)
{
    return SmartPtr<MusicProduct>(pointee);
}
 
 
//-----------------------------------------------------------------------------
// 하지만 다음과 같은 경우에는 스마트 포인터가
// 벙어리 포인터의 흉내를 낼 수 없습니다.
 
template <class T>                // 이전과 같습니다. 타입변환 연산자를
class SmartPtr { ... };            // 위한 멤버 함수 템플릿이 들어 있습니다.
 
class MusicProduct { ... };
class Cassette : public MusicProduct { ... };
class CasSingle : public Cassette { ... };
 
void displayAndPlay(const SmartPtr<MusicProduct>& pmp,
                    int howMany);
 
void displayAndPlay(const SmartPtr<Cassette>& pc,
                    int howMany);
 
SmartPtr<CasSingle> dumbMusic(new CasSingle
                                ("Achy Breaky Heart"));
 
displayAndPlay(dumbMusic, 1);    // 에러입니다!
 
// 우리들 예상에는 오버로딩된 함수 중에
// SmartPtr<Cassette>을 받는 버전이
// 선택될 것 같습니다.
 
// CasSingle은 Cassette로부터 직접 파생되었고,
// MusicProduct로부터는 간접적으로 파생되었기
// 때문입니다.
 
// 벙어리 포인터의 경우엔 이 예상이 맞습니다.
 
// 하지만 스마트 포인터에서는 멤버 함수를 써서
// 타입변환을 수행하기 때문에, C++ 컴파일러의
// 눈에는 모든 변환 함수가 똑같이 보입니다.
 
// 결국 적합한 displayAndPlay를 선택하지 못하고
// "모호(ambiguous)하다"라는 에러를 냅니다.

```

### 스마트 포인터와 const
* 벙어리 포인터를 선언할 때 붙일 수 있는 const 키워드로는 포인터가 가리키는 것과 포인터 그 자체를 상수(constant)로 만들 수 있습니다. 이 개념을 그대로 스마트 포인터로 가져오면, 다음과 같이 네 가지 경우를 표현할 수 있습니다.

```c++
// 비상수 객체,
// 비상수 포인터
SmartPtr<CD> p;

// 상수 객체,
// 비상수 포인터
SmartPtr<const CD> p;

// 비상수 객체,
// 상수 포인터
const SmartPtr<CD> p;

// 상수 객체,
// 상수 포인터
const SmartPtr<const CD> p;
```

* 물론 SmartPtr<CD>에서 SmartPtr<const CD>로 변환이 가능하도록 멤버 함수 템플릿을 이용한 암시적 변환 함수를 만들어 줘야 합니다.

* 하지만 멤버 함수 템플릿을 지원하지 않는 구형 컴파일러를 사용하고 있다면 다른 방법을 사용해야 합니다.

* 상수 객체가 할 수 있는 모든 행위를 비상수 객체도 할 수 있다는 점은 public 상속 관계와 매우 유사합니다. 따라서 상수 객체를 가리키는 스마트 포인터를 public 상속받아, 비상수 객체를 가리키는 스마트 포인터를 구현함으로써 이 문제를 해결할 수 있습니다.


```c++
template <class T>                        // 상수 객체에 대한
class SmartPtrToConst {                    // 스마트 포인터 클래스
    ...                                    // 스마트 포인터 클래스가 가진
                                        // 공통의 멤버 함수는 모두 포함
protected:
    union {
        const T* constPointee;            // SmartPtrToConst에서 쓰기 위한 것
        T* pointee;                        // SmartPtr에서 쓰기 위한 것
    };
};
 
template <class T>                        // 비상수 객체에 대한
class SmartPtr:                            // 스마트 포인터 클래스
    public SmartPtrToConst<T> {
        ...                                // 데이터 멤버는 없습니다.
};
 
SmartPtr<CD> pCD = new CD("Famous Movie Themes");
SmartPtrToConst<CD> pConstCD = pCD;        // 문제 없습니다.
 
// 단 공용체를 써서 구현한 경우
// 각 클래스의 멤버 함수에 적절한
// 타입의 포인터를 쓰는 것은
// 개발자의 의무입니다.
 
// 이 부분만큼은 컴파일러의 도움을 받아
// 문법적 제약을 가한다든지 할 수가
// 없습니다.
 
// 이것이 공용체의 위험한 점이죠.
```


## 항목 29 : 참조 카운팅(Reference Counting)
* 참조 카운팅(reference counting) 이란 
    * 여러 개의 객체들이 똑같은 값을 가졌으면, 그 객체로 하여금 그 값을 나타내는 하나의 데이터를 공유하게 해서 데이터 양을 절약하는 기법
        * 1. 참조 카운팅이 만들어진 첫 번째 이유는, 힙 객체를 둘러싼 내부 정보를 유지하는 작업을 단순하게 하자는 것입니다
            * 참조 카운팅은 객체 소유권을 추적해야 하는 고민을 없앱니다.
        * 2. 두 번째 이유는 지극히 상식적인 것입니다. 똑같은 값을 갖고 있는 객체들이 각각 메모리를 할당하여 값의 복사본을 가지고 있는 것은 공간적, 시간적 낭비임에 틀림없습니다.
            * 참조 카운팅을 사용하면, 공통된 값을 나타내는 데이터 하나를 공유하게 하면 되므로 여러모로 이득입니다.
* 가령 string a,b,d,e에 hello 라는 문자열을 저장한다고 생각해보면
    * 아래처럼 하면 낭비일 것입니다. 
```
a -> heollo
b -> heollo
c -> heollo
d -> heollo
e -> heollo
```
* a,b,c,d,e가 전부 같은 데이터를 가리키면 될것인데
    * 실제로는 이렇게 할 수 없습니다.
        * 값을 공유하고 있는 개체가 몇개인지에 대한 정보도 필요합니다.  
```
a -> 
b -> 
c -> heollo
d -> 
e -> 
```
* 가령 아래 처럼 객체에 대한 참조 횟수 즉 참조 카운트가 고려된 형태처럼 되어야 겠지요.
```
a -> 
b -> 
c -> 5 -> hello
d -> 
e -> 
```
### 참조 카운팅은 기본적으로 이렇게 구현합니다. 
* 참조 카운팅을 하기 위해서는, 공유할 값과 그 값을 갖는(가리키는) 객체가 몇 개인지를 담을 변수(참조 카운트)가, 각 값마다 하나씩 필요합니다. String 클래스를 참조 카운팅을 적용하여 구현한 형태를 예로 들어 보겠습니다.
```c++
// 참조 카운트는 굳이 String 객체 내에
// 있을 필요가 없습니다. 왜냐하면 실제
// 문자열값 하나에 대해서 참조 카운트는
// 하나만 있으면 되기 때문입니다.
 
// 따라서 참조 카운트와 값을 묶는 클래스를
// 만드는 편이 관리하기 좋을 것입니다.
 
// 이 클래스의 이름은 StringValue이고,
// 이 클래스의 존재 이유는 String 클래스의
// 구현을 보조하는 것이므로, String의
// private 영역에 이 클래스를
// 중첩(nesting)시키겠습니다.
 
// 덧붙여서 StringValue 데이터 구조는
// String의 모든 멤버 함수가 접근할
// 수 있도록 StringValue는 클래스(class)
// 가 아닌 구조체(struct)로 선언하겠습니다.
 
class String {
public:
    String(const char *initValue = "");
    String(const String& rhs);
    ~String(void);
    
    String& operator =(const String& rhs);
 
    ...                                    // String에 관련된 멤버 함수
                                        // 는 여기에 있습니다.
private:
    struct StringValue {                // 참조 카운트와 실제 문자열값을
        int refCount;                    // 가집니다.
        char *data;
        
        StringValue(const char *initValue);
        ~StringValue(void);
    };
 
    StringValue *value;                    // 이 String 객체의 값
};
 
String::StringValue::StringValue(const char *initValue)
: refCount(1)
{
    data = new char[strlen(initValue) + 1];
    strcpy(data, initValue);
}
 
String::StringValue::~StringValue(void)
{
    delete[] data;
}
 
String::String(const char *initValue)    // 이 생성자는 인자로 넘기는
: value(new StringValue(initValue))        // char* 포인터를 써서
{}                                        // StringValue 객체를 새로
                                        // 생성한 후에, 새로 만들어진
                                        // StringValue를 역시
                                        // 새로 만들어진 String 객체가
                                        // 가리키게 하는 것입니다.
 
String::String(const String& rhs)        // 복사 생성자는 꼭 필요한
: value(rhs.value)                        // 것일 뿐만 아니라, 효율을
{                                        // 위한 장치입니다.
    ++value->refCount;                    // 새로 생성된 String 객체는
}                                        // 복사되는 String 객체와
                                        // 똑같은 StringValue 객체를
                                        // 공유하게 됩니다.
 
String::~String(void)                    // 소멸자의 경우 복사 생성자와
                                        // 반대로 참조 카운트를 줄이고,
                                        // 참조 카운트가 1인 경우에는
{                                        // value를 삭제합니다.
    if (--value->refCount == 0) delete value;
}
 
String& String::operator =(const String& rhs)
{
    if (value == rhs.value) {            // 이미 같은 값이면 아무 것도
        return *this;                    // 하지 않습니다. 보통 이런 연산자
    }                                    // 에서 행하는 this와 &rhs의 대등
                                        // 비교 대신에 사용된 코드입니다.
    
    if (--value->refCount == 0) {        // 원래의 값을 사용하는 객체가
        delete value;                    // 없으면 *this의 값을 소멸시킵니다.
    }
    
    value = rhs.value;                    // *this로 하여금 rhs의 값을
    ++value->refCount;                    // 공유하도록 합니다.
    
    return *this;
}
```
### 기록시점 복사(Copy-on-Write)
* 참조 카운팅 기능을 갖는 문자열의 기본적인 뼈대를 마무리하는 마지막 부품은 배열-대괄호 연산자([])입니다. 이 연산자는 String 객체가 가리키는 문자열에 들어 있는 문자 하나씩을 읽고 쓰는데 사용됩니다.
    * 이 함수의 const 버전은 문자열값을 변경하지는 않으므로 단순히 넘겨받은 인덱스의 문자를 반환하기만 하면 됩니다. 하지만 const가 아닌 operator [] 버전은 문자열값을 변경할 수도, 하지 않을 수도 있기 때문에 신경써서 구현해야 합니다. 왜냐하면 두 String 객체가 같은 문자열값을 가리키고 있는 경우, 한 객체의 문자열값이 변경되면 다른 객체에도 영향을 주기 때문입니다. 따라서 문자열값을 변경할 가능성이 있는 멤버함수는, 자신이 갖는 문자열값을 가리키는 다른 객체가 존재할 경우에는 문자열을 복사한 후 작업해야 합니다.
```c++
class String {
public:
    const char& operator [](int index) const;    // 상수 String 객체에 대한 [] 연산자
    
    char& operator [](int index);                // 비상수 String 객체에 대한 [] 연산자
    ...
};
 
const char& String::operator [](int index) const
{
    return value->data[index];                    // 예제를 간결하게 하기 위해
}                                                // 인덱스 유효성 검사는 생략했습니다.
 
String s;
...
cout << s[3];        // 문자 읽기
s[5] = 'x';            // 문자 쓰기
 
// 위의 예제에서 문자를 읽고 쓰는 경우 모두
// 비상수 String 객체에 대한 [] 연산자가
// 사용되었습니다.
 
// C++ 컴파일러의 힘을 빌려 operator []가
// 쓰기 동작에 쓰였는지 읽기 동작에 쓰였는지를
// 구분할 방법은 없기 때문에, 비상수 String
// 객체에 대한 [] 연산자는 항상 쓰기를 한다고
// 가정을 할 수 밖에 없습니다.
 
// 단 읽기와 쓰기를 구분하는 데 항목 30에서
// 다룰 프로시(Proxy) 클래스의 도움을 받을 수
// 있긴 합니다.
 
char& String::operator [](int index)
{
    // 다른 String 객체와 문자열값을 공유하고 있는 상태에서는,
    // 이 String 객체만의 문자열 사본을 별도로 마련합니다.
    if (value->refCount > 1) {
        --value->refCount;                        // 현재 문자열값에 대한 참조 카운트를
                                                // 감소시킵니다. 왜냐하면 이제 그 값을
                                                // 사용하지 않겠다는 표시를 해야 하니까요.
        
        value =                                    // 이 String 객체만의
            new StringValue(value->data);        // 문자열 사본을 마련합니다.
    }
    // 공유 관계가 끊긴 StringValue 객체 안의
    // 문자에 대한 참조자를 반환합니다.
    return value->data[index];
}

```

### 포인터, 참조자, 그리고 기록시점 복사
* 기록시점 복사를 하는 경우 const가 아닌 operator []에 대해서 다음과 같은 문제가 발생할 수 있습니다.
```c++
String s1 = "Hello";
char *p = &s1[1];

String s2 = s1;
//이때 아래의 문장이 들어가면!
*p = 'x';//s1과 s2의 값을 모두 바꾸는 것입니다.
```
* 이 때, 기록시점 복사를 하기 때문에 s1과 s2는 하나의 문자열값을 공유하고 있어서, 결국 마지막 문장에 의해 s1과 s2모두 "Hxllo"로 바뀌게 됩니다. 이 문제는 포인터 뿐만이 아니라, 참조자를 사용한 경우에도 발생할 수 있습니다.
* 이 문제를 해결할 수 있는 방법은 최소한 세 가지 입니다.
    * 1. 이런 문제를 무시하고, 문제가 없는 것처럼 꾸미기
        * 참조 카운트 기능을 갖는 문자열을 구현한 클래스 라이브러리를 살펴보면, 이런 경우가 애처로울 정도로 흔합니다. 하지만 별로 바람직하지는 못한 방법이죠.
    * 2. 불법으로 규정하기
        * 라이브러리의 사용 문서(설명서)에 이러한 코드를 작성하면 안된다고 못을 박는 것입니다. 그리고 사용자가 고의든 아니든 이런 코드를 작성하여 발생한 불만에 대해서는 "아참, 설명서를 읽으셔야죠. 하지 말라고 되어 있는데"라고 응답하면 그만입니다.
    * 3. 이 문제를 적극적으로 직접 없애기
        * 구현하기는 어렵지 않지만 객체들 간의 값 공유 횟수가 줄어들 수 있습니다. 이 방법의 핵심은 각 StringValue 객체에 그 객체의 공유 가능 여부를 표시하는 플래그를 넣는 것입니다. 처음에는 이 플래그를 켜고('공유가능'이란 뜻) 나중에 그 객체가 나타내는 값에 대해 비상수 operator []가 호출될 때마다 이 플래그를 끕니다. 일단 이 플래그가 한 번 꺼지면 다시는 켜지지 않도록 합니다.

* std::string 타입의 경우 두 번째와 세 번째 방법을 섞어서 사용하고 있습니다. 비상수 operator []에서 반환된 참조자는 그 문자열을 수정하는 다른 함수가 호출되기 전까지는 유효합니다. 즉 다른 함수가 호출된 후에는 그 참조자(혹은, 그 참조자로 참조한 문자)를 사용했다간 오작동이 일어날 수 있습니다. 그렇기 때문에, 문자열을 수정하는 함수가 호출될 때마다 공유 가능성 플래그를 true로 되돌려 놓아도 되도록 만들어져 있습니다.
```c++
class String {
private:
    struct StringValue {
        int refCount;
        bool shareable;                // 이것이 추가된 것입니다.
        char *data;
        
        StringValue(const char *initValue);
        ~StringValue(void);
    };
    ...
};
 
String::StringValue::StringValue(const char *initValue)
:    refCount(1),
    shareable(true)                    // 플래그 초기화용으로 추가했습니다.
{
    data = new char[strlen(initValue) + 1];
    strcpy(data, initValue);
}
 
String::StringValue::~StringValue(void)
{
    delete[] data;
}
 
String::String(const String& rhs)
{
    if (rhs.value->shareable) {        // 플래그가 공유 가능함을
        value = rhs.value;            // 나타낼 때에만 공유합니다.
        ++value->refCount;
    }
    else {
        value = new StringValue(rhs.value->data);
    }
}
 
char& String::operator [](int index)
{
    if (value->refCount > 1) {
        --value->refCount;
        value = new StringValue(value->data);
    }
    
    value->shareable = false;        // 이것이 추가되었습니다.
    return value->data[index];
}
 
// 항목 30에 나온 프록시 클래스 기법을 써서
// operator []의 읽기/쓰기를 구별할 경우에는,
// 공유 가능성 플래그가 false로 표시되는
// StringValue 객체의 개수가 줄어드는 것이
// 보통입니다.
```

### 참조 카운팅 기능을 가진 기본 클래스
* 참조 카운팅(Reference counting) 기능은 비단 문자열에만 필요한 것은 아닙니다. 하지만 번듯하게 만들어져 있는 멀쩡한 클래스에 참조 카운팅 기능을 넣기에는 너무 많은 노력이 필요하게 됩니다. 이러한 노력을 피하기 위한 첫 단추는 참조 카운팅 기능을 가진 객체의 기반 코드를 제공하는 C++ 기본 클래스를 만드는 것 부터 시작합니다. 이름하여 RCObject, 자동 참조 카운팅 기능을 이용하고자 하는 클래스는 이 클래스를 상속하면 됩니다.

```c++
//-----------------------------------------------------------------------------
class RCObject {
public:
    RCObject(void);
    RCObject(const RCObject& rhs);
    RCObject& operator =(const RCObject& rhs);
    virtual ~RCObject(void) = 0;
    void addReference(void);
    void removeReference(void);
    void markUnshareable(void);
    
    bool isShareable(void) const;
    
    bool isShared(void) const;
 
private:
    int refCount;
    bool shareable;
};
 
RCObject::RCObject(void)
: refCount(0), shareable(true) {}
 
RCObject::RCObject(const RCObject&)
: refCount(0), shareable(true) {}
RCObject& RCObject::operator =(const RCObject&)
{ return *this; }
 
RCObject::~RCObject(void) {}    // 가상 소멸자는 반드시 항상
                                // 코드 몸체를 가져야 합니다.
                                // 가상 소멸자가 순수 가상 함수이고
                                // 아무 것도 하지 않더라도 구현해야
                                // 합니다(항목 33을 같이 보세요).
 
void RCObject::addReference(void)
{ ++refCount; }
 
void RCObject::removeReference(void)
{ if (--refCount == 0) delete this; }
 
void RCObject::markUnshareable(void)
{ shareable = false; }
 
bool RCObject::isShareable(void) const
{ return shareable; }
 
bool RCObject::isShared(void) const
{ return refCount > 1; }
 
// 여기서 이상한 부분을 하나 찾을 수 있는데,
// refCount가 두 개의 생성자에서 0으로
// 세팅되고 있다는 점입니다. 보시면 알겠지만,
// 이것은 RCObject를 생성하는 쪽에서
// refCount를 1로 초기화 하는 작업을
// 간단히 한 것이기 때문에, 여기에서는 그냥
// 건드리지 않는 것이 좋겠습니다.
 
// 이상한 부분 또 하나는 복사 생성자가 무조건
// refCount를 0으로 세팅하고 있다는 점입니다.
// 그 이유는 값을 나타내는 객체를 새로 만들고
// 있고, 새로 만들어진 객체의 값은 언제나
// 공유되지 않은 상태에 있으며, 그 객체를
// 생성한 쪽에서만 참조하기 때문입니다.
// 다시 말씀드리지만 객체를 생성한 쪽에서는
// refCount에 적당한 값을 넘겨주어야 합니다.
 
// RCObject의 대입 연산자는 아무 것도
// 하지 않습니다. 사실 이 연산자가 호출될
// 가능성은 영원히 없을 것 같습니다.
// RCObject는 공유되는 값을 나타내는
// 객체에 대한 기본 클래스이고, 참조 카운팅을
// 사용해서 만든 시스템에서 이런 객체가
// 다른 객체에 대입될 일은 없습니다.
 
// 그럼에도 불구하고, 어쩌다가 RCObject로부터
// 상속받은 클래스에서 참조 카운트로 관리되는
// 값에 대한 대입 연산이 이루어질 것으로
// 예상하기는 그리 어렵지 않습니다.
// 그렇기 때문에 RCObject의 대입 연산자는
// 어떤 일이라도 해야 하고, 그 '어떤 일'이
// 바로 '무위(아무 것도 안 함)'라는 것입니다.
 
// 예를 들어 StringValue 객체인 sv1과 sv2가
// 있을 때, 다음과 같이 대이 연산이 수행되면
// sv1과 sv2의 참조 카운트에는 어떻게
// 변해야 할까요?
 
sv1 = sv2;                        // sv1과 sv2의 참조 카운트는
                                // 어떤 영향을 받을까요?
 
// 위 문장이 수행된 결과, sv1의 공유값
// (문자열값)만 변하면 되고, sv1과 sv2의
// 참조 카운트는 모두 변하지 않습니다.
 
 
//-----------------------------------------------------------------------------
// 실제로 RCObject를 사용한 예를 봅시다.
// StringValue 클래스가 이 RCObject
// 기본 클래스로부터 참조 카운팅 기능을 물려받도록
// 고쳐 봅시다.
 
class String {
private:
    struct StringValue: public RCObject {
        char *data;
        
        StringValue(const char *initValue);
        ~StringValue(void);
    };
    ...
};
 
String::StringValue::StringValue(const char *initValue)
{
    data = new char[strlen(initValue) + 1];
    strcpy(data, initValue);
}
 
String::StringValue::~StringValue(void)
{
    delete[] data;
}
```
### 참조 카운트 동작을 자동화하기
* RCObject 클래스에는 참조 카운트를 저장할 수 있는 공간과, 이 참조 카운트를 조작할 수 있는 멤버 함수가 들어 있습니다. 하지만 이 함수는 직접 호출해 주어야 합니다. 이를테면, StringValue 객체에 대해 addReference와 removeReference를 호출하는 일은 String의 복사 생성자와 String의 대입 연산자가 마땅히 해 주어야 한다는 이야기입니다. 여기서 StringValue 객체를 가리키는 포인터(String 클래스의 경우 value 멤버)에, 벙어리 포인터 대신 스마트 포인터를 사용하면, addReference와 removeReference의 호출도 자동화할 수 있습니다.
```c++
// T에 대한 스마트 포인터의 템플릿. T는 RCObject
// 인터페이스를 지원해야 하는데, 대개 이것은 RCObject를
// 상속함으로써 해결합니다.
 
template <class T>
class RCPtr {
public:
    RCPtr(T* realPtr = 0);
    RCPtr(const RCPtr& rhs);
    ~RCPtr(void);
    
    RCPtr& operator =(const RCPtr& rhs);
    
    T* operator ->(void) const;        // 항목 28을 보세요.
    T& operator *(void) const;        // 항목 28을 보세요.
    
private:
    T *pointee;                        // 이 객체가 흉내내는
                                    // 벙어리 포인터
    void init(void);                // 공통 초기화
};                                    // 코드
 
template <class T>
RCPtr<T>::RCPtr(T* realPtr) : pointee(realPtr)
{
    init();
}
 
template <class T>
RCPtr<T>::RCPtr(const RCPtr& rhs) : pointee(rhs.pointee)
{
    init();
}
 
template <class T>
void RCPtr<T>::init(void)
{
    if (pointee == 0) {                // 벙어리 포인터가 널(null)이면,
        return;                        // 스마트 포인터도 널입니다.
    }
    if (pointee->isShareable() == false) {    // 이 객체의 값을 공유하는
        pointee = new T(*pointee);    // 것이 불가능하면,
    }                                // 그 값을 복사합니다.
    
    pointee->addReference();        // 이 값에 대한 새 참조자가 생겼음을
}                                    // 알 수 있습니다.
 
// 여기서 문제가 한가지 있습니다.
// 위에서 등장한 문장 중 아래 문장을 잘 봅시다.
 
pointee = new T(*pointee);
 
// pointee의 타입은 T에 대한 포인터입니다.
// 따라서 이 문장은 T 객체를 새로 생성하고
// T의 복사 생성자를 호출하여 그 객체를
// 초기화 합니다.
 
// RCPtr이 String 클래스에 사용되는 경우에
// T는 String::StringValue가 되므로,
// 앞의 문장은 결국 String::StringValue의
// 복사 생성자를 호출하는 셈이 되지요.
 
// 하지만, 이 클래스에 대해 우리가 선언해놓은
// 복사 생성자가 없기 때문에, 컴파일러가 대신
// 만들어 준 복사 생성자를 통해 얕은 복사를
// 수행하게 됩니다. 이것은 재앙이나 마찬가지입니다.
 
// 그렇기 때문에, 포인터를 가진 클래스에 대해서는
// 복사 생성자와 대입 연산자를 꼭 직접 만드는
// 습관을 들여야 합니다.
 
// 이에 따라 StringValue의 내부를 조금
// 고칠 필요가 있겠습니다.
 
class String {
private:
    struct StringValue: public RCObject {
        StringValue(const StringValue& rhs);
        ...
    };
    ...
};
 
String::StringValue::StringValue(const StringValue& rhs)
{
    data = new char[strlen(rhs.data) + 1];
    strcpy(data, rhs.data);
}
 
// 이 복사 생성자는 RCPtr<T>에 대한
// 가정 하나를 깔고 만들어졌습니다.
// 그 가정이란, T는 RCObject에서
// 파생된 것이든지, 아니면 최소한 RCObject의
// 기능을 모두 갖고 있어야 한다는 것입니다.
// 이 부분에 대해서는 설명 문서에 언급해 두어야 합니다.
 
// RCPtr<T>의 가정 또 한 가지는
// '가리킴을 당하는' 객체의 타입이
// 반드시 T라는 것입니다.
// 사실 RCPtr<T>는 T의 파생 타입도
// 가리킬 수 있지만, 현재의 String
// 클래스의 경우, StringValue에서
// 클래스가 파생되는 것 자체를 가정하지
// 않기 때문에, 이 부분에 대한 사항은
// 무시하려고 합니다.
 
template <class T>
RCPtr<T>& RCPtr<T>::operator =(const RCPtr& rhs)
{
    if (pointee != rhs.pointee) {            // 값이 변하지 않으면
                                            // 대입 연산을 수행하지 않고
                                            // 건너 뜁니다.
        
        T *oldPointee = pointee;            // 바로 전의 pointee 값을 저장합니다.
        
        pointee = rhs.pointee;                // 새 값을 가리킵니다.
        init();                                // 가능하면 이 값을 공유하고,
                                            // 그렇지 않으면 별도의 사본을 만듭니다.
        
        if (oldPointee) {
            oldPointee->removeReference();    // 현재의 값에 대한 참조자를
        }                                    // 제거합니다.
    }
    
    return *this;
}
 
template <class T>
RCPtr<T>::~RCPtr(void)
{
    if (pointee) pointee->removeReference();
}
 
template <class T>
T* RCPtr<T>::operator ->(void) const { return pointee; }
 
template <class T>
T& RCPtr<T>::operator *(void) const { return *pointee; }
```
### 헤쳐 모여!
* 지금까지 공부하며 만들어 놓은 이것저것을 모아서, RCObject와 RCPtr 클래스에 기반한 참조 카운팅 String 클래스를 만들어 보자고요.

```c++
template <class T>                    // T에 대한 스마트 포인터 객체를
class RCPtr {                        // 만드는 템플릿. T는 반드시
public:                                // RCObject에서 파생된 것이어야 합니다.
    RCPtr(T* realPtr = 0);
    RCPtr(const RCPtr& rhs);
    ~RCPtr(void);
    
    RCPtr& operator =(const RCPtr& rhs);
    
    T* operator ->(void) const;
    T& operator *(void) const;
    
private:
    T *pointee;
    void init(void);
};
 
class RCObject {                    // 참조 카운팅 기능을 갖춘 객체를 위한
public:                                // 기본 클래스
    void addReference(void);
    void removeReference(void);
    
    void markUnshareable(void);
    bool isShareable(void) const;
    
    bool isShared(void) const;
    
protected:
    RCObject(void);
    RCObject(const RCObject& rhs);
    RCObject& operator =(const RCObject& rhs);
    virtual ~RCObject(void) = 0;
    
private:
    int refCount;
    bool shareable;
};
 
class String {                        // 애플리케이션 개발자가
public:                                // 사용하게 될 클래스
    String(const char *value = "");
    
    const char& operator [](int index) const;
    char& operator [](int index);
    
private:
    // 문자열 값을 나타내는 클래스
    struct StringValue: public RCObject {
        char *data;
        
        StringValue(const char *initValue);
        StringValue(const StringValue& rhs);
        void init(const char *initValue);
        ~StringValue(void);
    };
    RCPtr<StringValue> value;
};
 
// 이제 String 클래스에는
// 복사 생성자, 대입 연산자, 소멸자가 모두
// 필요 없어졌습니다.
 
// 컴파일러가 만들어 낸 String의 복사 생성자가
// 자동으로 RCPtr멤버의 복사 생성자를 호출하는데,
// 이 클래스의 복사 생성자가 StringValue
// 객체를 적절하게 조작해 줍니다.
 
// 물론 깊은 복사 외에, 참조 카운팅, 소멸과
// 같은 작업도 다 알아서 처리해 줍니다.
 
 
// RCObject의 구현 코드를 보시죠.
 
RCObject::RCObject(void)
: refCount(0), shareable(true) {}
 
RCObject::RCObject(const RCObject&)
: refCount(0), shareable(true) {}
 
RCObject& RCObject::operator =(const RCObject&)
{ return *this; }
 
RCObject::~RCObject(void) {}
 
void RCObject::addReference(void) { ++refCount; }
 
void RCObject::removeReference(void)
{ if (--refCount == 0) delete this; }
 
void RCObject::markUnshareable(void)
{ shareable = false; }
 
bool RCObject::isShareable(void) const
{ return shareable; }
 
bool RCObject::isShared(void) const
{ return refCount > 1; }
 
 
// 그리고 RCPtr을 구현한 코드도요.
 
template <class T>
void RCPtr<T>::init(void)
{
    if (pointee == 0) return;
    if (pointee->isShareable() == false) {
        pointee = new T(*pointee);
    }
    
    pointee->addReference();
}
 
template <class T>
RCPtr<T>::RCPtr(T* realPtr)
: pointee(realPtr)
{ init(); }
 
template <class T>
RCPtr<T>::RCPtr(const RCPtr& rhs)
: pointee(rhs.pointee)
{ init(); }
 
template <class T>
RCPtr<T>::~RCPtr(void)
{ if (pointee) pointee->removeReference(); }
 
template <class T>
RCPtr<T>& RCPtr<T>::operator =(const RCPtr& rhs)
{
    if (pointee != rhs.pointee) {
        T *oldPointee = pointee;
        pointee = rhs.pointee;
        init();
        if (oldPointee) oldPointee->removeReference();
    }
    
    return *this;
}
 
template <class T>
T* RCPtr<T>::operator ->(void) const { return pointee; }
 
template <class T>
T& RCPtr<T>::operator *(void) const { return *pointee; }
 
 
// String::StringValue를 구현한 코드는 다음과 같습니다.
 
void String::StringValue::init(const char *initValue)
{
    data = new char[strlen(initValue) + 1];
    strcpy(data, initValue);
}
 
String::StringValue::StringValue(const char *initValue)
{ init(initValue); }
 
String::StringValue::StringValue(const StringValue& rhs)
{ init(rhs.data); }
 
String::StringValue::~StringValue(void)
{ delete[] data; }
 
 
// 결국 모든 길은 String으로 향하게 되어 있습니다.
// String 클래스는 이렇게 구현하고요.
 
String::String(const char *initValue)
: value(new StringValue(initValue)) {}
 
const char& String::operator [](int index) const
{ return value->data[index]; }
 
char& String::operator [](int index)
{
    if (value->isShared()) {
        value = new StringValue(value->data);
    }
    
    value->markUnshareable();
    
    return value->data[index];
}
```
### 기존의 클래스에 참조 카운팅 기능을 부착하는 방법
* 지금까지 공부한 모든 내용은 클래스의 소스 코드를 우리가 직접 건드릴 수 있다는 가정을 아래에 깔고 있었습니다. 하지만 참조 카운팅 기능을 넣었으면 하는 클래스가 애석하게도 우리의 손이 미치지 않는 라이브러리에 들어있다면, 어떻게 하시겠습니까? 이를테면, 어떤 클래스 라이브러리에 Widget이란 클래스가 들어있는데, 이 Widget에 참조 카운팅 기능을 붙이고 싶다고 합시다. 지금까지 우리가 공부한 것처럼 Widget이 RCObject을 상속하게 한다든지 하는 방법을 쓸 수 없기 때문에, RCPtr 같은 것도 쓸 수 없습니다.
* 우선, Widget이 RCObject를 상속했더라면 어떤 설계가 나올 수 있을지 생각해 보는 것으로 시작하지요. 이 경우에는 RCWidget이란 클래스를 만들어서 사용자가 쓸 수 있게 해야 할 것입니다. 하지만 이렇게 놓고 보면 모든 상황이 String/StringValue 예제와 흡사하게 만들어집니다. RCWidget은 String에, Widget은 StringValue에 대응되는 것이죠.
* 하지만 실제로는 Widget이 RCObject를 상속받을 수 없으니, 간접적인 수단을 덧붙여서 해결해야 합니다. 먼저 CountHolder라는 이름의 클래스를 끌어들였습니다. 참조 카운트를 가지는 역할을 맡은 이 클래스는 RCObject를 상속합니다. 게다가 이 클래스에는 Widget에 대한 포인터도 넣습니다. 마지막으로 RCPtr 템플릿 대신에 RCIPtr란 이름의 템플릿으로 교체합니다. 이 템플릿은 CountHolder 클래스를 조작할 수 있습니다(RCIPtr의 "I"는 "indirect"를 의미합니다).
```c++
template<class T>
class RCIPtr {
public:
    RCIPtr(T* realPtr = 0);
    RCIPtr(const RCIPtr& rhs);
    ~RCIPtr(void);
    
    RCIPtr& operator =(const RCIPtr& rhs);
    
    const T* operator ->(void) const;    // 이 함수들이 이런
    T* operator ->(void);                // 방식으로 선언된 이유는
    const T& operator *(void) const;    // 소스 코드 이후에
    T& operator *(void);                // 설명되어 있습니다.
    
private:
    struct CountHolder: public RCObject {
        ~CountHolder(void) { delete pointee; }
        T* pointee;
    };
    
    CountHolder *counter;
    
    void init(void);
    void makeCopy(void);                // 다음을 보세요.
};
 
template <class T>
void RCIPtr<T>::init(void)
{
    if (counter->isShareable() == false) {
        T *oldValue = counter->pointee;
        counter = new CountHolder;
        counter->pointee = new T(*oldValue);
    }
    
    counter->addReference();
}
 
template <class T>
RCIPtr<T>::RCIPtr(T* realPtr)
: counter(new CountHolder)
{
    counter->pointee = realPtr;
    init();
}
 
template <class T>
RCIPtr<T>::RCIPtr(const RCIPtr& rhs)
: counter(rhs.counter)
{ init(); }
 
template <class T>
RCIPtr<T>::~RCIPtr(void)
{ counter->removeReference(); }
 
template <class T>
RCIPtr<T>& RCIPtr<T>::operator =(const RCIPtr& rhs)
{
    if (counter != rhs.counter) {
        counter->removeReference();
        counter = rhs.counter;
        init();
    }
    return *this;
}
 
template <class T>                        // 기록시점 복사
void RCIPtr<T>::makeCopy(void)            // (Copy-on-write(COW))
{                                        // 의 복사 부분을 구현한 코드
    if (counter->isShared()) {
        T *oldValue = counter->pointee;
        counter->removeReference();
        counter = new CountHolder;
        counter->pointee = new T(*oldValue);
        counter->addReference();
    }
}
 
template <class T>                                // 읽기 전용(const) 액세스
const T* RCIPtr<T>::operator ->(void) const        // 즉, COW가 필요없습니다.
{ return counter->pointee; }
 
template <class T>                                // 읽기/쓰기(const가 아님)
T* RCIPtr<T>::operator ->(void)                    // 액세스. 즉, COW가
{ makeCopy(); return counter->pointee; }        // 필요합니다.
 
template <class T>                                // 읽기 전용(const) 액세스
const T& RCIPtr<T>::operator *(void) const        // 즉, COW가 필요없습니다.
{ return *(counter->pointee); }
 
template <class T>                                // 읽기/쓰기(const가 아님)
T& RCIPtr<T>::operator *(void)                    // 액세스. 즉, COW를
{ makeCopy(); return *(counter->pointee); }        // 수행합니다.
 
// 이렇게 만들어진 RCIPtr은 RCPtr과 비교해서
// 두 가지가 다릅니다.
 
// 첫째, RCPtr은 값을 직접 가리키고 있는 반면,
// RCIPtr은 CountHolder 객체로 하여금
// 값을 가리키게 하고 자신은 CountHolder를
// 가리킵니다.
 
// 둘째, RCIPtr은 operator->와 operator*를
// 오버로딩해서, 가리킴을 당하는 객체에 대해 읽기/쓰기
// 액세스가 이루어질 때마다 자동으로 기록시점 복사가
// 이루어지도록 하였습니다.
```
* RCIPtr이 만들어진 이상, RCWidget를 구현하기는 더 이상 꿈이 아닙니다. RCWidget의 멤버 함수는 Widget 객체에 대한 RCIPtr을 통해 Widget 객체의 멤버 함수에게 책임을 전가하는(!) 것으로 구현할 수 있기 때문입니다.
```c++
// 예를 들어 Widget이 다음과 같이 생겼다면,
 
class Widget {
public:
    Widget(int size);
    Widget(const Widget& rhs);
    ~Widget(void);
    Widget& operator =(const Widget& rhs);
    
    void doThis(void);
    int showThat(void) const;
};
 
// RCWidget은 이렇게 정의할 수 있을 것입니다.
 
class RCWidget {
public:
    RCWidget(int size) : value(new Widget(size)) {}
    void doThis(void) { value->doThis(); }
    int showThat(void) const { return value->showThat(); }
    
private:
    RCIPtr<Widget> value;
};
 
// RCWidget의 멤버 함수가 하는 일은,
// value가 가리키고 있는 Widget 객체가
// 가진, 똑같은 이름의 멤버 함수를 호출하고 그
// 반환값을 그대로 반환하는 일 뿐입니다.
 
// 이와 아울러, RCWidget에는 복사 생성자,
// 대입 연산자, 소멸자가 없습니다. String
// 클래스와 마찬가지로, 이 세 개의 함수는
// 작성할 필요가 없습니다. RCIPtr의 똘똘한
// 동작 덕택이죠.
```
### 총정리
* 참조 카운팅은 무료로 얻어지지 않습니다. 참조 카운팅으로 유지되는 객체값(object value)에는 참조 횟수(카운트)가 반드시 포함되며, 관련된 거의 모든 동작에서 이 참조 횟수를 조작해야 합니다. 게다가 참조 카운팅을 쓰는 클래스의 코드는 그렇지 않은 클래스보다 더 복잡합니다. String클래스를 예로 들면 참조 카운팅을 하기 위해 세 개의 보조 클래스(StringValue, RCObject, RCPtr)가 필요합니다. 이를 제작, 시험, 문서화, 유지보수하는 것이 한 개의 클래스에 대해 이렇게 하는 것 보다 힘든 것은 불을 보듯 뻔합니다.

* 참조 카운팅은 객체들 사이에서 값의 공유가 빈번하다는 가정 하에 고안한 최적화 기법입니다(항목 18도 같이 보세요). 만약에 참조 카운팅을 쓴 코드가 메모리도 더 많이 먹고 코드도 더 많이 실행한다면, 이 가정은 실패한 것입니다. 참조 카운팅이 효율 향상에 효과적일 수 있는 상황은 다음의 두 가지로 정리할 수 있습니다.
    * 상대적으로 많은 객체들이 상대적으로 적은 값을 공유할 때.
        * 이러한 상황은 대개 대입 연산자와 복사 생성자가 호출될 때 만들어집니다. 객체 개수에 대한 값 개수의 비율이 클수록 참조 카운팅을 할 필요가 더 커집니다.
    * 어떤 객체값을 생성하거나 소멸시키는데 많은 비용이 들거나 메모리 소모가 클 때.
        * 이러한 경우라고 해도, 여러 객체들이 값을 공유하지 않으면 참조 카운팅은 별 효과를 보이지 않습니다.


* 앞의 조건이 만족되는지 확인하는 방법은 오로지 하나입니다. 프로그램을 프로파일링하고 설치해 보는 것입니다(항목 16 참조).

* 앞의 조건이 충족되었다고 하더라도 참조 카운팅을 사용한 코드 설계가 여전히 적합하지 않을 경우도 있습니다. 어떤 자료구조는 자기 참조형(self-referential) 혹은 원형 의존형(circular dependency) 구조를 띄고 있어서(방향성 그래프(directed graph)가 한 예인데, 트리(tree)구조가 여기에 속합니다), 참조 카운팅 적용에 애로사항을 일으키기도 합니다. 이런 자료구조에서는 사용되지 않는(고립된) 객체들이 생기는 경향이 있는데, 이 객체들의 참조 카운트는 절대로 0이 되지 않습니다. 왜냐하면, 사용되지 않은 구조 안의 객체는 같은 구조 안에 있는 최소한 하나의 다른 객체가 가리키고 있기 때문입니다.

* 마지막으로 한 가지만 더 이야기하고 이번 항목을 마무리지어볼까 합니다. RCObject::removeReference는 객체의 참조 카운트를 감소시키고, 감소 후 값이 0인 경우 this에 대해 delete를 호출함으로써 객체를 소멸시킵니다. delete를 호출한다는 것은, RCObject의 생성이 오로지 new를 통해 힙에 할당되어야 한다는 것을 의미하고, 이것이 지켜지도록 제약할 수 있는 어떤 방법이 필요합니다.

* 간단히 말해, 이번 항목에서 사용한 방법은 'RCObject를 생성할 수 있는 클래스를 정해 놓고, 그 클래스를 사용할 때 그 제약을 잘 지키라고 알리는 것'이라고 할 수 있겠습니다. 참조 카운팅 객체를 생성할 수 있는 자격을 제한하고, 자격을 가진 쪽은 반드시 그 객체 생성에 수반되는 규칙을 따를 것을 명확히 알려야 한다는 것이죠.


## 항목 30 : 프록시(Proxy) 클래스
* C++은 배열을 사용하는 방법이 다차원과 거리가 멀다. 
```c++
int data[10][20];//2차원 배열: 10행 20열
//하지만 변수를 배열의 차원으로 사용하는 경우 컴파일러가 싫어합니다.

void processInput(int dim1, int dim2)
{
    int data[dim1][dim2];//에러. 배열의 차원은 컴파일 과정에 정확히 주어져야함
}

int* data = new int[dim1][dim2];//에러입니다. 

```

### 2차원 배열 구현하기
* C/C++에서의 다차원 배열은 사실 컴파일러가 1차원 배열을 가지고 다차원 배열을 흉내낸 것에 불과합니다. 게다가 C/C++은 어떤 변수의 값 만큼의 크기를 갖는 배열을 선언하는것을 싫어합니다. 이것을 극복하기 위해 C++에서는 OOP의 성질을 살리겠다고 '배열을 나타내는 클래스'를 만들어 구현하는것이 보통입니다. 이를테면 2차원 배열을 나타내는 클래스를 만들어 사용하는 것이죠.

* 그런데 이런 배열 객체는 왠지 조금 뻑뻑한 느낌을 줍니다. C/C++ 스타일의 배열 조작에 익숙한 우리들에게는 인덱스 연산자인 대괄호([])를 썼으면 하는 소망이 있을 테니까요. 다음과 같이 말이죠.
```c++
Array2D<int> data(10, 20);
cout << data[3][6];
```
* 하지만 operator[][] 란 것은 없기 때문에 직접적으로 이런 객체를 구현할 수는 없습니다. 여기서 1차원 배열로 다차원 배열을 흉내내는 방법에 대해서 생각해 볼 필요가 있습니다.
```c++
int data[10][20];
```
* 위에서 data는 사실 2차원 배열이 아니라 단지 10 개의 요소를 가진 1차원 배열입니다. 10개의 요소 자체가 바로 20개 요소의 배열이기 때문에, 결국 data[3][6]이란 표현식은 (data[3])[6]으로 풀어서 이해하시면 됩니다. data의 4번째 요소가 되는 배열 안의 7번째 요소인 것이죠.

* 이 원리를 똑같이 써서 Array2D를 가지고 놀 수 있습니다. 우선, operator[]의 오버로딩을 통해 또 다른 배열을 나타내는 클래스 객체를 반환하게 합니다. 이 클래스가 Array1D 입니다. 그리고 나서, Array1D의 operator[]를 오버로딩해서 원래의 2차원 배열 내의 요소를 반환하게 하는 것입니다.


```c++
template <class T>
class Array2D {
public:
    class Array1D {
    public:
        T& operator [](int index);
        const T& operator [](int index) const;
        ...
    };
    
    Array1D operator [](int index);
    const Array1D operator [](int index) const;
    ...
};
 
// 이렇게 만든 Array2D로는 이제
// 다음의 코드를 쓸 수 있습니다.
 
Array2D<float> data(10, 20);
...
cout << data[3][6];                // 오옷! 아무 문제없습니다.
```

* Array2D 클래스를 쓰는 개발자는 Array1D 클래스에 대해 전혀 알 필요가 없습니다. 이렇게 또 다른 객체를 대신하는 객체를 두는 방법이 꽤 쓸모가 있어서 이런 객체에 프록시 객체(proxy object)라는 이름을, 그리고 프록시 객체를 만드는 클래스를 가리켜 프록시 클래스(proxy class)라고 부릅니다. 이번 예제의 Array1D는 프록시 클래스이고, 이 클래스의 인스턴스는 개념적으로는 존재하지 않는 1차원 배열을 대신합니다.

### '진짜로' operator[] 의 읽기/쓰기를 구분하는 방법
* 프록시 클래스는 상당히 넓은 응용폭을 자랑합니다. 프록시 클래스의 국가대표급 응용 사례는 operator[] 의 읽기/쓰기 동작을 구분하는 것입니다. 이 순간, operator[] 를 지원한 참조 카운팅 문자열이 어렴풋이 생각나려고 합니다(항목 29 를 보세요).

* 이미 알고 계시겠지만 operator[]는 문자를 읽고, 문자를 쓰는 두 가지 상황에서 호출될 수 있습니다. 여기서 우리가 원하는 것은 operator[]가 읽기를 위해 쓰였는지 쓰기를 위해 쓰였는지를 구분하는 것입니다. 왜냐하면 우항값 연산(읽기)이 좌항값 연산(쓰기)보다 비용이 덜 들기 때문입니다. 특히 이 효과는 참조 카운팅 기능을 가진 자료구조에서 두드러집니다.

* 기본 아이디어는 이렇습니다. operator[]의 결과값이 사용되기까지 좌항값/우항값 연산을 늦추면 읽기와 쓰기를 다르게 처리할 수 있다는 것입니다. 우리가 필요로 하는 지연 시간을 벌어주는 일이 바로 프록시 클래스의 역할입니다. 이제 operator[]는 문자열 내의 문자 자체를 반환하지 않고 문자를 대신하는 프록시 객체를 반환하도록 바뀝니다.

* 예제 코드를 보기 전에 이 프록시를 정확하게 이해하기로 합시다. 프록시를 가지고 할 수 있는 일은 단 세 개뿐입니다.
    * 1. 프록시를 생성합니다. 즉, 프록시가 대신할 문자열 내의 문자를 지정합니다.
    * 2. 이 프록시를 대입 연산의 대상(target)으로 사용합니다. 이 경우는 프록시가 대신하는 문자에 대해 직접 대입 연산을 수행하는 것인데, 이렇게 사용되는 프록시는 operator[]를 호출한 문자열에 대한 좌항값을 대신한다고 할 수 있습니다.
    * 3. 이 프록시를 이외의 경우에 사용합니다. 이렇게 사용되는 프록시는 operator[]를 호출한 문자열에 대한 우항값을 대신한다고 할 수 있습니다.

```c++
class String {                // 참조 카운팅 기능을 가진 문자열
public:                        // 자세한 내용은 항목 29에서 공부하세요.
    
    class CharProxy {        // 문자열 내의 문자(char)를 대신하는 프록시
    public:
        CharProxy(String& str, int index);                // 프록시 생성
        
        CharProxy& operator =(const CharProxy& rhs);    // 좌항값(lvalue)
        CharProxy& operator =(char c);                    // 연산
        
        operator char(void) const;                        // 우항값 연산
        
    private:
        String& theString;                                // 이 프록시가 관계한 문자열 객체
        
        int charIndex;                                    // 이 문자열 안에서 프록시가
                                                        // 직접 대신하고 있는 문자
    };
    
    // String 클래스의 계속
    const CharProxy operator [](int index) const;        // 상수 String 객체를 위한 함수
    CharProxy operator [](int index);                    // 비상수 String 객체를 위한 함수
    ...
    friend class CharProxy;
    
private:
    RCPtr<StringValue> value;
};
 
// 다음 문장을 잘 봐주세요.
 
String s1;
...
cout << s1[5];
 
// 이 문장에서 s1[5]는 CharProxy 객체를
// 반환합니다. 이 객체에는 출력용 연산자(<<)가
// 정의되어 있지 않기 때문에, C++ 컴파일러는
// operator <<를 성사시키기 위해
// char타입으로의 암시적 변환 함수를
// 호출하도록 합니다.
 
// CharProxy가 우항값으로 사용될 때에는
// 반드시 CharProxy에서 char로의 변환이
// 이루어집니다.
 
// 좌항값 연산은 조금 다르게 처리됩니다.
// 다음을 보세요.
 
String s2;
...
s2[5] = 'x';
 
// 표현식 s2[5]가 CharProxy 객체를 반환한다는
// 것은 이전의 경우와 마찬가지입니다.
// 하지만 이번의 CharProxy 객체는 대입
// 연산의 대상이 되는 객체입니다.
 
// 그렇기 때문에, 호출되는 대입 연산자는
// CharProxy 안에 정의되어 있는 것이란
// 사실은 확실합니다.
 
// 따라서 CharProxy 안에 정의된
// 대입 연산자는 이 문자에 대한 좌항값
// 연산을 구현하는 데에 필요한 동작을
// 취하면 될 것입니다.
 
const String::CharProxy String::operator [](int index) const
{
    return CharProxy(const_cast<String&>(*this), index);
}
 
String::CharProxy String::operator [](int index)
{
    return CharProxy(*this, index);
}
 
// const 버전의 operator []는
// 반환할 CharProxy 객체를 생성할 때
// *this에 const_cast(항목 2 참조)
// 를 사용하고 있습니다. 캐스트 연산자는
// 골칫거리이기 쉽지만, 이런 경우에는
// operator []가 반환한 CharProxy
// 객체가 상수 타입이기 때문에, 이 프록시가
// 나타내는 문자를 가진 String 객체가
// 변경될 위험은 없습니다.
 
String::CharProxy::CharProxy(String& str, int index)
: theString(str), charIndex(index) {}
 
String::CharProxy::operator char(void) const
{
    return theString.value->data[charIndex];
}
 
String::CharProxy& String::CharProxy::operator =(const CharProxy& rhs)
{
    // 이 String 객체가 다른 String 객체와 값을 공유하고 있는 상태에 있다면,
    // 이 String 객체만 가질 수 있는 값의 사본을 따로 만듭니다.
    if (theString.value->isShared()) {
        theString.value = new StringValue(theString.value->data);
    }
    
    // 이제 대입합니다. rhs가 나타내는 문자값을
    // *this가 나타내는 문자값에 대입합니다.
    theString.value->data[charIndex] =
        rhs.theString.value->data[rhs.charIndex];
    
    return *this;
}
 
// 문자값 대입을 위해 String의 private 멤버 데이터인
// value가 직접 사용되었기 때문에, CharProxy를
// String의 프렌드로 선언한 것입니다.
 
// 두 번째의 대입 연산자 역시 거의 비슷한 구조로 만들어져 있습니다.
String::CharProxy& String::CharProxy::operator =(char c)
{
    if (theString.value->isShared()) {
        theString.value = new StringValue(theString.value->data);
    }
    
    theString.value->data[charIndex] = c;
    
    return *this;
}
 
// 중복된 코드를 제거하는 일은 여러분에게 맡깁니다.
```

### 프록시 클래스라고 능사는 아니다
* 프록시 클래스에도 단점이 있습니다. 우리에겐 프록시 객체와 프록시 객체가 대신하는 객체가 감쪽같이 자리바꿈했으면 하는 소망이 있지만, 실상은 이게 꽤 만만찮습니다. 왜냐하면, 객체가 좌항값으로 사용되는 경우는 대입 연산 외에서도 찾을 수 있고, 이런 경우에 프록시 객체를 사용하면 원래의 객체를 사용하는 것과 사뭇 다른 동작 결과를 보게 되기 때문입니다.

* 항목 29에서 StringValue 객체에 공유 가능성 플래그를 추가할 필요성을 보여준 그 코드를 다시 가져와 보았습니다. String::operator[]가 반환하는 것이 char&가 아닌 CharProxy이면 다음의 코드는 절대로 컴파일되지 않습니다.
```c++
String s1 = "Hello";
char *p = &s1[1];
// 에러입니다!
```
* 이 난관을 극복하려면 CharProxy 클래스의 주소 연산자(address of, & 연산자)를 오버로딩할 수 밖에 없습니다.
```c++
class String {
public:
    
    class CharProxy {
    public:
        ...
        char* operator &(void);
        const char* operator &(void) const;
        ...
    };
    ...
};
 
const char* String::CharProxy::operator &(void) const
{
    return &(theString.value->data[charIndex]);
}
 
char* String::CharProxy::operator &(void)
{
    // 이 함수가 반환하는 포인터가 가리키는 문자가
    // 다른 String 객체들이 공유하는 문자열에 속해 있지 않도록 주의합니다.
    if (theString.value->isShared()) {
        theString.value = new StringValue(theString.value->data);
    }
    
    // 이 함수가 반환한 포인터가 언제까지 사용되는지
    // 알 수 없기 때문에, 이 StringValue 객체는 절대로
    // 공유되지 않도록 설정합니다.
    theString.value->markUnshareable();
    
    return &(theString.value->data[charIndex]);
}
 
// CharProxy의 다른 멤버 함수에서
// 익히 봐 왔던 코드가 대부분이기 때문에,
// 역시 별도의 private 멤버 함수에 몰아
// 넣고 싶은 독자가 있을 것 같네요.
```

* CharProxy로 char를 대신할 수 없는 경우의 두 번째 예는 다음의 코드를 가지고 설명할까 합니다. Array는 참조 카운팅 기능을 가진 배열의 템플릿인데, operator[]의 좌항값/우항값 연산을 구분하려고 프록시 클래스를 사용했습니다.
```c++
Array<int> intArray;
...
intArray[5] = 22;
// 여기까진 문제없다가

intArray[5] += 5;
// 윽, 에러입니다!

++intArray[5];
// 또 에러입니다!
```
* 원인은 간단합니다. operator []는 Proxy 객체를 반환하는데, Proxy 클래스 안에는 operator +=나 operator ++ 함수가 정의되어 있지 않으니 컴파일이 안 되는 것이죠. 이 외에도 좌항값을 필요로 하는 연산자(operator *, operator <<=, operator -- 등)에서는 이런 상황이 어김없이 발생합니다. 이런 연산자들에 대해서도 제대로 작동하게 하려면, Array<T>::Proxy 클래스 안에 이 연산자들을 정의해 넣어야 합니다.

* 프록시를 통해 원래 객체의 멤버 함수를 호출할 때에도 비슷한 문제가 끼어 듭니다. Rational는 유리수를 나타내는 클래스 입니다. 위의 Array 템플릿에 Rational 클래스를 적용하면 다음과 같은 문제가 발생합니다.
```c++
Array<Rational> array;
// 여기까지는 좋습니다.

cout << array[4].numerator();
// 에러입니다!

int denom =
    array[22].denominator();
// 에러입니다!
```
* 결국 프록시가 원래의 객체처럼 동작하게 만들려면, 원래 객체에 적용할 수 있는 함수를 모조리 오버로딩해서 프록시에도 적용될 수 있도록 해야 합니다.

* 프록시가 원래의 객체를 대신할 수 없는 경우는 아직도 남아 있습니다. 바로 비상수 객체 참조자를 받아들이는 함수의 매개변수로 프록시를 넘기는 경우입니다.
```c++
void swap(char& a, char& b);
// a와 b의 값을 맞바꾸는 함수

String s = "+C+";
// 윽, "C++"을 잘못 썼습니다.

swap(s[0], s[1]);
// 이렇게 해서 문제를 해결했으면
// 좋겠는데, 이 코드는 컴파일되지
// 않습니다.
```

* 이제 프록시 객체가 원래 객체를 대체할 수 없는 경우가 딱 하나 남았습니다. '암시적 타입변환'이 바로 마지막 경우입니다.

```c++
class TVStation {
public:
    TVStation(int channel);
    ...
};
 
void watchTV(const TVStation& station, float hoursToWatch);
 
// int를 TVStation 타입으로 바꾸는
// 암시적 타입변환 함수(항목 5 참조)가
// 있기 때문에, 다음과 같은 코드를
// 쓸 수도 있습니다.
 
watchTV(10, 2.5);            // 2시간 30분 동안
                            // 채널 10을 봅니다.
 
// 그런데 프록시 클래스로 operator []의
// 좌항값/우항값 연산을 구분하고 참조 카운팅
// 기능도 가진 배열 템플릿을 써서 비슷한
// 코드를 쓰려고 하면 에러가 납니다.
 
Array<int> intArray;
intArray[4] = 10;
watchTV(intArray[4], 2.5);    // 에러입니다! Proxy<int>를
                            // TVStation 타입으로
                            // 변환할 수 없습니다.
 
// 이 경우에 int 타입이 TVStation 타입으로
// 변환되려면 다음과 같이 두 번의
// 암시적 타입변환 과정이 필요합니다.
// Proxy<int> -> int -> TVStation
 
// 하지만 C++에서는 두 번 이상의
// 암시적 타입변환이 금지되어 있습니다.
 
// 사실, 이 경우에는 좀 더 신경써서
// 설계한 TVStation 클래스였다면
// 생성자를 explicit로 선언해 두었을
// 것이고, 첫 번째 watchTV 호출도
// 컴파일되지 않았을 것입니다.
```
### 총정리
* 프록시 클래스를 이용하면 어떤 특정한 동작 원리 몇 가지를 아주 쉽게 구현할 수 있습니다. 다차원 배열이 첫 번째 예이고, 좌항값/우항값 구분이 두 번째 예이며, 암시적 타입변환(항목 5 참조)의 방지가 세 번째 예입니다.

* 이와 동시에, 프록시 클래스에는 단점도 있습니다. 함수로부터 반환되는 프록시 객체는 임시 객체(temporary, 항목 19 참조)이기 때문에, 객체 생성 및 소멸 과정이 저절로 수반되어야 합니다(비용이 듭니다). 또한 프록시 클래스를 사용한 소프트웨어 시스템은 그렇지 않은 것보다 더 복잡합니다. 클래스 몇 개가 더 추가되기 때문에 설계, 구현, 파악, 유지보수 등에 관한 작업이 어려워질 수밖에 없지요.

* 마지막으로, 원래의 객체를 사용하는 클래스를 프록시를 사용하는 버전으로 바꿀 때에는 그 클래스의 의미구조도 어쩔 수 없이 바뀌어야 하는 경우도 더러 있습니다. 프록시 객체는 대개 원래 객체와 약간 다른 동작 패턴을 보이기 때문입니다. 하지만 프록시를 사용자에게까지 드러내는 동작이 필요한 경우는 별로 많지 않습니다.


## 항목 31 : 함수를 두 개 이상의 객체(타입)에 대해 가상 함수처럼 동작하도록 만들기
* 두 개 이상의 객체에 대해 가상 함수처럼 동작하는 함수가 필요한 경우의 예

* 여러분은 게임을 하나 만들기로 작정했습니다. 이 게임에는 GameObject를 상속하여 구현한 SpaceShip(우주선), SpaceStation(정거장), Asteroid(소행성)가 있고, 이들끼리 충돌한 경우 다음과 같은 규칙을 따른다고 합시다.
    * 우주선과 정거장이 느린 속도로 충돌하면, 우주선은 정거장에 도킹합니다. 충돌할 때의 속도가 느리지 않았다면, 우주선과 정거장은 충돌 속도에 비례하여 손상을 입습니다.
    * 우주선과 우주선, 또는 정거장과 정거장이 충돌하면, 충돌한 것들 모두 충돌 속도에 비례하는 손상을 입습니다.
    * 작은 소행성이 우주선 또는 정거장과 충돌하면, 소행성이 부서집니다. 충돌한 소행성이 좀 큰 것이라면 우주선과 정거장이 부서집니다.
    * 한 소행성이 다른 소행성과 충돌하면, 두 소행성이 작은 소행성들로 쪼개지면서 모든 방향으로 튕겨져 나갑니다.

* 이 게임에는 오브젝트들의 충돌을 처리하는 다음과 같은 함수가 있을 것입니다.
```c++
void checkForCollision(
    GameObject& object1,
    GameObject& object2)
{
    if (theyJustCollided(object1,
                         object2))
    {
        processCollision(object1,
                         object2);
    }
    else
    {
        ...
    }
}
```
* processCollision이 호출되었을 때, 함수 안에서는 object1과 object2가 충돌되었다는 사실을 알고 있기 때문에, object1과 object2가 실제로 어떤 것이냐에 따라 적절한 충돌처리를 해 주어야 합니다. 하지만 충돌한 객체가 진짜로 무엇인지는 알 방법이 없습니다.

* 만약 object1의 동적 타입만 보고 충돌처리를 할 수 있다면, processCollision을 GameObject 클래스의 가상 함수로 선언해 놓고 object1.processCollision(object2)을 호출하면 문제가 해결되었을 것입니다.

* 하지만 이 경우에 충돌 처리를 제대로 하려면 object1과 object2의 동적 타입이 모두 필요합니다. 이런 경우에 필요한 것이 두 개 이상의 객체 타입에 대해 가상 함수처럼 동작하는 함수입니다.

* 하지만 C++에는 이런 기능이 없기 때문에, 이중 디스패치(double-dispatch, 일반화 하여 여러 개의 매개변수에 대해 가상 함수처럼 동작하는 함수를 다중 디스패치(multiple dispatch)라고 한다)라고 알려진 기법을 스스로 구현해야 합니다.

### 가상 함수와 RTTI를 사용하여 구현하는 이중 디스패치
* 가상 함수는 단일 디스패치를 구현한 것이라고 생각할 수 있습니다. 목표가 이중 디스패치이니, 일단 목표의 반은 이룬 셈이죠. GameObject에다 collide라는 가상 함수를 선언하는 것으로 일을 시작합니다.

* 첫 번째로 보여드릴 이중 디스패치 구현 방법은 가장 흔한 방법이면서도 지저분한 방법입니다. if-then-else를 줄줄이 엮어 가상 함수를 흉내내는, 용서가 안 되는 아수라장에 발을 들이는 것이기 때문입니다. 우선 otherObject의 진짜 타입을 알아낸 후에, 그 타입에 대해 생길 수 있는 모든 가능성을 조사합니다.
```c++
// 이 예제에서는 SpaceShip 파생 클래스에 대한
// 코드만 보였는데, 나머지 파생 클래스들도 똑같은 방식으로
// 처리된다고 생각해 두시면 됩니다.
 
class GameObject {
public:
    virtual void collide(GameObject& otherObject) = 0;
    ...
};
 
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
    ...
};
 
// 충돌한 객체의 타입이 전혀 모르는 타입이면,
// 그 객체를 사용해서 만든 예외를 발생시킵니다.
class CollisionWithUnknownObject {
public:
    CollisionWithUnknownObject(GameObject& whatWeHit);
    ...
};
 
void SpaceShip::collide(GameObject& otherObject)
{
    const type_info& objectType = typeid(otherObject);
    
    if (objectType == typeid(SpaceShip)) {
        SpaceShip& ss = static_cast<SpaceShip&>(otherObject);
        
        우주선-우주선 충돌을 처리합니다;
    }
    else if (objectType == typeid(SpaceStation)) {
        SpaceStation& ss =
            static_cast<SpaceStation&>(otherObject);
        
        우주선-우주정거장 충돌을 처리합니다;
    }
    else if (objectType == typeid(Asteroid)) {
        Asteroid& a = static_cast<Asteroid&>(otherObject);
        
        우주선-소행성 충돌을 처리합니다;
    }
    
    else {
        throw CollisionWithUnknownObject(otherObject);
    }
}
 
// SpaceShip의 멤버 함수에서
// *this는 SpaceShip 객체입니다.
// 따라서 otherObject의 진짜(동적)
// 타입만 알아내면 문제가 풀립니다.
```
* 이 코드를 쓸 때부터 이미 우리는 '캡슐화'와는 안녕을 고한 셈입니다. collide 멤버 함수는 무늬만 멤버 함수이지, 해당 클래스의 형제(sibling) 클래스, 즉 GameObject에서 파생된 클래스를 모두 알고 있어야 합니다. 이게 나쁜 이유는 새로운 형제 클래스가 추가되었을때 if-then-else 끄나풀에 새로 추가된 객체 타입을 반영해야 하기 때문입니다.

* 코드를 우겨 넣고 기존의 코드를 많이 바꾸어야 하기 때문에 상당한 타이핑 중노동이 뒤따를 것은 물론이고, 타입 하나를 빠뜨리기라도 하면 버그가 생길 수도 있습니다. 이런 버그는 찾기도 힘들고, 컴파일러가 잡아낼수도 없습니다.

### 가상 함수만 사용하여 구현하는 이중 디스패치
* '가상 함수만 쓰는 방법'은 RTTI 방법과 똑같은 기본 골격으로 시작합니다. collide 함수는 GameObject 클래스에서 가상 함수로 선언되고 각 파생 클래스에서 재정의됩니다. 덧붓여, collide는 각 파생 클래스에서 오버로딩되는데, 이 함수는 클래스 계통 구조의 모든 파생 클래스 타입에 대해 오버로딩되어 있습니다.

* 이 방법의 기본 아이디어는 이중 디스패치를 단일 디스패치 두 개로 구현하자는 것입니다. 즉, 독립된 가상 함수 두 개를 호출하자는 말입니다.


```c++
class SpaceShip;
class SpaceStation;
class Asteroid;
 
class GameObject {
public:
    virtual void collide(GameObject& otherObject) = 0;
    virtual void collide(SpaceShip& otherObject) = 0;
    virtual void collide(SpaceStation& otherObject) = 0;
    virtual void collide(Asteroid& otherObject) = 0;
    ...
};
 
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
    virtual void collide(SpaceShip& otherObject);
    virtual void collide(SpaceStation& otherObject);
    virtual void collide(Asteroid& otherObject);
    ...
};
 
void SpaceShip::collide(GameObject& otherObject)
{
    otherObject.collide(*this);
}
 
// 언뜻 보기에는 매개변수가 뒤바뀐 재귀함수 호출로밖에
// 보이지 않습니다. 그러나 컴파일러는 자신이 호출할
// 함수를 결정할 때 함수에 넘겨진 인자의 정적 타입
// 을 보고 결정합니다.
 
// 이 경우 SpaceShip 클래스의 멤버 함수 안으로
// 들어왔으니, *this의 정적 타입은 SpaceShip입니다.
// 따라서, otherObject에서는 SpaceShip& 타입을
// 받는 collide 함수가 호출되는 것입니다.
// GameObject&를 받는 collide 함수가 아닙니다.
 
void SpaceShip::collide(SpaceShip& otherObject)
{
    우주선-우주선 충돌을 처리합니다;
}
 
void SpaceShip::collide(SpaceStation& otherObject)
{
    우주선-우주 정거장 충돌을 처리합니다;
}
 
void SpaceShip::collide(Asteroid& otherObject)
{
    우주선-소행성 충돌을 처리합니다;
}

```

* 가상 함수만 쓴 방법의 치명적 단점은, 이 방법 역시 모든 클래스가 다른 형제 클래스에 대해 알고 있어야 한다는 전제 조건이 필요하다는 것입니다. 하지만 업데이트 방법이 약간 다릅니다. if-then-else 사슬을 고치는 일은 아니지만, 더 나쁜 영향을 끼칠 수 있다는 것이 문제입니다. 클래스에 새 가상 함수를 정의해 넣어야 하기 때문입니다.

* 기존의 클래스를 고치는 일은 여러분이 손댈 수 없는 부분일 때가 많습니다. GameObject가 라이브러리에 있는 클래스라면, 건드릴 수 없습니다. 설령 여러분이 사용하는 툴킷이 사용자 수정이 가능하게 만들어져 있다 해도, 실제적으로 수정하기는 쉽지 않습니다.

* 예를 들어 개발 팀원들이 GameObject와 기타 자질구레한 클래스로 이루어진 라이브러리를 사용한다고 가정합시다. 이 라이브러리를 사용하는 사람이 한둘이 아니기 때문에, 여러분이 새로운 형태의 객체를 게임 프로그램에 추가할 때마다 이 라이브러리를 사용하는 모든 프로그램을 다시 컴파일해야 할 것입니다.

* 요컨데 이 방법은 RTTI를 쓰는 방법보다는 안전하지만, 헤더 파일을 고치는 등의 시스템 확장에는 제약이 따른다는 단점이 있습니다. 한편, RTTI 방법을 쓰면 이후의 재컴파일 작업이 필요하진 않으나 일반적으로 프로그램의 유지보수가 불가능해집니다.


### 가상 함수 테이블의 유사구현(emulation)을 통한 이중 디스패치
* 이제는 앞의 두 가지 방법을 개선할 수 있는 방법을 찾아보도록 합시다. 컴파일러는 함수 포인터의 배열(vtbl)로 C++의 가상 함수 호출장치를 만들어 놓고, 가상 함수가 호출될 때 그 배열을 인덱싱합니다(항목 24를 보세요).

* 항목 24에서는 힘들다고 설레발을 쳤지만, 가상 함수 메커니즘을 여러분 스스로 구현하지 못할 이유는 없습니다. RTTI 기반의 코드를 더욱 효율적으로 만들고, 한 곳(함수 포인터 배열이 초기화되는 부분)에서만 RTTI를 사용하도록 할 것입니다.

```c++
class GameObject {
public:
    virtual void collide(GameObject& otherObject) = 0;
    ...
};
 
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
    virtual void hitSpaceShip(SpaceShip& otherObject);
    virtual void hitSpaceStation(SpaceStation& otherObject);
    virtual void hitAsteroid(Asteroid& otherObject);
    ...
};
 
void SpaceShip::hitSpaceShip(SpaceShip& otherObject)
{
    우주선-우주선 충돌을 처리합니다;
}
 
void SpaceShip::hitSpaceStation(SpaceStation& otherObject)
{
    우주선-정거장 충돌을 처리합니다;
}
 
void SpaceShip::hitAsteroid(Asteroid& otherObject)
{
    우주선-소행성 충돌을 처리합니다;
}
 
// GameObject 클래스에 충돌 처리용 함수가
// 하나밖에 들어 있지 않습니다. 이 함수는
// 두 번의 함수 디스패치 중 첫 번째
// 디스패치를 맡습니다.
 
// 또한, RTTI 방법 뒤에 나왔던
// 가상 함수 방법과 마찬가지로 충돌하는
// 객체들의 상호작용은 별도의 함수에서
// 처리되는데, 각자 별도의 이름을
// 가지고 있습니다. 오버로딩을 피한
// 이유는 곧 확인하게 될 것입니다.
 
// SpaceShip::collide는 hitXXX 함수
// 들이 호출되는 시작 위치입니다.
 
// SpaceShip::collide 함수 안에는,
// 클래스 이름과, 멤버 함수 포인터를
// 짝으로 묶는 일종의 연관 자료구조를
// 만들어 두는 것이 쉬운 방법일 것
// 같습니다.
 
// 그리고 collide가 이 연관 자료구조에
// 직접 액세스하기보다는, 대신 액세스해주는
// 함수를 하나 만들고 collide가 이 함수를
// 호출하는 편이 이해하기 좋을 것입니다.
 
// 이 함수의 이름을 lookup이라고 합시다.
// 이 함수는 GameObject&를 받아,
// 그 객체의 진짜 타입에 맞는 멤버 함수
// 포인터를 반환합니다.
 
class SpaceShip: public GameObject {
private:
    typedef void (SpaceShip::*HitFunctionPtr)(GameObject&);
    static HitFunctionPtr lookup(const GameObject& whatWeHit);
    ...
};
 
void SpaceShip::collide(GameObject& otherObject)
{
    HitFunctionPtr hfp =
        lookup(otherObject);
    if (hfp) {
        (this->*hfp)(otherObject);
    }
    else {
        throw CollisionWithUnknownObject(otherObject);
    }
}
 
// 이제 lookup이 어떻게 생겨먹었는지만
// 보면 될 것입니다.
 
// lookup에 사용될 연관 자료구조는
// 자신이 사용되기 전에 생성과 초기화가
// 완료되어야 하고, 더 사용되지 않을
// 때 소멸되어야 합니다.
 
// new와 delete를 쓰기보다는,
// 컴파일러가 자동으로 해 주는 것이
// 오류를 막기 쉽습니다.
 
// 그래서 이 연관 자료구조를 lookup의
// 정적 데이터로 선언하는 것입니다.
 
class SpaceShip: public GameObject {
private:
    typedef void (SpaceShip::*HitFunctionPtr)(GameObject&);
    typedef std::map<std::string, HitFunctionPtr> HitMap;
    ...
};
 
SpaceShip::HitFunctionPtr
    SpaceShip::lookup(const GameObject& whatWeHit)
{
    static HitMap collisionMap;        // 이 연관 자료구조(맵)의
                                    // 초기화는 나중에 설명합니다.
 
    // whatWeHit의 타입에 맞는 충돌처리 함수를 찾습니다.
    // 반환값은 "반복자(iterator)"라고 불리는
    // 포인터 비슷한 객체입니다(항목 35를 보세요).
    HitMap::iterator mapEntry =
        collisionMap.find(typeid(whatWeHit).name());
    
    // 탐색이 실패하면 mapEntry == collisionMap.end() 입니다;
    // 이것은 map의 표준적인 동작방식입니다. 역시 항목 35를 참고하십시오.
    if (mapEntry == collisionMap.end()) return 0;
    
    // 이 부분으로 들어왔다면 탐색이 성공한 것입니다.
    // mapEntry는 map에 저장되는 실제 엔트리를 가리키는데,
    // 이 엔트리의 타입은 string과 HitFunctionPtr를 묶은 pair입니다.
    // 지금은 이 페어의 두 번째(second) 부분만 필요합니다.
    return (*mapEntry).second;
}
 
// 이 함수의 마지막 문장에서 맵 엔트리의 둘째 멤버를 반환할 때
// mapEntry->second 대신에 (*mapEntry).second를 쓴 이유는
// 아직 안정화되지 않은 STL을 사용하는 경우를 감안했기 때문입니다.
 
// C++ 표준에는 type_info::name의 반환값을 확실히 지정해 놓지
// 않았기 때문에, 이 함수의 동작은 C++ 컴파일러마다 다르게
// 구현되어 있습니다.
 
// 이 코드를 좀 더 안전하게 설계하려면 type_info 객체의 주소를
// 가지고 클래스를 식별하도록 고치는 것이 좋습니다. 왜냐하면
// type_info 객체의 주소는 유일하기 때문입니다.
 
// 이 경우 HitMap은 map<const type_info*, HitFunctionPtr>으로
// 선언되어야 합니다.
```

### 유사 가상 함수 테이블의 초기화
* 어찌 하다보니 이제 collisionMap의 초기화를 해야하는 때가 되었습니다. collisionMap이 생성될 때 딱 한 번만 초기화를 하려면, 별도의 정적 멤버 함수를 두어 여기서 맵을 생성하고 초기화하도록 하면 됩니다. 그래서 이런 일을 하는 initializeCollisionMap이란 함수를 따로 만들었습니다. 이 함수의 반환값은 collisionMap을 초기화하는 데 쓰입니다.

* 하지만 initializeCollisionMap에서 반환된 객체가 collisionMap으로 복사된다는 점이 마음에 걸립니다(항목 19와 20을 참고하세요). 포인터를 반환하면 복사 비용을 물지 않지만, 적절한 시기를 보아 우리가 직접 소멸시켜야 하는 문제가 있습니다. 이런 고민들은 스마트 포인터(항목 28 참조)를 사용하여 일거에 없앨 수 있습니다. lookup 함수 안에 선언된 collisionMap의 타입을 정적 auto_ptr로 바꾸고, initializeCollisionMap에서는 초기화된 map 객체의 포인터를 반환하게 합시다.

* 아직 문제는 남아 있습니다. collisionMap은, string와 HitFunctionPtr를 묶은 pair인데 HitFunctionPtr은 인자 타입이 GameObject&인 함수 포인터 입니다. 그런데 hitXXX 함수들의 인자 타입은 파생 클래스 타입의 참조자입니다. 따라서 hitXXX함수의 포인터를 collisionMap에 담을 수 없는 문제가 생깁니다.

* 당장에 어떻게든 컴파일이 되게 하려고 reinterpret_cast(항목 2 참조)를 써서 함수 포인터 타입을 캐스팅 하는 방법을 떠올릴 수 있겠으나, 만약에 GameObject의 파생 클래스들이 다중 상속으로 만들어졌다거나 가상 기본 클래스를 가지고 있으면, 함수 포인터를 통해 함수가 호출되는 부분에서 심각한 런타임 에러가 발생할 것입니다. 왜냐하면 다중 상속으로 만들어졌거나 가상 기본 클래스를 가지고 있는 경우, 하나의 객체가 어떤 타입으로 해석되냐에 따라 포인터의 값(주소)이 다를 수 있기 때문입니다(항목 24 참조).

* 이것을 해결할 방법은 단 하나입니다. hitSpaceShip, hitSpaceStation, hitAsteroid 함수가 모두 GameObject를 인자로 받아들이도록 타입을 바꾸는 것입니다. 이번 방법에서 함수 오버로딩을 피했던 이유가 바로 hitXXX 함수가 받는 매개변수 타입을 모두 똑같게(GameObject&) 하기 위해서 였습니다.

* 애석하게도, hitXXX 계열의 모든 함수가 받는 매개변수의 타입이 GameObject라는 일반 타입으로 바뀌었지만, hitXXX 함수는 여전히 자신들이 요구했던 객체 타입을 가지고 동작해야 합니다. 따라서 dynamic_cast의 힘을 빌어와야 하겠습니다.

```c++
//-----------------------------------------------------------------------------
// 잘못 구현된 코드
SpaceShip::HitFunctionPtr
    SpaceShip::lookup(const GameObject& whatWeHit)
{
    static HitMap collisionMap;
    
    // 함수가 호출될 때 마다 초기화 됩니다.
    collisionMap["SpaceShip"] = &hitSpaceShip;
    collisionMap["SpaceStation"] = &hitSpaceStation;
    collisionMap["Asteroid"] = &hitAsteroid;
    ...
}
 
 
//-----------------------------------------------------------------------------
class SpaceShip: public GameObject {
private:
    static HitMap* initializeCollisionMap(void);
    ...
};
 
SpaceShip::HitFunctionPtr
    SpaceShip::lookup(const GameObject& whatWeHit)
{
    static auto_ptr<HitMap> collisionMap(initializeCollisionMap());
    ...
}
 
SpaceShip::HitMap* SpaceShip::initializeCollisionMap(void)
{
    HitMap *phm = new HitMap;
    
    (*phm)["SpaceShip"] = &hitSpaceShip;
    (*phm)["SpaceStation"] = &hitSpaceStation;
    (*phm)["Asteroid"] = &hitAsteroid;
    
    return phm;
}
 
class GameObject {            // 이 부분은 안 바뀝니다.
public:
    virtual void collide(GameObject& otherObject) = 0;
    ...
};
 
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
    
    // 이제 이 함수들은 모두 GameObject 매개변수를 취합니다.
    virtual void hitSpaceShip(GameObject& spaceShip);
    virtual void hitSpaceStation(GameObject& spaceStation);
    virtual void hitAsteroid(GameObject& asteroid);
    ...
};
 
void SpaceShip::hitSpaceShip(GameObject& spaceShip)
{
    SpaceShip& otherShip = dynamic_cast<SpaceShip&>(spaceShip);
    
    우주선-우주선 충돌을 처리합니다;
}
 
void SpaceShip::hitSpaceStation(GameObject& spaceStation)
{
    SpaceStation& station = dynamic_cast<SpaceStation&>(spaceStation);
    
    우주선-우주정거장 충돌을 처리합니다;
}
 
void SpaceShip::hitAsteroid(GameObject& asteroid)
{
    Asteroid& theAsteroid = dynamic_cast<Asteroid&>(asteroid);
    
    우주선-소행성 충돌을 처리합니다;
}
 
// dynamic_cast 연산자는 다운 캐스팅이 실패하면
// bad_cast 예외를 발생합니다. 물론 다운 캐스팅은
// 절대로 실패해선 안 됩니다. 여기서 가정하지 않은
// 매개변수 타입에 대해 hitXXX 함수가 호출되면
// 안 되니까요.
```

### 충돌처리 함수를 클래스 멤버로 넣지 않고 구현하는 이중 디스패치
* 클래스 이름과 함수 포인터를 묶어서 관리하는 연관 자료구조(맵)로 구현한, 유사 가상 함수 테이블이 '멤버 함수에 대한 포인터'를 관리한다는 점은 왠지 모를 껄끄러움을 우리에게 선사하고 있습니다. 왜냐하면, GameObject의 새로운 파생 클래스가 게임에 추가되기라도 하면, 여전히 클래스 정의를 수정해야 하기 때문입니다. 이렇게 되면 새로운 클래스에 대해 관심이 없는 개발자라도 재컴파일을 피할 수 없습니다. 이 문제는 가상 함수만 써서 이중 디스패치를 구현하는 방법의 문제와 똑같습니다.

* 이것을 해결하기 위해 충돌처리 함수를 GameObject 클래스 계통 구조에서 분리하고, hitXXX 함수나 collide 함수를 비멤버 함수로 만드는 방법을 사용할 수 있습니다.
```c++
#include "SpaceShip.h"
#include "SpaceStation.h"
#include "Asteroid.h"
 
namespace {                // 이름 없는 네임스페이스 - 다음을 보세요.
                        // 기본적인 충돌 처리 함수
    void shipAsteroid(GameObject& spaceShip,
                    GameObject& asteroid);
 
    void shipStation(GameObject& spaceShip,
                    GameObject& spaceStation);
    
    void asteroidStation(GameObject& asteroid,
                    GameObject& spaceStation);
    ...
    // 매개변수 타입이 뒤바뀌었을 때를 감안하여 만든
    // 부수의 충돌처리 함수. 매개변수를 다시 뒤바꾸어
    // 기본 충돌처리 함수를 호출합니다.
    void asteroidShip(GameObject& asteroid,
                    GameObject& spaceShip)
    { shipAsteroid(spaceShip, asteroid); }
    
    void stationShip(GameObject& spaceStation,
                    GameObject& spaceShip)
    { shipStation(spaceShip, spaceStation); }
    
    void stationAsteroid(GameObject& spaceStation,
                    GameObject& asteroid)
    { asteroidStation(asteroid, spaceStation); }
    ...
    // 이 타입과 함수에 대한 설명은 다음에서 읽을 수 있습니다.
    typedef void (*HitFunctionPtr)(GameObject&, GameObject&);
    typedef map<pair<string, string>, HitFunctionPtr> HitMap;
    
    pair<string, string> makeStringPair(const char *s1,
                                    const char *s2);
    
    HitMap * initializeCollisionMap(void);
    
    HitFunctionPtr lookup(const string& class1,
                        const string& class2);
}    // 네임스페이스 끝
 
void processCollision(GameObject& object1,
                    GameObject& object2)
{
    HitFunctionPtr phf = lookup(typeid(object1).name(),
                                typeid(object2).name());
    
    if (phf) phf(object1, object2);
    else throw UnknownCollision(object1, object2);
}
 
// 이름 없는 네임스페이스의 안에 있는
// 모든 것들은 현재의 컴파일 단위
// (즉, 현재의 파일)에서만 의미를
// 가지고, 외부에서는 볼 수 없게
// 됩니다.
 
// 말하자면, 함수 앞에 static을 붙여
// 정적 함수로 선언한 것과 똑같은
// 효과를 갖는 것이죠.
 
// 네임스페이스가 도입되면서 파일 범위
// 에서의 정적 변수/함수 선언은
// 폐기되었습니다.
 
// 기존과 달라진 점이 몇 가지 있습니다.
 
// 첫째, HitFunctionPtr이 비멤버 함수
// 포인터의 typedef로 바뀌었습니다.
 
// 둘째, 예외 클래스인 CollisionWithUnknownObject가
// UnknownCollision으로 이름이 바뀌고,
// 두 개의 객체를 받아들이게 되었습니다.
 
// 마지막으로, lookup 함수는 이제
// 두 개의 타입 이름을 받아 두 단계의
// 이중 디스패치를 한꺼번에 수행합니다.
// 즉, 이제 연관 자료구조에서 세 개의
// 정보(두 개의 타입 이름과 한 개의
// HitFunctionPtr)를 관리해야 합니다.
 
// 이 함수는 char* 리터럴을 받아 pair<string, string>
// 을 만드는데 사용하고, 실제로 initializeCollisionMap에서
// 사용됩니다. 이 함수에서 어떻게 반환값 초기화
// (RVO, 항목 20을 보세요)를 가능하게 했는지 눈여겨 보세요.
 
namespace {                // 이름 없는 네임스페이스 재출현 - 다음을 보세요.
    pair<string, string> makeStringPair(const char *s1,
                                        const char *s2)
    { return pair<string, string>(s1, s2); }
}    // 네임스페이스의 끝
 
namespace {                // 이름 없는 네임스페이스 거듭 출현 - 다음을 보세요.
    HitMap *initializeCollisionMap(void)
    {
        HitMap *phm = new HitMap;
        (*phm)[makeStringPair("SpaceShip", "Asteroid")] =
            &shipAsteroid;
        
        (*phm)[makeStringPair("SpaceShip", "SpaceStation")] =
            &shipStation;
        ...
        return phm;
    }
}    // 네임스페이스의 끝
 
// lookup 함수 역시 바뀌어야 합니다.
namespace {                // 맹세코 다음에서 설명합니다. - 이 사람 믿어 주세요.
    HitFunctionPtr lookup(const string& class1,
                        const string& class2)
    {
        static auto_ptr<HitMap>
            collisionMap(initializeCollisionMap());
        
        // make_pair의 설명은 다음에서 확인하세요.
        HitMap::iterator mapEntry =
            collisionMap->find(make_pair(class1, class2));
        
        if (mapEntry == collisionMap->end()) return 0;
        return (*mapEntry).second;
    }
}    // 네임스페이스의 끝
 
// make_pair는 페어 객체를 생성할 때
// 타입을 일일히 지정해 줄 필요가 없도록
// 만들어진 함수(템플릿)로서, 표준 라이브러리
// (항목 35 참조)에 들어 있습니다.
 
// 즉 다음 두 문장은 똑같은 역할을
// 수행합니다.
 
HitMap::iterator mapEntry =
    collisionMap->find(make_pair(class1, class2));
 
HitMap::iterator mapEntry =
    collisionMap->find(pair<string, string>(class1, class2));
 
// makeStringPair, initializeCollisionMap, lookup 함수는
// 모두 이름 없는 네임스페이스 안에 선언되어 있습니다.
// 그렇기 때문에 이 함수들은 동일한 네임스페이스
// 안에서 구현(정의)되어야 합니다.
 
// 앞의 함수들에 대한 구현 코드가 이름 없는
// 네임스페이스(즉, 함수가 선언된 컴파일 단위(파일)
// 와 똑같은 컴파일 단위에 대해)에 들어 있는
// 이유도 그 때문입니다.
```

* 이제는 GameObject에서 새로 파생된 클래스가 클래스 계통 구조에 추가되더라도 이미 만들어진 클래스를 다시 컴파일할 필요가 없습니다. initializeCollisionMap에서 맵 정보를 추가하고, processCollision이 구현된 파일에다 추가된 클래스에 대한 충돌처리 함수를 새로 만들어 넣기만 하면 됩니다.

### 클래스 상속이 유사 가상 함수 테이블에 끼치는 영향
* 아직도 해결해야 할 마지막 문제가 남아 있습니다. 우리가 만든 이중 디스패치 장치는 충돌 처리 함수를 호출할 때 상속 기반의 타입변환을 허용하지 않으면 문제 없이 잘 동작합니다.

* 예를 들어 볼까요? 우주선(SpaceShip)을 상업용 우주선(CommercialShip)과 군사용 우주선(MilitaryShip)으로 구분할 필요가 생겼습니다. 항목 33의 가이드라인을 준수해서 SpaceShip을 추상 클래스로 바꾸고 CommercialShip과 MilitaryShip이 이 SpaceShip 클래스에서 파생되도록 했습니다.

* 상업용 우주선과 군사용 우주선은 충돌할 때의 행동패턴이 똑같다고 가정합시다. 이렇게 하면, 이 두 클래스에 대한 충돌처리 함수로는 이전의 SpaceShip에 썼던 그 함수가 쓰일 수 있을 것 같습니다. 이를테면, MilitaryShip 객체와 Asteroid 객체가 충돌할 때,

```c++
void shipAsteroid(
                GameObject& spaceShip,
                GameObject& asteroid);
```
* 이 함수가 호출될 것으로 생각할 수 있다는 것이죠. 하지만, 이 함수는 호출되지 않고 UnknownCollision 예외가 발생됩니다.

* 이유는 이렇습니다. lookup함수에서는 "MilitaryShip"과 "Asteroid"란 타입 이름에 대응되는 함수를 찾아야 하는데, 불행하게도 "MilitaryShip"은 collisionMap의 엔트리로 입력되어 있지 않습니다. MilitaryShip이 SpaceShip와 같은 종류로 취급될 수 있다고 하지만, lookup은 그것을 알 턱이 없습니다.

* 게다가 lookup에게 그것을 알려 주는 것이 쉬운 것도 아닙니다. 그래서 이중 디스패치를 구현해야 하고 이런 식의 상속 기반 매개변수 변환을 지원해야 한다면, 앞에서 공부했던 이중 가상 함수 호출 메커니즘을 이용해야 합니다. 물론 이 방법을 쓰면 클래스 상속 계통 구조가 바뀔 때마다 프로그램을 재컴파일하는 고통을 감내하셔야 합니다.

### 유사 가상 함수 테이블의 초기화(제 2탄)
* collisionMap을 초기화 하는 방법에 대해서 조금만 더 말씀드리려고 합니다. 이번 항목에서 설계한 유사 가상 함수 테이블 메커니즘은 모든 것이 정적(static)입니다. 하지만 게임이 진행되면서 충돌처리 함수가 추가될 수도 있고, 제거될 수도 있고, 바뀔 수도 있는 것 아니겠습니까? 지금의 유사 가상 함수 테이블엔 그런 기능이 없습니다.

* 그런 기능이 없는 것은 만들지 않았기 때문입니다. 단순히 충돌처리 함수를 저장하는 맵의 개념을, 동적으로 맵의 내용을 수정하는 멤버 함수를 제공하는 클래스로 바꾸어 해결할 수 있습니다.

```c++
class CollisionMap {
public:
    typedef void (*HitFunctionPtr)(GameObject&, GameObject&);
    
    void addEntry(const string& type1,
                const string& type2,
                HitFunctionPtr collisionFunction,
                bool symmetric = true);        // 다음을 보세요.
    
    void removeEntry(const string& type1,
                    const string& type2);
    
    HitFunctionPtr lookup(const string& type1,
                        const string& type2);
    
    // 이 함수는 유일한 단 하나의 맵에 대한 참조자를
    // 반환합니다. 항목 26을 참고하세요.
    static CollisionMap& theCollisionMap(void);
    
private:
    // 여러 개의 맵이 생기지 않도록 이 함수를 private 영역에
    // 넣습니다. 역시 항목 26을 참고하세요.
    CollisionMap(void);
    CollisionMap(const CollisionMap&);
};
 
// 현재의 시스템에는 맵이 하나만 있기 때문에
// CollisionMap 객체의 개수를 하나로 제한했습니다.
 
// 마지막으로, addEntry가 호출될 때 넘겨지는
// symmetric 인자를 통해, 타입이 뒤바뀐
// 맵 엔트리를 자동으로 추가할 수 있는 옵션을
// 만들었습니다.
 
// 이제는 이렇게 맵 엔트리를 추가할 수 있게 됩니다.
 
void shipAsteroid(GameObject& spaceShip,
                GameObject& asteroid);
CollisionMap::theCollisionMap().addEntry("SpaceShip",
                                        "Asteroid",
                                        &shipAsteroid);
 
void shipStation(GameObject& spaceShip,
                GameObject& spaceStation);
CollisionMap::theCollisionMap().addEntry("SpaceShip",
                                        "SpaceStation",
                                        &shipStation);
 
void asteroidStation(GameObject& asteroid,
                    GameObject& spaceStation);
CollisionMap::theCollisionMap().addEntry("Asteroid",
                                        "SpaceStation",
                                        &asteroidStation);
...
 
// 여기서 주의할 부분은, 어떤 충돌이 일어나
// 충돌처리 함수를 호출하는 일이 벌어지기 전에
// 맵 엔트리를 모두 추가할 수 있도록 해야
// 한다는 것입니다.
 
// 조금 괜찮은 방법으로 '타입 이름 - 충돌 함수'
// 매핑을 담당하는 장치를 따로 만드는 것입니다.
 
class RegisterCollisionFunction {
public:
    RegisterCollisionFunction(
        const string& type1,
        const string& type2,
        CollisionMap::HitFunctionPtr collisionFunction,
        bool symmetric = true)
    {
        CollisionMap::theCollisionMap().addEntry(type1, type2,
                                                collisionFunction,
                                                symmetric);
    }
};
 
// 이 타입의 객체는 전역 객체로 띄워 사용합니다.
// 이 클래스의 객체가 전역 메모리에 올라오는 동시에
// 필요한 함수가 자동으로 등록됩니다.
 
RegisterCollisionFunction cf1("SpaceShip", "Asteroid",
                            &shipAsteroid);
 
RegisterCollisionFunction cf2("SpaceShip", "SpaceStation",
                            &shipStation);
 
RegisterCollisionFunction cf3("Asteroid", "SpaceStation",
                            &asteroidStation);
...
int main(int argc, char * argv[])
{
    ...
}
 
// 만약, 이후에 어떤 클래스가 새로 추가되었다고 하면,
// 그리고 이 클래스에 대한 충돌처리 함수가 새로 만들어졌다고 하면,
 
class Satellite: public GameObject { ... };
 
void satelliteShip(GameObject& satellite,
                    GameObject& spaceShip);
 
void satelliteAsteroid(GameObject& satellite,
                    GameObject& asteroid);
 
// 기존의 코드를 흔들지 않은 채로, 비슷하게
// 새 함수를 맵에 추가하면 됩니다.
 
RegisterCollisionFunction cf4("Satellite", "SpaceShip",
                            &satelliteShip);
RegisterCollisionFunction cf5("Satellite", "Asteroid",
                            &satelliteAsteroid);
 
// 프로그래밍이 조금 더 세련되어 지긴 했지만
// '다중 디스패치를 완벽하게 구현하는 방법은 없다'
// 라는 사실은 바뀌지 않습니다.

```




# Chpater 6 이외의 이야기들(Miscellany)

## 항목 32 : 미래 지향적인 프로그래머가 되자

* 미래 지향적인 개발자가 변화에 잘 적응하는 훌륭한 소프트웨어를 만듭니다. 
* 미래 지향적 프로그래밍이란, 변화를 받아들이고 변화에 대비하는 것입니다. 
    * 사용하는 라이브러리에 함수가 새로추가 될수 있으며 오버로딩도 추가로 이루어 질수 있습니다. 
    * 소프트웨어를 만든 사람이 유지보수를 맡지 않을 경우가 다반사입니다.
        * 다른 사람이 이해하고, 수정하고, 개선하는데 큰 문제를 일으키지 않는 방법으로 설계하고 구협합시다.
* 이렇게 하는 한 가지 방법은 설계 제약을 나타내는데 있어 주석이나 문서화 대신 C++의 특징을 이용하는 것입니다.
    * 예를 들어, 어떤 클래스가 (설계적으로) 절대로 파생되지 않는 클래스라면 그 클래스에 대한 헤더 파일에는 주석문을 넣지 말고 C++의 특성을 써서 아예 파생을 못하게 막아버리세요(항목 26 참조).
    * 어떤 클래스는 반드시 힙에 인스턴스가 생성되어야 한다면, 사용자에게 문서로 알리지 말고 진짜로 힙에만 만들어지도록 하는 장치를 만들어 두라 이겁니다(항목 27 참조).
    * 복사나 대입이 불필요한 클래스의 경우에는, 복사 생성자와 대입 연산자를 private로 선언하여 복사나 대입을 막습니다.
    * C++는 엄청난 강력함과 유연성, 표현력을 가지고 있습니다. C++의 언어적 특성을 통해 여러분의 설계 의도를 반영에 두는 것이 최상입니다.
* "사용해 줄 사람을 기다리는(demand-paged)" 가상 함수를 만들지 마십시오. 누군가 와서 그렇게 하라고 하기 전에는 멀쩡한 함수를 가상 함수로 만들지 말라는 뜻 입니다.
    * 어떤 함수를 만들 때마다 그 함수가 파생 클래스에서 재정의될 가치가 있는지를 결정하도록 하세요. 확신이 선다면, 아무도 그 함수를 재정의하지 않더라도 그 함수를 가상 함수로 만들어도 됩니다. 확신이 서지 않으면, 그 함수를 비가상 함수로 선언하고, 그 이후에는 가상 함수로 만들라는 잔소리가 들린다고 해서 바꾸면 안됩니다.
    * 항상 그 변화는 전체 클래스와 그 클래스가 나타내는 추상화의 한도 내에서 의미를 갖도록 하십시오.
* 대입과 복사 생성은 모든 클래스에 대해 처리해 두어야 합니다. 복사나 생성을 사용하는 사람이 없을 것이라는 생각이 들어도 그렇게 해야 합니다. 지금 사용하지 않는다는 것은 나중에도 사용하지 않는다는 뜻이 아니니까요.
    * 복사나 생성 처리가 까다로운 경우에는 복사 생성자와 대입 연산자를 private로 선언해 두기 바랍니다. 컴파일러가 어쩌다가 만들어 놓은 코드 때문에 오작동이 일어나는 불상사를 막을 수 있습니다.
* C++ 프로그래밍에서 어기면 안 되는 원칙이 있습니다. 바로 "황당함 최소화의 원칙(the principle of least astonishment)" 이란 것입니다.
    * 어떤 클래스에 대해 만들어지는 연산자와 함수는, 다른 사람들이 자연스럽게 사용할 수 있는 문법과 직관적인 의미구조를 갖도록 하라는 것이죠. C++의 기본 제공 타입과 동일한 동작원리를 갖도록 하세요. '아리송하면 int의 동작원리대로 만들지어다(when in doubt, do as the ints do)' 아시죠?
* 누구라고 일으킬 수 있는 일은 반드시 터지고야 만다는 사실도 잊어선 안 됩니다.
    * 일반적으로 컴파일러가 별 불평을 하지 않는 동작은 누군가가 반드시 하게 된다고 생각하는 것이 좋습니다. 따라서 C++ 클래스를 만들 때에는, 맞게 사용하기에는 쉽고 틀리게 사용하기에는 어렵도록 클래스를 만들어야 합니다.
    * 그리고 이런 실수를 방지하고, 알아내고, 고칠 수 있도록 여러분의 클래스를 설계해야 합니다(항목 33에서 이야기한 내용이 그런 예입니다).
* 코드의 이식성도 최대한 신경 쓰셔야 하는 부분입니다.
    * 이식이 안 되는 코드보다 이식이 잘 되는 코드를 작성하기가 훨씬 어렵습니다. 그리고 이식이 안 되는 코드를 꼭 써야 할 정도로 수행 성능의 차이가 현격하게 나는 경우도 드뭅니다(항목 16을 보세요).
* 변경이 필요할 때 그 변경의 영향이 제한된 부분에만 미치도록 코드를 설계하기 바랍니다. 여러분이 힘닿는 데까지 캡슐화를 하고, 구현에 관련된 상세한 부분은 외부에 노출시키지 마세요.
    * 사용하는 개발 도구가 지원하는 기능에 맞추어 이름 없는 네임스페이스(unnamed namespace)를 사용하든지, 파일 범위(file-scope)의 정적 변수(객체)나 정적 함수를 선언하도록 하세요(항목 31에서 참고할 수 있습니다).
    * 가상 기본 클래스가 필요한 설계는 피하는 것이 좋습니다. if-then-else 문을 줄줄이 늘어놓는 RTTI 기반의 설계(역시 항목 31에서 참고할 수 있습니다)도 피하는 것이 좋습니다.

* 아래의 조언들을 잊지 말자
* B* 가 실제로 D를 가리키는 경우에만 B에 대한 가상 소멸자가 필요합니다.
```c++
class B{...};//가상 소멸자 필요없음
class D:public B{...};
B *pb = new D;
//하지만 다음의 경우엔 상황이 바뀜
delete pb;//이제 B에 가상 소멸자를 두어야 합니다.
```
* 기본 클래스에 가상 소멸자가 들어 있지 않으면 파생 클래스나 파생 클래스의 멤버에는 (가상) 소멸자가 들어가선 안됩니다. 
```c++
class string{
public:
    ~string();
}

class B{...};//B엔 가상 소멸자도 필요없고, 소멸자를 가진 멤버로 필요없음
//이 상황에서 B에서 어떤 클래스가 파생되어 새로 만들어지면 이야기가 달라짐
class D: public B{
    string name;//이제 ~B는 가상 소멸자이어야 합니다. 
};
```
* 다중 상속이 이루어진 클래스 계통구조에 소멸자가 한 곳에라도 끼어 있으면, 모든 기본 클래스에는 가상 소멸자가 들어가야 합니다.


* 클래스는 완벽하게 만듭니다. 어떤 부분은 지금 사용되지 않아도 완벽하게 만들어야 합니다.
    * 그래야 나중에 새로운 기능이 필요해질 때 그 클래스로 되돌아가서 근본부터 수정하는 일이 줄어듭니다.
* 인터페이스를 설계할 때에는 자주 사용하는 기능은 모두 포함하고, 자주 일어나는 실수는 막아 둡니다. 맞게 사용하기에는 쉽고, 틀리게 사용하기에는 어렵도록 클래스를 만들어야 합니다.
    * 이를테면, 복사나 대입이 필요 없는 경우에는 복사나 생성이 일어나지 않도록 합니다. 부분 대입(partial assignment, 항목 33 참조) 현상은 반드시 막아야 합니다.
* 코드를 일반화하더라도 별 손실이 없는 경우라면, 코드는 일반화합니다.
    * 예를 들어, 트리 횡단(tree traversal)에 필요한 알고리듬은 방향성 비순환 그래프(directed acyclic graph)에 쓸 수 있도록 일반화하도록 하십시오.
* 미래 지향적인 사고도 중요하지만, 그렇다고 해서 현재 지향적인 사고가 엉터리라는 말은 아닙니다.
    * 여러분이 개발하는 소프트웨어는 지금(현재) 사용하는 컴파일러로 만들어질 수밖에 없습니다. 컴파일러에 최신의 기능이 구현될 때까지 기다릴 수는 없으니 현재 구현된 기능만으로 개발해야 합니다.
    * 소프트웨어는 여러분이 지금 지원하는 하드웨어에서 동작하도록 해야 하고, 사용자들이 쓰고 있을 플랫폼도 고려해야 합니다. 수행 속도도 웬만큼 나와 주어야 합니다.
    * 또, 지금 신명나게 개발하고 있는 소프트웨어는 '곧 출시됨(soon)'이어야 합니다. 언젠가는 '곧 출시됨(soon)'이 최근의 과거를 뜻하는 경우가 많다는 사실도 주의하십시오.
    * 미래 지향적인 사고도 중요하지만, 현재를 무시해선 안 됩니다. 미래 지향적 프로그램은 현재의 제약과 보조를 맞출 때 비로소 제 힘을 발휘할 수 있습니다.



## 항목 33 : 상속 관계의 말단에 있지 않은 (non-leaf) 클래스는 반드시 추상 클래스로 만들자

* 부분 대입(partial assignment) 현상
* 동물(animal) 데이터를 취급하는 소프트웨어를 개발하는 프로젝트를 수행하고 있다고 가정합시다. 다른 동물들은 똑같이 처리하지만 유독 도마뱀(lizard)과 닭(chicken)에 대해서만 특별한 처리가 필요하게 되었습니다. 따라서, 일반적인 동물을 상속하여 닭과 도마뱀을 구현하였습니다. 이들 세 클래스(Animal, Lizard, Chicken)에는 각각 대입 연산자가 정의되어 있습니다. 이런 상황에서 다음과 같은 코드를 실행한다고 생각해 봅시다.
```c++
Lizard liz1;
Lizard liz2;

Animal *pAnimal1 = &liz1;
Animal *pAnimal2 = &liz2;
...
*pAnimal1 = *pAnimal2;
```
* 여기엔 두 가지 문제가 잠자고 있습니다. 우선, 마지막 줄에서 호출되는 대입 연산자는 Animal 클래스의 것입니다. 이렇기 때문에 liz1에 들어 있는 멤버 전체가 바뀌지 않고 Animal의 멤버인 부분만 바뀌고 맙니다. 이것을 가리켜 부분 대입 현상이라고 합니다. 물론 이런 현상이 일어나도록 두면 안 됩니다.

* 두 번째 문제는 더욱 심각합니다. 바로, 프로그래머들이 진짜로 이렇게 코딩한다는 사실입니다. 항목 32에서 이야기 한 것처럼 클래스 설계는 '맞게 사용하기엔 쉽고, 틀리게 사용하기엔 어렵게' 해야 하지만, 지금 우리가 보고 있는 클래스는 틀리게 사용하기가 쉬워 보이네요.


* 대입 연산자를 가상 함수로 바꾸면? - 타입 불일치 대입(mixed-type assignment) 현상
* C++ 표준화 위원회에 의해 최근에 변경된 사항 덕택에 대입 연산자의 반환값 타입은 파생 클래스의 참조자로 맞추어 줄 수 있게 되었지만, 가상 함수의 매개변수 타입만은 기본 클래스의 것을 그대로 쓸 수밖에 없습니다.
즉, Lizard와 Chicken 클래스의 대입 연산자에 넘겨질 수 있는 객체는 Animal 타입이 될 수 있는 것이면 아무거나 다 된다는 뜻입니다. 다음과 같이 말이죠.
```c++
Lizard liz;
Chicken chick;

Animal *pAnimal1 = &liz;
Animal *pAnimal2 = &chick;
...
*pAnimal1 = *pAnimal2;
// 도마뱀에다가
// 닭을 대입하려 하다니요!
```
* 이것을 가리켜 타입 불일치 대입 현상이라고 합니다. 타입 불일치 대입은 C++에서 그다지 큰 문제는 아닙니다. 왜냐하면 C++는 타입 제약이 엄격해서 컴파일 단계에서 이런 경우를 잡아내기 때문입니다. 그런데 어떤 C++ 클래스의 대입 연산자가 가상 함수로 만들어지면 컴파일러가 이런 경우를 잡아내지 못하게 됩니다.
* dynamic_cast(항목 2 참조)를 쓴다면?
* 일단, dynamic_cast를 쓰면 소기의 목적은 달성할 수 있습니다. 예를 들어 Lizard 클래스의 대입 연산자에서 const Animal& 타입으로 받은 인자를 dynamic_cast를 사용해서 const Lizard& 타입으로 캐스팅 한 뒤 대입하는 것입니다. 만일 인자가 실제로 Lizard 타입이 아니었다면 std::bad_cast 예외가 던져질 것입니다.

```c++
Lizard& Lizard::operator =(const Animal& rhs)
{
    // rhs가 진짜로 Lizard 타입인지 확인합니다.
    const Lizard& rhs_liz = dynamic_cast<const Lizard&>(rhs);
    
    rhs를 *this에 대입합니다;
}
 
// 진짜 Lizard 객체 사이에 대입이
// 수행되는 경우가 훨씬 흔하지만,
// 어떤 대입이든 타입 점검을 해야
// 하니 비용이 많이 듭니다.
 
// 이런 경우에는 필요 없는 비용이
// 들지 않도록 Lizard 클래스에
// 그 클래스만의 대입 연산자를
// 추가하면 간단히 해결됩니다.
 
class Lizard: public Animal {
public:
    virtual Lizard& operator =(const Animal& rhs);
    
    Lizard& operator =(const Lizard& rhs);        // 이것을 추가합니다.
    ...
};
 
Lizard liz1, liz2;
...
liz1 = liz2;                // const Lizard&를 받는
                            // operator =를 호출합니다.
Animal *pAnimal1 = &liz1;
Animal *pAnimal2 = &liz2;
...
*pAnimal1 = *pAnimal2;        // const Animal&를 받는
                            // operator =를 호출합니다.
 
// 사실, 새로 추가된 operator =를 사용하면
// 첫 번째 대입 연산자도 간단히 구현할 수 있습니다.
 
Lizard& Lizard::operator =(const Animal& rhs)
{
    return operator =(dynamic_cast<const Lizard&>(rhs));
}
```

* 하지만 가뜩이나 런타임에 할 일도 많은데, 대입 연산자에까지 dynamic_cast로 타입을 비교하고 어쩌고 하는 것은 그다지 마음에 들지는 않습니다.

* 게다가, 이런 Lizard와 Chicken 클래스를 가지고 프로그래밍하는 개발자는 항상 bad_cast에 대한 대비를 해야 합니다. 대입 연산자가 등장하는 부분마다 catch 문이 붙어야 한다니, 정상적인 사고를 지닌 C++ 프로그래머라면 쌍수를 들고 울 일입니다.

* 행여나 예외 처리를 하지 않을 수도 있잖아요? 부분 대입을 막으려고 애썼던 것들이 모두 물거품이 될지도 모릅니다.


* 이 문제는 조금 다른 시각에서 볼 필요가 있습니다. 개발자로 하여금 애초부터 이상한 대입을 하지 못하게 할 방법을 찾아야 합니다. 가장 쉬운 방법은 Animal::operator = 함수를 private 영역에 두는 것입니다.
```c++
class Animal {
private:
    Animal& operator =(const Animal& rhs);        // 이 함수는 이제
    ...                                            // private 멤버가 되었습니다.
};
 
class Lizard: public Animal {
public:
    Lizard& operator =(const Lizard& rhs);
    ...
};
 
class Chicken: public Animal {
public:
    Chicken& operator =(const Chicken& rhs);
    ...
};
 
Lizard liz1, liz2;
...
liz1 = liz2;                // 문제없습니다.
 
Chicken chick1, chick2;
...
chick1 = chick2;            // 문제없습니다.
 
Animal *pAnimal1 = &liz1;
Animal *pAnimal2 = &chick1;
...
*pAnimal1 = *pAnimal2;        // 에러입니다! private 멤버인
                            // Animal::operator = 를 호출하려고 합니다.

```
* 애석하게도 Animal은 구체 클래스(concrete class)이기 때문에, 이 방법 역시 다음의 문장이 컴파일 되지 않는다는 허점을 가지고 있습니다.
```c++
Animal animal1, animal2;
...
animal1 = animal2;
// 에러입니다. private 멤버인
// Animal::operator =를
// 호출하려고 합니다.
```
* 게다가, 이 방법을 사용하면 Lizard와 Chicken의 대입 연산자를 맞게 구현할 수도 없습니다. 왜냐하면, 파생 클래스의 대입 연산자는 기본 클래스의 대입 연산자를 호출해서 기본 클래스 부분의 대입 연산을 대신 맡겨야 하는 의무가 있기 때문입니다.

* 후자의 문제야 Animal::operator = 함수를 protected 멤버로 선언하면 해결되지만, Animal 객체끼리 대입 연산이 되지 않는 문제는 여전히 남아 있습니다.

* Animal 객체끼리 대입할 필요를 만들지 않는 건 어떨까요? 가장 쉬운 일이 아닐까 생각 됩니다. 그리고 'Animal 객체끼리 대입할 필요를 만들지 않으려면' Animal을 추상 클래스로 만드는 것이 가장 쉽습니다.
* Animal은 객체로 써먹으려고 만든 클래스인데, 졸지에 추상 클래스가 되어 버리면 애초의 설계가 엉망이 되고 만다는 문제가 있습니다. 하지만 이런 것쯤은 간단히 피해갈 수 있습니다.

* Animal을 추상 클래스로 만들지 말고 새로운 추상 클래스(AbstractAnimal)를 하나 만들고, Animal, Lizard, Chicken이 갖게 되는 공통적인 특징을 모두 갖게 합니다. 그리고, 세 개의 구체 클래스는 AbstractAnimal을 상속하도록 구성 합니다.
```c++
class AbstractAnimal {
protected:
    AbstractAnimal& operator =(const AbstractAnimal& rhs);
    
public:
    virtual ~AbstractAnimal(void) = 0;        // 다음을 보세요.
    ...
};
 
class Animal: public AbstractAnimal {
public:
    Animal& operator =(const Animal& rhs);
    ...
};
 
class Lizard: public AbstractAnimal {
public:
    Lizard& operator =(const Lizard& rhs);
    ...
};
 
class Chicken: public AbstractAnimal {
public:
    Chicken& operator =(const Chicken& rhs);
    ...
};
 
// 이 설계라면 우리가 바라던 바는
// 모두 이루어질 것 같습니다.
 
// 같은 타입끼리의 대입 연산은
// 도마뱀과 닭은 물론 동물에 대해서도
// 가능해지고, 부분 대입이나
// 다른 타입끼리의 대입은 막힙니다.
 
// 그리고 파생 클래스의 대입 연산자는
// 기본 클래스의 대입 연산자를 문제 없이
// 호출할 수 있게 됩니다.
 
// 게다가, Animal, Lizard, 혹은 Chicken
// 클래스에 사용된 코드는 전혀
// 고칠 필요가 없습니다(물론
// 재컴파일은 필요합니다).
 
 
// AbstractAnimal이 추상 클래스가 되기
// 위해서는 순수 가상 함수를 최소한
// 한 개 가져야 합니다.
 
// 순수 가상 함수로 만들만한
// 멤버 함수가 하나도 없는 드문
// 경우에는 보통 소멸자를
// 순수 가상 함수로 선언해서
// 해결합니다.
 
// 소멸자는 순수 가상함수이더라도
// 정의를 가져야 합니다.
 
// 왜냐하면, 파생 클래스의 소멸자가
// 호출될 때 결국 이 함수도
// 호출되기 때문입니다.
```

* 사실 지금까지 한 이야기는 구체(concrete) 기본 클래스가 데이터 멤버를 가지고 있을 때를 가정하고 한 이야기 입니다. 기본 클래스에 데이터 멤버가 없을 경우라면, 지금까지 한 이야기는 전부 아무 문제도 아닙니다. 데이터가 없는 구체 클래스로부터 구체 클래스를 파생시키는 것은 몇 백 번을 하더라도 안전합니다.
* 구체 기본 클래스에 데이터가 없을 수 있는 상황은 두 가지 중 하나입니다. 이후에 데이터를 가질 수 있으나 지금 안 가진 경우와 진짜로 데이터가 없는 경우이죠.

* 첫째 경우에 여러분은 데이터 멤버가 추가되는 미래를 신경 쓰지 않고 문제들을 덮어두기만 하면 됩니다. 미래의 머리아픔 대신에 현재의 편리함을 택하자는 것입니다.

* 두 번째 경우는 왠지 구체 클래스가 있을 자리에 추상 클래스가 있어야 할 것 같은 느낌을 줍니다. 데이터가 없는 구체 클래스라니, 어디에 써먹는단 말입니까?

* Animal같은 구체 기본 클래스 대신에 AbstractAnimal같은 추상 기본 클래스로 바꾸면, 객체의 배열을 다형적으로 조작하는 실수를 할 가능성도 적어집니다(항목 3 참조).
* 하지만 무엇보다 중요한 이득은 프로그램 설계 쪽에서 찾을 수 있습니다. 구체 기본 클래스를 추상 기본 클래스로 바꾸면, '뭔가 쓸모 있는' 추상 타입이 존재한다는 것을 명시적으로 알리는 셈이 된다는 것입니다.

* 즉, 프로그래머는 이 사실을 자연스럽게 알게 되기 때문에 프로그램 제작에 도움이 되는 그 개념을 추상화한 클래스를 만들어 사용할 수 있습니다. 그런 개념이 있다는 사실을 모를지라도 말이죠.

* 구체 기본 클래스에서 추상 기본 클래스로 변환하는 데에는 확실한 명분이 있습니다. 기존의 구체 클래스를 기본 클래스로 사용해야 할 때에만, 즉 이 클래스가 다른 경우에서 재사용되어야 할 때에만 추상 클래스가 새로 도입되어야 한다는 것입니다.


* 이번 항목에서 보여드린 클래스 계통 구조를 변환하는 과정은 추상 클래스가 필요한 경우를 찾아내는 유일한 방법이 아닙니다. 이외에도 방법은 아주 많으니, 이런 부분에 대해서는 객체지향 분석에 관련된 책을 뒤져서 필요한 만큼 공부하시면 되겠습니다.

* 외부 라이브러리에 속해 있는 클래스의 기능을 끌어온 파생 클래스를 만들어 사용하고 싶은 경우도 많을 것입니다.
* 알다시피 라이브러리는 바이너리이기 때문에, 추상 클래스를 새로 만들어 끼워 넣는 것은 불가능합니다. 따라서 선택의 폭은 그만큼 좁아지고, 그나마 선택할 수 있는 것마저도 그리 썩 마음에 들지 않습니다.

* 기존의 구체 클래스로부터 여러분의 구체 클래스를 파생시켜 사용하되, 이번 항목의 앞에서 이야기한 객체 대입에 관련된 문제는 꾹꾹 참고 지켜냅니다. 또한, 항목 3에서 이야기한 객체 배열에 관련된 함정도 조심해야 합니다.
* 해당 라이브러리 클래스보다 상위 계통 구조에 있으며 여러분이 원하는 기능을 많이 가진 추상 클래스를 찾아서, 그 클래스로부터 파생 클래스를 만듭니다. 물론 원하는 추상 클래스가 없을 수도 있고, 있다고 해도 여러분이 기본 클래스로 삼으려고 했던 그 라이브러리 클래스의 동작 원리를 반복해서 구현해 줘야 하므로 중노동이 배가됩니다.
* 기본 클래스로 삼으려고 했던 라이브러리 클래스를 사용해서 독립적인 클래스를 새로 구현합니다. 이를테면, 새 클래스에는 라이브러리 클래스의 인스턴스를 데이터 멤버로 넣고, 라이브러리 클래스의 인터페이스를 다시 구현하는 것입니다.

```c++
class Window {                        // 이 클래스는 라이브러리 클래스입니다.
public:
    virtual void resize(int newWidth, int newHeight);
    virtual void repaint(void) const;
    
    int width(void) const;
    int height(void) const;
};
 
class SpecialWindow {                // 이 클래스는 Window로부터
public:                                // 파생시켜 만들고 싶었던
    ...                                // 클래스입니다.
    
    // Window 객체를 써서 결과를 내어주는 비가상 함수
    int width(void) const { return w.width(); }
    int height(void) const { return w.height(); }
    
    // "상속하고 싶었던" 가상 함수를 새로 구현한 부분
    virtual void resize(int newWidth, int newHeight);
    virtual void repaint(void) const;
    
private:
    Window w;
};
 
// 물론 라이브러리가 업데이트 되었을 경우에는
// 그에 따른 조치를 취해야 합니다.
 
// 또한, 라이브러리 클래스에 선언된 가상
// 함수를 어떻게 해보겠다는 생각은 버리는게
// 좋습니다. 직접 상속하지 않으면 가상
// 함수는 재정의(오버라이드)할 수 없으니까요.
```
* 그냥 있는 대로 삽시다. 라이브러리에 있는 클래스를 사용할 수 있도록 여러분의 프로그램을 고치세요. 파생 클래스를 만들어서 추가하고 싶었던 기능은 별도의 비멤버 함수(전역이든 정적이든)로 구현합니다. 이렇게 만든 소프트웨어는 그리 깔끔하지도, 효율적이지도, 유지보수성이 좋지도, 확장성이 좋지도 않겠지만, 어쨌든 여러분이 원하던 바대로 돌아가긴 할 것입니다.


* 딱히 마음에 드는 것이 하나도 없겠지만, 그렇기 때문에 프로젝트 진행 상황을 잘 살펴서 결단을 내린 후에, 가장 덜 고통스러울 만한 방법을 선택하도록 하십시오.

* 그래도 이 후의 인생만은 지금보다 덜 피곤하게 만들고 싶다면, 라이브러리 제작사의 고객 센터에 항의해서 설계 좀 잘 하라고 압력을 넣으세요.

* 외부 라이브러리를 사용하는 경우에는 어쩔 수 없지만, 여러분이 건드릴 수 있는 코드에 대해서는 '상속 관계의 말단에 있지 않은 (non-leaf) 클래스는 반드시 추상 클래스여야 한다' 라는 법칙을 지키도록 하십시오.

## 항목 34 : 한 프로그램에서 C++와 C를 함께 사용하는 방법을 이해하자

### 네임 맹글링
* 네임 맹글링, 우리말로 '함수 이름 뒤섞기'라고 불리는 이 동작은 C++ 컴파일러가 함수에 대해 독특한 내부 호출용 이름을 만들어 붙이는 동작을 의미합니다.

* 이 동작은 C++의 함수 오버로딩을 위한 기능인데, 링커는 같은 이름을 가진 여러 개의 함수를 구분하지 못하기 때문에 이런 링커의 딱한 사정을 봐준 결과 네임 맹글링이란 기능이 생기게 되었습니다.

* C++만 사용하여 개발하는 경우에는 네임 맹글링이 아무런 문제가 되지 않습니다. 예를 들어 drawLine이란 이름의 C++ 함수가 있는데 컴파일러가 이 이름을 xyzzy로 맹글링했다고 해도, drawLine이란 이름을 사용하는 데에는 여전히 문제가 없습니다.

* 그런데 drawLine 함수가 C로 제작한 라이브러리 함수라면 이야기가 달라집니다. 이 경우 C++ 소스 파일에는 다음과 같은 선언문이 적힌 헤더 파일이 인클루드 되겠지요.
```c++
void drawLine(int x1, int y1,
                int x2, int y2);
```
* 문제는 이 함수를 호출하는 C++ 문장을, 컴파일하는 컴파일러가 C++ 컴파일러라는 사실입니다. 즉, drawLine이란 이름은 네임 맹글링을 통해 내부 호출용 이름으로 바뀌기 때문에, 다음과 같은 함수 호출 코드는,
```c++
drawLine(a, b, c, d);
// 맹글링되지 않은 이름으로 호출
```
* 컴파일러에 의해 다음과 같은 함수 호출문으로 해석되고, 이 C++ 소스 파일을 통해 만들어진 목적 파일에 들어갑니다.

```c++
xyzzy(a, b, c, d);
// 맹글링된 함수 이름으로 호출
```
* 그러나 drawLine은 C 함수이기 때문에 네임 맹글링이 일어나지 않아 목적 파일 역시 drawLine이란 이름을 사용합니다.

* 이 문제를 해결하려면, C++ 컴파일러에게 drawLine을 호출한 코드에 대해서는 네임 맹글링을 수행하지 않도록 extern "C" 지시자(directive)를 사용해야 합니다.
```c++
// drawLine란 이름의 함수를
// 선언하되, 이 이름은
// 맹글링하지 않습니다.
extern "C"
void drawLine(int x1, int y1,
                int x2, int y2);
```
* 지시자 안에 "C"가 들어 있다고 해서 extern "Pascal" 이나 extern "FORTRAN" 도 있지 않을까 착각할 수 있지만, 그런 것은 없습니다.

* 어떤 함수를 어셈블러로 만들게 되었다면, 이 함수를 C++에서 사용하려고 할때 역시 extern "C"를 붙여서 선언해야 합니다.

* C++로 만든 라이브러리를 다른 언어를 쓰는 사람들에게 공급하려고 할 때도 extern "C" 지시자를 쓸 수 있습니다.
```c++
// 이 C++ 함수는 C++ 이외의 언어에서 사용할 수 있도록
// 설계되었습니다. 따라서 네임 맹글링을 하지 않도록 합니다.
extern "C" void simulate(int iterations);
 
// 네임 맹글링을 막고 싶은 함수가 여러 개 되는 경우도
// 있을 수 있습니다. 이럴 때는 다음과 같이 간편하게
// 쓸 수 있습니다.
 
extern "C" {    // 이후의 모든 함수에 대해서는
                // 네임 맹글링을 막습니다.
    
    void drawLine(int x1, int y1, int x2, int y2);
    void twiddleBits(unsigned char bits);
    void simulate(int iterations);
    ...
}
```

* C++와 C를 동시에 사용하는 경우에 헤더 파일의 관리를 단순하게 하는 방법도 있습니다. C++ 컴파일중에만 __cplusplus 라는 전처리자 심볼이 정의(define)된다는 사실을 이용하는 것입니다.

```c++
#ifdef __cplusplus
 
extern "C" {
 
#endif
    
    void drawLine(int x1, int y1, int x2, int y2);
    void twiddleBits(unsigned char bits);
    void simulate(int iterations);
    ...
#ifdef __cplusplus
}
 
#endif
 
```

* 마지막으로 덧붙이자면, C++ 컴파일러의 "표준" 네임 맹글링 알고리듬 같은 것은 없습니다. 컴파일러마다 구현 방식이 천차만별이며, 동작방식도 제각각입니다.

* 때문에 호환되지 않는 C++ 컴파일러로 생성한 목적 코드(object code)를 섞으려고 하면 바로 링크 에러가 날 것입니다. 맹글링된 이름이 일치될 리가 없으니까요.


### 정적 데이터 초기화
* C++에서는 main 함수가 호출되기 전과 호출된 후에 많은 코드들이 실행됩니다. 특히, main의 몸체가 실행되기 전에 정적 클래스 객체와 전역 객체, 네임스페이스와 파일 범위 바깥에 있는 객체의 생성자가 실행된다는 사실은 심각하게 짚어 봐야 할 필요가 있습니다. 그리고 이렇게 생성된 객체들은 main의 실행이 끝난 후에 소멸됩니다.

* 대다수의 C++ 컴파일러는 main의 시작 부분에 특수한 전용 함수를 호출하는 코드를 삽입하는 방법으로, 정적 데이터를 초기화합니다. 비슷한 이치로, main의 마지막 부분에는 정적 데이터 소멸을 수행하는 함수를 호출하는 코드를 끼워 넣었습니다.

```c++
int main(int argc, char *argv[])
{
    performStaticInitialization();        // 컴파일러가 생성한
                                        // 코드
    
    main에 개발자가 써넣는 코드가 이 부분에 들어갑니다;
    
    performStaticDestruction();            // 컴파일러가 생성한
                                        // 코드
};
 
// performStaticInitialization 함수와
// performStaticDestruction 함수는
// 실제로 이런 이름이 아닐 수도 있습니다.
 
// 암호문처럼 전혀 알아보지 못할 이름일 수도 있고,
// 인라인 함수로 구현되어 실제 목적 파일에는
// 함수 이름조차 찾기 힘들 수도 있습니다.
```
* 이러한 이유로 main이 C++로 작성되지 않으면 정적 객체의 초기화화 소멸이 정상적으로 이루어 지지 않습니다. 그러므로 C++를 써서 소프트웨어의 일부를 만들고 있었다면 main은 C++로 작성하시는 편이 정신건강에 좋습니다.

* 프로그램의 거의 대부분을 C로만 짰고 C++라이브러리를 조금 사용했다고 하더라도 C++ 라이브러리에 정적 객체가 포함되어 있을수도 있으므로, main은 C++로 작성하시는게 좋습니다. 멀쩡한 C 코드를 갈아엎고 C++로 다시 만들라는 뜻은 아니고, C로 만들었던 main에는 realMain이란 새 이름을 지어주고, C++로 만든 main에서 realMain을 호출하게 하는 것입니다.

```c++
extern "C"                                    // 이 함수를
int realMain(int argc, char *argv[]);        // C로 구현합니다.
 
int main(int argc, char *argv[])            // 이 main은 C++로 작성합니다.
{
    return realMain(argc, argv);
}
 
// 이런 방법을 썼을 경우엔 main 위에 간단한
// 주석문을 붙여서 알아보기 쉽게 하는 것이
// 좋습니다.
```

* 컴파일러 제작사들도 이런 문제를 잘 알고 있기 때문에, 거의 모든 제작사들이 정적 데이터 초기화 및 소멸을 맡아주는 별도의 메커니즘을 언어 외적으로 제공하고 있습니다. 이런 기능에 대한 정보는 여러분이 사용하고 있는 컴파일러 설명 문서를 찾아서 참고하시는 것이 좋겠습니다.


### 동적 메모리 할당
* 말은 쉽지만, 생각보다 실천하기 힘든 때도 있습니다. strdup 함수 아시죠? 이 함수는 C 표준 API도 아니고 C++ 표준 API도 아니지만, 상당히 많이 사용되고 있는 함수입니다.
```c++
char * strdup(const char *ps);
// ps가 가리키고 있는 문자열을
// 별도의 힙 메모리에 복사하고
// 그 포인터를 반환합니다.
```
* 메모리 누수를 막기 위해서는 strdup 안에서 할당된 힙 메모리를 strdup의 호출부가 해제해 주어야 합니다. 그런데 free를 사용하여 해제해야 할까요? delete를 사용하여 해제해야 할까요? 이 함수가 C 라이브러리에 속해 있었다면 전자가 정답이겠고, C++ 라이브러리에 속해 있었다면 후자가 정답이겠지요. 즉, strdup를 호출한 이후의 처리 방법이 시스템은 물론 컴파일러마다 다르다는 것입니다.

* 이런 골치 아픈 호환성 문제를 줄이려면, 표준 라이브러리에 속해 있지도 않으며 대부분의 플랫폼에서 안정된 형태로 쓸 수도 없는 함수는, 아예 호출하지 않도록 해야 합니다.

### 자료 구조의 호환성

* C++와 C 컴파일러가 호환성 있는 코드를 만들어낸다면, 두 언어로 만든 함수 사이에 객체의 포인터가 오가거나 비멤버 함수나 정적 함수에 대한 포인터가 오가는 데에는 별 문제가 없습니다. 구조체 종류(struct나 class, 이후에는 구조체로 지칭)와 기본 제공 타입(int, char 등) 역시 C++와 C 경계를 자유롭게 오갈 수 있습니다.

* C++ 구조체의 메모리 배열 구조에 대한 규칙은 C 구조체와 동일하기 때문에, 두 언어로 컴파일되는 구조체는 메모리 배열 구조에 있어서도 동일하다고 가정해도 별 문제는 없습니다. C++ 구조체에 멤버 함수를 추가한다고 해도, 그 멤버 함수가 가상 함수만 아니면 메모리 배열 구조는 변하지 않아야 하므로, 비가상 함수만 가지고 있는 객체는 C 에서도 쓸 수 있습니다.

* 하지만 가상 함수가 하나라도 있는 경우에는, 가상 함수 테이블에 관련된 숨겨진 데이터 멤버(항목 24 참조)로 인해 C++ 구조체의 메모리 배열 구조가 바뀌기 때문에 C 와의 호환성이 깨집니다.

* 또한 한 구조체에서 다른 구조체를 파생시켜도 메모리 배열 구조가 바뀌는 것이 보통이므로, 상속 계통 구조로 짜여진 C++ 구조체 타입은 C 함수에 쓰기 힘듭니다.

### 요약
* 두 개의 언어에서 동시에 사용되는 함수는 extern "C"로 선언합니다.
* 가능하면 main 은 C++로 작성합니다.
* new 로 할당한 메모리는 delete 로 해제하고 malloc로 할당한 메모리는 free로 해제합니다. 
* 두 개의 언어로 작성한 함수 사이에 전달할 수 있는 데이터 타입은 C 컴파일러로 컴파일 되는 것으로만 한정합니다. C와 호환 가능한 C++ 구조체는 비가상 멤버함수까지만 가질수 있습니다. 

## 항목 35 : C++ 언어의 최신 표준안과 표준 라이브러리에 대해 익숙해지자
* 굳이. 여기까지만 하자. 정말 고생했다. 수고했어.