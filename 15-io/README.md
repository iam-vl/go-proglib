# Input / output, buffers, file handling

## Input / output

### Reading 
Package `io.Reader`:
```go
type Reader interface {
    // Must return a number of bytes & an io.EOF error
    Read(p []byte) (n int, err error)
}
```
If you know size: `io.Readfull()`:
```go
// Checks if the buffer is loaded, and returns io.ErrUnexpectedEOF
func ReadFull(r Reader, buf []bute) (n int, err error)
```

### Writing 
io.Writer:
```go
type Writer interface {
    // Writes len(p) bytes from p to the buffer
    // Returns bytes it recorded and the err
    // if it read n < len(p) bytes, must return non-nil
    Write(p []byte) (n int, err error)
}
```

### Copying
C data src -> dst before `io.EOF`: `io.Copy()` / `io.CopyN()`
```go
// number of bytes written + err (nil if everything correct)
func Copy(dst Writer, src Reader) (written int64, err error)
// n = max size to copy
// if n == written, err = nil
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```

