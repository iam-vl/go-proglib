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
Example:
```go
func ReaderCopy() {
	r := strings.NewReader("Данные для чтения ")
	fmt.Printf("R type: %+T\n", r)
	_, err := io.Copy(os.Stdout, r)
	if err != nil {
		log.Fatal(err)
	}
}
```

Impl:
```go
func NewReader(s string) *Reader { return &Reader{s, 0, -1} }
type Reader struct {
	s        string
	i        int64
	prevRune int
}
```

## Buffered input / output (`bufio`)

Improve perf by using buffers. Package `bufio` wraps `io.Reader` and `io.Writer`. 
Main types: `Reader`, `Writer`, and `Scanner`. 

### Bufio.Reader

```go
Std size buffer (B4K), explicit size buffer
func NewReader(rd io.Reader) *Reader
func NewReaderSize(rd io.Reader, size int) *Reader
// Reads until the first `delim` occurrence and return this string (inclusive)
func (b *Reader) ReadString(delim byte) (string, error)
```

Example: 
```go
	r := bufio.NewReader(os.Stdin)
	str, err := r.ReadString(';')
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", strings.TrimSpace(str))
```

### Bufio.Writer
Bufferizes `io.Writer`
Here:
```go
// Std size (B4K) and custom size writers 
func NewWriter(w io.Writer) *Writer
func NewReaderSize(rd io.Reader, size int) *Reader
//Writes any buffered data to basic interface `io.Writer`
func (b *Writer) Flush() error
```
Separate writers for `string`, `byte`, nad `rune` values:
```go
// Write a string, retn number of bytes written. If bytes < len(s), retns an error
func (b *Writer) WriteString(s string) (int, error)
// Write a byte, return an err if any
func (b *Writer) WriteByte(c byte) error
``` 

Example: 
```go
func BufioWriteStr() {
	w := bufio.NewWriter(os.Stdout)
	r := bufio.NewReader(os.Stdin)
	// Read until the first line break
	name, err := r.ReadString('\n')
	if err != nil {
		log.Fatal(err)
	}
	w.WriteString("Hi, " + name)
	w.Flush()
}
```

### Bufio.Scanner 
Interface for reading data line by line. 
Usually combined with `Scanner.Scan()` (iterated over the datasource token, defined by `SplitFunc` type)

Here:
```go
func (s *Scanner) Scan() bool
// Sets up the splitting function
// Can be used to scan the file by bytes, runes, strings and words (space-separated)
func (s *Scanner) Split(split SplitFunc)
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```

Example: 
```go
sc := bufio.NewScanner(os.Stdin)
for sc.Scan() {
	fmt.Println(sc.Text())
}
err := sc.Err()
if err != nil {
	fmt.Fprintln(os.Stderr, "reading os.Stdin:", err)
}
```
