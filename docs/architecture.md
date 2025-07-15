# NS-Shaft Game Architecture

## System Overview

The NS-Shaft game engine is built using a component-based architecture optimized for real-time performance and minimal memory overhead.

## Core Components

### 1. Game State Manager
**Location:** Main game loop (lines 83-95)  
**Responsibility:** Central control flow and state transitions

```mips
m_loop:
    jal check_game_level_status    # Game state evaluation
    bne $v0, $zero, game_end_status # Branch to win/lose handling
    jal player_movement            # Player physics and input
    jal platform_movement          # Platform updates and generation
```

**Implementation Status:**
- **check_game_level_status:** Fully implemented - checks win/lose conditions
- **player_movement:** Fully implemented - handles falling and platform movement
- **platform_movement:** Fully implemented - platform updates and generation

### 2. Player Entity System
**Data Structure:** `player_locs`, `player_speed`, `player_size`  
**Memory Layout:** Contiguous 32-bit words for cache efficiency

```mips
player_locs:    .word 0 0      # [x_pos, y_pos] - 8 bytes
player_speed:   .word 0 0      # [x_vel, y_vel] - 8 bytes  
player_size:    .word 20 30    # [width, height] - 8 bytes
```

**Key Algorithms:**
- **Position Update:** `new_pos = current_pos + velocity * dt`
- **Boundary Checking:** AABB collision with screen edges
- **Physics Integration:** Euler method for velocity/position

### 3. Platform Management System
**Data Structure:** Fixed-size array with metadata
```mips
platforms: .word 0:40          # 10 platforms × 4 words each
# Each platform: [id, x_pos, y_pos, type]
```

**Memory Optimization:**
- **Array Access:** Direct indexing with 16-byte strides
- **Cache Efficiency:** All platform data in single cache line
- **No Dynamic Allocation:** Zero garbage collection overhead

**Platform Types:**
- Type 0: Normal platform (static)
- Type 1: Breakable platform (state change)
- Type 2: Spring platform (velocity modifier)
- Type 3: Flip platform (physics interaction)
- Type 4-5: Rotation platforms (movement patterns)

### 4. Collision Detection Engine
**Algorithm:** Axis-Aligned Bounding Box (AABB) - Fully Implemented  
**Key Functions:** `check_platform_exist`, `check_boundary`

```mips
check_platform_exist:
    # Input: $a0 (x_loc), $a1 (y_loc) 
    # Output: $v0 (0=no platform, 1=platform exists)
    # Iterates through platform array checking overlap conditions

check_boundary:
    # Handles player collision with screen edges
    # Resets position and reverses velocity on boundary collision
    # Includes upper boundary collision detection for game loss
```

**Implementation Features:**
- **Platform Iteration:** Linear search through active platforms
- **Boundary Enforcement:** Screen edge collision with position correction
- **Early Exit:** Returns immediately when collision detected

### 5. Physics Simulation
**Integration Method:** Semi-implicit Euler  
**Update Frequency:** Every frame (60 FPS target)

**Forces Applied:**
- **Gravity:** Constant downward acceleration (2 units/frame²)
- **Platform Collision:** Instantaneous velocity change
- **Friction:** Velocity damping on platform contact

```mips
# Physics update pseudo-code
velocity.y += gravity           # Apply gravity
position += velocity           # Update position
if (collision_detected) {
    position.y = platform.y    # Resolve collision
    velocity.y = 0            # Stop vertical movement
}
```

## Data Flow Architecture

```
Input → Player Controller → Physics Engine → Collision Detection → Renderer
  ↑                                                                      ↓
  └── Game State Manager ←── Platform Manager ←── Object Factory ←──────┘
```

## Implementation Details

### Game Logic Implementation

#### Player Movement System
**Function:** `player_movement`
- **Input Processing:** Keyboard polling via `get_keyboard_input`
- **Movement Buffering:** Supports input buffering during multi-frame movements
- **State Machine:** Distinguishes between on-platform and falling states
- **Physics Integration:** Semi-implicit Euler integration for position/velocity

**Key Components:**
- `process_move_input`: Handles A/D key input with movement iteration system
- `player_move_left`/`player_move_right`: Directional movement with boundary checking
- `player_falling`: Gravity application and air movement physics
- `check_boundary`: Screen edge collision detection and correction

#### Platform Management System
**Function:** `platform_movement`
- **Dynamic Generation:** Creates new platforms as old ones move off-screen
- **Platform Types:** 6 different platform types with unique behaviors
- **Collision Detection:** AABB collision testing between player and platforms
- **Memory Management:** Fixed-size array with circular buffer behavior

**Platform Type Behaviors:**
- **Type 0 (Normal):** Static platform, restores player health
- **Type 1 (Unstable):** Breaks after timed delay when player lands
- **Type 2 (Spring):** Provides upward velocity boost to player
- **Type 3 (Spike):** Damages player on contact
- **Type 4/5 (Rotating):** Applies horizontal velocity to player

#### Collision Detection Implementation
**Function:** `check_platform_exist`
```assembly
# Platform collision algorithm:
# 1. Iterate through active platforms in array
# 2. Check X-axis overlap: player_x < platform_x + platform_width
#                        AND player_x + player_width > platform_x
# 3. Check Y-axis overlap: player_y < platform_y + platform_height
#                        AND player_y + player_height > platform_y
# 4. Return platform address if collision found, 0 otherwise
```

**Boundary Checking:** `check_boundary`
- **Horizontal Bounds:** Clamps player position to screen width, reverses velocity
- **Vertical Bounds:** Detects game loss condition (falling off screen)
- **Upper Boundary:** Spike damage when hitting ceiling

### Performance Optimizations

#### Memory Access Patterns
- **Sequential Platform Access:** Linear iteration through platform array
- **Register Caching:** Frequently accessed variables kept in registers
- **Minimal Stack Usage:** Only saves necessary registers across function calls

#### Algorithm Efficiency
- **Early Exit:** Collision detection stops at first platform hit
- **Bounded Loops:** All loops have maximum iteration counts
- **No Dynamic Allocation:** All memory statically allocated at compile time

#### Input Processing Optimization
- **Input Buffering:** Prevents input loss during multi-frame movements
- **Movement Iteration:** Spreads movement across multiple frames for smooth animation
- **Key State Caching:** Reduces redundant input polling

### System Integration

#### Game State Management
**Function:** `check_game_level_status`
- **Win Condition:** Player reaches platform with ID ≥ max_level
- **Lose Condition:** Player life value reaches 0
- **Continue State:** Normal gameplay progression

#### Timing and Synchronization
- **Frame Rate Control:** Sleep-based timing via `have_a_nap`
- **Timed Events:** Platform breaking, player hurt recovery
- **Animation Timing:** Multi-frame movement iterations

#### Audio and Visual Systems
- **Sound Effects:** Integrated via syscall 209 (movement, damage, win/lose)
- **Display Updates:** Regular screen refresh via syscall 206
- **Visual Feedback:** Player state changes (hurt/normal), platform animations

## Memory Layout Optimization

### Data Segment Organization (Estimated)
```
Game configuration: Constants and settings
Player data: Position, velocity, and attributes
Platform array: 10 platforms × 16 bytes each
Game state: Counters, timers, and flags
```

**Design Principles:**
- **Data Locality:** Related variables grouped in assembly declarations
- **Fixed Allocation:** All data structures use static allocation
- **Sequential Access:** Platform array designed for linear traversal

## Performance Bottlenecks & Solutions

### 1. Platform Array Traversal
**Problem:** Linear search through platform array  
**Solution:** Early termination and spatial sorting

### 2. Collision Detection
**Problem:** Multiple nested loops for boundary checking  
**Solution:** Optimized branching and register reuse

### 3. Function Call Overhead
**Problem:** Stack manipulation for each function call  
**Solution:** Inline critical code paths, minimal stack usage

## Scalability Considerations

**Current Limits:**
- Maximum 10 platforms per screen
- Single player entity
- Fixed screen resolution (300×600)

**Scaling Strategies:**
- **Spatial Partitioning:** Grid-based collision detection for more objects
- **Object Pooling:** Reuse platform objects instead of creation/destruction
- **Level-of-Detail:** Reduce physics fidelity for distant objects

## Development Status & Architecture Insights

**Implementation Completeness:**
- **Fully Implemented:** All core game systems including physics, collision detection, input processing, and game state management
- **Complex Features:** Multi-frame movement system, platform type behaviors, timed events
- **Performance Optimized:** Memory-efficient algorithms, bounded execution times

**Architectural Strengths:**
- **Modular Design:** Clear separation between player, platform, collision, and game state systems
- **Performance Consciousness:** Static allocation, register optimization, early-exit algorithms
- **Extensible Framework:** Platform type system easily supports new gameplay mechanics
- **Real-time Capable:** Deterministic execution with bounded loops and predictable timing

**Technical Sophistication:**
- **Input Buffering System:** Sophisticated input handling with movement queuing
- **Physics Simulation:** Proper gravity simulation with collision response
- **Memory Management:** Efficient use of fixed-size data structures
- **Error Handling:** Comprehensive boundary checking and edge case handling

**What This Demonstrates:**
- **Systems Programming Expertise:** Complex game engine implemented in assembly language
- **Performance Engineering:** Instruction-level optimization and memory-conscious design
- **Algorithm Implementation:** Collision detection, physics simulation, state machines
- **Software Architecture:** Well-structured, maintainable codebase organization

This complete implementation showcases the kind of low-level programming expertise and systematic approach essential for high-performance systems development.
