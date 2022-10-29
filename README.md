# Concurrency in C++
## When to use thread
<ol>
  <li>Heavy Computaions</li>
  <li>Using threads to seperate work</li>
</ol>

## Thread Libraries in C
- Before C++11/14/17/20, there existed threading libraries with different semantics
- Libraries like "Boost", Intel "Thread Building Blocks", or "pthread" were used 
  - Perhaps you have used pthread at least in C
  - (std::thread I believe is implemented with pthread most posix systems)
- Typically today I would personally recommend using the standard C++ threading library for portability reasons as the default choice.

## C++ thread liabrary
[Concurrency support library](https://en.cppreference.com/w/cpp/thread)

## First C++ thread example
```
// @file thread1.cpp
#include <iostream>
#include <thread>

void test(int x){
    std::cout << "Hello from thread!" << std::endl;
    std::cout << "Argument passed in: " << x << std::endl;
}


int main(){

    std::thread myThread(&test, 100);
    myThread.join();

    std::cout << "hello from my main thread" << std::endl;
    return 0;
}
```
```
...$ g++ -std=c++17 thread1.cpp -o prog -lpthread  
...$ ./prog
```
notice:  
- the code ```-lpthread ``` is used to link in the a thread liabrary.  
- ```myThread.join();``` means the main thread will wait fot the thread myThread.  

## std::thread with a lambda in modern c++  
```
// @file thread2.cpp
#include <iostream>
#include <thread>
int main(){

    auto lambda=[](int x){
        std::cout << "Hello from thread!" << std::endl;
        std::cout << "Argument passed in: " << x << std::endl;
    };

    std::thread myThread(lambda, 100);
    myThread.join();

    std::cout << "hello from my main thread" << std::endl;
    return 0;
}
```
notice:  
- lambda function returns a r value reference
- te constructor grammer:  
```
template< class Function, class... Args >
explicit thread( Function&& f, Args&&... args );
``` 
## Launching multiple std::thread in C++
```
// @file thread3.cpp
#include <iostream>
#include <thread>
#include <vector>

int main(){

    auto lambda=[](int x){
        std::cout << "Hello from thread!" << std::this_thread::get_id() << std::endl;
        std::cout << "Argument passed in: " << x << std::endl;
    };

    std::vector<std::thread> threads;
    for(int i=0; i < 10; i++){
        threads.push_back(std::thread(lambda, i));
    }

    for(int i=0; i < 10; i++){
        threads[i].join(); 
    }

    std::cout << "hello from my main thread" << std::endl;
    return 0;
}
```
notice:
 - if we write the code as below, the thread in the vector will be executed in order.
 - It told the compiler that any new thread must wait for the former thread to be terminated. 
 - This is not a right way to write code because it is actually not concurrency.
 ```
    std::vector<std::thread> threads;
    for(int i=0; i < 10; i++){
        threads.push_back(std::thread(lambda, i));
        threads[i].join(); 
    }
 ```
 ## jthread std::jthread in C++ 20
 ```
 // @file thread4.cpp
#include <iostream>
#include <thread>
#include <vector>

int main(){

    auto lambda=[](int x){
        std::cout << "Hello from thread!" << std::this_thread::get_id() << std::endl;
        std::cout << "Argument passed in: " << x << std::endl;
    };

    std::vector<std::jthread> jthreads;
    for(int i=0; i < 10; i++){
        jthreads.push_back(std::jthread(lambda, i));
    }


    std::cout << "hello from my main thread" << std::endl;
    return 0;
}
```
notice:
- the  ```std::jthread``` dose not need join procedure.
- the  ```std::jthread``` will not execute sequentially because the join process will be done out of the current scope.
 
## std::mutex and preventing data races in C++

```
// @file thread5.cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

std::mutex gLock;
static int shared_value= 0;

void shared_value_increment(){
    gLock.lock();
        shared_value = shared_value + 1;
    gLock.unlock();
}

int main(){

    std::vector<std::thread> threads;
    for(int i=0; i < 1000; i++){
        threads.push_back(std::thread(shared_value_increment));
    }

    for(int i=0; i < 1000; i++){
        threads[i].join(); 
    }

    std::cout << "Shared value:" << shared_value << std::endl;
    return 0;
}
```
notice:
- mutex will lock the access to the shared data when one thread comes into execution.

## Preventing deadlock with std::lock_guard in modern C++  
- The class lock_guard is a mutex wrapper that provides a convenient RAII-style mechanism for owning a mutex for the duration of a scoped block.
- When a lock_guard object is created, it attempts to take ownership of the mutex it is given. When control leaves the scope in which the lock_guard object was created, the lock_guard is destructed and the mutex is released.
```
// @file thread6.cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

std::mutex gLock;
static int shared_value= 0;

void shared_value_increment(){
    std::lock_guard<std::mutex> lockGuard(gLock);
    shared_value = shared_value + 1;
}

int main(){

    std::vector<std::thread> threads;
    for(int i=0; i < 1000; i++){
        threads.push_back(std::thread(shared_value_increment));
    }

    for(int i=0; i < 1000; i++){
        threads[i].join(); 
    }

    std::cout << "Shared value:" << shared_value << std::endl;
    return 0;
}
```
notice: 
- std::lock_guard<std::mutex> is especially usefull in code with exception because manually setting unlock won't cover everywhere throwing a exception.
## Using std::atomic in modern C++ to update a shared valueUsing std::atomic in modern C++ to update a shared value
[std::atomic](https://en.cppreference.com/w/cpp/atomic/atomic)
```
// @file atomics.cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <vector>

static std::atomic<int> shared_value= 0;

void shared_value_increment(){
    shared_value+=1;
}

int main(){

    std::vector<std::thread> threads;
    for(int i=0; i < 1000; i++){
        threads.push_back(std::thread(shared_value_increment));
    }

    for(int i=0; i < 1000; i++){
        threads[i].join(); 
    }

    std::cout << "Shared value:" << shared_value << std::endl;
    return 0;
}
```





