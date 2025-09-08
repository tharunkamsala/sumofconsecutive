# Gleam Parallel Perfect Square Finder

## What does this thing do?

Ever wondered about sequences of consecutive numbers where their squares add up to another perfect square? Like how 3² + 4² = 25 = 5²? This program finds all such sequences for you, and it does it really fast using multiple cores!

**The Math Problem:**
We're looking for starting points `s` where: `s² + (s+1)² + ... + (s+k-1)² = perfect square`

**Cool Examples:**
- 3² + 4² = 9 + 16 = 25 = 5² ✨
- 20² + 21² = 400 + 441 = 841 = 29² 🎯

## Why Gleam? Why not just Python?

Great question! I chose Gleam because:
- It compiles to Erlang and runs on the BEAM VM (super reliable)
- Built-in actor model for parallel processing (no thread headaches!)
- Type-safe but not verbose like Java
- Handles failures gracefully with the "let it crash" philosophy
- Can easily spawn thousands of lightweight processes

## Getting Started

### Prerequisites
You'll need:
- Erlang/OTP (version 27 or newer)
- Gleam compiler

### Installation

**macOS (using Homebrew):**
```bash
brew install erlang gleam
```

**Ubuntu/Debian:**
```bash
# Install Erlang first
sudo apt update
sudo apt install erlang-base erlang-dev

# Then install Gleam
wget https://github.com/gleam-lang/gleam/releases/latest/download/gleam-x86_64-unknown-linux-gnu.tar.gz
tar -xzf gleam-x86_64-unknown-linux-gnu.tar.gz
sudo mv gleam /usr/local/bin/
```

### Project Setup
```bash
git clone <this-repo>
cd lukas
gleam deps download
gleam build
```

## How to Use It

It's super simple! Just run:
```bash
cd lukas
gleam run -- <N> <k>
```

Where:
- `N` = upper limit to search up to
- `k` = how many consecutive numbers in each sequence

**Try these examples:**
```bash
# Small problem with detailed steps
gleam run -- 25 2

# Medium problem 
gleam run -- 2000 3

# Big problem (this will flex those CPU cores!)
gleam run -- 100000 4
```

## What Happens When You Run It

1. **Setup Phase**: Shows you the problem size and how many worker processes it's creating
2. **Performance Estimates**: Gives you a heads-up on expected timing
3. **Processing**: Workers crunch numbers in parallel
4. **Results**: 
   - For small problems (N ≤ 100): Shows every calculation step
   - For big problems: Just shows the summary and solutions found
5. **Metrics File**: Automatically saves detailed performance stats to a text file

## The Secret Sauce: How Parallelism Works

This is where it gets cool! The program uses Erlang's actor model:

```
       Boss Actor (coordinator)
            |
    ┌───────┼───────┐
    │       │       │
 Worker1  Worker2  Worker3  ...
```

**The Strategy:**
- Split the work into chunks (work units)
- Each worker gets a range to process
- Workers run completely independently 
- Boss collects all results when done

**Work Distribution Logic:**
```gleam
work_unit_size = case problem_size {
  n > 10000  -> 1000  // Big chunks for big problems
  n > 1000   -> 200   // Medium chunks
  _          -> 10    // Small chunks (forces more workers)
}
```

This ensures you get good parallelism even for smaller problems!

## Under the Hood: The Math Functions

### Core Functions

**`is_perfect_square(n)`** - Checks if a number is a perfect square
```gleam
// Uses floating point square root, then verifies with integers
// Example: is_perfect_square(25) -> True (because 5² = 25)
```

**`sum_of_squares(start, k)`** - Adds up k consecutive squares
```gleam
// Example: sum_of_squares(3, 2) -> 3² + 4² = 9 + 16 = 25
```

**`find_solutions_in_range(start, end, k)`** - The main algorithm
```gleam
// Tests every starting point in the range
// Returns list of valid starting points
```

### The Algorithm
It's basically a smart brute force approach:
1. For each possible starting point `s` from 1 to N
2. Calculate `s² + (s+1)² + ... + (s+k-1)²`
3. Check if the result is a perfect square
4. If yes, record `s` as a solution

The magic is in doing steps 1-4 across multiple CPU cores simultaneously!

## Performance Metrics That Matter

The program tracks several key metrics:

**CPU Time / Real Time Ratio** - This is the big one!
- Ratio > 2.0 = Great parallelism (multiple cores working hard)
- Ratio 1.1-2.0 = Some parallelism 
- Ratio ≤ 1.1 = Barely any parallelism (not good!)

**Other Cool Stats:**
- Real execution time
- Solutions found per second (throughput)
- Number of worker processes created
- Memory efficiency

## Output Files

Every run creates a detailed metrics file like `metrics_N2000_k3.txt` containing:
- All the performance statistics
- Complete list of solutions found
- System architecture details
- Timing breakdown

These files are saved in the project directory for later analysis.

## Examples in Action

**Small Problem (shows every step):**
```
$ gleam run -- 25 2

s = 1 → 1² + 2² = 1 + 4 = 5 (not a perfect square)
s = 2 → 2² + 3² = 4 + 9 = 13 (not a perfect square)  
s = 3 → 3² + 4² = 9 + 16 = 25 = 5² ✓ SOLUTION!
...
```

**Large Problem (summary mode):**
```
$ gleam run -- 10000 2

🚀 STARTING PARALLEL PROCESSING...
Workers: 10, Actors: 11 total
Expected CPU/Real ratio: ~8.5x

📋 RESULTS:
Found 13 perfect square solutions!
CPU/Real ratio: 8.2 (excellent parallelism!)
```

## Dependencies We Use

The project is pretty lightweight:
- `gleam_stdlib` - Basic Gleam functions
- `gleam_otp` - The actor model magic ⭐
- `gleam_erlang` - Erlang integration
- `argv` - Command line argument parsing
- `simplifile` - File operations for saving metrics

All managed through Gleam's package manager automatically!

## Technical Details (for the curious)

**Runtime Environment:**
- BEAM Virtual Machine (same as Erlang/Elixir)
- Preemptive scheduling across all CPU cores
- Fault-tolerant with supervisor trees
- Shared-nothing memory model per actor

**Performance Characteristics:**
- Successfully tested up to N=10,000,000 
- Scales linearly with problem size
- Minimal overhead from message passing
- Memory usage stays reasonable even with hundreds of workers

## Troubleshooting

**"Not enough parallelism" (ratio < 1.5):**
- Try a larger N value 
- Check if you have multiple CPU cores
- Make sure the work unit size isn't too large

**"Out of memory":**
- Reduce the work unit size in the code
- Try smaller N values first

**"Takes forever to run":**
- Start with smaller problems (N < 10000)
- Check your CPU - this is computationally intensive!

## Contributing

Found a bug or want to make it faster? Pull requests welcome! This project taught me a ton about parallel processing and I'd love to see what improvements others can make.

## The Math Behind It All

This problem is related to some fascinating number theory:
- Pythagorean triples (like 3, 4, 5)
- Square pyramidal numbers
- Sums of consecutive squares

If you're into math, try running it with different k values and see what patterns emerge!

---

**Authors:** 
- Somu Geetha Sravya [LinkedIn](https://www.linkedin.com/in/geetha-sravya-somu/)
- Tharun Kamsala [LinkedIn](https://www.linkedin.com/in/tharun-kamsala-b648571b9/)

Built with ❤️ using Gleam and lots of coffee ☕
