---
title:  "[C++]C++11, condition_variable 샘플 정리"
excerpt: "C++11에 추가된 condition_variable 를 맛보다."

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2015-07-27
last_modified_at: 2015-07-27
---

# intro

* CreateEvent 사용하는거랑 비슷. 단순 예제닌깐 leak 있는건 알아서 잘~


# sample
* 샘플
```c++
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
#include <condition_variable>
std::mutex mtx_lock1;
std::mutex mtx_lock2;
std::mutex mtx_lock3;
std::condition_variable cond1;
std::condition_variable cond2;
void test_condition_variable()
{
    //wait_for, wait_until 이런것도 있긴함.
    std::thread thread1([](){
        printf("%s: thread1 start\n", get_time_for_test());
        Sleep(3000);
        //해당 조건변수를 기다리고 있는 thread 중 하나를 깨운다.
        cond1.notify_one();
        printf("thread1 notify_one\n");
 
        printf("%s: thread1 end\n", get_time_for_test());
    });
    std::thread thread2([](){
        printf("%s: thread2 start\n", get_time_for_test());
        std::unique_lock<std::mutex> unique_lock(mtx_lock1);
        cond1.wait(unique_lock);
        printf("%s: thread2 wake up\n", get_time_for_test());
        Sleep(3000);
        //해당 조건변수를 기다리고 있는 thread 를 전부 깨운다.
        cond2.notify_all();
        printf("thread2 notify_all\n");
 
        printf("%s: thread2 end\n", get_time_for_test());
    });
    std::thread thread3([](){
        printf("%s: thread3 start\n", get_time_for_test());
        std::unique_lock<std::mutex> unique_lock(mtx_lock2);
        cond2.wait(unique_lock);
        printf("%s: thread3 end\n", get_time_for_test());
    });
    std::thread thread4([](){
        printf("%s: thread4 start\n", get_time_for_test());
        std::unique_lock<std::mutex> unique_lock(mtx_lock3);
        cond2.wait(unique_lock);
        printf("%s: thread4 end\n", get_time_for_test());
    });
    thread1.join();
    thread2.join();
    thread3.join();
    thread4.join();
}
```

* 결과는 이런식. 점점 편해지는 세상.
```
13:54:15: thread1 start
13:54:15: thread4 start
13:54:15: thread2 start
13:54:15: thread3 start
threadl notify_one
13:54:18: thread2 wake up
13:54:18: thread1 end
thread2 notify_all
13:54:21: thread3 end
13:54:21: thread4 end
13:54:21: thread2 end
```

