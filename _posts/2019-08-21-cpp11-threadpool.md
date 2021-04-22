---
title:  "[C++]C++11, threadpool 샘플"
excerpt: "C++11 threadpool 샘플을 살펴보자."

categories:
  - Cpp
tags:
  - [Cpp, Programming]

toc: true
toc_sticky: true
 
date: 2019-08-21
last_modified_at: 2019-08-21
---

# 참고
* http://progsch.net/wordpress/?p=106



# 샘플코드

```c++
#ifndef THREAD_POOL_H
#define THREAD_POOL_H

#include <vector>
#include <queue>
#include <memory>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <stdexcept>

class ThreadPool {
public:
	ThreadPool(size_t);
	template<class F, class... Args>
	auto enqueue(F&& f, Args&&... args)
		->std::future<typename std::result_of<F(Args...)>::type>;
	~ThreadPool();
private:
	// need to keep track of threads so we can join them
	std::vector< std::thread > workers;
	// the task queue
	std::queue< std::function<void()> > tasks;

	// synchronization
	std::mutex queue_mutex;
	std::condition_variable condition;
	bool stop;
};

// the constructor just launches some amount of workers
inline ThreadPool::ThreadPool(size_t threads)
: stop(false)
{
	for (size_t i = 0; i<threads; ++i)
		workers.emplace_back(
		[this]
	{
		for (;;)
		{
			std::function<void()> task;
			{
				std::unique_lock<std::mutex> lock(this->queue_mutex);
				this->condition.wait(lock,
					[this]{ return this->stop || !this->tasks.empty(); });
				if (this->stop && this->tasks.empty())
					return;
				task = std::move(this->tasks.front());
				this->tasks.pop();
			}

			task();
		}
	}
	);
}

// add new work item to the pool
template<class F, class... Args>
auto ThreadPool::enqueue(F&& f, Args&&... args)
-> std::future<typename std::result_of<F(Args...)>::type>
{
	using return_type = typename std::result_of<F(Args...)>::type;

	auto task = std::make_shared< std::packaged_task<return_type()> >(
		std::bind(std::forward<F>(f), std::forward<Args>(args)...)
		);

	std::future<return_type> res = task->get_future();
	{
		std::unique_lock<std::mutex> lock(queue_mutex);

		// don't allow enqueueing after stopping the pool
		if (stop)
			throw std::runtime_error("enqueue on stopped ThreadPool");

		tasks.emplace([task](){ (*task)(); });
	}
	condition.notify_one();
	return res;
}

// the destructor joins all threads
inline ThreadPool::~ThreadPool()
{
	{
		std::unique_lock<std::mutex> lock(queue_mutex);
		stop = true;
	}
	condition.notify_all();
	for (std::thread &worker : workers)
		worker.join();
}

#endif
```

## 사용
* 아래 사용예시는 검증 해야함.
```c++
//test thread pool
#include "ThreadPool.h"
class TEST_CLASS2
{
public:
	TEST_CLASS2(int threadCount)
	{
		pool_ = new ThreadPool(threadCount);
	};

	virtual ~TEST_CLASS2()
	{
	};

	void member1_of_class(int args)
	{
		std::cout << "call member2_of_class before: " << args << std::endl;
		pool_->enqueue(&TEST_CLASS2::member2_of_class, this, args, 5);
		std::cout << "call member2_of_class after: " << args << std::endl;
	};

	void member2_of_class(int args1, int args2)
	{
		std::lock_guard<std::mutex> guard(mtx_lock_);

		Sleep(1000);
		std::thread::id thread_id = std::this_thread::get_id();
		std::cout << "thread id : " << thread_id << " : " << args1 << ", " << args2 << std::endl;
	};

private:
	std::mutex mtx_lock_;
	ThreadPool* pool_;
};


void test_thread_pool()
{
	TEST_CLASS2 test_class(2);
	for (int i = 0; i < 1000; i++)
	{
		test_class.member1_of_class(i);
	}

	while (true)
	{

	}
}
```
