# Custom Syscall Documentation

## Overview

The NS-Shaft game engine utilizes custom MARS MIPS simulator syscalls (200-213) designed for educational game development. These syscalls provide a hardware abstraction layer for graphics, input, and game object management.

## Game Engine Syscalls (200-213)

### Core Game Management

#### Syscall 200: Create Game Object
**Purpose:** Initialize the game window and rendering context  
**Usage:** `li $v0, 200; syscall`
```mips
main:
    li $v0, 200      # Create Game Object
    syscall          # Initialize game window (300×600 pixels)
```
**Parameters:** None  
**Returns:** Game object ID in $v0

#### Syscall 203: Display Text Message
**Purpose:** Display text message at specified screen coordinates  
**Usage:** Text parameters in $a0-$a3
```mips
display_text:
    li $a0, -2              # Text ID (-2=win, -3=lose)
    li $a1, 80              # X coordinate
    li $a2, 280             # Y coordinate  
    la $a3, game_win        # Address of text string
    li $v0, 203             # Display text
    syscall
```
**Parameters:**
- $a0: Text ID (negative values for system messages)
- $a1: X coordinate for text position
- $a2: Y coordinate for text position
- $a3: Address of null-terminated string

**Returns:** None

### Player Management

#### Syscall 201: Create Player
**Purpose:** Initialize player entity with position and properties  
**Usage:** Player data in $a0-$a3
```mips
create_player:
    lw $a0, player_id        # Player ID
    lw $a1, player_x         # X position
    lw $a2, player_y         # Y position  
    lw $a3, player_size      # Size data address
    li $v0, 201              # Create player
    syscall
```
**Parameters:**
- $a0: Player ID
- $a1: Initial X coordinate  
- $a2: Initial Y coordinate
- $a3: Address of size data [width, height]

**Returns:** Player object reference

#### Syscall 204: Update Layer Display
**Purpose:** Update the layer number shown on screen  
**Usage:** Layer ID in $a0
```mips
update_layer_display:
    la $a0, current_platform_address
    lw $a0, 0($a0)          # Get platform address
    lw $a0, 0($a0)          # Get platform ID
    li $v0, 204             # Update layer display
    syscall
```
**Parameters:** $a0: Current layer/platform ID  
**Returns:** None

#### Syscall 205: Set Object Direction
**Purpose:** Set the visual direction/orientation of game objects  
**Usage:** Direction data in $a0-$a1
```mips
set_player_direction:
    li $a0, -1              # Player ID
    li $a1, 0               # Direction (0=left, 1=right, 2=front)
    li $v0, 205             # Set direction
    syscall
```
**Parameters:**
- $a0: Object ID 
- $a1: Direction code (0=left, 1=right, 2=front)

**Returns:** None

#### Syscall 210: Update Life Display
**Purpose:** Update player health/life visualization  
**Usage:** Life value in $a0
```mips
update_life_value:
    lw $a0, life_value      # Current life value
    li $v0, 210             # Update life display
    syscall
```
**Parameters:** $a0: Current life value (0-10)

### Platform Management

#### Syscall 202: Create Platform
**Purpose:** Create new platform object with specified properties  
**Usage:** Platform data in $a0-$a3
```mips
create_platform:
    lw $a0, platform_id     # Platform ID
    lw $a1, platform_x      # X position
    lw $a2, platform_y      # Y position
    lw $a3, platform_type   # Platform type (0-5)
    li $v0, 202             # Create platform
    syscall
```
**Parameters:**
- $a0: Platform ID (unique identifier)
- $a1: X coordinate
- $a2: Y coordinate  
- $a3: Platform type (see Platform Types section)

**Returns:** Platform object reference

#### Syscall 207: Update Object Location
**Purpose:** Move game object (player/platform) to new position  
**Usage:** Object data in $a0-$a2
```mips
update_object_location:
    lw $a0, object_id       # Object ID
    lw $a1, new_x          # New X position
    lw $a2, new_y          # New Y position
    li $v0, 207            # Update object location
    syscall
```
**Parameters:**
- $a0: Object ID
- $a1: New X coordinate
- $a2: New Y coordinate

### Random Generation

#### Syscall 211: Random Platform X Location
**Purpose:** Generate random X coordinate for platform placement  
**Usage:** `li $v0, 211; syscall`
```mips
generate_random_x:
    li $v0, 211            # Random X location
    syscall                # Returns random X in $v0
    # $v0 = random value [0, screen_width - platform_width]
```
**Parameters:** None  
**Returns:** Random X coordinate in $v0 (range: 0-200)

#### Syscall 212: Random Platform Type  
**Purpose:** Generate random platform type for variety  
**Usage:** `li $v0, 212; syscall`
```mips
generate_random_type:
    li $v0, 212            # Random platform type
    syscall                # Returns random type in $v0
    # $v0 = random type [0-5]
```
**Parameters:** None  
**Returns:** Random platform type in $v0 (range: 0-5)

### Display & Rendering

#### Syscall 206: Update Display
**Purpose:** Refresh screen with current game state  
**Usage:** `li $v0, 206; syscall`
```mips
update_display:
    li $v0, 206            # Update display
    syscall                # Refresh screen
```
**Parameters:** None  
**Returns:** None

#### Syscall 209: Audio/Sound System
**Purpose:** Play sound effects during gameplay  
**Usage:** Sound parameters in $a0-$a1
```mips
play_sound:
    li $a0, 3               # Sound ID (1=keypress, 2=lose, 3=win, 4=hurt)
    li $a1, 0               # Play mode (0=once, other=loop)
    li $v0, 209             # Play sound
    syscall
```
**Parameters:**
- $a0: Sound effect ID
- $a1: Play mode (0=play once, other values=repeat)

**Returns:** None

## Platform Types Reference

| Type | Name | Behavior | Use Case |
|------|------|----------|----------|
| 0 | Normal | Static platform | Basic gameplay |
| 1 | Breakable | Disappears after use | Temporary support |
| 2 | Spring | Increases jump height | Vertical boost |
| 3 | Flip | Rotates on contact | Dynamic obstacle |
| 4 | Left Rotation | Moves left continuously | Moving platform |
| 5 | Right Rotation | Moves right continuously | Moving platform |

## Performance Characteristics

### Implementation Notes

The syscalls are implemented within the MARS MIPS simulator's educational game framework. Performance characteristics depend on the Java VM and host system rather than the MIPS assembly code itself.

### Syscall Categories

**Object Management:**
- 200: Create Game Object
- 201: Create Player  
- 202: Create Platform
- 205: Set Object Direction
- 207: Update Object Location
- 213: Set Object Status

**Display Operations:**
- 203: Display Text Message
- 204: Update Layer Display
- 206: Update Display
- 210: Update Life Display

**Utility Functions:**
- 209: Audio/Sound System
- 211: Random X Location
- 212: Random Platform Type

### Memory Usage

**Per Object Memory Overhead:**
- Object creation: Implementation dependent
- Position updates: No additional memory
- Display operations: Framebuffer dependent

**Total Memory Budget:**
- Player objects: ~100 bytes estimated
- Platform objects: ~640 bytes estimated (10 platforms)
- Display buffer: Implementation dependent

## Error Handling

### Common Error Conditions
1. **Invalid Object ID:** Syscall fails silently
2. **Out-of-Bounds Coordinates:** Coordinates clamped to screen
3. **Invalid Platform Type:** Defaults to type 0 (normal)
4. **Memory Exhaustion:** Limited object count enforced

### Error Detection Strategy
```mips
# Validate parameters before syscall
blt $a1, $zero, invalid_x        # X < 0
bge $a1, screen_width, invalid_x # X >= screen_width
# ... parameter validation ...
li $v0, 202                     # Safe to call syscall
syscall
```

## Integration with Low-Latency Systems

### Relevance to Trading Applications

**Similar Patterns in HFT:**
1. **Hardware Abstraction:** Syscalls abstract hardware complexity
2. **Deterministic Performance:** Bounded execution time per operation
3. **Resource Management:** Limited object pools prevent allocation spikes
4. **Error Handling:** Graceful degradation under edge cases

**Performance Lessons:**
- **Batch Operations:** Group multiple updates into single display refresh
- **Cache Syscall Results:** Avoid redundant status checks
- **Async Pattern:** Display updates don't block game logic
- **Resource Pooling:** Object reuse prevents creation overhead

### Optimization Strategies

**Syscall Batching:**
```mips
# Inefficient: Multiple display updates
li $v0, 206; syscall    # Update 1
li $v0, 206; syscall    # Update 2
li $v0, 206; syscall    # Update 3

# Efficient: Single display update after all changes
# ... make all position updates ...
li $v0, 206; syscall    # Single display refresh
```

**Conditional Syscalls:**
```mips
# Only update display if something changed
beq $t0, $zero, skip_display    # No changes made
li $v0, 206                     # Update display
syscall
skip_display:
```

This syscall architecture demonstrates the kind of system-level programming interface design that's crucial in low-latency trading systems, where hardware abstraction must be balanced with performance requirements.
