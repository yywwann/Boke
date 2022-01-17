
```go
type methodTree struct {
	// http method, eg. GET, POST, PUT...
	method string
	root   *node
}

type methodTrees []methodTree

type node struct {
    path      string
    indices   string
    children  []*node
    handlers  HandlersChain
    priority  uint32
    nType     nodeType
    maxParams uint8
    wildChild bool
    fullPath  string
}


```
