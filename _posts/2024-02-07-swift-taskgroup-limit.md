---
layout: post
title: Swift TaskGroup限制并发数量
subtitle:
categories: iOS
tags: [Swift, Swift]
---

## Swift TaskGroup 限制并发数量

```swift
import Foundation


actor Semaphore {
    private var value: Int
    private var waiters: [CheckedContinuation<Void, Never>] = []

    init(value: Int = 0) {
        self.value = value
    }

    func wait() async {
        value -= 1
        if value >= 0 { return }
        await withCheckedContinuation {
            waiters.append($0)
        }
    }

    func signal(value: Int = 1) {
        assert(value >= 1)
        self.value += value
        for _ in 0..<value {
            if waiters.isEmpty { return }
            waiters.removeFirst().resume()
        }
    }
}

func performTask(_ index: Int) async -> Int64 {
    await semaphore.wait()
    print("[\(index)] Starting task")

    let sleepTime =  2_000_000_000 + Int64(Float.random(in: -500_000_000...500_000_000))
    print("[\(index)] sleep for \(sleepTime) ns")
    try? await Task.sleep(nanoseconds: UInt64(sleepTime))
    print("[\(index)] Finished task \(index)")
    await semaphore.signal()
    return sleepTime
}


let semaphore = Semaphore(value: 2)

let returnValue = try? await withThrowingTaskGroup(
    of: Int64.self
) { group -> [Int64] in
    for i in 0..<5 {

        group.addTask {
            await performTask(i)
        }
    }


    var results = [Int64]()
    for try await value in group {
        results.append(value)
    }
    return results
}

if let returnValue{
    print("All tasks finished with results:")
    for i in 0 ..< returnValue.count {
        print("[\(i)] \(returnValue[i])")
    }
}

```

这里如果用 DispatchSemaphore 会有如下报错 (GCD 和 Task Async-Await 不能混用)

```
Instance method 'wait' is unavailable from asynchronous contexts; Await a Task handle instead; this is an error in Swift 6
```

参考资料 <https://forums.swift.org/t/semaphore-alternatives-for-structured-concurrency/59353>
