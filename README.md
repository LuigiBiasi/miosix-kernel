## Miosix Porting to STM32L073RZ-Nucleo

<div align="right">Luigi Biasi<br/>A.A. 2025/2026<br/>Embedded Systems Project</div>

#### Summary
Port of Miosix kernel to the STM32L073RZ-Nucleo board with boot, scheduling and UART.
#### Goals and scope
- Goals: add portability for STM32L073RZ-Nucleo with correct memory management, system clock and serial ports support.

>[!Note]
>I took as a reference to implement the porting the STM32L053r8 Nucleo, that was already supported by MIOSIX.
#### Hardware overview
- Board: STM32L073RZ-Nucleo
- MCU: Arm® 32-bit Cortex®-M0+ with MPU, 192-Kbyte Flash memory with ECC, 20-Kbyte RAM
- Key components: PLL for CPU clock, Pre-programmed bootloader
#### Toolchain and host environment
- Host OS: Linux Mint 22.3 Cinnamon (via VirtualBox v7.0.14)
- Toolchain: MiosixToolchain 15.2.0mp4.2
- Debug tools: gdb 15.1, QSTLink2
#### Project Layout and low level initialization
Files added/modified
- `config/`
	- `Makefile.inc` -> added as target board the `stm32l073rz_nucleo`
- `config/board/stm32l073rz_nucleo`
	- `board_settings.h` -> adjusted the stack size (set to $4K$, proportional wrt the reference board stack size), set the working frequency at $32MHz$ (maximum for this board), serial speed at $115200$
	- `Makefile.inc`
- `miosix-kernel/miosix/arch/board/stm32l073rz_nucleo`
	- `unikernel.ld` -> set memory size: flash=$192k$, ram=$20k$
	- `Makefile.inc` -> define the board name for querying as `DBOARD_STM32L073RZ_NUCLEO`
	- `interfaces_impl/arch_registers_impl.h` -> define `STM32L073xx`
	- `interfaces_impl/boot.cpp` -> set the `IRQSetupClockTree` with $32MHz$ CPU frequency and PLL clock source.
		- Switch to voltage range $1$
		- Set flash latency to $1$ wait state
		- Set the correct PLL multiplier and divider
		- Use PLL as system clock
>[!Note]
>Actually this boot script was the one requiring some troubleshooting effort, because I was working on the MIOSIX version linked in the old version of MIOSIX wiki.
>In this new version of the repo the `boot.cpp` file is basically equal to the one of the reference board STM32L053r8 Nucleo (same clock frequency, same clock source, same latency, same voltage range).
- `miosix-kernel/miosix/arch/drivers/serial/`
	- `stm32f7_serial.cpp` -> added the support for the target board to the table of hardware configurations
- `main.cpp` -> implemented the blinking led script to test the memory and clock functionalities
#### Implemented peripherals (Serial ports)
###### USART
- Peripheral instance(s): USART1, USART2, USART4, USART5
- DMA usage: yes for USART1, no for the others peripherals due to simplified channel allocation
- Test: to be tested in the laboratory

**Pin mapping and AF settings**
The AF spans are defined as objects containing `{GPIO A/B/C, Pin Number, AF Number}` for each pin to set.

| Port   | TX                 | RX                  | RTS | CTS | CK  |
| ------ | ------------------ | ------------------- | --- | --- | --- |
| USART1 | GPIO A, Pin 9, AF4 | GPIO A, Pin 10, AF4 | -   | -   | -   |
| USART2 | GPIO A, Pin 2, AF4 | GPIO A, Pin 3, AF4  | -   | -   | -   |
| USART4 | GPIO A, Pin 0, AF6 | GPIO A, Pin 1, AF6  | -   | -   | -   |
| USART5 | GPIO B, Pin 3, AF6 | GPIO B, Pin 4, AF6  | -   | -   | -   |
###### LPUART
- Peripheral instance(s): LPUART1
- DMA usage: yes
- Test: to be tested in laboratory

**Pin mapping and AF settings**

| Port    | TX                  | RX                  | RTS                 | CTS                 | CK  |
| ------- | ------------------- | ------------------- | ------------------- | ------------------- | --- |
| LPUART1 | GPIO B, Pin 10, AF4 | GPIO B, Pin 11, AF4 | GPIO B, Pin 14, AF4 | GPIO B, Pin 14, AF4 | -   |
#### Build and boot procedure
- `make program` -> build and upload to the board. Blinking led script by default.

#### Tests and results
- Results table:

| Test                    |              Description | Result | Notes |
| :---------------------- | -----------------------: | -----: | :---- |
| Kernel memory and clock |      Blinking led script |     OK | -     |
| Serial ports            | To be done in laboratory |      - | -     |
#### Performance and resource usage
- Final binary size (`main.elf`): $1.3 MB$
#### References
- https://miosix.org/wiki/index.php?title=Main_Page
- https://www.st.com/resource/en/datasheet/stm32l073v8.pdf
