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
- [Indent Error Flow](#indenterrorflow)
- [Initialisms](#initialisms)
- [Interfaces](#interfaces)
- [Line Length](#linelength)
- [Mixed Caps](#mixedcaps)
- [Named Result Parameters](#namedresultparameters)
- [Naked Returns](#nakedreturns)
- [Package Comments](#packagecomments)
- [Package Names](#packagenames)
- [Pass Values](#passvalues)
- [Receiver Names](#receivernames)
- [Receiver Type](#receivertype)
- [Synchronous Functions](#synchronousfunctions)
- [Useful Test Failures](#usefultestfailures)
- [Variable Names](#variablenames)

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

    Go에서는 훨씬 나은 해결책이 있는데 다중반환이다. [in-band 에러](https://wiki.sei.cmu.edu/confluence/display/java/ERR52-J.+Avoid+in-band+error+indicators)를 체크하는 대신에 error 나 bool을 함께 반환함으로써 결과값이 유효한지 알려줄 수 있다.

      // Lookup returns the value for key or ok=false if there is no mapping for key.
      func Lookup(key string) (value string, ok bool)

      Parse(Lookup(key))  // compile-time error

    bool 값이 같이 반환함으로써 결과값이 잘 못 사용되는것을 막고 코드를 좀 더 가독성있고 견고하게 만들어준다.

      value, ok := Lookup(key)
      if !ok {
        return fmt.Errorf("no value for %q", key)
      }
      return Parse(value)

    함수에 대한 nil, "", 0, -1과 같은 반환값이 유효하게 사용된다면 문제가 없다. 즉 콜러는 이 값들을 다른 값들과 별도로 취급 할 필요는 없다.
    Return values like nil, "", 0, and -1 are fine when they are valid results for a function, that is, when the caller need not handle them differently from other values.

    몇몇 스탠다드 라이브러리 펑션들 예를들어 "strings"와 같은 패키지는 이러한 인밴드에러(in-band error) 값을 반환한다. 이방식은 string을 사용하는 코드를 단순화시킨다. 프로그래머로부터 많은 deligence를 댓가로 요구한다. 일반적으로 Go 코드는
    에러 처리를 위한 추가적인 값은 반환한다.

  </a>
  <a name="Indent Error Flow">
    <h2>Indent Error Flow</h2>
    최소한의 들여쓰기로 코드를 유지하려고 노력해야 한다. 에러 핸들링의 경우는 들여쓰기를 하여 정상적인 코드들의 흐름을 빠르게 스캐닝할 수 있도록 하여 가독성을 향상 시킬 수 있다. 
      if err != nil {
        // error handling
      } else {
        // normal code
      }
    위의 코드 대신에, 아래와 같이 작성해라
      if err != nil {
        // error handling
        return // or continue, etc.
      }
      // normal code

    만약 아래와 같이 변수 선언 및 할당이 이루어지는 경우에는
      if x, err := f(); err != nil {
        // error handling
        return
      } else {
        // use x
      }

    아래와 같이 변수 선언 및 할당을 라인을 분리하여 짧게 진행할 수 있다.
      x, err := f()
      if err != nil {
        // error handling
        return
      }
      // use x

  </a>
  <a name="initialisms">
    <h2>Initialisms</h2>
    예를들어 URL 또는 NATO 과 같은 이니셜이나 축약어는 대소문자를 일관되게 사용해야 한다. 예를 들어 URL의 경우 "urlPony" 또는 "URLPony" 와 같이 "URL" 또는 "url"로 사용해야 하고 "Url"과 같이는 사용되지 말아야 한다. 추가로 "ServeHttp" 가 아니라 "ServeHTTP" 연결된 단어에서는 "xmlHTTPRequest" 나 "XMLHTTPRequest" 로 사용되어야 한다.

    이 룰은 "identifiers"의 약자인 "ID"에도 적용된다. "appId" 가 아니라 "appID"를 사용해라.

  </a>
  <a name="interfaces">
    <h2>Interfaces</h2>
    Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.

    Do not define interfaces on the implementor side of an API "for mocking"; instead, design the API so that it can be tested using the public API of the real implementation.

    Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.

      package consumer  // consumer.go

      type Thinger interface { Thing() bool }

      func Foo(t Thinger) string { … }

      package consumer // consumer_test.go

      type fakeThinger struct{ … }
      func (t fakeThinger) Thing() bool { … }
      …
      if Foo(fakeThinger{…}) == "x" { … }

      // DO NOT DO IT!!!
      package producer

      type Thinger interface { Thing() bool }

      type defaultThinger struct{ … }
      func (t defaultThinger) Thing() bool { … }

      func NewThinger() Thinger { return defaultThinger{ … } }

    Instead return a concrete type and let the consumer mock the producer implementation.

      package producer

      type Thinger struct{ … }
      func (t Thinger) Thing() bool { … }

      func NewThinger() Thinger { return Thinger{ … } }

  </a>  
  <a name="linelength">
    <h2>Line Length</h2>
    There is no rigid line length limit in Go code, but avoid uncomfortably long lines. Similarly, don't add line breaks to keep lines short when they are more readable long--for example, if they are repetitive.

Most of the time when people wrap lines "unnaturally" (in the middle of function calls or function declarations, more or less, say, though some exceptions are around), the wrapping would be unnecessary if they had a reasonable number of parameters and reasonably short variable names. Long lines seem to go with long names, and getting rid of the long names helps a lot.

In other words, break lines because of the semantics of what you're writing (as a general rule) and not because of the length of the line. If you find that this produces lines that are too long, then change the names or the semantics and you'll probably get a good result.

This is, actually, exactly the same advice about how long a function should be. There's no rule "never have a function more than N lines long", but there is definitely such a thing as too long of a function, and of too repetitive tiny functions, and the solution is to change where the function boundaries are, not to start counting lines.
</a>  
 <a name="mixedcaps">

<h2>Mixed Caps</h2>
See https://golang.org/doc/effective_go.html#mixed-caps. This applies even when it breaks conventions in other languages. For example an unexported constant is maxLength not MaxLength or MAX_LENGTH.

Also see Initialisms.
</a>
<a name="namedresultparameters">

<h2>Named Result Parameters</h2>
Consider what it will look like in godoc. Named result parameters like:

func (n *Node) Parent1() (node *Node) {}
func (n *Node) Parent2() (node *Node, err error) {}
will be repetitive in godoc; better to use:

func (n *Node) Parent1() *Node {}
func (n *Node) Parent2() (*Node, error) {}
On the other hand, if a function returns two or three parameters of the same type, or if the meaning of a result isn't clear from context, adding names may be useful in some contexts. Don't name result parameters just to avoid declaring a var inside the function; that trades off a minor implementation brevity at the cost of unnecessary API verbosity.

func (f \*Foo) Location() (float64, float64, error)
is less clear than:

// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f \*Foo) Location() (lat, long float64, err error)
Naked returns are okay if the function is a handful of lines. Once it's a medium sized function, be explicit with your return values. Corollary: it's not worth it to name result parameters just because it enables you to use naked returns. Clarity of docs is always more important than saving a line or two in your function.

Finally, in some cases you need to name a result parameter in order to change it in a deferred closure. That is always OK.
</a>  
 <a name="nakedreturns">

<h2>Naked Returns</h2>
A return statement without arguments returns the named return values. This is known as a "naked" return.

func split(sum int) (x, y int) {
x = sum \* 4 / 9
y = sum - x
return
}
See Named Result Parameters.

  </a>    
  <a name="packagecomments">
    <h2>Package Comments</h2>
    Package comments, like all comments to be presented by godoc, must appear adjacent to the package clause, with no blank line.

// Package math provides basic constants and mathematical functions.
package math
/_
Package template implements data-driven templates for generating textual
output such as HTML.
....
_/
package template
For "package main" comments, other styles of comment are fine after the binary name (and it may be capitalized if it comes first), For example, for a package main in the directory seedgen you could write:

// Binary seedgen ...
package main
or

// Command seedgen ...
package main
or

// Program seedgen ...
package main
or

// The seedgen command ...
package main
or

// The seedgen program ...
package main
or

// Seedgen ..
package main
These are examples, and sensible variants of these are acceptable.

Note that starting the sentence with a lower-case word is not among the acceptable options for package comments, as these are publicly-visible and should be written in proper English, including capitalizing the first word of the sentence. When the binary name is the first word, capitalizing it is required even though it does not strictly match the spelling of the command-line invocation.

See https://golang.org/doc/effective_go.html#commentary for more information about commentary conventions.
</a>
<a name="packagenames">

<h2>Package Names</h2>
All references to names in your package will be done using the package name, so you can omit that name from the identifiers. For example, if you are in package chubby, you don't need type ChubbyFile, which clients will write as chubby.ChubbyFile. Instead, name the type File, which clients will write as chubby.File. Avoid meaningless package names like util, common, misc, api, types, and interfaces. See http://golang.org/doc/effective_go.html#package-names and http://blog.golang.org/package-names for more.
</a>  
 <a name="passvalues">
<h2>Pass Values</h2>
Don't pass pointers as function arguments just to save a few bytes. If a function refers to its argument x only as *x throughout, then the argument shouldn't be a pointer. Common instances of this include passing a pointer to a string (*string) or a pointer to an interface value (\*io.Reader). In both cases the value itself is a fixed size and can be passed directly. This advice does not apply to large structs, or even small structs that might grow.
</a>  
 <a name="receivernames">
<h2>Receiver Names</h2>
The name of a method's receiver should be a reflection of its identity; often a one or two letter abbreviation of its type suffices (such as "c" or "cl" for "Client"). Don't use generic names such as "me", "this" or "self", identifiers typical of object-oriented languages that gives the method a special meaning. In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.
</a>  
 <a name="receivertype">
    <h2>Receiver Type</h2>
    메쏘드에 대한 리시버를 값으로 전달할지 아니면 포인터로 전달할지 선택하는 것은 쉽지 않은 문제이다. 만약 잘 모르겠다면 포인터를 사용해라. 그러나 기본 데이타 타입이거나 값이 변하지 않는 소규모 스트럭트(struct)인 경우에는 효율성 측면에서 밸류리시버(value receiver)가 더 나은 선택이 될 수 있다.

    * 리시버가 맵(map), 펑션(func), 채널(chan) 또는 값의 변경이 없는 슬라이스인 경우에는 포인터를 사용하지 마라.
    * 그러나 리시버의 변경이 필요한 경우 반드시 포인터를 사용해야 한다.
    * 리시버가 sync.Mutex나 유사한 싱크로나이즈 필드를 가진 스트럭트(struct)안 경우 반드시 포인트를 사용해야 한다.
    * 대규모 스트럭트(struct)나 배열(array)인 경우 포인터 리서버(point receive)가 효율적이다. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
    * Can function or methods, either concurrently or when called from this method, be mutating the receiver? 밸류(value) 타입인 경우 리시버가 복사가 되기 때문에 메쏘드 밖에서 발생한 리시버의 변경은 메쏘드 안에서는 영향을 주지 않기 때문에 리시버에 발생한 변화들에 대해 알아야 한다면 포인터 타입을 사용해야 한다.
    * If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention clearer to the reader.
    * If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
    * 리시버 타입을 혼용해서 사용하지 말고 하나를 선택해서 관련된 모든 메쏘드에 적용해라

    마지막으로, 헷갈린다면 포인터 리시버를 사용해라.

  </a>  
  <a name="synchronousfunctions">
    <h2>Synchronous Functions</h2>
    Prefer synchronous functions - functions which return their results directly or finish any callbacks or channel ops before returning - over asynchronous ones.

    Synchronous functions keep goroutines localized within a call, making it easier to reason about their lifetimes and avoid leaks and data races. They're also easier to test: the caller can pass an input and check the output without the need for polling or synchronization.

    If callers need more concurrency, they can add it easily by calling the function from a separate goroutine. But it is quite difficult - sometimes impossible - to remove unnecessary concurrency at the caller side.

</a>
<a name="usefultestfailures">

<h2>Useful Test Failures</h2>
  테스트 코드에서 실패 처리시 반드시 어떤 입력값에서 무엇이 잘못되었는지 어떤 값을 기대했는데 실제 어떤값이 들어왔는지 도움이 되는 메세지가 같이 있어야 한다. 많은 assert문을 작성하고자 하겠지만 이 때 반드시 유용한 에러 메세지가 생산되도록 해야한다. 테스트 코드를 실행하는 사람이 당신이나 당신팀이 아니라고 한다면 전형적인 Go 테스트는 아래와 같이 실패 메세지를 나타낼 수 있다.

    if got != tt.want {
      t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
    }

Note that the order here is actual != expected, and the message uses that order too. Some test frameworks encourage writing these backwards: 0 != x, "expected 0, got x", and so on. Go does not.

이것이 많은 타이핑이 필요한것 같다면 테이블 드리븐 테스트를 사용할 수 있다.

또 많이 사용되는 방식은 다양한 테스트 케이스를 가진 테스트 펑션을 만들고 이를 분명한 이름을 가진 펑션으로 감싸서 테스트하면 실패서 이 펑션명으로 실패하게 된다.

    func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
    func TestNoValues(t *testing.T) { testHelper(t, []int{}) }

미래에 당신의 코드를 디버깅하고 있을 누군가에게 실패시 도움이 되는 메세지를 줄지는 전적으로 당신에게 달려있다.

</a>  
 <a name="variablenames">
<h2>Variable Names</h2>
  Go에서 변수명은 짧은편이 좋다. 특히 범위가 작은 로컬 변수인 경우 예를 들면 lineCount 보다는 c를 sliceIndex보다는 i를 사용해라.

기본 룰 : 네이밍이 넓게 사용되면 될 수록 설명적인 네이밍을 사용해야한다. 메쏘드(method)의 반환값을 위해서 한두개의 문자는 불충분하다. 반복문의 색인과 같은 경우는 문자 하나 (i, r)로 사용될 수 있다. 그러나 글로벌 변수 등에는 좀 더 설명적인 네이밍을 해야 한다.
</a>
