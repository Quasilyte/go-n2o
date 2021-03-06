# n2o: nitrous boost for Go

## Quick overview

You start by writing ordinary Go code.
Then you spot some execution path that requires optimizations, the hot one.

```go
func array8sum(xs *[8]int) int {
	total := 0
	for _, x := range xs {
		total += x
	}
	return total
}
```

Suppose you think that loop unrolling might help.
Instead of doing this:

```go
func array8sum(xs *[8]int) int {
	total := 0
	total += xs[0]
	total += xs[1]
	total += xs[2]
	total += xs[3]
	total += xs[4]
	total += xs[5]
	total += xs[6]
	total += xs[7]
	return total
}
```

You can do this:

```go
func array8sum(xs *[8]int) int {
	total := 0
	//n2o: unroll
	for _, x := range xs {
		total += x
	}
	return total
}
```

And you get best of two worlds:
1. Readability of the initial version.
2. Performance of the optimized (unrolled) code.

There are other optimizations that can be performed by the `n2o`.
For example, you may ask to apply all sensible optimizations to the function:

```go
// Can also use func/size to optimize for code size.
//n2o: func/speed
func array8sum(xs *[8]int) int {
	total := 0
	for _, x := range xs {
		total += x
	}
	return total
}
```

Read docs to know more about supported directives and their meaning.

Basically, you write Go and then improve performance by optimizer hints, instead of doing dirty work by yourself.

## Optimizations

Please note that some optimizations may be dangerous when used inappropriately.

For example, one should not use `deadcode` when unsure that such code path is
never executed in the production environment.
This is as dangerous as turning off asserts in C and alike, caveat emptor.

For your convenience, such unsafe optimizations are listed in a separate list.

### Statement/expression local

Some optimizations are applied to the particular statement or expression:

**Safe optimizations**:

* `inline` - force function inlining at the specified call site.
* `unroll` - unroll a whole loop or make it execute several steps at each iteration.

**Unsafe optimizations**:

* `deadcode` - replace expression or statement that is believed to be "dead" during execution.

### Function attributes

On the function level, there are various optimization attributes that affect
whole function body.

```go
//n2o: inline
// Force function inlining at the every call site, even if `gc` compiler
// does not consider function as inlineable candidate.
// If possible, n2o may transform function body to make function
// inlineable by `gc` itself, otherwise it does inlining on its own.
func example() {}
```

There are also `speed` and `size` optional attributes that can't be used together:

```go
//n2o: size
// Try to make function more compact by applying various optimizations to its body.
func optimizedForSize() {}

//n2o: speed
// Try to make function run faster by applying various optimizations to its body.
func optimizedForExecutionTime() {}
```
