---
title:  "[C++]vector 순회 성능"
excerpt: "vector 순회 성능 테스트를 해 보았다."

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2015-08-03
last_modified_at: 2015-08-03
---

# intro
* 몇몇가지 테스트 한김에 vector 에 대한 순회 성능도 한번 보았다
* 실행PC 사양
```
프로세서: intel(R) Core(TM) i5-2400 CPU @ 3.10GHz 
설치된 메모리 4.00GB
시스템 종류: 32비트 운영 체제
```

# test code

```c++
#include <iostream>
#include <windows.h>
#include <list>
#include <vector>
#include <queue>
#include <string>
 
long long counter() {
    LARGE_INTEGER li;
    QueryPerformanceCounter(&li);
    return li.QuadPart;
}
 
long long get_frequency() {
    LARGE_INTEGER li;
    QueryPerformanceFrequency(&li);
    return li.QuadPart;
}
 
// to confine the test to run on a single processor in order to get consistent results for all tests.
void perf_startup() {
    SetThreadAffinityMask(GetCurrentThread(), 1);
    SetThreadIdealProcessor(GetCurrentThread(), 0);
    Sleep(1);
}
 
 
 
void test_vector_iterating(int intput_size)
{
    long long frequency = get_frequency();
    long long start = 0;
    long long finish = 0;
    double total = 0;
 
    printf("----------test_vector_iterating(%d)----------\n", intput_size);
    std::vector<DWORD> vector_;
    {
        for (int i = 0; i < intput_size; i++)
        {
            vector_.push_back(i);
        }
    }
 
    //case1 - None Const Iterating
    {
        start = counter();
 
        std::vector<DWORD>::iterator it = vector_.begin();
        std::vector<DWORD>::iterator it_end = vector_.end();
        long long temp = 0;
 
        for (; it != it_end; ++it)
        {
            temp += (*it);
        }
 
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("None Const Iterating: \t\t\t%f sec, %lld\n", total, temp);
 
    }
    //case2 - Const Iterating
    {
        start = counter();
 
        std::vector<DWORD>::const_iterator it = vector_.begin();
        std::vector<DWORD>::const_iterator it_end = vector_.end();
        long long temp = 0;
 
        for (; it != it_end; ++it)
        {
            temp += (*it);
        }
 
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("Const Iterating: \t\t\t%f sec, %lld\n", total, temp);
 
    }
    //case3 - std::for_each( Functor ) 사용
    {
        start = counter();
        long long temp = 0;
 
        struct STRUCT_SUM
        {
            STRUCT_SUM(long long* t):total(t){};
            long long* total;
 
            void operator()(DWORD element)
            {
                *total += element;
            };
        };
 
        STRUCT_SUM st_sum(&temp);
 
        std::for_each(vector_.begin(), vector_.end(), st_sum);
 
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("std::for_each( Functor ): \t\t%f sec, %lld\n", total, temp);
 
    }
 
    //case4 - for each(element in Container)
    {
        start = counter();
        long long temp = 0;
 
        for each(DWORD element in vector_)
        {
            temp += element;
        };
 
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("for each(element in Container): \t%f sec, %lld\n", total, temp);
 
    }
}

int main(void)
{
    test_vector_iterating(10000);
    test_vector_iterating(100000);
    test_vector_iterating(1000000);
    test_vector_iterating(10000000);
 
    __asm int 3;
    return 1;
}
```

# 실행결과
* 결과가 나름 잼있었는데, 
* 우선 VS2008 debug 와 release에선 아래처럼 결과가 나왔다.
* debug 모드

```
----------test_vector_iterating(10000)----------
None Const Iterating: 						0.002383 sec,
Const Iterating: 						0.001859 sec,
std::for_each( Functor ): 					0.002610 sec,
for each(element in Container):		    0.001847 sec,

----------test_vector_iterating(100000)----------
None Const Iterating: 						0.024610 sec,
Const Iterating: 						0.018763 sec,
std::for_each( Functor ): 					0.025965 sec,
for each(element in Container):		    0.018454 sec,
----------test_vector_iterating(1000000)----------
None Const Iterating: 						0.237435 sec,
Const Iterating: 						0.191960 sec,
std::for_each( Functor ): 					0.265545 sec,
for each(element in Container):		    0.185883 sec,
----------test_vector_iterating(10000000)----------
None Const Iterating: 						2.380271 sec,
Const Iterating: 						1.867360 sec,
std::for_each( Functor ): 					2.602249 sec,
for each(element in Container):		    1.859658 sec,
```
* release 모드

```
----------test_vector_iterating(10000)----------
None Const Iterating: 						0.000062 sec,
Const Iterating: 								0.000059 sec,
std::for_each( Functor ): 					0.000007 sec,
for each(element in Container):		0.000056 sec,

----------test_vector_iterating(100000)----------
None Const Iterating: 						0.000637 sec,
Const Iterating: 								0.000589 sec,
std::for_each( Functor ): 					0.000070 sec,
for each(element in Container):		0.000556 sec,
----------test_vector_iterating(1000000)----------
None Const Iterating: 						0.006262 sec,
Const Iterating: 								0.005918 sec,
std::for_each( Functor ): 					0.000769 sec,
for each(element in Container):		0.005839 sec,
----------test_vector_iterating(10000000)----------
None Const Iterating: 						0.058077 sec,
Const Iterating: 								0.054475 sec,
std::for_each( Functor ): 					0.007643 sec,
for each(element in Container):		0.051237 sec,
```
* functor가 inline 하기에 적합하다고 알려진만큼 functor을 사용한 경우 성능이 월등히 뛰어남을 볼수가 있다


* 그런데 VS2013에서는 다음과 같이 결과가 나왔다.
* debug모드에선 

```
----------test_vector_iterating(10000)----------
None Const Iterating: 						0.007786 sec,
Const Iterating: 								0.004365 sec,
std::for_each( Functor ): 					0.000317 sec,
for each(element in Container):		0.004041 sec,

----------test_vector_iterating(100000)----------
None Const Iterating: 						0.039324 sec,
Const Iterating: 								0.034428 sec,
std::for_each( Functor ): 					0.002614 sec,
for each(element in Container):		0.034356 sec,
----------test_vector_iterating(1000000)----------
None Const Iterating: 						0.392254 sec,
Const Iterating: 								0.342338 sec,
std::for_each( Functor ): 					0.026226 sec,
for each(element in Container):		0.342381 sec,
----------test_vector_iterating(10000000)----------
None Const Iterating: 						3.923250 sec,
Const Iterating: 								3.451026 sec,
std::for_each( Functor ): 					0.288544 sec,
for each(element in Container):		3.434354 sec,
```

* release모드에선 이렇게 여러가지 방법으로 했을때 모두, 이전 VS2008에서 functor를 사용할때만큼 성능이 다 좋게 나왔다. 

```
----------test_vector_iterating(10000)----------
None Const Iterating: 						0.000007 sec,
Const Iterating: 								0.000007 sec,
std::for_each( Functor ): 					0.000007 sec,
for each(element in Container):		0.000007 sec,

----------test_vector_iterating(100000)----------
None Const Iterating: 						0.000068 sec,
Const Iterating: 								0.000068 sec,
std::for_each( Functor ): 					0.000068 sec,
for each(element in Container):		0.000068 sec,
----------test_vector_iterating(1000000)----------
None Const Iterating: 						0.000752 sec,
Const Iterating: 								0.000747 sec,
std::for_each( Functor ): 					0.000791 sec,
for each(element in Container):		0.000726 sec,
----------test_vector_iterating(10000000)----------
None Const Iterating: 						0.008127 sec,
Const Iterating: 								0.007539 sec,
std::for_each( Functor ): 					0.009552 sec,
for each(element in Container):		0.007675 sec,
```

* 결국, 컴파일러가 좋아질수록 ,성능을 생각한다고 매우 고민하며 알아보기 힘들게? 코드를 작성하는것보단 알아보기 쉽게 작성해도 그다지 별 문제가 없을것이란 결론?
