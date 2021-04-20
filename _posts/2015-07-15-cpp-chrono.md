---
title:  "[C++]C++11, chrono 샘플 정리"
excerpt: "java11에 추가된 http client를 사용해볼까?"

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2015-07-15
last_modified_at: 2015-07-15
---

# intro

* 라이센스문제?로 VS2008만 쓰다가 드디어 VS2013을 쓰기 시작하면서 C++11을 슬쩍 보고있는데, C++ 표준에 드디어 좋은게 생겼었구나라고 좋아라하고있음

# sample
```c++
#include <chrono>
//기존에 GetTickCount, QueryPerformanceCounter, QueryPerformanceFrequency 등을 이용했었는데 완전 편해졌다. 
void test_chrono()
{
    std::chrono::system_clock::time_point start_time = std::chrono::system_clock::now();
    Sleep(2000);
    std::chrono::system_clock::time_point end_time = std::chrono::system_clock::now();
    // 나노 초 단위 (1/1000000000)
    std::chrono::nanoseconds nano = std::chrono::duration_cast<std::chrono::nanoseconds>(end_time - start_time);
    //마이크로 초 단위 (1/1000000)
    std::chrono::microseconds micro = std::chrono::duration_cast<std::chrono::microseconds>(end_time - start_time);
    //밀리 초 단위 (1/1000)
    std::chrono::milliseconds mill = std::chrono::duration_cast<std::chrono::milliseconds>(end_time - start_time);
    std::chrono::seconds sec = std::chrono::duration_cast<std::chrono::seconds>(end_time - start_time);
    std::chrono::minutes min = std::chrono::duration_cast<std::chrono::minutes>(end_time - start_time);
    std::chrono::hours hour = std::chrono::duration_cast<std::chrono::hours>(end_time - start_time);
    std::cout << "sleep time: " << nano.count() << " nanoseconds" << std::endl;
    std::cout << "sleep time: " << micro.count() << " microseconds" << std::endl;
    std::cout << "sleep time: " << mill.count() << " milliseconds" << std::endl;
    std::cout << "sleep time: " << sec.count() << " seconds" << std::endl;
    std::cout << "sleep time: " << min.count() << " minutes" << std::endl;
    std::cout << "sleep time: " << hour.count() << " hour" << std::endl;
}
```