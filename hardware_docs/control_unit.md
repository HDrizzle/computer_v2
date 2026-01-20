# Control unit

https://lucid.app/lucidchart/32ad8e9d-d2d1-47b0-a4b6-105ece57e4e0/edit?viewport_loc=-87%2C-294%2C1954%2C1174%2C0_0&invitationId=inv_57dea9bb-0ba1-45a8-8f8a-2a729ad6f8bd

## Startup

1. Started by the `Set` input from the startup controller going high, which will disable the GOTO latches OE, post-adder latch OE, and the call stack OE. Pullup resistors will set the inputs to the pre-adder latch to 0xFFFF so when it is incremented it starts at instruction 0x0000.
2. Clock pre-adder address latch
3. Begin regular cycle

## Cycle, PC incrementation may be done in parallel

There will be an SR latch `Instruction done` which will be read on +edge to continue with `Begin instruction sequence` for the next instruction.

Load from FLash
The short clock pulses are incase of instructions that take one clock cycle and the latch CLKs cannot be high all the time.
```
{
  signal: [
	{name: "CLK", wave: "lhlhlhl"},
	{},
    {name: "Load Instruction", wave: "lh.l..."},
	{name: "Post-adder latch OE", wave: "lh.l..."},
	{name: "Flash read (PC MSB=0)", wave: "lh.l..."},
    {name: "Instruction pre-latches CLK", wave: "l.hl...", node: "..a"},
	{name: "Instruction post-latch CLK", wave: "l..hl..", node: "...b"},
	{name: "Begin instruction sequence", wave: "l..h.l."},
	{name: "Instruction done", wave: "l.hl..."}
  ],
	edge: ["a~>b Wait for instruction done"]
}
```

Load from RAM
```
{
  signal: [
	{name: "CLK", wave: "lhlhlhlh"},
	{},
    {name: "Load Instruction", wave: "lh.l...."},
	{name: "Post-adder latch OE", wave: "lh.l...."},
	{name: "RAM read (PC MSB=1)", wave: "lh...l.."},
	{name: "RAM LSB", wave: "l..h.l.."},
	{name: "Instruction pre-latch A CLK", wave: "l.hl...."},
	{name: "Instruction pre-latch B CLK", wave: "l...hl..", node: "....a"},
	{name: "Instruction post-latch CLK", wave: "l....h.l", node: ".....b"},
	{name: "Begin instruction sequence", wave: "l....h.l"},
	{name: "Instruction done", wave: "l...hl.."}
  ],
	edge: ["a~>b Wait for instruction done"]
}
```

## Instruction sequences

### MOVE & WRITE

The data source and destination will have their own sequences

RX Ready case
```
{
  signal: [
    {name: "CLK", wave: "lhlhlh"},
	{},
	{name: "Begin instruction sequence (MOVE/WRITE)", wave: "lh.l.."},
	{},
    {name: "Bus TX", wave: "lh.l..", node: ".a"},
	{name: "TX ready / OE", wave: "l..h.l", node: "...b"},
	{name: "RX not ready", wave: "l....."},
	{name: "Bus save CLK", wave: "l....."},
	{name: "Bus save OE", wave: "l....."},
	{name: "Instruction done", wave: "l...h."}
  ],
  edge: ["a~>b TX Delay (possibly instant)"]
}
```

RX Not ready case
```
{
  signal: [
    {name: "CLK", wave: "lhlhlhlh"},
	{},
	{name: "Begin instruction sequence (MOVE/WRITE)", wave: "lh.l...."},
    {},
	{name: "Bus TX", wave: "lh.l....", node: ".a"},
	{name: "TX ready / OE", wave: "l..h.l..", node: "...b"},
	{name: "RX not ready", wave: "l..h.l.."},
	{name: "Bus save CLK", wave: "l...h.l."},
	{name: "Bus save OE", wave: "l....h.l"},
	{name: "Instruction done", wave: "l.....h."}
  ],
  edge: ["a~>b TX Delay (possibly instant)"]
}
```

### GOTO (flow-control)

```
{
  signal: [
    {name: "CLK", wave: "lhlhl"},
	{},
	{name: "Begin instruction sequence (GOTO)", wave: "lh.l."},
	{},
	{name: "GOTO latches OE", wave: "lh.l."},
	{name: "Pre-adder latch CLK", wave: "l.h.l"},
    {name: "Load Instruction", wave: "l..h."}
  ],
  edge: []
}
```

### GOTO-IF (flow-control)

```
{
  signal: [
    {name: "CLK", wave: "lhlhl"},
	{},
	{name: "Begin instruction sequence (GOTO-IF)", wave: "lh.l."},
	{},
	{name: "GOTO latches OE", wave: "l2.l.", data: ["DECIDER"]},
	{name: "Post-adder latch OE", wave: "l2.l.", data: ["!DECIDER"]},
	{name: "Pre-adder latch CLK", wave: "l.h.l"},
    {name: "Load Instruction", wave: "l..h."}
  ],
  edge: []
}
```

### HALT

```
{
  signal: [
    {name: "CLK", wave: "lhlh"},
	{},
	{name: "Begin instruction sequence (HALT)", wave: "lh.l"},
	{name: "Instruction done", wave: "l.h."}
  ],
  edge: []
}
```

### CALL (flow-control)

```
{
  signal: [
    {name: "CLK", wave: "lhlhlhl"},
	{},
	{name: "Begin instruction sequence (CALL)", wave: "lh.l..."},
	{},
	{name: "Post-adder latch OE", wave: "lh.l..."},
	{name: "Call stack - Push", wave: "lh.l..."},
	{name: "GOTO latches OE", wave: "l..h.l."},
	{name: "Pre-adder latch CLK", wave: "l...h.l"},
    {name: "Load Instruction", wave: "l....h."}
  ],
  edge: []
}
```

### RETURN (flow-control)

```
{
  signal: [
    {name: "CLK", wave: "lhlhl"},
	{},
	{name: "Begin instruction sequence (RETURN)", wave: "lh.l."},
	{},
	{name: "Post-adder latch OE", wave: "lh.l."},
	{name: "Call stack - Pop & OE", wave: "lh.l."},
	{name: "Pre-adder latch CLK", wave: "l.h.l"},
    {name: "Load Instruction", wave: "l..h."}
  ],
  edge: []
}
```