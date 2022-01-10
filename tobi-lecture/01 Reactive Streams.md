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


  

