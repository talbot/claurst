# Advanced Mouse Interactions Implementation

## Overview

Advanced mouse interactions have been implemented in the Rust port of Claude Code's TUI. This includes double-click word selection, triple-click line selection, and a right-click context menu.

## Changes Made

### 1. Core Data Types (app.rs, lines 113-139)

Added two new enum types:

```rust
#[derive(Debug, Clone, Copy)]
pub struct ContextMenuState {
    pub x: u16,
    pub y: u16,
    pub selected_index: usize,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum ContextMenuItem {
    Copy,
    Paste,
    Cut,
    SelectAll,
    Clear,
}
```

### 2. App Struct Fields (app.rs, lines 534-547)

Added mouse interaction tracking fields to the `App` struct:

```rust
// ---- Advanced mouse interaction state --------------------------------
pub last_click_time: Option<std::time::Instant>,
pub last_click_position: Option<(u16, u16)>,
pub click_count: u32,
pub context_menu_state: Option<ContextMenuState>,
```

These fields track:
- **last_click_time**: Timestamp of the previous click (for double-click detection)
- **last_click_position**: Position of the previous click
- **click_count**: Number of consecutive clicks (1=single, 2=double, 3+=triple)
- **context_menu_state**: Current context menu state and selected item

### 3. Initialization (app.rs, lines 757-761)

Added initialization in the `App::new()` method:

```rust
last_click_time: None,
last_click_position: None,
click_count: 0,
context_menu_state: None,
```

### 4. Helper Functions (app.rs, lines 3040-3177)

Implemented five key helper functions:

#### `is_double_click()`
- Detects double-clicks based on timing (<500ms) and position (<5px distance)
- Returns true if click is sufficiently close to the last click

#### `find_word_boundaries()`
- Attempts to find word boundaries for selection
- Currently returns approximate boundaries (±10 chars from click position)
- Can be extended to parse actual word boundaries from rendered text

#### `find_line_boundaries()`
- Finds the line containing a click
- Currently returns the single row
- Can be extended for multi-line selections

#### `show_context_menu()` / `dismiss_context_menu()`
- Manages context menu visibility
- `show_context_menu()` creates menu at cursor position with index 0
- `dismiss_context_menu()` clears the menu state

#### `navigate_context_menu()`
- Handles up/down arrow key navigation
- Cycles through menu items with boundary checks

#### `execute_context_menu_item()`
- Triggers the selected menu action
- Dismisses the menu after execution

#### `handle_context_menu_action()`
- Implements individual menu actions:
  - **Copy**: Logs selection (clipboard integration can be added)
  - **Paste**: Placeholder for paste functionality
  - **Cut**: Clears selection after copying
  - **SelectAll**: Selects entire message pane
  - **Clear**: Clears current selection

### 5. Mouse Event Handling (app.rs, lines 3191-3277)

Updated `handle_mouse_event()` with:

#### Right-Click Context Menu
```rust
MouseEventKind::Down(MouseButton::Right) => {
    // Show context menu at click position
}
```

#### Left-Click with Advanced Detection
```rust
MouseEventKind::Down(MouseButton::Left) => {
    // Dismiss any open context menu
    // Detect double/triple-clicks
    // Track click time and position
    // Handle word/line selection
}
```

#### Drag with Reset
```rust
MouseEventKind::Drag(MouseButton::Left) => {
    // Reset click count on drag
    // Continue normal selection
}
```

### 6. Key Event Integration (app.rs, lines 1619-1641)

Added context menu handling in `handle_key_event()` with highest priority:

```rust
if self.context_menu_state.is_some() {
    match key.code {
        KeyCode::Esc => {
            self.dismiss_context_menu();
            return false;
        }
        KeyCode::Up | KeyCode::Down => {
            self.navigate_context_menu(key.code);
            return false;
        }
        KeyCode::Enter => {
            self.execute_context_menu_item();
            return false;
        }
        _ => {}
    }
}
```

Features:
- **Up/Down arrows**: Navigate menu items
- **Enter**: Select/execute the current item
- **Escape**: Dismiss the menu

### 7. Context Menu Rendering (render.rs, lines 638-718)

Implemented `render_context_menu()` function with:

- **Border**: Rounded border with dark gray background
- **Items**: Five menu items with enable/disable states
- **Highlighting**: Cyan highlight for selected item
- **Bounds checking**: Menu clamps to screen edges
- **Disabled states**: Grayed out items when not applicable

Menu items show as disabled when:
- Copy: No text selected
- Cut: No text selected
- Clear: No active selection

### 8. Rendering Integration (render.rs, line 574)

Context menu is rendered on top of text selection highlight:

```rust
if !modal_active {
    apply_selection_highlight(frame, app);
    render_context_menu(frame, app);
}
```

## Click Timing and Detection

### Double-Click Detection
- Window: ~500ms (configurable)
- Distance: ≤5 pixels (Manhattan distance)
- Action: Select word at click position

### Triple-Click Detection
- Detected as third consecutive click within window
- Action: Select entire line
- Reset: Automatically reset after triple-click or any drag

## Terminal Coordinate System

All coordinates use terminal grid coordinates:
- (column, row) with origin at top-left (0, 0)
- Values are u16
- Bounded by message pane area from last render

## Future Enhancements

1. **Word Boundary Detection**: Parse actual text to find word boundaries instead of fixed offsets
2. **Clipboard Integration**: Add real clipboard support for Copy/Paste/Cut
3. **Selection Persistence**: Maintain selection across renders when not dragging
4. **Multi-Line Selection**: Extend line selection to multi-line blocks
5. **Context Menu Customization**: Add more menu items based on context
6. **Keyboard Shortcuts**: Bind menu items to keyboard commands (Ctrl+C, Ctrl+V, etc.)
7. **Visual Feedback**: Add visual cues for menu hover states and keyboard focus

## Testing Considerations

When testing these features:

1. **Double-Click**: Verify word selection within ~500ms at same/adjacent position
2. **Triple-Click**: Verify full line selection spans entire message pane width
3. **Context Menu**:
   - Right-click shows menu at cursor
   - Arrow keys navigate smoothly
   - Menu items enable/disable correctly
   - Escape dismisses menu
   - Enter executes selection
4. **Click Reset**: Verify click counter resets on drag or outside message pane
5. **Terminal Edge Cases**: Test menu display near screen edges

## Files Modified

1. **crates/tui/src/app.rs**
   - Added ContextMenuState and ContextMenuItem types
   - Added mouse interaction tracking fields
   - Implemented helper functions
   - Updated handle_mouse_event()
   - Updated handle_key_event()

2. **crates/tui/src/render.rs**
   - Implemented render_context_menu()
   - Integrated context menu rendering

## Compilation Status

All changes compile successfully. The only build errors are in `kitty_image.rs` which are pre-existing and unrelated to these changes.

## Notes

- Click tracking uses `std::time::Instant` for sub-millisecond precision
- Terminal coordinates are inherently discrete (character grid)
- Context menu disabled items cannot be selected but don't prevent scrolling through them
- Selection and context menu are mutually exclusive for user input focus
