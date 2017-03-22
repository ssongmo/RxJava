## RxJava 입문
- **Reactive X** :  Observer 패턴과, Iterator 패턴, 함수형 프로그래밍의 특성을 이용하여 비동기 구성과 관측 가능한 데이터 스트림 또는 이벤트 스트림을 다루는데 초점을 맞춘 API이다. 실시간 데이터를 다루기 위해 효율적이고, 명확하며, 확장가능한 접근법을 가지는 것은 중요하다. 실시간 데이터를 다루기 위해 Observables와 operator를 사용하여, ReactiveX는 쓰레드 생성과 동시성 문제와 같은 비동기 프래그래밍의 일반적인 관심사를 단순화 하면서 데이터 스트림의 생성과 작용을 위한 구성 가능하고 유연한 API를 제공한다.


 - **구성 Class** : 두개의 메인 클래스는 Observable과 Subscriber이다. RxJava에서 Observable는 데이터 스트림이나 이벤트 스트림을 보내는 클래스이며, Subscriber는 받은 아이템에 따라 행동하는 클래스이다.



1. ***실제 Task를 처리하는 객체 ( 발행자 )***

```
  Observable<String> simpleObservable =
      Observable.create(new Observable.OnSubscribe<String>() {
          @Override
          public void call(Subscriber<? super String> subscriber) {

              //네트웤을 통해서 데이터를 긁어온다
              // 반복문을 돌면서 ==============
              //for(네트웤에서 가져온 데이터) { json
              subscriber.onNext("Hello RxAndroid !!");
              subscriber.onNext("Hello RxAndroid !! 1");
              subscriber.onNext("Hello RxAndroid !! 2");
              subscriber.onNext("Hello RxAndroid !! 3");
              //}
              //=============================
              subscriber.onCompleted();
          }
      });
 ```
2. ***Subscriber( 구독자 ).** Subscriber안에 구성된 메소드는 다음과 같다.*

```
  Subscriber integerSubscriber = new Subscriber() {
     @Override
     public void onCompleted() {
         System.out.println("Complete!");
     }

     @Override
     public void onError(Throwable e) {

     }

     @Override
     public void onNext(Integer value) {
         System.out.println("onNext: " + value);
     }
  };
```

* 옵저버를 등록하는 함수인데 진화형이라고 생각하면 된다.

```
// 각 함수를 콜백객체에 나눠서 담아준다
simpleObservable
    .subscribe(new Action1<String>() { // onNext함수와 동일한 역할을 하는 콜백객체
        @Override
        public void call(String s) {  //받는 인자가 1개.
            Toast.makeText(MainActivity.this, "[Observer2]" +s, Toast.LENGTH_SHORT).show();
        }
    }, new Action1<Throwable>() {      // onError함수와 동일한 역할을 하는 콜백객체
        @Override
        public void call(Throwable throwable) {
            Log.e(TAG, "[Observer] error: " + throwable.getMessage());
        }
    }, new Action0() {               //onComplete와 동일한 역할을 하는 콜백 객체.  받는 인자가 0개.
        @Override
        public void call() {
            Log.d(TAG, "[Observer] complete!");
        }
    });

```

* 옵저버를 등록하는 함수 - 최종진화형(람다식)
```
simpleObservable.subscribe(
        (string) -> {Toast.makeText(MainActivity.this,
          "[Observer3]"+ string, Toast.LENGTH_SHORT).show();}

        ,(error) -> {Log.e(TAG, "[Observer2] error: " + error.getMessage());}
        ,() -> {Log.d(TAG, "[Observer2] complete");}
);

```
#### *사용 시 주의사항*

```
Schedulers.computation() - 이벤트 그룹에서 간단한 연산이나 콜백 처리를 위해 사용.  
                           I/O 처리를 여기에서 해서는 안됨.
                          RxComputationThreadPool라는 별도의 스레드 풀에서 최대 cpu갯수
                          만큼 순환하면서 실행.

Schedulers.immediate() - 현재 스레드에서 즉시 수행. observeOn()이 여러번 쓰였을 경우.
                         immediate()를 선언한 바로 윗쪽의 스레드에서 실행.

Schedulers.from(executor) - 특정 executor를 스케쥴러로 사용.

Schedulers.io() - 동기 I/O를 별도로 처리시켜 비동기 효율을 얻기 위한 스케줄러.
                  자체적인 스레드 풀에 의존.
                  일부 오퍼레이터들은 자체적으로 어떤 스케쥴러를 사용할지 지정한다.
                  예를 들어 buffer 오퍼레이터는 Schedulers.computation()에 의존하며
                  repeat은 Schedulers.trampoline()를 사용한다.

Schedulers.newThread() - 새로운 스레드를 만드는 스케쥴러

Schedulers.trampoline() - 큐에 있는 일이 끝나면 이어서 현재 스레드에서 수행하는 스케쥴러.

AndroidSchedulers.mainThread() - 안드로이드의 UI 스레드에서 동작.


HandlerScheduler.from(handler) - 특정 핸들러 handler에 의존하여 동작
```
RxAndroid가 제공하는 AndroidSchedulers.mainThread() 와 RxJava가 제공하는 Schedulers.io()를 조합해서 Schedulers.io()에서 수행한 결과를 AndroidSchedulers.mainThread()에서 받아 UI에 반영하는 형태가 많이 사용되고 있다.




### Rx에서 Thread지정. 이용하기.

***1. 먼저 call매소드에 아래와 같이 반복문을 추가한다.***

```
@Override
   public void call(Subscriber<? super String> subscriber) {

       //네트웤을 통해서 데이터를 긁어온다
       // 반복문을 돌면서 ==============
       //for(네트웤에서 가져온 데이터) { json
       for(int i = 0; i<3; i++) {
           subscriber.onNext("Hello RxAndroid !!" + i);
           try {
               Thread.sleep(2000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }

       subscriber.onCompleted();
   }
});
```

***2. 아래와 같이 subscribeOn()함수와 observeOn()를 추가한다.***
```
simpleObservable
   .subscribeOn(Schedulers.io())  //발행자를 별도의 thread에서 동작시킨다
   .observeOn(AndroidSchedulers.mainThread()) // 구독자를 mainThread에서 동작 시킨다.
   .subscribe(new Subscriber<String>() {  //observer (구독자)
       @Override
       public void onCompleted() {
           Log.d(TAG, "[Observer1] complete!");
       }

       @Override
       public void onError(Throwable e) {
           Log.e(TAG, "[Observer1] error: " + e.getMessage());
       }

       @Override
       public void onNext(String text) {
           Toast.makeText(MainActivity.this, "[Observer1]" +text, Toast.LENGTH_SHORT).show();
       }
   });
```
