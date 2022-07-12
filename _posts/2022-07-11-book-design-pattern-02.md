---
title:  "[book] GoF 디자인 패턴! 이렇게 활용한다 : C++로 배우는 패턴의 이해와 활용 03"
excerpt: "GoF 디자인 패턴! 이렇게 활용한다 : C++로 배우는 패턴의 이해와 활용 03"

categories:
  - Book
tags:
  - [Cpp, Programming, Book]

toc: true
toc_sticky: true
 
date: 2022-07-11
last_modified_at: 2022-07-11

published: true
---


# PART 3. 구조 개선을 위한 디자인 패턴

## 9\. 기존 모듈 재사용을 위한 인터페이스 변경 문제

### 문제 사례
* 그래픽 편집기를 만든다고 가정해 보자. 모든 형태를 대표하기 위한 Shape 클래스를 추상클래스로 정의했고 하위ㅇ에 Line, Rectangle, Circle등이 정의되어 있다. 그런데 선이나 다각형이 아닌 텍스트를 그리거나 편집할 수 있는 기능을 추가로 구현해 달라는 요청을 받았다. 

### Adapter 패턴
* 클라이언트가 사용하는 객체의 자료형이 다르다는게 문제의 원인이다.  이를 해결하기 위해 사용되는 객체의 자료형을 통일시켜주는것이 유일한 방법이다. 
* 모든 객체가 Shape 클래스의 자료형이 되게한다. 이를 위해 텍스트를 다루기 위한 클래스도 TextShape 의 이름으로 Shape 클래스의 하위 클래스로 정의하는것이 바람직하다. 그러면 내부 구현은 어떻게 하면 좋을까?
* 2가지 형태를 생각해 볼 수 있다.
    * 1. TextShape 클래스의 인터페이스가 호출되면 그 내부에서 다시 TextView 클래스의 인터페이스를 호출해서 원하는 기능을 구현해 주는 방식.
        * 객체를 참조한다고 하여 Object Adapter 패턴이라고도 한다. 
    * 2. TextShape 클래스를 정의하되 Shape 클래스와 TextView 클래스로부터 동시에 상속을 받아 정의하는 방식
        * 다중 상속을 통해 클래스를 참조한다고 하여 Class Adapter 패턴이라고도 한다. 

### 샘플 코드
* Object Adapter 패턴 예제

```c++
class Rectangle {  
  public :    
    Rectangle(int x1, int y1, int x2, int y2) {      
      x1_ = x1; y1_ = y1; x2_ = x2; y2_ = y2;    
    }    
    void Draw() {}  
  private :    
    int x1_, y1_, x2_, y2_;
};

class TextView {  
  public :     
    Rectangle GetExtent() {      
      return Rectangle(x1_, y1_, x1_ + width_, y1_ + height_);    
    }  
  private :     
    int x1_, y1_;    int width_, height_;
};

class Shape {  
  public :     
    virtual Rectangle BoundingBox() = 0;
};

class LineShape : public Shape {  
  public :    
    Rectangle BoundingBox() {      
      return Rectangle(x1_, y1_, x2_, y2_);    
    }  
  private :     
    int x1_, y1_, x2_, y2_;
};

class TextShape : public Shape {
  public : 
    TextShape() {
      pText_ = new TextView;
    }

    Rectangle BoundingBox() {
      return pText_->GetExtent();
    }
  private:
    TextView *pText_;
};

void DisplayBoundingBox(Shape *pSelectedShape) 
{  
  (pSelectedShape->BoundingBox()).Draw();
}

void 
main()
{
  TextShape text;
  DisplayBoundingBox(&text);
}
```

* Class Adapter 패턴 예제

```c++
class Rectangle {  
  public :    
    Rectangle(int x1, int y1, int x2, int y2) {      
      x1_ = x1; y1_ = y1; x2_ = x2; y2_ = y2;    
    }    
    void Draw() {}  
  private :    
    int x1_, y1_, x2_, y2_;
};

class TextView {  
  public :     
    Rectangle GetExtent() {      
      return Rectangle(x1_, y1_, x1_ + width_, y1_ + height_);    
    }  
  private :     
    int x1_, y1_;    int width_, height_;
};

class Shape {  
  public :     
    virtual Rectangle BoundingBox() = 0;
};

class LineShape : public Shape {  
  public :    
    Rectangle BoundingBox() {      
      return Rectangle(x1_, y1_, x2_, y2_);    
    }  
  private :     
    int x1_, y1_, x2_, y2_;
};

class TextShape : public Shape, private TextView {
  public : 
    Rectangle BoundingBox() {
      return GetExtent();
    }
};

void DisplayBoundingBox(Shape *pSelectedShape) 
{  
  (pSelectedShape->BoundingBox()).Draw();
}

void 
main()
{
  TextShape text;
  DisplayBoundingBox(&text);
}
```

### 고려사항
* Class Adapter 패턴의 구현은 다중 상속이기에 Client 에 공개될 인터페이스를 가진 클래스는 public 로 상속하고, 내부 구현을 위해 사용할 클래스는 private 형태로 상속하는 것이 일반적인 방법이다. 
* Object Adapter 패턴 구현시 내부적으로 구현을 위해 참조하는 객체는 언제 생성할 것인가 고민거리이다. 
* Object Adapter 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-01.png)

* Class Adapter 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-02.png)




## 10\. 인터페이스와 구현의 명확한 분리 문제

### 문제 사례
* 검색 엔진을 만드는 회사가 있다고 생각하자. 이것은 windows 에서도 실행되어야 하고 osx 에서도 실행되어야 한다. 그러나 이 검색 엔진을 사용해서 검색 서비스를 개발하는 사람들에게는 플랫폼과 관계없이 동일한 인터페이스를 제공하려고 한다. 그렇다면 어떤 형태로 검색 엔진에 대한 클래스 라이브러리를 제공하는 것이 좋을지 고민해 보자. 
* 만약 운영체제에 따라 OsxSerchEngine, WindowsSerchEnginge 와 같은 클래스를 정의하게 되면 문제가 있을 것이다. 
    * 이럴 경우 어떻게 클래스를 설계하는 것이 좋을까?

### Bridge 패턴
* #ifdef, #dlse, #endif 로 처리하는것은 문제가 많을 것이다. 
* 하위 클래스를 정의하는 방식또한 새로운 하위 클래스가 추가될때 마다 정의해야할 것들이 많아지는 문제가 있다. 
* 기본적으로 클래스 상속 관계를 정의함에 있어 분류 기준이 두 가지 이상이 되면 문제가 된다. 이런 문제를 해결하기 위한 방법은 분류 기준에 따라 각각 독립된 클래스 상속 관계를 정의하는 것이다. 
    * 즉 사용 용도에 따른 논리적 관점의 클래스 상속 구조와 구현 플랫폼별 클래스 상속 구조를 별개로 정의하는 것이다. 
    * 이때 사용 용도에 따른 논리적 관점의 클래스 객체는 구현 관련된 클래스의 객체를 데이터 멤버 형태로 참조하는 형태가 될 것이다. 
* 이처럼 클래스 구조를 정의함에 있어 외부에 공개되는 인터페이스 및 그에 따른 논리적 관점의 상속 구조와 이들을 구현하기 위한 클래스으 ㅣ상속 구조를 독립적으로 정의하고 논리적 관점의 클래스에서 구현 클래스를 참조하는 형태로 설계된 클래스 구조를 Bridge 패턴이라고 한다. 
* 이 패턴의 목적은 외부에 공개하기 위한 인터페이스와 그것을 구현하는 부분을 명백히 다른 클래스로 정의하는 방식을 통해 구현 방식과는 독릭된 인터페이스를 제공하는 것이 목적이다.  

### 샘플 코드

```c++
class BTree { };

class SearchEngineImp {
  public :
    virtual bool Search(string s, string idxFn) = 0;
    virtual bool Search(string s, BTree& bTree) = 0;
};

class UnixSearchEngineImp : public SearchEngineImp {
  public :
    bool Search(string s, string idxFn) {
      // -- Unix 환경에 맞추어 idxFn 에서 문자열 검색
      return true;
    }
    bool Search(string s, BTree& bTree) {
      // -- Unix 환경에 맞추어 BTree 에서 문자열 검색
      return true;
    }
};

class WindowsSearchEngineImp : public SearchEngineImp {
  public :
    bool Search(string s, string idxFn) {
      // -- MS Windows 환경에 맞추어 idxFn 에서 문자열 검색
      return true;
    }
    bool Search(string s, BTree& bTree) {
      // -- MS Windows 환경에 맞추어 BTree 에서 문자열 검색
      return true;
    }
};

class SearchEngine {
  public :
    SearchEngine() { pImp_ = 0; }
    virtual bool Search(string s) = 0;

  protected :
    SearchEngineImp* GetSearchEngineImp() {
      if (pImp_ == 0) {
#ifdef  __WIN32__
        pImp_ = new WindowsSearchEngineImp;
#else
        pImp_ = new UnixSearchEngineImp;
#endif
      }
      return pImp_;
    }

  private :
    SearchEngineImp* pImp_;
};

class WebSearchEngine : public SearchEngine {
  public :
    WebSearchEngine(string idxFn) { indexFn_ = idxFn; }
    bool Search(string s) {
      return GetSearchEngineImp()->Search(s, indexFn_);
    }

  private :
    string indexFn_;
};

class DBSearchEngine : public SearchEngine {
  public : 
    bool Search(string s) {
      return GetSearchEngineImp()->Search(s, bTree_);
    }

  private :
    BTree bTree_;
};

void 
main() 
{
  WebSearchEngine finder("inverted_file4web.idx");
  finder.Search("디자인 패턴");
}
```

* 구현 클래스의 상위에 추상 클래스를 두는 것은 왜 필요할까?
    * 구현 클래스가 단 한 개 뿐이라면 굳이 추상 클래스가 필요가 없다. 구현 클래스가 여러개가 만들어질 상황이라면 이를 사용하는 입장에서는 결국 각각을 구별해서 개발을 해야하는 문제가 있기에 각각의 구현 클래스가 동일한 자료형으로 인식되고, 외부에 제공되는 인터페이스가 동일하게 만들어 주어야 한다. 
    * 결국 구현 클래스가 하나 이상 늘어날 가능성이 있다면 구현 클래스의 상위에 추상 클래스를 두는 것이 꼭 필요하다고 볼수 있다. 

* Bridge 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-03.png)


* Bride 패턴 대신에 Adapter 패턴을 적용하는 것을 가정해 보면 다음과 같은 차이가 있다. 
    * Class Adapeter 패턴을 적용하는것은 인터페이스 클래스는 public  상속을 하고 구현 클래스는  private 상속을 한 클래스를 정의하면 Client 입장에서는 인터페이스 클래스가 제공하는 인터페이스를 사용하면서 내부 구현은 구현 클래스를 이용해 작성하게 될 것이다. 그러나 이 방식은 상속 받은 클래스가 정적으로 결정되기 때문에 Bridge 패턴에서처럼 동적으로 구현 클래스를 변경하는 것은 불가능한 차이가 있다.
    * Object Adapter 패턴을 적용하는 경우는 
        * Bridge 패턴은 인터페이스와 구현을 분리하기 위한 것이고 Adapter패턴은 이미 존재하고있는 객체를 이용해 다른 객체의 인터페이스를 만들어 주기 위한 것이다.. 
        * 설계 측면에서 Bridge 패턴은 인터페이스와 구현이 서로 독립적으로 변경될 수 있게 처음부터 고려한 것이고 Adapter 패턴은 이미 존재하는 클래스를 어떻게 활용할 것인가하는 측면에서 접근하는 것이 다르다. 
        * 구현측면에서는 Adapter패턴은 구현 클래스의 객체를 데이터 멤버로 반드시 가지고 있을 필요가 없으며 Client 가 필요로 하는 인터페이스라도 Adapter 에 해당하는 클래스가 그 기능을 제공하지 않을수 있지만, Bridge패턴의 경우에는 항상 구현 클래스의 객체가 있어야 한다. 

## 11\. 부분\-전체 관계 구조 취급 문제
* 객체 하나는 개체 여러개로 구성된닥도 할 수 있다. 대부분의 객체에서 이런 구성은 부분-전체 관계는 데이터 멤버를 통해 정적으로 정의된다. 그러나 때론 동적으로 정의할 필요도 있다. 이럴때 어떤 방식으로 클래스를 설계하는 것이 좋을지 살펴본다. 

### 문제 사례
* 6장에서 보았던 그래핅 편집기를 다시 생각해 본다. 팔레트에 기본 여러 도형이 있고, 이것을 끌어다 가져다 놓으며 그림을 그리게 된다. 이때 어떤 이용자가 기본적으로 제공되는 도형을 조합해서 새로운 도형을 만들었다고 생각해보자. 새로운 도형을 만들었을때 이 사용자는 그 도형을 반복적으로 사용하고 싶어졌을때 어떻게 하면 좋을까?
* visio 같은 경우는 사용자가 정의한 도형을 팔레트에 등록할 수 있도록 해 준다. 

### Composite 패턴
* 처음부터 제공되는 기본 객체와 이용자가 도형 여러 개를 조합해서 구성한 구성 객체라는 용어를 사용하여 보겠다. 
* 기본 객체들을 임의로 조합해서 만든 구성 객체를 어떻게 생성, 관리할 것인가와 기본 객체와 구성 객체가 어떻게 하면 동일한 형태로 사용 될 수 있을까를 정리해 보자. 
* 기본 객체와 구성 객체에 대한 클래스들이 동일한 자료형으로 다루어 지도록 만들어야 한다. 이를 위한 좋은 방법은 기본 객체에 대한 클래스가 동일한 상속 구조하에 놓여지도록 하면 된다. 
* 기본 객체나 구성 객체 모두 구성 객체의 구성원이 될 수 있게 만들어줘야하는데, 이는 기본 객체에 대한 클래스와 구성 객체에 대한 클래스를 공통으로 대표할 수 있는 클래스를 구성 객체가 참조하게 클래스를 설계하면 된다. 

### 샘플 코드

```c++

class Graphic {
  public :
    virtual void Draw() = 0;
    virtual void Add(Graphic* pObj) {}
    virtual void Remove(Graphic* pObj) {}
    virtual Graphic* GetChild(int nth) { return 0; }
};

class Line : public Graphic {
  public :
    void Draw() { 
      // -- 라인 그리기 
    }
};

class Triangle : public Graphic {
  public :
    void Draw() { 
      // -- 삼각형 그리기 
    }
};

class Rectangle : public Graphic {
  public :
    void Draw() {
      // -- 사각형 그리기
    }
};

class ComposedGraphic : public Graphic {
  public :
    void Draw() {
      list<Graphic*>::iterator iter1;
      for (iter1 = components_.begin(); iter1 != components_.end(); iter1++) 
        (*iter1)->Draw();
    }

    void Add(Graphic *pObj) {
      components_.push_front(pObj);
    }

    void remove(Graphic *pObj) {
      list<Graphic*>::iterator iter1;
      for (iter1 = components_.begin(); iter1 != components_.end(); iter1++) {
        if (*iter1 == pObj) 
          components_.erase(iter1);
      }
    }

    Graphic* GetChild(int nth) {
      int i;
      list<Graphic*>::iterator iter1;
      for (i=0, iter1 = components_.begin(); 
                iter1 != components_.end(); iter1++, i++) {
        if (i == nth) return *iter1;
      }
      return 0;
    }

  private :
    list<Graphic *> components_;
};

class Palette {
  public :
    Palette() {
      Graphic* pGraphic = new Triangle;
      items_[1] = pGraphic;

      pGraphic = new Rectangle;
      items_[2] = pGraphic;

      // -- 필요한 만큼 기본 도형 등록
    }
    
    void RegisterNewGraphic(Graphic* pGraphic) {
      items_[items_.size()+1] = pGraphic;
    }

  private :
    map<int, Graphic*> items_;
};

void
main()
{
  Triangle aTriangle;
  Rectangle aRectangle;
  ComposedGraphic aComposedGraphic2;

  aComposedGraphic2.Add(&aTriangle);
  aComposedGraphic2.Add(&aRectangle);

  Line aLine;
  Rectangle aRectangle2;
  ComposedGraphic aComposedGraphic;

  aComposedGraphic.Add(&aComposedGraphic2);
  aComposedGraphic.Add(&aLine);
  aComposedGraphic.Add(&aRectangle2);
}
```

* ComposedGraphi 클래스는 Graphic의 하위 클래스로 정의되었으며, 내부에는 데이터 멤버로 Graphic 클래스 객체에 대한 포인터 변수를 리스트로 저장, 관리할수 있도록 하였다. 
* Composite 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-04.png)



## 12\. 특정 객체의 기능 동적 추가\, 삭제 문제
* 클래스 정의와는 무관하게 특정 객체에게 동적으로 기능을 추가하거나 추가했던 기능을 삭제하려면 어떻게 할 수 있을까? 이럴때 어떤 형태로 클래스를 설계하는 것이 바람직한지 살펴 보자

### 문제 사례
* 슈팅게임을 생각해 보자. 아이템을 취득해서 공격 범위하 확대된다던지, 무기가 바뀐다던지는 어떤 식으로 제공하였을까?

### Decorator 패턴
* 아이템의 종류를 공격 방향을 추가하거나 삭제하는 것으로만 한정지어 생각해 보겠다. 
* 우리가 원하는 해결 방법은 특정 객체에게만 기능을 추가하너가 삭제하는 것이다. 객체 차원에서 동적으로 이렇게 하려면 어떻게 하면 될까?
    * 다른 객체를 이용해서 원래 객체를 꾸며주는 형태를 취해야 한다. 새로운 기능을 가진 객체가 원래 객체를 꾸며주면서 새로운 기능도 수행해주고 원래 객체가 가진 기능도 수행해주면 마치 새로운 기능이 추가된 것처럼 보이도록 만들 수 있다. 
    * 객체들간의 참조 연결 고리를 이용해서 객체의기능을 추가하거나 삭제하자. 
* Client 입장에서 동일한 형태로 사용하려면 객체 참조 연결고리에 포함되는 객체들이 모두 동일한 자료형으로 표현해야하며 이들이 외부에서 제공하는 인터페이스도 모두 동일해야 한다. 
    * 각 객체들이 속하는 클래스의 상위에 공통 부모 클래스를 정의하고 상속 관계에 놓여진 클래스들이 모두 동일한 인터페이스를 가지도록 하면 될 것이다. 다만 제일 마지막에 놓여진 객체는 다른 객체와는 달리 또 다른 객체를 참조하고 있지 않다. 이것을 다른 클래스와 구분이 되도록 해야할 것이다. 

### 샘플 코드

```c++
class Airplane {
  public :
    virtual void Attack() = 0;
};

class FrontAttackAirplane : public Airplane {
  public :
    void Attack() {
      // -- 전방 공격 수행
      cout << "전방공격" << endl;
    }
};

class Decorator : public Airplane {
  public :
    Decorator(Airplane* pObj) { pComponent_ = pObj; }
    virtual ~Decorator()=0;

    virtual void Attack() {
      if (pComponent_ != 0) 
        pComponent_->Attack();
    }

  private :
    Airplane* pComponent_;
};

Decorator::~Decorator() {}

class SideAttackAirplane : public Decorator {
  public :
    SideAttackAirplane(Airplane* pObj) : Decorator(pObj) {}

    void Attack() {
      Decorator::Attack();
      // -- 측방 공격 수행
      cout << "측방공격" << endl;
    }
};

class RearAttackAirplane : public Decorator {
  public : 
    RearAttackAirplane(Airplane* pObj) : Decorator(pObj) {}

    void Attack() {
      Decorator::Attack();
      // -- 후방 공격 수행
      cout << "후방공격" << endl;
    }
};

void
main()
{
  Airplane* pFrontAttackAirplane = new FrontAttackAirplane;
  Airplane* pSideAttackAirplane = new SideAttackAirplane(pFrontAttackAirplane);
  Airplane* pRearAttackAirplane = new RearAttackAirplane(pSideAttackAirplane);

  pRearAttackAirplane->Attack();
  delete pRearAttackAirplane;

  pSideAttackAirplane->Attack();
} 
```

* SideAttackAirplane 과 RearAttackAirplane 클래스의 객체를 생성하는 방식에 대해 살펴보면, 각 클래스의 생성자는 인자로 참조할 객체를 직접 전달 받고 있다.
* Decorator 패턴의 최상위 클래스는 최대한 가볍게 유지되는것이 좋다. 데이터 멤버를 정의하지 않고 외부에 제공할 인터페이스도 가능한 최소로 정희하는 것이 바람직하다. 
* Decorator 패턴의 특징은 특정 객체에게 동적으로 새로운 기능을 추가하거나, 이미 추가했떤 기능을 삭제하기 위해 객체가 다른 객체를 참조할 수 있게 고안된 것이다. 
* Decorator 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-05.png)



## 13\. 명확한 서브시스템 정의 문제
* 서로 밀접하게 관련된 객체나 클래스들을 묶고 이를 추상화 시켜 서브 시스템이라 정의하겠다. 
    * 복잡한 설계를 간략회 시켜주는 도구이다. 

### 문제 사례
* 이용자가 어떤 검색 조건을 입력했을 때 해당 검색 조건을 만족하는 자료를 DB에서 찾아주는 프로그램을 작성한다고 생각해 보자. 
    * 이런 검색 프로그램은 검색 조건이 제대로 된 것인지 검사하는것 이외에 DBMS에 접속해야하고, 검색에 알맞는 SQL문을 생성해서 DBMS에 전달하여 그 결과를 받아오는등 여러가지 과정을 거쳐야 한다. 그러면 이 검색 프로그램을 이용하는 모든 프로그램은 이런 복잡한 과정을 모두 알아야 할까?

### Facade 패턴
* Client 가 일일이 필요한 클래스의 객체를 생성(DBMS, SQLGenerator, ListDBResult 등)하고 활용하는 형태는 이용하는 입장에서 너무 불편할 것이다. Client 입장에서는 단지 검색 조건을 전달하여 결과만 받아보면 된다. 
* Client 마다 별도의 검색 과정을 수행하기 위한 클래스를 정의하면 문제가 된다. 이를 해결하기 위해 미리 검색 과정을 수행하기 위한 클래스를 정의해서 Client 에 제공해주면 도니다. 
    * 이렇게 하면 Client 마다 별도로 검색 과정을 수행하기 위한 클래스를 정희해서 사용하는 문제를 해결할 수 있다. 더불어 이렇게 하면 클래스간의 관계가 정립되면서 자연스럽게 서브시스템이 정의될 수 있는 점이다. 

### 샘플 코드

```c++
class SearchCond {
  public :
    SearchCond(map<string, string> nvList) {
      condList_ = nvList;
    }

    bool CheckCond() {
      // -- name, value 쌍들의 리스트에 대해 검색 조건을 검사한다.
      // 예를 들어, age에 대해 숫자가 아닌 값이 설정되면 안되며, 
      // 음수값등이 설정되면 안 된다. 
      return true;
    }
  private :
    map<string, string> condList_;
};

class ListData {
  // -- 데이터베이스에서 검색되는 결과 레코드가 저장되기 위한 자료구조
};

class ListDBResult {
  public :
    int GetCount() {
      return result_.size();
    }

    void InitIterator() {
      iter_ = result_.begin();
    }

    ListData GetNextData() {
      return *iter_++;
    }
  private:
    list<ListData> result_;
    list<ListData>::iterator iter_;
};

class Database {
  public :
    Database() {
      // -- 데이터베이스 관리 시스템과 연결을 수행함.
    }

    bool Execute(string sql, ListDBResult& result) {
      // -- sql 수행 결과를 result에 설정하되, sql 수행 과정에서 
      // 문제가 있으면 false를 return하며, 그렇지 않은 경우에는 
      // true 를 return한다.
      return true;
    }
};

class SQLGenerator {
  public :
    string GenerateSQL(SearchCond cond) {
      string sql;

      // -- 검색 조건에 따라 sql 문장을 생성
      return sql;
    }
};


class DBHandler {
  public :
    bool Search(map<string, string> nvList, ListDBResult& result) {
      SearchCond cond(nvList);
      if (! cond.CheckCond()) {
        cout << "잘못된 검색조건" << endl;
        return false;
      }

      SQLGenerator generator;
      string sql = generator.GenerateSQL(cond);

      return db_.Execute(sql, result);
    }
  private :
    Database db_;
};

void
main()
{
  map<string, string> nvList;
  ListDBResult result;

  DBHandler handler;
  handler.Search(nvList, result);
}
```

* DBHandler 클래스 구현에서 보듯, 검색과 관련된 일련의 작업들은 모드 이 클래스에 의해 이루어진다. Facade 패턴을 활용하면 Client의 코딩을 원활히 하고 구분되는 서비 시스템을 블랙박스로 다룰 수 있는 장점이 있다. 

* Facade 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-06.png)



## 14\. 객체의 공유 문제

### 문제 사례
* 휴대폰에서 간단히 할 수 있는 게임을 개발한다고 해보자. 아군, 적군을 나뉠때 아군은 대부분 한명이지만 적군은 여러명일 경우가 많다. 이벤트에 따라 정해진 동작을 적군이 해야하는데, 이러한 동작 이미지들을 어떻게 효율적으로 관리하면 좋을까?

### Flyweight 패턴
* 개별 적군 객체마다 이미지 정보를 저장하기보단 어떻게 하면 논리적으로 개별 객체들 마다 이미지 정보가 저장, 관리되늰것처럼 보임녀서 많은 메모리를 사용하지 않을수 있을지가 관건이다. 
* 이미지 정보를 계산에 의해 생성하지 않고 미리 저장, 관리해야하고 이미지를 최소화하려면 이미지 정보를 개별 객체들이 공유하게 만들어야 한다. 
* 공유 가능 정보와 그렇지 않은 정보를 분리하자. 공유 가능 정보와 그렇지 않는 정보가 분리되어있으면 이들을 따로 관리하자.  이런 목적으로 정의되는 객체를 Flyweight 객체라고 한다. 여기서 공유 가능한 정보는 Flyweight 객체의 내부에 저장 관리되기에 Intrinsic State 라고 하고 , 공유 불가능한 정보는 Flyweight 객체 외부에 저장, 관리되기 때문에 Extrinsic State라고 한다. 

### 샘플 코드

```c++

class EnemyImage {
  public :
    virtual void Display(int x, int y)=0;
  protected :
    EnemyImage() {}
    EnemyImage(const EnemyImage& rhs);
};

class EnemyNoActionImage : public EnemyImage {
  public :
    static EnemyImage * CreateInstance() {
      if (pInstance_ == 0) {
        pInstance_ = new EnemyNoActionImage;
        cout << "EnemyNoActionImage" << endl;
      }
      return pInstance_;
    }

    void Display(int x, int y) {
      // -- x,y 위치에 비트맵 이미지 표시
    }
  protected :
    EnemyNoActionImage() { }
    EnemyNoActionImage(const EnemyNoActionImage& rhs);
    static EnemyImage * pInstance_;
};

EnemyImage* EnemyNoActionImage::pInstance_ = 0;

class EnemyMoveImage : public EnemyImage {
  public :
    static EnemyImage * CreateInstance() {
      if (pInstance_ == 0) {
        pInstance_ = new EnemyMoveImage;
        cout << "EnemyMoveActionImage" << endl;
      }
      return pInstance_;
    }

    void Display(int x, int y) {
      // -- x,y 위치에 비트맵 이미지 표시
    }
  protected :
    EnemyMoveImage() { }
    EnemyMoveImage(const EnemyMoveImage& rhs);
    static EnemyImage * pInstance_;
};

EnemyImage* EnemyMoveImage::pInstance_ = 0;

class EnemyAttackImage : public EnemyImage {
  public :
    static EnemyImage * CreateInstance() {
      if (pInstance_ == 0) {
        pInstance_ = new EnemyAttackImage;
        cout << "EnemyAttackActionImage" << endl;
      }
      return pInstance_;
    }

    void Display(int x, int y) {
      // -- x,y 위치에 비트맵 이미지 표시
    }
  protected :
    EnemyAttackImage() { }
    EnemyAttackImage(const EnemyAttackImage& rhs);
    static EnemyImage * pInstance_;
};

EnemyImage* EnemyAttackImage::pInstance_ = 0;

class EnemyDieImage : public EnemyImage {
  public :
    static EnemyImage * CreateInstance() {
      if (pInstance_ == 0) {
        pInstance_ = new EnemyDieImage;
        cout << "EnemyDieActionImage" << endl;
      }
      return pInstance_;
    }

    void Display(int x, int y) {
      // -- x,y 위치에 비트맵 이미지 표시
    }
  protected :
    EnemyDieImage() { }
    EnemyDieImage(const EnemyDieImage& rhs);
    static EnemyImage * pInstance_;
};

EnemyImage* EnemyDieImage::pInstance_ = 0;

class Enemy {
  public :
    Enemy(int x, int y) {
      curX_ = x; curY_ = y;
      pCurImage_ = EnemyNoActionImage::CreateInstance();
    }

    void Move(int x, int y) {
      curX_ = x; curY_ = y;
      pCurImage_ = EnemyMoveImage::CreateInstance();
    }

    void Attack() {
      pCurImage_ = EnemyAttackImage::CreateInstance();
    }

    void Display() {
      pCurImage_->Display(curX_, curY_);
    }

  private :
    int curX_, curY_;
    EnemyImage * pCurImage_;
};

void
main()
{
  Enemy e1(10,10), e2(20,20);

  e1.Move(30, 30);
  e2.Attack();
  e2.Move(40, 40);
}
```

* Enemy 객체들은 pCurImage_ 데이터 멤버를 통해 자신이 참조하는 이미지 정보를 가리키고 있다. pCurImage_ 데이터 멤버에 의해 가리켜지는 이미지 객체는 Enemy 객체마다 매번 생성하는 것이 아니라 공유하는 형태이다. 
* Flyweight 패턴은 객체 공유를 통해 자원 사용량을 줄여주기위한 설계라고 볼수 있다. 


## 15\. 대리 객체를 통한 작업 수행 문제

### 문제 사례
* 인터넷에서 웹으로 만화 서비스를 제공하는 경우를 생각해 보자. 일반적인 웹 페이지에 비해 데이터 량이 많은 이미지 파일로 서비스가 제공이 될텐데, 어떻게 한정된 자원으로 안정되고 빠른 만화 서비스를 제공할 수 있을까?

### Proxy 패턴
* 서비스 요청에 어떤 단계에서 시간이 가장 많이 걸릴지를 확인해 보면 네트워크를 통해 서버에서 웹 브라우저로 이미지 파일을 전송하는 것과 웹 서버 프로그램이 디스크로 부터 이미지 파일을 ㅇ릭어내는 작업일 것이다. 그렇다면 어떤 방식으로 접근하면 디스크로부터 이미지 파일을 읽어들이는데 걸리는 시간을 줄여 전체적인 서비스 반응 속도를 높일 수 있을까?
* Client 입장에서 이미지를 캐싱하는 곳에서 가져오든지 원본 소스에서 가져오든지 굳이 알 필요가 없는 정보는 관리할 필요가 없다.  이렇게 하지 않으려면 어떻게 하면 좋을까?
* 기본적으로 어떤 역할을 수행하고 있는 클래스가 존재할 때 그 클래스가 제공하는 기능이나 역할을 그대로 활용함녀서 부가적인 기능이나 역할을 수행해 주기 위해 새로운 클래스를 정의하고 Client 의 모든 요청을 새로 정의한 클래스 객체를 거쳐 원래의 클래스 객체에게 전달하는 방식의 클래스 구조를 Proxy 패턴이라고 한다. 
    * 이때 새로 정의한 클래스는 원래 존재하였던 클래스와 동일한 인터페이스를 제공해 주어야 한다. 

### 샘플 코드

```c++

#define MAX_CACHE_COUNT       50

class File {
  public :
    virtual int GetFile(string fn, void *pOut) = 0;
};

class ImageFile : public File {
  public :
    int GetFile(string fn, void *pOut) {
      struct stat statBuf;
      if (stat(fn.data(), &statBuf) < 0) {
        cout << "File Not Found" << endl;
        return (0);
      }

      int fd = _open(fn.data(), O_RDONLY);		// changed to _open in VC++
      if (fd < 0) {
        cout << "File Open Error" << endl;
        return (0);
      }

      pOut = new char[statBuf.st_size];
      int byteCnt = _read(fd, pOut, statBuf.st_size);	// changed to _read in VC++
      _close(fd);		// changed to _close in VC++

      if (byteCnt != statBuf.st_size) {
        cout << "Error while File Reading" << endl;
        return(0);
      }
      return(byteCnt);
    }
};

class ImageFileCache : public File {
  public :
    int GetFile(string fn, void *pOut) {
      pOut = fileCache_[fn];

      struct stat realStat;
      if (stat(fn.data(), &realStat) < 0) {
        cout << "File Not Found" << endl;
        return(0);
      }

      struct stat *pFileStat = fileStat_[fn];
      if (pOut == NULL || pFileStat == 0 
                        || realStat.st_mtime != pFileStat->st_mtime) {
        // -- 캐싱이 안 되어 있거나, 캐싱된 정보가 실제 파일과 다를때
        ImageFile f;
        void* pFileOut;
        int fileSize = f.GetFile(fn, pFileOut);

        if (fileSize <= 0) {
          pOut = NULL;
          return (0);
        }

        RegisterCache(fn, &realStat, pFileOut);
        pOut = pFileOut;
      }
      else {
        // -- 최근 읽은 시간 수정
        for (int i=0; i < MAX_CACHE_COUNT; i++) {
          if (lruInfo_[i].fn_ == fn) {
            lruInfo_[i].lastReadTime_ = time(0);
            break;
          }
        }
      }

      return(realStat.st_size);
    }

  protected :
    void RegisterCache(string fn, struct stat* pFileStat, void* pFile) {
      int cachePos = 0;
      time_t oldestReadTime = time(0);
      for (int i=0; i < MAX_CACHE_COUNT; i++) {
        if (lruInfo_[i].lastReadTime_ == 0) {
          // -- cache 공간이 남아 있는 상태
          cachePos = i;
          break;
        }
        else if (oldestReadTime > lruInfo_[i].lastReadTime_) {
          cachePos = i;
          oldestReadTime = lruInfo_[i].lastReadTime_;
        }
        else {}
      }

      if (lruInfo_[cachePos].lastReadTime_ != 0) {
        // -- 이전 캐싱 정보 삭제
        struct stat *pOldFileStat = fileStat_[lruInfo_[cachePos].fn_];
        void *pOldImgFile = fileCache_[lruInfo_[cachePos].fn_];

        delete fileStat_[lruInfo_[cachePos].fn_];
        delete fileCache_[lruInfo_[cachePos].fn_];

        fileStat_.erase(lruInfo_[cachePos].fn_);
        fileCache_.erase(lruInfo_[cachePos].fn_);
      }

      lruInfo_[cachePos].fn_ = fn;
      lruInfo_[cachePos].lastReadTime_ = time(0);

      struct stat* pNewStat = new struct stat;
      // bcopy(pFileStat, pNewStat, sizeof(struct stat));
	  memcpy(pNewStat, pFileStat, sizeof(struct stat));		// changed to memcpy in VC++

      fileStat_[fn] = pNewStat;
      fileCache_[fn] = (char *)pFile;
    }

  private :
    class LRUInfo {
      public : 
        LRUInfo() { lastReadTime_ = 0; }
        string fn_;
        time_t lastReadTime_;
    }; 

    LRUInfo lruInfo_[MAX_CACHE_COUNT];
    map<string, struct stat*> fileStat_;
    map<string, char*> fileCache_;
};


void
main()
{
  File *pFileServer = new ImageFileCache;

  void *pImgData;
  int fileSize = pFileServer->GetFile("./comic/img/hero1.jpg", pImgData);
}
```

* ImageFileCache 클래스는 이미지 파일이 메모리 상에 캐생되어 있든, 그렇지 않든 Client 가 요청하는 이미지 파일을 찾아 그 내용을 되될리는 역할을 한다. 

* Proxy 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-02-07.png)




## 16\. 구조 개선을 위한 디자인 패턴 정리

### 문제 사례