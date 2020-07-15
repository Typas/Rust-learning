# Notes

這章講並行處理。

## 並行處理常見問題

- 資料競爭：執行緒間存取資料或資源的順序不一致。
- 死鎖：兩執行緒互相等待，形成無窮迴圈。
- 只在特定情況出現的bug，難以重現並難以修復。

## 創建新執行緒

以`thread::spawn`來建立一個新執行緒，以`join`來控制執行緒的順序性，如下所示：

``` rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

而若要在內使用到變數，最簡易的方法是以`move` closure達成，如下所示：

``` rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## 在執行緒間傳遞資料

Go語言有句標語：「不要用共享記憶體來溝通，要以溝通來共享記憶體。」
而rust中以`channel`或`sync_channel`來進行溝通，不過其前置的縮寫不好記。
使用例如下：

``` rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

其中的`recv()`會暫停主執行緒的執行到值傳下來為止。若想不暫停，可以改用`try_recv()`，會立即回傳，若無值可立即回傳則會回傳錯誤。其中的`tx`可以用`clone()`達成*multiple producer, single consumer*也就是`mpsc`的作用，而`rx`則不可複製。

### 所有權的轉移

請注意，`tx.send()`會把所有權從`tx`中轉移到`rx`。

## 共享狀態的並行

除了上面的溝通法外，rust也支援共享記憶體的方法，以`Mutex<T>`與`Arc<T>`來進行操作。
如下所示：

``` rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });

        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

其中的`Arc::clone(&counter)`可以`counter.clone()`替代。
`Arc<T>`對應到`Rc<T>`，而`Mutex<T>`則對應到`RefCell<T>`。
`Arc<T>`的潛在風險與`Rc<T>`相同，有循環參考的問題；`Mutex<T>`與其他語言中的mutex一樣，有造成死鎖的可能性，`MutexGuard`能提供一些資訊。

## 並行所需的Trait

通常使用`#derive[(Send, Sync)]`就會自動實作，手動實作則需要用到`unsafe`。

### `Send`

這個trait允許型態的所有權在執行緒之間轉移。

### `Sync`

這個trait允許多執行緒的存取。
