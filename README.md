# OS_Final_2023Spring

## Q1.a: What is the content of the matrix Need?
### Need Matrix
    A B C D
    0 0 0 0
    0 7 5 0
    1 0 0 2
    0 0 2 0
    0 6 4 2

## Q1.b: Is the system in a safe state?
Yes, It's in safe state.
One possible work sequence is P0 -> P2 -> P1 -> P3 -> P4

## Q2: Create a nodejs code which implements a get method where you send "Hello {Name}" as the body and the response body is "Welcome {Name}"?
### 
```
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());

app.get('', (req, res) => {
  const name = req.query.name;
  console.log(name);
  
  if (name) {
    res.send("Welcome " + name + "\n");
  } else {
    res.end("Hello World \n");
  }
});

const port = 3000;
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
// http://localhost:3000/?name=YimingZhang
```

## Q3: Provide two programming examples in which multithreading provides better performance than a single-threaded solution.
1. Web server: A web server can handle multiple requests from clients simultaneously. Using a multithreaded approach, the server can spawn a new thread for each incoming request and handle them independently. This allows the server to handle more requests simultaneously and improves the response time for each client.

2. Matrix multiplication: Matrix multiplication is a computationally intensive task that can be performed faster using multithreading. By breaking the matrix into smaller sub-matrices and assigning each sub-matrix to a different thread, the computation can be performed in parallel, reducing the time required for the operation. This approach is particularly effective for large matrices, where the performance gain from multithreading can be significant.


## Q4: Provide two examples where multithreading does not provide better performance than single threaded. 

1. Simple computations: In cases where the computation is simple and does not require a lot of resources, multithreading may not provide significant performance gains. In fact, the overhead of creating and managing multiple threads can actually slow down the process.

2. IO-bound tasks: When a task is primarily I/O bound, meaning it involves mostly waiting for data to be read or written to a file or network, multithreading may not provide significant performance gains. This is because the waiting time is not reduced by using multiple threads, and the overhead of creating and managing multiple threads can actually slow down the process.


## Q5: Describe the differences among short-term, medium-term, and long- term scheduling.

Short-term scheduling determines which process will run next in the CPU from the pool of ready processes. It's responsible for managing the CPU and making decisions in a very short time, typically in milliseconds.

Medium-term scheduling controls the degree of multiprogramming by removing processes from memory and transferring them to secondary storage when memory is low. It can be used to optimize system performance by swapping out memory-hungry processes that are not actively being used.

Long-term scheduling determines which processes will be admitted into the system for execution. It can be used to manage the number of active processes in the system to ensure efficient use of resources and to prevent overloading the system. It's responsible for deciding which processes will be loaded into memory and when.

## Q6: What is the average turnaround time for these processes with the FCFS scheduling algorithm?

### The turnaround time:
P1: 8 - 0 = 8,  
P2: 12 - 0.4 = 11.6,  
P3: 13 - 1 = 12. 
### The average turnaround time: 
(8 + 11.6 + 12) / 3 = 10.2 


## Q7：Bank System

```
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

class BankAccount {
  constructor() {
    this.balance = 0;
  }

  deposit(amount) {
    this.balance += amount;
  }

  withdraw(amount) {
    if (this.balance >= amount) {
      this.balance -= amount;
      return true;
    } else {
      return false;
    }
  }

  getBalance() {
    return this.balance;
  }
}

if (isMainThread) {
  // Create a new BankAccount instance
  const account = new BankAccount();

  // Start 4 threads to add money to the account
  for (let i = 0; i < 4; i++) {
    new Worker(__filename, {
      workerData: { type: 'add', amount: 100 }
    });
  }

  // Start 5 threads to take money out of the account
  for (let i = 0; i < 5; i++) {
    new Worker(__filename, {
      workerData: { type: 'subtract', amount: 50 }
    });
  }

  // Listen for messages from the worker threads
  let numWorkersDone = 0;
  const handleWorkerMessage = (msg) => {
    if (msg.type === 'done') {
      numWorkersDone++;
      if (numWorkersDone === 9) {
        console.log(`Final account balance: ${account.getBalance()}`);
        process.exit(0);
      }
    }
  };
  for (const worker of require('worker_threads').workerData) {
    worker.on('message', handleWorkerMessage);
  }
} else {
  // Get the type and amount of the operation to perform from the workerData
  const { type, amount } = workerData;

  // Perform the operation and send a message to the main thread when done
  const account = new BankAccount();
  if (type === 'add') {
    account.deposit(amount);
  } else if (type === 'subtract') {
    while (!account.withdraw(amount)) {
      // Wait until there's enough money in the account to withdraw
    }
  }
  parentPort.postMessage({ type: 'done' });
}

```
