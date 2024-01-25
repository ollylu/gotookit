## 获取系统信号


```go
term := make(chan os.Signal, 1)
signal.Notify(term, os.Interrupt, syscall.SIGTERM)

hup := make(chan os.Signal, 1)
signal.Notify(hup, syscall.SIGHUP)

for {
	select {
	case <-term:
		'''
		'''
	case <-hup:
		''''
		''''
	}
}
```
