# ms_tmr32
A 32-bit Timer/Counter/Capture/PWM Soft IP in Verilog. It comes with wrappers for:
- [x] WB bus (Classical).
- [x] APB (needs Verifications)
- [ ] AHB (TBD)

## Features
- Up/Down Counting
- One Shot or Periodic Timer
- PWM Generation
- External Events Capturing/Counting
- 8-bit Clock Prescaler (Timer only)
- 3 Interrupt sources
    - Timer Time-out
    - Counter Match
    - Event Capture

## The Interface
### ms_tmr32
<img src="./docs/ms_tmr32.svg" alt= “” width="60%" height="60%">


| Port name  | Direction | Type   | Description |
| ---------- | --------- | ------ | ----------- |
| clk        | input     |        | Clock            |
| rst_n      | input     |        | Active low reset            |
| ctr_in     | input     |        | External events input            |
| pwm_out    | output    |        | PWM Out            |
| period     | input     | [31:0] | 32-bit Timer Period             |
| pwm_cmp    | input     | [31:0] | 32-bit PWM Compare Value            |
| ctr_match  | input     | [31:0] | 32-bit match value (counter mode)            |
| tmr        | output    | [31:0] | Current timer value            |
| cp_count   | output    | [31:0] | Current counter value            |
| clk_src    | input     | [3:0]  | clk source (9: ctr_in, 8: clk, 0-7: clk/2 to clk/256)            |
| to_flag    | output    |        | Time out flag             |
| match_flag | output    |        | Match flag            |
| tmr_en     | input     |        | Timer enable            |
| one_shot   | input     |        | One Shot mode enable (default: periodic)           |
| up         | input     |        | Up counting enable (default: down counting)            |
| pwm_en     | input     |        | PWM generator enable            |
| cp_en      | input     |        | External events capturing enable            |
| cp_event   | input     | [1:0]  | External Event type (1: posedge, 2: negedge, 3: both)            |
| cp_flag    | output    |        | Capture event received flag           |
| en         | input     |        | Global enable            |

### ms_tmr32_wb
<img src="./docs/ms_tmr32_wb.svg" alt= “” width="80%" height="80%">

| Port name | Direction | Type           | Description |
| --------- | --------- | -------------- | ----------- |
| clk_i     | input     | wire           |             |
| rst_i     | input     | wire           |             |
| adr_i     | input     | wire    [31:0] |             |
| dat_i     | input     | wire    [31:0] |             |
| dat_o     | output    | wire    [31:0] |             |
| sel_i     | input     | wire    [3:0]  |             |
| cyc_i     | input     | wire           |             |
| stb_i     | input     | wire           |             |
| ack_o     | output    |                |             |
| we_i      | input     | wire           |             |
| ctr_in    | input     | wire           | External events input            |
| pwm_out   | output    | wire           | PWM output            |
| irq       | output    | wire           | Interrupt ReQuest output            |

### ms_tmr32_apb
<img src="./docs/ms_tmr32_apb.svg" alt= “” width="70%" height="70%">

| Port name | Direction | Type         | Description |
| --------- | --------- | ------------ | ----------- |
| PCLK      | input     | wire         |             |
| PRESETn   | input     | wire         |             |
| PADDR     | input     | wire  [31:0] |             |
| PWRITE    | input     | wire         |             |
| PSEL      | input     | wire         |             |
| PWDATA    | input     | wire  [31:0] |             |
| PRDATA    | output    | wire  [31:0]       |             |
| PREADY    | output    | wire             |             |
| ctr_in    | input     | wire           | External events input            |
| pwm_out   | output    | wire           | PWM output            |
| irq       | output    | wire           | Interrupt ReQuest output            |
## ms_tmr32 Internals

![Diagram](./docs/ms_tmr32_bd.png "Diagram")

## Modes of Operation
| mode | tmr_en | pwm_en | cp_en | clk_src| description |
|------|--------|--------|-------|--------|-------------|
| Timer|1       |0       |0      |0-8|Counts clock cycles to keep track of time. <br>- <b>clk_src</b>: 8: clk, 0-7: clk/2 to clk/256 <br>- <b>up</b>: 0: down counting (period to 0), 1: up counting (0 to period) <br>- <b>one_shot</b>: 0:periodic, 1:one shot<br>- <b>period</b>: starting/terminal count for down/up counting.<br>- <b>to_flag</b>: is set when the tmr reaches the terminal count (up:period, down:0)|
| Counter|1|0|0|9 | Counts external pulses coming on <b>ctr_in</b>.<br><b>match_flag</b> is set when <b>tmr</b> matches <b>ctr_match</b>. |
|PWM|1|1|0|0-8|Generates PWM signal on <b>pwm_out</b>. <br> The duty cycle is determined by <b>pwm_cmp</b>. <br><b>period</b> determines the PWM signal period.|
|Event Capture|0|0|1|0-8|Capture the time between two events on <b>ctr_in</b>.<br><b>cp_event</b> sets the event: 1:Raising Edge, 2:Falling Edge or 3:Both.<br><b>cp_flag</b> is set when two two consecutive events are observed; once set, <b>cp_count</b> shows the current count.|

## Wishbone Registers
### Timer Value Register [offset: 0x00, RO]
![Diagram](./docs/reg32.svg "Diagram")
### Timer Period Register [offset: 0x04, RW]
![Diagram](./docs/reg32.svg "Diagram")
### PWM Compare Register [offset: 0x08, RW]
![Diagram](./docs/reg32.svg "Diagram")
### Counter Match Register [offset: 0x0C, RW]
![Diagram](./docs/reg32.svg "Diagram")
### Control Register [offset: 0x100, RW]
![Diagram](./docs/ctrl.svg "Diagram")
### Raw Interrupts Status Register [offset: 0x200, RO]
Reflects the status of interrupts trigger conditions detected (raw, prior to masking). 
- to: Time-out Flag
- cp: Capture Flag
- match: Match Flag

![Diagram](./docs/flags.svg "Diagram")
### Masked Interrupts Status Register [offset: 0x204, RO]
Similar to RIS but shows the state of the interrupt after masking. MIS register is always RIS & IM.
![Diagram](./docs/flags.svg "Diagram")

### Interrupts Mask Register [offset: 0x208, RW]
Disabling/ENabling an interrupt source.
![Diagram](./docs/flags.svg "Diagram")

### Interrupt Clear Register [offset: 0x20c, RW]
Writing a 1 to a bit in this register clears the corresponding interrupt state in the RIS Register. 
![Diagram](./docs/flags.svg "Diagram")
