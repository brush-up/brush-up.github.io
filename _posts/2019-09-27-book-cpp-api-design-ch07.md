---
title:  "[BOOK]C++ API 디자인 - CHAPTER 07 성능"
excerpt: "C++ API 디자인 - CHAPTER 07 성능"

categories:
  - Book
tags:
  - [Book, Programming]

toc: true
toc_sticky: true
 
date: 2019-09-27
last_modified_at: 2019-09-27

published: false
---
# CHAPTER 07 성능

* 7.1 상수 참조로 입력 파라미터 전달
	* 함수가 입력 파라미터를 변경하지 않는 상황이라면 포인터보다 상수 참조를 통해 입력 파라미터를 전달하는 것이 더 좋다.
	* 출력 파라미터의 경우 비상수 참조보다는 포인터를 사용해야 한다. 그 이유는 클라이언트에게 이 변수는 분명히 변경될수 있다고 인지하는 효과가 있다.
* 7.2 #INCLUDE 의존성 최소화
* 7.2.1 한곳에 몰아두는 헤더 금지
* 7.2.2 전방 선언
* 7.2.3 #include 중복 방지

* 7.3 상수선언
	* 전역 상수를 extern으로 선언하거나 클래스 내에서 static const 상수로 선언하라. 그 후에는 .cpp파일에서 상수의 값을 정의한다. 
* 7.3.1 새로운 constexpr 키워드

* 7.4 초기화 리스트
	* 생성자에서 각각의 멤버 변수를 간단히 초기화 하면 어느정도의 성능 향상을 얻을 수 있다. 
	```cpp
	class Avatar
	{
	public:
	Avatar(const std::string &first, const std::string &last)
	{
	mFirstName first;
	mLastName last;
	}
	private:
	std::string mFirstName;
	std::string mLastName;
	};
	```
	* 이것보다는 

	```cpp
	class Avatar
	{
	public:
	Avatar(const std::string &first, const std::string &last) :
	mFirstName(first),
	mLastName(last)
	{}
	private:
	std::string mFirstName;
	std::string mLastName;
	};

	```
	* 이것이 좋다. 그 이유는 멤버 변수들은 생성자가 호출되기 이전에 생성된다. 
	* 첫번째 예제 코드에서 mName멤버 변수를 초기화 하기 ㅜ이해 std::string 객체의 기본 생성자를 호출한 뒤 다시 생성자 내에서 할당 연산자를 호출한다. 
	* 그러나 두번째 예제의 경우 오직 할당 연사자만 사용된다. 
	* 결국 초기화 리스트를 사용하면 초기화 리스트에 명시한 각각의 멤버 변수들은 기본 생성자를 호출하지 않고도 초기화 할 수 있다.


* 7.5 메모리 최적화
	* 타입별로 멤버 변수 묶기-현대의 컴퓨터는 word 단위로 메모리에 접근하기에 C++컴파일러는 word크기의 영역에 메모리 주소를 할당하도록 특정 데이터 멤버별로 정렬할 것이다.
	* 필요하기 전까지 가상 메소더를 추가하지 않기
	```cpp
	class Fireworks A
	{
	bool mIsActive;
	int mOriginX;
	int mOriginY;
	bool mVaryColor;
	char mRed;
	int mRedVariance;
	char mGreen;
	int mGreenVariance;
	char mBlue;
	int mBlueVariance;
	bool mRepeatCycle;
	int mTotalParticles;
	bool mFadeParticles;
	};
	```
	* 위에 것보다 아래처럼 하면 이 구조체의 크기는 32바이트가 되고 33%의 공간이 줄어든다.
	```cpp
	class Fireworks A
	{
	bool mIsActive;
	bool mVaryColor;
	bool mRepeatCycle;
	bool mFadeParticles;
	char mRed;
	char mGreen;
	char mBlue;
	int mOriginX;
	int mOriginY;
	int mRedVariance;
	int mGreenVariance;
	int mBlueVariance;
	int mTotalParticles;
	};
	```
	* 여기서 비트필드를 사용하면 더 줄일수는 있다. 
	![image]({{ site.url }}{{ site.baseurl }}/assets/images/2019-09-27-book-cpp-api-design-ch07-001.png)


* 7.6 필요시까지 인라인 코드 사용 금지

* 7.7 카피-온-라이트

* 7.8 요소 반복
* 7.8.1 반복자
* 7.8.2 임의접근
* 7.8.3 배열 참조


* 7.9 성능분석
* 7.9.1 시간 기반 분석
* 7.9.2 메모리 기반 분석
* 7.9.3 멀티스레드 분석
