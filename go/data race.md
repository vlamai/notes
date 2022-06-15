it's occurs when two goroutines access the same variable concurrently and at least one the accesses is a write.

> [[_articles#^go-memory-model-article-link]]  for details


## usage 

```
$ go test -race mypkg    // to test the package
$ go run -race mysrc.go  // to run the source file
$ go build -race mycmd   // to build the command
$ go install -race mypkg // to install the package
```






#go