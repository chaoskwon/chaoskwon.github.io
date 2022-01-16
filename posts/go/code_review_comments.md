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
- Crypto Rand
- Declaring Empty Slices
- Doc Comments
- Don't Panic
- Error Strings
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
