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

## Handle files 

### Opening and closing 

Basic funcs: 
```go
func OpenFile(name string, flag int, perm FileMode) (*File, error)
// Open wraps OpenFile (read-only)
func Open(name string) (*File, error) {
	return OpenFile(name, O_RDONLY, 0)
}
```
About `OpenFile()`: 

Example: 
```go
func OpenCloseFile() {
	file, err := os.Open("filename.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer file.Close()
	fmt.Printf("Variable type: %+T\n", file)
	fmt.Printf("Value: %+v\n", *file)
	x1 := reflect.ValueOf(file).Elem()
	fmt.Printf("Type 2: %s\n", x1)
}
```

### Flags 

Also see: [Open(2): Linux man page](https://linux.die.net/man/2/open)

The `os` package provides constants wrapping the OS flags: 

```go
const (
	O_RDONLY int = syscall.O_RDONLY // открыть файл только для чтения.
	O_WRONLY int = syscall.O_WRONLY // открыть файл только для записи.
	O_RDWR   int = syscall.O_RDWR   // открыть файл только для чтения и записи.
	O_APPEND int = syscall.O_APPEND // добавлять данные в файл при записи.
	O_CREATE int = syscall.O_CREAT  // создать новый файл, если его не существует.
	O_EXCL   int = syscall.O_EXCL   // используется с O_CREATE, открытие завершится с ошибкой, если файл существует
	O_SYNC   int = syscall.O_SYNC   // открыть для синхронного ввода-вывода
	O_TRUNC  int = syscall.O_TRUNC  // обрезать при открытии файл, доступный для записи
)
```

### Creating, renaming, deleting files 

Use funcs: `os.Create()`, `os.Remove()`, and `os.Rename()`:

`Create`: creates or cuts (if already exists) the said file. If creating, create a file with FileMode=0666 (read & write).
* If successful: the fun returns a `File` with the `O-RDWR` descriptor (can read & write).
* All errors: `*PathError`.
```go
func Create(name string) (*File, error) {
	return OpenFile(name, O_RDWR|O_CREATE|O_TRUNC, 0666)
}
```

`os.Remove` signature: `func Remove(name string) error`: deletes the file or return a `*PathError` error. 
`Rename()` signature: `func Rename(oldpath, newpath string) error`: moves the file. If `newpath` already exists, the func will overwrite it,

### Reading files 

`os.ReadFile()`, `io.ReadAll()`, `bufio.NewScanner()`, `os.File.Read()`

Basic funcs: 
```go
```
Example: 
```go
```

Basic funcs: 
```go
```
Example: 
```go
```

Basic funcs: 
```go
```
Example: 
```go
```

Basic funcs: 
```go
```
Example: 
```go
```
### Writing to file 

Basic funcs: 
```go
```
Example: 
```go
```

### Choosing packages 
Packages to choose from: `io`, `bufio`, `os`. 

Basic funcs: 
```go
```
Example: 
```go
```

## Task 1
Напишите программу для подсчета количества слов в стандартном потоке ввода и последующего их вывода на экран с символом переноса строки. Ввод-вывод осуществляйте с использованием пакета bufio.

## Task 2
Напишите программу для подсчета общей стоимости заказа. Данные о купленных товарах находятся в файле order.txt в формате "товар-цена". Размер файла может быть довольно большим, поэтому не стоит считывать его целиком.



