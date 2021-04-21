---
title:  "[C++]C++11, rvalue reference에 대한 성능측정"
excerpt: "C++11에서 rvalue reference에 대한 성능 비교를 해보았다."

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2015-07-28
last_modified_at: 2015-07-28
---

* 이 당시 왜 이렇게 했었는지 기억이  1도 안난다.

# intro
* C++11 에서 RValue Reference 사용으로 인항 성능 향상이 있다고 해서 몇몇가지 테스트를 해봄
* 몇차례 수행 후 평균값을 내었고, 테스트 환경(여러가지가 돌고있어서 결과값은 상대적으로 봐야할듯)은 아래처럼
* 실행PC 사양
```
프로세서: intel(R) Core(TM) i5-2400 CPU @ 3.10GHz 
설치된 메모리 4.00GB
시스템 종류: 32비트 운영 체제
```

# test

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
 
void test_rvalue_reference(int count)
{
    std::cout << "----------test_rvalue_reference----------" << std::endl;
 
    long long frequency = get_frequency();
    long long start = 0;
    long long finish = 0;
    double total = 0;
    int temp = 0;
 
    //case1
    {
        start = counter();
 
        for (int i = 0; i < count; i++)
        {
            std::string msg1("Hello world !");
            std::string msg2("Hello world !");
            std::string msg3("Hello world !");
 
            std::string output = msg1 + " " + msg2 + " " + msg3;
            temp += output.size();
        }
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("test_rvalue_reference1: \t%f sec, %d\n", total, temp);
 
    }
 
    //case2
    {
        long long total_copy_move_count = 0;
        long long total_vector_resizes = 0;
        double total_pushback_time = 0.0;
 
        std::vector<std::string> v_str;
 
        start = counter();
 
        for (int i = 0; i<count; ++i)
        {
            size_t size = v_str.size();
            size_t capacity = v_str.capacity();
            v_str.push_back("hello world!");
 
            // indication that vector reallocation has occurred.
            if(size == capacity) 
            {
                ++total_vector_resizes;
                total_copy_move_count += size;
            }
        }
        finish = counter();
        total = ((finish - start)*1.0 / frequency);
        printf("test_rvalue_reference2: \t%f sec, %lld, %lld\n", total, total_vector_resizes, total_copy_move_count);
     }
}
 
int main(void)
{
    test_rvalue_reference(100000);
    test_rvalue_reference(200000);
    test_rvalue_reference(300000);
    return 1;
}
```

# 실행결과
* 성능차이가 예상보다..

|  |  case1 |  case2 |
| --- | --- | --- |
|  VS2008 (100000번)|  0.715383 sec|  0.024797 sec|
|  VS2008 (200000번)|  1.416200 sec|  0.042661 sec|
|  VS2008 (300000번)|  2.123057 sec|  0.063813 sec|
|  VS2013 (100000번)|  0.122426 sec|  0.008985 sec|
|  VS2013 (100000번)|  0.236044 sec|  0.013602 sec|
|  VS2013 (100000번)|  0.354466 sec|  0.020742 sec|

