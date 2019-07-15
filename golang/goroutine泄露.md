<span id="menu">[goroutine退出机制](#finish)</span>

[goroutine泄露及防止](#leak)

[调试和发现goroutine泄露](#find)

### <span id="finish">[goroutine退出机制](#menu)</span>
#### 当它完成了它的工作
goroutine自己运行结束，main退出
#### 因为不可恢复的错误，它不能继续工作
panic退出
#### 当它被告知需要终止工作
通过channel、context通知退出

### <span id="leak">[goroutine泄露及防止](#menu)</span>

#### 发送到channel的数据没有消费

##### 示例
```
package main

import (
	"fmt"
	"math/rand"
	"runtime"
	"time"
	"os"
)

func main() {
	newRandStream := func() <-chan int {
		randStream := make(chan int)
		go func() {
			defer fmt.Println("newRandStream closure exited.") // <1>
			defer close(randStream)
            // 死循环：不断向channel中放数据，直到阻塞
			for {
				randStream <- rand.Int()
			}
		}()

		return randStream
	}
	randStream := newRandStream()
	fmt.Println("3 random ints:")
    // 只消耗3个数据，然后去做其他的事情，此时生产者阻塞，
	// 若主goroutine不处理生产者goroutine，则就产生了泄露
	for i := 1; i <= 3; i++ {
		fmt.Printf("%d: %d\n", i, <-randStream)
	}
	fmt.Fprintf(os.Stderr, "%d\n", runtime.NumGoroutine())
	time.Sleep(50)
	fmt.Fprintf(os.Stderr, "%d\n", runtime.NumGoroutine())
}

输出：
3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821
2
2
```
生产协程进入死循环，不断产生数据，消费协程，也就是主协程只消费其中的3个值，然后主协程就再也不消费channel中的数据，去做其他的事情了。此时生产协程放了一个数据到channel中，但已经不会有协程消费该数据了，所以，生产协程阻塞。 此时，若没有人再消费channel中的数据，生成协程是被泄露的协程。

##### 解决方法：
总的来说，要解决channel引起的goroutine leak问题，主要是看在channel阻塞goroutine时，该goroutine的阻塞是正常的，还是可能导致协程永远没有机会执行。若可能导致协程永远没有机会执行，则可能会导致协程泄露。 所以，在创建协程时就要考虑到它该如何终止。
解决一般问题的办法就是，当主线程结束时，告知生产线程，生产线程得到通知后，进行清理工作：或退出，或做一些清理环境的工作。
```
package main

import (
	"fmt"
	"math/rand"
	"os"
	"runtime"
	"time"
)

func main() {
	newRandStream := func(done <-chan interface{}) <-chan int {
		randStream := make(chan int)
		go func() {
			defer fmt.Println("newRandStream closure exited.")
			defer close(randStream)
			for {
				select {
				case randStream <- rand.Int():
				case <-done:
					return
				}
			}
		}()

		return randStream
	}
	done := make(chan interface{})
	randStream := newRandStream(done)
	fmt.Println("3 random ints:")
	for i := 1; i <= 3; i++ {
		fmt.Printf("%d: %d\n", i, <-randStream)
	}
	close(done)

	time.Sleep(1*time.Second)
	fmt.Fprintf(os.Stderr, "%d\n", runtime.NumGoroutine())
}

输出：
3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821
newRandStream closure exited.
1
```

#### 接受channel的数据没有生产者
```
示例：
package main

import (
	"fmt"
)

func main() {
	doWork := func(strings <-chan string) <-chan interface{} {
		completed := make(chan interface{})
		go func() {
			defer fmt.Println("doWork exited.")
			defer close(completed)
			for s := range strings {
				// Do something interesting
				fmt.Println(s)
			}
		}()
		return completed
	}

	doWork(nil)
	// Perhaps more work is done here
	fmt.Println("Done.")
}
输出：
Done.
```
我们看到doWork被传递了一个nil通道。所以strings通道永远无法读取到其承载的内容，而且包含doWork的goroutine将在这个过程的整个生命周期中保留在内存中
```
解决
package main

import (
	"fmt"
	"time"
)

func main() {
	doWork := func(
		done <-chan interface{},
		strings <-chan string,
	) <-chan interface{} { // <1>
		terminated := make(chan interface{})
		go func() {
			defer fmt.Println("doWork exited.")
			defer close(terminated)
			for {
				select {
				case s := <-strings:
					// Do something interesting
					fmt.Println(s)
				case <-done: // <2>
					return
				}
			}
		}()
		return terminated
	}

	done := make(chan interface{})
	terminated := doWork(done, nil)

	go func() { // <3>
		// Cancel the operation after 1 second.
		time.Sleep(1 * time.Second)
		fmt.Println("Canceling doWork goroutine...")
		close(done)
	}()

	<-terminated // <4>
	fmt.Println("Done.")
}
输出：
Canceling doWork goroutine...
doWork exited.
Done.
```
尽管向doWork传递了nil给strings通道，我们的goroutine依然正常运行至结束。与之前的例子不同，本例中我们把两个goroutine连接在一起之前，我们建立了第三个goroutine以取消doWork中的goroutine，并成功消除了泄漏问题。

### <span id="find">[调试和发现goroutine泄露](#menu)</span>

#### runtime

可以通过runtime.NumGoroutine()函数来获取后台服务的协程数量。通过查看每次的协程数量的变化和增减，我们可以判断是否有goroutine泄露发生。
```
fmt.Fprintf(os.Stderr, "%d\n", runtime.NumGoroutine())
time.Sleep(10e9) //等一会，查看协程数量的变化
fmt.Fprintf(os.Stderr, "%d\n", runtime.NumGoroutine())
```

#### pprof

```
import (
  "runtime/debug"
  "runtime/pprof"
)

func getStackTraceHandler(w http.ResponseWriter, r *http.Request) {
    stack := debug.Stack()
    w.Write(stack)
    pprof.Lookup("goroutine").WriteTo(w, 2)
}
func main() {
    http.HandleFunc("/_stack", getStackTraceHandler)
}
```
参看： [实战Go内存泄露](https://www.cnblogs.com/sunsky303/p/11077030.html)
