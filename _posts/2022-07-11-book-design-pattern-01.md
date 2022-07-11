---
title:  "[book] GoF 디자인 패턴! 이렇게 활용한다 : C++로 배우는 패턴의 이해와 활용"
excerpt: "GoF 디자인 패턴! 이렇게 활용한다 : C++로 배우는 패턴의 이해와 활용"

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

아래책을 기반으로 함.
GoF 디자인 패턴! 이렇게 활용한다 : C++로 배우는 패턴의 이해와 활용
http://www.yes24.com/Product/Goods/1389246

* 책관련 소스
    * https://dw.hanbit.co.kr/exam/1281/


# PART 2. 객체 생성을 위한 디자인 패턴

## 3\. 제품군별 객체 생성 문제 : Abstract Factory 패턴

### 문제 사례

* 컴파일러를 개발하기 위해 설계를 한다고 생각해 보자.
    * 입력되는 원시 코드를 토큰단위로 잘라주기위한 스캐너, 구문분석을 하기위한 파서, 코드 생성기 등으로 구성된다고 가정한다.
    * 컴파일러가 실행될 시스템이 다를 경우 기계어 코드를 생성하기 위한 클래스의 구현도 각 시스템마다 달라져야한다.

### Abstract Factory 패턴

* 구체적인 제품군별 Factory 클래시 상위에 Abstract Base Class 를 정의한 형태의 설계 구조를 Abstract Factory 패턴이라고 한다.
    * 개별 제품 클래스의 객체를 생성할때마다 동일한 제품군에 속하는 객체를 생성하기 위해 일일이 조건 검사를 할 필요가 없어졌다는 것과 새로운 제품군을 생성하려고 할 경우 기존 소스코드와 독립적으로 새로운 제품군을 추가하는 것이 가능하다는 장점이 있다.
* 샘플 코드
    * client 에 해당하는 main()을 보면 처음 한번은 어떤 제품군을 생성할 것인지 결정하기 위한 조건 비교문장을 사용한다.
    * 그러나 일단 결정되면 더이상 조건 비교 문장을 사용할 필요가 없다. 객체를 생성하는 인터페이스는 제품군에 관계없이 동일하기 때문에 다형성을 이용하면 원하는 제품군의 객체를 마음대로 생성할수 있기 때문.
    * 이와 같은 다형성은 생성된 제품 객체에 대해서도 적용 가능하다.

``` c++
class Scanner {
  public : 
    virtual ~Scanner()=0;
};

class Parser {
  public : 
    virtual ~Parser()=0;
};

class CodeGenerator {
  public : 
    virtual ~CodeGenerator()=0;
};

class Optimizer {
  public : 
    virtual ~Optimizer()=0;
};

Scanner::~Scanner() {}
Parser::~Parser() {}
CodeGenerator::~CodeGenerator() {}
Optimizer::~Optimizer() {}

class HPScanner : public Scanner {};
class HPParser : public Parser {};
class HPCodeGenerator : public CodeGenerator {};
class HPOptimizer : public Optimizer {};

class SunScanner : public Scanner {};
class SunParser : public Parser {};
class SunCodeGenerator : public CodeGenerator {};
class SunOptimizer : public Optimizer {};

class CompilerFactory {
  public :
    virtual Scanner* CreateScanner() = 0;
    virtual Parser* CreateParser() = 0;
    virtual CodeGenerator* CreateCodeGenerator() = 0;
    virtual Optimizer* CreateOptimizer() = 0;
};

class HPCompilerFactory : public CompilerFactory {
  public :
    Scanner* CreateScanner() { new HPScanner; }
    Parser* CreateParser() { new HPParser; }
    CodeGenerator* CreateCodeGenerator() { new HPCodeGenerator; }
    Optimizer* CreateOptimizer() { new HPOptimizer; }
};

class SunCompilerFactory : public CompilerFactory {
  public :
    Scanner* CreateScanner() { new SunScanner; }
    Parser* CreateParser() { new SunParser; }
    CodeGenerator* CreateCodeGenerator() { new SunCodeGenerator; }
    Optimizer* CreateOptimizer() { new SunOptimizer; }
};

CompilerFactory *pFactory;

int main() {
  struct utsname sysInfo;

  // -- OS 버전 및 하드웨어 타입 정보 얻기 위한 시스템 함수
  if (uname(&sysInfo) < 0) { 
    cout << "Error Occurred" << endl;
    return(-1);
  }

  if (strncasecmp(sysInfo.sysname, HPUX, strlen(HPUX)) == 0) {
    // -- HP 용 객체 생성 및 사용
    pFactory = new HPCompilerFactory;
  }
  else if (strncasecmp(sysInfo.sysname, SUNOS, strlen(SUNOS)) == 0) {
    // -- Sun 용 객체 생성 및 사용
    pFactory = new SunCompilerFactory;
  }
  else {
    // -- 지원 안하는 시스템 환경
    cout << sysInfo.sysname << endl;
    return(0);
  }

  Scanner *pScanner = pFactory->CreateScanner();
  Parser *pParser = pFactory->CreateParser();
  CodeGenerator *pCodeGenerator = pFactory->CreateCodeGenerator();
  Optimizer *pOptimizer = pFactory->CreateOptimizer();
}
```

### 실제 구현할때 고려해야할 사항 3가지

* Factory 객체를 하나만 생성, 유지하는 방법
* 객체 생성시 원본 객체를 봊게하는 방식으로 객체를 생성하는 방법
* 새로운 종류의 제품이 추가되었을때 어떻게 대처 가능한가에 대한 것이다.

### Factory 객체를 하나만 생성, 유지하는 방법

* Abstract Factory 패턴을 실제 구현할 경우 구체적인 Concerete Factory 객체는 하나면 충분할 경우가 많다.

``` c++
#include <iostream>
#include <strings.h>
#include <sys/utsname.h>
using namespace std;

#define HPUX "HPUX"
#define SUNOS "SunOS"

class Scanner {
    public :
          virtual ~Scanner()=0;
};

class Parser {
  public : 
    virtual ~Parser()=0;
};

class CodeGenerator {
  public : 
    virtual ~CodeGenerator()=0;
};

class Optimizer {
  public : 
    virtual ~Optimizer()=0;
};

Scanner::~Scanner() {}
Parser::~Parser() {}
CodeGenerator::~CodeGenerator() {}
Optimizer::~Optimizer() {}

class HPScanner : public Scanner {};
class HPParser : public Parser {};
class HPCodeGenerator : public CodeGenerator {};
class HPOptimizer : public Optimizer {};

class SunScanner : public Scanner {};
class SunParser : public Parser {};
class SunCodeGenerator : public CodeGenerator {};
class SunOptimizer : public Optimizer {};

class CompilerFactory {
  public :
    virtual Scanner* CreateScanner()=0;
    virtual Parser* CreateParser()=0;
    virtual CodeGenerator* CreateCodeGenerator()=0;
    virtual Optimizer* CreateOptimizer()=0;
  protected :
    CompilerFactory() {}
    CompilerFactory(const CompilerFactory& rhs);
    static CompilerFactory *pInstance_;
};

class HPCompilerFactory : public CompilerFactory {
  public :
    static HPCompilerFactory* CreateInstance() {
      if (pInstance_ == 0)
         pInstance_ = new HPCompilerFactory;
      return (HPCompilerFactory*) pInstance_;
    }

    Scanner* CreateScanner() { new HPScanner; }
    Parser* CreateParser() { new HPParser; }
    CodeGenerator* CreateCodeGenerator() { new HPCodeGenerator; }
    Optimizer* CreateOptimizer() { new HPOptimizer; }

  protected :
    HPCompilerFactory() {}
    HPCompilerFactory(const HPCompilerFactory& rhs);
};

class SunCompilerFactory : public CompilerFactory {
  public :
    static SunCompilerFactory* CreateInstance() {
      if (pInstance_ == 0)
         pInstance_ = new SunCompilerFactory;
      return (SunCompilerFactory*) pInstance_;
    }

    Scanner* CreateScanner() { new SunScanner; }
    Parser* CreateParser() { new SunParser; }
    CodeGenerator* CreateCodeGenerator() { new SunCodeGenerator; }
    Optimizer* CreateOptimizer() { new SunOptimizer; }

  protected :
    SunCompilerFactory() {}
    SunCompilerFactory(const SunCompilerFactory& rhs);
};

CompilerFactory* CompilerFactory::pInstance_ = 0;

CompilerFactory *pFactory;

int main() {
  struct utsname sysInfo;

  // -- OS 버전 및 하드웨어 타입 정보 얻기 위한 시스템 함수
  if (uname(&sysInfo) < 0) { 
    cout << "Error Occurred" << endl;
    return(-1);
  }

  if (strncasecmp(sysInfo.sysname, HPUX, strlen(HPUX)) == 0) {
    // -- HP 용 객체 생성 및 사용
    pFactory = HPCompilerFactory::CreateInstance();
  }
  else if (strncasecmp(sysInfo.sysname, SUNOS, strlen(SUNOS)) == 0) {
    // -- Sun 용 객체 생성 및 사용
    pFactory = SunCompilerFactory::CreateInstance();
  }
  else {
    // -- 지원 안하는 시스템 환경
    cout << sysInfo.sysname << endl;
    return(0);
  }

  Scanner *pScanner = pFactory->CreateScanner();
  Parser *pParser = pFactory->CreateParser();
  CodeGenerator *pCodeGenerator = pFactory->CreateCodeGenerator();
  Optimizer *pOptimizer = pFactory->CreateOptimizer();
}
```

* 일반적인 싱글톤 패턴의 경우에는 클래스 상속 구조 상에서 최상위 클래스의 Create Instance() 멤버 함수가 하위 클래스의 객체를 생성시켜주는 형태지만, Abstract Base Class인 ComplierFactory 클래스가 CreateInstance() 멤버 함수를 가지지 않고 그 역할을 각각의 하위 클래스가 대행한다.

### 복제를 통해 제품 객체를 생성하는 방법

### 새로운 제품 종류의 추가 시 문제 해결 방법

* 특정 제품군에 속하는 제품 객체를 생성하는 프로그램을 한 곳으로 모아 새로운 제품군의 추가가 용이하게 만든 클래스 설계가 Abstract FActory 패턴이다.
* Factory 클래스에 의해 생성되는 제품의 종류가 새로 추가되어야 하는 상황이 발생했다고 하자.
    * 새로운 종류의 제품에 해당하는 클래스도 추가해야 겠지만 각각의 Factory 클래스들도 모두 수정해야 한다. 새로운 종류의 제품을 생성하기 위한 멤버 함수를 각 Factory 클래스에 추가해야하기 때문이다.
* Abstract Factory 패턴의 가장 큰 단점은 Factory 클래스가 새로운 종류의 제품을 생성하도록 추가로 요청 받았을때 이를 반영하기 위해 모든 Factory 클래스들을 수정해야한다는 사실이다.
* 이 단점을 해결하기 위해선
    * 생성할 제품의 종류에 무관하게 Factory 클래스가 사용하는 멤버 함수를 하나로 통일시키면 된다.
    * 즉 Factory 클래스에서 객체 생성을 위한 모든 멤버 함수를 CreateProduct() 와 같은 하나의 멤버 함수로 통일시키면 된다.
* 최종 샘플

``` c++
class Product {
  public :
    virtual Product* Clone()=0;
};

class HPScanner : public Product {
  public :
    Product* Clone() { return new HPScanner(*this); }
};

class HPParser : public Product {
  public :
    Product* Clone() { return new HPParser(*this); }
};

class HPCodeGenerator : public Product {
  public :
    Product* Clone() { return new HPCodeGenerator(*this); }
};

class HPOptimizer : public Product {
  public :
    Product* Clone() { return new HPOptimizer(*this); }
};

class SunScanner : public Product {
  public :
    Product* Clone() { return new SunScanner(*this); }
};

class SunParser : public Product {
  public :
    Product* Clone() { return new SunParser(*this); }
};

class SunCodeGenerator : public Product {
  public :
    Product* Clone() { return new SunCodeGenerator(*this); }
};

class SunOptimizer : public Product {
  public :
    Product* Clone() { return new SunOptimizer(*this); }
};

// -- 추가 필요 부분 시작
class HPErrorHandler : public Product {
  public :
    Product* Clone() { return new HPErrorHandler(*this); }
};

class SunErrorHandler : public Product {
  public :
    Product* Clone() { return new SunErrorHandler(*this); }
};
// -- 추가 필요 부분 끝

class CompilerFactory {
  public :
    virtual Product* CreateProduct(Product *p) = 0;
};

class HPCompilerFactory : public CompilerFactory {
  public :
    Product* CreateProduct(Product *p) { 
       return p->Clone();
    }
};

class SunCompilerFactory : public CompilerFactory {
  public :
    Product* CreateProduct(Product *p) { 
      return p->Clone();
    }
};

CompilerFactory *pFactory;

int main() {
  Product *pScanner, *pParser, *pCodeGenerator, *pOptimizer;
  Product *pErrorHandler; // -- 추가 필요
  struct utsname sysInfo;

  // -- OS 버전 및 하드웨어 타입 정보 얻기 위한 시스템 함수
  if (uname(&sysInfo) < 0) { 
    cout << "Error Occurred" << endl;
    return(-1);
  }

  if (strncasecmp(sysInfo.sysname, HPUX, strlen(HPUX)) == 0) {
    // -- HP 용 객체 생성 및 사용
    pScanner = new HPScanner;
    pParser = new HPParser;
    pCodeGenerator = new HPCodeGenerator;
    pOptimizer = new HPOptimizer;
    pErrorHandler = new HPErrorHandler; // -- 추가 필요

    pFactory = new HPCompilerFactory;
  }
  else if (strncasecmp(sysInfo.sysname, SUNOS, strlen(SUNOS)) == 0) {
    // -- Sun 용 객체 생성 및 사용
    pScanner = new SunScanner;
    pParser = new SunParser;
    pCodeGenerator = new SunCodeGenerator;
    pOptimizer = new SunOptimizer;
    pErrorHandler = new SunErrorHandler; // -- 추가 필요

    pFactory = new SunCompilerFactory;
  }
  else {
    // -- 지원 안하는 시스템 환경
    cout << sysInfo.sysname << endl;
    return(0);
  }

  Product *pNewScanner = pFactory->CreateProduct(pScanner);
  Product *pNewParser = pFactory->CreateProduct(pParser);
  Product *pNewCodeGenerator = pFactory->CreateProduct(pCodeGenerator);
  Product *pNewOptimizer = pFactory->CreateProduct(pOptimizer);
  // -- Product *pNewErrorHandler = pFactory->CreateProduct(pErrorHandler);
}
```

* Abstract Factory 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-01-01.png)

## 4\. 부분 부분 생성을 통한 전체 객체 생성 문제 : Builder 패턴

* 때로는 클래스의 생성자를 불러서 객체를 한번에 생성하는 것이 아니라 객체를 구성하는 부분 부분을 따로 생성하고 이를 조합해서 전체 객체를 생성해주는 것이 보다 효과적일 때가 있다.
* 객체의 생성을 객체의 자료형인 클래스의 생성자를 통해 직접 수행하지 않고, 객체를 구성하는 부분부분을 생성한 다음, 이를 조합해서 원하는 객체를 생성하고자 할때 어떤 식으로 객체를 생성하는 것이 좋을지 살펴 보자

### 문제 사례

* 전 셰계로 수출되는 전자 제품을 만드는 회사에서 여러 제품에 대한 메뉴얼을 수출할 각 국가의 언어로 작성해야한다. 이때 한국어로 된 메뉴얼을 입력하면 영어, 일본어 등으로된 메뉴얼을 만들어 내는 것을 목적으로 한다. 메뉴얼에 사용되는 문장은 평서문, 의문문, 명령문으로만 구성되어 있으며 자동번역 기능은 문장 단위로 번역하면 된다고 새악ㄱ해보자.

### 예시

* 빌더 패턴이 적용된 예시

``` c++
const int UNDEF_SENTENCE=0;
const int NORMAL_SENTENCE=1;
const int INTERROGATIVE_SENTENCE=2;
const int IMPERATIVE_SENTENCE=3;

class Sentence {
  public :
    Sentence() { data_ = ""; type_ = UNDEF_SENTENCE;}

    int GetType() { return type_; }
    string GetString() { return data_; }

    void SetSentenceData(string s) { 
      SetSenteceType(s);
      data_ = s; 
    }

  protected :
    void SetSenteceType(string s) {
      // -- 문장 유형을 판단하여 type_에 설정, default는 평서문
      type_ = NORMAL_SENTENCE;
    }

  private :
    string data_;
    int type_;
};

class Manual {
  public :
    string GetContents() { return contents_; }
    void AddContents(string s) { contents_ += s; }

  private : 
    string contents_;
};

class Translator {
  public :
    Manual GetResult() { return result_; }

    virtual void TransNormalSentence(string s)=0;
    virtual void TransInterrogativeSentence(string s)=0;
    virtual void TransImperativeSentence(string s)=0;

  protected :
    Manual result_;
};

class EnglishTranslator : public Translator {
  public : 
    void TransNormalSentence(string s)
    {
      string output;
      // -- s를 영어로 번역
      result_.AddContents(output);
    }

    void TransInterrogativeSentence(string s)
    {
      string output;
      // -- s를 영어로 번역
      result_.AddContents(output);
    }

    void TransImperativeSentence(string s)
    {
      string output;
      // -- s를 영어로 번역
      result_.AddContents(output);
    }
};

class JapaneseTranslator : public Translator {
  public : 
    void TransNormalSentence(string s)
    {
      string output;
      // -- s를 일어로 번역
      result_.AddContents(output);
    }

    void TransInterrogativeSentence(string s)
    {
      string output;
      // -- s를 일어로 번역
      result_.AddContents(output);
    }

    void TransImperativeSentence(string s)
    {
      string output;
      // -- s를 일어로 번역
      result_.AddContents(output);
    }
};

class FrenchTranslator : public Translator {
  public : 
    void TransNormalSentence(string s)
    {
      string output;
      // -- s를 프랑스어로 번역
      result_.AddContents(output);
    }

    void TransInterrogativeSentence(string s)
    {
      string output;
      // -- s를 프랑스어로 번역
      result_.AddContents(output);
    }

    void TransImperativeSentence(string s)
    {
      string output;
      // -- s를 프랑스어로 번역
      result_.AddContents(output);
    }
};

class Director {
  public : 
    void DoTranslate(char* pInFile, Translator& t)
    {
      ifstream ifs(pInFile);
      if (!ifs) {
        cout << "Can't Open File : " << pInFile << endl;
        return;
      }

      Sentence next;
      while (!(next=GetSentence(ifs)).GetString().empty()) {
        switch (next.GetType()) {
          case NORMAL_SENTENCE : // -- 평서문
            t.TransNormalSentence(next.GetString());
            break;
          case INTERROGATIVE_SENTENCE : // -- 의문문
            t.TransInterrogativeSentence(next.GetString());
            break;
          case IMPERATIVE_SENTENCE : // -- 명령문
            t.TransImperativeSentence(next.GetString());
            break;
          default :
            cout << "Untranslatable sentence type" << endl;
            return;
        }
      }
    }

  protected:
    Sentence GetSentence(ifstream& ifs) 
    {
      int c;
      string s;
      Sentence sentence;
      while ((c=ifs.get()) != EOF) {
        s += c;
        if (c == '?' || c == '.') 
          break;
      }
      sentence.SetSentenceData(s);
      return sentence;
    }
};

void main()
{
  Director d;
  EnglishTranslator t;
  d.DoTranslate("input.txt", t);

  Manual out = t.GetResult();
  cout << out.GetContents() << endl;
}
```

* Builder 패턴의 일반적인 클래스 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-01-02.png)


## 5\. 대행 함수를 통한 객체 생성 문제

### 문제 사례

* 윈도우 운영체제에서 문서 파일을 더블클릭한 경우를 생각해 보자. 문서 파일 이름의 확장자에 따라 적절한 응용 프로그램이 실행될것이다.
    * hwp면 한글이, doc면 ms 워드가.
    * 이 동작을 객체지향 관점에서 생각해보면 두 가지 종류의 객체가 생성되어야 함을 알 수 있다.
        * 하나는 응용 프로그램 객체이고
        * 다른 하나는 문서 파일에 대한 객체일 것이다.
* 그렇다면 두 가지 종류의 객체는 누구에 의해서 생성되는 것이 바람직할까?
    * 응용 프로그램 객체는 OS에 의해 생성해야하고
    * 문서 파일에 대한 객체는 응용 프로그램에 의해서 생성하는게 맞다.
* 어떻게 운영체제가 호출하는 인터페이스는 응용 프로그램에 상관없이 동일하게 유지되면서, 응용 프로그램마다 생성되는 객체는 각 응용 프로그램마다 고유하게 만들어지도록 할 수 있을까?

### Factory Method 패턴

* 각 응용프로그램 별로 NewDocument( ) 멤버 함수를 구현하는 방식은 응용 프로그램의 수가 증가함에 따라 비용 이슈와 멤버함수 의 구현 내용을 변경하고자 하는 경우 각 응용 프로그램별 클래스를 모두 수정해야하는 문제가 있을 것이다.
* 각 응용 프로그램 클래스 마다 NewDocument( ) 멤버 함수를 따로 구현하는데 문제의 원인이 있다. 이를 위한 가장 좋은 장소는 Application 클래스인데 모든 응용 프로그램 클래스를 대표할 수 있기 때문이다.
* 응용 프로그램 마다 구현했던 NewDocument( ) 멤버 함수를 하나로통합해서 구현하려면 각 응용 프로그램 클래스마다 구현 내용이 달랐던 문서 파일 객체 생성 부분을 응용 프로그램 클래스에 상관없이 동일하도록 수정해야한다.
    * 이를 위한 좋은 방법은 문서 파일 객체 생성 부분을 별도의 멤버 함수로 정의하고 이를 NewDocument( ) 멤버 함수에서 불러 사용하게 만들어주는 것이다.
    * 즉 CreateDocument( ) 와 같이 문서 파일 객체 생성을 대행해주는 멤버 함수를 새로 정의하고 NewDocument( ) 멤버 함수에서는 이를 불러쓰게 만들어 주는 것이다.
    * 그런데 이 방법을 쓰려면 2가지를 더 해결해야 한다.
        * CreateDocument( ) 멤버 함수에 의해 실제 생성되는 문서 파일 객체는 각 응용 프로그램 클래스 마다 서로 달라야 하는것
        * CreateDocument( ) 멤버 함수에 의해 생성된 문서 파일 객체는 응용 프로그램 클래스에 상관없이 NewDocument( ) 멤버 함수 내에서는 동일하게 취급되어야 한다는 것.
        * 이 두 가지를 어떻게 해결 할 수 있을까?
        * 각 응용프로그램 클래스마다 CreateDocument( ) 멤버 함수에 의해 생성되는 문서 파일 객체가 다르도록 만들어줄 방법은 한 가지 뿐이다.
            * 각 응용 프로그램 클래스마다 CreateDocument( ) 멤버 함수를 직접 구현하는 것이다.
            * 물론 이때 각 응용 프로그램 클래스에서 구현하는 CreateDocument( ) 멤버 함수는 Application 클래스의 CreateDocument( ) 멤버 함수를 Overriding 해서 정의하는 형태여햐 할 것이다.
* 한편 각 응용프로그램 클래스마다 서로 다른 문서 파일 객체를 생성하더라도 NewDocument( ) 멤버 함수에서는 이들을 동일하게 취급할 수 있기 위해서는 응용 프로그램 클래스에서 생성 가능한 모든 문서 파일 객체를 대표할 수 있는 별도의 클래스 정의가 필요할 것이다.
* 이처럼 객체를 생성하되 직접 객체 생성자를 호출해서 객체를 생성하는 것이 아니라 대행 함수를 통해 간접적으로 객체를 생성하는 방식을 Factory Method 패턴이라고 한다. 또한 이 객체 생성을 대행해주는 함수를 Factory Method 라고 한다.
    * 통상적으로 클래스 상속 구조와 같이 사용되는데, 여기서 상위 클래스는 생성하는 객체의 종류와 상관없이 공통적으로 사용할수 있는 처리 모듈을 포함하게 된다.

### 샘플 코드

* 운영체제에 의해 응용 프로그램 객체가 생성되고 응용 프로그램 개체에 의해 다시 문서 파일 객체가 생성되는 방식을 Factory Mthod 패턴으로 한 샘플 코드이다.

``` c++
class Document {
  public :
    virtual bool Open(char* pFileName) = 0;
};

class HwpDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- HWP 고유 프로세싱
      return true;
    }
};

class MsWordDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- MS-Word 고유 프로세싱
      return true;
    }
};

class Application {
  public :
    void NewDocument(char* pFileName) {
      Document *pDoc = CreateDocument();
      docs_[pFileName] = pDoc;
      pDoc->Open(pFileName);
    }

    virtual Document* CreateDocument() = 0;

  private :
    map<string, Document *> docs_;
};

class HwpApplication : public Application {
  public :
    Document* CreateDocument() {
      return new HwpDocument;
    }
};

class MsWordApplication : public Application {
  public :
    Document* CreateDocument() {
      return new MsWordDocument;
    }
};

void		
main()
{
  HwpApplication hwp;
  hwp.NewDocument("input.hwp");
}
```

* NewDocument( ) 멤버 함수가 Application 클래스에만 정의 되었고 다른 클래스에는 정의되어있지 않다
* 반면 CreateDocument( ) 멤버 함수는 Applicatino 클래스에서는 완전 가상함수(pure virtual function)정의하였지만 하위는 실제로 구현하고 있다.

### 추가 고려 사항

* 기본적으로 Factory Method 패턴은 생성할 객체의 종류가 추가되면 이를 생성시켜주기 위한 클래스를 따로 정의하는 식이다.
    * 그러나 생성해야할 객체의 종류가 늘어날 때마다 클래스를 계속 추가하는것은 못할 짓이다.
    * 이를 방지하기 위해서는 생성할 객체의 종류와 상관없이 동일한 클래스에서 객체를 생성해주는 형태로 변경이 필요하며, 이때 객체 생성을 담당하는 멤버 함수는 어떤 객체를 생설해줄지에 대한 인자로 전달받는 식이어야 한다.
    * 이에 대한 샘플을 보자.

``` c++
#define UNDEF_DOCUMENT        0
#define HWP_DOCUMENT          1
#define MSWORD_DOCUMENT       2
#define ANOTHER_DOCUMENT      3

class Document {
  public :
    virtual bool Open(char* pFileName) = 0;
};

class HwpDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- HWP 고유 프로세싱
      return true;
    }
};

class MsWordDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- MS-Word 고유 프로세싱
      return true;
    }
};

class AnotherDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- 알맞은 프로세싱
      return true;
    }
};

class Application {
  public :
    void NewDocument(char* pFileName) {
      Document *pDoc = CreateDocument(GetDocType(pFileName));
	  if (pDoc == NULL) exit(0);
      docs_[pFileName] = pDoc;
      pDoc->Open(pFileName);
    }

    virtual Document* CreateDocument(int docType) {
      switch (docType) {
        case HWP_DOCUMENT : 
          return new HwpDocument;
        case MSWORD_DOCUMENT : 
          return new MsWordDocument;
      }
      return 0;
    }

  protected : 
    int GetDocType(char *pFileName) {
      char *pExt = strrchr(pFileName, '.');
      if (strcmp(pExt, ".hwp") == 0)
        return HWP_DOCUMENT;
      else if (strcmp(pExt, ".doc") == 0)
        return MSWORD_DOCUMENT;
      else
        return UNDEF_DOCUMENT;
    }

  private :
    map<string, Document *> docs_;
};

class HwpApplication : public Application { };

class MsWordApplication : public Application { };

class AnotherApplication : public Application {
  public :
    Document* CreateDocument(int docType) {
      if (docType == ANOTHER_DOCUMENT)
        return new AnotherDocument;
      else // -- 자신이 생성할 수 없는 객체는 상위 클래스가 생성토록 위임
        return Application::CreateDocument(docType);
    }
};

void		
main()
{
  AnotherApplication another;
  another.NewDocument("input.dat");
}
```

* Applicatino 클래스의 CreateDocument( ) 멤버 함수에 인자를 둠으로 HwpApplication 이나 MsWordApplication 클래스 없이 HwpDocument 나 MsWordDocument 객체를 생성할 수 있음을 알 수 있다.
* C++와 같이 template 클래스를 허용하는 경우는 여러 종류의 객체를 생성하기 위해 각 객체의 종류마다 별도의 하위 클래스를 정의할 필요는 없으며 하나의 하위 클래스에 tmeplate 클래스 인자를 둠으로 여러 종류의 객체가 생성될 수 있도록 만들수 있다.
    * 이에 대한 예시를 보자.

``` c++
class Document {
  public :
    virtual bool Open(char* pFileName) = 0;
};

class HwpDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- HWP 고유 프로세싱
      return true;
    }
};

class MsWordDocument : public Document {
  public :
    bool Open(char* pFileName) {
      ifstream ifs(pFileName);
      if (!ifs)
        return false;
      // -- MS-Word 고유 프로세싱
      return true;
    }
};

class Application {
  public :
    void NewDocument(char* pFileName) {
      Document *pDoc = CreateDocument();
      docs_[pFileName] = pDoc;
      pDoc->Open(pFileName);
    }

  protected :
    virtual Document* CreateDocument() = 0;

  private :
    map<string, Document *> docs_;
};

template <class DocType>
class ConcreteApplication : public Application {
  protected :
    Document* CreateDocument() {
      return new DocType;
    }
};

void
main()
{
  ConcreteApplication<HwpDocument> hwp;
  hwp.NewDocument("input.hwp");
}
```

* 다만 문제처럼 어차피 응용 프로그램 마다 별도의 클래스가 필요하다면 굳이 이런 tmeplate 기법은 필요가 없다.
* 일반적인 구조
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-01-03.png)


## 6\. 복제를 통한 객체 생성 문제

* 객체를 생성하는 방법중 대행해주는 함수를 통해 객체를 생성하거나, 객체를 구성하는 부분 부분을 먼저 생성한 후 이를 조합해서 생성하는 방법도 있지만, 이미 생성된 객체를 복제하여 새로운 객체를 생성하는 방법도 있다.

### 문제 사례

* 그래팩 편집기를 개발한다고 가정해보자. 왼쪽에 팔레트가 있고 이중 하나를 선택하여 끌어다 놓으면 문서의 일부분으로 동작하는 방식을 생각해보자.
* 이를 객체 지향으로 생각해 보면, 문서에 추가되는 각각이 객체로 생성될 것이고 , 문서는 이들을 포함하는 형태가 될 것이다.
* 여기서 고민해볼 사항은 각각의 그래픽 요소들이 문서에 추가될 때 어떤 방식으로 객체를 생성해 주는것이 가장 쉬우면서 추후의 변동 등에 유연하게 대처할수 있을까 이다.

### Prototype 패턴

* 직접 객체를 생성하는 방식을 생각해 볼 수 있는데, 그래픽 요소의 개수가 많아지면 그만큼 소스 코드 관리가 불편할 것이다.아마도 switch 문의로 일일이 구분하여 생성할 것이다.
    * 새로운 형태의 도형이 추가되면 이를 반영하기 위해 코드 수정이 불가피 할 것이다.
* switch 와 같은 비교 문장을 포함하지 않는 형태의 접근 방식이 문제를 해결할 바람직할 방법일 것인다.
* 그런데, 이런 비교 문장이 없을 경우 이용자가 선택한 그래픽 요소에 대해 객체를 생성하려고 할 때 객체의 자료형을 결정할 수 없다는 문제가 있다. 
* 객체를 생성하는 시점에 객체의 자료형을 정해주지 않아도 되는 생성되는 객체의 자료형이 결정되도록 하려면 어떻게 해야 할까?
    * 기존의 객체를 복제해서 새로운 객체를 생성한다면 어떻게 될까?
* 이처럼 객체를 생성할때 기존 객체를 복제해서 객체를 생성하는 방식이 Prototype  패턴이다. 
* 위의 예시에서는 Graphic 및 하위긔 클래스들을 모두 Clone( ) 라는 멤버 함수를 가지게 될 것이다. 
    * 팔레트에서는 각 그래픽 요소에 대응하는 객체를 미리 생성해서 관리하는 형태여야 할 것이다. 

### 샘플코드

```c++
class Position {
  public : 
    Position() {}
    Position(int x, int y) { x_ = x; y_ = y; }
    int x_, y_;
};

class Graphic {
  public :
    virtual void Draw(Position& pos) = 0;
    virtual Graphic* Clone() = 0;
};

class Triangle : public Graphic {
  public :
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Triangle(*this); }
};

class Rectangle : public Graphic {
  public : 
    void Draw(Position& pos) {}
    Graphic* Clone() { return new Rectangle(*this); }
};

class GraphicComposite : public Graphic {
  public : 
    void Draw(Position& pos) {}
    Graphic* Clone() { 
      GraphicComposite *pGraphicComposite = new GraphicComposite(*this);
      list<Graphic*>::iterator iter1;
      list<Graphic*>::iterator iter2;

      iter2 = pGraphicComposite->components_.begin();
      for (iter1 = components_.begin(); iter1 != components_.end(); iter1++) {
        Graphic* pNewGraphic = (*iter1)->Clone();
        *iter2 = pNewGraphic;
        iter2++;
      }

      return pGraphicComposite;
    }

  private : 
    list<Graphic*> components_;
};

class Document {
  public :
    void Add(Graphic* pGraphic) {}
};

class Mouse {
  public :
    bool IsLeftButtonPushed() { 
      static bool isPushed = false;
      // -- GUI 함수 활용 Left Button 상태 체크
      isPushed = ! isPushed;
	  return isPushed;
    }

    Position GetPosition() {
      Position pos;
      // -- GUI 함수 활용 현재 마우스 위치 파악
      return pos;
    }
};

Mouse _mouse; // -- Global Variable

class GraphicEditor {
  public : 
    void AddNewGraphics(Graphic* pSelected) {
      Graphic* pObj = pSelected->Clone();
      while (_mouse.IsLeftButtonPushed()) {
        Position pos = _mouse.GetPosition();
        pObj->Draw(pos);
      }

      curDoc_.Add(pObj);
    }

  private :
    Document curDoc_;
};

class Palette {
  public :
    Palette() {
      Graphic* pGraphic = new Triangle;
      items_[1] = pGraphic;

      pGraphic = new Rectangle;
      items_[2] = pGraphic;

      // -- 필요한 만큼 등록 
    }

    void RegisterNewGraphic(Graphic* pGraphic) {
      items_[items_.size()+1] = pGraphic;
    }

    Graphic* GetSelectedObj() {
      return items_[GetItemOrder()];
    }

    int GetItemOrder() {
      int i=1;
      Position curPos = _mouse.GetPosition();
      // -- 현재 마우스 위치가 몇 번째 항목을 지정하는 지 판별
      return i;
    }

  private :
    map<int, Graphic*> items_;
};

void
main()
{
  Palette palette;
  GraphicEditor ged;

  ged.AddNewGraphics(palette.GetSelectedObj());
}
```

* 위의 코드에서 아래내용을 주의를 기울이자. 
    * 각 클래스의 객체를 복사해주는 Clone( ) 멤버 함수의 구현방식을 보면 Deep Copy 가 이루어지도록 하는 것이다. 
    * Palette 클래스에서 새로운 그래픽 요소를 동적으로 등록하기 위한 registerNewGraphic( ) 멤버 함수를 제공하는 점이다. 
* Prototype 패턴의 일반적인 클래스 구조 및 가상 코드
![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-01-04.png)


## 7\. 최대 N개로 객체 생성을 제한하는 문제
* 굳이 정리하지 않음.

## 8\. 객체 생성을 위한 디자인 패턴 정리
* 굳이 정리하지 않음.