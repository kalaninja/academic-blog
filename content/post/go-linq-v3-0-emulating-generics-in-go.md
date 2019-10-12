+++
tags = ["generics", "go-linq", "golang", "linq"]
categories = ["Go"]
title = "go-linq v3.0: Emulating generics in Go"
date = 2017-02-24T20:37:00+03:00
draft = false
+++

About a month ago we welcomed a new developer to our go-linq maintainers team [cleitonmarx](https://github.com/cleitonmarx) (Cleiton Marques) who introduced an interesting pattern for emulating generics in Go and became the main show-maker of the third version of the library. Here is a small story of how the generics behavior was emulated in go-linq.

<!--more-->

## The problem

Being one of the most requested features in Go language for years, the question of generics has been discussed up and down and back and forth. For now it seems that they won't appear in the near future. In fact, the language designers aren't against generics, but they have not found or seen a good proposal that allows generics in the language without significantly complicating it. Anyway, we got used to living without generics. This resulted in a slight adjustment to our way of thinking and brought to life tons of elegant solutions. But sometimes there are situations when generics are indispensable. For example, in the go-linq library methods are intended to be used with delegates of any data type, so we made them accept functions of empty interfaces. This interface has no particular behavior, hence objects with any behavior satisfy this interface and the end user of the library is responsible for doing the right type casts. Despite its good performance, this approach isn't use-of-wrong-data-type proof and looks clumsy. Consider the difference:
```go
From(cars).Where(func(c interface{}) bool {
    return c.(Car).year >= 2015
}).Select(func(c interface{}) interface{} {
    return c.(Car).owner
})
```
vs
```go
From(cars).Where(func(c Car) bool {
    return c.year >= 2015
}).Select(func(c Car) string {
    return c.owner
})
```
and it's getting worse depending on the complexity of your code. So, we introduced a pattern for emulating/improving generics-like methods in our library that uses the reflection mechanism internally and make your code look much easier to read and understand.

## The solution

The idea is to make a new method that wraps around the original one with empty interfaces and does all the casts automatically using reflection. The second part of the solution comes from the fact that the signature to the new method is `func (q Query) WhereT(predicateFn interface{}) Query` , so anything can be specified as the argument and we need a way to validate it.

First, let's define a structure to store all the necessary data:
```go
type functionCache struct {
	MethodName string
	ParamName  string
	FnValue    reflect.Value
	FnType     reflect.Type
	TypesIn    []reflect.Type
	TypesOut   []reflect.Type
}

type genericFunc struct {
	Cache *functionCache
}

func (g *genericFunc) Call(params ...interface{}) interface{} {
 paramsIn := make([]reflect.Value, len(params))
 for i, param := range params {
 paramsIn[i] = reflect.ValueOf(param)
 }
 paramsOut := g.Cache.FnValue.Call(paramsIn)
 if len(paramsOut) >= 1 {
 return paramsOut[0].Interface()
 }
 return nil
}
```
`MethodName` and `ParamName` are used in error handling to form a reasonable message. `TypesIn` and `TypesOut` store respectively the input and output parameters of the specified function.

Secondly, we need a factory method for the 'genericFunc':
```go
func newGenericFunc(methodName, paramName string,
	fn interface{},
	validateFunc func(*functionCache) error) (*genericFunc, error) {

	cache := &functionCache{}
	cache.FnValue = reflect.ValueOf(fn)

	if cache.FnValue.Kind() != reflect.Func {
		return nil,
			fmt.Errorf("%s: parameter [%s] is not a function type. It is a '%s'",
				methodName, paramName, cache.FnValue.Type())
	}
	cache.MethodName = methodName
	cache.ParamName = paramName
	cache.FnType = cache.FnValue.Type()
	numTypesIn := cache.FnType.NumIn()
	cache.TypesIn = make([]reflect.Type, numTypesIn)
	for i := 0; i < numTypesIn; i++ {
		cache.TypesIn[i] = cache.FnType.In(i)
	}

	numTypesOut := cache.FnType.NumOut()
	cache.TypesOut = make([]reflect.Type, numTypesOut)
	for i := 0; i < numTypesOut; i++ {
		cache.TypesOut[i] = cache.FnType.Out(i)
	}
	if err := validateFunc(cache); err != nil {
		return nil, err
	}

	return &genericFunc{Cache: cache}, nil
}
```

it populates the `genericFunc` structure and validates it. The validation function must be provided by the user of the method, but we have a helper to produce it:
```go
func simpleParamValidator(In []reflect.Type, Out []reflect.Type) func(cache *functionCache) error {
	return func(cache *functionCache) error {
		var isValid = func() bool {
			if In != nil {
				if len(In) != len(cache.TypesIn) {
					return false
				}
				for i, paramIn := range In {
					if paramIn != genericTp && paramIn != cache.TypesIn[i] {
						return false
					}
				}
			}
			if Out != nil {
				if len(Out) != len(cache.TypesOut) {
					return false
				}
				for i, paramOut := range Out {
					if paramOut != genericTp && paramOut != cache.TypesOut[i] {
						return false
					}
				}
			}
			return true
		}

		if !isValid() {
			return fmt.Errorf(
				"%s: parameter [%s] has a invalid function signature. Expected: '%s', actual: '%s'",
				cache.MethodName,
				cache.ParamName,
				formatFnSignature(In, Out),
				formatFnSignature(cache.TypesIn, cache.TypesOut))
		}
		return nil
	}
}
```

The last thing we need is the wrapper around the original method that does all the magic. For the above-mentioned `Where` method it is as simple as:
```go
func (q Query) WhereT(predicateFn interface{}) Query {

	predicateGenericFunc, err := newGenericFunc(
		"WhereT", "predicateFn", predicateFn,
		simpleParamValidator(newElemTypeSlice(new(genericType)), newElemTypeSlice(new(bool))),
	)
	if err != nil {
		panic(err)
	}

	predicateFunc := func(item interface{}) bool {
		return predicateGenericFunc.Call(item).(bool)
	}

	return q.Where(predicateFunc)
}
```

So, you provide a function with your custom types to a new `WhereT` method and it produces a wrapper with empty interfaces that will suffice for the original method `Where`.

## The test

If you are writing unit tests on your project and are pursuing a 100% coverage level, as we do in go-linq, then it is not an easy task to achieve with this pattern. After trying several approaches we ended up with writing several tests:

*   one for the right case when everything is ok
*   and the second that tests the validation, recovers the panic and checks the error message.

```go
func TestWhereT_PanicWhenPredicateFnIsInvalid(t *testing.T) {
	mustPanicWithError(
		t,
		"WhereT: parameter [predicateFn] has a invalid function signature. Expected: 'func(T)bool', actual: 'func(int)int'",
		func() {
			From([]int{1, 1, 1, 2, 1, 2, 3, 4, 2}).WhereT(func(item int) int { return item + 2 })
		})
}

func mustPanicWithError(t *testing.T, expectedErr string, f func()) {
	defer func() {
		r := recover()
		err := fmt.Sprintf("%s", r)
		if err != expectedErr {
			t.Fatalf("got=[%v] expected=[%v]", err, expectedErr)
		}
	}()
	f()
}
```

## The downsides

The first and probably the main downside of this solution is the performance penalty. Running the same scenarios that I used in one of the previous articles [Manipulating data with iterators in Go](/2016/07/16/manipulating-data-with-iterators-in-go/) we can see that this approach is around 5x-10x slower. That is why we decided to provide users with both versions of the methods leaving the original as is and calling the new ones by the _-T_ naming convention.

```
BenchmarkSelectWhereFirst-4              5000000               352 ns/op
BenchmarkSelectWhereFirst_generics-4     1000000              2094 ns/op
BenchmarkSum-4                                50          37427788 ns/op
BenchmarkSum_generics-4                        3         360509066 ns/op
BenchmarkZipSkipTake-4                  10000000               224 ns/op
BenchmarkZipSkipTake_generics-4          1000000              1265 ns/op
```
The second issue comes from the signature of the _-T method_, as you can see it accepts an empty interface, i.e. `predicateFn` can be literally anything.
```go
decode:true">func (q Query) WhereT(predicateFn interface{}) Query
```

Of course, a user of the method can have problems understanding what `predicateFn` should look like, especially if he is new to the library. We conducted a long discussion to deal with the problem and ended up with the idea that the best we can do is to provide really good documentation and examples of each method (thanks [Cleiton](http://www.cleitonmarques.com/) for being so patient and hardworking).

## Summary

Do I need generics in Go? Not really, especially with the new [go:generate](https://blog.golang.org/generate) feature. But if you still need them, there is always a way to emulate generic behavior even though it isn't supported by the language, either by using empty interfaces and maintaining good performance, or with the pattern shown above making the code cleaner, more readable and free of type assertions. **Note.** The full source code is available in [genericfunc.go](https://github.com/ahmetalpbalkan/go-linq/blob/master/genericfunc.go) in our repository. Please, feel free to ask any questions.