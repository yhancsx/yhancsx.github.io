---
published: false
---

```javascript
function outer() {
  var a = 2;
  return function inner() {
    console.log(a);
  }
}

var func = outer();
func(); // 2
```

https://hyunseob.github.io/2016/08/30/javascript-closure/
https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures
https://github.com/baeharam/Must-Know-About-Frontend/blob/master/Notes/javascript/closure.md
https://meetup.toast.com/posts/86