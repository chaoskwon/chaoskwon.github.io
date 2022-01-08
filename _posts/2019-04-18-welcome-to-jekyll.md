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

You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

이 페이지는 Go 코드리뷰 동안 만들어진 코멘트들을 모아놓은것입니다. 하나하나의 설명들은 속기로 이루어졌기 때문에 스타일 가이드라기 보다는 자주 발생하는 실수들의 리스트입니다. 
이 페이지는 <a href='https://go.dev/doc/effective_go'>Effective Go</a> 의 부록이라고 할 수 있다.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

```ruby
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
