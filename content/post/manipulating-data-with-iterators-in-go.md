+++
tags = ["go-linq", "go2linq", "golang", "iterator", "linq"]
categories = ["Go"]
title = "Manipulating Data With Iterators in Go"
date = 2016-07-16T23:53:00+03:00
draft = false
+++

Several months ago I started learning Go language and came across an interesting library [go-linq](https://github.com/ahmetalpbalkan/go-linq) which is an implementation of Microsoft’s [LINQ](http://msdn.microsoft.com/en-us/library/bb397926.aspx) technology in Go. And while it is a good library I find its performance to be really weak because of its design. Trying to improve the situation resulted in a complete rewrite using iterators in a lightweight and simple manner.
<!--more-->

## go-linq

The key type there is Query:

```go
type Query struct {
	values []T
	err    error
}
```

It stores all the data in its `values` field and is a receiver for all the methods defined to manipulate data. Each time a method is executed it generates a new Query with a new `values` slice. So, for example, a Where() method can be implemented as simply as:

```go
func (q Query) Where(f func(T) (bool, error)) (r Query) {
	for _, i := range q.values {
		ok, err := f(i)
		if err != nil {
			r.err = err
			return r
		}
		if ok {
			r.values = append(r.values, i)
		}
	}
	return
}
```

And although it generally is a good idea, it has poor performance for several reasons. First, each method allocates a new slice, so when you chain methods (as you normally do in linq), like `From(slice).Where(wat).Select(dat)` , a lot of slices are created. This results in redundant memory traffic and additional garbage collection. Secondly, since it is a push model, each method manipulates the whole collection, even if there is only a single element needed in the end, e. g.  `From(slice).Where(wat).First()` . In this case the `wat`  predicate is executed for each element in the slice even if First() takes a single item.

## Iterator approach

In order to get rid of the problems shown above, I decided to rewrite the library from scratch using the iterator pattern. Iterator is a design pattern which is used to traverse a container and access the container's elements. It decouples algorithms from containers, which is exactly what is needed to achieve a pull model, so that the predicate in the previous example can be executed only for the elements that are really needed. According to the Gang of Four iterator prescribes the following interface for iteration over a container with elements of type T:

```java
interface Iterator {
    void First();    // Restart iteration
    void Next();     // Advance to next item
    bool IsDone();   // Are we done yet?
    T CurrentItem(); // Get current item
}
```

In C# (where LINQ originated) iterator is called an Enumerator and implements the following interface:

```cs
public interface IEnumerator
{
    object Current { get; }
    bool MoveNext();
    void Reset();
}
```

But as Go is about simplicity, I wanted my iterators to be as lightweight as possible. So decided to omit the Reset() method (as it is not really needed in LINQ) and benefit from Go's ability to return multiple values. So I ended with the following pattern:

```go 
type Iterator func() (item interface{}, ok bool)

type Query struct {
    Iterate func() Iterator
}

next := query.Iterate()
for item, ok := next(); ok; item, ok = next() {
    fmt.Println(item)
}
```

Now it became really simple to work with. Each time next() is called it returns the next element of the container and the boolean value indicating whether this element exists. Now the Where() method mentioned above can be rewritten in this way:

```go
func (q Query) Where(predicate func(interface{}) bool) Query {
	return Query{
		Iterate: func() Iterator {
			next := q.Iterate()

			return func() (item interface{}, ok bool) {
				for item, ok = next(); ok; item, ok = next() {
					if predicate(item) {
						return
					}
				}

				return
			}
		},
	}
}
```

Now Where() doesn't really process anything, instead it generates a new iterator that will manipulate data only when it is iterated.

## Benchmark

Rewriting each method to use iterators instead of slices increased  the performance dramatically. For example, talking about the previously mentioned case  `From(slice).Where(wat).First()` the source slice now is scanned only until the first element that satisfies the `wat` condition is found. Another good thing is that it allocates no additional memory. Lets make a simple benchmark comparing the iterators approach to go-linq's:

```go
const (
	size = 1000000
)

func BenchmarkSelectWhereFirst(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Range(1, size).Select(func(i interface{}) interface{} {
			return -i.(int)
		}).Where(func(i interface{}) bool {
			return i.(int) > -1000
		}).First()
	}
}

func BenchmarkSelectWhereFirst_golinq(b *testing.B) {
	for n := 0; n < b.N; n++ {
		golinq.Range(1, size).Select(func(a golinq.T) (golinq.T, error) {
			return -a.(int), nil
		}).Where(func(a golinq.T) (bool, error) {
			return a.(int) > -1000, nil
		}).First()
	}
}

func BenchmarkSum(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Range(1, size).Where(func(i interface{}) bool {
			return i.(int)%2 == 0
		}).SumInts()
	}
}

func BenchmarkSum_golinq(b *testing.B) {
	for n := 0; n < b.N; n++ {
		golinq.Range(1, size).Where(func(a golinq.T) (bool, error) {
			return a.(int)%2 == 0, nil
		}).Sum()
	}
}

func BenchmarkZipSkipTake(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Range(1, size).Zip(Range(1, size).Select(func(i interface{}) interface{} {
			return i.(int) * 2
		}), func(i, j interface{}) interface{} {
			return i.(int) + j.(int)
		}).Skip(2).Take(5)
	}
}

func BenchmarkZipSkipTake_golinq(b *testing.B) {
	for n := 0; n < b.N; n++ {
		golinq.Range(1, size).Zip(golinq.Range(11, size).Select(func(i golinq.T) (golinq.T, error) {
			return i.(int) * 2, nil
		}), func(i, j golinq.T) (golinq.T, error) {
			return i.(int) + j.(int), nil
		}).Skip(2).Take(5)
	}
}
```

The result of this benchmark on my machine (MacBookPro8,1 Intel Core i5 2,4 GHz):

```
BenchmarkSelectWhereFirst-4          3000000           561 ns/op         224 B/op         10 allocs/op
BenchmarkSelectWhereFirst_golinq-4         2     555810859 ns/op    120546360 B/op   2000085 allocs/op
BenchmarkSum-4                            20      73847428 ns/op     8000289 B/op    1000019 allocs/op
BenchmarkSum_golinq-4                      5     253731714 ns/op    69161392 B/op    1000053 allocs/op
BenchmarkZipSkipTake-4               5000000           351 ns/op         192 B/op          6 allocs/op
BenchmarkZipSkipTake_golinq-4              2     672403213 ns/op    144520824 B/op   3000075 allocs/op
```

## Resume

As you can see using this technique you can easily implement methods with lazy execution which seriously outperform a more straightforward and immediate execution. I have already implemented all the methods from traditional LINQ. This work can be found in my github repository [go2linq](https://github.com/kalaninja/go2linq). Feel free to use it, ask for improvements or implement new methods.