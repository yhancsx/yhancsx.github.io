---
title: '[기술서적 리뷰] Clean Architecture - 2. 벽돌부터 시작하기: 프로그래밍 패러다임'
category: programming
tags:
  - study
  - clean architecture
  - structured programming
  - object oriented programming
  - functional programming
  - encapsulation
  - inheritance
  - polymorphism
---

## Chapter 3. 패러다임 개요

1938년 앨런 튜링(Alan Turing)이 바이너리 언어와 반복문, 분기문, 할당문 등의 구조를 이용해 실질적인 프로그램을 작성한 이후로 CS분야에는 많은 변화가 일어났다.

프로그래밍 측면에서는 어셈블러, 컴파일러가 등장하고 COBOL, PL/1, C, PASCAL, JAVA등 새로운 프로그래밍 언어가 쏟아졌다.

프로그래밍 패러다임 측면에서도 혁신적인 변화가 발생했다. 패러다임이란 프로그래밍을 하는 방법 또는 구조에 관한 것으로 언어에는 독립적이다. 현재까지 **구조적**, **객체지향**, **함수형** 3가지 형태의 패러다임이 등장하였고, 뒤에서 설명할 이유 때문에 이 세가지가 유일한 패러다임일 확률이 높다.

#### 구조적 프로그래밍

1968년 우리 모두가 알고 있는 다익스트라(Dijkstra)가 발견했다. 무분별한 goto 문장을 지양하고 if/then/else, do/while/until과 같은 구조로 대체할 것을 제시했다.

> 구조적 프로그래밍은 제어흐름의 직접적인 전환에 대해 규칙을 부여한다.

#### 객체 지향 프로그래밍

1966년 ALGOL 언어의 함수 호출 스택(stack) 프레임을 힙(heap) 공간으로 옮기면 함수 호출이 반환된 이후에도 함수에서 선언된 지역변수가 오랫동안 유지될 수 있음을 발견했다. 이러한 함수가 클래스의 생성자가 되었고, 지역변수는 멤버 변수, 중첩 함수는 메서드가 되었다. 함수 포인터를 특졍 규칙에 따라 사용하는 과정을 통해 다형성이 등장하였다.

> 객체 지향 프로그래밍은 제어흐름의 간접적인 전환에 대해 규칙을 부과한다.

#### 함수형 프로그래밍

수학적 문제를 해결하는 과정에서 발견한 람다(lambda) 계산법에서 영향을 받아 발견되었다. 1958년에 만들어진 LISP 언어의 근간이 되는 개념이 이 람다 계산법이다. 이 방법의 기초가 되는 개념은 불변성(immutability)으로, 변수의 값이 변경되지 않는다는 개념이다. 함수형 언어에서 할당문의 사용은 굉장히 까다로운 조건 아래서만 가능하다.

> 함수형 프로그래밍은 할당문에 대해 규칙을 부과한다.

#### 생각할 점

이들 세가지 패러다임은 프로그래머에게서 어떤 권한을 박탈한다. 무엇을 해야할지가 아닌 무엇을 하지 말아야 할지를 말해준다. 각각 goto문, 함수 포인터, 할당문을 앗아간다. 이 외에 우리에게서 가져갈 수 있는게 더 남아 있을까? 따라서 프로그래밍 패러다임은 앞으로도 딱 이 세가지 정도 밖에 없을 것이다.

1958년부터 1968년까지 10년 동안 이 세가지 패러다임이 만들어 졌지만 이후 새롭게 등장한 패러다임은 전혀 없다.

#### 결론

이 패러다임들은 우리가 공부할 아키텍처와 어떤 관계가 있을까? 

아키텍처 경계를 넘다들기 위한 매커니즘으로 다형성(객체 지향)을 이용한다. 함수형 프로그래밍을 이용하여 데이터의 위치와 접근 방법에 관한 규칙을 부과한다. 각 모듈의 기반 알고리즘으로 구조적 프로그램을 사용한다.

## Chapter 4. 구조적 프로그래밍

1950년대 다익스트라는 프로그래밍을 수학적으로 증명하는 과정에서 goto 문장이 모듈을 더 작은 단위로 재귀적으로 분해하는 과정에 방해가 되는 경우가 있다는 사실을 발견했다. 모듈을 분해할 수 없다면 분할 정복 접근법 같은 증명 방법을 사용할 수 없게 된다.

반면 모듈을 분해할 때 문제가 되지 않는 goto문의 **좋은** 사용방식은 if/then/else, do/while과 같은 분기 및 반복이라는 단순한 제어 구조에 해당한다는 사실을 발견했다.

이는 모든 프로그램은 순차(sequence), 분기(selection), 반복(iteration)이라는 세가지 구조만으로 표현할 수 있다는 순차 실행(sequential execution) 방법론과도 결합할 때 더욱 놀라웠다. 모듈을 증명 가능하게 하는 바로 그 제어 구조가 모든 프로그램을 만들 수 있는 제어 구조의 최소 집합과 동일하다는 것이었다.

다익스트라는 1968년 이 연구 결과를 **goto문의 해로움(Go To Statement Considered Harmful**이라는 글로 공개했고 이후 10년간의 논란 끝에 대다수의 현대 언어에서 goto 문장을 포함하지 않으며 다익스트라의 승리로 끝났다.

현재 언어에서 제어흐름을 전환할 수 있는 선택권 자체를 제공하지 않으므로 우리 모두는 여지 없이 구조적 프로그래머이다.

#### 결론

구조적 프로그래밍이 오늘날까지 가치 있는 이유는 프로그래밍에서 모듈을 작은 단위로 분해 해 반증 가능하게(테스트하기 쉽도록) 만들수 있는 능력 때문이다.
## Chapter 5. 객체 지향 프로그래밍

객체 지향(Object Oriented, OO) 설계 원칙이란 무엇일까?

캡슐화(encapsulation), 상속(inheritance), 다형성(polymorphism) 세가지 개념을 지원하는 것이라고 설명하는 사람들이 있다.

#### 캡슐화

OO는 데이터와 함수를 쉽고 효과적으로 캡슐화 하는 방법을 제공하긴 한다. 캡슐 바깥에서 데이터는 은닉되고, 일부 함수만이 외부에 노출된다. private, public 등의 개념을 통해 표현된다.

하지만 이는 OO에만 국한된 개념이 아니다. C언어에서도 완벽한 캡슐화가 가능하다.

```c
// point.h
struct Point;
struct Point *makePoint(double x, double y);
double distance(struct Point *p1, struct Point *p2);

// point.c
struct Point {
  double x, y;
};
struct Point *makePoint(double x, double y) {
  struct Point* p = malloc(sizeof(struct Point));
  p->x = x;
  p->y = y;
  return p;
};
double distance(struct Point *p1, struct Point *p2) {
  double dx = p1->x - p2->x;
  double dy = p1->y - p2->y;
  return sqrt(dx*dx+dy*dy);
};
```

point.h를 사용하는 측면에서는 `struct Point`의 멤버에 접근할 방법이 전혀 없다. 데이터의 구조와 함수가 어떻게 구현되었는지에 대해서 알 수 없다.

#### 상속

상속이란 단순히 어떤 변수와 함수를 하나의 유효 범위로 묶어서 재정의하는 일에 불과하다. OO 언어 이전 C 프로그래머는 언어의 도움 없이 손수 이러한 방식을 흔히 사용해 왔다.

```c
// point.h
struct NamedPoint;
struct NamedPoint *makeNamedPoint(double x, double y);
void setName(struct NamedPoint* np, char* name);
char* getName(struct NamedPoint* np);

// point.c
struct NamedPoint {
  double x, y;
  char* name;
};
struct Point *makeNamedPoint(double x, double y, char* name) {
  struct NamedPoint* p = malloc(sizeof(struct NamedPoint));
  p->x = x;
  p->y = y;
  p->name = name;
  return p;
};
void setName(struct NamedPoint* np, char* name) {
  np->name = name;
};
char* getName(struct NamedPoint* np) {
  return np->name
};
```

```c
// main.c
int main() {
  struct NamedPoint* origin = makeNamedPoint(0, 0, "origin");
  struct NamedPoint* upperRight = makeNamedPoint(1, 1, "upperRight");
  printf(
    distance(
      (struct Point*) origin, 
      (struct Point*) upperRight
    )
  );
}
```

main 부분을 보면 NamedPoint 데이터 구조가 마치 Point 데이터 구조에서 파생된 것처럼 동작하고 있다. 이는 NamedPoint가 Point의 멤버 변수를 그대로 포함하는 상위 집합으로 설계 되었기 때문이다.

이렇듯 OO 출현 이전부터 상속과 비슷한 기법이 사용되었다고 말할 수 있으나 흉내내는 요령이지 상속만큼 편리한 방식은 아니다. 특히나 이 기법으로 다중 상속을 구현하기란 훨씬 더 어렵다.

OO 언어가 완전히 새로운 개념을 만들지는 못했지만 데이터 구조에 가면을 씌우는 일을 상당히 편리한 방식으로 제공했다고 볼 수 있다. 0.5점 정도 점수를 줄 수 있다.

#### 다형성

OO 언어 이전에도 다형성을 표현할 수 있는 언어는 당연히 있었다.

```c
void copy() {
  int c;
  while ((c=getchar()) != EOF)
    putchar(c);
}
```

getchar가 문자를 읽는 `STDIN`, putchar가 문자를 쓰는 `STDOUT` 장치는 다형적이다.

유닉스 운영체제의 경우 모든 입출력 장치 드라이버는 open, close, read, write, seek 이 다섯가지 표준 함수를 제공해야 한다. FILE 데이터 구조는 이 다섯 함수를 가리키는 포인터를 포함한다.

```c
struct FILE {
  void (*open)(char* name, int mode);
  void (*close)()
  int (*read)();
  void (*write)(char c);
  void (*seek)(long index, int mode);
}
```

각 입출력 드라이버는 이들 함수를 토대로 구현체를 만든다.

```c
void open(char* name, int mode) {/*...*/};
void close() {/*...*/};
int read() {/*...*/};
void write(char c) {/*...*/};
void seek(long index, int mode) {/*...*/};

struct FILE console = {open, close, read, write, seek};
```

이제 STDIN을 FILE*로 선언하고 getchar를 구현할 수 있다.

```c
extern struct FILE* STDIN;

int getchar() {
  return STDIN->read();
}
```

함수를 가리키는 포인터를 응용한 것이 다형성이고, OO 이전의 언어에서도 다형성을 활용해 왔다. 

OO 언어는 다형성을 새롭게 제공한 것은 아니지만 좀 더 안전하고 편리하게 사용할 수 있게 해준다. 함수에 대한 포인터를 직접 사용하여 다형적 행위를 하는 이 방식은 준수해야 할 관례가 많고 버그가 발생하기 쉽다. OO 언어는 이러한 관례를 없애주며 실수할 위험이 없다. OO 언어를 사용하면 다형성은 대수롭지 않은 일이 된다.

- **다형성이 가진 힘**

다형성이 왜 좋을까? 앞의 입출력 장치 프로그램을 다시 살펴보자. 새로운 입출력 장치(필기체 인식, 음성 장치 등)가 추가되어 이 프로그램이 동작하도록 만드려면 어떤 수정을 가해야 할까? 

아무런 변경도 필요하지 않다!

복사 프로그램을 다시 컴파일할 필요 조차 없다. 복사 프로그램의 코드는 입출력 드라이버의 코드에 의존하지 않기 때문이다. 새로운 입출력 장치가 FILE에 정의된 다섯 가지 표준 함수를 구현한다면 복사 프로그램에서는 이 장치를 얼마든지 사용할 수 있다.

입출력 드라이버가 복사 프로그램의 plugin이 된 것이다. OO의 등장으로 어디서든 이 플러그인(plugin) 아키텍처를 적용할 수 있게 되었다.

- **의존성 역전 (Dependency Inversion)**

![foc]({{site.url}}{{site.baseurl}}/assets/images/clean_architecture/ca_foc.jpeg)

다형성 이전의 소프트웨어는 main 함수에서 고주순, 중간 수준 함수를 거쳐 저수준 함수를 호출하는 방향으로 설계되었다. 이러한 방식에서 소스 코드 의존성의 방향은 반드시 제어흐름을 따르게 된다.

main 함수가 고수준 함수를 호출하려면 고수준 함수가 포함된 모듈의 이름을 지정해야 한다. C의 `#include`, JAVA의 `import` 구문 등이다.

![di]({{site.url}}{{site.baseurl}}/assets/images/clean_architecture/ca_di.jpeg){: width="400" height="400"}

하지만 다형성이 끼어들면 특별한 일이 벌어진다.

HL1 모듈은 ML1 모듈의 F() 함수를 호출한다. 소스 코드 상에서는 HL1 모듈은 인터페이스를 통해 F() 함수를 호출하지만 런타임에서 이 인터페이스는 존재하지 않는다.

ML1과 I 인터페이스 사이의 소스코드 의존성(상속 관계)는 제어 흐름과는 반대인 점을 주목하자. 이를 의존성 역전(dependency Inversion)이라고 부른다.

OO 언어가 다형성을 안전하고 편리하게 제공한다는 사실은 소스 코드 의존성을 어디에서든 역전시킬 수 있다는 뜻이기도 하다. 소스 코드 의존성 전부에 대해 방향을 결정할 수 있는 절대적인 권한을 갖는 것이다.

이것이 바로 OO가 제공하는 힘이고 OO가 지향하는 것이다.

#### 결론

OO란 무엇인가? 이 질문에 대한 답변을 이제 할 수 있다.

OO란 다형성을 이용하여 시스템의 모든 소스 코드 의존성에 대한 절대적인 제어 권한을 획득할 수 있는 능력이다.

플러그인 아키텍처를 구성할 수 있고, 이를 통해 고주순의 정책을 포함하는 모듈은 저수준의 세부사항을 포함하는 모듈에 대해 독립성을 보장할 수 있다. 저수준의 세부사항은 중요도가 낮은 플러그인 모듈로 만들 수 있고, 고수준의 정책을 포함하는 모듈과는 독립적으로 개발하고 배포할 수 있다.

## Chapter 6. 함수형 프로그래밍

함수형 언어에서 변수는 변경되지 않는다.

#### 불변성

아키텍처를 고려할 때 이 개념이 중요한 이유는 단순하다. 경합(race)조건, 교착상태(deadlock) 조건, 동시 업데이트(concurrent update) 문제가 모두 가변 변수로 인해 발생하기 때문이다. 

다시 말해 우리가 동시성 어플리케이션, 다수의 스레드와 프로세스를 사용하는 어플리케이션에 마주치는 문제는 가변 변수가 없다면 발생하지 않을 것이다.

저장 공간이 무한하고 프로세서의 속도가 무한히 빠르다고 전제한다면 우리는 완전한 불변성을 띄는 시스템을 설계할 수 있다.

그래서 적당한 타협을 해야한다.

#### 가변성의 분리

![transaction memory]({{site.url}}{{site.baseurl}}/assets/images/clean_architecture/ca_tm.png)

가장 주요한 타협 중 하나는 어플리케이션 내부의 서비스를 가변 컴포넌트와 불변 컴포넌트로 분리하는 일이다. 불변 컴포넌트는 순수하게 함수형 방식으로만 작업이 처리되며, 어떤 가변 변수도 사용되지 않는다.

변수를 변경하는 컴포넌트와 변경하지 않는 컴포넌트를 분리해야 한다는 것이 중요한 포인트이다. 

#### 이벤트 소싱

하드웨어 기술력의 발전으로 저장 공간과 처리 능력이 폭발적을 상승하고 있다. 이에 따라 필요한 가변 상태는 적어진다.

고객의 계좌 잔고를 관리하는 은행 어플리케이션을 생각해보자. 기존엔 잔고를 가변 변수로 관리하고 입금, 출금 트랜잭션이 실행되면 잔고를 변경해야 한다.

이제 계좌를 변경하는 대신 트랜잭션 자체를 저장한다고 상상해보자. 누군가 잔고 조회를 요청할 때마다 계좌 개설 시점부터 발생한 모든 트랜잭션을 더한다. 가변 변수가 하나도 필요 없는 것이다.

시간이 지날 수록 트랜잭션 수는 끝 없이 증가하고 현재 잔고를 계산하기 위한 컴퓨팅 자원도 덩달아 커진다. 그래서 무한한 저장 공간과 무한한 처리 능력을 전제하는 것이다.

하지만 이 전략이 영원이 동작하도록 할 필요는 없다. 각 어플리케이션의 수명주기 동안만 문제없이 동작할 정도의 공간과 처리 능력이 있으면 된다.

이벤트 소싱(event sourcing)에 깔려 있는 기본 발상이 바로 이 전략이다. 어플리케이션은 CRUD가 아니라 CR만 수행한다. 변경과 삭제가 발생하지 않으므로 동시 업데이트 문제 또한 일어나지 않는다.

이 이야기가 터무니 없이 들릴 수도 있지만 소스 코드 버전 관리 시스템(git)이 정확히 이 방식으로 동작하고 있다.


