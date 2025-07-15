# NS-Shaft: MIPS Assembly Game Engine

## Overview
A complete 2D platformer game implemented in pure MIPS assembly language, demonstrating low-level programming skills and performance-conscious design principles.

**Lines of Code:** 1,622 lines of MIPS assembly  
**Architecture:** Real-time game loop with deterministic execution patterns  
**Memory Management:** Stack-based allocation with no dynamic memory allocation

## Technical Highlights

### 🚀 Technical Features
- **Real-time collision detection** with linear search through active platforms
- **Fixed-size data structures** for predictable memory usage
- **Assembly-level game loop** with bounded execution paths
- **Manual register management** for frequently accessed variables

### 🔧 Low-Level Programming Concepts
- Direct hardware register usage in MIPS architecture
- Sequential data structure layout for cache efficiency
- Polling-based keyboard input processing
- Explicit memory management with predictable patterns

### ⚡ Programming Techniques Demonstrated
- **Structured assembly code** with clear function organization
- **Register allocation strategies** for frequently used variables
- **Bounded loops** for predictable execution behavior
- **Code organization** techniques for maintainable assembly

## Skills Demonstrated

This project showcases programming competencies relevant to performance-critical applications:

| Programming Skill | Demonstrated Through | Relevance |
|------------------|---------------------|-----------|
| **Assembly Programming** | 1,600+ lines of functional MIPS code | Low-level system understanding |
| **Memory Management** | Stack-based allocation, no dynamic memory | Predictable resource usage |
| **Performance Awareness** | Instruction counting and analysis | Optimization mindset |
| **System Architecture** | Game engine component design | Large system organization |
| **Real-time Constraints** | Bounded execution paths | Time-critical programming |

## Game Architecture

### Core Components
- **Player Entity System**: Position, velocity, and state management
- **Platform Management**: Dynamic creation/destruction with spatial indexing
- **Collision Detection**: Axis-aligned bounding box (AABB) algorithms
- **Physics Engine**: Gravity simulation and platform interactions
- **Input System**: Real-time keyboard event processing

### Estimated Performance Characteristics
- **Game Loop**: ~200-300 instructions per frame (estimated from code analysis)
- **Collision Detection**: Linear search through active platforms (typically 7-10)
- **Memory Footprint**: ~500 bytes data segment, no heap allocation
- **Execution Pattern**: Deterministic with bounded loops and fixed data structures

## Quick Start

### Prerequisites
- MARS MIPS Simulator (Educational version with syscalls 200-213)
- Java Runtime Environment

### Running the Game
```bash
# Clone the repository
git clone https://github.com/kel027/NS-Shaft.git
cd NS-Shaft

# Run with MARS simulator (require specfic version)
java -jar Mars.jar src/v0.asm
```

### Controls
- **A/D**: Move left/right on platforms
- **Objective**: Navigate upward through platforms to reach the top

## Technical Documentation

- [**Architecture Overview**](docs/architecture.md) - System design and component interaction
- [**Custom Syscalls**](docs/syscalls.md) - Game engine API documentation
- [**Memory Layout**](analysis/memory_layout.md) - Data structure organization

## Code Quality Features

### Memory Safety
- No buffer overflows through careful bounds checking
- Stack-based allocation with explicit size management
- Deterministic memory access patterns

### Error Handling
- Boundary condition checking in collision detection
- Input validation and edge case handling
- Robust game state management

## Development Insights

This project showcases understanding of:
- **Hardware-level programming** concepts
- **Real-time system constraints** and solutions
- **Performance optimization** at the instruction level
- **System architecture** for latency-sensitive applications

## Author
**Lau Chun Hin Kelvin**  
*Quantitative Developer & Blockchain Engineer*


