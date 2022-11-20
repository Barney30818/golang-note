# Ch7 interface 介面

### Agenda
- 7.1 何謂介面
- 7.2 介面基本用法
- 7.3 空介面
- 7.4 以介面為合約

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
		balance: 0,
	}
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
	fmt.Printf("EsunBank balance: %d\n", esun.GetBalance())
}
```
執行結果：
```
EsunBank balance: 300
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
		balance: 0,
		fee:     15,
	}
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
