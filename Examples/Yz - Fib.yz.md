Fibonacci

```javascript
//
fibonacci: { n Int 
  n <= 2 ? { return 1 }
  fibonacci(n - 1) + fibonacci(n -2)
}
fibonacci(10)

//
fibonacci: { n Int; current Int; next Int; result Int
  n == 0 ? { return current }
  fibonacci n - 1  next  current + next
}

fibonacci 10  0 1

//
fibonacci: { n Int

  first second: 0 1
  1.to(n).do({ _ Int
    first second = seconnd first
    second = second + first
  })
  first
}

//
main: {
  name: get_line()
  print `Hello $(name)`
}


main: {
  file1 file2 : get_args()
  str:  read_file file1
  write_file(file2).to_string()
}
```


Yz v1.0
```javascript
fib: { n Int
    f s: 0 1
    (n - 1).times {
        f s = s f
        s = s + f
    } 
    f
}

```
