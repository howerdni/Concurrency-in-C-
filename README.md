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
 






