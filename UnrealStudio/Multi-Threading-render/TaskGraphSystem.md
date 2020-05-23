﻿# Multi-Threading: Task Graph System

2019年2月27日

9:56

 

[Template:Rating](https://wiki.unrealengine.com/index.php?title=Template:Rating&action=edit&redlink=1)

**Contents**

 [hide] 

·         [1Overview](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Overview)

·         [1.1My Example: Calculate first 50,000 Prime Numbers](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#My_Example:_Calculate_first_50.2C000_Prime_Numbers)

·         [2Video](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Video)

·         [3Comparing Results](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Comparing_Results)

·         [4Entire .CPP Source Code For You!](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Entire_.CPP_Source_Code_For_You.21)

·         [5ENamedThreads::GameThread?](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#ENamedThreads::GameThread.3F)

·         [6Why a Name Space?](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Why_a_Name_Space.3F)

·         [7Knowing When All Tasks Are Complete](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Knowing_When_All_Tasks_Are_Complete)

·         [7.1Completion Events Are Filled All at Once](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Completion_Events_Are_Filled_All_at_Once)

·         [8Things to Note](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Things_to_Note)

·         [9AsyncWork Task Templates](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#AsyncWork_Task_Templates)

·         [10What About Platforms Without Multithreading?](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#What_About_Platforms_Without_Multithreading.3F)

·         [11Further Reading](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Further_Reading)

·         [11.1ParallelFor](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#ParallelFor)

·         [12Conclusion](https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System#Conclusion)

Overview

*Author:* [Rama](https://wiki.unrealengine.com/index.php?title=User:Rama) ([talk](https://wiki.unrealengine.com/index.php?title=User_talk:Rama))

Dear Community,

The UE4 task graph system allows you to perform many actions, each of which is relatively small, on separate threads from the game thread.

Please note that a task graph **will sometimes use the game thread,** and therefore for large tasks you should use a FRunnable as demonstrated in [my other multi threading wiki.](https://wiki.unrealengine.com/Multi-Threading:_How_to_Create_Threads_in_UE4)

**AsyncWork.h**

\* Tells the user job to do the work, sometimes called synchronously, sometimes from the thread pool. Calls the event tracker.

 

**My Example: Calculate first 50,000 Prime Numbers**

In the video below I am calculating the first 50,000 prime numbers while the rest of the game thread continues to run!

To put this in perspective, if I did what I do in the video without the UE4 task graph system, the game would be hanging while all those numbers get computed, for the total duration of the video :)

Video

<youtube><https://www.youtube.com/watch?v=cgELOodtoSU&feature=youtu.be></youtube>

Comparing Results

I show my results at the end of the video.

You can compare them with this webpage!

My multi-threading system did actually calculate the first 50,000 (+1) prime numbers!

[**|First 50,000 Prime Numbers**](http://www.cs.arizona.edu/icon/oddsends/primes.htm)

Entire .CPP Source Code For You!

I am doing all of this as a test in my player controller class :)

I did just about everything in the .cpp file alone using the Namespace.

Please note that the timer that checks when the threads are done is also in the .h, as is the function start the whole process. (see toward the end of the code)

*//Multi thread Test, finding prime number*
 **namespace** VictoryMultiThreadTest
 {
         *//Multi-threaded link to UObjects, do not create,modify,destroy UObjects / AActors via this link!*
         AVictoryGamePlayerController* ThePC;
         
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*
         *//   OUTPUT RESULTS OF TASK THREADS*
         TArray<uint32> PrimeNumbers;
         
         *// This is the array of thread completions to know if all threads are done yet*
         FGraphEventArray                VictoryMultithreadTest_CompletionEvents;
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*
         *//~~~~~~~~~~~~~~~~~~~~~~~~~~*

*//~~~~~~~~~~~~~~~*
         *//Are All Tasks Complete?*
         *//~~~~~~~~~~~~~~~*
         bool TasksAreComplete()
         {
                 *//Check all thread completion events*
                 **for** (int32 Index = 0; Index < VictoryMultithreadTest_CompletionEvents.Num(); Index++)
                 {
                         *//If  ! IsComplete()*
                         **if** (!VictoryMultithreadTest_CompletionEvents[Index]->IsComplete())
                         {
                                 **return** false;
                         }
                 }
                 **return** true;
         }
         *//~~~~~~~~~~~*
         *//Actual Task Code*
         *//~~~~~~~~~~~*
         int32 FindNextPrimeNumber()
         {
                 *//Last known prime number  + 1*
                 int32 TestPrime = PrimeNumbers.Last();
                 
                 bool NumIsPrime = false;
                 **while**( ! NumIsPrime)
                 {
                         NumIsPrime = true;
                         
                         *//Try Next Number*
                         TestPrime++;
                         
                         *//Modulus from 2 to current number - 1* 
                         **for**(int32 b = 2; b < TestPrime; b++)
                         {
                                 **if**(TestPrime % b == 0) 
                                 {
                                         NumIsPrime = false;
                                         **break**;
                                         *//~~~*
                                 }
                         }
                 }
                 
                 *//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*
                 *//Did another thread find this number already?*
                 **if**(PrimeNumbers.Contains(TestPrime))
                 {
                         **return** FindNextPrimeNumber(); *//recursion*
                 }
                 *//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*
                 
                 *//Success!*
                 **return** TestPrime;
         }
         

*//~~~~~~~~~~~*
         *//Each Task Thread*
         *//~~~~~~~~~~~*
         **class** **FVictoryTestTask**
         {
         
           **public**:
                 FVictoryTestTask() *//send in property defaults here*
                 {
                         *//can add properties here*
                 }

*/** return the name of the task **/*
                 **static const** TCHAR* GetTaskName()
                 {
                         **return** TEXT("FVictoryTestTask");
                 }
                 FORCEINLINE **static** TStatId GetStatId()
                 {
                         RETURN_QUICK_DECLARE_CYCLE_STAT(FVictoryTestTask, STATGROUP_TaskGraphTasks);
                 }
                 */** return the thread for this task **/*
                 **static** ENamedThreads::Type GetDesiredThread()
                 {
                         **return** ENamedThreads::AnyThread;
                 }
                 
                 
                 */**
                 *namespace ESubsequentsMode*
                 *{*
                         *enum Type*
                         *{*
                                 *//Necessary when another task will depend on this task.* 
                                 *TrackSubsequents,*
                                 *//Can be used to save task graph overhead when firing off a task that will not be a dependency of other tasks.* 
                                 *FireAndForget*
                         *};*
                 *}*
                 **/*
                 **static** ESubsequentsMode::Type GetSubsequentsMode() 
                 { 
                         **return** ESubsequentsMode::TrackSubsequents; 
                 }
                 
                 *//~~~~~~~~~~~~~~~~~~~~~~~~*
                 *//Main Function: Do Task!!*
                 *//~~~~~~~~~~~~~~~~~~~~~~~~*
                 void DoTask(ENamedThreads::Type CurrentThread, **const** FGraphEventRef& MyCompletionGraphEvent)
                 {
                         *//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~*
                         
                         PrimeNumbers.Add(FindNextPrimeNumber());
                         
                         *//****************************************
                         *//Show Incremental Results in Main Game Thread!*
                         
                         *//        Please note you should not create, destroy, or modify UObjects here.*
                         *//          Do those sort of things after all thread are completed.*
                         
                         *//          All calcs for making stuff can be done in the threads*
                         *//             But the actual making/modifying of the UObjects should be done in main game thread,*
                         *//                 which is AFTER all tasks have completed :)*
                         
                         ThePC->ClientMessage(FString("A thread completed! ~ ") + FString::FromInt(PrimeNumbers.Last()));
                         *//****************************************
                 }
         };
         
         *//~~~~~~~~~~~~~~~~~~~*
         *//  Multi-Task Initiation Point* 
         *//~~~~~~~~~~~~~~~~~~~*
         void FindPrimes(**const** uint32 TotalToFind)
         {
                 PrimeNumbers.Empty();
                 PrimeNumbers.Add(2);
                 PrimeNumbers.Add(3);
                                 
                 *//~~~~~~~~~~~~~~~~~~~~*
                 *//Add thread / task for each of total prime numbers to find*
                 *//~~~~~~~~~~~~~~~~~~~~*
                 
                 **for**(uint32 b = 0; b < TotalToFind; b++)
                 {
                         VictoryMultithreadTest_CompletionEvents.Add(TGraphTask<FVictoryTestTask>::CreateTask(NULL, ENamedThreads::GameThread).ConstructAndDispatchWhenReady()); *//add properties inside ConstructAndDispatchWhenReady()*
                 }
         }
         
 }

*//~~~ In the Game Thread ~~~*

*//timer to check when threads are done*
 *//Please note timers must be in the game thread / main / normal thread*
 void AVictoryGamePlayerController::VictoryCheckAllThreadsDone()
 {
         
         **if**(VictoryMultiThreadTest::TasksAreComplete() )
         {
                 *//Clear Timer*
                 GetWorldTimerManager().ClearTimer(**this**, 
                         &AVictoryGamePlayerController::VictoryCheckAllThreadsDone);
                         
                 ClientMessage("Multi Thread Test Done!");
                 
                 VShow("Prime Numbers Found:");
                 **for**(int32 v = 0; v < VictoryMultiThreadTest::PrimeNumbers.Num(); v++)
                 {
                         VShow(FString::FromInt(v) + FString("~ "), VictoryMultiThreadTest::PrimeNumbers[v]);
                 }
         }
 }

*//Starting the Tasks / Threads*
 void AVictoryGamePlayerController::StartThreadTest()
 {

VictoryMultiThreadTest::ThePC = **this**;
         VictoryMultiThreadTest::FindPrimes(50000); *//first 50,000 prime numbers*
                         
         *//Start a timer to check when all the threads are done!*
         GetWorldTimerManager().SetTimer(**this**, 
                 &AVictoryGamePlayerController::VictoryCheckAllThreadsDone, 1, true);        
 }

 

ENamedThreads::GameThread?

Please note in this line:

VictoryMultithreadTest_CompletionEvents.Add(
   TGraphTask<FVictoryTestTask>::CreateTask(NULL, ENamedThreads::GameThread).ConstructAndDispatchWhenReady()
 );

GameThread simply signifies where the task is being initiated from, not where it is allowed to run.

Unless you have threads starting other threads you will want to always specify GameThread :)

Why a Name Space?

*//Multi thread Test, finding prime number*
 **namespace** VictoryMultiThreadTest
 {

It's just a convenient way to organize the required info for multi-threading :)

Knowing When All Tasks Are Complete

I use a looping 1 second timer that runs this function:

*//Array of all task threads, instantly and entirely filled when FindPrimes is first called*
 FGraphEventArray                VictoryMultithreadTest_CompletionEvents;

*//~~~~~~~~~~~~~~~*
 *//Are All Tasks Complete?*
 *//~~~~~~~~~~~~~~~*
 bool TasksAreComplete()
 {
         *//Check all thread completion events*
         **for** (int32 Index = 0; Index < VictoryMultithreadTest_CompletionEvents.Num(); Index++)
         {
                 *//If  ! IsComplete()*
                 **if** (!VictoryMultithreadTest_CompletionEvents[Index]->IsComplete())
                 {
                         **return** false;
                 }
         }
         **return** true;
 }

the critical line is this

**if** (!VictoryMultithreadTest_CompletionEvents[Index]->IsComplete())

See TaskGraphInterfaces.h for more info!

**Completion Events Are Filled All at Once**

Please note that ALL completion events are created the instant the multi-threading process is initiated!

This part is NOT multi-threaded :)

*//~~~~~~~~~~~~~~~~~~~*
 *//  Multi-Task Initiation Point* 
 *//~~~~~~~~~~~~~~~~~~~*
 void FindPrimes(**const** uint32 TotalToFind)
 {
         PrimeNumbers.Empty();
         PrimeNumbers.Add(2);
         PrimeNumbers.Add(3);
                         
         *//~~~~~~~~~~~~~~~~~~~~*
         *//Add thread / task for each of total prime numbers to find*
         *//~~~~~~~~~~~~~~~~~~~~*
         
         **for**(uint32 b = 0; b < TotalToFind; b++)
         {
                 VictoryMultithreadTest_CompletionEvents.Add(TGraphTask<FVictoryTestTask>::CreateTask(NULL, ENamedThreads::GameThread).ConstructAndDispatchWhenReady()); *//add properties inside ConstructAndDispatchWhenReady()*
         }
 }

Things to Note

·         Do not create, delete, or modify UObjects inside the DoTask() of each task thread ( unless you are on 4.8 or after -Rama )

·         Do not call timers inside of DoTask()

·         Dont try to draw debug lines/points etc, it will likely crash (true as of 4.6.1)

·         If you follow the above rules you can send incremental progress to yourself on the main thread as I am doing.

I am actually calling a function in my HUD class from my player controller pointer, from DoTask() !

AsyncWork Task Templates

In AsyncWork.h you will find some great templates for tasks, such as FAsyncTask and FAutoDeleteAsyncTask

I would post the code here but I can't due to the fact that this a public wiki.

Have fun exploring AsyncWork.h!

What About Platforms Without Multithreading?

Note that the task graph system accounts for whether the destination platform does not support multithreading, which at the moment is only HTML5 as far as I know. iOS and Android have multi-threading support.

Inside of FTaskGraphImplementation::QueueTask

**if** (FPlatformProcess::SupportsMultithreading())
 {
 *//...*
 }
 **else**
 {
   ThreadToExecuteOn = ENamedThreads::GameThread;
 }

Further Reading

Check out

 Core/Private/Async/TaskGraph.cpp 

to see how the tasks are created, and how whether the platform supports multi threading is taken into account!

**ParallelFor**

This is a great article, but if you have a "really simple function" that you just want to run split-up between threads, take a look at ParallelFor (Runtime/Core/Public/Async/ParallelFor.h). ParallelFor uses the Task Graph System underneath the covers. Use ParallelFor if you NEED results for a bunch of calculations this frame, and you can't use results that are a couple of frames old.

int32 MaxEntries = 500;
 ParallelFor( MaxEntries, [](int32 CurrIdx)
 {
         *// Your implementation may fetch/store results based on the CurrIdx, but*
         *// for simplicity we just have some dummy "expensive" calculation here.*
         int32 Sum = 0;
         **for** ( int32 Idx = 0; Idx < 1000 * 10; ++Idx )
         {
                 Sum += FMath::Sqrt( 1234.56f );
         }
 } );

For example, the above calculation takes about 30ms (on my PC) to run with a single thread, but just using ParallelFor will split the calculation between threads, making it complete in about 10ms instead.

Conclusion

Enjoy!

 

来自 <<https://wiki.unrealengine.com/Multi-Threading:_Task_Graph_System>> 

 

 

 

 

2019年2月27日

9:58
