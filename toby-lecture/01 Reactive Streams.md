# 01 Reactive Streams

Iterable 인터페이스는 데이터에 대해서 땡겨오는 `pull` 방식으로 진행된다.           
예를 들어, `next()` 라는 메서드를 통해서 다음에 올 데이터를 가져오는 작업을 수행한다.      
     
Iterable 의 상대성은 Observable 이다.            
Observable 은 `push()` 방식으로 진행된다.         
즉, 내가 땡겨오는게 아닌 타인이 나에게 보내주는 작업을 수행한다.       
  
**Observable**  
* JDK9 부터 Depreacated 된 인터페이스다.      
* 옵저버 객체를 등록하면, 해당 객체에게 이벤트/데이터를 전달하는 일종의 EventSource 역할을 한다.       
* 즉, 특정 이벤트가 발생하면 등록된 옵저버 객체에게 이를 알리는 역할이라고 봐도 된다.      
* 옵저버블이 만들어내는 이벤트/데이터를 받고 싶은 옵저버 객체를 등록하는 역할이라고도 봐도 된다.     

```java
@SuppressWarnings("deprecation")
public class Ob {

    static class IntObservable extends Observable implements Runnable {

        @Override
        public void run() {
            for (int i=1; i <= 10; i++) {
                setChanged();
                notifyObservers(i);
            }
        }
    }

    public static void main(String[] args) {
        Observer ob = (o, arg) -> System.out.println(arg);

        IntObservable io = new IntObservable();
        io.addObserver(ob);

        io.run();
    }
}
```
`notifyObservers(i)`가 실행되면서, 옵저버 객체에서 해당 데이터를 수신하여 출력하고 있다.      
즉, 앞서 언급했던대로 이벤트를 구독하고 이벤트가 발생하면 해당 이벤트(데이터)를 받는 구조로 동작하고 있다.    

* `int i = int.next()` : 리턴 값을 통해서 값을 받아온다.     
* `notifyObservers(i)` : 파라미터를 통해서 값을 전달한다.   

이 두 방법은 똑같은 기능을 수행하지만 방향이 완전히 반대인 것을 알 수 있다.(상대성)      
pull 방식은 미리 정해진 데이터에 대해서 값을 가져오는데 편하다.      
반면에, push 방식은 정해진 것이 없지만 **구독해서 반응**하므로 훨씬 다이나믹하다는 특징을 가지고 있다.      
 
* Observer 구현체는 여러 요소에 이벤트를 등록할 수 있더.      
* Observable 구현체는 여러 Observer 구현체들에게 이벤트(데이터)를 전달할 수 있다.  
* 별도의 스레드에서 비동기적으로 동작시킬 수 있다.
    * 다른 스레드에서 동작하는 여러 옵저버들이 결과 정보를 한번에 받아올 수 있다.   

```java
@SuppressWarnings("deprecation")
public class Ob {

    static class IntObservable extends Observable implements Runnable {

        @Override
        public void run() {
            for (int i = 1; i <= 10; i++) {
                setChanged();
                notifyObservers(i);
            }
        }
    }

    public static void main(String[] args) {
        Observer ob = (o, arg) -> System.out.println(Thread.currentThread().getName() + " " + arg);

        IntObservable io = new IntObservable();
        io.addObserver(ob);

        ExecutorService es = Executors.newSingleThreadExecutor();
        es.execute(io);

        System.out.println(Thread.currentThread().getName() + " EXIT");

        es.shutdown();
    }
}
```
별개의 쓰레드에서 `IntObservable`가 run을 실행했고, `Observer`는 해당 별개 쓰레드에서 출력을 실행했다.             
즉, 기존 쓰레드가 아닌 별개의 쓰레드에서 이벤트 발생과 처리를 한번에 수행할 수 있게끔 할 수 있다.(별개 쓰레드를 손쉽게 만들 수 있다.)        
이 같은 방법은 push 방식으로 만들었다면 엄청 복잡했을 것이다.      
  
이렇듯 Observer 패턴은 많은 장점이 있지만,     
그럼에도 Observer 패턴은 부족하다고 생각했고 더 좋게 만들 수 있도록 2가지를 제안했다.     
    
1. Complete : `다 보냄` 이라는 개념이 없다.        
2. Error : 예외가 전파되는 방식, 받은 예외를 어떻게 처리해야하는가, 재시도나 폴백 옵션들에 대한 고민   

이러한 2가지 문제를 확장해 새로운 라이브러리를 만들엇고     
기존 Observable은 Deprecated가 된 것이다.     

# 리액티브 스트림스 
