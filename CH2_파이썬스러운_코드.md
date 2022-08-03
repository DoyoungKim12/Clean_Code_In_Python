
# CH2. 파이썬스러운(Pythonic) 코드
- 아이디어를 파이썬으로 표현하는 방식과 그 특수성을 살펴볼 것
- 프로그래밍에서 관용구(idiom)는 특정 작업을 수행하기 위해 코드를 작성하는 특별한 방법
  - 일반적으로 관용적 표현이란 2개 이상의 단어가 모여 원래와 다른 의미를 가지는 것
    - 예를 들어, 호랑이 담배피던 시절이라고 하면 우리는 누구나 아주 먼 옛날을 뜻한다고 이해한다.  
  - 관용구는 코드이므로 언어에 따라 다르며, 이 관용구를 따른 코드를 '관용적'이라고 하는데 특히 파이썬에서는 이를 **'파이썬스럽다'** 고 한다.
  - 권장사항을 따르고 파이썬스러운 코드를 작성하는 이유는 여러가지가 있다.
    - 관용적인 방식으로 작성한 코드가 일반적으로 더 나은 성능을 낸다.
    - 코드의 사이즈가 더 작고 이해하기도 쉽다.
    - 전체 개발팀이 동일한 패턴과 구조에 익숙해지면 실수를 줄이고 문제의 본질에 집중할 수 있다.
- 이 장의 목표는 아래와 같다.
  - 인덱스와 슬라이스를 이해하고, 인덱싱 가능한 객체를 올바른 방식으로 구현하기
  - 시퀀스와 이터러블 구현하기
  - 컨텍스트 관리자를 만드는 모범사례 연구
  - 매직 메서드를 사용해 보다 관용적인 코드 구현
  - 파이썬에서 부작용을 유발하는 흔한 실수 피하기  

<br><br>

## 인덱스와 슬라이스
- 일부 데이터구조, 또는 타입은 자신이 가진 요소에 인덱스로 접근하는 것을 지원함
  - 파이썬은 다른 언어와 색다른 방법으로 접근하는 것을 지원한다.
    - 음수 인덱스를 사용한 접근
    - slice를 사용한 특정 구간의 요소로의 접근
    - 튜플, 문자열, 리스트의 특정 요소를 가져오려고 한다면, for 루프를 돌면서 수작업으로 요소를 선택하지 말고 이러한 방법을 사용하는 것이 좋다.

<br>

### 자체 시퀀스 생성
- 위에서 설명한 기능은 \_\_getitem__이라는 매직메서드 덕분에 동작 
  - myobject\[key]와 같은 형태를 사용할 때 호출되는 메서드
  - key에 해당하는 대괄호 안의 값을 파라미터로 전달

<br>

- 시퀀스는 \_\_getitem__과 \_\_len__을 모두 구현하는 객체이므로 반복이 가능 
  - 리스트, 튜플, 문자열은 표준 라이브러리에 있는 시퀀스 객체의 대표적인 예시

<br>

- 여기서는 시퀀스나 이터러블 객체를 만들지 않고 키로 객체의 특정 요소를 가져오는 방법에 대해 다룰 것

<br>

- 업무 도메인에서 사용하는 사용자정의 클래스에 \_\_getitem__을 구현할 때 고려해야 할 사항
  - 클래스가 표준 라이브러리의 래퍼(wrapper)일 경우, 기본 객체에 가능한 많은 동작을 위임할 수 있음
    - 즉, 클래스가 리스트의 래퍼인 경우, 리스트의 동일 메서드를 호출하여 호환성 유지 가능
    - 필요한 메서드가 있는 경우, 그냥 list 객체에 있는 동일한 메서드에 위임하면 됨
  - 래퍼도 아니고 내장 객체도 사용하지 않는 경우는 자신만의 시퀀스를 구현할 수 있으나 아래 사항에 유의
    - 범위로 인덱싱한 결과는 해당 클래스와 같은 타입의 인스턴스여야 함
    - slice에 의해 제공된 범위는 마지막 요소를 제외해야 함

<br><br>   

## 컨텍스트 관리자 (context manager)
- 컨텍스트 관리자는 파이썬이 제공하는 유용한 기능으로, 특별히 유용한 이유는 패턴에 잘 대응되기 때문
  - 해당 패턴은 모든 코드에 적용될 수 있으며, 사전조건과 사후조건을 가짐 (즉, 주요동작 전후에 작업을 실행하려 할 때 유용함)
  - 일반적으로 리소스 관리와 관련하여 컨텍스트 관리자를 자주 볼 수 있음
    - 예를 들어, 파일을 열면 파일 디스크립터 누수를 막기 위해 작업이 끝나면 적절히 닫히길 기대함
    - 서비스나 소켓에 대한 연결을 열었을 때에도 적절히 닫거나 임시파일을 제거하는 등의 작업이 필요함
  - 이러한 모든 경우에는 일반적으로 할당된 모든 리소스를 제거해야 함
    - 모든 것이 잘 처리되었을 경우의 해제는 쉽지만, 예외가 발생하거나 오류를 처리해야 한다면?
    - 가장 일반적인 방법은 finally 블록에 정리 코드를 넣는 것
    - 그러나 같은 기능을 **매우 우아하고 파이썬스러운** 방법으로 구현할 수 있음

<br>

```python
# 파이썬스럽지 못한 코드
fd = open(filename)
try:
  process_file(fd)
finally:
  fd.close()
  
# 파이썬스러운 코드
# with문은 컨텍스트 관리자로 진입하게 함
# open 함수는 컨텍스트 관리자 프로토콜을 구현 (즉, 예외가 발생한 경우에도 블록이 완료되면 파일이 자동으로 닫힘)
with open(filename) as fd:
  process_file(fd)
```

<br>

- 컨텍스트 관리자는 \_\_enter__와 \_\_exit__의 2개 매직메서드로 구성됨
  - 첫번째 줄에서의 with문은  \_\_enter__메서드를 호출, 이 메서드가 무엇를 반환하든 as 이후에 지정된 변수에 할당
  - 해당 라인이 시작되면 다른 파이썬 코드가 실행될 수 있는 새로운 컨텍스트로 진입
  - 해당 블록의 마지막 문장이 끝나면 컨텍스트가 종료, 처음 호출한 컨텍스트 관리자 객체의 \_\_exit__메서드 호출
    - 예외, 오류가 있는 경우에도 여전히 \_\_exit__ 메서드가 호출되므로 정리 조건을 안전하게 실행하기 편함

<br>

- 컨텍스트 관리자가 리소스 관리에 자주 사용되긴 하지만, 오직 해당 분야에만 사용하는 것은 아님
  - 블록 전후에 필요한 특정 로직을 제공하기 위해 자체 컨텍스트 관리자를 구현할 수도 있음
  - 관심사를 분리하고 독립적으로 유지되어야 하는 코드를 분리하기 좋은 방법

<br>

- 스크립트를 사용해 데이터베이스 백업을 하려는 경우를 생각해보자.
  - 주의사항은 백업을 오프라인 상태에서 해야한다는 점
  - 백업이 끝나면 백업 프로세스의 성공 여부에 관계없이 프로세스를 다시 시작해야 함
  - 첫번째 방법은 서비스 중지 > 백업 > 예외 및 특이사항 처리 > 서비스 재시작 으로 이루어진 거대한 단일 함수를 만드는 것
    - 진짜 이렇게 구현하는 경우가 있기 때문에, 바로 해결법을 제시하지 않고 좀 더 자세히 살펴봄

<br>

```python
def stop_database():
  run("systemctl stop postgresql.service")

def start_database():
  run("systemctl start postgresql.service")
  
class DBhandler:
  def __enter__(self):
    stop_database()
    return self
    
  def __exit__(self, exc_type, ex_value, ex_traceback):
    start_database()
    
def db_backup():
  run("pg_dump database")
  
# main 함수에서는 유지보수 작업과 상관없이 백업 실행, 백업에 오류가 있어도 여전히 __exit__ 호출
# __exit__에서는 블록에서 발생한 예외를 파라미터로 받음 (블록에 예외가 없으면 모두 None인 값들)
# __exit__의 반환 값을 잘 생각해야 함. 만약 True를 반환하면 잠재적으로 발생한 예외를 호출자에게 전파하지 않고 멈춤을 의미함
# 위 예시에서는 따로 return을 쓰지 않았는데, 그럼 True를 반환한다는 뜻?

def main():
# 블록 내부에서 컨텍스트 관리자의 결과를 사용하지 않음
# 적어도 이런 경우에는 __enter__의 반환 값은 쓸모가 없음 (as로 반환되는 무언가를 블록 내부에서 써야 한다는 말인가?)
# 일반적으로 필수는 아니지만, __enter__에서 무언가를 반환하는 것이 좋은 습관 (디자인할 때 블록이 시작된 후에 무엇이 필요한지 고려해야 함)
  with DBHandler(): 
    db_backup()
```

<br>

###  컨텍스트 관리자 구현
- 앞의 예제와 같은 방법으로 컨텍스트 관리자를 구현할 수 있음
  - \_\_enter__와 \_\_exit__ 매직메서드만 구현하면 해당 객체는 컨텍스트 관리자 프로토콜을 지원
  - 이렇게 구현하는 것이 일반적이지만 유일한 방법은 아님
  - 이 섹션에서는 컨텍스트 관리자를 좀 더 간결하게 구현하는 방법과, 특히 contextlib 모듈을 사용하여 보다 쉽게 구현하는 방법을 살펴볼 것

<br>

- contextlib 모듈은 컨텍스트 관리자를 구현하거나 더 간결한 코드를 작성하는 데 도움이 되는 많은 도우미 함수와 객체를 제공
  - 먼저 contextmanager 데코레이터를 보자
    - 함수에 contextlib.contextmanager 데코레이터를 적용하면 해당 함수의 코드를 컨텍스트 관리자로 변환함
    - 함수는 **제너레이터**라는 특수한 함수의 형태여야 함 (코드의 문장을 \_\_enter__와 \_\_exit__ 매직메서드로 분리)
    - 지금은 데코레이터와 제너레이터에 익숙하지 않아도 아래 예제는 이해가능함 (관련하여 자세한 내용은 7장에서 다룸)

<br>

```python
import contextlib

@contextlib.contextmanager
def db_handler():
  stop_database()
  yield
  start_database()
  
with db_handler():
  db_backup()
```

<br>

- 먼저 제너레이터 함수를 정의하고 @contextlib.contextmanager 데코레이터를 적용
  - yield문을 사용했으므로 제너레이터 함수가 됨
  - 중요한 것은, 데코레이터를 사용하면 yield문 앞의 모든 것은 \_\_enter__ 메서드의 일부로 취급된다는 것
  - yield문 다음에 오는 모든 것들은 \_\_exit__ 로직이 됨

<br>

- 이렇게 컨텍스트 매니저를 작성하면 기존 함수를 리팩토링하기 쉬운 장점이 있음
  - 많은 상태를 관리할 필요가 없고, 다른 클래스와 독립된 컨텍스트 관리자 함수를 만드는 경우에는 이렇게 하는 것이 좋음
  - 컨텍스트 관리자를 구현할 수 있는 더 많은 방법이 있으며, 표준 라이브러리인 contextlib 패키지에 있음

<br>

- 또 다른 도우미 클래스는 contextlib.ContextDecorator이다.
  - 컨텍스트 관리자 안에서 실행될 함수에 데코레이터를 적용하기 위한 로직을 제공하는 믹스인 클래스
    - 믹스인 클래스는 다른 클래스에서 필요한 기능만 섞어서 사용할 수 있도록 메서드만을 제공하는 유틸리티 형태의 클래스를 말함
  - 컨텍스트 관리자 자체의 로직은 앞서 언급한 매직메서드를 구현하여 제공해야 함

<br>

```python 
class dbhandler_decorator(contextlib.ContextDecorator):
  def __enter__(self):
    stop_database()
  
  def __exit__(self, ext_type, ex_value, ex_traceback):
    start_database()

@dbhandler_decorator()
def offline_backup():
  run("pg_dump database")
```

<br>

- 이전 예제와 다른 점은 with문이 없다는 것
  - 그저 함수를 호출하기만 하면 offline_backup 함수가 컨텍스트 관리자 안에서 자동으로 실행됨
  - 원본 함수를 래핑하는 데코레이터가 하는 일
  - 이 접근법의 유일한 단점은 완전히 독립적이라는 것
    - 이것은 좋은 특성이지만, 컨텍스트 관리자 내부에서 사용하고자 하는 객체를 얻을 수는 없음을 의미
    - \_\_enter__ 메서드가 반환한 객체를 사용해야 하는 경우는 이전의 방식을 선택해야 함
  - 데코레이터로서의 장점은 로직을 한번만 정의하면 동일한 로직이 필요한 함수에 원하는 만큼 재사용할 수 있다는 것    

<br>

- 마지막으로 contextlib의 기능 하나를 살펴보자.
  - contextlib.suppress는 컨텍스트 관리자에서 사용하는 util 패키지로 제공한 예외가 발생한 경우에는 실패하지 않도록 함
  - try/except 블록에서 코드를 실행하고 예외를 전달하는 것과 비슷하지만, 차이점은 로직에서 자체적으로 처리하고 있는 예외임을 명시한다는 것

<br>

```python
import contextlib

# DataConversionException은 입력데이터가 이미 기대한 것과 같은 포맷이어서 변환할 필요가 없으므로, 무시해도 안전하다는 것을 뜻함
with contextlib.suppress(DataConversionException):
  parse_data(input_json_or_dict)
```

<br><br>

##  프로퍼티, 속성과 객체 메서드의 다른 타입들
- 파이썬 객체의 모든 프로퍼티와 함수는 public이다. 
  - 다른 언어들은 public, private, protected의 프로퍼티를 가짐
  - 즉, 호출자가 객체의 속성을 호출하지 못하도록 할 방법이 없음
  - 엄격한 강제사항은 없지만, 몇가지 규칙이 존재함
     - 밑줄(_)로 시작하는 속성은 해당 객체에 대해 private을 의미, **외부에서 호출하지 않기를 기대**하는 것
     - 다시 말하지만, 이걸 금지하는 것은 절대 아님
     - 파이썬에서의 밑줄이 어떤 특성을 갖는지 그 관습을 이해하고 속성의 범위를 살펴보는 것이 의미가 있을 것

<br>

### 파이썬에서의 밑줄
- 파이썬에서 밑줄을 사용하는 몇가지 규칙과 세부 사항

<br>

```python
class Connector:
  def __init__(self, source):
    self.source = source
    self._timeout = 60
    
conn = Connector("postgresql://localhost")
conn.source
# 'postgresql://localhost' 라는 string을 반환
conn._timeout
# 60 을 반환
conn.__dict__
# {'source':'postgresql://localhost', '_timeout':60} 를 반환
```

<br>

- Connector 객체는 source로 생성되며, source와 \_timeout이라는 2개 속성을 가짐
  - source는 public, \_timeout은 private
  - 그러나 실제로는 두 속성에 모두 접근이 가능함
    - \_timeout은 connector 자체에서만 사용되고, 호출자는 해당 속성에 접근하지 않아야 함
    - 내부에서만 사용되고 바깥으로 호출되지 않아서 동일한 인터페이스를 유지, 언제든 필요할 때 안전하게 리팩토링 가능해야 함
    - 객체의 인터페이스가 유지되어 파급효과를 걱정하지 않아도 되므로 유지보수가 쉽고 견고한 코드 작성이 가능
    - 동일한 원칙이 메서드에도 적용됨

<br>

- 객체는 외부 호출 객체와 관련된 속성과 메서드만을 노출해야 한다. 
- 객체의 인터페이스로 공개하는 용도가 아니라면 모든 멤버에는 접두사로 **하나의 밑줄**을 사용하는 것이 좋다.

<br>

- 이는 객체의 인터페이스를 명확히 구분하기 위한 파이썬스러운 방식
  - 그러나 일부 속성과 메서드를 실제로 private으로 만들 수 있다고 오해하는 경우가 있고, 이는 잘못된 생각임 
  - 아래 예시에서는 timeout 속성을 이중 밑줄로 정의했다고 가정함
    - 일부 개발자는 이를 통해 일부 속성을 숨길 수 있으므로, timeout이 이제 private이며 다른 객체가 수정할 수 없다고 생각함
    - 그런데 \_\_timeout에 접근하려고 할 때 발생하는 예외는 AttributeError임
    - 이는 속성 자체가 존재하지 않는다는 것으로, 뭔가 기대한 바와 다른 일이 벌어졌음을 뜻함
  - 밑줄 2개를 사용하면 실제로 파이썬은 의도한 바와는 다른 이름을 만들고 이를 **맹글링**이라고 함
    - **\_<클래스명>\_\_<속성명>** 으로, 의도했던 \_\_<속성명>과는 다른 이름이 만들어짐
    - 이렇기 때문에, \_\_<속성명>의 속성은 존재하지 않는다는 AttributeError가 발생한 것
    - **이중 밑줄은 여러 번 확장되는 클래스의 메서드를 이름 충돌 없이 오버라이드** 하기 위해 만들어진 것
  - 속성을 private으로 정의하려는 경우, 하나의 밑줄을 사용하는 파이썬스러운 관습을 지키도록 하자.
    - 의도한 것이 아니라면, 이중 밑줄을 사용하지 말자.

<br>

### 프로퍼티
- 객체의 상태나 각 속성의 값을 기반으로 계산 등을 하려 한다면 프로퍼티를 사용하는 것이 좋음
- 프로퍼티는 객체의 속성에 대한 접근을 제어할 때 사용
  - 이렇게 하는 것도 파이썬스러운 코드임 (자바 등의 다른 언어에서는 접근 메서드를 만들지만, 파이썬은 프로퍼티 사용)
- 사용자가 등록한 정보에 잘못된 정보가 입력되지 않게 보호하는 예시를 보자.

<br>

```python
import re

EMAIL_FORMAT = re.compile(r"[^@]+@{[^@]+[^@]+")

def is_valid_email(potentially_valid_email: str):
  return re.match(EMAIL_FORMAT, potentially_valid_email) is not None
  
Class User:
  def __init__(self, username):
    self.username = username
    self._email = None

# 책에서는 여기에서 들여쓰기가 안되어있어서 마치 아래 2개 함수가 클래스와는 무관한 함수인 것처럼 보임
# 실제로 그게 맞는것인지 오타인지 잘 모르겠음

@property
def email(self):
  return self._email
  
@email.setter
def email(self, new_email):
  if not is_valid_email(new_email):
    raise ValueError(f"유효한 이메일이 아니므로 {new_email} 값을 사용할 수 없음")
  self._email = new_email

```

<br>

- 이메일 함수에 프로퍼티를 사용하여 몇가지 이점을 얻을 수 있음
  - 첫번째 @property 메서드는 private 속성인 email 값을 반환함
  - 두번째 메서드는 앞에서 정의한 프로퍼티에 @email.setter를 추가함
    - 이 메서드는 \<User>.email = \<new_email>이 실행될 때 호출되는 코드
    - 설정하려는 값이 실제 이메일 주소가 아닌 경우, 명확하게 유효성 검사를 시행, 문제가 없으면 새 값으로 속성 업데이트 
    - get_, set_ 접두어를 사용하여 사용자 메서드를 만드는 것보다 훨씬 간단하고, 이메일을 입력으로 받기 때문에 기대하는 값의 형태가 명확함

<br>

- 객체의 모든 속성에 대해 get_, set_ 메서드를 작성할 필요가 없다. 
- 대부분의 경우 일반 속성을 사용하는 것으로 충분하며, 속성 값을 가져오거나 수정할 때 특별한 로직이 필요한 경우에만 프로퍼티를 사용하자.

<br>

- 프로퍼티는 **명령-쿼리 분리 원칙(command and query separation - CC08)** 을 따르기 위한 좋은 방법
  - 명령-쿼리 분리 원칙은 객체의 메서드가 상태 변경, 값 반환 중 하나만 수행해야지 둘 다 동시에 하면 안된다는 것
  - 메서드 이름에 따라 실제 코드가 무엇을 하는지 혼돈스럽고 이해하기 어려운 경우가 있음
    - 예를 들어, set_email이라는 메서드를 if self.set_email("a@j.com")처럼 사용했다면?
    - a@j.com로 값을 설정하려는 것일까, 아니면 해당 값으로 설정되었는지 확인하려는 것일까?
    - 혹은 둘다? 
  - 프로퍼티를 사용하면 이러한 종류의 혼동을 피할 수 있음
    - @property 데코레이터는 무언가에 응답하기 위한 쿼리
    - @\<property_name>.setter는 무언가를 하기 위한 커맨드
  - 한 메서드에서 한 가지 이상의 일을 하지 말라.
    - 작업을 처리한 다음 상태를 확인하려면 메서드를 분리해야 한다. 

<br><br>

## 이터러블 객체
- 파이썬에는 리스트, 튜플과 같은 반복가능한 객체가 있음
- 그러나 이러한 내장 반복형 객체만 for 루프에서 사용가능한 것은 아님
  - 반복을 위해 정의한 로직을 사용해 자체 이터러블을 만들 수도 있음
  - 엄밀히 말하면 이터러블은 \_\_iter__ 매직메서드를 구현한 객체, 이터레이터는 \_\_next__ 매직메서드를 구현한 객체 (자세한 내용은 7장)
  - 파이썬의 반복은 이터러블 프로토콜이라는 자체 프로토콜을 사용해 동작함
    - 객체를 반복할 수 있는지 확인하기 위해 파이썬은 다음 2가지를 차례로 검사함
    - 객체가 \_\_next__나 \_\_iter__ 메서드 중 하나를 포함하는지 여부
    - 객체가 시퀀스이고 \_\_len__과 \_\_getitem__을 모두 가졌는지 여부 

<br>

### 이터러블 객체 만들기
- 객체를 반복하려고 하면 파이썬은 해당 객체의 iter() 함수를 호출함
  - 이 함수가 처음으로 하는 일은 해당 객체에 \_\_iter__ 메서드가 있는지를 확인하는 것
  - 만약 있으면 해당 메서드를 실행
  - 아래 예시는 일정 기간의 날짜를 하루 간격으로 반복하는 객체의 코드

<br>

```python
from datetime import timedelta

class DateRangeIterable:
  """자체 이터레이터 메서드를 가지고 있는 이터러블"""
  
  def __init__(self, start_date, end_date):
    self.start_date = start_date
    self.end_date = end_date
    self._present_day = stard_date
    
  def __iter__(self):
    return self
    
  def __next__(self):
    if self._present_day >= self.end_date:
      raise StopIteration
    today = self._present_day
    self._present_day += timedelta(days=1)
    return today
```

<br>

- 이 객체는 한 쌍의 날짜를 통해 생성되며, 다음과 같이 해당 기간의 날짜를 반복하면서 하루 간격으로 날짜를 표시함
  - for 루프는 만들어진 객체를 사용해 새로운 반복을 시작
  - iter() 함수를 호출, 이 함수는 \_\_iter__ 매직메서드 호출
  - \_\_iter__ 메서드는 self를 반환하고 있어, 객체 자신이 곧 이터러블임을 나타내고, 따라서 루프의 각 단계마다 자신의 next() 함수를 호출
  - next() 함수는 다시 \_\_next__메서드에게 위임, 해당 메서드는 요소를 어떻게 생산하고 하나씩 반환할지 결정
  - 더 이상 생산할 것이 없을 경우 StopIteration 예외를 발생시켜 알려줘야 함
  - 즉, for 루프가 작동하는 원리는 StopIteration 예외가 발생 때까지 next()를 호출하는 것과 같음

<br>

```python
r = DataRangeIterable(date(2019,1,1), date(2019,1,5))
next(r)
# datetime.date(2019,1,1)을 반환
next(r)
# datetime.date(2019,1,2)을 반환
next(r)
# datetime.date(2019,1,3)을 반환
next(r)
# datetime.date(2019,1,4)을 반환
next(r)
# StopIteration 예외 발생
```

<br>

- 이 예제는 잘 동작하지만 작은 문제가 하나 있음
  - 일단 한번 실행하면 끝의 날짜에 도달한 상태가 되므로, 이후에 호출하면 계속 StopIteration 예외가 발생
  - 즉, 두번째 for 루프에서는 작동하지 않음

<br>

- 이 문제가 발생하는 이유는 반복 프로토콜이 작동하는 방식 때문임
  - 이터러블 객체는 이터레이터를 생성하고, 이를 사용해 반복을 시행함
  - 위의 예제에서 \_\_iter__는 self를 반환했지만, 호출될 때마다 새로운 이터레이터를 만들 수도 있음
    - 매번 새로운 DataRangeIterable 인스턴스를 만들거나
    - \_\_iter__에서 제너레이터(이터레이터 객체)를 사용

<br>

```python
class DataRangeContainerIterable:
  def __init__(self, start_date, end_date):
    self.start_date = start_date
    self.end_date = end_date
    
  def __iter__(self):
    current_day = self.start_date
    while currunt_day < self.end_date:
      yeild current_day
      current_day += timedelta(days=1)
```

<br>

- 달라진 점은 각각의 for 루프는 \_\_iter__를 호출하고, \_\_iter__는 다시 제너레이터를 생성한다는 점
- 이러한 형태의 객체를 **컨테이너 이터러블** 이라고 함
  - 일반적으로 제너레이터를 사용할 때에는 컨테이너 이터러블을 사용하는 것이 좋음 

<br>

### 시퀀스 만들기
- 객체에 \_\_iter__() 메서드를 정의하지 않았지만 반복하기를 원하는 경우
  - iter() 함수는 객체에 \_\_iter__가 정의되어있지 않으면 \_\_getitem__을 찾고, 없으면 TypeError를 발생시킴
  - 시퀀스는 \_\_len__과 \_\_getitem__을 구현하고, 첫번째 인덱스 0부터 포함된 모든 요소를 한 번에 하나씩 차례로 가져올 수 있어야 함
  - 이번 섹션의 예제는 메모리를 적게 사용한다는 장점이 있음
    - 한 번에 하나의 날짜만 보관하고, 한 번에 하나씩 날짜를 생성하는 법을 알고 있음
    - 그러나 n번째 요소를 얻고 싶다면 도달할 때까지 n번 반복한다는 단점 발생
    - 이는 컴퓨터 과학에서 발생하는 전형적인 메모리와 CPU 사이의 트레이드오프
    - 이터러블을 사용하면 메모리는 적게쓰지만 n번째 요소를 얻기 위한 시간복잡도는 O(n)
    - 시퀀스를 사용하면 더 많은 메모리를 사용하지만(모든 것을 한 번에 보관하므로) 인덱싱의 시간복잡도는 O(1)

<br>

```python
class DateRangeSequence:
  def __init__(self, start_date, end_date):
    self.start_date = start_date
    self.end_date = end_date
    self._range = self._create_range()
    
  def _create_range(self):
    days=[]
    currunt_day = self.start_date
    while current_day < self.end_date:
      days.append(current_day)
      current_day += timedelta(days=1)
    return days
    
  def __getitem__(self, day_no):
    return self._range[day_no]
    
  def __len__(self):
    return len(self._range)
```

<br>

- DateRangeSequence 객체가 모든 작업을 래핑된 객체인 리스트에 위임하는 것을 확인
  - 음수 인덱스 등이 잘 동작함
  - 호환성과 일관성을 유지하는 가장 좋은 방법 

<br>

- 두 가지 구현 중 어느 것을 사용할지 결정할 때에는 메모리와 CPU 사이의 트레이드오프를 계산해보자.
- 일반적으로 이터레이션이 더 좋은 선택이고, 제너레이터는 더 바람직하지만 모든 경우의 요건을 염두에 두어야 한다. 

<br>

## 컨테이너 객체
- 컨테이너는 \_\_contains__메서드를 구현한 객체로, 해당 메서드는 일반적으로 Boolean 값을 반환
  - 파이썬에서 in 키워드가 발견될 때 호출
  - 예를 들면 아래의 첫번째 코드를 두번째 코드와 같이 해석 
    - element in container 
    - container.\_\_contains__(element)
  - 이 메서드를 잘 사용하면 코드의 가독성이 정말 높아짐 (파이썬스러운 코드다!)

<br>

- 2차원 게임 지도에서 특정 위치에 표시를 해야하는 문제
  - 아래와 같은 함수를 생각해볼 수 있음
  - 첫번째 if문이 상당히 난해해보임
    - 의도를 이해하기 어렵고, 직관적이지 않으며, 무엇보다 매번 경계선을 검사하기 위해 if문을 중복하여 호출
  - 지도에서 자체적으로 grid라 부르는 영역을 판단한다면 어떨까? 그리고 그 일을 더 작은 객체에 위임한다면?
    - 이렇게 하면 지도에게 특정 좌표가 포함되었는지만 물어보면 됨

<br>

```python
def mark_coordinate(grid, coord):
  if 0 <= coord.x < grid.width and 0 <= coord.y < grid.height:
    grid[coord] = MARKED
```

<br>

```python
class Boundaries:
  def __init__(self, width, height):
    self.width = width
    self.height = height
    
  def __contains__(self, coord):
    x,y = coord
    return 0 <= x < self.width and 0 <= y < self.height
    
class Grid:
  def __init__(self, width, height):
    self.width = width
    self.height = height
    self.limits = Boundaries(width, height)
    
  def __contains__(self, coord):
    return coord in self.limits

```

<br>

- 이 코드만으로 훨씬 효과적인 구현이 됨
  - 구성이 간단하고, 위임을 통해 문제를 해결함
  - 두 객체 모두 최소한의 논리를 사용, 메서드는 짧고 응집력이 있음
  - coord in self.limits는 의도를 직관적으로 표현하여 더 이상의 설명이 필요없음
 
<br><br>

## 객체의 동적인 속성
- \_\_getattr__ 매직메서드를 사용해 객체에서 속성을 얻는 방법을 제어할 수 있음
  - \<myobject>.\<myattribute>를 호출하면 파이썬은 객체의 사전에서 \<myattribute>를 찾아서 \_\_getattribute__를 호출함
  - 객체에 찾고 있는 속성이 없는 경우, 속성의 이름을 파라미터로 전달하여 \_\_getattr__라는 추가 메서드가 호출됨
    - 해당 값을 사용하여 반환 값을 제어하거나 새로운 속성을 만들 수 있음
  - 아래의 예시로 \_\_getattr__ 메서드를 설명

<br>

```python
class DynamicAttributes:
  def __init__(self, attribute):
    self.attribute = attribute
  
  def __getattr__(self, attr):
    if attr.startswith("fallback_"):
      name = attr.replace("fallback_", "")
      return f"[fallback resolved] {name}"
    raise AttributeError(f"{self.__class__.__name__}에는 {attr} 속성이 없음.")
    
dyn = DynamicAttributes("value")
dyn.attributes
# 'value' 반환
dyn.fallback_test
# '[fallback resolved] test' 반환
dyn.__dict__['fallback_new'] = 'new_value'
dyn.fallback_new
# 'new value' 반환
getattr(dyn, 'something', 'default')
# 'default' 반환

```

<br>

- 첫번째 호출은 객체에 있는 속성을 요청하고, 그 결과 값을 반환
- 두번째 호출은 객체에 없는 fallback_test라는 메서드를 호출하기 때문에 \_\_getattr__이 호출되어 값을 반환
- 세번째 예제에서 fallback_new라는 새 속성이 생성된 점이 흥미로움
  - 실제로 이 호출은 dyn.fallback_new = 'new_value'를 실행한 것과 동일함
- 마지막 예제에서는 값을 검색할 수 없는 경우 AttributeError가 발생한다는 점에 유의해야 함
  - 예외 메세지를 포함하여 일관성을 유지할 뿐만 아니라, 내장 getattr() 함수에서도 필요한 부분
  - 해당 예외가 발생하면 기본 값을 반환
  - \_\_getattr__와 같은 동적인 메서드를 구현할 때에는 AttributeError를 발생시켜야 한다는 것에 주의하자.

<br><br>

## 호출형 객체
- 함수처럼 동작하는 객체를 정의하면 매우 편리함
  - 가장 흔한 사례는 데코레이터이지만, 그것만 존재하는 것은 아님
  - 매직메서드 \_\_call__을 사용하면 객체를 일반 함수처럼 호출할 수 있음
  - 이렇게 사용할 때의 이점은 객체에는 상태가 있기 때문에 함수 호출 사이에 정보를 저장할 수 있다는 점
  - 파이썬은 object(\*args, \*\*kwargs)와 같은 구문을 object.\_\_call__(\*args, \*\*kwargs)로 변환함
  - 해당 메서드는 객체를 파라미터가 있는 함수처럼 사용하거나 정보를 기억하는 함수처럼 사용할 때 유용함
  - 아래 예제에서는 입력된 파라미터와 동일한 값으로 몇번이나 호출되었는지를 반환하는 객체를 만들 때 해당 메서드를 사용함

<br>

```python
from collections import defaultdict

class CallCount:
  def __init__(self):
    self._counts = defaultdict(int)
    
  def __call__(self, argument):
    sefl._counts[argument] += 1
    return self,._counts[argument]

cc = CallCount()
cc(1)
# 1
cc(2)
# 1
cc(1)
# 2
cc(1)
# 3
cc('something')
# 1
```

<br><br>

## 매직메서드 요약
- 이전 섹션에서 설명한 개념을 아래의 컨닝페이퍼 형태로 요약할 수 있음
- 파이썬에서의 실행은 매직메서드를 내포하고, 다음의 개념을 가짐

|문장|매직 메서드|파이썬 컨셉|
|-----|---|---|
|obj\[key], obj\[i:j], obj\[i:j:k]|\_\_getitem__(key)|첨자형 객체|
|with obj: ...|\_\_enter__, \_\_exit__|컨텍스트 관리자|
|for i in obj: ...|\_\_iter__, \_\_next_, \_\_len__, \_\_getitem__|이터러블 객체, 시퀀스|
|obj.\<attribute>|\_\_getattr__|동적 속성 조회|
|obj(\*args, \*\*kwargs)|\_\_call__(\*args, \*\*kwargs)|호출형 객체|


<br><br>

## 파이썬에서 유의할 점
- 방어코드를 작성하지 않으면 오랜 시간 디버깅으로 고생할 수 있는 일반적인 이슈를 살펴볼 것
- 여기서 논의되는 내용들은 대부분 예방 가능하고, 감히 안티패턴을 정당화하는 시나리오는 거의 없다고 볼 수 있음
  - 따라서 작업 중 이러한 코드를 발견하면 제안된 방식으로 리팩토링 필요 (무언가 수정이 필요하다는 분명한 신호)

<br>

### 변경가능한 파라미터의 기본 값
- 변경 가능한 객체를 함수의 기본 인자로 사용해서는 안됨
- 아래의 함수 예시를 보자

<br>

```python
def wrong_user_display(user_metadata: dict={"name":"John", "age":"30"}):
  name = user_metadata.pop("name")
  age = user_metadata.pop("age")
  return f"{name} ({age})"
  
wrong_user_display()
# 'John (30)'을 반환
wrong_user_display("name":"Jane", "age":"25"})
# 'Jane (25)'을 반환
wrong_user_display()
# KeyError: 'name'
```

<br>

- 여기에는 사실 2개의 문제가 있음
  - 변경 가능한 인자를 사용한 것
  - 함수의 본문에서 가변 객체를 사용한 것 

<br>

- 이 함수는 인자를 사용하지 않고 처음 호출할 때만 동작하고, 이후에는 명시적으로 user_metadata를 전달하지 않으면 KeyError 발생
  - 기본 값을 사용해 함수를 호출하면, 기본 데이터로 사용될 딕셔너리를 한 번만 생성하고, user_metadata는 이를 바라봄
  - 해당 값은 프로그램이 실행되는 동안 계속 메모리에 남아있게 되는데, 함수의 본체에서 객체를 수정하고 있음
  - 이 상태에서 함수의 파라미터에 새로운 값을 전달하면 조금 전에 사용한 기본 인자 대신 새로운 값을 사용
  - 이후 다시 파라미터를 사용하지 않고 기본 값을 호출하면 실패함 (첫번째 호출 시 key를 지워버렸기 때문)
  - 수정방법은 간단함 (기본 초기 값으로 None을 사용, 함수 본문에서 기본 값을 할당)

<br>

### 내장(built-in) 타입 확장
- 리스트, 문자열, 딕셔너리와 같은 내장 타입을 확장하는 올바른 방법은 collection 모듈을 사용하는 것
- dict, list에서 직접 확장하지 말고, 대신 Collection.UserDict, Collection.UserList를 사용해야 함 (String도 마찬가지)

<br><br>






