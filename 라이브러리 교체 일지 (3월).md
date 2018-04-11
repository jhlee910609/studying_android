# 라이브러리 업데이트 작업 (3월)

## 1. 교체 순서

### 1.1. Network library 업데이트
- `Retrofit 1.x` -> `Retrofit 2.x` : `AdapterFactory` 달아서 Network Library 부터 교체 
- `OkHttp 2.x` -> `OkHttp 3.x` : Dagger2가 적용된 `ApiModule.java` `Okhttp client`부분 부터 수정

### 1.2. RxJava 1.x -> RxJava 2.x
- `.addCallAdapterFactory`에 RxJava2  와 RxJava1 혼용하며 순차적 교체 진행
- 프로젝트 전체 라이브러리 교체 후, RxJava1 제거

### 1.3. 새로 추가한 Library

```java
	// Network Library
	implementation "com.squareup.okhttp3:okhttp:3.9.1"
	implementation "com.squareup.okhttp3:logging-interceptor:3.9.1"
	implementation "com.squareup.retrofit2:retrofit:2.4.0"
	implementation "com.squareup.retrofit2:converter-gson:2.4.0"
	
	// RxJava2
	implementation "io.reactivex.rxjava2:rxjava:2.1.10"
	implementation "io.reactivex.rxjava2:rxandroid:2.0.2"
	implementation "com.jakewharton.rxbinding2:rxbinding:2.1.1"
```

## 2. Okhttp 2.x -> Okhttp 3.x : [ChangeLog](https://github.com/square/okhttp/blob/master/CHANGELOG.md#version-300-rc1) 
### 2.1. Deprecated  `OkhttpClient.clone()`
- `OkUrlFactory` 객체가 deprecated 됨에 따라 함께 deprecated 되었다. [Link](http://square.github.io/okhttp/3.x/okhttp-urlconnection/okhttp3/OkUrlFactory.html)
- `.newBuilder.setXXX()`을 통해 `OkhttpClilent`에 set해주면 된다.


### 2.2. `HttpLoggingInterceptor`
- 네트워크 통신 상황을 로깅하기 위해 [`HttpLoggingInterceptor`](https://square.github.io/okhttp/3.x/logging-interceptor/okhttp3/logging/HttpLoggingInterceptor.Level.html) 를 `OKhttpClient`에 set해줘야했다.
- 로그 Level이 `const`로 세분화 되어있다. 
	- `NONE/BASIC/HEADER/BODY`

```java
HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
OkHttpClient newClient = client.newBuilder().addInterceptor(loggingInterceptor).build();
```

## 3. Retrofit 1.x -> Retrofit 2.x
- [해당 미디움 포스트] (https://medium.com/@sasa_sekulic/quick-and-easy-guide-to-retrofit-2-0-setup-or-migration-with-rxjava-ab7a11bc17df)와 [Future studio](https://futurestud.io/tutorials/retrofit-2-upgrade-guide-from-1-9)를 참고 후, Retrofit version update 작업을 이행했다.
- Retrofit 2의 이해를 위해 [해당 블로그](https://inthecheesefactory.com/blog/retrofit-2.0/en)와 명원님께서 주신 eBook을 참고했다. 

### 3.1. `RetrofitError` 객체 사라짐
- 기존 `Retrofit 1.x` 방법과 달리 Error handling을 해야했다.
- `Retrofit 2.x`에서 `Network Error (code != 200)`가 발생할 경우, `IOException`을 발생시킨다.
![by Jake Wharton](https://ws1.sinaimg.cn/large/006tKfTcgy1fptirku238j317s0kedjj.jpg)
- 로그인할 때 이메일 중복, 비밀번호가 틀렸을 경우`(code != 200)`에도 `IOException`을 발생시켜 `try-catch`로 감싸 상황별 다르게 분기처리를 하였다. 문제는 서버로부터 response받은 body에 실려온 메시지를 꺼내는 방법이 달라졌다. 아래 문서를 참고해 해결하였다.
	- [Retrofit 1.x과 2.x의 에러 핸들링 변화](https://bytes.babbel.com/en/articles/2016-03-16-retrofit2-rxjava-error-handling.html)

## 4. RxJava 1.x -> RxJava 2.x : [ChangeLog](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0)
- 선행 교체작업을 이행한 `Retrofit`에 `addCallAdapterFactory`를 달아 순차적으로 진행하였다.

### 4.1. `CompositeSubscription` -> `CompositeDisposable`
- `.dispose()` vs `.clear()` : [stackoverflow](https://stackoverflow.com/questions/47057885/when-to-call-dispose-and-clear-on-compositedisposable?rq=1)
	- `activity, fragment` 혹은 `application lifecycle`와 연관지어서 생각하면 용도가 더 명확해진다.  

### 4.2. RxJava `.doOnXX();`
- 로그 찍거나 `.doXX()`가 콜 되었을 때, 간단한 처리(`loadingView.hide()/.show() 등`)를 주로 해준다.
	- Observable 스트림에 영향을 주지 않고, 함수만 넘긴다. 
- 언제 효율적일지 사실 감이 잘오지 않았지만 `onXX()` 안에 비즈니스 로직이 있을 경우, default로 해줘야 하는 작업들(`loadingView.hide()/.show(), logging 등`)은 doOnXX에서 처리하면 좀 더 가독성이 좋고 깔끔한 코드를 작성할 수 있지 않을까 생각한다. 
- [호출 시점, 디버깅 용도로 많이 활용](https://www.grokkingandroid.com/rxjavas-side-effect-methods/)
- [간단 작업 처리하는 용도](http://eyeahs.github.io/rxjava/blog/2016/09/07/rxjava-side-effect/)
- [RxJava error handling 1](http://www.baeldung.com/rxjava-error-handling)
- [RxJava error handling 2](http://blog.danlew.net/2015/12/08/error-handling-in-rxjava/)
- [RxJava error handling 3](http://eyeahs.github.io/rxjava/blog/2016/09/07/rxjava-side-effect/)
- [RxJava do~ Marvel diagram](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html#doOnError-io.reactivex.functions.Consumer-)
- [아래 로직에서...](https://medium.com/mindorks/rxjava2-and-retrofit2-error-handling-on-a-single-place-8daf720d42d6)

### 4.3. `from` -> `fromIterable()` || `fromArray()`로 기능 분리 
- Array -> `fromArray()`
- 자료구조 Collection Framework -> `fromIterable()`
- RxJava 1.x에서 혼용했던 `from()` 메소드를 2가지 기능으로 나눈 이유는 다음과 같다고 한다.

> Some operator overloads have been renamed with a postfix, such as fromArray, fromIterable etc. The reason for this is that when the library is compiled with Java 8, the javac often can't disambiguate between functional interface types.


### 4.5. `Single`

- 단 건의 Response & Request (페이징처리 x, 지속적인 값 배출 x)에 Observable을 Single로 바꿈
- 굳이 계속 어떤 데이터의 변화를 구독할 필요가 없을 경우, 그리고 결과 데이터가 딱 1개일 경우에 사용.
- 성능상의 이점보다, 다른 상황에 대한 용도가 다른 Observable 종류 중 하나 [Link](http://reactivex.io/documentation/ko/single.html)


## 5. 기타 작업
### 5.1. Lamda식 적용
- 메소드 안의 파라미터로 익명객체를 넘기는 코드를 Lamda식으로 일부 교체하였다.
	- 코드 간소화를 통해 가독성을 높히려고 하였다.
- 라이브러리를 교체하면서 미사용 코드들을 일부 거둬냈다.


## 6. 회고
- 안드로이드 개발에 있어 코어한 서드파티 라이브러리들을 공부하며 변화 과정을 알 수 있었다.
- app 전반적으로 사용된 라이브러리를 교체하는 작업이니만큼 꼼꼼히 하려고 노력했고, 해먹남녀의 앱의 전반적인 구조와 기능을 알 수 있었다.

	
### 6.1. 부족했던 점
- RxJava2 객체들에 대한 이해도 높히기 
	- Single 객체에 대한 이해도가 부족했다. 조금 더 자세히 알아보고 테스팅 또한 하드하게 해뵬 필요가 있는 것 같다.
	- 여러 오퍼레이터들에 대한 이해도를 높혀야 한다. 
- Dagger2에 대한 이해도 높히기
	- 라이브러리 교체 작업을 하면서 DI의 중요성 및 편리함을 알 수 있었지만 아직 이해도가 부족하다. 지속적으로 조금씩 공부해나가야할 것 같다. 






