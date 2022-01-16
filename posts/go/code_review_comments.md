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
- Examples
- Goroutine Lifetimes
- Handle Errors
- Imports
- Import Blank
- Import Dot
- In-Band Errors
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
		
		Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation, since they are usually printed following other context. That is, use fmt.Errorf("something bad") not fmt.Errorf("Something bad"), so that log.Printf("Reading %s: %v", filename, err) formats without a spurious capital letter mid-message. This does not apply to logging, which is implicitly line-oriented and not combined inside other messages.
	</a>
ExamplesWhen adding a new package, include examples of intended usage: a runnable Example, or a simple test demonstrating a complete call sequence.

Read more about testable Example() functions.

Goroutine LifetimesWhen you spawn goroutines, make it clear when - or whether - they exit.

Goroutines can leak by blocking on channel sends or receives: the garbage collector will not terminate a goroutine even if the channels it is blocked on are unreachable.

Even when goroutines do not leak, leaving them in-flight when they are no longer needed can cause other subtle and hard-to-diagnose problems. Sends on closed channels panic. Modifying still-in-use inputs "after the result isn't needed" can still lead to data races. And leaving goroutines in-flight for arbitrarily long can lead to unpredictable memory usage.

Try to keep concurrent code simple enough that goroutine lifetimes are obvious. If that just isn't feasible, document when and why the goroutines exit.

Handle ErrorsSee https://golang.org/doc/effective_go.html#errors. Do not discard errors using _ variables. If a function returns an error, check it to make sure the function succeeded. Handle the error, return it, or, in truly exceptional situations, panic.

ImportsAvoid renaming imports except to avoid a name collision; good package names should not require renaming. In the event of collision, prefer to rename the most local or project-specific import.

Imports are organized in groups, with blank lines between them. The standard library packages are always in the first group.

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

goimports will do this for you.

Import BlankPackages that are imported only for their side effects (using the syntax import _ "pkg") should only be imported in the main package of a program, or in tests that require them.

Import DotThe import . form can be useful in tests that, due to circular dependencies, cannot be made part of the package being tested:

package foo_test

import (
	"bar/testutil" // also imports "foo"
	. "foo"
)

In this case, the test file cannot be in package foo because it uses bar/testutil, which imports foo. So we use the 'import .' form to let the file pretend to be part of package foo even though it is not. Except for this one case, do not use import . in your programs. It makes the programs much harder to read because it is unclear whether a name like Quux is a top-level identifier in the current package or in an imported package.

In-Band ErrorsIn C and similar languages, it's common for functions to return values like -1 or null to signal errors or missing results:

// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"

Go's support for multiple return values provides a better solution. Instead of requiring clients to check for an in-band error value, a function should return an additional value to indicate whether its other return values are valid. This return value may be an error, or a boolean when no explanation is needed. It should be the final return value.

// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)

This prevents the caller from using the result incorrectly:

Parse(Lookup(key))  // compile-time error

And encourages more robust and readable code:

value, ok := Lookup(key)
if !ok {
	return fmt.Errorf("no value for %q", key)
}
return Parse(value)

This rule applies to exported functions but is also useful for unexported functions.

Return values like nil, "", 0, and -1 are fine when they are valid results for a function, that is, when the caller need not handle them differently from other values.

Some standard library functions, like those in package "strings", return in-band error values. This greatly simplifies string-manipulation code at the cost of requiring more diligence from the programmer. In general, Go code should return additional values for errors.
