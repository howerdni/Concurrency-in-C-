# Concurrency in C++
## Table of contents
=================
- [When to use thread](#1)
- [Thread Libraries in C](#2)
- [C++ thread liabrary](#3)
- [First C++ thread example](#4)
- [std::thread with a lambda in modern c++](#5)
- [Launching multiple std::thread in C++](#6)
- [jthread std::jthread in C++ 20](#7)
- [std::mutex and preventing data races in C++](#8)
- [Preventing deadlock with std::lock_guard in modern C++](#9)
- [Using std::atomic in modern C++ to update a shared valueUsing std::atomic in modern C++ to update a shared value](#10)
- [Example Data Parallel C++ Program using multiple threads in SFML](#11)
- [Condition Variable in Modern cpp and unique lock](#12)
- [std::async in cpp with background thread loading data example](#13)

## 1 
## When to use thread
<ol>
  <li>Heavy Computaions</li>
  <li>Using threads to seperate work</li>
</ol>

## 2
## Thread Libraries in C
- Before C++11/14/17/20, there existed threading libraries with different semantics
- Libraries like "Boost", Intel "Thread Building Blocks", or "pthread" were used 
  - Perhaps you have used pthread at least in C
  - (std::thread I believe is implemented with pthread most posix systems)
- Typically today I would personally recommend using the standard C++ threading library for portability reasons as the default choice.
## 3
## C++ thread liabrary
[Concurrency support library](https://en.cppreference.com/w/cpp/thread)
## 4
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
## 5
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
## 6
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
 ## 7
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
## 8
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
## 9
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
## 10
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
## 11
## Example Data Parallel C++ Program using multiple threads in SFML

```
// @file sfml_grid.cpp
// g++ -std=c++17 sfml_grid.cpp -o prog -lpthread -lsfml-graphics -lsfml-window -lsfml-system
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>
#include <algorithm> // for fill
#include <memory>
#include <chrono>  // For sleep

// Third party libraries
#include <SFML/Graphics.hpp>

// Globally available grid
static std::vector<int> grid;
// Global array of all of our objects
std::vector<std::unique_ptr<sf::Shape>> shapes;
// Keeps track of the program running
bool isRunning = true;

// Function to update grid
// This will be handled in each thread, independently working
// on the task.
void update_grid(int x, int y){
	while(isRunning){
		std::this_thread::sleep_for(std::chrono::milliseconds(1000));
		grid[y*2+x] = rand()%4;
	}
}


// Entry point into our program
int main(){
    // Initialize our grid with 4 entries
    grid.reserve(4);
    std::fill(begin(grid),end(grid),0);
	// Pick out the shape being drawn
	// in our grid and update it accordingly.
    for(int x=0; x < 2; x++){
        for(int y=0; y<2; y++){
			shapes.push_back(std::make_unique<sf::CircleShape>(100.0f));	
		}
	}

    // Launch threads
    std::vector<std::thread> threads;
    for(int x=0; x < 2; x++){
        for(int y=0; y<2; y++){
            threads.push_back(std::thread(update_grid,x,y));	
        }
    }

    // Main program loop
	sf::RenderWindow window(sf::VideoMode(400, 400), "SFML with C++ threads");

	// Main Game loop
	while (window.isOpen() && isRunning)
	{
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::Closed){
				window.close();
				isRunning = false;
			}
		}
		
		// Clear the window
		window.clear();
		for(int x=0; x < 2; x++){
			for(int y=0; y<2; y++){
				// Set the position
				shapes[y*2+x]->setPosition(x*200,y*200);
				// Update the color
				if(0==grid[y*2+x]){
					shapes[y*2+x]->setFillColor(sf::Color::Red);
				}else if(1==grid[y*2+x]){
					shapes[y*2+x]->setFillColor(sf::Color::Green);	
				}else if(2==grid[y*2+x]){
					shapes[y*2+x]->setFillColor(sf::Color::Blue);
				}else if(3==grid[y*2+x]){
					shapes[y*2+x]->setFillColor(sf::Color::White);	
				}
			}
		}
	
		// Draw all of our shapes
		for(auto& shape: shapes){
			window.draw(*shape);
		}
		window.display();
	}

    // Join threads before program execution termintes
    for(auto& th: threads){
        th.join(); 
    }

    // Program finish
    std::cout << "Program Terminating" << std::endl;

    return 0;
}
```
## 12
## Condition Variable in Modern cpp and unique lock
- Condition Variable: to avoid thread spinning and constantly checking to see if there is work available
- what we need to use Condition Variable
1. boolean  
2. lock/unique_lock  
3. condition_variable  
4. 2 threads worker/reporter  
```
// @file cv.cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>
#include <condition_variable>

std::mutex gLock;
std::condition_variable gConditionVariable;

int main(){

    int result = 0;
    bool notified = false;

    // Reporting thread
    // Must wait on work, done by the working thread
    std::thread reporter([&]{
        std::unique_lock<std::mutex> lock(gLock);
        if(!notified){
            gConditionVariable.wait(lock);
        }
        std::cout << "Reporter, result is: " << result << std::endl;
    });

    // Working thread
    std::thread worker([&]{
        std::unique_lock<std::mutex> lock(gLock);
        // Do our work, because we have the lock
        result = 42 + 1 + 7;
        // Our work is done
        notified = true;
        std::this_thread::sleep_for(std::chrono::seconds(5));
        std::cout << "Work complete\n";
        // Wake up a thread, that is waiting, for some
        // condition to be true
        gConditionVariable.notify_one();
    });

    reporter.join();
    worker.join();

    std::cout << "Program complete" << std::endl;
    return 0;
}
```
- variable notified is set because even the thread reporter is executed first, the code:
```if(!notified){gConditionVariable.wait(lock);}``` will not allow the thread reporter execute before gConditionVariable notify.  
- [std::condition_variable::wait](https://en.cppreference.com/w/cpp/thread/condition_variable/wait)
- tips：unique_lock符合RAII，当超过作用域时，会进行析构（对持有的mutex进行解锁)
## 13
## std::async in cpp with background thread loading data example

```
// @file async_buffer.cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

bool bufferedFileLoader(){
    size_t bytesLoaded = 0;
    while(bytesLoaded < 20000){
        std::cout << "thread: loading file..." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(250));
        bytesLoaded += 1000;
    }
    return true;
}

int main(){

    std::future<bool> backgroundThread = std::async(std::launch::async,
                                                    bufferedFileLoader);

    std::future_status status;
    // Our main program loop
    while(true){
        std::cout << "Main thread is running" << std::endl;
        // artificial sleep for our program
        std::this_thread::sleep_for(std::chrono::milliseconds(50));
        status = backgroundThread.wait_for(std::chrono::milliseconds(1));
        // If our data is ready, that is, our background
        // thread has completed
        if(status == std::future_status::ready){
            std::cout << "Our data is ready..." << std::endl;
            break;
        }

    }

    std::cout << "Program is complete" << std::endl;

    return 0;
}
```
- the std::future_status and std::future<bool>.wait_for cooperate vary well.
- [std::future_status](https://en.cppreference.com/w/cpp/thread/future_status)
- [wait_for()](https://en.cppreference.com/w/cpp/thread/future/wait_for)  
  - return a std::future_status
- [get()](https://en.cppreference.com/w/cpp/thread/future/get)  
  - return the thread result
- [std::launch::async](https://en.cppreference.com/w/cpp/thread/launch)
	
	
