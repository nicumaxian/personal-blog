---
title: Implementing IsASCII method. Benchmark.
date: 2018-11-01
---

These days I needed a realisation of string ASCII validator. After few google attempts, I found a single constant `unicode.MaxASCII` in Go SDK, which could help to resolve this problem.
To be sure, I asked on [stackoverflow](https://stackoverflow.com/questions/53069040/checking-a-string-contains-only-ascii-characters). 

Since we were discussing about performance, user [peterSO](https://stackoverflow.com/users/221700/peterso) did a benchmark, which had a really interesting numbers.
I decided to go forward and create more implementations. Source code can be found [here](https://github.com/nicumaxian-ellation/benchmarks/tree/master/isascii).

## Implementations

### Using Go RANGE

This method uses a `range` iteration over a string. It's one of the cleaner solutions in this topic.
```go
func IsASCIIRange(s string) bool {
	for _, c := range s {
		if c > unicode.MaxASCII {
			return false
		}
	}

	return true
}
```

### Using classic for-loop

This is a trivial iteration over string. Which will make a really big difference.
```go
func IsASCIIClassic(s string) bool {
	for i := 0; i < len(s); i++ {
		if s[i] > unicode.MaxASCII {
			return false
		}
	}

	return true
}
```

### Using regex

Regex. Nothing more to say. Cool, but not always.
```go
var asciRegex = regexp.MustCompile(`^[\x00-\x7F]*$`)

func IsASCIIRegex(s string) bool {
	return asciRegex.MatchString(s)
}
```

### Using Rune

Using `strings.IndexFunc` we can find first character which satisfy condition of `unicode`.
```go
func IsASCIIRune(s string) bool {
	nonASCIIChar := func(r rune) bool {
		return r > unicode.MaxASCII
	}

	return strings.IndexFunc(s, nonASCIIChar) == -1
}
```

## Benchmark

```text
goos: darwin
goarch: amd64
pkg: github.com/nicumaxian-ellation/benchmarks/isascii
BenchmarkIsASCIIRange-8     	50000000	        35.8 ns/op
BenchmarkIsASCIIClassic-8   	100000000	        19.2 ns/op
BenchmarkIsASCIIRegex-8     	 2000000	       987 ns/op
BenchmarkIsASCIIRune-8      	20000000	       104 ns/op
PASS
ok  	github.com/nicumaxian-ellation/benchmarks/isascii	8.965s
```

## Conclusion

We didn't usually thing on such small things. Actually, small things matters. There is a big performance difference between `Range` and `classic for` solutions.
The diff between these two is `~1.86%`. Which as you see, is quite big. 

If you're working with a big amount of data, than you obviously should consider it.
For a small amount of data, I would choose cleanest solution. For me it was solution #1 (`using range`).
