# Parametric FIFO (SystemVerilog)

An independent digital design and verification project focused on building a reusable FIFO component.

## Project Roadmap
- [x] Phase 1: Architecture and Specification Document (FWFT_FIFO_Architecture_Doc).
- [ ] Phase 2: Synchronous FIFO Core RTL implementation (with overflow/underflow protection).
- [ ] Phase 3: FWFT (First-Word Fall-Through) wrapper logic for zero-latency reads.
- [ ] Phase 4: Asynchronous FIFO extension (CDC, Double-Flop Synchronizers, Gray Code Pointers).
- [ ] Phase 5: Verification environment (Testbench).

## Data Flow Architecture

The following diagram illustrates the internal data and control paths, separating the Standard Core FIFO from the FWFT Pre-fetch wrapper logic.

```mermaid
flowchart LR
    %% External Inputs
    wr_en([wr_en])
    data_in([data_in])
    rd_en([rd_en])

    %% External Outputs
    data_out([data_out])
    full([full])
    empty([empty])
    overflow([overflow])
    underflow([underflow])

    %% Core FIFO Block
    subgraph Core_FIFO ["Sync FIFO Core"]
        direction TB
        wr_logic["Write Logic & Pointers"]
        mem[/"SRAM (Memory Array)"/]
        rd_logic["Read Logic & Pointers"]
        
        wr_logic -- Write Addr & Data --> mem
        mem -- Read Data --> rd_logic
    end

    %% FWFT Wrapper Block
    subgraph FWFT_Wrapper ["FWFT Logic (Pre-fetch)"]
        direction TB
        fwft_ctrl{"FWFT State Machine"}
        fwft_reg["Output Data Register"]
    end

    %% Write Connections
    data_in --> wr_logic
    wr_en --> wr_logic
    wr_logic -- "full / overflow" --> full & overflow

    %% Internal Read Connections
    rd_logic -- "core_dout" --> fwft_reg
    rd_logic -- "core_empty" --> fwft_ctrl
    fwft_ctrl -- "rd_en_internal" --> rd_logic

    %% Read Connections
    rd_en --> fwft_ctrl
    fwft_ctrl -- "empty / underflow" --> empty & underflow
    fwft_reg --> data_out
    
    %% Styling
    classDef core fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef wrapper fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    class Core_FIFO core;
    class FWFT_Wrapper wrapper;