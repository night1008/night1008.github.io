写业务过程中，经过会遇到当数据不存在则创建记录，已存在则更新记录的情况，

一开始使用一下代码中的第一种方式，没发现没生效，后面改成第二种才生效，在此记录一下。

```go
package main

import (
	"log"

	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

type User struct {
	ID    uint64 `gorm:"primaryKey"`
	Email string `gorm:"uniqueIndex"`
	Name  string
	Age   int64
}

func main() {
	db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
	if err != nil {
		log.Fatal(err)
	}
	if err := db.AutoMigrate(&User{}); err != nil {
		log.Fatal(err)
	}

	user := User{
		Email: "demo@qq.com",
		Name:  "demo111",
		Age:   211,
	}

	// 错误写法
	if err := createOrUpdate1(db, &user); err != nil {
		log.Fatal(err)
	}

	// 正确写法
	if err := createOrUpdate2(db, &user); err != nil {
		log.Fatal(err)
	}
}

// 错误写法
func createOrUpdate1(db *gorm.DB, user *User) error {
	result := db.Where("email = ?", user.Email).FirstOrCreate(&user)
	if result.Error != nil {
		return result.Error
	}

	// 此时 user 的字段会更新城数据库里面的值，导致下面语句无效执行
	if result.RowsAffected == 0 {
		if err := db.Save(&user).Error; err != nil {
			return err
		}
	}
	return nil
}

// 正确写法
func createOrUpdate2(db *gorm.DB, user *User) error {
	if err := db.Where("email = ?", user.Email).
		Assign(User{Name: user.Name, Age: user.Age}).
		FirstOrCreate(&user).Error; err != nil {
		return err
	}
	return nil
}
```