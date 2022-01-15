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

        <font color="red">// Request 는 명령어를 실행하라는 요청을 나타냅니다.</font>
        type <font color="red">Request</font> struct { ...

        <font color="red">// Encode 는 req의 JSON 인코딩을 w에 씁니다.</font>
        func <font color="red">Encode</font>(w io.Writer, req *Request) { ...

    </a>

    <a name="contexts"> 
      <h2>Contexts</h2>
      context.Context는 API와 프로세스간의 보안자격증명(security credentials), 트레킹 정보(tracing information), 데드라인, 취소시그날 등을 전달합니다. Go 프로그램들은 인커밍하는 RPC 또는 HTTP 리퀘스트로터 컨텍스트(Contexts)들을 펑션콜 체인을 따라 최종 외부로 나가는 리퀘스트에 명시적으로 전달한다.

      Context를 사용하는 대부분의 펑션은 Context를 첫번째 파라미터로 전달한다:

        func F(<font color="red">ctx context.Context</font>, /* other arguments */) {}

      request를 사용하지 않는 펑션이라면 Context를 전달할때 err도 같이 전달하여(err은 필요하지 않떠라도) context.Background()을 바로 사용할 수 있습니다.
      Context를 전달하는 디폴트 케이스:
      A function that is never request-specific may use context.Background(), but err on the side of passing a Context even if you think you don't need to. The default case is to pass a Context; only use context.Background() directly if you have a good reason why the alternative is a mistake.

      struct 타입에는 Context를 멤버로 직접 추가하지 말고 대신에 메소드에 ctx 파라미터를 추가하여 사용하세요. 유일한 예외가 있는데 이는 메쏘드의 [시그니처](https://blog.shovelman.dev/330)가 스탠다드 라이브러리 또는 써드파티 라이브러리(third party library)의 인터페이스와 매칭되어야 하는 경우입니다.

      Don't add a Context member to a struct type; instead add a ctx parameter to each method on that type that needs to pass it along. The one exception is for methods whose signature must match an interface in the standard library or in a third party library.

      사용자 정의 Context 타입을 생성하거나 함수시그니처에서 컨텍스트 이외의 인터페이스를 사용하지 마십시오.
      Don't create custom Context types or use interfaces other than Context in function signatures.

      전달해야 하는 application 데이타가 있다면 파라미터로 만들어서 리시버(receiver)에 글로벌(globals)에 또는
      If you have application data to pass around, put it in a parameter, in the receiver, in globals, or, if it truly belongs there, in a Context value.

      Contexts are immutable, so it's fine to pass the same ctx to multiple calls that share the same deadline, cancellation signal, credentials, parent trace, etc.

    </a>
    
    <a name="copying>
      <h2>Copying</h2>
      To avoid unexpected aliasing, be careful when copying a struct from another package. For example, the bytes.Buffer type contains a []byte slice. If you copy a Buffer, the slice in the copy may alias the array in the original, causing subsequent method calls to have surprising effects.

      In general, do not copy a value of type T if its methods are associated with the pointer type, *T.

    </a>
