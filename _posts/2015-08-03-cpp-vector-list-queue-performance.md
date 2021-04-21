---
title:  "[C++]간단한 vector, list, queue 성능비교 테스트"
excerpt: "간단히 vector, list, queue 의 성능비교 테스트를 해 보았다."

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
* vector, list, queue에 대한 간략한 성능비교(뒤늦게마나 vs2013사용한 기념)
* 한 차례 수행 후 값을 내었고(귀찮..), 테스트 환경(여러가지가 돌고있어서 결과값은 상대적으로 봐야할듯)은 아래처럼
* 실행PC 사양
```
프로세서: intel(R) Core(TM) i5-2400 CPU @ 3.10GHz 
설치된 메모리 4.00GB
시스템 종류: 32비트 운영 체제
```
*  정교한 테스트는 아니기 때문에 대략 이러한 형태를 가지고 있다라고만 보면 될듯, 데이터 타입의 다양성과 여러차례에 걸친 평균등 이 필요하긴하다.
	* https://baptiste-wicht.com/posts/2012/12/cpp-benchmark-vector-list-deque.html
	* (나~~~ 중에 시간이 되면 이정되는 테스트는 해보고 싶긴함.)

# test code

```c++
#include <iostream>
#include <list>
#include <vector>
#include <queue>
#include <string>
#include <windows.h>
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
 
 
 
void clear_queue(std::queue<DWORD> &someQueue)
{
    std::queue<DWORD> empty;
    std::swap(someQueue, empty);
}
 
void test_compare_stl(int intput_size)
{
    printf("\n");
//    perf_startup();
    long long frequency = get_frequency();
    long long start = 0;
    long long finish = 0;
    double total = 0;
 
    // queue
    {
        std::cout << "----------queue(" << intput_size  << ")----------" << std::endl;
        std::queue<DWORD> queue_;
        {
            start = counter();
 
            for (int i = 0; i < intput_size; i++)
            {
                queue_.push(i);
            }
            finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("queue insert: \t%f sec\n", total);
        }
        {
            start = counter();
 
            long long temp = 0;
            while (!queue_.empty())
            {
                temp += queue_.front();
                queue_.pop();
            }
 
            finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("queue pop: \t%f sec, %lld\n", total, temp);
        }
        start = counter();
    }
    finish = counter();
    total = ((finish - start)*1.0 / frequency);
    printf("queue release: \t%f sec\n", total);
 
 
 
 
    // list
    {
        std::cout << "----------stl----------" << std::endl;
        std::list<DWORD> list_;
        {
            start = counter();
 
            for (int i = 0; i < intput_size; i++)
            {
                list_.push_back(i);
            }
            finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("list insert: \t%f sec\n", total);
        }
        {
            start = counter();
 
            std::list<DWORD>::iterator it;
            long long temp = 0;
            for (it = list_.begin(); it != list_.end(); it++)
            {
                temp += (*it);
            }
            finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("list loop: \t%f sec, %lld\n", total, temp);
        }
        {
            start = counter();
            list_.clear();
            finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("list clear: \t%f sec\n", total);
        }
 
        start = counter();
    }
    finish = counter();
    total = ((finish - start)*1.0 / frequency);
    printf("list release: \t%f sec\n", total);
 
 
    // vector
    {
        std::cout << "----------vector----------" << std::endl;
        std::vector<DWORD> vector_;
        {
            long long start = counter();
 
            for (int i = 0; i < intput_size; i++)
            {
                vector_.push_back(i);
            }
            long long finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("vector insert: \t%f sec\n", total);
        }
        {
            long long start = counter();
 
            std::vector<DWORD>::iterator it;
            long long temp = 0;
            for (it = vector_.begin(); it != vector_.end(); it++)
            {
                temp += (*it);
            }
            long long finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("vector loop: \t%f sec, %lld\n", total, temp);
        }
        {
            long long start = counter();
 
            vector_.clear();
 
            long long finish = counter();
            total = ((finish - start)*1.0 / frequency);
            printf("vector clear: \t%f sec\n", total);
        }
 
        start = counter();
    }
    finish = counter();
    total = ((finish - start)*1.0 / frequency);
    printf("vector release: %f sec\n", total);
 
 
}
 
 
 
int main(void)
{
    test_compare_stl(100 + 1);
    test_compare_stl(500 + 1);
    test_compare_stl(1000 + 1);
    test_compare_stl(5000 + 1);
    test_compare_stl(10000 + 1);
    test_compare_stl(50000 + 1);
    test_compare_stl(100000 + 1);
    test_compare_stl(500000 + 1);
    test_compare_stl(1000000 + 1);
    test_compare_stl(5000000 + 1);
 
    __asm int 3;
    return 1;
}
```

# 실행결과

|  |  vs2008 list insert |  vs2008 list loop | vs2008 list clear | vs2008 vector insert |  vs2008 vector loop |  vs2008 vector clear |
| --- | --- | --- | --- | --- | --- | --- |
|  100건         |  0.000061 |  0.000001 | 0.000057 | 0.000019 | 0.000001 | 0 |
|  500건         |  0.000381 |  0.000003 | 0.000972 | 0.000047 | 0.000003 | 0 |
|  1000건       |  0.001411 |  0.000005 | 0.003763 | 0.0001 | 0.000006 | 0 |
|  5000건       |  0.007481 |  0.000027 | 0.061879 | 0.00015 | 0.00003 | 0 |
|  10000건     |  0.065947 |  0.000058 | 0.305491 | 0.000305 | 0.00006 | 0 |
|  50000건     |  0.678637 |  0.000306 | 2.312486 | 0.00077 | 0.000299 | 0 |
|  100000건   |  0.237424 |  0.000629 | 4.752217 | 0.000933 | 0.000598 | 0 |
|  500000건   |  0.736932 |  0.003109 | 24.430509 | 0.00321 | 0.002991 | 0 |
|  1000000건 |  1.806386 |  0.006019 | 49.198097 | 0.005039 | 0.005983 | 0 |
|  5000000건 |  6.409442 |  0.029849 | 248.287567 |0.025977 | 0.030323 | 0 |

|  |  vs2013 list insert |  vs2013 list loop | vs2013 list clear | vs2013 vector insert |  vs2013 vector loop |  vs2013 vector clear |
| --- | --- | --- | --- | --- | --- | --- |
|  100건         |  0.000257 | 0.000002 | 0.000115 | 0.000031 | 0.000001 | 0 |
|  500건         |  0.000786 | 0.000005 | 0.001994 | 0.000081 | 0.000001 | 0 |
|  1000건       |  0.002748 | 0.000006 | 0.006578 | 0.000099 | 0.000001 | 0 |
|  5000건       |  0.012442 | 0.000013 | 0.094908 | 0.000531 | 0.000004 | 0 |
|  10000건     |  0.099526 | 0.000046 | 0.382695 | 0.001168 | 0.000007 | 0 |
|  50000건     |  0.031496 | 0.000185 | 2.394466 | 0.001156 | 0.000035 | 0 |
|  100000건   |  0.647545 | 0.000473 | 4.811623 | 0.00149 | 0.00007 | 0 |
|  500000건   |  1.802835 | 0.00219 | 24.720386 | 0.003305 | 0.00035 | 0 |
|  1000000건 |  3.155605 | 0.004051 | 48.326815 | 0.006048 | 0.00082 | 0 |
|  5000000건 |  5.979239 | 0.019601 | 248.121181 | 0.028945 | 0.003951 | 0 |


|  |  vs2008 queue insert |  vs2008 queue pop | vs2013 queue insert | vs2013 queue pop | 
| --- | --- | --- | --- | --- | 
|  100건         | 0.000019 | 0.000001 | 0.000068 | 0.000002 |
|  500건         | 0.000131 | 0.000004 | 0.000286 | 0.000007 |
|  1000건       | 0.00059 | 0.000007 | 0.001328 | 0.000014 |
|  5000건       | 0.00373 | 0.000035 | 0.003852 | 0.000035 |
|  10000건     | 0.027921 | 0.000072 | 0.058983 | 0.000065 |
|  50000건     | 0.216583 | 0.000349 | 0.308407 | 0.000321 |
|  100000건   | 0.312508 | 0.000704 | 0.889107 | 0.000651 ||
|  500000건   | 1.383747 | 0.003529 | 0.977399 | 0.003263 |
|  1000000건 | 1.722964 | 0.007054 | 1.476969 | 0.006659 |
|  5000000건 | 2.282444 | 0.035637 | 2.119721 | 0.032525 |

* 원래  위의 자료로 표도 있었지만.... 