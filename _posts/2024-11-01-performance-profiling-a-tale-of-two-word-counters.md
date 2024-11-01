# Performance Profiling in Go: A Tale of Two Word Counters

## The Spark of Curiosity

It all started with a code review. While reading through someone's implementation of a word counter that used `regex.MustCompile`, I had that familiar gut feeling that many developers know well: "There must be a better way." The code worked, but something didn't sit right. Go's standard library provides `bufio.Scanner` with `ScanWords`, which seemed like it would be more memory efficient.

But here's the thing about gut feelings in software engineering - they need to be validated with data.

## The Hypothesis

My initial hypothesis was straightforward:
- `bufio.Scanner` with `ScanWords` would be more memory efficient
- The regex-based approach would likely have more overhead
- The performance difference would be significant

Like any good engineer, I decided to put these assumptions to the test.

## The Implementation

Let's look at both approaches:

```go
// Approach 1: Scanner-based
func useScanLn(file string) (int, error) {
    scanner := bufio.NewScanner(f)
    scanner.Split(bufio.ScanWords)
    var wordNumber int
    for scanner.Scan() {
        wordNumber++
    }
    return wordNumber, nil
}

// Approach 2: Regex-based
func useRegex(file string) (int, error) {
    reader := bufio.NewReader(f)
    reg := regexp.MustCompile(`\S+`)
    for {
        line, err := reader.ReadString('\n')
        words := reg.FindAllString(line, -1)
        wordNumber += len(words)
        // ... error handling
    }
    return wordNumber, nil
}
```

## The Benchmark Setup

Our investigation used a carefully crafted benchmark command:

```bash
go test -bench . -count 5 -benchtime=10x -benchmem
```

Let's break down this command:
- `-bench .` - Run all benchmarks in the current package
- `-count 5` - Run the entire benchmark suite 5 times
- `-benchtime=10x` - Run each benchmark exactly 10 times within each suite run
- `-benchmem` - Include memory allocation statistics

This setup provides:
1. **Multiple Levels of Sampling**
   - 10 iterations per benchmark
   - 5 complete suite runs
   - 50 total samples per implementation
2. **Comprehensive Metrics**
   - Operation timing
   - Memory allocation
   - System resource usage

## The Results

To ensure the benchmark results were realistic, the tests ran on a substantial dataset: **2,500 files** totalling **2.5 million lines of data**. This large dataset offered a realistic measure of the performance and resource impact of each implementation under considerable load. It provided insights into how each approach scales and highlighted nuances that might only appear with this scale of data processing.

### Scanner-based Implementation:
```
BenchmarkReadWords-4   	10	301467 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	259073 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	246092 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	296075 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	340974 ns/op	22197 B/op	403 allocs/op
PASS
ok  	github.com/GoWild/readWord	21.968s
```

### Regex-based Implementation:
```
BenchmarkReadWords-4   	10	299295 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	286157 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	282313 ns/op	22226 B/op	403 allocs/op
BenchmarkReadWords-4   	10	325948 ns/op	22197 B/op	403 allocs/op
BenchmarkReadWords-4   	10	280714 ns/op	22216 B/op	403 allocs/op
PASS
ok  	github.com/GoWild/readWord	120.254s
```

## Breaking Down the Data

### 1. Per-Operation Performance
The Scanner-based implementation achieved an average of 288,736 nanoseconds per operation, with execution times ranging from 246,092 to 340,974 nanoseconds, resulting in a standard deviation of approximately 37,000 nanoseconds. In comparison, the Regex-based implementation averaged 294,885 nanoseconds per operation, with a tighter range of 280,714 to 325,948 nanoseconds and a lower standard deviation of around 18,000 nanoseconds.
### 2. Memory Usage
Both implementations demonstrated nearly identical memory usage patterns, consuming approximately 22,200 bytes per operation with 403 allocations per run. Memory variation across runs was minimal, highlighting consistency in memory consumption between the two approaches.
### 3. The Critical Discovery: Total Execution Time
The **Scanner-based implementation completed the benchmark suite in 21.968 seconds**, while **the Regex-based implementation took significantly longer at 120.254 seconds**—**approximately 5.5 times slower** overall.

| **Metric**                     | **Scanner Implementation** | **Regex Implementation** |
|--------------------------------|----------------------------|--------------------------|
| **Mean Time (ns/op)**          | 288,736                    | 294,885                  |
| **Range (ns/op)**              | 246,092 - 340,974         | 280,714 - 325,948       |
| **Standard Deviation**         | ~37,000 ns/op             | ~18,000 ns/op           |
| **Memory Usage (bytes/op)**    | ~22,200                   | ~22,200                 |
| **Allocations per Op**         | 403                        | 403                      |
| **Total Execution Time (s)**   | 21.968                     | 120.254                  |
| **Execution Time Difference**  | —                          | ~5.5x slower             |

## Key Insights
### 1. Initialization Costs
The Regex implementation incurs significant setup overhead, as pattern compilation and optimization take time. These initialization costs are hidden in per-operation metrics but impact total execution time.
### 2. System-Level Impact
While per-operation performance appears similar, the Regex implementation likely incurs more background overhead, leading to a greater impact on system resources over extended use.
### 3. Deployment Implications
For short-lived processes, the Scanner implementation is clearly advantageous due to its faster startup time. However, in long-running services, this performance difference may diminish. For microservices and serverless environments, where efficient startup is crucial, the Scanner approach is typically the better choice.

## The Final Verdict

The Scanner-based implementation proves superior for most use cases:
- Similar per-operation performance
- Significantly faster startup time
- More predictable resource usage
- Better suited for modern deployment patterns

Unless specific regex functionality is required, the Scanner implementation is the clear winner, especially in contemporary containerized and serverless environments where efficiency and quick startup times are paramount.
## Conclusion

This journey from intuition to data-driven decision making exemplifies modern software engineering. While both implementations are valid, the Scanner-based approach proves superior in most contexts, not just for its slight performance advantage but for its simplicity, predictability, and efficiency in real-world deployment scenarios.

The most valuable outcome wasn't just proving one approach better, but understanding the nuanced ways in which different implementations can impact system performance. This kind of detailed analysis helps us make better engineering decisions and build more efficient systems.

---

Check out the code on my Github via this [link](https://github.com/TaskMasterErnest/GoWild/tree/main/readWord).