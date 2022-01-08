---
title: "Go 코드리뷰 커멘트"
date: 2021-01-03T21:17:30-04:00
categories:
  - blog
tags:
  - Go 고
  - 코드리뷰
---

원본 : https://github.com/golang/go/wiki/CodeReviewComments
원본 마지막 편집일 : 2021.09.09 by 이안 랜스 테일러(an Lance Taylor)

이 페이지는 Go 코드리뷰 동안 만들어진 코멘트들을 모아놓은것입니다. 각 디테일한 설명들은 속기로 이루어졌기 때문에 스타일 가이드라기 보다는 자주 발생하는 실수들의 리스트입니다. 
이 페이지는 [Effective Go](https://go.dev/doc/effective_go)의 부록이라고 할 수 있습니다.

 [Gofmt](#gofmt)
Gofmt
Comment Sentences
Contexts
Copying
Crypto Rand
Declaring Empty Slices
Doc Comments
Don't Panic
Error Strings
Examples
Goroutine Lifetimes
Handle Errors
Imports
Import Blank
Import Dot
In-Band Errors
Indent Error Flow
Initialisms
Interfaces
Line Length
Mixed Caps
Named Result Parameters
Naked Returns
Package Comments
Package Names
Pass Values
Receiver Names
Receiver Type
Synchronous Functions
Useful Test Failures
Variable Names



<a name="gofmt">
  <h2>Gofmt</h2>
  Run gofmt on your code to automatically fix the majority of mechanical style issues. Almost all Go code in the wild uses gofmt. The rest of this document     addresses non-mechanical style points.
</a>


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
