Tests for implicit dependency resolution
========================================

Notation: we use capital letters for names of deps, and lower case letters for
their implementations. That is:

```
a:
  I1 = i1
```

is shorthand for a `Move.toml` in directory `a` containing `I1 = { local = "../i1" }`

In the following, all tests have the following implicit dependencies:

```
i1:
 I2 = i2

i2: no deps
```

Tests
-----


### Simple #########################################################################################

implicit deps should be added

```
a: no deps
```

Expected: `a` should have implicit deps added

```
a ─→ i1 ──┐
│         ↓
└───────→ i2
```

### Transitive #####################################################################################

```
a:
  B: b

b: no deps
```

Expected: `a` and `b` should both have implicit deps added


```
a ───→ b
│└────┼┐│
│┌────┘││
↓↓     ↓↓
i1 ──→ i2
```


### Override #######################################################################################

```
a:
  I2 = i2'

i2': no deps
```

Expected: `a` should not have any implicit deps because any explicits should
turn off implicits

```
a ─→ i2'
```

### Override in root 1 #############################################################################

```
a:
  B: b
  I1: i1'

b: no deps
i1': no deps
```

Expected:
 - no implcits for `a` (because of explicit),
 - nor for `i1'` or `i2` (because they are system packages)
 - implicits added for `b`, but `i1` is overridden to `i1'` (because of override in `a`)

```
a ─→ b ─→ i2
│    ↓
└──→ i1'
```

### Override in root 1 error #######################################################################

```
a:
  B: b
  I1: i1'

b: no deps

i1':
  I2: i2'
```

Expected:
 - Error because `i1'` and `b` have incompatible deps on `i2`

```
a ─→ b ───→ i2  ┐
│    ↓          ≠ error!
└──→ i1' ─→ i2' ┘
```

### Override in root 2 #############################################################################

```
a:
  B: b
  I2: i2'

b: no deps
i1': no deps
```

Expected:
 - no implicits for `a`
 - implicits added for `b`
 - `i2` is overridden to `i2'` in both `b` and `i1` (because of override in `a`)

```
a ─→ b ──→ i1
│    ↓     │
└──→ i2' ←─┘
```

### Override in dep 1 ##############################################################################

```
a:
  B: b

b:
  I1: i1'

i1': no deps
```

Expected:
 - implicits added for `a`
 - no implicits added for `b`, but `i1'` is replaced with `i1` because of implicit override in `a`
 - note difference between situation when `b` has no deps: no dep from `b` to `i2`

```
a ───→ b
│└────┼┐
│┌────┘│
↓↓     ↓
i1 ──→ i2
```

### Override in dep 2 ##############################################################################

```
a:
  B: b

b:
  I1: i1'

i1': no deps
```

Expected:
 - implicits added for `a`
 - no implicits added for `b`, but `i1'` is replaced with `i1` because of implicit override in `a`
 - note difference between situation when `b` has no deps: no dep from `b` to `i2`

```
a ───→ b
│└────┐│
│     ││
↓     ↓↓
i1 ─→ i2
```
