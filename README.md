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



