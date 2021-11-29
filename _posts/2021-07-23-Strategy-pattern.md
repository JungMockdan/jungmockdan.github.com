---
title: "Strategy pattern"
date: 2021-07-23 13:56:28 -0400
categories: design-pattern
tags: gof java design-pattern Strategy 전략패턴


# 목차
toc: true  
toc_sticky: true
---

* 유사행위들을 캡슐화함.
* 객체 행위를 바꾸고 싶은 경우 직접 객체변경이 아닌 전략만 변경하여 유연하게 확장함.
* SOLID 원칙들 중에서 객방패쇄원칙(OCP)과 의존역전원칙(DIP)을 따름

## 예제코드 (인코더)
- 인코딩 없는 전략객체, Base64, Base32 객체등 의 여러 전략객체를 사용하는 컨텍스트인 Encoder 그리고, 전략객체를 생성해 컨텍스트에 주입하는 Client로 이루어진 인코더를 프로그래밍 해본다.

### 전략패턴 적용 전
#### 인코딩 전략 인터페이스와 전략들
```java
public interface EncodingStrategy{
    String encode(String text);
}
```

```java
public class NormalStrategy implements EncodingStrategy{
  @Override
  public String encode(String text){
      return text;
  }
}
```
```java
import java.util.Base64;
public class Base64Strategy implements EncodingStrategy{
  @Override
  public String encode(String text){
      return Base64.getEncoder().ecodeToString(text.getBytes());
  }
}
```
```java
public class OtherStrategy implements EncodingStrategy{
    @Override
    public String encode(String text){
        return "otherStrategy : "+text;
    }
}
```

- 아래 클래스의 변경없이 Strategy의 추가나 수정만으로 다양한 케이스에 대해 대응할 수 있다.
```java
public class Encoder {
  private EncodingStrategy encodingStrategy;

  public void setEncodingStrategy(EncodingStrategy encodingStrategy) {
    this.encodingStrategy = encodingStrategy;
  }
}
```

### 실행코드
```java
public class Main {

    public static void main(String[] args){
    	
        Encoder encoder = new Encoder();
        String message = "hello~";
    	
    	// base64
        EncodingStrategy base64 = new Base64Strategy();
        encoder.setEncodingStrategy(base64);
        String base64Result = encoder.getMessage(message);

        // normal
        EncodingStrategy normal = new NormalStrategy();
        encoder.setEncodingStrategy(normal);
        String normalResult = encoder.getMessage(message);
        
        // other
        EncodingStrategy normal = new OtherStrategy();
        encoder.setEncodingStrategy(normal);
        String normalResult = encoder.getMessage(message);
    }
}
```

```



