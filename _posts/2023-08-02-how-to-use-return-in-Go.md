# How To Use The Return Keyword in Go

- In simple terms, `return` is a keyword in Go that is used to end the execution of a function and return a value to the caller of the function.
- When a function is executed, it may perform some computations or modify some data, but at some point, it may need to provide a result back to the caller of the function. This is where `return` comes in.
- To use `return` in Go, you simply write the keyword `return` followed by the value that you want to return. Here's an example:
	```Go
	func add(x, y int) int {
    return x + y
	}
	```
	- In this example, we define a function called `add` that takes two integer arguments `x` and `y`. The function returns the sum of `x` and `y` using the `return` keyword followed by the expression `x + y`

- You can also use `return` to exit a function early if some condition is met. For example:
	```Go
	func divide(x, y float64) (float64, error) {
	    if y == 0 {
	        return 0, fmt.Errorf("division by zero")
	    }
	    return x / y, nil
	}
	```
  - In this example, we define a function called `divide` that takes two float64 arguments `x` and `y`. The function checks if `y` is zero, and if it is, it returns an error using the `return` keyword followed by the error message.
	- If `y` is not zero, the function returns the result of dividing `x` by `y` using the `return` keyword followed by the expression `x / y` and `nil` to indicate that there is no error.


![lets-go-fishing](https://th.bing.com/th/id/R.a801c5e00d5fb915e939a5c30385b941?rik=7ygoHJl4WuKeJQ&riu=http%3a%2f%2fwww.animatedimages.org%2fdata%2fmedia%2f157%2fanimated-fishing-image-0131.gif&ehk=rQq%2bd2QDkwjFPyX1hPoyHfP%2bFC7iis9SSANWH6%2bO9dk%3d&risl=&pid=ImgRaw&r=0)
