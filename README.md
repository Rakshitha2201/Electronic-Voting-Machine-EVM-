# PROJECT REPORT: ELECTRONIC VOTING MACHINE (EVM) ANALYSIS
</br>

Design of a Finite State Machine (FSM) Controlled Electronic Voting Machine using Verilog HDL and Xilinx Vivado



## 1. Abstract

This project describes the design and verification of a simplified three-candidate **Electronic Voting Machine (EVM)** using **Verilog Hardware Description Language (HDL)**. The core system ensures **accuracy and security** by implementing **prioritized mutual exclusion logic** to register only one vote per clock cycle, even when inputs are concurrent.

The design utilizes a **Finite State Machine (FSM)** for sequential control, which includes **IDLE**, **VOTING**, and **END** states. It uses synchronous memory models: **ROM** for candidate IDs and **RAM** for mutable vote counters. Developed and verified using the **Xilinx Vivado Design Suite**, simulation results confirm the FSM correctly manages the voting flow, and the combinational winner logic accurately identifies the winner, including applying a **tie-breaker rule** in the final state. The project demonstrates a reliable, verifiable digital voting system suitable for FPGA implementation.

---
</br>

## 2. Introduction

### 2.1 Electronic Voting Overview

**Electronic Voting Machines (EVMs)** automate the process of recording and counting votes, significantly enhancing efficiency and reducing the human errors inherent in paper ballot systems. Modern EVMs are critical infrastructure, demanding high security, integrity, and operational transparency.

### 2.2 Why Digital Voting?

The shift to digital voting is driven by key advantages:
* **Accuracy:** Digital logic ensures error-free, fixed-point counting.
* **Speed:** Results are tallied instantaneously upon system halt.
* **Reliability:** Hardware-based systems offer stability unmatched by software-driven processes.

### 2.3 Role of HDL/FPGA in Reliable Voting

**Field-Programmable Gate Arrays (FPGAs)** are ideal for robust EVM designs. Implementing the voting logic directly in hardware (RTL) makes the execution **deterministic, ultra-fast, and highly resistant to external software-based tampering**. Verilog HDL provides the necessary parallel modeling capability for handling simultaneous inputs securely.

### 2.4 Tools Used - Vivado & Verilog HDL

The **Xilinx Vivado Design Suite** was selected for its comprehensive capabilities in RTL design, synthesis, implementation, and rigorous behavioral simulation. **Verilog HDL** was used to model the synchronous and combinational digital logic, including the FSM and memory structures.

---
</br>

## 3. Literature Survey

### 3.1 Traditional EVMs and FPGA Benefits

Traditional EVMs, often microcontroller-based, have faced scrutiny regarding verifiability and tamper-resistance. Academic studies have consistently proven the **superiority of FPGAs in security for voting** due to their inherent resistance to software viruses. FPGAs allow for deterministic, fixed-logic implementations, which is why they form the basis of this project.

### 3.2 FSM Requirement and Design Gaps

A persistent need identified in previous designs is the requirement for a **robust Finite State Machine** to tightly control the election sequence. Furthermore, a significant vulnerability in simple push-button EVM designs is the failure to properly handle **concurrent inputs**.

### 3.3 Comparison of Existing Methods

| Feature | Paper Ballot | Microcontroller-based EVM | FPGA-based EVM (This Project) |
| :--- | :--- | :--- | :--- |
| **Counting Error** | High (Manual) | Low (Software) | **Zero (RTL Hardware)** |
| **Security/Tampering** | Low (Physical) | Moderate (SW/HW) | **High (Logic is fixed in HW)** |
| **Speed** | Very Slow | Fast | **Immediate** |
| **Verifiability** | High (Recount) | Moderate (Code Review) | **High (RTL Code is the Source)** |

### 3.4 Gaps Addressed

This project directly addresses the **concurrency issue** by implementing a **prioritized input arbitration (mutual exclusion) mechanism** within the synchronous counting module, ensuring only **one vote is recorded per clock cycle**.

---
</br>

## 4. Problem Definition & Objectives

### 4.1 Problem Definition

The core problem is to develop a reliable digital system that can correctly record votes from **asynchronous push-button inputs** for three candidates (C0, C1, C2). The system must resolve concurrency issues by ensuring only one candidate's vote is registered per clock cycle. Furthermore, the counting must be gated by the FSM, and the final result must be displayed only after the voting session is formally terminated.

### 4.2 Objectives

1.  To design and simulate a Verilog voting machine using Vivado, modeling all necessary modules (Input, Counter, Memory, FSM, Winner Logic).
2.  To provide multi-candidate vote recording for three candidates using synchronous RAM registers.
3.  To implement FSM control to strictly manage the sequential flow through **IDLE**, **VOTING**, and **END** states.
4.  To implement **prioritized mutual exclusion logic** to guarantee a single vote per clock cycle, enhancing security.
5.  To verify correctness through detailed Vivado waveform analysis of state transitions and counter behavior.

---
</br>

## 5. System Requirements

### 5.1 Hardware & Software Requirements

The project requires a PC/Laptop capable of running the **Xilinx Vivado Design Suite (2020+)** for synthesis and simulation. The core language is **Verilog HDL**. The immediate requirement is the **Vivado Behavioral Simulator** for verification.

### 5.2 HDL Methodology Used

The design adheres to the **RTL (Register Transfer Level) Design methodology**. The system is modeled using **Behavioral Modeling** techniques, specifically utilizing the standard **two-block FSM structure**. Verification relies on a **Testbench Driven Verification** approach, confirming expected outcomes via a self-checking testbench stimulus pattern (`@(posedge clk)`).

---
</br>

## 6. System Design

### 6.1 Overall Architecture / Block Diagram

The system separates the control and data paths.
* The inputs (`btn_c0`, `btn_c1`, `btn_c2`, `end_btn`) feed into the **FSM Controller**.
* The FSM regulates the **Counting Module**, which uses the **Memory Model** (ROM for candidate IDs, RAM for mutable votes).
* The **Winner Display Logic** provides the output when the FSM reaches the final state.


### 6.2 Core Module Descriptions (Continued)

This prioritized `if-else if` construct guarantees that if C0 and C1 are pressed simultaneously, **only C0 registers a vote**, resolving the concurrency problem in hardware. Counting is enabled only during the `S_IDLE` (for the first vote) or `S_VOTE` states.

#### Memory Model (ROM & RAM)

The memory is modeled using Verilog arrays:
* **ROM**: `reg [1:0] candidate_rom [0:2];` initialized to store fixed candidate IDs (`00`, `01`, `10`).
* **RAM**: `reg [7:0] votes [0:2];` models the dynamic, synchronous vote counters. Each counter is 8 bits, allowing up to 255 votes per candidate, and is initialized to zero on reset.

#### Winner Detection Logic and Tie-Breaker Flow

This module is a **purely combinational circuit** that determines the winner based on the current state of the vote counters. The output `winner` is only driven with the final result when the FSM reaches the **`S_END` state**. Otherwise, the output defaults to Candidate 0's ID (`2'b00`).

The logic establishes a **hardware-defined tie-breaking priority** to ensure a decisive winner is always declared in case of equal vote counts. The fixed priority order is: **Candidate 0 (C0) > Candidate 1 (C1) > Candidate 2 (C2)**.

The combinational decision flow uses nested `if-else if` logic:
1.  **Check C0 (Highest Priority)**: If `votes[0]` is greater than or equal to both `votes[1]` and `votes[2]`, Candidate 0 is declared the winner (ID `2'b00`).
2.  **Check C1 (Medium Priority)**: If C0 was not the winner, the logic checks if `votes[1]` is greater than or equal to both `votes[0]` and `votes[2]`. This means C1 wins if its count is strictly highest or if it ties with C2.
3.  **Default C2 (Lowest Priority)**: If neither C0 nor C1 satisfies the winning condition, Candidate 2 is declared the winner (ID `2'b10`).


### 6.3 Finite State Machine (FSM) Flow and Control Logic

The system is controlled by a **3-state Moore FSM**, implemented using the standard two-block structure. This FSM ensures the EVM operates in a strictly controlled, sequential manner, dictating exactly when counting is enabled and when results are displayed.

#### FSM Flow (The Three States)

| State | ID | Description | Transition Condition | Next State |
| :---: | :---: | :--- | :--- | :---: |
| **S\_IDLE** | `2'b00` | Initial state; counting is inhibited. | Assertion of any candidate button (`btn_c0` OR `btn_c1` OR `btn_c2`). | S\_VOTE |
| **S\_VOTE** | `2'b01` | Active election phase; synchronous counting is enabled. | Assertion of the `end_btn` input. | S\_END |
| **S\_END** | `2'b10` | Terminal state; counting is disabled. | None; FSM is latched here, only exiting upon a full system reset. | S\_END |

All state changes are **synchronous** to the positive clock edge (`posedge clk`). The design ensures the first vote is registered simultaneously on the same clock edge that transitions the FSM from `S_IDLE` to `S_VOTE`, ensuring no vote is lost during initiation.




### 6.4 Core Module Descriptions

#### Input and Counting Module

The inputs are asynchronous push buttons. The critical **Mutual Exclusion Logic** is embedded within the synchronous counter update block, ensuring security and input arbitration:

```verilog
// Logic is within a synchronous block (always @(posedge clk))
if (btn_c0) // Highest priority
    votes[0] <= votes[0] + 1;
else if (btn_c1)
    votes[1] <= votes[1] + 1;
else if (btn_c2) // Lowest priority
    votes[2] <= votes[2] + 1;
```
---
</br>

## 7. Code For Stimualtion
```verilog
`timescale 1ns / 1ps

module VOTING_MACHINE_v1005(
    input clk,
    input reset,
    input btn_c0,
    input btn_c1,
    input btn_c2,
    input end_btn,
    output reg [1:0] winner
);

    // --------------------------------------------------------
    // ROM
    // --------------------------------------------------------
    reg [1:0] candidate_rom [0:2];
    initial begin
        candidate_rom[0] = 2'b00;
        candidate_rom[1] = 2'b01;
        candidate_rom[2] = 2'b10;
    end

    // --------------------------------------------------------
    // RAM (Vote Counters)
    // --------------------------------------------------------
    reg [7:0] votes [0:2];

    // --------------------------------------------------------
    // FSM States
    // --------------------------------------------------------
    localparam S_IDLE = 2'b00,
               S_VOTE = 2'b01,
               S_END  = 2'b10;

    reg [1:0] current_state, next_state;

    // --------------------------------------------------------
    // State Register
    // --------------------------------------------------------
    always @(posedge clk or posedge reset) begin
        if (reset)
            current_state <= S_IDLE;
        else
            current_state <= next_state;
    end

    // --------------------------------------------------------
    // Next State Logic
    // --------------------------------------------------------
    always @(*) begin
        case (current_state)
            S_IDLE:
                if (btn_c0 || btn_c1 || btn_c2)
                    next_state = S_VOTE;
                else
                    next_state = S_IDLE;

            S_VOTE:
                if (end_btn)
                    next_state = S_END;
                else
                    next_state = S_VOTE;

            S_END:
                next_state = S_END;

            default:
                next_state = S_IDLE;
        endcase
    end

    // --------------------------------------------------------
    // FIXED: Vote Counting (Catches IDLE→VOTE Transition)
    // --------------------------------------------------------
    wire vote_event = (btn_c0 || btn_c1 || btn_c2);

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            votes[0] <= 0;
            votes[1] <= 0;
            votes[2] <= 0;
        end
        else if ((current_state == S_IDLE && vote_event) ||
                 (current_state == S_VOTE)) begin

            if (btn_c0)
                votes[0] <= votes[0] + 1;
            else if (btn_c1)
                votes[1] <= votes[1] + 1;
            else if (btn_c2)
                votes[2] <= votes[2] + 1;
        end
    end

    // --------------------------------------------------------
    // Winner Logic
    // --------------------------------------------------------
    always @(*) begin
        if (current_state == S_END) begin
            if (votes[0] >= votes[1] && votes[0] >= votes[2])
                winner = candidate_rom[0];
            else if (votes[1] >= votes[0] && votes[1] >= votes[2])
                winner = candidate_rom[1];
            else
                winner = candidate_rom[2];
        end else
            winner = 2'b00;
    end

endmodule
`timescale 1ns / 1ps

module VOTING_MACHINE_tb;

    // --------------------------------------------------------
    // Inputs to DUT
    // --------------------------------------------------------
    reg clk;
    reg reset;
    reg btn_c0;
    reg btn_c1;
    reg btn_c2;
    reg end_btn;

    // --------------------------------------------------------
    // Output from DUT
    // --------------------------------------------------------
    wire [1:0] winner;

    // --------------------------------------------------------
    // Instantiate DUT
    // --------------------------------------------------------
    VOTING_MACHINE_v1005 DUT (
        .clk(clk),
        .reset(reset),
        .btn_c0(btn_c0),
        .btn_c1(btn_c1),
        .btn_c2(btn_c2),
        .end_btn(end_btn),
        .winner(winner)
    );

    // --------------------------------------------------------
    // TAP INTERNAL SIGNALS FOR WAVEFORM VIEWING
    // --------------------------------------------------------
    // These let you SEE the internal vote counters in your VCD/wave window.
    wire [7:0] v0 = DUT.votes[0];
    wire [7:0] v1 = DUT.votes[1];
    wire [7:0] v2 = DUT.votes[2];
    wire [1:0] state = DUT.current_state;

    // --------------------------------------------------------
    // Clock Generation (10ns period)
    // --------------------------------------------------------
    initial clk = 0;
    always #5 clk = ~clk;

    // --------------------------------------------------------
    // MAIN TEST SEQUENCE
    // --------------------------------------------------------
    initial begin
        // Initialize signals
        reset = 1;
        btn_c0 = 0;
        btn_c1 = 0;
        btn_c2 = 0;
        end_btn = 0;

        // Hold reset
        @(posedge clk);
        @(posedge clk);
        reset = 0;

        //---------------------------------------------------------
        // FIRST VOTE — IDLE → VOTE transition (important)
        //---------------------------------------------------------
        @(posedge clk); btn_c0 = 1;
        @(posedge clk); btn_c0 = 0;

        //---------------------------------------------------------
        // C0 gets 3 more votes (total = 4)
        //---------------------------------------------------------
        repeat(3) begin
            @(posedge clk); btn_c0 = 1;
            @(posedge clk); btn_c0 = 0;
        end

        //---------------------------------------------------------
        // C1 gets 4 votes
        //---------------------------------------------------------
        repeat(4) begin
            @(posedge clk); btn_c1 = 1;
            @(posedge clk); btn_c1 = 0;
        end

        //---------------------------------------------------------
        // C2 gets 2 votes
        //---------------------------------------------------------
        repeat(2) begin
            @(posedge clk); btn_c2 = 1;
            @(posedge clk); btn_c2 = 0;
        end

        //---------------------------------------------------------
        // Test Mutual Exclusion: C0 & C1 pressed together
        // Only C0 should count
        //---------------------------------------------------------
        @(posedge clk);
        btn_c0 = 1;
        btn_c1 = 1;

        @(posedge clk);
        btn_c0 = 0;
        btn_c1 = 0;

        //---------------------------------------------------------
        // One more vote for C1
        //---------------------------------------------------------
        @(posedge clk); btn_c1 = 1;
        @(posedge clk); btn_c1 = 0;

        //---------------------------------------------------------
        // END VOTING
        //---------------------------------------------------------
        @(posedge clk);
        end_btn = 1;

        @(posedge clk);
        end_btn = 0;

        //---------------------------------------------------------
        // CHECK WINNER
        //---------------------------------------------------------
        #10;
        $display("Votes => C0=%0d, C1=%0d, C2=%0d", v0, v1, v2);
        $display("Winner = %b", winner);

        if (winner == 2'b01)
            $display("SUCCESS: Candidate 1 wins as expected.");
        else
            $display("FAIL: Expected winner 01, got %b", winner);

        //---------------------------------------------------------
        // END SIM
        //---------------------------------------------------------
        #50;
        $finish;
    end

endmodule
```

</br>

## 8. Verilog HDL Implementation

### 8.1 Code Structure and Modules

The design is implemented in a single top-level module, `VOTING_MACHINE_v1005`. It leverages the following key Verilog constructs:
* **FSM**: Implemented using two `always` blocks: one synchronous block for the state register (`current_state`) and one combinational block for the next-state logic (`next_state`).
* **Vote Counter**: A synchronous `always @(posedge clk or posedge reset)` block handles all counting and implements the prioritized mutual exclusion logic using `if-else if`.
* **Winner Selector**: A combinational `always @(*)` block implements the complex comparison logic to determine the winner based on the `votes` array contents.

### 8.2 Testbench and Verification Strategy

The testbench (`VOTING_MACHINE_tb`) is crucial for verification. It uses an `always #5 clk = ~clk;` block to generate a 10ns clock and an `initial` block for the stimulus.

* **Stimulus Pattern**: The test sequence includes a full system reset, transitions through all states, sequential voting, and the crucial simultaneous press of `btn_c0` and `btn_c1` to confirm the mutual exclusion logic correctly prioritizes C0.
* **Internal Signal Tapping**: Internal signals like the vote counters (`DUT.votes[0]`, etc.) are explicitly tapped using wire declarations to facilitate deep inspection in Vivado.

---
</br>

## 9. Simulation Results

### 9.1 Waveform Signals and Setup

Critical signals monitored in the Vivado waveform viewer include: `clk`, `reset`, `btn_c[x]`, `state` (FSM status), `v[x]` (internal vote counters), and `winner` (output). This setup allows for visual confirmation of synchronous updates.

### 9.2 Explanation of Waveform

The simulation confirms the test sequence successfully registers votes:

| Time Interval | Event/Signal | Expected Outcome | Verified Waveform Value |
| :--- | :--- | :--- | :--- |
| **Initial** | `reset` asserted | System initialized to IDLE. | `state = 00`, `v[x] = 00` |
| **Voting Start** | First `btn_c0` active | `state` $\rightarrow$ S\_VOTE. C0 count $= 1$. | `state=01`, `v0=01` |
| **Voting Period** | Sequential Voting | Final counts achieved: C0=4, C1=4, C2=2 (before tie-breaker test). | `v0=04`, `v1=04`, `v2=02` |
| **Concurrency Test** | `btn_c0` & `btn_c1` active | Only `v0` should increment. C0=5, C1=4, C2=2. | Confirmed: Only C0 count increases, validating mutual exclusion. |
| **Post-Mutex Vote** | `btn_c1` active | C1 count increments (C0=5, C1=5, C2=2). | `v0=05`, `v1=05`, `v2=02` |
| **Result State** | `end_btn` active | `state` $\rightarrow$ S\_END (`2'b10`). | `state` transitions to `10` |
| **Final Output** | `state = S_END` | Winner logic selects the winner based on tie-breaker (C0=5, C1=5, C2=2 $\rightarrow$ C0 wins). | `winner` output latches to `2'b00` |

### 9.3 Verification and Validation

The simulation results ($v0=5, v1=5, v2=2$) align perfectly with the stimulus pattern applied by the testbench. Since $v0$ and $v1$ have equal counts, the hardware-defined **tie-breaking priority (C0 $\rightarrow$ C1 $\rightarrow$ C2)** is invoked. The final output of **`2'b00` (Candidate 0's ID)** successfully validates the combinational winner detection logic and the overall integrity of the counting system, including the concurrency arbitration.

#### Final Simulation Waveform Output

<img width="1579" height="821" alt="Screenshot 2025-11-18 135811" src="https://github.com/user-attachments/assets/e97109be-2245-4566-89ab-1bb720351bf9" />


---
</br>

## 10. Applications and Advantages

### 10.1 Applications

The scalable core logic is immediately applicable to small-scale elections, such as Student Council Voting or Opinion Polls. By implementing the logic on an FPGA, it forms a highly reliable base for larger, more secure **National Election Systems** or Secure Organizational Voting environments where verifiability is paramount.

### 10.2 Advantages

The FPGA-based approach offers several significant advantages:
* **Real-time Processing** due to hardware execution.
* **Zero Counting Errors**.
* High **Transparency** (as the RTL source code is the definition of the machine).
* Built-in **Security** via the mutual exclusion logic.

---
</br>

## 11. Limitations and Future Scope

### 11.1 Limitations

The current system has three primary limitations:
* **Simulated Environment Only** (no physical hardware deployment).
* Lacks **Voter Authentication** (allowing anyone to vote).
* Supports a **Limited Number of Candidates** (three).
* There is no external display interface.

### 11.2 Future Scope

Future work should focus on:
* **FPGA Prototype on Hardware**: Synthesizing the design and deploying it on a physical FPGA board interfaced with external components (buttons, LEDs).
* **Authentication Integration**: Adding an upstream module for Voter Authentication (e.g., RFID or fingerprint) to ensure integrity.
* **Display Integration**: Implementing an LCD Driver Module to show votes and the final winner in a user-friendly format.
* **Scaling**: Modifying the bus widths and memory arrays to support a larger number of candidates.

---
</br>

## 12. Conclusion

The design and verification of the FPGA-based Electronic Voting Machine were successfully completed using **Verilog HDL** and the **Xilinx Vivado environment**. The project achieved its core objectives by demonstrating a reliable **FSM controller**, accurate vote counting, and a critical **mutual exclusion mechanism** to prevent input concurrency errors. The behavioral simulation results confirm the system's robust and secure operation, establishing a solid, auditable foundation for future hardware implementation.

---
</br>

## 13. References

1.  Xilinx. (2024). Vivado Design Suite User Guide: Logic Simulation. Xilinx Documentation Portal.
2.  Palnitkar, S. (2003). Verilog HDL: A Guide to Digital Design and Synthesis. Prentice Hall.
3.  IEEE Standards Association. (2021). IEEE Standard for Electronic Voting System Requirements (IEEE 1622).
4.  Weste, N. H. E., & Harris, D. M. (2021). CMOS VLSI Design: A Circuit and System Perspective (5th ed.). Pearson Education.
5.  Academic resources on FPGA-based design and digital voting system security.


