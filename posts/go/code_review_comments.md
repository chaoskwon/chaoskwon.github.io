---
title: "Go 코드리뷰"
date: 2021-01-03T21:17:30-04:00
categories:
  - post
tags:
  - Go
  - Golang
  - 고
  - 코드 리뷰
---

이 페이지는 아래 원본 사이트의 번역본입니다.  
 원본 : https://github.com/golang/go/wiki/CodeReviewComments.  
 원본 마지막 편집일 : 2021.09.09 by 이안 랜스 테일러 Ian Lance Taylor.

이 페이지는 Go 코드리뷰 동안 만들어진 코멘트들을 모아놓은것입니다. 각 디테일한 설명들은 속기로 이루어졌기 때문에 스타일 가이드라기 보다는 자주 저지르는 실수들의 리스트입니다.
이 페이지는 [Effective Go](https://go.dev/doc/effective_go)의 부록이라고 할 수 있습니다.

- [Gofmt](#gofmt)
- [Comment Sentences](#commentsentences)
- [Contexts](#contexts)
- [Copying](#copying)
- [Crypto Rand](#cryptorand)
- [Declaring Empty Slices](#declaringemptyslices)
- [Doc Comments](#doccomments)
- [Don't Panic](#don'tpanic)
- [Error Strings](#errorstrings)
- [Examples](#examples)
- [Goroutine Lifetimes](#goroutinelifetimes)
- [Handle Errors](#handleerrors)
- [Imports](#imports)
- [Import Blank](#importblank)
- [Import Dot](#importdot)
- [In-Band Errors](#inbanderrors)
- Indent Error Flow
- Initialisms
- Interfaces
- Line Length
- Mixed Caps
- Named Result Parameters
- Naked Returns
- Package Comments
- Package Names
- Pass Values
- Receiver Names
- Receiver Type
- Synchronous Functions
- Useful Test Failures
- Variable Names

    <a name="gofmt">
      <h2>Gofmt</h2>
      gofmt 명령어를 실행하면 기계적 스타일 이슈 대부분이 자동으로 해결됩니다. 실무의 거의 모든 Go 코드에서 gofmt는 사용됩니다. 문서의 나머지에서는 비 기계적이슈에 대해서만 다루게 됩니다. 
    </a>

    <a name="commentsentences">
      <h2>Comment Sentences</h2> 
      <https://golang.org/doc/effective_go.html#commentary>를 참조하세요. 선언부에 대한 주석은 중복이 있다 하더라도 완성된 문장이어야 합니다. 이러한 접근은 잘 포맷된 형태로 godoc documentation으로 추출될 수 있도록 합니다. 주석은 설명해야 할 대상의 명칭으로 시작해서 마침표로 끝을 맺어야 합니다:

        // Request 는 명령어를 실행하라는 요청을 나타냅니다.
        type <font color="red">Request</font> struct { ...

        // Encode 는 req의 JSON 인코딩을 w에 씁니다.
        func <font color="red">Encode</font>(w io.Writer, req *Request) { ...

    </a>

    <a name="contexts"> 
      <h2>Contexts</h2>
      context.Context는 API와 프로세스 사이에서 보안자격증명(security credentials), 트래킹 정보(tracing information), 데드라인, 취소시그날 등을 전달하는데   
      Go 프로그램은 컨텍스를 RPC 또는 HTTP 리퀘스트로부터 최종 외부로 나가는 리퀘스트까지 전체 펑션 체인을 따라가며 명시적으로 전달한다.   
      이 때 Context를 사용하는 대부분의 펑션은 Context를 첫번째 파라미터로 전달한다:

        func F(ctx context.Context, /* other arguments */) {}

      컨텍스를 직접 생성할 때에는 context.Background()를 사용할 수 있는데 이 때 리턴값으로 Context와 함께 err을 같이 전달하여 사용하는 것이 좋다.
      그러나 기본은 컨텍스를 생성하는 것이 아니라 전달하는 것이다. 그러므로 context.Background()를 사용할 때에는 충분한 이유가 있어야 한다.

      struct 타입일 경우에는 컨텍스트를 멤버로 직접 추가하지 말고 필요한 메소드에 컨텍스트 타입으로 ctx 파라미터를 추가하여 사용해야 한다. 그러나
      메쏘드의 [시그니처](https://developer.mozilla.org/ko/docs/Glossary/Signature/Function)가 스탠다드 라이브러리 또는 써드파티 라이브러리의 인터페이스와 매칭이 필요한 경우에는 예외로 한다.

      사용자 정의 Context 타입을 생성하거나 함수시그니처에서 컨텍스트 이외의 인터페이스를 사용해서는 안된다.
      Don't create custom Context types or use interfaces other than Context in function signatures.

      전달해야 하는 데이타가 있을때 파마미터, 리시버(receiver) 또는 글로벌 변수(golbals)에 넣을 수 있고 필요에 따라 컨텍스트에 넣어 전달 할 수 있다.
      If you have application data to pass around, put it in a parameter, in the receiver, in globals, or, if it truly belongs there, in a Context value.

      컨텍스트는 불변(immutable) 하기 때문에 컨텍스트를 다수의 콜에 전달해서 사용하기에 좋다. 각 콜들은 동일한 데드라인, 취소 시그날, 자격증명, 상위 트래킹 정보(parent trace)를 공유한다.

    </a>

    <a name="copying">
      <h2>Copying</h2>
      다른 패키지의 스트럭처를 복사할때 원치않는 [알리아싱](https://seonggyu.tistory.com/24)이 발생하지 않도록 조심해야 한다. 예를들어 bytes.Buffer 타입은 []byte slice를 포함하고 있는데
      Buffer를 복사 하게 되면 복사된 Buffer의 slice는 원본에 있는 배열을 알리아싱하게 될 수도 있다. 이럴 경우 생각지 못한 결과를 메쏘드 콜을 발생시킬 수도 있다. 
      To avoid unexpected aliasing, be careful when copying a struct from another package. For example, the bytes.Buffer type contains a []byte slice. If you copy a Buffer, the slice in the copy may alias the array in the original, causing subsequent method calls to have surprising effects.

  T라는 값이 있을때 메소드가 포인터 *T와 연계되어 있다면 일반적으로 T의 값을 복사하지 않는다.
  In general, do not copy a value of type T if its methods are associated with the pointer type, *T.

    </a>
    <a name="cryptorand">
      <h2>Crypto Rand</h2>
      한번 쓰고 버리는 키라고 하여도 키 생성을 위해서 **math/rand**를 사용하지 마라. 키 생성을 위해 시드(seed) 값이 없는 경우 제너레이터는 완벽하게 예측가능하다. time.Nanoseconds()로 시드(seed)를 사용하는 경우, 약간의 엔트로피(불확실) 비트들이 존재한다. 대신에 crypto/rand 의 Reader를 사용해라 텍스트가 필요하면 hexadecimal 이나 based64에 프린트해라:

        import (
          "crypto/rand"
          // "encoding/base64"
          // "encoding/hex"
          "fmt"
        )

        func Key() string {
          buf := make([]byte, 16)
          _, err := rand.Read(buf)
          if err != nil {
            panic(err)  // out of randomness, should never happen
          }
          return fmt.Sprintf("%x", buf)
          // or hex.EncodeToString(buf)
          // or base64.StdEncoding.EncodeToString(buf)
        }

    </a>
    
    <a name="#declaringemptyslices">
      <h2>Declaring Empty Slices</h2>
      빈 슬라이스를 선언할 때 위의 형식을 아래 형식보다 선호한다.
        var t []string
      
        t := []string{}
      
      전자의 경우 nil로 초기화 되고 후자의 경우 nil이 아닌 길이가 0인 슬랑이스로 초기화된다. 기능적으로는 len 과 cap 이 모두 0으로 동일하지만 nil 슬라이스가 좀 더 선호된다. 
     
      그러나 상황에 따라 nil이 아닌 길이가 0인 슬라이스가 더 선호되는 경우가 있는데 JSON 오브젝트로 엔코딩하는 경우이다. 이 경우 nil 슬라이스는 null로 변환되고 길이가 0인 슬라이스는 JSON array []로 변환된다.

      인터페이스를 디자인할때 nil 슬라이스와 non-nil 길이가 0인 슬라이스간의 차이로 미묘한 프로그램 에러가 발생할 수 있기 때문에 조심해야 한다. Go에서 nil에 대해 좀 더 알고 싶다면 Francesc Campoy's talk [Understanding Nil](https://youtu.be/ynoY2xz-F8s)을 확인해라

    </a>

    <a name="doccomments">
      <h2>Doc Comments</h2>
    	모든 탑레벨 익스포트에는 문서 주석이 있어야 한다. 사소하지 않은 unexported 타입이나 함수 선언도 마찬가지이다. 주석 규약에 대한 좀 더 많은 정보를 위해서 <https://golang.org/doc/effective_go.html#commentary>를 참조할 수 있다.
  	</a>
  	<a name="#don'tpanic">
  		<h2>Don't Panic</h2>
  		<https://golang.org/doc/effective_go.html#errors>를 참조해라. 일반적인 에러 핸들링을 위해 패닉을 사용하지말고 반환값으로 에러를 사용해라
  	</a>
  	<a name="errorstrings">
  		<h2>Error Strings</h2>
  		에러 메세지는 고유명사나 축약어가 아니라면 첫 글자를 대문자를 시작하거나 마침표로 끝나면 안된다. fmt.Errorf("Something bad") 가 아니라 fmt.Errorf("something bad")를 사용해야 log.Printf("Reading %s: %v", filename, err)형식으로 나타내는 경우 문장 중간에 대문자가 나타나지 않게 된다. 이것은 로깅시에는 적용되지 않는데 로깅은 암묵적으로 라인단위로 표시되고 내부적으로 다른메세지와 연결되지 않는다.
  	</a>
    <a name="examples">
      <h2>Examples</h2>
      새로운 패키지를 추가할 때 사용방법에 대한 실행 예제나 간단한 테스트용 데모 예제를 포함해라.
      [testable Example() functions](https://go.dev/blog/examples)를 참조해라.
    </a>
    <a name="goroutinelifetimes">
      <h2>Goroutine Lifetimes</h2>
  When you spawn goroutines, make it clear when - or whether - they exit.

      고루틴은 채널 전송 및 수신을 블러킹 됨으로써 메모리릭이 발생할 수 있다. 채널이 끊긴 상태로 블럭되었어도 가비지컬렉션은
      터미네이션 시키지 못한다. 메모리릭이 발생하지 않았다해도 더이상 사용되지 않음에도 불구하고 사용중인 상태로 남겨놓음으로써 원인을 찾기 어려운 미묘한 문제들을 발생시킬 수 있다. 종료된 채널에는 패닉을 보내라 여전히 사용중인 입력을 "결과가 필요가 없어졌어" 로 수정하는 것은 여전히 [데이타 레이스](https://namu.wiki/w/%EA%B2%BD%EC%9F%81%20%EC%83%81%ED%83%9C?from=%EB%A0%88%EC%9D%B4%EC%8A%A4%20%EC%BB%A8%EB%94%94%EC%85%98)를 발생시킬 수 있다. 그리고 고루틴을 진행중인 상태로 오랫동안 두는것도 예측할 수 없는 메모리 사용으로 문제가 될 수 있다.

      동시성 코드는 고루틴 라이프타임이 아주 분명하게 드러날 수 있도록 심플해야 한다. 만약 어렵다면 고루틴 엑싯을 언제 어떻게 할지에 대해 문서화해라

    </a>
    <a name="handleerrors">
      <h2>Handle Errors</h2>
      https://golang.org/doc/effective_go.html#errors를 참조해라. _ 변수를 사용해서 에러를 버리지마라. 함수가 에러를 반환하면 함수가 제대로 수행되고 있는것인지 반드시 체크해라. 에러를 핸들링하고 발생하면 반환을 하고 아주 예외적인 상황에서만 패닉처리해라.
    </a>
    <a name="imports">
      <h2>Imports</h2>
      네이밍 충돌이 발생하지 않았다면 imports 시 리네이밍하는것을 피해라. 잘 네이밍된 패키지명을 변경할 필요는 없다. 
      충돌이 발생한 경우에는 가장 지역적이거나 프로젝트 스페식한 imports를 리네이밍하도록 해라.

      imports를 그룹으로 묶고 각 그룹들 사이는 빈줄로 나누어라. 스탠다드 라이브러리 패키지는 항상 첫번째 그룹에 위치하는데 goimports가 자동으로 실행할것이다.

        package main

        import (
          "fmt"
          "hash/adler32"
          "os"

          "appengine/foo"
          "appengine/user"

          "github.com/foo/bar"
          "rsc.io/goversion/version"
        )

    </a>
    <a name="importblank">
    <h2>Import Blank</h2>
      [부가작업](https://golangbyexample.com/blank-identifier-import-golang/)을 위해 _를 이용한 패키지임포트(_ "pkg")는 메인 패키지에서 또는 테스트에서만 사용해야 한다.
    </a>
    <a name="importdot">
      <h2>Import Dot</h2>
      import . 형태는 테스트시에 유용하다. 순환종속성 때문에 패키지의 일부로 만들어질 수는 없다.

        package foo_test

        import (
          "bar/testutil" // also imports "foo"
          . "foo"
        )

      위와 같은 경우에 테스트 파일은 패키지 foo에 있을 수는 없다. 왜냐하면 foo를 임포트하고 있는 bar/testutil를 사용하고 있기 때문이다. 'import .' 형태를 사용하는 이유는 위의 파일이 foo가 아니고 foo_test 이지만 마치 foo인것처럼 할 수 있기 때문이다. 이런 경우를 제외하고는 import . 을 사용하지 마라. 이름이 어디서 왔는지 모호하게 하여 가독성을 떨어지게 한다.

    </a>
    <a name="inbanderrors">
      <h2>In-Band Errors</h2>
      C나 그와 유사한 언어들에서 에러가 발생했거나 결과값에 문제가 생겼음을 알리기 위해 -1이나 null값을 반환하곤 한다.

        // Lookup returns the value for key or "" if there is no mapping for key.
        func Lookup(key string) string

        // Failing to check for an in-band error value can lead to bugs:
        Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"

      Go에서는 훨씬 나은 해결책이 있는데 다중반환이다. [in-band 에러](https://wiki.sei.cmu.edu/confluence/display/java/ERR52-J.+Avoid+in-band+error+indicators)를 체크하는 대신에 error 나 boolean 을 함께 반환함으로써 결과값이 유효한지 알려줄 수 있다.

        // Lookup returns the value for key or ok=false if there is no mapping for key.
        func Lookup(key string) (value string, ok bool)

      bool 값을 같이 반환함으로써 결과값이 잘 못 사용되는것을 막고

        Parse(Lookup(key))  // compile-time error

      코드를 좀 더 가독성있고 견과하게 만들어준다.

        value, ok := Lookup(key)
        if !ok {
          return fmt.Errorf("no value for %q", key)
        }
        return Parse(value)


      Return values like nil, "", 0, and -1 are fine when they are valid results for a function, that is, when the caller need not handle them differently from other values.

      Some standard library functions, like those in package "strings", return in-band error values. This greatly simplifies string-manipulation code at the cost of requiring more diligence from the programmer. In general, Go code should return additional values for errors.

    </a>
