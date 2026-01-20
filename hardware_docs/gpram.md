# General Purpose RAM (GPRAM)

There will be no input/output latches like in the previous version

### TX (Read / Read ++addr)

```
{
  signal: [
    {name: "CLK", wave: "lhlhlh"},
	{},
	{name: "TX (Read / Read ++addr)", wave: "lh.l.."},
	{name: "TX ready / OE", wave: "lh.l.."},
    {},
    {name: "Post-adder latch CLK", wave: "l.h.l."},
    {name: "Pre-adder latch CLK", wave: "l..2.l", data: ["++addr"]},
    {name: "Memory read", wave: "lh.l.."}
  ]
}
```

### TX (Read addr A / B)

```
{
  signal: [
    {name: "CLK", wave: "lhlh"},
	{},
    {name: "TX (Read addr A / B)", wave: "lh.l"},
	{name: "TX ready / OE", wave: "lh.l"},
    {},
    {name: "Pre-adder address A/B OE to bus", wave: "lh.l"}
  ]
}
```

### RX (Write / Write ++addr)

```
{
  signal: [
    {name: "CLK", wave: "lhlhlhl"},
	{},
    {name: "TX ready (Write / Write ++addr)", wave: "lh.l..."},
    {},
    {name: "Post-adder latch CLK", wave: "l..h.l."},
    {name: "Pre-adder latch CLK", wave: "l...2.l", data: ["++addr"]},
    {name: "Memory write", wave: "lhl...."}
  ]
}
```

### RX (Write addr A / B)

```
{
  signal: [
    {name: "CLK", wave: "lhlhl"},
	{},
    {name: "TX ready (Write addr A / B)", wave: "lh.l."},
    {},
    {name: "Pre-adder latch CLK", wave: "l.h.l"},
    {name: "Address A/B select / A/B bus-read CLK", wave: "lh.l."}
  ]
}
```