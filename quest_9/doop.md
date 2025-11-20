# Квест IX. doop

## Название файла (go)

```
doop.go
```

---

## Что нужно сделать — Instruction

Написать программу `doop`, которая принимает **три аргумента**:

1. Число (`int64`)
2. Оператор (`+`, `-`, `*`, `/`, `%`)
3. Число (`int64`)

Программа должна:

* Выполнить арифметическую операцию над числами.
* Обрабатывать переполнения и деление на ноль.
* При неверном операторе, неправильном количестве аргументов или переполнении — ничего не выводить.
* Для деления на 0 выводить: `"No division by 0\n"`.
* Для деления по модулю на 0 выводить: `"No modulo by 0\n"`.

Пример использования:

```bash
go run doop.go 42 + 58
# 100

go run doop.go 10 / 0
# No division by 0
```

---

## Решение

```go
package main

import "os"

// Константы для int64
const (
	MaxInt64 = 1<<63 - 1
	MinInt64 = -1 << 63
)

// writeString выводит строку s в stdout
func writeString(s string) {
	os.Stdout.Write([]byte(s))
}

// writeInt64 печатает int64 и перевод строки
func writeInt64(n int64) {
	if n == 0 {
		writeString("0\n")
		return
	}
	if n < 0 {
		os.Stdout.Write([]byte{'-'})
		n = -n
	}
	var buf [20]byte
	i := len(buf)
	for n > 0 {
		i--
		buf[i] = byte('0' + n%10)
		n /= 10
	}
	os.Stdout.Write(buf[i:])
	os.Stdout.Write([]byte{'\n'})
}

// parseInt64 парсит строку в int64
// возвращает (значение, ok). ok=false при ошибке или переполнении
func parseInt64(s string) (int64, bool) {
	if len(s) == 0 {
		return 0, false
	}
	i := 0
	sign := int64(1)
	if s[0] == '-' {
		sign = -1
		i++
		if i == len(s) {
			return 0, false
		}
	}
	var val int64 = 0
	for ; i < len(s); i++ {
		ch := s[i]
		if ch < '0' || ch > '9' {
			return 0, false
		}
		d := int64(ch - '0')
		if val > (MaxInt64-d)/10 {
			return 0, false
		}
		val = val*10 + d
	}
	if sign < 0 {
		val = -val
	}
	return val, true
}

// safeAdd возвращает (a+b, ok), ok=false при переполнении
func safeAdd(a, b int64) (int64, bool) {
	if b > 0 && a > MaxInt64-b {
		return 0, false
	}
	if b < 0 && a < MinInt64-b {
		return 0, false
	}
	return a + b, true
}

// safeSub возвращает (a-b, ok)
func safeSub(a, b int64) (int64, bool) {
	return safeAdd(a, -b)
}

// safeMul возвращает (a*b, ok)
func safeMul(a, b int64) (int64, bool) {
	if a == 0 || b == 0 {
		return 0, true
	}
	if a == MinInt64 && b == -1 {
		return 0, false
	}
	if b == MinInt64 && a == -1 {
		return 0, false
	}
	absA := a
	if absA < 0 {
		absA = -absA
	}
	absB := b
	if absB < 0 {
		absB = -absB
	}
	if absA > MaxInt64/absB {
		return 0, false
	}
	return a * b, true
}

func main() {
	if len(os.Args) != 4 {
		return
	}
	aStr := os.Args[1]
	op := os.Args[2]
	bStr := os.Args[3]

	a, okA := parseInt64(aStr)
	b, okB := parseInt64(bStr)
	if !okA || !okB {
		return
	}

	switch op {
	case "+":
		if res, ok := safeAdd(a, b); ok {
			writeInt64(res)
		}
	case "-":
		if res, ok := safeSub(a, b); ok {
			writeInt64(res)
		}
	case "*":
		if res, ok := safeMul(a, b); ok {
			writeInt64(res)
		}
	case "/":
		if b == 0 {
			writeString("No division by 0\n")
			return
		}
		if a == MinInt64 && b == -1 {
			return
		}
		writeInt64(a / b)
	case "%":
		if b == 0 {
			writeString("No modulo by 0\n")
			return
		}
		writeInt64(a % b)
	default:
		return
	}
}
```

---

## Разбор решения

### 1. Парсинг аргументов

* `parseInt64` проверяет, что строка — валидное число `int64` и нет переполнения.
* Если число некорректно — программа ничего не выводит.

### 2. Безопасные операции

* `safeAdd`, `safeSub`, `safeMul` — предотвращают переполнение для сложения, вычитания и умножения.
* Для деления и модуля проверяется деление на 0.
* Для MinInt64 / -1 предотвращается переполнение.

### 3. Вывод

* `writeInt64` печатает число с переходом строки.
* `writeString` используется для сообщений об ошибке (деление на 0).

### 4. Обработка операторов

Используется `switch` для выбора операции. Если оператор неизвестен — программа ничего не выводит.

---

## Учебный материал

| Термин                      | Объяснение                                                |
| --------------------------- | --------------------------------------------------------- |
| **int64**                   | 64-битное целое число.                                    |
| **Overflow / переполнение** | Когда результат операции выходит за пределы int64.        |
| **os.Args**                 | Аргументы командной строки.                               |
| **switch**                  | Конструкция выбора для нескольких вариантов.              |
| **return**                  | Завершает функцию. Для main это завершает программу.      |
| **const**                   | Определение констант MaxInt64 и MinInt64.                 |
| **byte / []byte**           | Используется для записи в stdout через `os.Stdout.Write`. |

---

## Заключение

Программа `doop` реализует безопасный калькулятор командной строки для `int64`, с проверкой переполнений и деления на ноль. Она использует низкоуровневые функции вывода для прямого контроля над форматом вывода и полностью соответствует условию:

* Три аргумента
* Операторы `+`, `-`, `*`, `/`, `%`
* Обработка ошибок и переполнений