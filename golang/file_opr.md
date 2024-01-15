## filepath.Glob


    filepath.Glob 是 Go 语言标准库中的函数，用于执行文件名模式匹配和查找符合指定模式的文件或目录。它的主要用途是搜索文件系统中的文件或目录，以满足通配符模式。


```go
// 使用 filepath.Glob 查找文件
matches, err := filepath.Glob("/path/to/files/*.txt")
if err != nil {
    // 处理错误
    fmt.Println(err)
} else {
    // 处理匹配到的文件
    for _, match := range matches {
        fmt.Println(match)
    }
}
```

