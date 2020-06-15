```javascript
function fetch() {
  return new Promise((resolve, reject) => setTimeout(() => {
    resolve('end');
  }, 2000));
}

fetch().then(param => {
  console.log(param);
  return 'start';
}).then(param => console.log(param));
```