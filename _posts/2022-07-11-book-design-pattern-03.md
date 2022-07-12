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


# PART 4. 행위 수행 개선을 위한 디자인 패턴

## 17\. 수행 가능 객체까지 요청 전파 문제
### 문제 사례 설명
* GUI 환경에서 이용자에게 도움말을 제공하는 시스템을 개발한다고 가정해 보자. 
* 이용자가 처한 상황을 시스템이 판별해서 이용자에게 알맞는 도움말을 제공하는 형태를 Context Sensitive 도움말 시스템이라고 한다. 그런데 실제로 모든 사용자 인터페이스 객체마다 적절한 도움말을 정의할 수는 없다. 현재 상황에 정의된 도움말이 없을 경우에는 어떤 식으로 도우말을 보여주는 것이 좋을까?

### Chain of Responsibility 패턴
* 사용자 인터페이스 객체 중에는 이용자가 처한 상황과는 무관하게 일반적으로 사용되는 객체들도 많이 있기에 모든 사용자 인터페이스 객체마다 도움말을 지정할 수는 없다.그렇지만 지정되지 않는 사용자 인터페이스 객체에 대해서도 도움말이 요청될 수 있다. 이 같은 경우 Context sensitive 도움말 제공 시스템은 어떤 식으로 이용자가 원하는 도움말을 제공해줄 수 있을까?
* 복잡하고 귀찮은 기능이라고해서 무작정 전담 객체를 정의해서 사용하는 방식은 또 다른 문제를 야기하기도 한다. 도움말이 지정되지 않은 사용자 인터페이스 객체가 활성화되어 있는 상태에서 이용자가 도움말을 요청하면 요청을 받은 사용자 인터페이스 객체는 자신을 포함하고 있는 또 다른 사용자 인터페이스 객체에게 다시 요청을 처리하도록 자신이 받은 요청을 전달하는 것이다. 
* 그리고 그 객체도 없으면 자신을 감싸고 있는 또 다른 사용자 인터페이스 객체에게 요청을 전달해서 최종적으로 도움말이 지정된 객체가 자신에게 지정된 도움말을 보여주도록 하는 것이다. 
* 객체들간에 도움말 요청이 전달될 때 사용하는 인터페이스가 동일해야 한다. 
* 각 객체들이 도움말 요청을 처리하지 못할 경우 이를 처리하도록 도움말 요청을 전담할 객체가 지정될 수 있어야 한다. 
* 이처럼 객체들간의 관계에 의해 체인을 구성해두고 특정 객체에게 요청이 전달될 경우, 해당 객체가 요청을 처리하지 못하면 체인 상에 있는 다른 객체에게 요청을 대신 처리하게 하는 방식의 설계를 Chain of Responsibilty 패턴이라고 한다. 

### 샘플 코드

```c++
class HelpHandler {
  public :
    HelpHandler(HelpHandler* pObj=0, string helpMsg="") {
      pSuccessor_ = pObj; helpMsg_ = helpMsg; 
    }
    virtual void SetHandler(HelpHandler* pObj, string helpMsg) {
      pSuccessor_ = pObj; helpMsg_ = helpMsg; 
    }
    virtual bool HasHelp() { return !helpMsg_.empty(); }
    virtual void HandleHelp() {
      if (pSuccessor_) pSuccessor_->HandleHelp();
    }
    virtual void ShowHelpMsg() {
      // -- helpMsg_를 보여준다.
      cout << helpMsg_ << endl;
    }
  protected:
    string helpMsg_;
  private :
    HelpHandler* pSuccessor_;
};

class Widget : public HelpHandler {
  protected :
    Widget(Widget* pObj, string helpMsg="") : HelpHandler(pObj, helpMsg) {
      pParent_ = pObj;
    }
  private :
    Widget* pParent_;
};

class Button : public Widget {
  public : 
    Button(Widget* pObj, string helpMsg="") : Widget(pObj, helpMsg) {}
    virtual void HandleHelp() {
      if (HasHelp()) ShowHelpMsg();
      else HelpHandler::HandleHelp();
    }
    virtual void ShowHelpMsg() {
      // -- helpMsg_ 내용을 Button 도움말 양식에 맞추어 보여준다.
      cout << helpMsg_ << endl;
    }
};

class Dialog : public Widget {
  public : 
    Dialog(HelpHandler *pObj, string helpMsg="") : Widget(0) {
      SetHandler(pObj, helpMsg);
    }
    virtual void HandleHelp() {
      if (HasHelp()) ShowHelpMsg();
      else HelpHandler::HandleHelp();
    }
    virtual void ShowHelpMsg() {
      // -- helpMsg_ 내용을 Dialog 도움말 양식에 맞추어 보여준다.
      cout << helpMsg_ << endl;
    }
};

class Application : public HelpHandler {
  public : 
    Application(string helpMsg) : HelpHandler(0, helpMsg) {}
    virtual void HandleHelp() {
      ShowHelpMsg();
    }
    virtual void ShowHelpMsg() {
      // -- helpMsg_ 내용을 보여준다.
      cout << helpMsg_ << endl;
    }
};

void
main() 
{
  Application *pApp = new Application("Application Help");
  Dialog* pDialog = new Dialog(pApp, "Dialog Help");
  Button* pButton = new Button(pDialog);

  pButton->HandleHelp();
}
```

* Button 객체에 해대 handleHelp( ) 멤버 함수가 호출되었지만, 실제 결과는 Dialog 객체에 의해 설정된 도움말 메시지가 출력되는 것을 확인할 수 있다. 


## 18\. 수행할 작업의 일반화 문제
### 문제 사례 설명
* 웹으로 게시판 서비스를 제공하는 프로그램을 작성한다고 가정해보자. 게시판에서 제공하는 서비스 항목이 처음부터 명확히 정해지지 않고 개발 도중에 점차 추가될 가능성이 많다고 하면 어떻게 하면 기존에 개발해 두었던 부분들을 수정하지 않고 쉽게 새로운 서비스 항목을 추가할 수 있겠는가?

### Command 패턴
* 서비스 항목이 추가된다는 것은 게시판 프로그램으로 전달되는 요청의 종류가 추가된다는 것이며 이럴때의 설계는 어떻게 하는 것이 좋을까?
* if, else 문으로 요청을 처리한다고 하면 Client 에서 요청을 처리할 객체를 일일이 알고 있고, 새로운 종류의 요청 처리가 필요할때마다 Client 를 수정해야하는 문제가 있다. 
* 요청을 처리할 객체들의 클래스를 모두 동일한 상속 구조에 포함시켜 정의하는 방법이 있다.. (동일한 인터페이스를 가지도록 만들자. )
    * 이것은 별다른 관련이 없는데도 같은 상속 구조하에서 동일한 자료형으로 다루어지는 문제가 있을 수 있다. 
* 다른 방법으로 Client 가 하던 역할을 대신하는 객체를 두는 것이다. 요청의 종류에 따라 처리 객체를 지정하는 일을 대신 수행하는 객체를 정의하고 Client 가 하던 역할을 모두 이 객체에게 위임시키는 것이다. 
* 요청의 종류에 따라 분기하는 방식의 두 가지 문제를 해결하기 위한 방법을 종합해보면
    * 별도의 클래스 상속 구조를 정의해서 활용하는 것이다.
* 이처럼 요청과 그 요청을 처리할 객체를 중계하기 위한 클래스 상속 구조를 Command 패턴이라고 한다. 
    * Command 클래스는 요청을 처리할 객체를 내부적으로 저장, 관리하면서 요청이 주어졌을때 요청을 처리할 객체의 멤버 함수를 불러주는 역할을 한다. 
    * 이때 요청을 처리할 객체는 Command 클래스의 객체가 생성되는 시점에 지정되며 Client 가 요청을 전달하는 방식은 Command 및 그 하위 클래스들이 제공하는 공통된 인터페이스를 통해 이루어진다. 

### 샘플 코드

```c++

static const int NA_POS = -1;

#define CMD_NAME    "cmd"

#define LOGIN_VAL   "login"
#define BBSLIST_VAL "bbslist"
#define BBSREAD_VAL "bbsread"

class Request {
  public :
    string GetValue(string name) {
      return nvList_[name];
    }
    void SetNameValue(string name, string value) {
      nvList_[name] = value;
    }
  private :
    map<string, string> nvList_;
};

class RequestParser {
  public :
    bool GetRequest(string input, Request& req) {
      string key, value, str;
      int start = 0, pos = 0;
      while (pos != NA_POS) {
        pos = input.find("=", start);
        if (pos == NA_POS) continue;

        key = input.substr(start, pos-start);
        start = pos+1;

        pos = input.find("&", start);
        if (pos == NA_POS)
          value = input.substr(start, input.length()-start);
        else 
          value = input.substr(start, pos-start);
        start = pos+1;

        if (! key.empty()) {
          req.SetNameValue(DecodeString(key), DecodeString(value));
          key = "";
        }
      }
	  return true;
    }
  protected :
    string DecodeString(string s) {
      string output;
      int len = s.length();
      for (int i=0; i < len; i++) {
        if (s[i] == '+')
          output += ' ';
        else if (s[i] == '%') {
          const char *pStr = s.data()+i+1;
          char ch = Hex2Digit(pStr);
          output += ch;
          i += 2;
        }
        else 
          output += s[i];
      }
      return output;
    }
    unsigned int Hex2Digit(const char* hex) {
      register char digit;
      digit = (hex[0] >= 'A' ? ((hex[0] & 0x4F) - 'A')+10 : (hex[0] - '0'));
      digit <<= 4;
      digit += (hex[1] >= 'A' ? ((hex[1] & 0x4F) - 'A')+10 : (hex[1] - '0'));

      return digit;
    }
  private :
};

// ----------------------------------------
class UserManager {
  public :
    bool CheckPasswd(Request& req) {
      cout << "Passwd OK" << endl;
      return true;
    }
};

class BBS {
  public :
    void DisplayList(Request& req) {
      cout << "Display BBS List" << endl;
    }
    void DisplayItem(Request& req) {
      cout << "Display BBS Item" << endl;
    }
};

// ----------------------------------------
class Command {
  public :
    virtual void Execute(Request& req) = 0;
};

class LoginCommand : public Command {
  public :
    LoginCommand(UserManager *pUserMan) {
      pUserMan_ = pUserMan;
    }
    void Execute(Request& req) {
      pUserMan_->CheckPasswd(req);
    }
  private :
    UserManager *pUserMan_;
};

class ListCommand : public Command {
  public :
    ListCommand(BBS *pBbs) {
      pBbs_ = pBbs;
    }
    void Execute(Request& req) {
      pBbs_->DisplayList(req);
    }
  private :
    BBS *pBbs_;
};

class ReadCommand : public Command {
  public :
    ReadCommand(BBS *pBbs) {
      pBbs_ = pBbs;
    }
    void Execute(Request& req) {
      pBbs_->DisplayItem(req);
    }
  private :
    BBS *pBbs_;
};

// ----------------------------------------
// Global Instance
// ----------------------------------------
BBS _bbs;
UserManager _userMan;
map<string, Command *> _req2cmd;

void
RegisterCommand() {
  _req2cmd[LOGIN_VAL] = new LoginCommand(&_userMan);
  _req2cmd[BBSLIST_VAL] = new ListCommand(&_bbs);
  _req2cmd[BBSREAD_VAL] = new ReadCommand(&_bbs);
}

void
main()
{
  string input;
  RequestParser parser;

  RegisterCommand();

  while (1) {
    cin >> input;

    Request req;
    parser.GetRequest(input, req);

    string cmd = req.GetValue(CMD_NAME);
    Command *pCmd = _req2cmd[cmd];
    if (pCmd != NULL) 
      pCmd->Execute(req);
    else 
      cout << "Not Available Command : " << cmd << endl;
  }
}

```

* 각각의 Command 클래스들은 요청 처리시 필요한 객체에 대한 정보를 별도의 자료구조를 참조하고 있으며 Client 가 요청을 전달한 인터페이스도 Command 클래스의 종류에 상관없이 Execute( )라는 멤버 함수를 공통으로 이용함을 알 수 있다. 또한 비교 문장을 없애기 위해 미리 각 요청별로 Command 클래스 객체를 등록해 두고 사용하는 형태를 취하고 있다. 
* Command 패턴의 일반적인 클래스 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-03-01.png)


## 19\. 간단한 문법에 기반한 검증 및 작업 처리 문제
### 문제 사례 설명
* 문서 파일에서 특정 문자열 패턴이 존재하는지를 검사해주는 프로그램을 작성한다고 생각해보자. 이럴경우 입력된 문자열 패턴이 관련 문법을 준수하는지 검증한뒤 문서 파일에서 입력된 문자열 패턴을 실제로 검색해보는 과정을 거칠 것이다. 이 같은 작업들을 수행하기 위해 적절한 설계는 무엇일까?

### Interpreter 패턴
* 프로그램이 수행해야 할 두 개의 작업을 각각 별개로 독립된 클래스를 이용해서 처리하는 것은 어떻까? 노력대비 효용 측면에서 비경제적인 문제가 있을 것이다. 구문 트리, 파싱, 등등의 자료구조가 필요하다. 
* 비교적 간단한 문법에 대해 문법 자체를 클래스로 정의해서 문법 검사와 함께 필요한 기능도 별도의 자료구조 없이 효율적으로 수핼할 수 있게 만들어주는 클래스 구조를 Interpreter 패턴이라고 한다. 

### 샘플 코드

```c++

static const int NA_POS = -1;

class RegularExp {
  public :
    virtual bool Match(string context) = 0;
};

class LiteralExp : public RegularExp {
  public :
    LiteralExp(const char *pStr) : literal_(pStr) { }
    LiteralExp(const string str) : literal_(str) { }

    bool Match(string context) {
      string str;
      ifstream ifs(context.data());
      while (! ifs.eof()) {
        ifs >> str;
        if (literal_ == str) 
          return true;
      }
      return false;
    }
  private :
    string literal_;
};

class OrExp : public RegularExp {
  public :
    OrExp(RegularExp *pExp1, RegularExp *pExp2)
      : pOrExp1_(pExp1), pOrExp2_(pExp2) { }

    bool Match(string context) {
      if (pOrExp1_->Match(context))
        return true;
      else {
        return pOrExp2_->Match(context);
      }
    }
  private :
    RegularExp *pOrExp1_;
    RegularExp *pOrExp2_;
};

class AndExp : public RegularExp {
  public : 
    AndExp(RegularExp *pExp1, RegularExp *pExp2)
      : pAndExp1_(pExp1), pAndExp2_(pExp2) { }

    bool Match(string context) {
      return pAndExp1_->Match(context) && pAndExp2_->Match(context);
    }
  private :
    RegularExp *pAndExp1_;
    RegularExp *pAndExp2_;
};

RegularExp * 
CreateRegularExp(string searchStr)
{
  int len = searchStr.length();
  if (len == 0) return NULL;
  else 
    cout << "===>" << searchStr << endl;

  int pos = searchStr.find_first_of("(&|");
  if (searchStr[pos] == '(') {
    int endParenPos = 0;
    int parenCnt = 1;
    for (int i = pos+1; i < len; i++) {
      if (searchStr[i] == '(') parenCnt++;
      else if (searchStr[i] == ')') parenCnt--;
      else {}

      if (parenCnt == 0) {
        int nextOpPos = searchStr.find_first_of("&|", i+1);
        if (nextOpPos != NA_POS) {
          if (searchStr[nextOpPos] == '&') 
            return new AndExp(CreateRegularExp(searchStr.substr(pos+1,i-pos-1)),
                          CreateRegularExp(searchStr.substr(nextOpPos+1, 
                                                          len-nextOpPos-1)));
          else 
            return new OrExp(CreateRegularExp(searchStr.substr(pos+1,i-pos-1)),
                          CreateRegularExp(searchStr.substr(nextOpPos+1, 
                                                          len-nextOpPos-1)));
        }
        else
          return CreateRegularExp(searchStr.substr(pos+1, i-pos-1));
      }
    }
    // -- searchStr 수식이 잘못된 것임
    return NULL;
  }
  else if (searchStr[pos] == '&') {
    if (pos >= len-1) return NULL;
    return new AndExp(CreateRegularExp(searchStr.substr(0, pos)), 
                  CreateRegularExp(searchStr.substr(pos+1, len-pos-1)));
  }
  else if (searchStr[pos] == '|') {
    if (pos >= len-1) return NULL;
    return new OrExp(CreateRegularExp(searchStr.substr(0, pos)), 
                  CreateRegularExp(searchStr.substr(pos+1, len-pos-1)));
  }
  else {
    // -- 앞뒤 White-space 제거
    string literal;
    strstream strm;
    strm << searchStr;
    strm >> literal;
    if (literal.empty())
      return NULL;

    return new LiteralExp(literal);
  }
}

void
main()
{
  string str;
  getline(cin, str);

  RegularExp *pRegExp = CreateRegularExp(str);
  if (pRegExp == NULL) {
    cout << "Search Pattern Error" << endl;
    exit(0);
  }

  if (pRegExp->Match("data.txt"))
    cout << "Found the search string" << endl;
  else
    cout << "Not Exist the search string" << endl;
}

```

* CreateReqularExp( ) 함수는 간단히 입력된 문자열 패턴을 파싱해서 구문 트리를 생성해주는 역할을 한다. 


## 20\. 동일 자료형의 여러 객체에 대한 순차적 접근 문제
### 문제 사례 설명
* 동일한 자료형의 객체가 여러 개 모여 리스트나 트리와 같은 자료구조를 형성하고 있다고 가정해 보자. 이 각 객체들을 순차적으로 접근할 필요가 있을때 적당한 방법은 무엇일까?

### Iterator 패턴
* 리스트를 구성하는 각 항목들이 다음 항목을 가리키게 포인터를 두고 이를 통해 리스트 내의 각 항목들을 접근하는 것은 리스트 내의 각 항목들이 어떤 형태의 자료구조를 가졌는지 미리 알고 있어야 적용 가능할 것이다. 그렇다면 리스트 내부의 자료구조와 무관하게 리스트 내의 각 항목들을 차례로 접근할 수 있는 객체지향적 방법은 무엇일까?
* 리스트 내부의 자료구조를 직접 사용하지 않고 리스트의 각 항목을 순차적으로 접근하기 위해 List 클래스를 정의해서 사용ㅇ하는 방법은 한 클래스에 너무 많은 인터페이스가 집중된다는 단점과 동일한 리스트의 서로 다른 위치를 동시에 접근해서 작업을 수행하는 것이 불가능하다는 문제가 있을 것이다. 

### 샘플 코드

```c++
#ifndef ITEM_H
#define ITEM_H

#include <string>
using namespace std;

class Item {
  public :
	Item(string str): data_(str) {}
    string data_;
};

#endif


#ifndef LINKED_TREE_LIST_H
#define LINKED_TREE_LIST_H

#include "item.h"

class Iterator;

class AbstractList {
  public :
    virtual Iterator * CreateIterator() = 0;

    int Count();
    virtual void AddNext(Item *pNewItem, Item *pItem=0) = 0;
    virtual void AddChild(Item *pNewItem, Item *pItem=0) = 0;
    virtual void Remove(Item *pItem) = 0;
    virtual Item* GetItem(int pos) = 0;
  protected :
    AbstractList() : totalCnt_ (0) {}
    int totalCnt_;
};

class LinkedList : public AbstractList {
  public :
    LinkedList() : pFirst_(0) {}

    struct LinkedItem {
      LinkedItem(Item *pItem=0, LinkedItem *pNext=0) 
        : pData_(pItem), pNext_(pNext) {}
      Item* pData_;
      LinkedItem* pNext_;
    };

    Iterator *CreateIterator();
    void AddNext(Item *pNewItem, Item *pItem=0);
    void AddChild(Item *pNewItem, Item *pItem=0);
    void Remove(Item *pItem);
    Item* GetItem(int pos);

  private :
    LinkedItem *pFirst_;
};

class TreeList : public AbstractList {
  public :
    TreeList() : pRoot_(0) {}

    struct TreeItem {
      TreeItem(Item* pItem=0, TreeItem* pParent=0, 
                TreeItem *pFirstChild=0, TreeItem* pSibling=0)
        : pData_(pItem), pParent_(pParent), 
                pFirstChild_(pFirstChild), pSibling_(pSibling) {}

      Item* pData_;
      TreeItem* pParent_;
      TreeItem* pFirstChild_;
      TreeItem* pSibling_;
    };

    Iterator *CreateIterator();
    void AddNext(Item *pNewItem, Item *pItem=0);
    void AddChild(Item *pNewItem, Item *pItem=0);
    void Remove(Item *pItem);
    Item* GetItem(int pos);

  protected :
    TreeItem* GetNext(const TreeItem* pStart);
  private :
    TreeItem *pRoot_;
};

#endif





#ifndef ITER_H
#define ITER_H

#include "item.h"

class LinkedList;
class TreeList;

class Iterator {
  public :
    void First();
    void Next();
    virtual bool IsDone() = 0;
    virtual Item* GetCurItem() = 0;
  protected :
    int curPos_;
    Iterator() { curPos_ = 0; }
};

class ListIterator : public Iterator {
  public :
    ListIterator(LinkedList* pList);

    bool IsDone();
    Item* GetCurItem();

  private :
    LinkedList* pList_;
};

class TreeIterator : public Iterator {
  public : 
    TreeIterator(TreeList* pTree);

    bool IsDone();
    Item* GetCurItem();

  private :
    TreeList* pTree_;
};

#endif 





#include <iostream>
#include "iter.h"
#include "list.h"
using namespace std;

void 
Iterator::First() { curPos_ = 0; }

void 
Iterator::Next() { curPos_++; }

// ---------------------------------------------------

ListIterator::ListIterator(LinkedList *pList) {
  pList_ = pList;
}

bool 
ListIterator::IsDone() { 
  if (curPos_ >= pList_->Count()) return true;
  else return false;
}

Item* 
ListIterator::GetCurItem() { return pList_->GetItem(curPos_); }

// ---------------------------------------------------

TreeIterator::TreeIterator(TreeList* pTree) {
  pTree_ = pTree;
}

bool 
TreeIterator::IsDone() {
  if (curPos_ >= pTree_->Count()) return true;
  else return false;
}

Item* 
TreeIterator::GetCurItem() { return pTree_->GetItem(curPos_); }




#include <iostream>
#include "list.h"
#include "iter.h"
using namespace std;

int
AbstractList::Count() { return totalCnt_; }

// ---------------------------------------------------

Iterator *
LinkedList::CreateIterator() {
  return new ListIterator(this);
}

void 
LinkedList::AddNext(Item *pNewItem, Item *pItem) {
  if (pFirst_ == 0) 
    pFirst_ = new LinkedItem(pNewItem);
  else if (pItem == 0 || pItem == pFirst_->pData_) {
    LinkedItem* pTmp = pFirst_->pNext_;
    pFirst_->pNext_ = new LinkedItem(pNewItem, pTmp);
  }
  else {
    LinkedItem* pPrev = 0;
    LinkedItem* pTmp = pFirst_;
    while (pTmp != 0 && pTmp->pData_ != pItem) {
      pPrev = pTmp;
      pTmp = pTmp->pNext_;
    }

    if (pTmp != 0) {
      LinkedItem* pTmp2 = pTmp->pNext_;
      pTmp->pNext_ = new LinkedItem(pNewItem, pTmp2);
    }
    else
      pPrev->pNext_ = new LinkedItem(pNewItem, 0);
  }
  totalCnt_++;
}

void 
LinkedList::AddChild(Item *pNewItem, Item *pItem) { 
  AddNext(pNewItem, pItem); 
}

void 
LinkedList::Remove(Item *pItem) {
  if (pItem == 0) return ;

  LinkedItem *pPrev = 0;
  LinkedItem *pTmp = pFirst_;
  while (pTmp != 0 && pTmp->pData_ != pItem) {
    pPrev = pTmp;
    pTmp = pTmp->pNext_;
  }

  if (pTmp != 0) {
    if (pTmp == pFirst_) {
      delete pTmp;
      pFirst_ = 0;
    }
    else {
      pPrev->pNext_ = pTmp->pNext_;
      delete pTmp;
    }
    totalCnt_--;
  }
}

Item* 
LinkedList::GetItem(int pos) {
  int cnt = 0;
  LinkedItem *pTmp = pFirst_;
  while (pTmp != 0 && cnt != pos) {
    pTmp = pTmp->pNext_;
    cnt++;
  }

  if (pTmp != 0) return pTmp->pData_;
  else           return 0;
}

// ---------------------------------------------------

Iterator *
TreeList::CreateIterator() {
  return new TreeIterator(this);
}

void 
TreeList::AddNext(Item *pNewItem, Item *pItem) {
  if (pRoot_ == 0)  // -- 처음 등록하는 경우
    pRoot_ = new TreeItem(pNewItem);
  else if (pItem == 0 || pItem == pRoot_->pData_) {
    // -- Root 항목의 첫번째 자식으로 삽입
    TreeItem *pTmp = pRoot_->pFirstChild_;
    pRoot_->pFirstChild_ = new TreeItem(pNewItem, pRoot_, 0, pTmp);
  }
  else {
    // -- pData_ 값이 pItem과 같은 항목의 형제 노드로 삽입
    TreeItem *pPrev = 0;
    TreeItem *pTmp = GetNext(pRoot_);
    while (pTmp != 0 && pTmp->pData_ != pItem) {
      pPrev = pTmp;
      pTmp = GetNext(pTmp);
    }

    if (pTmp != 0) {
      TreeItem *pTmp2 = pTmp->pSibling_;
      pTmp->pSibling_ = new TreeItem(pNewItem, pTmp->pParent_, 0, pTmp2);
    }
    else // -- 같은 노드가 없으면 맨 마지막 노드의 형제로 삽입
      pPrev->pSibling_ = new TreeItem(pNewItem, pPrev->pParent_, 0, 0);
  }
  totalCnt_++;
}

void 
TreeList::AddChild(Item *pNewItem, Item *pItem) {
  if (pRoot_ == 0)
    pRoot_ = new TreeItem(pNewItem);
  else if (pItem == 0 || pItem == pRoot_->pData_) {
    // -- Root 항목의 첫번째 자식으로 삽입
    TreeItem *pTmp = pRoot_->pFirstChild_;
    pRoot_->pFirstChild_ = new TreeItem(pNewItem, pRoot_, 0, pTmp);
  }
  else {
    // -- pData_ 값이 pItem과 같은 항목의 자식 노드로 삽입
    TreeItem *pPrev = 0;
    TreeItem *pTmp = GetNext(pRoot_);
    while (pTmp != 0 && pTmp->pData_ != pItem) {
      pPrev = pTmp;
      pTmp = GetNext(pTmp);
    }

    if (pTmp != 0) {
      TreeItem *pTmp2 = pTmp->pFirstChild_;
      pTmp->pFirstChild_ = new TreeItem(pNewItem, pTmp, 0, pTmp2);
    }
    else // -- 같은 노드가 없으면 맨 마지막 노드의 자식으로 삽입
      pPrev->pFirstChild_ = new TreeItem(pNewItem, pPrev, 0, 0);
  }
  totalCnt_++;
}

void 
TreeList::Remove(Item *pItem) {
  if (pItem == 0) return;

  TreeItem *pPrev = 0;
  TreeItem *pTmp = pRoot_;
  while (pTmp != 0 && pTmp->pData_ != pItem) {
    pPrev = pTmp;
    pTmp = GetNext(pTmp);
  }

  if (pTmp != 0) {
    if (pTmp->pFirstChild_ != 0) 
      cout << "Can't Remove the requested Node" << endl;
    else {
      if (pTmp == pRoot_) {
        delete pTmp;
        pRoot_ = 0;
      }
      else if (pTmp->pParent_->pFirstChild_ == pTmp) {
        // -- 지우려는 항목이 첫번째 자식 노드일 경우
        pTmp->pParent_->pFirstChild_ = GetNext(pTmp);
        delete pTmp;
      }
      else {
        pPrev->pSibling_ = pTmp->pSibling_;
        delete pTmp;
      }
      totalCnt_--;
    }
  }
}

Item* 
TreeList::GetItem(int pos) {
  int cnt = 0;
  TreeItem* pTmp = pRoot_;
  while (pTmp != 0 && cnt != pos) {
    pTmp = GetNext(pTmp);
    cnt++;
  }

  if (pTmp != 0) return pTmp->pData_;
  else           return 0;
}

TreeList::TreeItem* 
TreeList::GetNext(const TreeItem* pStart) { // -- Bread First 방식
  if (pStart == 0) return 0;
  else if (pStart->pSibling_ != 0)  // -- 형제 노드가 존재시
    return pStart->pSibling_;
  else if (pStart->pParent_ == 0) // -- Root 노드의 경우
    return pStart->pFirstChild_;
  else   // -- 현재 레벨에서 마지막 형제 노드의 경우
    return pStart->pParent_->pFirstChild_->pFirstChild_;
}



#include <iostream>
#include "item.h"
#include "list.h"
#include "iter.h"
using namespace std;

void
PrintData(Iterator* pIter)
{
  for (pIter->First(); ! pIter->IsDone(); pIter->Next()) 
    cout << pIter->GetCurItem()->data_ << endl;
}

void
main()
{
  Item item1("A"), item2("B"), item3("C"), item4("D");

  LinkedList list;
  list.AddNext(&item1);
  list.AddNext(&item2, &item1);
  list.AddNext(&item3, &item2);
  list.AddNext(&item4, &item3);

  ListIterator *pListIter = 
	  (ListIterator*) list.CreateIterator();
  PrintData(pListIter);

  cout << "--------------------------------" << endl;

  TreeList tree;
  tree.AddNext(&item1);
  tree.AddNext(&item2, &item1);
  tree.AddChild(&item3, &item2);
  tree.AddChild(&item4, &item2);

  TreeIterator *pTreeIter = 
	  (TreeIterator *) tree.CreateIterator();
  PrintData(pListIter);
}

```

* 굳이 정리 안함.


## 21\. 복잡한 M:N 객체 관계의 완화 문제
### Mediator 패턴
* 굳이 정리 안함. 

## 22\. 객체의 이전 상태 복원 문제
### 문제 사례 설명
* 바둑 게임을 위한 프로그램을 만든다고 생각해보자.바둑판의 현재 상태를 저장하기 위한 바둑판 객체는 꼭 필요할 것이다. 그런데 이 바둑판 객체는 현재 상태만 저장하고 있어서는 안된다. 수를 무를수도 있고 해서 이전 바둑판 상태로 되돌아 갈수 있어야 한다. 그렇다면 어떻게 설계하는 것이 좋을까?

### Memento 패턴
* board[ ][ ] 이차원 배열을 사용할수도 있으나, 이것만은 가지고 정보가 부족할 것이다. 바둑판 객체가 가지고 있는 데이터 멤버를 하나의 클래스로 정의하고 이 클래스를 이용해서 바둑판 상태 정보를 시점별로 저장, 관리하는 리스트를 만들어 사용하도록 하는것이 좋을 것이다. 
* 이와 같이 객체의 상태 정보를 별도의 클래스로 정의하고, 리스트 자료구조에 그 클래스의 객체를 필요한 시점마다 저장해두었다가 특정 시점의 상태 복원을 쉽게 할 수 있도록 만든 클래스 구조를 Memento 패턴이라고 한다. 

### 샘플 코드

* 바둑판 상태는 historyList_ 에 저장하고 새로운 바둑판 상태를 pCurBoard_ 데이터 멤버로 관리하는 것으로, 무르기 요청시 historyList_ 데이터 멤버에 저장된 객체를 이용해 무르기가 가능하다. 

```c++

#define GO_BOARD_SIZE     19

class GoMemento {
    friend class GoBoard;
  public :
    GoMemento() {
      for (int i=0; i < GO_BOARD_SIZE; i++)
        for (int j=0; j < GO_BOARD_SIZE; j++)
          board_[i][j] = 0;

      whiteDeadNum_ = blackDeadNum_ = 0;
      paePosX_ = paePosY_ = -1;
    }

    GoMemento(const GoMemento& rhs) { CopyBoard(rhs); }
    GoMemento& operator=(const GoMemento& rhs) { CopyBoard(rhs); }

    void GetOutDeadStone() {
      // -- 확실히 죽은 돌을 골라낸다. 
      // -- 이때 whiteDeadNum_ 이나 blackDeadNum_ 값이 조정됨
    }

    bool IsPaePosition(int x, int y) {
      // -- x,y 위치가 패이면 true 되돌림
      return false;
    }

  protected :
    void CopyBoard(const GoMemento& src) {
      for (int i=0; i < GO_BOARD_SIZE; i++)
        for (int j=0; j < GO_BOARD_SIZE; j++)
          board_[i][j] = src.board_[i][j];

      whiteDeadNum_ = src.whiteDeadNum_;
      blackDeadNum_ = src.blackDeadNum_;
    }
  private :
    // -- 0 은 돌없음, 양수는 백, 음수는 흑, 절대값은 수순
    int board_[GO_BOARD_SIZE][GO_BOARD_SIZE];

    int whiteDeadNum_; // -- 흰돌 죽은 수
    int blackDeadNum_; // -- 검은돌 죽은 수

    int paePosX_;
    int paePosY_;
};

class GoBoard {   // -- 바둑판
  public :
    GoBoard(int firstTurn=-1) { 
      pCurBoard_ = new GoMemento(); 
      whoseTurn_ = firstTurn;
      totalStoneNum_ = 0;
    }

    void PutStone(int x, int y) {
      if (pCurBoard_->board_[x][y] != 0 || 
          (pCurBoard_->paePosX_ == x && pCurBoard_->paePosY_ == y)) {
        cout << "Can't Be Put Stone There." << endl;
        return;
      }
      
      GoMemento *pNewBoard = new GoMemento(*pCurBoard_);
      totalStoneNum_++;
      pNewBoard->board_[x][y] = whoseTurn_ * totalStoneNum_;
      whoseTurn_ *= -1;
      if (pCurBoard_->IsPaePosition(x,y)) {
        pCurBoard_->paePosX_ = x;
        pCurBoard_->paePosY_ = y;
      }
      else {
        pCurBoard_->paePosX_ = -1;
        pCurBoard_->paePosY_ = -1;
      }

      pNewBoard->GetOutDeadStone();

      historyList_.push_front(pCurBoard_);
      pCurBoard_ = pNewBoard;
    }

    void RetractStone(int cnt) {
      // -------------------------------------------
      // cnt 값만큼 수를 물린다. 
      // 이때 1수를 물리는 것은 pCurBoard_ 값을 historyList_의 
      // 첫 항목과 대치하는 것과 동일하다. 
      // -------------------------------------------
      if (cnt <= 0) return;

      for (int i=0; i < cnt-1; i++) { 
        GoMemento *pTmpBoard = historyList_.front();
        delete pTmpBoard;
        historyList_.pop_front();
        totalStoneNum_--;
      }

      delete pCurBoard_;
      totalStoneNum_--;
      if (historyList_.empty()) 
        pCurBoard_ = new GoMemento();
      else 
        pCurBoard_ = historyList_.front();
    }

    void PrintBoard() {
      for (int i=0; i < GO_BOARD_SIZE; i++) {
        for (int j=0; j < GO_BOARD_SIZE; j++) 
          cout << pCurBoard_->board_[i][j] << " ";
        cout << endl;
      }
      cout << "--- total stone --- " << totalStoneNum_ << endl;
    }
  private :
    list<GoMemento*> historyList_;
    GoMemento* pCurBoard_;
    int whoseTurn_;    // -- 다음 둘 차례(1 은 백, -1 은 흑)
    int totalStoneNum_;
};

void
main()
{
  GoBoard board;

  board.PutStone(3,3);
  board.PutStone(16,16);
  board.PutStone(16,3);
  board.PutStone(3, 16);
  board.PrintBoard();

  board.RetractStone(2);
  board.PrintBoard();
}
```

* Memento 패턴의 일반적인 클래스 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-03-01.png)

 
## 23\. ONE SOURCE MULTIPLE USE 문제
* 하나의 원본 데이터를 동시에 참조하고 있는 여러개의 서로 다른 표현 양식간에 데이터 값의 변화를 즉시 반영할 수 있는 방법에 대해 고민해 보자. 
### 문제 사례 설명
* 선생님이 학생들의 성적을 MS 엑셀을 이용해서 저장 관리한다고 하자. 학생의 국,영,수 성적을 표 형태로 입력하였다. 성적차이등을 보기 위해 막대그래프나, 선 그래프로 표현하였는데, 학생의 성적을 바꾸는 경우 이를 바로 각종 표, 그래프 등에 반영하려면 어떻게 하면 좋을까?

### Observer 패턴
* 원본 데이터를 직접 참조하는 방식으로 여러 다른 표현 양식간에 데이터를 일관성을 유지하게 만드는 것은 데이터 변경이 일어날때 바로 반영하지 못하는 점과 원본 데이터를 직접 참조함으로 정보 은닉이 깨진다는 단점이 있다. 
* 21장의 Mediator 패턴에서처럼 매개 객체를 사용하는 것이다. 즉 데이터 값이 변경된 표현 양식은 매개 객체에게 데이터 값이 변경되었음을 통보하고 , 매개 객체가 이를 전체 표현 객체에게 알려주는 방식이다. 객체 내부의 자료구조와는 무관하게 표현 객체들이 원본 데이터의 값을 참조할 수 있게 표준화된 자료구조를 제공하는 것이 좋다. 원본 데이터의 값을 완변히 표현할 수 있는 자료구조를 별도로 정의하고 객체 내부의 자료구조와 무관하게 별도로 정의된 자료 구조를 통해 객체간의 정보 교류가 이루어지도록 만들어 주면 된다. 

### 샘플 코드

```c++
#ifndef SCORECARD_H
#define SCORECARD_H

#include <string>
using namespace std;

#define MOTHER_LANG_SUBJECT     1
#define MATH_SUBJECT            2
#define ENGLISH_SUBJECT         3

class ScoreCard {
  public :
    ScoreCard() {
      motherLangScore_ = mathScore_ = englishScore_ = 0;
    }

    string name_;
    int motherLangScore_; // -- 국어
    int englishScore_;    // -- 영어
    int mathScore_;       // -- 수학
};

#endif







#ifndef SUBJECT_H
#define SUBJECT_H

#include <string>
#include <list>
#include "scorecard.h"
using namespace std;

class Observer;

class Subject {
  public :
    virtual ~Subject() = 0;
    virtual void Attach(Observer *pObj);
    virtual void Detach(Observer *pObj);
    virtual void Notify();
  protected :
    list<Observer *> observers_;
};

class ScoreData : public Subject {
  public :
    void AddScore(ScoreCard *pScore);
    void RemoveScore(ScoreCard *pScore);

    list<ScoreCard*>  GetScoreList();
    void SetScore(string name, int subjectType, int score);
  private :
    list<ScoreCard*> scoreList_;
};

#endif







#include <string>
#include <list>
#include <algorithm>
#include "observer.h"
#include "subject.h"
using namespace std;

Subject::~Subject() {}

void
Subject::Attach(Observer *pObj) {
  observers_.push_front(pObj);
}

void
Subject::Detach(Observer *pObj) {
  list<Observer*>::iterator iter;
  for (iter = observers_.begin(); iter != observers_.end(); iter++)
    if (*iter == pObj) observers_.erase(iter);
}

void
Subject::Notify() {
  list<Observer*>::iterator iter;
  for (iter = observers_.begin(); iter != observers_.end(); iter++) 
    (*iter)->Update();
}

// -------------------------------------------------

void
ScoreData::AddScore(ScoreCard *pScore) {
  scoreList_.push_front(pScore);
  Notify();
}

void
ScoreData::RemoveScore(ScoreCard *pScore) {
  list<ScoreCard*>::iterator iter;
  iter = find(scoreList_.begin(), scoreList_.end(), pScore);
  if (iter != scoreList_.end()) {
    ScoreCard *pTmpScore = *iter;
    delete pTmpScore;
    scoreList_.erase(iter);
    Notify();
  }
}

list<ScoreCard*>
ScoreData::GetScoreList() { return scoreList_; }

void
ScoreData::SetScore(string name, int subjectType, int score) {
  list<ScoreCard*>::iterator iter;
  for (iter=scoreList_.begin(); iter != scoreList_.end(); iter++) {
    ScoreCard *pTmpScore = *iter;
    if (pTmpScore->name_ == name) {
      switch (subjectType) {
        case MOTHER_LANG_SUBJECT :
            pTmpScore->motherLangScore_ = score;
            break;
        case ENGLISH_SUBJECT :
            pTmpScore->englishScore_ = score;
            break;
        case MATH_SUBJECT :
            pTmpScore->mathScore_ = score;
            break;
      }
      Notify();
      break;
    }
  }
}





#ifndef OBSERVER_H
#define OBSERVER_H

#include <string>
#include "scorecard.h"
using namespace std;

#define MAX_STUDENT_NUM     10

class Subject;
class ScoreData;

class Observer {
  public :
    virtual void Update()=0;
};

class BarGraph : public Observer {
  public :
    BarGraph(ScoreData *pScoreData);
    void Update();
    void PrintOut();
    void ChangeScore(string name, int subjectType, int score);
  private :
    ScoreData *pScoreData_;
    string name_[MAX_STUDENT_NUM];
    int motherLangScore_[MAX_STUDENT_NUM];
    int englishScore_[MAX_STUDENT_NUM];
    int mathScore_[MAX_STUDENT_NUM];
};

class PieChart : public Observer {
  public :
    PieChart(string student, ScoreData *pScoreData);
    void Update();
    void PrintOut();
    void ChangeScore(string name, int subjectType, int score);
  private :
    ScoreData *pScoreData_;
    string name_;
    int motherLangScore_;
    int englishScore_;
    int mathScore_;
};

class LineGraph : public Observer {
  public :
    LineGraph(ScoreData *pScoreData);
    void Update();
    void PrintOut();
    void ChangeScore(string name, int scoreType, int score);
  private :
    ScoreData *pScoreData_;
    string name_[MAX_STUDENT_NUM];
    int motherLangScore_[MAX_STUDENT_NUM];
    int englishScore_[MAX_STUDENT_NUM];
    int mathScore_[MAX_STUDENT_NUM];
};

#endif






#include <iostream>
#include "subject.h"
#include "observer.h"
using namespace std;

BarGraph::BarGraph(ScoreData *pScoreData) : pScoreData_(pScoreData) {
  pScoreData_->Attach(this);
}

void
BarGraph::Update() {
  list<ScoreCard*> scoreList;
  scoreList = pScoreData_->GetScoreList();

  int nthStudent = 0;
  list<ScoreCard*>::iterator iter;
  for (iter=scoreList.begin(); iter != scoreList.end(); iter++) {
    ScoreCard *pTmpScore = *iter;
    name_[nthStudent] = pTmpScore->name_;
    motherLangScore_[nthStudent] = pTmpScore->motherLangScore_;
    englishScore_[nthStudent] = pTmpScore->englishScore_;
    mathScore_[nthStudent] = pTmpScore->mathScore_;
    nthStudent++;
  }
}

void
BarGraph::PrintOut() {
  for (int i = 0; i < MAX_STUDENT_NUM; i++) {
    if (name_[i].empty()) break;
    cout << name_[i] << "=>" << motherLangScore_[i] 
      << ":" << englishScore_[i] << ":" << mathScore_[i] << endl;
 }
}

void
BarGraph::ChangeScore(string name, int subjectType, int score) {
  pScoreData_->SetScore(name, subjectType, score);
}

// ---------------------------------------------------

PieChart::PieChart(string student, ScoreData *pScoreData) 
  : pScoreData_(pScoreData) { 
  name_ = student; 
  pScoreData_->Attach(this);
}

void
PieChart::Update() {
  list<ScoreCard*> scoreList;
  scoreList = pScoreData_->GetScoreList();

  list<ScoreCard*>::iterator iter;
  for (iter=scoreList.begin(); iter != scoreList.end(); iter++) {
    ScoreCard *pTmpScore = *iter;
    if (name_ == pTmpScore->name_) {
      motherLangScore_ = pTmpScore->motherLangScore_;
      englishScore_ = pTmpScore->englishScore_;
      mathScore_ = pTmpScore->mathScore_;
      break;
    }
  }
}

void
PieChart::PrintOut() {
  cout << name_ << "=>" << motherLangScore_ 
      << ":" << englishScore_ << ":" << mathScore_ << endl;
}

void
PieChart::ChangeScore(string name, int subjectType, int score) {
  pScoreData_->SetScore(name, subjectType, score);
}

// ---------------------------------------------------

LineGraph::LineGraph(ScoreData *pScoreData) : pScoreData_(pScoreData) {
  pScoreData_->Attach(this);
}

void
LineGraph::Update() {
  list<ScoreCard*> scoreList;
  scoreList = pScoreData_->GetScoreList();

  int nthStudent = 0;
  list<ScoreCard*>::iterator iter;
  for (iter=scoreList.begin(); iter != scoreList.end(); iter++) {
    ScoreCard *pTmpScore = *iter;
    name_[nthStudent] = pTmpScore->name_;
    motherLangScore_[nthStudent] = pTmpScore->motherLangScore_;
    englishScore_[nthStudent] = pTmpScore->englishScore_;
    mathScore_[nthStudent] = pTmpScore->mathScore_;
    nthStudent++;
  }
}

void
LineGraph::PrintOut() {
  for (int i = 0; i < MAX_STUDENT_NUM; i++) {
    if (name_[i].empty()) break;
    cout << name_[i] << "=>" << motherLangScore_[i] 
      << ":" << englishScore_[i] << ":" << mathScore_[i] << endl;
  }
}

void
LineGraph::ChangeScore(string name, int subjectType, int score) {
  pScoreData_->SetScore(name, subjectType, score);
}







#include <iostream>
#include "scorecard.h"
#include "subject.h"
#include "observer.h"
using namespace std;

void
main() 
{
  ScoreData termScore;
  BarGraph bar(&termScore);
  LineGraph line(&termScore);

  ScoreCard stu1, stu2, stu3;
  stu1.name_ = "철수";
  stu1.motherLangScore_ = 70;
  stu1.englishScore_ = 60;
  stu1.mathScore_ = 90;

  stu2.name_ = "영희";
  stu2.motherLangScore_ = 70;
  stu2.englishScore_ = 80;
  stu2.mathScore_ = 50;

  stu3.name_ = "순이";
  stu3.motherLangScore_ = 95;
  stu3.englishScore_ = 90;
  stu3.mathScore_ = 85;

  termScore.AddScore(&stu1);
  termScore.AddScore(&stu2);
  termScore.AddScore(&stu3);

  cout << "-----------------------" << endl;
  bar.PrintOut();
  cout << "-----------------------" << endl;
  line.PrintOut();

  bar.ChangeScore("철수", ENGLISH_SUBJECT, 85);

  cout << "-----------------------" << endl;
  bar.PrintOut();
  cout << "-----------------------" << endl;
  line.PrintOut();
}
```

* 



## 24\. 객체 상태 추가에 따른 행위 수행 변경 문제
### 문제 사례 설명
* 게임플레이어의 등급에 따라 공격 가능한 유형이 결정되는 격투 게임 프로그램을 작성한다고 가정해 보자. 이와 같은 경우 게임 플레이어가 임의의 공격 명령을 내렸을 때 자신의 등급 상태에 따라 가능한 공격만을 수행하게 프로그램을 설계한다면 어떤식이 좋을까?

### State 패턴
* 객체 내부의 상태 비교를 통해 행위 수행을 변경하는 방식은 일반적이지만 새로운 종류의 상태가 추가될 경우 기존 소스코드를 분석해서 수정해야하는 문제가 있다. 
* 기존 소스코드 수정이 없게 추가하는 대표적인 방법은 클래스 상속을 이용하는 것이다. 다형성에 의해 추가된 구현 내용을 코드 수정없이 적용할 것이다. 
* 클래스 상속을 통해 새로운 종류의 상태를 소스코드 수정없이 추가했다면, 추가된 상태를 포함해서 객체의 상태 변화가 있을 때마다 행위 수행 변경을 기존 코드 수정엇이 처리해주는 방법은 무엇일까?
* 게임 플레이어 객체가 다른 객체에게 등급 변화에 따른 행위 수행 변경을 위임시키는 것이다. 
* 어떤 객체의 내부 상태가 계속 추가될 가능성이 있을때 새로운 상태의 추가도 쉽도록 도와주고 , 추가된 상태를 포함해서 객체의 상태 변화 시 기존 소스코드 변경없이 이 행위 수행 변경이 가능하도록 객체 상태 정보를 클래스 상속 구조로 정의해서 사용하는 방식을 State 패턴이라고 한다.

### 샘플 코드

```c++

class GameLevel {
  public :
    static GameLevel* CreateInstance() { return 0; }
    virtual void SimpleAttack() = 0;
    virtual void TurnAttack() = 0;
    virtual void FlyingAttack() = 0;
  protected :
    GameLevel() {}
};

class GameLevel0 : public GameLevel {
  public :
    static GameLevel* CreateInstance() {
      if (pInstance_ == 0) pInstance_ = new GameLevel0;
      return pInstance_;
    }

    void SimpleAttack() { cout << "Simple Attack" << endl; }
    void TurnAttack() { cout << "Not Allowed" << endl; }
    void FlyingAttack() { cout << "Not Allowed" << endl; }
  protected :
    GameLevel0() { }
  private : 
    static GameLevel0 *pInstance_;
};
GameLevel0* GameLevel0::pInstance_ = 0;

class GameLevel1 : public GameLevel {
  public :
    static GameLevel* CreateInstance() {
      if (pInstance_ == 0) pInstance_ = new GameLevel1;
      return pInstance_;
    }

    void SimpleAttack() { cout << "Simple Attack" << endl; }
    void TurnAttack() { cout << "Turn Attack" << endl; }
    void FlyingAttack() { cout << "Not Allowed" << endl; }
  protected :
    GameLevel1() { }
  private : 
    static GameLevel1 *pInstance_;
};
GameLevel1* GameLevel1::pInstance_ = 0;

class GameLevel2 : public GameLevel {
  public :
    static GameLevel* CreateInstance() {
      if (pInstance_ == 0) pInstance_ = new GameLevel2;
      return pInstance_;
    }

    void SimpleAttack() { cout << "Simple Attack" << endl; }
    void TurnAttack() { cout << "Turn Attack" << endl; }
    void FlyingAttack() { cout << "Flying Attack" << endl; }
  protected :
    GameLevel2() { }
  private : 
    static GameLevel2 *pInstance_;
};
GameLevel2* GameLevel2::pInstance_ = 0;

class GamePlayer {
  public :
    GamePlayer() { pGameLevel_ = GameLevel0::CreateInstance(); }

    void UpgradeLevel(GameLevel* pLevel) {
      pGameLevel_ = pLevel;
    }

    void SimpleAttack() { pGameLevel_->SimpleAttack(); }
    void TurnAttack() { pGameLevel_->TurnAttack(); }
    void FlyingAttack() { pGameLevel_->FlyingAttack(); }
  private :
    GameLevel *pGameLevel_;
};

void
main()
{
  GamePlayer user1;

  user1.SimpleAttack();
  user1.TurnAttack();
  user1.FlyingAttack();
  cout << "--------------------------" << endl;

  GameLevel *pGameLevel1 = GameLevel1::CreateInstance();
  user1.UpgradeLevel(pGameLevel1);

  user1.SimpleAttack();
  user1.TurnAttack();
  user1.FlyingAttack();
  cout << "--------------------------" << endl;

  GameLevel *pGameLevel2 = GameLevel2::CreateInstance();
  user1.UpgradeLevel(pGameLevel2);

  user1.SimpleAttack();
  user1.TurnAttack();
  user1.FlyingAttack();
}
```

* GamePlayer 클래스는 게임 플레이어 등급에 영향을 받는 모든 유형의 공격에 대해 GameLevel 및 그 하위 클래스 객체에게 처리를 위임하는 형태를 가진다.

* State 패턴의 일반적인 클래스 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-03-01.png)



## 25\. 동일 목적 알고리즘의 동적 적용 문제
### 문제 사례 설명
* 굳이 정리하지 않음.

## 26\. 알고리즘 기본 골격 재사용과 상세 구현 변경 문제
### 문제 사례 설명
* 굳이 정리하지 않음.


## 27\. 작업 종류를 효율적으로 추가\, 변경 문제
### 문제 사례 설명
* 여러 종류의 계좌를 운영, 관리하는 은행 프로그램을 만든다고 생각해보자. 이때 클래스는 계좌 종류별로 정의될 수 있을 것이다. 입금, 출금을 하기 위한 인터페이스 외에도 입출금 내역 조회, 계좌별 총 보유금액 조회등가 같이 다양한 작업을 수행하기 위한 인터페이스를 가질 것이다.
* 이 계좌 클래스는 개인 고객별 계좌 객체를 생성하기 위해 사용될 것이고 리스트와 같은 형태로 관리 될 것이다. 그런데 이때 리스트 형태의 관리에서 다양한 작업을 해주고 싶을때 어떤 형태로 접근하는 것이 좋겠는가?
	* 여러 개좌에 대해 고객명과 계좌별 총 보유 금액을 특정한 포맷으로 출력한다던가, 계좌별 입출금 내역을 출력하게 만드는 등 및 세법이나 이자율 조정 등으로 수시로 추가 , 변경 될 수 가 있을 것이다.


### Vistor 패턴
* 클래스를 정의한 이후에 수행할 작업 항목에 따라 멤버 함수가 추가, 변경되는 것은 작업의 종류가 늘어날 때 마다 기존의 클래스를 수정해야 하는 문제가 있다. 
* 또한 이 방식은 객체 여러 개에 대해 다양한 작업을 수행하고자 할 때 작업 종류마다 수행 함수를 구현해야하는 문제도 있다. 
* 문제의 핵심은 새로운 종류의 작업을 쉽게 추가할 수 있어야 한다는 것이다. 기존 소스 코드에 영향을 주지 않고 새로운 종류의 작업을 추가할수 있어야 한다는 것이다. 
	* 객체지향 설계에서 이를 위한 대표적인 방법은 클래스 상속을 이용하는 방법이다. 
	* 클래스 상속을 이용해 새로운 종류의 작업을 쉽게 추가할 수 있게 만드는 방법은 
		* 클래스로정의해야하는 것은 데이터가 아니라 작업 항목이다. 
			* 추가, 변경되는 것이 작업 항목이고 이를 쉽게 추가, 변경시킬수 있도록 하려면 작업 항목 자체가 클래스로 정의되어 하위 클래스로 확장이 이루어져야 한다. 
		* 작업 항목을 클래스로 정의 했을때 작업 대상과 작업 항목을 어떻게 연결 시켜 줄수 있을까?
			* 작ㅇ버 항목 객체에게 작업 대상 객체를 전달해주는 것이다. 
* 통상적으로 Client 는 작업 대상 객체에 대해 멤버 함수를 호출하는 형태로 작업 수행을 요청한다. 
* 작업 대상과 작업 항목을 별도의 클래스 구조로 구분하게 되면, 새로운 작업 항목을 추가, 변경하기기 원활 경우 작업 대상 클래스는 수정할 필요없이 작업 항목 클래스 구조에 새로운 하위 클래스를 추가하거나 변경해주면 되므로 편리하다. 
* 새로운 종류의 작업을 쉽게 추가할 수 있고, 객체 여러 개에 대해서도 다양한 작업을 불편없이 수행할 수 있게 작업 대상과 작업 항목을 분리해서 클래스 구조로 정의하는 것을  Visitor 패턴이라고 한다. 

### 샘플 코드

```c++
#ifndef ACCOUNT_H
#define ACCOUNT_H

#include <list>
#include <string>
using namespace std;

class Reporter;

class Account {
  public :
    Account(string n);
    virtual void Accept(Reporter& rpt)=0;
    virtual int GetTotalSum() = 0;
    list<int> GetHistory();
    string GetCustomerName();
    void Deposit(int inMoney);
    void Withdraw(int outMoney);
  protected :
    int netMoney_;
    list<int> history_; // -- 양수는 입금, 음수는 출금을 의미
    string customerName_;
};

class PassbookAccount : public Account {
  public :
    PassbookAccount(string n);
    void Accept(Reporter& rpt);
    int GetTotalSum();
};

class CheckingAccount : public Account {
  public :
    CheckingAccount(string n);
    void Accept(Reporter& rpt);
    int GetTotalSum();
};

#endif 






#include <math.h>
#include "account.h"
#include "reporter.h"
using namespace std;

#define lround(x)	((long)((double(x)) + 0.5))		// added in VC++

Account::Account(string n) { 
  customerName_ = n; 
  netMoney_ = 0;
}

list<int>
Account::GetHistory() { return history_; }

string
Account::GetCustomerName() { return customerName_; }

void
Account::Deposit(int inMoney) { 
  netMoney_ += inMoney;
  history_.push_back(inMoney);
}

void
Account::Withdraw(int outMoney) {
  netMoney_ -= outMoney;
  history_.push_back(-outMoney);
}

// ---------------------------------------
PassbookAccount::PassbookAccount(string n) : Account(n) {}

void
PassbookAccount::Accept(Reporter& rpt) {
  rpt.VisitPassbookAccount(this);
}

int
PassbookAccount::GetTotalSum() {
  return netMoney_; // -- 별도 이자 없음.
}

// ---------------------------------------
CheckingAccount::CheckingAccount(string n) : Account(n) {}

void
CheckingAccount::Accept(Reporter& rpt) {
  rpt.VisitCheckingAccount(this);
}

int
CheckingAccount::GetTotalSum() {
  return netMoney_ + lround(netMoney_ * 0.1); // -- 10% 이자
}





#include <iostream>
#include <list>
#include <string>
#include "account.h"
#include "reporter.h"
using namespace std;

void DoReport(list<Account *>& accountList, Reporter& rpt) {
  list<Account *>::iterator iter;
  for (iter=accountList.begin(); iter != accountList.end(); iter++)
    (*iter)->Accept(rpt);
}

void 
main()
{
  PassbookAccount a("영희"), b("철수");
  CheckingAccount c("아빠"), d("엄마");

  list<Account *> accountList;
  accountList.push_front(&a);
  accountList.push_front(&b);
  accountList.push_front(&c);
  accountList.push_front(&d);

  TotalSumReporter sumRpt;
  HistoryReporter  histRpt;

  a.Deposit(1000); 
  b.Deposit(2000);
  c.Deposit(3000);
  d.Deposit(4000);

  DoReport(accountList, sumRpt);
  cout << endl;

  a.Withdraw(500);
  c.Withdraw(1000);

  DoReport(accountList, histRpt);
  cout << endl;
  DoReport(accountList, sumRpt);
}





#ifndef REPORTER_H
#define REPORTER_H

class PassbookAccount;
class CheckingAccount;

class Reporter {
  public :
    virtual void VisitPassbookAccount(PassbookAccount* pAccount) = 0;
    virtual void VisitCheckingAccount(CheckingAccount* pAccount) = 0;
};

class TotalSumReporter : public Reporter {
  public :
    void VisitPassbookAccount(PassbookAccount* pAccount);
    void VisitCheckingAccount(CheckingAccount* pAccount);
};

class HistoryReporter : public Reporter {
  public :
    void VisitPassbookAccount(PassbookAccount* pAccount);
    void VisitCheckingAccount(CheckingAccount* pAccount);
};

#endif






#include <iostream>
#include "reporter.h"
#include "account.h"
using namespace std;

void
TotalSumReporter::VisitPassbookAccount(PassbookAccount* pAccount) {
  cout << pAccount->GetCustomerName() 
        << "님의 PassbookAccount Total Sum : " 
        << pAccount->GetTotalSum() << endl;
}

void
TotalSumReporter::VisitCheckingAccount(CheckingAccount* pAccount) {
  cout << pAccount->GetCustomerName() 
        << "님의 CheckingAccount Total Sum : " 
        << pAccount->GetTotalSum() << endl;
}

// ----------------------------------------------------------------
void
HistoryReporter::VisitPassbookAccount(PassbookAccount* pAccount) {
  cout << pAccount->GetCustomerName() 
        << "님의 PassbookAccount 입출금 내역 : " << endl;
  list<int> inOutHist = pAccount->GetHistory();
  list<int>::iterator iter;
  for (iter=inOutHist.begin(); iter != inOutHist.end(); iter++) 
    cout << *iter << endl;
  cout << endl;
}

void
HistoryReporter::VisitCheckingAccount(CheckingAccount* pAccount) {
  cout << pAccount->GetCustomerName() 
        << "님의 CheckingAccount 입출금 내역 : " << endl;
  list<int> inOutHist = pAccount->GetHistory();
  list<int>::iterator iter;
  for (iter=inOutHist.begin(); iter != inOutHist.end(); iter++) 
    cout << *iter << endl;
  cout << endl;
}

```

* Visitor 패턴의 일반적인 클래스 구조

![image]({{ site.url }}{{ site.baseurl }}/assets/images/2022-07-11-book-design-pattern-03-01.png)


## 28\. 행위 수행 개선을 위한 디자인 패턴 정리
### 문제 사례 설명
* 굳이.