# Ch7 interface 介面

### Agenda
- 7.1 何謂介面
- 7.2 介面基本用法
- 7.3 空介面
- 7.4 以介面為合約
- 7.5 介面嵌入介面
- 7.6 介面值
- 7.7 sort.Interface排序
- 7.8 error介面

## 7.1 何謂介面
1. Go的介面是一種**抽象型別**
2. 它是滿足**隱含性的**，無需聲明哪個struct實現了這個介面
3. 可以用來定義方法，更精確的說是它的方法提供什麼行為
4. 利用 interface 可實現泛型、多型的功能，從而可以調用同一個函數名的函數但實現完全不同的功能。

## 7.2 介面基本用法
- 如何宣告

用type來宣告interface

裡面宣告抽象方法
```
type BankAccount interface {
	GetName() string
	GetBalance() int
	Deposit(amount int)
	Withdraw(amount int) error
}
```
可以看到裏面宣告了三個方法，不過現在也只有定義並沒有實踐

- 如何實踐

Golang 中如果自定義型態實現了 interface 的所有方法，那麼它就會認定該自定義型態也是 interface 型態的一種

例如：在這邊宣告一個```EsunAccount```的struct，並且實踐了```GetBalance()``` ```Deposit()``` ```Withdraw```這三個方法，那麼Golang就會自動判定我們實作了```BankAccount```這個介面

```
type EsunAccount struct {
}

func (e *EsunAccount) GetName() string {
	//TODO implement me
}

func (e *EsunAccount) GetBalance() int {
	//TODO implement me
}

func (e *EsunAccount) Deposit(amount int) {
	//TODO implement me
}

func (e *EsunAccount) Withdraw(amount int) error {
	//TODO implement me
}
```
特別注意的是：當我們實踐方法時，都是用指標```(*EsunAccount)```

這個所代表的意思是透過傳遞指標來操控同一個struct實例

那趕緊來看看介面的效益吧
```
//esun.go
type EsunAccount struct {
	balance int
}

func NewEsunAccount() *EsunAccount {
	return &EsunAccount{
		accountName: "esunAccount",
		balance: 0,
	}
}

func (e *EsunAccount) GetName() string {
	fmt.Printf("Account Name= %s\n", e.accountName)
	return e.accountName
}

func (e *EsunAccount) GetBalance() int {
	return e.balance
}

func (e *EsunAccount) Deposit(amount int) {
	e.balance += amount
}

func (e *EsunAccount) Withdraw(amount int) error {
	newBalance := e.balance - amount
	if newBalance < 0 {
		return errors.New("insufficient funds")
	}
	e.balance = newBalance
	return nil
}
```

```
//main.go
func main() {
	esun := NewEsunAccount()
	//存錢
	esun.Deposit(300)
	esun.Deposit(100)
	esun.Deposit(500)

	//領錢
	esun.Withdraw(600)
	//印出帳戶剩多少錢
	fmt.Printf("%s balance: %d\n", esun.GetName(), esun.GetBalance())
}
```
執行結果：
```
esunAccount balance: 300
```
接下來要展現介面真正的力量了

當我們多一個銀行帳號```CtbcAccount```
```
//ctbc.go
type CtbcAccount struct {
	balance int
	fee     int
}

func NewCtbcAccount() *CtbcAccount {
	return &CtbcAccount{
		accountName: "ctbcAccount",
		balance: 0,
		fee:     15,
	}
}

func (c *CtbcAccount) GetName() string {
	fmt.Printf("Account Name= %s\n", c.accountName)
	return c.accountName
}

func (c *CtbcAccount) GetBalance() int {
	return c.balance
}

func (c *CtbcAccount) Deposit(amount int) {
	c.balance += amount
}

func (c *CtbcAccount) Withdraw(amount int) error {
	newBalance := c.balance - amount - c.fee
	if newBalance < 0 {
		return errors.New("insufficient funds")
	}
	c.balance = newBalance
	return nil
}
```
可以看到ctbc這銀行在領錢的時候會多一層手續費，多一個```fee```屬性，並在```Withdraw()```方法的實作內會多扣掉這個```fee```

讓我們回到```main.go```裡面去看，同樣對兩間銀行做存錢領錢的動作，帳戶會剩多少錢

```
//main.go
func main() {

	myAccounts := []BankAccount{
		NewEsunAccount(),
		NewCtbcAccount(),
	}

	for _, account := range myAccounts {
		//存錢
		account.Deposit(100)
		//領錢
		if err := account.Withdraw(50); err != nil {
			fmt.Printf("ERR: %d\n", err)
		}
		//印出帳戶剩多少錢
		fmt.Printf("%s balance: %d\n", account.GetName(), account.GetBalance())
	}
}
```
執行結果：
```
esunAccount balance: 50
ctbcAccount balance: 35
```
也可以這樣寫：
```
//main.go
func ShowAccountName(m BankAccount) {
	m.GetName()
}

func main() {
	var m BankAccount
	esun := NewEsunAccount()
	ctbc := NewCtbcAccount()
	m = esun
	ShowAccountName(m)
	m = ctbc
	ShowAccountName(m)
}
```
執行結果：
```
Account Name= esunAccount
Account Name= ctbcAccount
```
這樣的做法，也展現了介面可以實現了**多型**的行為

## 7.3 空介面
Go程式的interface除了定義型態的行為，本身也是一種型態。而空介面則代表任意型態。

空介面宣告方式：
```
interface{}
```

例如下面的變數```i```的型態為empty interface，所以可以接收任意型態的值。
```
type BankAccount struct {
    accountName string
    balance     int
}

func printValueAndType(i interface{}) { // take empty interface as parameter
    fmt.Printf("value=%v, type=%T\n", i, i)
}

func main() {
    var i interface{} // declare var as empty interface type

    i = "abc"
    printValueAndType(i) // value=abc, type=string

    i = 123
    printValueAndType(i) // value=123, type=int

    i = BankAccount{"ctbc", 1000}
    printValueAndType(i) // value={ctbc 1000}, type=main.BankAccount
}
```
有沒有覺得空介面其實很像```泛型(generics)```

[Go 1.18](https://tip.golang.org/doc/go1.18#generics)加入的泛型(generics)新增了一個關鍵字```any```作為空介面```interface{}```的別名，所以之後改用```any```即可
```
m := make(map[int]any)
m["a"] = 123
m["b"] = "abc"
```
也可以看到最經典最常用標準函式庫的例子

**fmt package**

func Println
```
func Println(a ...any) (n int, err error)
```
所以其實平常用```Println()```印出什麼型別的東西都可以

範例：
```
func main() {
    fmt.Println(123)
    fmt.Println("abc")
    fmt.Println(EsunAccount{"esun", 100})
}
```
執行結果：
```
123
abc
{esun 100}
```

## 7.4 以介面為合約
上面有提到```Println```方法，這邊來看一下```fmt```的三個函式的源碼
```
package fmt
func Fprintf(w io.Writer, format string, a ...any) (n int, err error)

func Printf(format string, a ...any) (n int, err error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```
可以看到```Printf()```和```Sprintf()```都會呼叫```Fprintf()```方法，並以```io.Writer```介面當成```Fprintf```與呼叫方的合約

先來看一下```Writer```這個介面
```
type Writer interface {
	Write(p []byte) (n int, err error)
}
```
一方面合約要求呼叫方要有呼叫```Write```方法的具體型別，另一方面合約保證Fprintf在值符合io.Writer介面時會執行工作

那我們就來追看看被當成```w```傳入的參數```os.Stdout```，果然在```os```底下看到一個函式```Write```
```
// Write writes len(b) bytes to the File.
// It returns the number of bytes written and an error, if any.
// Write returns a non-nil error when n != len(b).
func (f *File) Write(b []byte) (n int, err error) {
        if err := f.checkValid("write"); err != nil {
                return 0, err
        }
}
```
(想要追更底層可以參考[link](https://ithelp.ithome.com.tw/articles/10218227))

所以任何有實作```io.Writer```介面的具體型別都可以當成實作替換的參數

## 7.5 介面嵌入介面
剛剛有提到```Writer```介面
現在再來看一下另外兩個
```
package io

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```
可以在介面裡嵌入另一個介面

來看```io```的另一個interface
```
type ReadWriter interface {
	Reader
	Writer
}
```
直接宣告介面的名稱即可

也可以直接宣告要嵌入介面的方法

舉例：
```
type ReadWriter interface {
	Read(p []byte) (n int, err error)
	Write(p []byte) (n int, err error)
}
```
甚至混合兩種風格
```
type ReadWriter interface {
	Read(p []byte) (n int, err error)
	Writer
}
```

## 7.6 介面值
介面值有分為**型別**和**值**兩個元件

介面零值的型別和值皆為nil

這裡再舉例一個fmt套件庫的重要介面
```
package fmt

type Stringer interface {
	String() string
}
```
String方法用於輸出，接受任何格式的或像**Print函數**印出來為格式化的值

這邊我實作了兩個不同型別```Temp```,```Point```來實作**Stringer**介面
```
type Temp int

func (t Temp) String() string {
	return strconv.Itoa(int(t)) + "°C"
}

type Point struct {
	x, y int
}

func (p *Point) String() string {
	return fmt.Sprintf("%d, %d", p.x, p.y)
}

```
再來我們回到了主題來探討介面值

宣告介面並指派具體的型別，來看看各種情況下的介面型別與值
```
func main() {
	var x fmt.Stringer
	fmt.Printf("value=%v, type=%T\n", x, x) //value=<nil>, type=<nil>

	x = Temp(27)
	fmt.Printf("value=%v, type=%T\n", x, x) //value=27°C, type=main.Temp

	x = &Point{1, 2}
	fmt.Printf("value=%v, type=%T\n", x, x) //value=1, 2, type=*main.Point

	x = (*Point)(nil)
	fmt.Printf("value=%v, type=%T\n", x, x) //value=<nil>, type=*main.Point
}
```

**既然有值，就可以比較**

介面值可以用```==``` ```!=```比較

- 兩介面值若型別相同才可以比較
- 動態值均為nil或是動態型別相同且動態值根據其行為相等則介面值相等時相等時，```==```才成立

範例：
```
func main() {
	//實作AnotherTemp，並且與Temp行為相同
	var x, y fmt.Stringer

	fmt.Printf("%v %v\n", x, y) //印出兩者的介面值
	fmt.Println(x == y)         //型別相同且動態值相同
	fmt.Println("--------------")
	x = Temp(27)
	y = AnotherTemp(27)
	fmt.Printf("%v %v\n", x, y)
	fmt.Println(x == y)  //動態值相同但動態型別不同
	fmt.Println("--------------")
	y = Temp(27) //把y指派給Temp型別
	fmt.Printf("%v %v\n", x, y)
	fmt.Println(x == y) //動態型別和動態值皆相同
}
```
執行結果：
```
<nil> <nil>
true
--------------
27°C 27°C
false
--------------
27°C 27°C
true
```
⚠️特別注意：有些型別完全不能比較(slice, map與函式)

舉例：
```
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: []int型別不能比較
