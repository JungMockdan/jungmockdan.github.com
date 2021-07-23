---
title: "Facade pattern"
date: 2021-07-23 08:26:28 -0400
categories: 디자인패턴
tags: gof java design-pattern Facade
# 목차
toc: true  
toc_sticky: true
---

* façade : (프) 건물의 정면. 건물 뒷면에 뭐가 있는지 모름...
* 여러개의 객제와 실제 사용하는 서브객체 사이의 복잡한 의존관계가 존재할때, 중간에 facade라는 객체를 두고, 여기서 제공하는 interface만을 활용하여 기능을 사용하는 방식.
* 각 인터페이스와 클래스의 기능을 명확히 알아야 한다.
* 실제 ftp client를 생각해보면 이해에 도음이 된다.
    * 각 ftp, writer, reader는 커넥션과 커넥션 클로즈 같은 메소드를 각각 필요로 하게 되는데 이경우 중간에 facade를 둠으로서 그 혼재된 상황정리해서 facade가 다양한 메서드를 제공하게 된다. 실제 아래 예시도 이와 같은 형태로 구현될 예정이다.


## 예제코드 (FTP Client)
### facade 적용 전
#### 클라이언트 각 클래스들(FTP, Writer, Reader)
```java
public class FTP{
    private String host;
    private int port;
    private String path;
    
    public FTP(String host, int port, String path){
        this.host = host;
        this.port = port;
        this.path = path;
    }
    public void connect(){
      System.out.println("FTP host : "+host+", Port : "+port+", path : "+ path);
    }
    
    public void moveDirectory(){
      System.out.println("FTP path : "+ path+"로 이동합니다.");
    }
    public void disconnect(){
      System.out.println("FTP 연결을 종료합니다.");
        
    }

}
```
```java
public class Writer{
  private String fileName;

  public Writer(String fileName) {
    this.fileName = fileName;
  }

  public void fileConnect() {
    System.out.println("Writer "+ this.fileName+"로 연결 합니다.");
  }
  public void write() {

    System.out.println("Writer "+ this.fileName+"로 파일을 씁니다.");
  }
  public void fileDisconnect() {
    System.out.println("Writer "+ this.fileName+"연결 종료합니다.");
  }
}
```

```java
public class Reader {
  private String fileName;

  public Reader(String fileName) {
    this.fileName = fileName;
  }

  public void fileConnect() {
    System.out.println("Reader "+ this.fileName+"로 연결 합니다.");
  }
  public void fileRead() {
    
    System.out.println("Reader "+ this.fileName+"의 내용을 읽어옵니다.");
  }
  public void fileDisconnect() {
    System.out.println("Reader "+ this.fileName+"연결 종료합니다.");
  }
}
```

### 실행코드
- facade 적용 전 : 실제 코드 사용시 메서드 호출 순서가 매우 중요해지고, 호출하는 메서드도 많다.
```java
public class Main {

    public static void main(String[] args){
    	FTP ftpClient = new FTP("www.test.co.kr". 22, "/home/test");
        ftpClient.connect();
        ftpClient.moveDirectory();
        
    	Writer writer = new Writer("test.txt");
    	writer.fileConnect();
    	writer.write();
    	
    	Reader reader = new Reader("test.txt");
    	reader.fileConnect();
        reader.fileRead();

        reader.fileDisconnect();
        writer.fileDisconnect();
        ftpClient.disconnect();
    }
}
```

### 파사드  패턴 구현 (기존 객체 조합해서 하나의 창구(?)가 됨.)
#### Facade형태 객체 생성

```java
public class SftpClient {
  private Ftp ftp;
  private Writer writer;
  private Reader reader;

  public SftpClient(Ftp ftp, Writer writer, Reader reader) {
    this.ftp = ftp;
    this.writer = writer;
    this.reader = reader;
  }

  public SftpClient(String host, int port, String path, String fileName) {
    this.ftp = new FTP(host, port, path);
    this.writer = new Writer(fileName);
    this.reader = new Reader(fileName);
  }

  public void connet() {
    ftp.connect();
    ftp.moveDirectory();
    writer.fileConnect();
    reader.fileConnect();
  }

  public void disconnect() {
    reader.fileDisconnect();
    writer.fileDisconnect();
    ftpClient.disconnect();
  }

  public void read() {
    reader.fileRead();
  }
  public void write() {
    writer.write();
  }
}
```
### 실행코드
- 파서드 패턴 적용 후 , 각 객체안의 메소드를 각각 불러주고 순서를 고려했던것과는 다르게 정리된 코드를 확인할 수 있다.
```java
public class Main {
   
    public static void main(String[] args){
      SftpClient sftpClient = new FTP("www.test.co.kr". 22, "/home/test","test.txt");
      sftpClient.connect();
      sftpClient.write();
      sftpClient.read();
      sftpClient.disconnect();
    }
    
    
}
```



