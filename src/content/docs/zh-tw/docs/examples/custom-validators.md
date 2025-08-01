---
title: "自訂驗證器"
---

也可以註冊自訂驗證器。請參閱[範例程式碼](examples/custom-validation/server.go)。

```go
package main

import (
  "net/http"
  "reflect"
  "time"

  "github.com/gin-gonic/gin"
  "github.com/gin-gonic/gin/binding"
  "github.com/go-playground/validator/v10"
)

// Booking 包含已綁定和已驗證的資料。
type Booking struct {
  CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
  CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn,bookabledate"
    time_format:"2006-01-02"`
}

func bookableDate(
  v *validator.Validate, topStruct reflect.Value, currentStructOrField reflect.Value,
  field reflect.Value, fieldType reflect.Type, fieldKind reflect.Kind, param string,
) bool {
  if date, ok := field.Interface().(time.Time); ok {
    today := time.Now()
    if today.Year() > date.Year() || today.YearDay() > date.YearDay() {
      return false
    }
  }
  return true
}

func main() {
  route := gin.Default()

  if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
    v.RegisterValidation("bookabledate", bookableDate)
  }

  route.GET("/bookable", getBookable)
  route.Run(":8085")
}

func getBookable(c *gin.Context) {
  var b Booking
  if err := c.ShouldBindWith(&b, binding.Query); err == nil {
    c.JSON(http.StatusOK, gin.H{"message": "預訂日期有效！"})
  } else {
    c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
  }
}
```

```sh
$ curl "localhost:8085/bookable?check_in=2018-04-16&check_out=2018-04-17"
{"message":"預訂日期有效！"}

$ curl "localhost:8085/bookable?check_in=2018-03-08&check_out=2018-03-09"
{"error":"Key: 'Booking.CheckIn' Error:Field validation for 'CheckIn' failed on the 'bookabledate' tag"}
```

也可以用這種方式註冊[結構層級的驗證](https://github.com/go-playground/validator/releases/tag/v8.7)。
請參閱 [struct-lvl-validation 範例](examples/struct-lvl-validations)以了解更多資訊。
