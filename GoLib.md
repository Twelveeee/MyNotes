https://www.yuque.com/qyuhen/go

# go标准库

# 输入输出

## io

定义 I/O 基本接口，及相关组合。

```
●Reader, Writer, Closer, Seeker
●ReadWriter, ReaderAt, WriterAt, ReaderFrom, WriterTo
●ReadCloser, WriteCloser, ReadWriteCloser
●ByteReader, ByteWriter, StringWriter
```



```go
package main

import (
	"bufio"
	"log"
	"os"
)

func main() {
	f, err := os.Create("./demo.txt")
	if err != nil {
		log.Fatalln(err)
	}

	defer f.Close()
	defer f.Sync()

	w := bufio.NewWriter(f)
	defer w.Flush()

	w.WriteString("hello, world!")
}
```



**源码解析**

### Reader

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

所有实现了 Read 方法的类型都满足 io.Reader 接口，也就是说，在所有需要 io.Reader 的地方，可以传递实现了 Read() 方法的类型的实例。



demo:

```go
func main() {
	// 从标准输入读取
	// data, err = ReadFrom(os.Stdin, 11)

	// 从普通文件读取，其中 file 是 os.File 的实例
	// data, err = ReadFrom(file, 9)

	// 从字符串读取
	data, err := ReadFrom(strings.NewReader("from string"), 12)
	fmt.Println(data, err)
}

func ReadFrom(reader io.Reader, num int) ([]byte, error) {
	p := make([]byte, num)
	n, err := reader.Read(p)
	if n > 0 {
		return p[:n], nil
	}
	return p, err
}
```

ReadFrom 函数将 io.Reader 作为参数，也就是说，ReadFrom 可以从任意的地方读取数据，只要来源实现了 io.Reader 接口。比如，我们可以从标准输入、文件、字符串等读取数据，

### Writer

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

同样的，所有实现了Write方法的类型都实现了 io.Writer 接口



demo: `fmt.Println`

```go
// Fprintln 使用其参数默认格式进行格式化并写入 w.
// 参数之间始终添加空格并附加换行符。
// 它返回写入的字节数以及遇到的任何写入错误。
func Fprintln(w io.Writer, a ...any) (n int, err error) {
	p := newPrinter()
	p.doPrintln(a)
	n, err = w.Write(p.buf)
	p.free()
	return
}

// Println 使用其参数的默认格式进行格式化并写入标准输出。
// 参数之间始终添加空格并附加换行符。
// 它返回写入的字节数以及遇到的任何写入错误。
func Println(a ...any) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```





os.File 同时实现了这io.Reader 接口和 io.Writer 接口。我们还看到 os.Stdin/Stdout 这样的代码，它们似乎分别实现了 io.Reader/io.Writer 接口。没错，实际上在 os 包中有这样的代码：

```go
var (
    Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
    Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
    Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

也就是说，Stdin/Stdout/Stderr 只是三个特殊的文件类型的标识（即都是 os.File 的实例），自然也实现了 io.Reader 和 io.Writer。

- os.File 同时实现了 io.Reader 和 io.Writer
- strings.Reader 实现了 io.Reader
- bufio.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- bytes.Buffer 同时实现了 io.Reader 和 io.Writer
- bytes.Reader 实现了 io.Reader
- compress/gzip.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- crypto/cipher.StreamReader/StreamWriter 分别实现了 io.Reader 和 io.Writer
- crypto/tls.Conn 同时实现了 io.Reader 和 io.Writer
- encoding/csv.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- mime/multipart.Part 实现了 io.Reader
- net/conn 分别实现了 io.Reader 和 io.Writer(Conn接口定义了Read/Write)

除此之外，io 包本身也有这两个接口的实现类型。如：

```
实现了 Reader 的类型：LimitedReader、PipeReader、SectionReader
实现了 Writer 的类型：PipeWriter
```

以上类型中，常用的类型有：os.File、strings.Reader、bufio.Reader/Writer、bytes.Buffer、bytes.Reader

从接口名称很容易猜到，一般地， Go 中接口的命名约定：接口名以 er 结尾。注意，这里并非强行要求，你完全可以不以 er 结尾。标准库中有些接口也不是以 er 结尾的。

### ReaderAt, WriterAt

```go
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}
```

ReaderAt 接口使得可以从指定偏移量处开始读取数据。

demo:

```go
func main() {
	reader := strings.NewReader("01twelve test")
	p := make([]byte, 6)
	n, err := reader.ReadAt(p, 2)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s, %d\n", p, n)
}
// twelve, 6
```





```go
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
```

WriterAt 接口将数据写入到数据流的特定偏移量之后。

demo :

```go
func main() {
	file, err := os.Create("demo.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()
	file.WriteString("Golang__12")
	n, err := file.WriteAt([]byte("twelve"), 8)
	if err != nil {
		panic(err)
	}
	fmt.Println(n)
}
// Golang__twelve
```



### ReaderFrom, WriterTo

```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

ReadFrom 从 r 中读取数据，直到 EOF 或发生错误。其返回值 n 为读取的字节数。除 io.EOF 之外，在读取过程中遇到的任何错误也将被返回。

demo:

```go
func main() {
	file, err := os.Open("demo.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()
	writer := bufio.NewWriter(os.Stdout)
	writer.ReadFrom(file)
	writer.Flush()
}

```



```go
type WriterTo interface {
    WriteTo(w Writer) (n int64, err error)
}
```

WriteTo 将数据写入 w 中，直到没有数据可写或发生错误。其返回值 n 为写入的字节数。 在写入过程中遇到的任何错误也将被返回。

```go
func main() {
	reader := bytes.NewReader([]byte("golang 666"))
	reader.WriteTo(os.Stdout)
}
```



### Seeker 

```go
type Seeker interface {
    Seek(offset int64, whence int) (ret int64, err error)
}
```

Seek 方法是用于设置偏移量的，这样可以从某个特定位置开始操作数据流。

听起来和 ReaderAt/WriteAt 接口有些类似，不过 Seeker 接口更灵活，可以更好的控制读写数据流的位置。

demo 

```go
func main() {
	reader := strings.NewReader("Golang语言")
	reader.Seek(-6, io.SeekEnd)
	r, _, _ := reader.ReadRune()
	fmt.Printf("%c\n", r)
}
// 语
```

whence 的值，在 io 包中定义了相应的常量，应该使用这些常量

```go
const (
  SeekStart   = 0 // seek relative to the origin of the file
  SeekCurrent = 1 // seek relative to the current offset
  SeekEnd     = 2 // seek relative to the end
)
```

### Closer

```go
type Closer interface {
    Close() error
}
```

该接口比较简单，只有一个 Close() 方法，用于关闭数据流。

文件 (os.File)、归档（压缩包）、数据库连接、Socket 等需要手动关闭的资源都实现了 Closer 接口。

实际编程中，经常将 Close 方法的调用放在 defer 语句中。

错误示范：

```go
func main() {
	file, err := os.Open("demo2.txt")
	defer file.Close() // should check returned error before deferring file.Close() (SA5001)go-staticcheck

	if err != nil {
		fmt.Println(err)
	}
}
```

### PipeReader, PipeWriter

```go
// pipe 是 PipeReader 和 PipeWriter 底层的共享管道结构。
type pipe struct {
	wrMu sync.Mutex // Serializes Write operations
	wrCh chan []byte
	rdCh chan int

	once sync.Once // Protects closing done
	done chan struct{}
	rerr onceError
	werr onceError
}
```

----

```go
type PipeReader struct {
    p *pipe
}
```

PipeReader（一个没有任何导出字段的 struct）是管道的读取端。它实现了 io.Reader 和 io.Closer 接口。

从管道中读取数据。该方法会堵塞，直到管道写入端开始写入数据或写入端被关闭。如果写入端关闭时带有 error（即调用 CloseWithError 关闭），该Read返回的 err 就是写入端传递的error；否则 err 为 EOF。

----

```go
type PipeWriter struct {
    p *pipe
}
```

写数据到管道中。该方法会堵塞，直到管道读取端读完所有数据或读取端被关闭。如果读取端关闭时带有 error（即调用 CloseWithError 关闭），该Write返回的 err 就是读取端传递的error；否则 err 为 ErrClosedPipe。

----

```go
func Pipe() (*PipeReader, *PipeWriter) {
	p := &pipe{
		wrCh: make(chan []byte),
		rdCh: make(chan int),
		done: make(chan struct{}),
	}
	return &PipeReader{p}, &PipeWriter{p}
}
```

用于创建一个同步的内存管道 (synchronous in-memory pipe)

它将 io.Reader 连接到 io.Writer。一端的读取匹配另一端的写入，直接在这两端之间复制数据；它没有内部缓存。它对于并行调用 Read 和 Write 以及其它函数或 Close 来说都是安全的。一旦等待的 I/O 结束，Close 就会完成。并行调用 Read 或并行调用 Write 也同样安全：同种类的调用将按顺序进行控制。

正因为是*同步*的，因此不能在一个 goroutine 中进行读和写。

另外，对于管道的 close 方法（非 CloseWithError 时），err 会被置为 EOF。



demo:

```go
func main() {
	pipeReader, pipeWriter := io.Pipe()
	go PipeWrite(pipeWriter)
	go PipeRead(pipeReader)
	time.Sleep(30 * time.Second)
}

func PipeWrite(writer *io.PipeWriter) {
	data := []byte("golang 12345")
	for i := 0; i < 3; i++ {
		n, err := writer.Write(data)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Printf("写入字节 %d\n", n)
	}
	writer.CloseWithError(errors.New("写入段已关闭"))
}

func PipeRead(reader *io.PipeReader) {
	buf := make([]byte, 128)
	for {
		fmt.Println("接口端开始阻塞5秒钟...")
		time.Sleep(5 * time.Second)
		fmt.Println("接收端开始接受")
		n, err := reader.Read(buf)
		if err != nil {
			fmt.Println(err)
			return
		}
		fmt.Printf("收到字节: %d\n buf内容: %s\n", n, buf)
	}
}
```



### Copy, CopyN

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

Copy 将 src 复制到 dst，直到在 src 上到达 EOF 或发生错误。它返回复制的字节数，如果有错误的话，还会返回在复制时遇到的第一个错误。

成功的 Copy 返回 err == nil，而非 err == EOF。由于 Copy 被定义为从 src 读取直到 EOF 为止，因此它不会将来自 Read 的 EOF 当做错误来报告。

若 dst 实现了 ReaderFrom 接口，其复制操作可通过调用 dst.ReadFrom(src) 实现。此外，若 src 实现了 WriterTo 接口，其复制操作可通过调用 src.WriteTo(dst) 实现。

demo:

```go
func main() {
	io.Copy(os.Stdout, strings.NewReader("Golang 1234"))
}
```



----

```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```

CopyN 将 n 个字节(或到一个error)从 src 复制到 dst。 它返回复制的字节数以及在复制时遇到的最早的错误。当且仅当err == nil时,written == n 。

若 dst 实现了 ReaderFrom 接口，复制操作也就会使用它来实现。

demo:

```go
func main() {
	io.CopyN(os.Stdout, strings.NewReader("Golang_see you"), 6)
}
// Golang
```

## ioutil

废弃，相关功能迁移至 io 和os



## bufio

平台无关的缓冲 I/O，提升读写效率。

```go
package main

import (
	// "bufio"
	"io"
	"os"
)

func main() {
	f, _ := os.Open("./tmp.dat")
	defer f.Close()

	var r io.Reader = f
	// r = bufio.NewReaderSize(r, 8192)

	for {
		buf := make([]byte, 512)
		 _, err := r.Read(buf)
		if err == io.EOF { break }
	}
}

/*

$ dd if=/dev/random of=tmp.dat bs=1M count=100
$ strace ./test 2>&1 | grep "read" | wc -l

*/
```



### **Reader**

```go
// bufio/bufio.go

const (
	defaultBufSize = 4096 // 默认缓存大小
)

var (
	ErrInvalidUnreadByte = errors.New("bufio: invalid use of UnreadByte")
	ErrInvalidUnreadRune = errors.New("bufio: invalid use of UnreadRune")
	ErrBufferFull        = errors.New("bufio: buffer full")
	ErrNegativeCount     = errors.New("bufio: negative count")
)

// Reader 继承了  io.Reader 并且实现了
type Reader struct {
	buf          []byte
	rd           io.Reader // reader provided by the client
	r, w         int       // 缓存读写的位置
	err          error
	lastByte     int // 最后一次读到的字节; -1 means invalid
	lastRuneSize int // 最后一次读到的Rune的大小; -1 means invalid
}

const minReadBufferSize = 16
const maxConsecutiveEmptyReads = 100
```



**实例化**

新建Reader

```go
func NewReaderSize(rd io.Reader, size int) *Reader {
	// Is it already a Reader?
	b, ok := rd.(*Reader)
	if ok && len(b.buf) >= size {
		return b
	}
	if size < minReadBufferSize {
		size = minReadBufferSize
	}
	r := new(Reader)
	r.reset(make([]byte, size), rd)
	return r
}

func NewReader(rd io.Reader) *Reader {
	return NewReaderSize(rd, defaultBufSize)
}
```



**Read操作**

实现io的接口

```go
    func (b *Reader) Read(p []byte) (n int, err error)
    func (b *Reader) ReadByte() (c byte, err error)
    func (b *Reader) ReadRune() (r rune, size int, err error)
    func (b *Reader) UnreadByte() error
    func (b *Reader) UnreadRune() error
    func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
```



新的接口

```go
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
func (b *Reader) ReadBytes(delim byte) ([]byte, error)
func (b *Reader) ReadString(delim byte) (string, error)
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```

底层都是调用的`ReadSlice`方法



```go
func (b *Reader) ReadSlice(delim byte) (line []byte, err error) {
	s := 0 // search start index
	for {
		// Search buffer.
		if i := bytes.IndexByte(b.buf[b.r+s:b.w], delim); i >= 0 {
			i += s
			line = b.buf[b.r : b.r+i+1]
			b.r += i + 1
			break
		}

		// Pending error?
		if b.err != nil {
			line = b.buf[b.r:b.w]
			b.r = b.w
			err = b.readErr()
			break
		}

		// Buffer full?
		if b.Buffered() >= len(b.buf) {
			b.r = b.w
			line = b.buf
			err = ErrBufferFull
			break
		}

		s = b.w - b.r // do not rescan area we scanned before

		b.fill() // buffer is not full
	}

	// Handle last byte, if any.
	if i := len(line) - 1; i >= 0 {
		b.lastByte = int(line[i])
		b.lastRuneSize = -1
	}

	return
}
```



```go
func main() {
	reader := bufio.NewReader(strings.NewReader("aabbccdd. \n1122"))
	line, _ := reader.ReadSlice('\n')
	fmt.Printf("the line:%s\n", line)

	n, _ := reader.ReadSlice('\n')
	fmt.Printf("the line:%s\n", line)
	fmt.Println(string(n))
}


// the line:aabbccdd.

// the line:1122ccdd.

// 1122
```

注意返回的切片是对b.buf的引用，所以两次打印line不一样



填充操作 读一块进入缓存

```go
// fill reads a new chunk into the buffer.
func (b *Reader) fill() {
	// Slide existing data to beginning.
	if b.r > 0 {
		copy(b.buf, b.buf[b.r:b.w])
		b.w -= b.r
		b.r = 0
	}

	if b.w >= len(b.buf) {
		panic("bufio: tried to fill full buffer")
	}

	// Read new data: try a limited number of times.
	for i := maxConsecutiveEmptyReads; i > 0; i-- {
		n, err := b.rd.Read(b.buf[b.w:])
		if n < 0 {
			panic(errNegativeRead)
		}
		b.w += n
		if err != nil {
			b.err = err
			return
		}
		if n > 0 {
			return
		}
	}
	b.err = io.ErrNoProgress
}

```





### **Writer**

```go
type Writer struct {
	err error
	buf []byte
	n   int
	wr  io.Writer
}
```

**实例化**

```go
func NewWriterSize(w io.Writer, size int) *Writer {
	// Is it already a Writer?
	b, ok := w.(*Writer)
	if ok && len(b.buf) >= size {
		return b
	}
	if size <= 0 {
		size = defaultBufSize
	}
	return &Writer{
		buf: make([]byte, size),
		wr:  w,
	}
}

func NewWriter(w io.Writer) *Writer {
	return NewWriterSize(w, defaultBufSize)
}
```





**Write操作**

```go
// 返回缓存中有多少字节可用
func (b *Writer) Available() int { return len(b.buf) - b.n }

// 返回缓存中已经写入了多少字节
func (b *Writer) Buffered() int { return b.n }

func (b *Writer) Write(p []byte) (nn int, err error) {
    // 待写入数据量大于缓存区剩余空间，多次完成。
	for len(p) > b.Available() && b.err == nil {
		var n int
		if b.Buffered() == 0 {
			// Large write, empty buffer.
			// Write directly from p to avoid copy.
			n, b.err = b.wr.Write(p)
		} else {
            // 填满剩余缓存空间
			n = copy(b.buf[b.n:], p)
			b.n += n
			b.Flush()
		}
		nn += n
		p = p[n:]
	}
	if b.err != nil {
		return nn, b.err
	}
	n := copy(b.buf[b.n:], p)
	b.n += n
	nn += n
	return nn, nil
}
```

```go
// 将缓冲区数据 Flush 给底层 Writer。清空缓冲区。
func (b *Writer) Flush() error {
	if b.err != nil {
		return b.err
	}
	if b.n == 0 {
		return nil
	}
	n, err := b.wr.Write(b.buf[0:b.n])
	if n < b.n && err == nil {
		err = io.ErrShortWrite
	}
	if err != nil {
        // 部分写入。剩余数据前移。
		if n > 0 && n < b.n {
			copy(b.buf[0:b.n-n], b.buf[n:b.n])
		}
		b.n -= n
		b.err = err
		return err
	}
     // 全部写入。
	b.n = 0
	return nil
}
```



### **ReadWriter** 

```go
// ReadWriter 结构存储了 bufio.Reader 和 bufio.Writer 类型的指针（内嵌），它实现了 io.ReadWriter 结构。
type ReadWriter struct {
	*Reader
	*Writer
}

// NewReadWriter allocates a new ReadWriter that dispatches to r and w.
func NewReadWriter(r *Reader, w *Writer) *ReadWriter {
	return &ReadWriter{r, w}
}
```





### Scanner 

```go
type Scanner struct {
	r            io.Reader // The reader provided by the client.
	split        SplitFunc // The function to split the tokens.
	maxTokenSize int       // Maximum size of a token; modified by tests.
	token        []byte    // Last token returned by split.
	buf          []byte    // Buffer used as argument to split.
	start        int       // First non-processed byte in buf.
	end          int       // End of data in buf.
	err          error     // Sticky error.
	empties      int       // Count of successive empty tokens.
	scanCalled   bool      // Scan has been called; buffer is in use.
	done         bool      // Scan has finished.
}
```



**实例化**

```go
func NewScanner(r io.Reader) *Scanner {
	return &Scanner{
		r:            r,
		split:        ScanLines,
		maxTokenSize: MaxScanTokenSize,
	}
}
```



**SplitFunc**

```go
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```

SplitFunc 定义了 用于对输入进行分词的 split 函数的签名。
参数 data 是还未处理的数据，atEOF 标识 Reader 是否还有更多数据（是否到了EOF）。
返回值 advance 表示从输入中读取的字节数，token 表示下一个结果数据，err 则代表可能的错误。



**Split** 

通过 Split 方法为 Scanner 实例设置分词， Scanner 实例的默认 split 总是 ScanLines，如果我们想要用其他的 split，可以通过 Split 方法做到。注意，我们应该在调用 Scan 方法之前调用 Split 方法。

```go
func main() {
	const input = "This is The Golang Standard Library.\nWelcome you!"
	scanner := bufio.NewScanner(strings.NewReader(input))
	scanner.Split(bufio.ScanWords)
	count := 0
	for scanner.Scan() {
		count++
	}
	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}
	fmt.Println(count) // 8
}
```



demo：

```go
func main() {
	file, err := os.Create("demo.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()
	file.WriteString("Golang__twelve .\n hello .\n  welcome!")
	// 将文件 offset 设置到文件开头
	file.Seek(0, io.SeekStart)
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```



# 文本

## strings

字符串常见操作有：

- 字符串长度；

- 求子串；

- 是否存在某个字符或子串；

- 子串出现的次数（字符串匹配）；

- 字符串分割（切分）为[]string；

- 字符串是否有某个前缀或后缀；

- 字符或子串在字符串中首次出现的位置或最后一次出现的位置；

- 通过某个字符串将[]string 连接起来；

- 字符串重复几次；

- 字符串中子串替换；

- 大小写转换；

- Trim 操作；

- ...

   string 类型可以看成是一种特殊的 slice 类型，因此获取长度可以用内置的函数 len；同时支持 切片 操作，因此，子串获取很容易。

  这里说的字符，指得是 rune 类型，即一个 UTF-8 字符（Unicode 代码点）。

### Compare, EqualFold

**字符串比较**



```go
// go 1.20 strings/compare.go 
func Compare(a, b string) int {
    // 注意(rsc)：该函数不会调用运行时 cmpstring 函数，因为我们不想为使用 strings.Compare 提供任何性能理由。 基本上没有人应该使用 strings.Compare。
	if a == b {
		return 0
	}
	if a < b {
		return -1
	}
	return +1
}
```



```go
// 忽略大小写
func EqualFold(s, t string) bool
```

demo : 

```go
func main() {
	a := "gopher"
	b := "hello world"
	fmt.Println(strings.Compare(a, b)) // -1
	fmt.Println(strings.Compare(a, a)) // 0
	fmt.Println(strings.Compare(b, a)) // 1

	fmt.Println(strings.EqualFold("GO", "go")) // true
	fmt.Println(strings.EqualFold("壹", "一"))   // false
}
```

```go
Contains ContainsAny ContainsRune
```

### Contains

是否存在某个字符或子串

Contains, ContainsAny, ContainsRune

```go
// Contains 子串 substr 在 s 中，返回 true
func Contains(s, substr string) bool {
	return Index(s, substr) >= 0
}

// chars 中任何一个 Unicode 码位在 s 中，返回 true
func ContainsAny(s, chars string) bool {
	return IndexAny(s, chars) >= 0
}

// Unicode 码位 r 在 s 中，返回 true
func ContainsRune(s string, r rune) bool {
	return IndexRune(s, r) >= 0
}
```



demo:

```go
func main() {
	fmt.Println(strings.ContainsAny("team", "i"))         // false
	fmt.Println(strings.ContainsAny("failure", "u & i"))  // true
	fmt.Println(strings.ContainsAny("in failure", "s g")) // true
	fmt.Println(strings.ContainsAny("foo", ""))           // false
	fmt.Println(strings.ContainsAny("", ""))              // false
}
```

### Count

在数据结构与算法中，可能会讲解以下字符串匹配算法：

- 朴素匹配算法
- KMP 算法
- Rabin-Karp 算法
- Boyer-Moore 算法



在 Go 中，查找子串出现次数即字符串模式匹配，实现的是 Rabin-Karp 算法。：

```go
func Count(s, substr string) int {
	// special case
	if len(substr) == 0 {
		return utf8.RuneCountInString(s) + 1
	}
	if len(substr) == 1 {
		return bytealg.CountString(s, substr[0])
	}
	n := 0
	for {
		i := Index(s, substr)
		if i == -1 {
			return n
		}
		n++
		s = s[i+len(substr):]
	}
}
```



```go
// IndexRabinKarp uses the Rabin-Karp search algorithm to return the index of the
// first occurrence of substr in s, or -1 if not present.
func IndexRabinKarp(s, substr string) int {
	// Rabin-Karp search
	hashss, pow := HashStr(substr)
	n := len(substr)
	var h uint32
	for i := 0; i < n; i++ {
		h = h*PrimeRK + uint32(s[i])
	}
	if h == hashss && s[:n] == substr {
		return 0
	}
	for i := n; i < len(s); {
		h *= PrimeRK
		h += uint32(s[i])
		h -= pow * uint32(s[i-n])
		i++
		if h == hashss && s[i-n:i] == substr {
			return i - n
		}
	}
	return -1
}
```



当 sep 为空时，Count 的返回值是：utf8.RuneCountInString(s) + 1



demo: 

```go
func main() {
	fmt.Println(strings.Count("cheese", "e"))    // 3
	fmt.Println(len("谷歌中国"))                  // 12
	fmt.Println(strings.Count("谷歌中国", ""))    // 5
}
```



### Split

字符串分割为[]string

该包提供了六个三组分割函数：Fields 和 FieldsFunc、Split 和 SplitAfter、SplitN 和 SplitAfterN。



**Fields , FieldsFunc**

```go
func Fields(s string) []string
func FieldsFunc(s string, f func(rune) bool) []string
```

**Fields** 用一个或多个连续的空格分隔字符串 s，返回子字符串的数组（slice）。如果字符串 s 只包含空格，则返回空列表 ([]string 的长度为 0）。其中，空格的定义是 unicode.IsSpace，之前已经介绍过。

常见间隔符包括：`'\t', '\n', '\v', '\f', '\r', ' ', U+0085 (NEL), U+00A0 (NBSP)`

由于是用空格分隔，因此结果中不会含有空格或空子字符串，例如：

```go
fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
// Fields are: ["foo" "bar" "baz"]
```



**FieldsFunc** 用 Unicode  码位进行分隔：满足 f(c) 返回 true。该函数返回[]string。如果字符串 s 中所有的码位 (unicode code points) 都满足 f(c) 或者 s 是空，则 FieldsFunc 返回空 slice。

也就是说，我们可以通过实现一个回调函数来指定分隔字符串 s 的字符。比如上面的例子，我们通过 FieldsFunc 来实现：

```go
fmt.Println(strings.FieldsFunc("  foo bar  baz   ", unicode.IsSpace))
```



**Split 和 SplitAfter、 SplitN 和 SplitAfterN**

都是调用`genSplit` 函数

```go
// 分割字符串
func Split(s, sep string) []string { return genSplit(s, sep, 0, -1) }

// 在切割符号后，分割字符串(返回值包含切割符)
func SplitAfter(s, sep string) []string { return genSplit(s, sep, len(sep), -1) }

// 分割字符串，返回长度为n的切片
func SplitN(s, sep string, n int) []string { return genSplit(s, sep, 0, n) }

// 在切割符号后，分割字符串，返回长度为n的切片
func SplitAfterN(s, sep string, n int) []string { return genSplit(s, sep, len(sep), n) }
```



```go
// Generic split: splits after each instance of sep,
// including sepSave bytes of sep in the subarrays.
func genSplit(s, sep string, sepSave, n int) []string {
	if n == 0 {
		return nil
	}
	if sep == "" {
		return explode(s, n)
	}
	if n < 0 {
		n = Count(s, sep) + 1
	}

	if n > len(s)+1 {
		n = len(s) + 1
	}
	a := make([]string, n)
	n--
	i := 0
	for i < n {
		m := Index(s, sep)
		if m < 0 {
			break
		}
		a[i] = s[:m+sepSave]
		s = s[m+len(sep):]
		i++
	}
	a[i] = s
	return a[:i+1]
}
```



demo: 

```go
func main() {
	fmt.Printf("%q\n", strings.Split("a,b,c", ","))                        // ["a" "b" "c"]
	fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a ")) // ["" "man " "plan " "canal panama"]
	fmt.Printf("%q\n", strings.Split(" xyz ", ""))                         // [" " "x" "y" "z" " "]
	fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))            // [""]

	fmt.Printf("%q\n", strings.SplitN("a,b,c", ",", 2))      // ["a" "b,c"]
	fmt.Printf("%q\n", strings.SplitAfter("a,b,c", ","))     // ["a," "b," "c"]
	fmt.Printf("%q\n", strings.SplitAfterN("a,b,c", ",", 2)) // ["a," "b,c"]
}
```



### HasPrefix, HasSuffix

字符串是否有某个前缀或后缀

```go
// HasPrefix tests whether the string s begins with prefix.
func HasPrefix(s, prefix string) bool {
	return len(s) >= len(prefix) && s[0:len(prefix)] == prefix
}

// HasSuffix tests whether the string s ends with suffix.
func HasSuffix(s, suffix string) bool {
	return len(s) >= len(suffix) && s[len(s)-len(suffix):] == suffix
}
```



demo :

```go
func main() {
	fmt.Println(strings.HasPrefix("Gopher", "Go")) // true
	fmt.Println(strings.HasPrefix("Gopher", "C"))  // false
	fmt.Println(strings.HasPrefix("Gopher", ""))   // true
	fmt.Println(strings.HasSuffix("Amigo", "go"))  // true
	fmt.Println(strings.HasSuffix("Amigo", "Ami")) // false
	fmt.Println(strings.HasSuffix("Amigo", ""))    // true
}
```



### Index

```go
// 在 s 中查找 sep 的第一次出现，返回第一次出现的索引
func Index(s, sep string) int

// 在 s 中查找字节 c 的第一次出现，返回第一次出现的索引
func IndexByte(s string, c byte) int

// chars 中任何一个 Unicode码位 在 s 中首次出现的位置
func IndexAny(s, chars string) int

// 查找字符 c 在 s 中第一次出现的位置，其中 c 满足 f(c) 返回 true
func IndexFunc(s string, f func(rune) bool) int

// Unicode码位 r 在 s 中第一次出现的位置
func IndexRune(s string, r rune) int

// 有三个对应的查找最后一次出现的位置
func LastIndex(s, sep string) int
func LastIndexByte(s string, c byte) int
func LastIndexAny(s, chars string) int
func LastIndexFunc(s string, f func(rune) bool) int
```



demo: 

```go
func main() {
	han := func(c rune) bool {
		return unicode.Is(unicode.Han, c) // 汉字
	}
	fmt.Println(strings.Index("Hello, world", ","))     // 5
	fmt.Println(strings.LastIndex("Hello, world", "l")) // 10
	fmt.Println(strings.IndexFunc("Hello, world", han)) // -1
	fmt.Println(strings.IndexFunc("Hello, 世界", han))    // 7
}
```

### Join

```go
// Join concatenates the elements of its first argument to create a single string. The separator
// string sep is placed between elements in the resulting string.
func Join(elems []string, sep string) string {
	switch len(elems) {
	case 0:
		return ""
	case 1:
		return elems[0]
	}
	n := len(sep) * (len(elems) - 1)
	for i := 0; i < len(elems); i++ {
		n += len(elems[i])
	}

    // Builder 避免了大量字符串连接操作，最大限度地减少了内存复制
	var b Builder
	b.Grow(n)
	b.WriteString(elems[0])
	for _, s := range elems[1:] {
		b.WriteString(sep)
		b.WriteString(s)
	}
	return b.String()
}
```



demo:

```go
func main() {
	fmt.Println(strings.Join([]string{"name=xxx", "age=xx"}, "&"))
    // name=xxx&age=xx
}
```



### Replace

```go
// ReplaceAll 返回字符串 s 的副本，其中包含所有内容
func ReplaceAll(s, old, new string) string {
	return Replace(s, old, new, -1)
}

// Replace 返回字符串 s 的前 n 个副本
func Replace(s, old, new string, n int) string {
	if old == new || n == 0 {
		return s // 避免分配内存
	}

	// 计算Replace 的次数
	if m := Count(s, old); m == 0 {
		return s // avoid allocation
	} else if n < 0 || m < n {
		n = m
	}

	// Apply replacements to buffer.
	var b Builder
	b.Grow(len(s) + n*(len(new)-len(old)))
	start := 0
	for i := 0; i < n; i++ {
		j := start
		if len(old) == 0 {
			if i > 0 {
				_, wid := utf8.DecodeRuneInString(s[start:])
				j += wid
			}
		} else {
			j += Index(s[start:], old)
		}
		b.WriteString(s[start:j])
		b.WriteString(new)
		start = j + len(old)
	}
	b.WriteString(s[start:])
	return b.String()
}
```



demo :

```go
func main() {
	fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))      // oinky oinky oink
	fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1)) // moo moo moo
	fmt.Println(strings.ReplaceAll("oink oink oink", "oink", "moo"))  // moo moo moo
}
```



### Trim

```go
// 将 s 左侧和右侧中匹配 cutset 中的任一字符的字符去掉
func Trim(s string, cutset string) string

// 将 s 左侧的匹配 cutset 中的任一字符的字符去掉
func TrimLeft(s string, cutset string) string

// 将 s 右侧的匹配 cutset 中的任一字符的字符去掉
func TrimRight(s string, cutset string) string

// 如果 s 的前缀为 prefix 则返回去掉前缀后的 string , 否则 s 没有变化。
func TrimPrefix(s, prefix string) string

// 如果 s 的后缀为 suffix 则返回去掉后缀后的 string , 否则 s 没有变化。
func TrimSuffix(s, suffix string) string

// 将 s 左侧和右侧的间隔符去掉。常见间隔符包括：'\t', '\n', '\v', '\f', '\r', ' ', U+0085 (NEL)
func TrimSpace(s string) string

// 将 s 左侧和右侧的匹配 f 的字符去掉
func TrimFunc(s string, f func(rune) bool) string

// 将 s 左侧的匹配 f 的字符去掉
func TrimLeftFunc(s string, f func(rune) bool) string

// 将 s 右侧的匹配 f 的字符去掉
func TrimRightFunc(s string, f func(rune) bool) string
```



demo :

```go
func main() {
	x := "!!!@@@你好,!@#$ Gophers###$$$"
	fmt.Println(strings.Trim(x, "@#$!%^&*()_+=-"))                  // 你好,!@#$ Gophers
	fmt.Println(strings.TrimLeft(x, "@#$!%^&*()_+=-"))              // 你好,!@#$ Gophers###$$$
	fmt.Println(strings.TrimRight(x, "@#$!%^&*()_+=-"))             // !!!@@@你好,!@#$ Gophers
	fmt.Println(strings.TrimSpace(" \t\n Hello, Gophers \n\t\r\n")) // Hello, Gophers
	fmt.Println(strings.TrimPrefix(x, "!"))                         // !!@@@你好,!@#$ Gophers###$$$
	fmt.Println(strings.TrimSuffix(x, "$"))                         // !!!@@@你好,!@#$ Gophers###$$

	f := func(r rune) bool {
		return !unicode.Is(unicode.Han, r) // 非汉字返回 true
	}
	fmt.Println(strings.TrimFunc(x, f))      // 你好
	fmt.Println(strings.TrimLeftFunc(x, f))  // 你好,!@#$ Gophers###$$$
	fmt.Println(strings.TrimRightFunc(x, f)) // !!!@@@你好
}
```



### Case 

大小写转换

如果不是ASCII码，调用的都是`unicode`的方法

```go
func ToLower(s string) string
func ToLowerSpecial(c unicode.SpecialCase, s string) string

func ToUpper(s string) string
func ToUpperSpecial(c unicode.SpecialCase, s string) string
```



demo :

```go
func main() {
	fmt.Println(strings.ToLower("HELLO WORLD"))                             // hello world
	fmt.Println(strings.ToLower("Ā Á Ǎ À"))                                 // ā á ǎ à
	fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "壹"))           // 壹
	fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "HELLO WORLD")) // hello world
	fmt.Println(strings.ToLower("Önnek İş"))                                // önnek iş
	fmt.Println(strings.ToLowerSpecial(unicode.TurkishCase, "Önnek İş"))    // önnek iş
}
```



## bytes

该包定义了一些操作 byte slice 的便利操作。因为字符串可以表示为 `[]byte`，因此，bytes 包定义的函数、方法等和 strings 包很类似.

说明：为了方便，会称呼 `[]byte`为 字节数组



### Contains

是否存在某个子 slice ，和string 类似

```go
// Contains 子串 substr 在 s 中，返回 true
func Contains(b, subslice []byte) bool {
	return Index(b, subslice) != -1
}

// chars 中任何一个 Unicode 码位在 s 中，返回 true
func ContainsAny(b []byte, chars string) bool {
	return IndexAny(b, chars) >= 0
}

// Unicode 码位 r 在 s 中，返回 true
func ContainsRune(b []byte, r rune) bool {
	return IndexRune(b, r) >= 0
}
```



demo:

```go
func main() {
	a := []byte{'A', 'S', 'D', 'Q', 'W', 'e'}
	fmt.Println(bytes.Contains(a, []byte{'e'}))   // true
	fmt.Println(bytes.Contains(a, []byte{'E'}))   // false
	fmt.Println(bytes.Contains(a, []byte{'f'}))   // false
	fmt.Println(bytes.ContainsAny(a, "ASD"))      // true
	fmt.Println(bytes.ContainsRune(a, rune('A'))) // true
}
```

### Count

记录`[]byte` 出现的次数

```go
func Count(s, sep []byte) int {
	// special case
	if len(sep) == 0 {
		return utf8.RuneCount(s) + 1
	}
	if len(sep) == 1 {
		return bytealg.Count(s, sep[0])
	}
	n := 0
	for {
		i := Index(s, sep)
		if i == -1 {
			return n
		}
		n++
		s = s[i+len(sep):]
	}
}
```



demo: 

```go
func main() {
	a := []byte{'A', 'S', 'D', 'Q', 'W', 'e', 'e'}
	fmt.Println(bytes.Count(a, []byte{'e'}))      // 2
	fmt.Println(bytes.Count(a, []byte{'e', 'e'})) // 1
	fmt.Println(bytes.Count(a, []byte{'e', 'W'})) // 0
}
```



### Runes

Runes 类型转换

```go
// 将 []byte 转换为 []rune
func Runes(s []byte) []rune {
	t := make([]rune, utf8.RuneCount(s))
	i := 0
	for len(s) > 0 {
		r, l := utf8.DecodeRune(s)
		t[i] = r
		i++
		s = s[l:]
	}
	return t
}
```



demo :

```go
func main() {
	b := []byte("你好，世界")
	for k, v := range b {
		fmt.Printf("%d:%s |", k, string(v))
	}
	r := bytes.Runes(b)
	for k, v := range r {
		fmt.Printf("%d:%s|", k, string(v))
	}
}

// 0:ä |1:½ |2:  |3:å |4:¥ |5:½ |6:ï |7:¼ |8: |9:ä |10:¸ |11: |12:ç |13: |14: |
// 0:你 |1:好 |2:， |3:世 |4:界 |
```



### Reader 

```go
type Reader struct {
	s        []byte
	i        int64 // 当前下标
	prevRune int   // 前一个字符的下标，存在 < 0的情况
}
```

bytes 包下的 Reader 类型实现了 io 包下的 Reader, ReaderAt, RuneReader, RuneScanner, ByteReader, ByteScanner, ReadSeeker, Seeker, WriterTo 等多个接口。主要用于 Read 数据。

与 Buffer 不同，Reader 是只读的并且支持查找。

Reader 的零值操作就像空切片的 Reader 一样。



Reader 包含了 8 个读取相关的方法，实现了前面提到的 io 包下的 9 个接口（ReadSeeker 接口内嵌 Reader 和 Seeker 两个接口）：

```go
// 读取数据至 b 
func (r *Reader) Read(b []byte) (n int, err error) 

// 读取一个字节
func (r *Reader) ReadByte() (byte, error)

// 读取一个字符
func (r *Reader) ReadRune() (ch rune, size int, err error)

// 读取数据至 w
func (r *Reader) WriteTo(w io.Writer) (n int64, err error)

// 进度下标指向前一个字节，如果 r.i <= 0 返回错误。
func (r *Reader) UnreadByte() 

// 进度下标指向前一个字符，如果 r.i <= 0 返回错误，且只能在每次 ReadRune 方法后使用一次，否则返回错误。
func (r *Reader) UnreadRune() 

// 读取 r.s[off:] 的数据至b，该方法忽略进度下标 i，不使用也不修改。
func (r *Reader) ReadAt(b []byte, off int64) (n int, err error) 

// 根据 whence 的值，修改并返回进度下标 i ，当 whence == 0 ，进度下标修改为 off，当 whence == 1 ，进度下标修改为 i+off，当 whence == 2 ，进度下标修改为 len[s]+off.
// off 可以为负数，whence 的只能为 0，1，2，当 whence 为其他值或计算后的进度下标越界，则返回错误。
func (r *Reader) Seek(offset int64, whence int) (int64, error)
```



demo : 

```go
func main() {
	x := []byte("你好，世界")
	r1 := bytes.NewReader(x)

	ch, size, _ := r1.ReadRune()
	fmt.Println(size, string(ch)) // 3 你
	_ = r1.UnreadRune()
	ch, size, _ = r1.ReadRune()
	fmt.Println(size, string(ch)) // 3 你
	_ = r1.UnreadRune()

	by, _ := r1.ReadByte()
	fmt.Println(by) // 228
	_ = r1.UnreadByte()
	by, _ = r1.ReadByte()
	fmt.Println(by) // 228
	_ = r1.UnreadByte()

	d1 := make([]byte, 6)
	n, _ := r1.Read(d1)
	fmt.Println(n, string(d1)) // 6 你好

	d2 := make([]byte, 6)
	n, _ = r1.ReadAt(d2, 0)
	fmt.Println(n, string(d2)) // 6 你好

	w1 := &bytes.Buffer{}
	_, _ = r1.Seek(0, 0)
	_, _ = r1.WriteTo(w1)
	fmt.Println(w1.String()) // 你好，世界
}
```



### Buffer 

Buffer 是一个可变大小的字节缓冲区，具有 Read 和 Write 方法。

Buffer 的零值是一个可以使用的空缓冲区。

```go
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```



Buffer 包含了 21 个读写相关的方法，大部分同名方法的用法与前面讲的类似，这里只讲演示其中的 3 个方法：

```go
// 读取到字节 delim 后，以字节数组的形式返回该字节及前面读取到的字节。如果遍历 b.buf 也找不到匹配的字节，则返回错误(一般是 EOF)
func (b *Buffer) ReadBytes(delim byte) (line []byte, err error)

// 读取到字节 delim 后，以字符串的形式返回该字节及前面读取到的字节。如果遍历 b.buf 也找不到匹配的字节，则返回错误(一般是 EOF)
func (b *Buffer) ReadString(delim byte) (line string, err error)

// 截断 b.buf , 舍弃 b.off+n 之后的数据。n == 0 时，调用 Reset 方法重置该对象，当 n 越界时（n < 0 || n > b.Len() ）方法会触发 panic.
func (b *Buffer) Truncate(n int)
```





demo : 

```go
func main() {
	a := bytes.NewBufferString("Good Night")

	x, err := a.ReadBytes('t')
	if err != nil {
		fmt.Println("delim:t err:", err)
	} else {
		fmt.Println(string(x)) // Good Night
	}

	a.Truncate(0)
	a.WriteString("Good Night")
	fmt.Println(a.Len()) // 10
	a.Truncate(5)
	fmt.Println(a.Len()) // 5
	y, err := a.ReadString('N')
	if err != nil {
		fmt.Println("delim:N err:", err) // delim:N err: EOF
	} else {
		fmt.Println(y)
	}
}
```



## strconv

字符串和基本数据类型之间转换

这里的基本数据类型包括：布尔、整型（包括有 / 无符号、二进制、八进制、十进制和十六进制）和浮点型等。



### Error

strconv 中的错误处理。

由于将字符串转为其他数据类型可能会出错，strconv 包定义了两个 error 类型的变量：ErrRange 和 ErrSyntax。

ErrRange 表示值超过了类型能表示的最大范围，比如将 "128" 转为 int8 就会返回这个错误；
ErrSyntax 表示语法错误，比如将 "" 转为 int 类型会返回这个错误。

然而，在返回错误的时候，不是直接将上面的变量值返回，而是通过构造一个 NumError 类型的 error 对象返回。NumError 结构的定义如下：

```go
// NumError 记录失败的转换。
type NumError struct {
	Func string // 失败的函数 (ParseBool, ParseInt, ParseUint, ParseFloat, ParseComplex)
	Num  string // 输入
	Err  error  // 转换失败的原因 (e.g. ErrRange, ErrSyntax, etc.)
}
```

实现了error接口

```go
func (e *NumError) Error() string {
	return "strconv." + e.Func + ": " + "parsing " + Quote(e.Num) + ": " + e.Err.Error()
}
```



构造error

```go
func syntaxError(fn, str string) *NumError {
    return &NumError{fn, str, ErrSyntax}
}

func rangeError(fn, str string) *NumError {
    return &NumError{fn, str, ErrRange}
}
```



### int

字符串和整型之间的转换



**字符串转为整型**

```go
func ParseInt(s string, base int, bitSize int) (i int64, err error)
func ParseUint(s string, base int, bitSize int) (n uint64, err error)
func Atoi(s string) (i int, err error)
```

Atoi 是 ParseInt 的便捷版，内部通过调用 `ParseInt(s, 10, 0)` 来实现的；
ParseInt 转为有符号整型；
ParseUint 转为无符号整型；

参数 *base* 代表字符串按照给定的进制进行解释。一般的，base 的取值为 2~36，如果 base 的值为 0，则会根据字符串的前缀来确定 base 的值："0x" 表示 16 进制； "0" 表示 8 进制；否则就是 10 进制。

参数 *bitSize* 表示的是整数取值范围，或者说整数的具体类型。取值 0、8、16、32 和 64 分别代表 int、int8、int16、int32 和 int64。



当 bitSize==0 时

Go 中，int/uint 类型，不同系统能表示的范围是不一样的，目前的实现是，32 位系统占 4 个字节；64 位系统占 8 个字节。当 bitSize==0 时，应该表示 32 位还是 64 位呢？这里没有利用 *runtime.GOARCH* 之类的方式，而是巧妙的通过如下表达式确定 intSize：

```go
const intSize = 32 << uint(^uint(0)>>63)
const IntSize = intSize // number of bits in int, uint (32 or 64)
```



**整型转为字符串**

```go
func FormatUint(i uint64, base int) string    // 无符号整型转字符串
func FormatInt(i int64, base int) string    // 有符号整型转字符串
func Itoa(i int) string
```

Itoa 内部直接调用 FormatInt(i, 10)*实现的。base 参数可以取 2~36（0-9，a-z）。



### bool

字符串和布尔值之间的转换比较简单，主要有三个函数：

```go
// 接受 1, t, T, TRUE, true, True, 0, f, F, FALSE, false, False 等字符串；

// 其他形式的字符串会返回错误
func ParseBool(str string) (value bool, err error)

// 直接返回 "true" 或 "false"
func FormatBool(b bool) string

// 将 "true" 或 "false" append 到 dst 中
// 这里用了一个 append 函数对于字符串的特殊形式：append(dst, "true"...)
func AppendBool(dst []byte, b bool)
```





### float

字符串和浮点数之间的转换，包含三个函数：

```go
func ParseFloat(s string, bitSize int) (f float64, err error)
func FormatFloat(f float64, fmt byte, prec, bitSize int) string
func AppendFloat(dst []byte, f float64, fmt byte, prec int, bitSize int)
```



由于浮点数有精度的问题，精度不一样，ParseFloat 和 FormatFloat 可能达不到互逆的效果。如：

```go
s := strconv.FormatFloat(1234.5678, 'g', 6, 64)
strconv.ParseFloat(s, 64)
```



## unicode

 go 对 unicode 包的支持，由于 UTF-8 的作者 Ken Thompson 同时也是 go 语言的创始人，所以说，在字符支持方面，几乎没有语言的理解会高于 go 了。 go 对 unicode 的支持包含三个包 :

- unicode
- unicode/utf8
- unicode/utf16

unicode 包包含基本的字符判断函数。utf8 包主要负责 rune 和 byte 之间的转换。utf16 包负责 rune 和 uint16 数组之间的转换。

go 语言的所有代码都是 UTF8 的，所以如果我们在程序中的字符串都是 utf8 编码的，但是我们的单个字符（单引号扩起来的）却是 unicode 的。



### unicode

unicode 包含了对 rune 的判断。这个包把所有 unicode 涉及到的编码进行了分类，使用结构

```go
type RangeTable struct {
    R16         []Range16
    R32         []Range32
    LatinOffset int
}
```

### 



```go
func IsControl(r rune) bool  // 是否控制字符
func IsDigit(r rune) bool  // 是否阿拉伯数字字符，即 0-9
func IsGraphic(r rune) bool // 是否图形字符
func IsLetter(r rune) bool // 是否字母
func IsLower(r rune) bool // 是否小写字符
func IsMark(r rune) bool // 是否符号字符
func IsNumber(r rune) bool // 是否数字字符，比如罗马数字Ⅷ也是数字字符
func IsOneOf(ranges []*RangeTable, r rune) bool // 是否是 RangeTable 中的一个
func IsPrint(r rune) bool // 是否可打印字符
func IsPunct(r rune) bool // 是否标点符号
func IsSpace(r rune) bool // 是否空格
func IsSymbol(r rune) bool // 是否符号字符
func IsTitle(r rune) bool // 是否 title case
func IsUpper(r rune) bool // 是否大写字符
func Is(rangeTab *RangeTable, r rune) bool // r 是否为 rangeTab 类型的字符
func In(r rune, ranges ...*RangeTable) bool  // r 是否为 ranges 中任意一个类型的字符
```



demo:

```go
func main() {
	single := '\u0015'
	fmt.Println(unicode.IsControl(single)) // true
	single = '\ufe35'
	fmt.Println(unicode.IsControl(single)) // false

	digit := '1'
	fmt.Println(unicode.IsDigit(digit))  // true
	fmt.Println(unicode.IsNumber(digit)) // true

	letter := 'Ⅷ'
	fmt.Println(unicode.IsDigit(letter))  // false
	fmt.Println(unicode.IsNumber(letter)) // true

	han := '你'
	fmt.Println(unicode.IsDigit(han))                                   // false
	fmt.Println(unicode.Is(unicode.Han, han))                           // true
	fmt.Println(unicode.In(han, unicode.Gujarati, unicode.White_Space)) // false
}
```



### utf8

utf8 里面的函数就有一些字节和字符的转换。



```go
//判断是否符合 utf8 编码的函数：
func Valid(p []byte) bool
func ValidRune(r rune) bool
func ValidString(s string) bool

// 判断 rune 所占字节数：
func RuneLen(r rune) int

// 判断字节串或者字符串的 rune 数：
func RuneCount(p []byte) int
func RuneCountInString(s string) (n int)

// 编码和解码到 rune：
func EncodeRune(p []byte, r rune) int
func DecodeRune(p []byte) (r rune, size int)
func DecodeRuneInString(s string) (r rune, size int)
func DecodeLastRune(p []byte) (r rune, size int)
func DecodeLastRuneInString(s string) (r rune, size int)

// 是否为完整 rune：
func FullRune(p []byte) bool
func FullRuneInString(s string) bool

// 是否为 rune 第一个字节：
func RuneStart(b byte) bool
```



demo :

```go
func main() {

	word := []byte("界")

	fmt.Println(utf8.Valid(word[:2]))   // false
	fmt.Println(utf8.ValidRune('界'))    // true
	fmt.Println(utf8.ValidString("世界")) // true

	fmt.Println(utf8.RuneLen('界')) // 3

	fmt.Println(utf8.RuneCount(word))         // 1
	fmt.Println(utf8.RuneCountInString("世界")) // 2

	p := make([]byte, 3)
	utf8.EncodeRune(p, '好')
	fmt.Println(p)                                 // [229 165 189]
	fmt.Println(utf8.DecodeRune(p))                // 22909 3
	fmt.Println(utf8.DecodeRuneInString("你好"))     // 20320 3
	fmt.Println(utf8.DecodeLastRune([]byte("你好"))) // 22909 3
	fmt.Println(utf8.DecodeLastRuneInString("你好")) // 22909 3

	fmt.Println(utf8.FullRune(word[:2]))     // false
	fmt.Println(utf8.FullRuneInString("你好")) // true

	fmt.Println(utf8.RuneStart(word[1])) // false
	fmt.Println(utf8.RuneStart(word[0])) // true
}
```







### uft16

```go
// 将 uint16 和 rune 进行转换
func Encode(s []rune) []uint16
func EncodeRune(r rune) (r1, r2 rune)
func Decode(s []uint16) []rune
func DecodeRune(r1, r2 rune) rune

// 是否为有效代理对
func IsSurrogate(r rune) bool 
```

unicode 有个基本字符平面和增补平面的概念，基本字符平面只有 65535 个字符，增补平面（有 16 个）加上去就能表示 1114112 个字符。 utf16 就是严格实现了 unicode 的这种编码规范。

而基本字符和增补平面字符就是一个代理对（Surrogate Pair）。一个代理对可以和一个 rune 进行转换。



demo: 

```go
func main() {
	words := []rune{'𝓐', '𝓑'}

	u16 := utf16.Encode(words)
	fmt.Println(u16)               // [55349 56528 55349 56529]
	fmt.Println(utf16.Decode(u16)) // [120016 120017]

	r1, r2 := utf16.EncodeRune('𝓐')
	fmt.Println(r1, r2)                   // 55349 56528
	fmt.Println(utf16.DecodeRune(r1, r2)) // 120016
	fmt.Println(utf16.IsSurrogate(r1))    // true
	fmt.Println(utf16.IsSurrogate(r2))    // true
	fmt.Println(utf16.IsSurrogate(1234))  // false
}
```

