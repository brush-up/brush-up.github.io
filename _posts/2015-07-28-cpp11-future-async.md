---
title:  "[C++]C++11, future, std::async 샘플 정리"
excerpt: "C++11에 추가된 future, std::async 를 맛보다."

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2015-07-28
last_modified_at: 2015-07-28
---

# sample
* thread 와는 조금은 다른 async 
```c++
#include <future>
char* get_time_for_test()
{
    struct tm *newtime;
    __time64_t long_time;
    _time64(&long_time);
    newtime = _localtime64(&long_time);
    char* temp = new char[255];
    memset(temp, 0, 255);
    sprintf(temp, "%02d:%02d:%02d", newtime->tm_hour, newtime->tm_min, newtime->tm_sec);
    return temp;
} 
int test_sub_async(int arg1)
{
    printf("%s: [sub thread] start: %d\n", get_time_for_test(), arg1);
    Sleep(2000);
    printf("%s: [sub thread] end\n", get_time_for_test());
    return arg1 + 20;
}
void test_async()
{
    //std::launch::async 는 async 정의한후 바로 비동기로 실행
    //std::launch::deferred 는 future를 사용하여 결과를 대기할때 실행
    //std::launch::any, std::launch::sync 는 불필요
    {
        printf("------------------------std::launch::async 1---------------------------\n");
        printf("%s: [main thread] before - async\n", get_time_for_test());
        std::future<int> future = std::async(std::launch::async, test_sub_async, 30);
        printf("%s: [main thread] after - async1\n", get_time_for_test());
        printf("%s: [main thread] call - future\n", get_time_for_test());
        int output = future.get();
        printf("%s: [main thread] after - future1 : %d\n", get_time_for_test(), output);
        Sleep(3000);
        printf("%s: [main thread] after - future2 : %d\n", get_time_for_test(), output);
    }
    {
        printf("------------------------std::launch::async 2---------------------------\n");
        printf("%s: [main thread] before - async\n", get_time_for_test());
        std::async(std::launch::async, test_sub_async, 30);
        printf("%s: [main thread] after - async1\n", get_time_for_test());
        Sleep(3000);
        printf("%s: [main thread] after - async2\n", get_time_for_test());
    }
    {
        printf("------------------------std::launch::deferred 1---------------------------\n");
        printf("%s: [main thread] before - async\n", get_time_for_test());
        std::future<int> future = std::async(std::launch::deferred, test_sub_async, 30);
        printf("%s: [main thread] after - async1\n", get_time_for_test());
        printf("%s: [main thread] call - future\n", get_time_for_test());
        int output = future.get();
        printf("%s: [main thread] after - future1 : %d\n", get_time_for_test(), output);
        Sleep(3000);
        printf("%s: [main thread] after - future2 : %d\n", get_time_for_test(), output);
    }
    {
        printf("------------------------std::launch::deferred 2---------------------------\n");
        printf("%s: [main thread] before - async\n", get_time_for_test());
        std::async(std::launch::deferred, test_sub_async, 30);
        printf("%s: [main thread] after - async1\n", get_time_for_test());
        Sleep(3000);
        printf("%s: [main thread] after - async2\n", get_time_for_test());
    }
 
    printf("------------------------test_async end---------------------------\n");
 
    //누군가 이미 테스트를 했네.
	//http://www.ducons.com/blog/tests-and-thoughts-on-asynchronous-io-vs-multithreading
}
```

# 실행결과
* 실행 결과는 대충 이런식
```------------------------std::launch::async 1---------------------------
29:15: [main thread] before - async
29:15: [main thread] after - async1
29:15: [main thread] call - future
29:15: [sub thread] start: 30
29:17: [sub thread] end
29:17: [main thread] after - future1 : 50
29:20: [main thread] after - future2 : 50 
------------------------std::launch::async 2---------------------------
29:20: [main thread] before - async
29:20: [main thread] after - async1
29:20: [sub thread] start: 30
29:22: [sub thread] end
29:23: [main thread] after - async2
------------------------std::launch::deferred 1---------------------------
29:23: [main thread] before - async
29:23: [main thread] after - async1
29:23: [main thread] call - future
29:23: [sub thread] start: 30
29:25: [sub thread] end
29:25: [main thread] after - future1: 50
29:28: [main thread] after - future2 : 50
------------------------std::launch::deferred 2---------------------------
29:28: [main thread] before - async
29:28: [main thread] after - async1
29:31: [main thread] after - async2
------------------------test_async end---------------------------
```

# 참고
* 이것을 더 보는게 도움이 될듯
	* http://stackoverflow.com/questions/4024056/threads-vs-async
	* http://stackoverflow.com/questions/600795/asynchronous-vs-multithreading-is-there-a-difference
