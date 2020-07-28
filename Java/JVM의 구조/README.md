# JVM의 구조

 JVM(Java Virtual Machine)은 자바 어플리케이션을 실행시키기 위한 가상 머신입니다. Java는 WORA(Write Once Run Anywhere)이라는 목표로 개발된 언어입니다. 이러한 목표로 한 번 작성한 프로그램을 실행 환경에 구애받지 않고 어디서든 실행시킬 수 있도록 하기 위해 JVM이라는 가상 머신이 존재하는 것입니다.

## 자바가 실행되는 과정

 사용자가 작성한 소스 코드는 .java 라는 확장자를 가진 파일입니다. **.java 파일은 Compiler에 의해 .class 확장자를 가진 bite-code가 생성됩니다. bite-code는 JVM에 의해 로드되어 실행됩니다.** 아래는 JVM 구조에 대한 그림입니다.

![JVM Architecture](./images/JVM-Architecture.png)

 JVM은 위의 그림에서 보이는 것처럼 크게 3개의 서브시스템으로 나누어져 있습니다.

-   Class Loader Subsystem
-   Runtime Data Area
-   Execution Engine

### Class Loader Subsystem

 자바의 동적 클래스 로딩 기능은 Class Loader Subsystem에 의해 처리됩니다. 동적 클래스 로딩은 런타임 시에 클래스를 동적으로 읽어서 JVM 메모리에 로딩시키는 것입니다. 좀 더 자세한 내용은 아래의 링크를 참조하시면 좋습니다.

[https://futurists.tistory.com/43](https://futurists.tistory.com/43)

 Class Loader Subsystem은 런타임 중에클래스 파일이 처음으로 참조 되었을 때, 내부적으로 **Loading, Linking, Initialization**이라는 세 가지 단계를 거쳐서 JVM 메모리에 적재하게 됩니다.

#### Loading

 클래스들은 이 과정에서 로딩됩니다. 클래스들을 로딩하기 위해 **Bootstrap Class Loader, Extension Class Loader, Application Class Loader**라는 세 가지 로더가 이용됩니다.

**Bootstrap Class Loader**

 JVM이 실행될 때 가장 먼저 실행되는 로더입니다. $JAVA\_HOME/jre/lib에 있는 JVM 실행에 필요한 가장 기본적인 라이브러리(rt.jar 등)를 로딩합니다.

**Extension Class Loader**

 Bootstrap 로딩이 끝난 후에 $JAVA\_HOME/jre/lib/ext 에 있는 클래스들을 로딩합니다.

**Application Loader**

환경 변수와 같은 어플리케이션 수준의 classpath를 로딩합니다.

#### Linking

 링킹 과정에서는 **Verify, Prepare, Resolve** 과정을 거칩니다.

**Verify** - bite-code가 적절하게 포맷하여 생성되었는지 검증합니다. 만약 검증에 실패한다면 java.lang.VerifyError라는 런타임 익셉션이 발생합니다.

**Prepare** - 클래스 변수들을 위해 JVM 메모리를 할당하고, 해당 메모리를 default value로 초기화 합니다.

**Resolve** - 모든 Symbolic memory references를 Method Area의 Direct references로 대체합니다. 

#### Initialization

 초기화 과정은 Class Loader Subsystem의 마지막 단계로, code와 static block에 정의되어 있는 모든 정적 변수를 할당합니다. 이는 클래스 내에서 위에서 부터 아래 순으로 진행되고, 부모 클래스에서 자식 클래스로의 상속 구조 순으로 진행됩니다.

---

## 참고자료

[https://dzone.com/articles/jvm-architecture-explained](https://dzone.com/articles/jvm-architecture-explained)

[https://www.geeksforgeeks.org/jvm-works-jvm-architecture/](https://www.geeksforgeeks.org/jvm-works-jvm-architecture/)
