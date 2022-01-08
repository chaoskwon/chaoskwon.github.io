---
title: "Go 코드리뷰 커멘트"
date: 2021-01-03T21:17:30-04:00
categories:
  - blog
tags:
  - Go 고
  - 코드리뷰
---

이 페이지는 아래 원본 사이트의 번역본입니다. 
원본 : https://github.com/golang/go/wiki/CodeReviewComments
원본 마지막 편집일 : 2021.09.09 by 이안 랜스 테일러(an Lance Taylor)

이 페이지는 Go 코드리뷰 동안 만들어진 코멘트들을 모아놓은것입니다. 각 디테일한 설명들은 속기로 이루어졌기 때문에 스타일 가이드라기 보다는 자주 발생하는 실수들의 리스트입니다. 
이 페이지는 [Effective Go](https://go.dev/doc/effective_go)의 부록이라고 할 수 있습니다.

* [Gofmt](#gofmt)
* [Comment Sentences](#commentsentences)
* Contexts
* Copying
* Crypto Rand
* Declaring Empty Slices
* Doc Comments
* Don't Panic
* Error Strings
* Examples
* Goroutine Lifetimes
* Handle Errors
* Imports
* Import Blank
* Import Dot
* In-Band Errors
* Indent Error Flow
* Initialisms
* Interfaces
* Line Length
* Mixed Caps
* Named Result Parameters
* Naked Returns
* Package Comments
* Package Names
* Pass Values
* Receiver Names
* Receiver Type
* Synchronous Functions
* Useful Test Failures
* Variable Names


<a name="gofmt">
  <h2>Gofmt</h2>
  gofmt 명령어를 실행하면 기계적 스타일 이슈 대부분이 자동으로 해결됩니다. 실무의 거의 모든 Go 코드에서 gofmt는 사용됩니다. 문서의 나머지에서는 비 기계적이슈에 대해서만 다루게 됩니다. 
</a>

<a name="#commentsentences">
  <h2>Comment Sentences</h2>
  See https://golang.org/doc/effective_go.html#commentary. Comments documenting declarations should be full sentences, even if that seems a little redundant. This approach makes them format well when extracted into godoc documentation. Comments should begin with the name of the thing being described and end in a period:

  // Request represents a request to run a command.
  type Request struct { ...

  // Encode writes the JSON encoding of req to w.
  func Encode(w io.Writer, req *Request) { ...
  and so on.
  
</a>

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
