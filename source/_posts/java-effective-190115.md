---
title: '[Effective-java] Static Factory Method'
tags:
- java
---

이번 포스팅은 Effective-java 3rd(Joshua Bloch)를 읽고 정리한 내용이다. Effective-java의 구성은 Java를 구현할 때 생각해 봐야 할 규칙들을 정의하고 해당 규칙을 적용했을 때의 장점들을 정리하고 있다. 결국 Java에 대한 여러 가지 관점을 제시한 책이다. 필자는 책에서 소개하는 여러 가지 관점들을 정리하면서 포스팅할 예정이다.

## *규칙1) 생성자 대신 정적 팩토리 메소드를 사용할 수 없는 지 생각해 보자.*
1. 장점1
2. 장점2
3. 장점3
4. 장점4
5. 장점5   

클래스를 통해 객체를 생성하는 방법는 보통 `public`으로 선언된 생성자를 이용하는 방법이다. 하지만 생성자를 만들기 전에 한번쯤 고민해 봐야 할 방법이 하나 있는데, 규칙1)에서 소개하고 있는 정적 팩토리 메소드 방법이다. 책에서는 아래와 같은 코드를 예시로 들고 있다.   

``` java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```   

코드를 간단하게 살펴보면 메소드는 Return 타입이 Boolean이고 Boolean을 입력 매개변수로 받는다. 입력된 매개변수에 따라 반환값이 결정되는 메소드로 볼 수 있다. 과연 생성자를 쓰는 거 보다 더 좋은 점이 있을까? 일단 책에서 소개해 주는 장점들을 바탕으로 코드를 작성해 보면서 장점을 느껴보자.      

### 장점1) 생성자와는 달리 메소드에는 이름(name)이 있다.   
생성자에 전달되는 매개변수들로는 객체의 Identity를 설명하지 못한다. 하지만 정적 팩토리 메소드로 구현을 하게 되면 정의한 메소드 이름(name)으로 메소드의 Identity를 정의할 수 있게된다. 이는 클라이언트 코드에서 해당 메소드를 사용할 때 가독성(Readability)이 높아지게 된다. 참고로 생성자의 이름은 클래스의 이름과 동일하다.   

``` java   
/**************************************************************
  <장점1> static factory method에는 이름이 존재한다.
**************************************************************/
public class Drill {
    
    private String id;
    private int num;
    
    //Constructor
    public Drill() { }
    
    public Drill(String id) {
      this.id = id;
    }
    public Drill(int num) {
      this.num = num;
    }
    // 장점 1 > naming
    public static Drill withId(String id) {
      return new Drill(id);
    }
    public static Drill withNum(int num) {
      return new Drill(num);
    }
    
    //클라이언트 소스에서 사용할 때, 메소드 이름으로 접근
    public static void main(String[] args) {
          Drill drill = Drill.withId("minikuma");
          Drill numdrill = Drill.withNum(100);
    }
  }
```   

### 장점2) 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요가 없다.   
생성자를 생성하여 불변객체 혹은 캐시한 인스턴스를 반환하여 중복된 객체 생성을 방지할 수 있다.   

``` java      
/***********************************************************************
  <장점2> 객체를 호출 할 때마다 매번 새로운 객체를 생성할 필요는 없다.
***********************************************************************/
public class Drill {
  
    private static final Drill GOOD_ID  = new Drill.withId("minikuma");
    private static final Drill GOOD_NUM = new Drill.withNum(100);
    
    private String id;
    private int num;
    
    public Drill() { }
    
    public Drill(String id) {
      this.id = id;
    }
    public Drill(int num) {
      this.num = num;
    }    
    public static Drill withId(String id) {
      return new Drill(id);
    }
    public static Drill withNum(int num) {
      return new Drill(num);
    }    
    //클라이언트 소스에서 사용
    public static void main(String[] args) {
          //미리 생성된 객체를 재 사용
          Drill drill = GOOD_ID;
          Drill numdrill = GOOD_NUM;
    }
  }
```   

### 장점3) 해당 객체의 하위 타입을 리턴할 수 있다.   
객체를 사용하는 클라이언트에서는 클래스의 실제 구현체를 노출 시키지 않지만(사용하는 쪽에서는 구현체를 모름), 그 구현체를 사용가능하게 인스턴스로 만들어 줄 수 있다. 결론적으로 클라이언트는 객체로부터 제공받은 Interface로 코딩을 할 수 있게 된다. 이는 클라이언트가 구현할 때 알아야 할 API의 갯수가 줄어듬을 의미하기도 하며, 이를 책에서는 개념적인 무게(Conceptual Weight)가 줄어든다라는 표현으로 소개하였다.   

### 장점4) "장점3)"과 연관성 있는 장점으로 리턴하는 객체의 클래스가 입력 매개변수에 따라 다른 리턴값을 가지게 할 수 있다.   
"장점3)", "장점4)"을 통해 말하고자 하는거는 결국 클라이언트는 객체 혹은 클래스로부터 제공된 Interface 혹은 API를 바탕으로 코드를 작성하면 된다는 것이다.   

``` java   
/**************************************************************
  <장점3> 해당 객체의 하위 타입을 리턴할 수 있다.
  <장점4> 리턴하는 객체의 클래스가 입력 매개변수에 따라 다른 리턴값을 갖게 할 수 있다.
*************************************************************/
public class Drill {
  
    private String id;
    private int num;
    
    public Drill() { }

    // 장점 2 > Instance cache, immutable object
    private static final Drill CACHE = new Drill();
    
    public Drill(String id) {
      this.id = id;
    }
    public Drill(int num) {
      this.num = num;
    }
    // 장점 1 > naming
    public static Drill withId(String id) {
      Drill d = new Drill();
      d.id = id;
      return d;
    }
    public static Drill withNum(int num) {
      return new Drill(num);
    }
    // 장점 2, 3 > Instance cache, immutable object
    public static Drill getInstance(boolean flag) {
      return flag ? new Drill() : new BarDrill();
    }
    // Drill의 하위 Type Class인 BarDrill
    static class BarDrill extends Drill {
    }
    
    //client 소스에서 사용
    public static void main(String[] args) {
          // as-is 방식 (new)
          Drill d = new Drill("minikuma");
          //장점 1 > static method
          Drill d1 = withId("minikuma");
          //장점 3, 4 > cach, immutable object        
          Drill d2 = getInstance(false);

          System.out.println("d -> " + d);
          System.out.println("d1 -> " + d1);
          System.out.println("d2 -> " + d2);
    }
}
```   
**결과**   
![drill](https://user-images.githubusercontent.com/20740884/51237898-d7fefd00-19b8-11e9-95e4-5c73f0abb224.JPG)   

필자도 하위 객체를 어떻게 리턴할지 정확하게 감이 오질 않았다. 하지만 실제 코드로 구현한 결과를 보면 Drill 클래스의 하위 클래스인 BarDrill도 정적 팩토리 메소드로 생성이 가능함을 알 수 있다.   

### 장점5) 객체의 클래스의 리턴 값이 Static Method 작성 시점에 존재하지 않아도 된다.   
"장점3)", "장점4)"와 관련된 장점이고 결국 객체의 유연함에 대한 이야기 이다. Static Method안에서 다른 인스턴스를 Runtime 시에 공급받을 수도 있다. 해당 내용은 Service Loader에 관한 내용인데, 해당 내용은 다음 포스팅에서 다룰 예정이다.   
