<h1 align="center">interval.mq</h1>

> [!WARNING]
> This project is currently under active development. APIs and features may change without notice.

Sets of half-open intervals for [mq](https://github.com/harehare/mq), as a module. Use it to merge line ranges or time ranges, ask whether a point or a range is covered, and find what is left uncovered. An interval `[start, end)` includes `start` and excludes `end`, so `[1, 3)` and `[3, 5)` touch and merge into `[1, 5)`.

## Installation

Copy `interval.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp interval.mq ~/.local/mq/config/
```

HTTP imports are disabled by default; pass `--allow-http-import` to import directly from GitHub:

```sh
mq --allow-http-import -I null 'import "github.com/harehare/interval.mq" | interval::from([[5, 8], [1, 3], [3, 4]])'
```

## Usage

```sh
mq -L . -I null '
  import "interval"
  | let s = interval::from([[5, 8], [1, 3], [3, 4], [10, 12]])
  | [s, interval::gaps(s, 0, 13), interval::contains(s, 3), interval::contains(s, 4)]
'
# [[[1, 4], [5, 8], [10, 12]], [[0, 1], [4, 5], [8, 10], [12, 13]], true, false]
```

A set is a plain array of `[start, end]` pairs, sorted and never overlapping or touching.

| Function | Description |
| --- | --- |
| `new()` / `from(pairs)` | empty set / set merged from pairs in any order |
| `add(set, s, e)` / `remove(set, s, e)` | cover or uncover `[s, e)` |
| `contains(set, x)` / `covering(set, x)` | point test / the interval holding the point, or `None` |
| `overlaps(set, s, e)` | `[s, e)` shares a point with the set |
| `gaps(set, lo, hi)` | the parts of `[lo, hi)` not covered |
| `union(a, b)` / `intersect(a, b)` / `subtract(a, b)` | set operations |
| `count(set)` / `length(set)` | number of intervals / total length covered |

An interval whose end is before its start is an error, and an empty one (`s == e`) is ignored. Numbers may be fractions.

## Notes

- Every operation returns the updated set. `contains`, `covering` and `overlaps` take O(log n). `add` and `remove` rebuild the array, so they take O(n). `union`, `intersect`, `subtract` and `gaps` take time proportional to the sizes of their inputs.

## Development

```sh
mq-test interval_tests.mq
mq-bench -L . interval_bench.mq
```

Each `_small` / `_large` pair in `interval_bench.mq` runs the same work at two sizes. One of the goals of this module is also to check mq for performance problems.
