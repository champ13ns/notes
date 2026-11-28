# Operating Systems — I/O Management, Part 1
## I/O Fundamentals, Device Controllers, Programmed I/O, Polling, Interrupts and DMA

**Exam:** IBPS RRB Scale-II Specialist Officer (IT Officer)  
**Level:** Conceptual + exam traps + practical Linux/Node.js connections  
**Scope:** No numerical-heavy problems. CPU scheduling is deferred.

---

## 1. Why an Operating System manages I/O

**Input/Output (I/O)** allows software to receive data from and send data to devices or external systems. Typical operations include reading from a keyboard, receiving a network packet, reading a file, or writing to an SSD.

The OS provides a consistent interface to applications while handling device-specific behavior, access control, scheduling, buffering and errors. A program commonly requests I/O via system calls or runtime/library interfaces (which may ultimately invoke system calls).

### Do not confuse these layers

| Layer | Example | Function |
|---|---|---|
| Application | Node.js calling `fs.readFile()` | Requests an operation |
| System-call/kernel interface | Linux `read()` | Accesses an OS-managed abstraction, e.g. a file descriptor |
| Device driver | Linux NVMe or USB driver | Implements device-specific control in software |
| Device controller | NVMe controller or USB host controller | Hardware communicating with the peripheral/device |
| Physical device | SSD, keyboard | Produces or consumes data |

A Node.js function call **does not tell us** whether the underlying hardware transfer uses PIO or DMA. The data might already be in the OS page cache, in which case there may be no physical disk operation at all.

## 2. Device drivers, controllers and registers

### Device controller = hardware

A **device controller** is electronic hardware that manages device communication. A USB keyboard has an internal microcontroller; the computer has a USB **host controller** that manages USB transfers. For an NVMe SSD, the storage controller handles storage commands and data movement.

### Device driver = software

A **device driver** is OS code executed by the CPU. It configures and communicates with the controller, manages queues/buffers, starts operations, handles completion and errors, and supplies a standardized interface to the rest of the kernel.

### Registers = small storage locations in hardware

- **CPU registers:** Hold CPU working state, such as computation results, instruction position and flags.
- **Controller registers:** Expose status, controls, and sometimes data or queue-related state to software.

In a simplified controller:

| Register | Example meaning |
|---|---|
| Status | Data-ready bit = 1, busy bit = 0, error bit = 1 |
| Control | Enable a feature, issue/reset a command |
| Data | Value being received/sent in a simple PIO device |

The values and layouts are device-specific; these are *conceptual* examples, **not real USB-keyboard register definitions**.

### How can the CPU access controller registers?

**Memory-Mapped I/O (MMIO):** Device registers are mapped to a specified address range. CPU loads/stores to that address range are routed to the controller, not ordinary RAM. Normally only the kernel/driver has the required permissions and mappings.

**Port-Mapped I/O (PMIO):** Some architectures (notably x86) provide a separate I/O address space and special instructions such as `in` and `out`.

> Exam trap: Controller registers and CPU registers are different kinds of hardware state. Device drivers are software; device controllers are hardware.

## 3. Example journey: a USB keyboard keypress

1. A physical key is pressed.
2. The keyboard's internal circuitry detects the switch and updates a keyboard report (normally a key code, **not directly ASCII**).
3. The computer's USB host controller exchanges data with the keyboard according to the USB protocol.
4. When a scheduled USB transfer completes, the host controller may generate a **CPU hardware interrupt**.
5. Linux's USB driver, input subsystem and terminal environment interpret/deliver input.
6. An application can read from standard input. Depending on terminal settings, a canonical terminal typically delivers the line only after Enter; raw mode behaves differently.

**Important nuance:** USB calls one transfer category an *interrupt transfer*, but this does not mean the keyboard autonomously issues a CPU hardware interrupt. USB transfers are initiated/scheduled by the host; the host controller can interrupt the CPU upon completion.

## 4. Programmed I/O (PIO)

In **Programmed I/O**, CPU instructions perform data movement between device-controller registers and memory/CPU state.

Simplified fictional input-device example:

```c
// Illustrative pseudocode, not a real portable userspace program
while ((STATUS_REGISTER & DATA_READY) == 0) {
    // Wait for data availability (busy polling)
}

unsigned char value = DATA_REGISTER;
ram_buffer[0] = value;
```

The CPU explicitly reads the data register and writes the resulting value to memory. Real drivers must follow hardware protocol, memory-ordering and readiness requirements.

**PIO answers:** *Who executes instructions to move the device data?* → The CPU.

**Not the definition of PIO:** Merely writing a program that calls `read()` or `fs.readFile()`.

PIO may be used together with polling **or** interrupts; it does not inherently imply a busy-wait loop.

## 5. Polling and busy waiting

**Polling** means checking device/operation status repeatedly rather than depending solely on unsolicited notification. It answers *how readiness/completion is detected*, not who moves the bytes.

**Busy waiting** is tight repeated checking that consumes CPU execution capacity:

```c
while (!device_ready()) {
    // Tight loop occupies CPU time
}
```

**Periodic polling** checks at intervals, leaving time between checks:

```js
setInterval(async () => {
  // Application-level check of a resource
}, 1000);
```

The two are not equivalent. In Node.js, `setInterval()` does **not** execute a tight loop continuously between callback invocations.

### Advantages and tradeoffs

| Polling strengths | Polling costs |
|---|---|
| Simple; predictable checks | Tight busy polling burns CPU time |
| Can reduce notification overhead for short waits or batches | A long polling interval increases detection latency |
| Useful in certain high-throughput device paths | Frequent checks can cause unnecessary work |

Linux networking's **NAPI** commonly combines an initial interrupt notification with budgeted polling of incoming packets, limiting repeated per-packet interrupt overhead under load.

### Linux examples

```bash
# Application-level periodic status polling
watch -n 1 'cat /sys/class/net/*/operstate'
```

This periodically reads Linux-exposed network-interface state; it is **not a direct MMIO hardware register poll**.

Linux `poll()` and `epoll` are generally **readiness-notification APIs**, and waiting calls may sleep in the kernel; their names do not imply tight CPU busy polling.

## 6. Interrupt-Driven I/O

A **hardware interrupt** is a hardware-originated signal/event informing the CPU that attention is needed. The CPU can run other instructions instead of repeatedly checking a device's status register.

Simplified interrupt sequence:

1. A controller raises an interrupt request (or generates an MSI/MSI-X message).
2. An interrupt controller/CPU delivers/accepts the interrupt as permitted by priority/masking.
3. The CPU preserves essential execution state and transfers control to a corresponding handler.
4. The **Interrupt Service Routine (ISR)** / kernel interrupt-handling path identifies and acknowledges the event and performs or schedules necessary handling.
5. Execution resumes or the scheduler may choose another task.

### Definitions

- **ISR:** Software code executed by the CPU to service an interrupt. It is *not* executed by the device itself.
- **Interrupt vector:** Identifier the processor uses to select an interrupt handler entry.
- **Interrupt Vector Table / IDT:** Architecture-dependent data structure mapping interrupt vectors to handler entry information. On x86-64, the processor uses an **Interrupt Descriptor Table (IDT)**.
- **Interrupt controller:** Hardware responsible for routing, masking, prioritizing and/or helping acknowledge interrupts; e.g. APIC family on many x86 systems.
- **Hard interrupt versus deferred work:** Linux may keep immediate hard-IRQ work short, then do additional work through softirqs, threaded IRQ handlers, workqueues, or other kernel paths, depending on subsystem.

### Interrupt vs context switch

An interrupt is **not automatically a process context switch**. The CPU can enter an ISR and then resume the *same* process. A scheduler-driven context switch may happen afterward, but it is not mandatory.

### Overhead

Interrupt handling takes CPU time for entry/exit, state handling, acknowledgements and subsequent work. For very frequent high-rate events, interrupt batching, coalescing, or polling can be more efficient than processing one interrupt per tiny event.

### Linux inspection commands

```bash
cat /proc/interrupts     # Interrupt counters, often per CPU
watch -n 1 cat /proc/interrupts
cat /proc/softirqs       # Softirq execution counters
```

Actual output depends on system hardware, kernel and whether Linux runs in a VM/container.

### Node.js connection

```js
const readline = require('node:readline');
readline.emitKeypressEvents(process.stdin);
if (process.stdin.isTTY) process.stdin.setRawMode(true);

process.stdin.on('keypress', (ch, key) => {
  console.log('Key:', ch);
  if (key.ctrl && key.name === 'c') process.exit(0);
});
process.stdin.resume();
```

The **JavaScript callback is not a hardware ISR**. Hardware interrupts are handled in kernel/driver code; terminal input eventually reaches the user-space runtime through OS I/O APIs, and Node.js executes callbacks in user space.

## 7. Direct Memory Access (DMA)

**DMA** permits a DMA-capable hardware engine/device to transfer data to/from main memory without the CPU executing an instruction to copy **each data unit** between a device register and RAM.

Modern NVMe devices and NICs commonly incorporate DMA engines. In textbook diagrams a separate DMA controller may be shown. The same physical controller can manage the device and support DMA.

### Simplified SSD-to-RAM read

1. Application requests a file read.
2. If data is not in cache, the OS/storage driver prepares commands and appropriate memory buffers/mappings.
3. The CPU submits the operation to a DMA-capable storage controller.
4. The controller transfers requested data to RAM without CPU instructions copying each byte.
5. Completion is reported, often by a **hardware interrupt**; polling is also possible.
6. Linux completes the I/O and makes bytes available to the requesting application.

For a **buffered** Linux file read, DMA may put disk data into kernel page-cache pages, followed by a CPU memory copy into a userspace buffer. DMA is not a promise of zero copies everywhere.

### Data-transfer direction

- Device → RAM: input (e.g. SSD read, NIC receive).
- RAM → Device: output (e.g. SSD write, NIC transmit).

### What CPU still does

The CPU still sets up commands, configures/mapping buffers, submits work, deals with completion/errors and may move data between kernel and user memory. DMA also uses memory/interconnect bandwidth.

### DMA vs interrupt

- **DMA:** *How does bulk data move?* → Hardware transfer to/from RAM.
- **Interrupt:** *How does CPU learn about an event/completion?* → Notification.

DMA completion can use interrupts **or polling**. An interrupt does not imply DMA (it may notify the CPU to do PIO).

### DMA vs synchronous/asynchronous APIs

These are separate dimensions:

```js
const fs = require('node:fs');
const bytes = fs.readFileSync('file.bin'); // Blocks calling JS thread
```

```js
const fs = require('node:fs/promises');
const bytes = await fs.readFile('file.bin'); // Asynchronous API
```

Both may use DMA if physical disk access occurs. Neither guarantees disk access: cached data might be returned. On common Linux Node.js setups, asynchronous filesystem operations are executed using libuv's thread pool; DMA is a lower-level hardware concern.

**Exam trap:** "DMA always makes the application non-blocking" is **false**.

### Linux commands (observation, not DMA measurement)

```bash
lsblk -o NAME,TRAN,MODEL
cat /proc/interrupts | grep -i nvme
# If sysstat is installed:
iostat -dx 1
```

These expose device type, interrupt counters or I/O activity. They **do not directly measure DMA bytes**.

## 8. Master comparison — keep the dimensions separate

| Concept | What it answers | Typical mechanism |
|---|---|---|
| Programmed I/O | Who moves device data? | CPU reads/writes device data registers |
| DMA | Who moves device data? | DMA-capable hardware accesses RAM |
| Polling | How is readiness/completion noticed? | CPU/kernel repeatedly checks status |
| Interrupt-driven | How is readiness/completion noticed? | Device/controller alerts CPU |
| Blocking I/O | Does caller wait for result? | Caller cannot proceed until completion |
| Non-blocking I/O | What happens if result isn't ready? | Operation returns without waiting (often `EAGAIN`) |
| Synchronous API | How is operation's completion presented to caller? | Caller waits in the usual blocking-read sense (definitions vary by API) |
| Asynchronous API | How is completion delivered? | Completion later via callback, future/promise, event, or similar mechanism |

### Valid combinations

1. Polling to detect readiness + CPU transferring via PIO.
2. Interrupt signaling readiness + CPU transferring via PIO.
3. DMA transferring bytes + interrupt signaling completion.
4. DMA transferring bytes + polling detecting completion.

**A single real system can use several methods together.**

## 9. Frequently tested distinctions and traps

1. **System call ≠ PIO.** A system call is an OS service request; PIO is a hardware transfer method.
2. **Driver ≠ controller.** Driver is software; controller is hardware.
3. **Polling ≠ busy waiting.** Polling may be periodic or event-loop-driven.
4. **Interrupt ≠ process context switch.** The interrupted process can resume.
5. **ISR ≠ user-space callback.** A Node.js `keypress` callback does not run in hard-IRQ context.
6. **DMA ≠ interrupt.** They address data movement and notification respectively.
7. **DMA ≠ asynchronous application.** `readFileSync()` may ultimately use DMA hardware.
8. **File read ≠ guaranteed disk read.** Linux page cache may satisfy it.
9. **USB interrupt transfer ≠ keyboard directly interrupting the CPU.** USB host schedules USB transfers.
10. **Polling is not universally inferior.** It can be efficient for short waits/batches or at high packet rates.

## 10. Practice questions (no heavy numericals)

**Q1.** Which component is hardware?  
A. Device driver  B. Device controller  C. System call  D. Scheduler

**Q2.** A CPU repeatedly reads a status register until ready. This is:  
A. Spooling  B. DMA  C. Polling  D. Paging

**Q3.** CPU instructions read controller data registers and store bytes into RAM. This transfer mechanism is:  
A. PIO  B. DMA  C. Swapping  D. Spooling

**Q4.** A hardware interrupt always triggers a process context switch.  
A. True  B. False

**Q5.** Who executes an ISR?  
A. Keyboard electronics  B. CPU running handler code  C. RAM chip  D. Node.js callback alone

**Q6.** Which identifies a handler entry for a hardware interrupt?  
A. Inode  B. Interrupt vector  C. Page frame number  D. File descriptor

**Q7.** DMA primarily reduces:  
A. Number of files  B. CPU instructions for device payload movement  C. RAM requirements to zero  D. Need for all drivers

**Q8.** Can DMA completion be detected by polling rather than interrupts?  
A. Yes  B. No

**Q9.** Which statement is most accurate about `readFileSync()`?  
A. It prevents DMA  B. It guarantees a physical disk operation  C. It blocks the calling JS thread; hardware may use DMA  D. It runs in the ISR

**Q10.** Linux `epoll_wait` always spins in a busy loop until an event arrives.  
A. True  B. False

### Answer key with brief reasoning

1. **B** — Controller is hardware; driver is software.
2. **C** — Repeated readiness checks = polling.
3. **A** — CPU executes the transfer instructions = PIO.
4. **B** — ISR can return to same process; no mandatory switch.
5. **B** — ISR is CPU-executed code.
6. **B** — Vector chooses the corresponding entry.
7. **B** — DMA engine, not CPU copy instructions, handles payload transfer.
8. **A** — DMA and completion notification are separate issues.
9. **C** — Synchronous JS API does not dictate hardware transfer mechanism.
10. **B** — Kernel readiness waits can sleep rather than busy-spin.

---

## 11. 60-second revision checklist

- [ ] Can I explain application → system call → driver → controller → hardware?
- [ ] Can I distinguish controller registers from CPU registers?
- [ ] Can I distinguish MMIO from ordinary RAM?
- [ ] Can I explain PIO in one sentence?
- [ ] Can I distinguish polling from busy waiting?
- [ ] Can I explain what happens when a hardware interrupt arrives?
- [ ] Can I explain ISR, vector, IDT, and interrupt controller?
- [ ] Can I explain why a hardware interrupt does not imply context switching?
- [ ] Can I explain DMA and what CPU still does?
- [ ] Can I explain why DMA, interrupts and synchronous/asynchronous APIs are different dimensions?

**Next topic:** Buffering (then caching, then spooling, separately).
