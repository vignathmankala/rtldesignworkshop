# Day 3: Combinational and Sequential Optimization

Welcome to Day 3 of this workshop! Today we discuss optimization of combinational and sequential circuits, introducing techniques to enhance efficiency and performance.

---

## Table of Contents

- [1. Constant Propagation](#1-constant-propagation)
- [2. State Optimization](#2-state-optimization)
- [3. Cloning](#3-cloning)
- [4. Retiming](#4-retiming)
- [5. Labs on Optimization](#5-labs-on-optimization)
  - [Lab 1](#lab-1)
  - [Lab 2](#lab-2)
  - [Lab 3](#lab-3)
  - [Lab 4](#lab-4)
  - [Lab 5](#lab-5)
  - [Lab 6](#lab-6)

---

## 1. Constant Propagation

In VLSI design, constant propagation is a compiler optimization technique used to replace variables with their constant values during synthesis. This can simplify design and enhance performance.

**How it works:**  
Constant propagation analyzes the design code to identify variables with constant values. These are replaced directly, allowing tools to simplify logic and reduce circuit size.

**Benefits:**
- **Reduced Complexity:** Simpler logic, smaller circuit.
- **Performance Improvement:** Faster execution and reduced delays.
- **Resource Optimization:** Fewer gates or flip-flops required.

![Constant Propagation Example](https://github.com/user-attachments/assets/d7f06056-66c1-44af-99a8-623fdf5879be)

---

## 2. State Optimization

State optimization refines finite state machines (FSMs) to improve efficiency in IC design. It reduces the number of states, optimizes encoding, and minimizes logic.

**How it is done:**
- **State Reduction:** Merge equivalent states using algorithms.
- **State Encoding:** Assign optimal codes to states.
- **Logic Minimization:** Use Boolean algebra or tools for compact equations.
- **Power Optimization:** Techniques like clock gating reduce dynamic power.

---

## 3. Cloning

Cloning duplicates a logic cell or module to optimize performance, reduce power, or improve timing by balancing load or reducing wire length.

**How it’s done:**
- Identify critical paths using analysis tools.
- Duplicate the target cell/module.
- Redistribute connections to balance load.
- Place and route the cloned cell.
- Verify improvement via timing and power analysis.

![Cloning Example](https://github.com/user-attachments/assets/6bdd2c12-02a2-4ea5-895c-98e349b93bac)

---

## 4. Retiming

Retiming is a design optimization technique that improves circuit performance by repositioning registers (flip-flops) without changing functionality.

**How it is done:**
1. **Graph Representation:** Model circuit as a directed graph.
2. **Register Repositioning:** Move registers to balance path delays.
3. **Constraints Analysis:** Maintain timing and functional equivalence.
4. **Optimization:** Adjust register positions to minimize clock period and optimize power.

---

## 5. Labs on Optimization

### Lab 1

Below is the Verilog code for Lab 1:

```verilog
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```

**Explanation:**
- `assign y = a ? b : 0;` means:
  - If `a` is true, `y` is assigned the value of `b`.
  - If `a` is false, `y` is 0.

Follow the steps from [Day 1 Synthesis Lab](https://github.com/Ahtesham18112011/RTL_workshop/tree/main/Day_1#6-synthesis-lab-with-yosys) and add the following between `abc -liberty` and `synth -top`:
```shell
opt_clean -purge
```

<img width="1920" height="922" alt="opt1" src="https://github.com/user-attachments/assets/70f7af07-fc0b-43d6-8e4d-32c1668f096d" />
<img width="1920" height="922" alt="opt1gv" src="https://github.com/user-attachments/assets/e7016f23-52fb-484e-b1bf-bb16c4d29710" />


---

### Lab 2

Verilog code:

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

**Code Analysis:**
- Acts as a multiplexer:
  - `y = 1` if `a` is true.
  - `y = b` if `a` is false.
<img width="1920" height="922" alt="opt2" src="https://github.com/user-attachments/assets/cba982dd-513a-4dba-a673-8ab7dc26e0a9" />
<img width="1920" height="922" alt="opt2gv" src="https://github.com/user-attachments/assets/da9eb498-c3b5-4471-96d5-436c59ee2436" />



---

### Lab 3

Verilog code:

```verilog
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```

**Functionality:**  
2-to-1 multiplexer; `y = a ? 1 : b` (outputs `1` when `a` is true, otherwise `b`).

<img width="1920" height="922" alt="opt3" src="https://github.com/user-attachments/assets/83ed50ef-7b0b-40b2-99c3-0c22d59fe2ff" />
<img width="1920" height="922" alt="opt3gv" src="https://github.com/user-attachments/assets/49e3abdb-b0db-463c-a6e0-b23604718509" />


---

### Lab 4

Verilog code:

```verilog
module opt_check4 (input a , input b , input c , output y);
 assign y = a?(b?(a & c ):c):(!c);
 endmodule
```

**Functionality:**
- Three inputs (`a`, `b`, `c`), output `y`.
- Nested ternary logic:
  - If `a = 1`, `y = c`.
  - If `a = 0`, `y = !c`.
- Logic simplifies to:  
  `y = a ? c : !c`

<img width="1920" height="922" alt="opt4" src="https://github.com/user-attachments/assets/e58a43ec-27f8-4b63-9a7f-cf0d8a87532e" />
<img width="1920" height="922" alt="opt4gv" src="https://github.com/user-attachments/assets/b0512267-504d-47ab-9be3-77da2cb69f3d" />


---

### Lab 5

Verilog code:

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end
endmodule
```

**Functionality:**
- D flip-flop with:
  - Asynchronous reset to 0
  - Loads constant `1` when not in reset

<img width="1920" height="922" alt="dffconst1show" src="https://github.com/user-attachments/assets/bd1362e6-031b-49cb-b1d7-6952266442ce" />
<img width="1920" height="922" alt="gtkwaveconst1" src="https://github.com/user-attachments/assets/344ca9cc-552e-4eec-89a6-0c6ff3f4d835" />


---

### Lab 6

Verilog code:

```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end
endmodule
```

**Functionality:**
- D flip-flop always sets output `q` to `1` (regardless of reset or clock).

<img width="1920" height="922" alt="const2show" src="https://github.com/user-attachments/assets/c7780f0d-a699-4a96-bc4a-9801ff58a80b" />
<img width="1920" height="922" alt="const2gtkwave" src="https://github.com/user-attachments/assets/48b25fe0-cb25-4873-96e5-45bc99fa9d43" />
ASSIGNMENT:
D_flipflop


code for dff_const3,4,5


<img width="1920" height="922" alt="const3gvim" src="https://github.com/user-attachments/assets/de262207-ef7f-484b-a77b-c44d9125356f" />
<img width="1920" height="922" alt="const45gvim" src="https://github.com/user-attachments/assets/d5c2418d-bbdb-49ba-af8d-117d71d0f3e9" />
synthesis:


<img width="1920" height="922" alt="const3show" src="https://github.com/user-attachments/assets/41b95a70-c58b-4c75-9e74-5ce21d44f2a7" />
<img width="1920" height="922" alt="const4show" src="https://github.com/user-attachments/assets/b02cada5-54f0-49a3-bd7c-630e9c9afb91" />
<img width="1920" height="922" alt="const5show" src="https://github.com/user-attachments/assets/9b2bea43-6a83-4a08-97a3-1bd7dae1f3e0" />
simulation:


<img width="1920" height="922" alt="const4gtkwave" src="https://github.com/user-attachments/assets/b0cb865c-c876-4e03-9c79-30bd1cbb8242" />
<img width="1920" height="922" alt="const3gtkwave" src="https://github.com/user-attachments/assets/f1ebb60b-63fe-4118-ac64-e4aad8b53f9d" />
<img width="1920" height="922" alt="const5gtk" src="https://github.com/user-attachments/assets/93733de7-4e9a-4144-8143-70216d3cd22b" />


multiple_module

<img width="1920" height="922" alt="multoptgv1" src="https://github.com/user-attachments/assets/6bae8a12-5399-4b4c-8ecc-de1cf54991d8" />
<img width="1920" height="922" alt="multoptgv2" src="https://github.com/user-attachments/assets/3e715f0c-adf1-4574-87b0-325d637ce566" />
<img width="1920" height="922" alt="multiple1gv" src="https://github.com/user-attachments/assets/17dffe2e-7db4-4800-bb59-9890e4cc3f76" />

synthesis:


<img width="1920" height="922" alt="multiopt" src="https://github.com/user-attachments/assets/e71dbc9c-8bf9-4e36-b83a-869f51c4e137" />
<img width="1920" height="922" alt="multipleee1" src="https://github.com/user-attachments/assets/9bab5ca4-5a90-4c5a-8288-1b978b6afedb" />

counter:

codes

<img width="1920" height="922" alt="counteroptgvim" src="https://github.com/user-attachments/assets/84f8bd26-2525-4ee0-9141-9aa7b84e29f7" />

synthesis


<img width="1920" height="922" alt="countershow" src="https://github.com/user-attachments/assets/cb90f2e6-fc2f-4ef5-9d7f-84ae3cfdfb38" />
simulation
<img width="1920" height="922" alt="counteroptgtkwave" src="https://github.com/user-attachments/assets/04c6d0fc-68dc-4439-82e0-47376de56f0d" />


---

## Summary
- **Focus:** Optimization techniques for combinational and sequential circuits in digital design, with practical Verilog labs.
  
- **Topics Covered:**
  1. **Constant Propagation:** Replacing variables with constant values to simplify logic and improve circuit efficiency.
  2. **State Optimization:** Reducing states and optimizing encoding in finite state machines to use less logic and power.
  3. **Cloning:** Duplicating logic cells/modules to improve timing and balance load.
  4. **Retiming:** Repositioning registers in a circuit to enhance performance without altering its function.

  
