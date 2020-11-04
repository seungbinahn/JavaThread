# 자바 클래스로더
자바 클래스를 JVM에 동적으로 로딩하는 JRE(자바 런타임 환경)의 일부이다.    
일반적으로 클래스는 요성 시 한 차례만 로드된다. 자바 런타임 시스템은 클래스로더 때문에   
파일과 파일 시스템에 대해 알 필요가 없다.    
클래스 로더는 로딩(.class), 링크(코드 레퍼런스 연결), 초기화(클래스의 static 값)을 담당한다.

## 종류
- 부트스트랩 클래스 로더(네이티브 코드로 작성)
> JVM 런타임 실행을 위한 기반이 되는 파일들을 로드   
> <JAVA_HOME>/jre/lib에 위치한 핵심 자바 라이브러리를 로딩   

- 확장 클래스 로더(Java9부터는 플랫폼 클래스 로더)
> 확장 디렉터리(<JAVA_HOME>/jre/lib/ext 또는 java.dex.dirs 시스템 속성)에 지정된 코드를 로드   

- 애플리케이션 클래스로더(java9부터는 시스템 클래스 로더)
> 확장 클래스 로드를 상송      
> CLASSPATH에 정의되거나, JVM 옵션에서 -cp, -classpath에 의해 지정된 클래스들이 로딩된다.   

- 사용자 정의 클래스 로더
> 애플리케이션 사용자가 직접 코드 상에서 생성해서 사용하는 클래스 로더   

## 기능
- 로딩 
> .class파일을 읽어 바이트 코드를 메소드 영역에 저장   
> 클래스가 로딩되면 힙 메모리 영역에 로딩된 클래스 유형의 객체를 생성   

- 링킹(Linking)
> 증명(Verification) : class 파일의 정확성 보장, 파일이 적절한 포맷, 유요한 컴파일러로 생성되었는지 확인       
> 검증에 실패하면 런타임 에러 발생(java.lang.VerifyError)   

> 준비(Preparation) : JVM은 메모리를 기본 값으로 초기화 후 클래스 변수들을 메모리에 할당   

> 해결(Resolution) : 심볼릭 참조를 직접 참조로 변경   

- 초기화(Initialization) : 정적 변수 초기화

## 원칙
- 위임 원칙(Delegation Principle)
> 클래스 로딩이 필요할 때 부모 클래스 로더로 로딩을 위임하는 것 

System -(위임)> Extension -(위임)> Bootstrap   
Bootstrap -(실패)> Extension -(실패)> System

- 가시성 원칙(Visibility Principle)
> 하위 클래스로더는 상위 클래스 로더가 로딩한 클래스를 볼 수 있지만, 그 역은 성립하지 않는다.

- 유일성 원칙(Uniqueness Principle)
> 하위 클래스 로더는 상위 클래스로더가 로딩한 클래스를 다시 로딩하지 않도록 하여 유일성을 보장한다.

- 언로드 불가
> 클래스 로더에 의해 로딩된 클래스들은 다시 JVM 상에서 없앨 수 없다. 

## Java EE의 클래스로더
SE 에서는 일반적으로 parent-first 클래스 로더 위임 모델을 사용한다.
하지만 이는 대부분의 자바 EE 웹 애플리케이션에는 맞지 않는다. 자바 EE를 실행하는 서버는
복잡하며, 여러 공급 업체에서 구현을 제공한다. 따라서 서버가 애플리케이션에 사용되는 것과 동일한
타사 라이브러리를 사용하고, 이에 따라 버전 충돌이 발생할 수 있다. 이는 parent-last 클래스 로더 위임 모델을 적용한다.

자바 EE 웹 애플리케이션 서버는 각 웹 애플리케이션에서 공통 된 서버 ClassLoader를 상속하고, 격리된 ClassLoader를 할당한다. 
애플리케이션들이 격리되므로 다른 애플리케이션에 접근할 수 없다. 이는 클래스 출돌을 방지하고
다른 웹 애플리케이션을 방해하지 못하게 하는 보안 조치가 제공된다. 

웹 애플리케이션 ClassLoader는 일반적으로 자신이 먼저 클래스를 로드 할 수 없는 경우에만 부모 클래스 로더에게 로드하도록 한다.
이를 통해 라이브러리의 우선순위를 조정할 수 있다. 이 위임 모델은 대부분 잘 동작하지만, 자바 EE 준수 서버는
예외적인 상황을 위해 위임 모델을 변경하는 기능을 제공한다.