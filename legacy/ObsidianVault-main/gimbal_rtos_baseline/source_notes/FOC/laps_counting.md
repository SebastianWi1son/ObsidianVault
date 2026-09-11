```c
if (fabsf(d_raw) > (0.8f * TWO_PI))
	{ motor->full_rotations += (d_raw > 0.0f) ? -1 : 1; }
```

**capture the jump between 3.14 and 0**
	delta > 0.8 * PI  (3.14 -> 0)