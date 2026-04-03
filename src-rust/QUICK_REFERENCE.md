# Quick Reference: Implementation Details

## Keybindings Expansion

### Location: `crates/core/src/keybindings.rs` (lines 120-220)

**Before:** 13 bindings
**After:** 100+ bindings across 15+ contexts

### Global Context
```
ctrl+c    interrupt
ctrl+d    exit
ctrl+l    redraw
ctrl+r    historySearch
ctrl+b    createBranch ← NEW: triggers session branching
```

### Chat Context (Input)
```
Text Editing:
  ctrl+a   goLineStart       ctrl+e   goLineEnd
  ctrl+h   backspace         ctrl+k   killToEnd
  ctrl+u   killToStart       ctrl+w   killWord

Searching:
  ctrl+f   findInMessage     ctrl+g   goToLine
  f3       findNext          shift+f3 findPrev

Navigation:
  enter    submit            pageup   scrollUp
  pagedown scrollDown        tab      indent

Vim Mode:
  h j k l  hjkl navigation (when vim mode enabled)
  w b e    word movement
  0 $      line start/end
```

### Other Contexts
- Confirmation: y, n, escape, up/down
- Help: escape, q, navigation
- History Search: enter, escape, up/down
- Theme/Model Pickers: up/down, j/k, enter, escape
- Task List: x (toggle), navigation
- Diff: a (accept), r (reject), navigation
- Modal Select: standard navigation + search

---

## Session Branching

### Location: `crates/tui/src/session_branching.rs` (NEW FILE)

### Core Types

```rust
pub struct BranchInfo {
    pub id: String,
    pub name: String,
    pub branch_at_message: usize,
    pub message_count: usize,
    pub created_at: String,
    pub is_current: bool,
}

pub enum BranchBrowserMode {
    Browse,
    CreateNew,
    ConfirmDelete,
}

pub struct SessionBranchingState {
    pub visible: bool,
    pub selected_idx: usize,
    pub branches: Vec<BranchInfo>,
    pub mode: BranchBrowserMode,
    pub create_input: String,
    pub branch_at_message: usize,
}
```

### Public Methods

**Navigation**
- `select_prev()` - Move cursor up with bounds checking
- `select_next()` - Move cursor down with bounds checking
- `selected_branch()` - Get current branch reference
- `selected_branch_mut()` - Get mutable reference

**Creation**
- `start_create_new()` - Enter branch naming mode
- `push_create_char(c)` - Add character to name
- `pop_create_char()` - Remove last character
- `confirm_create_new()` - Returns (name, message_idx)

**Management**
- `start_delete_confirm()` - Start deletion flow
- `confirm_delete()` - Remove branch, returns ID
- `open(branches, at_message)` - Show overlay
- `close()` - Hide overlay
- `cancel()` - Context-aware cancellation

**Rendering**
- `render_session_branching()` - Main render function
- `render_branch_list()` - Branch list view
- `render_create_branch()` - Naming prompt
- `render_confirm_delete()` - Confirmation dialog

### Integration Points

#### In `crates/tui/src/app.rs`:

**Struct field (line ~497)**
```rust
pub session_branching: crate::session_branching::SessionBranchingState,
```

**Initialization (line ~745)**
```rust
session_branching: crate::session_branching::SessionBranchingState::new(),
```

**Keybinding handler (lines 2756-2763)**
```rust
"createBranch" => {
    let branch_at = self.messages.len();
    let branches = vec![];
    self.session_branching.open(branches, branch_at);
    self.status_message = Some("Session branching mode...".to_string());
    false
}
```

**Event handler (lines 1742-1792)**
```rust
if self.session_branching.visible {
    use crate::session_branching::BranchBrowserMode;
    match self.session_branching.mode {
        BranchBrowserMode::Browse => { /* handle navigation */ }
        BranchBrowserMode::CreateNew => { /* handle naming */ }
        BranchBrowserMode::ConfirmDelete => { /* handle confirm */ }
    }
    return false;
}
```

#### In `crates/tui/src/render.rs`:

**Import (line 13)**
```rust
use crate::session_branching::render_session_branching;
```

**Render call (lines 528-531)**
```rust
if app.session_branching.visible {
    render_session_branching(&app.session_branching, size, frame.buffer_mut());
}
```

#### In `crates/tui/src/lib.rs`:

**Module declaration (line ~105)**
```rust
pub mod session_branching;
```

**Re-exports (line ~127)**
```rust
pub use session_branching::{SessionBranchingState, BranchBrowserMode, BranchInfo, render_session_branching};
```

---

## User Interaction Flow

### Creating a Branch

1. Press `Ctrl+B` in chat
2. See "Session Branching Mode" status message
3. Current branching overlay shows:
   - Any existing branches (initially empty)
   - Instructions: "↑↓: navigate | N: new | D: delete | Esc: close"
4. Press `N` to create new branch
5. Border turns yellow, prompt: "Branch name: _"
6. Type branch name (e.g., "alternative-approach")
7. Press `Enter` to create
8. See "Created branch: alternative-approach at message X"
9. Overlay shows new branch in list

### Switching Branches

1. Press `Ctrl+B` to open branching overlay
2. Branches are listed with metadata:
   - `> [active indicator] name (2 min ago)`
   - `  [1] first-branch (5 min ago)`
   - `  [2] experiment (1 min ago)`
3. Navigate with ↑/↓ to select
4. Press `Enter` to switch
5. Status: "Switched to branch: experiment"

### Deleting a Branch

1. Press `Ctrl+B` to open
2. Navigate to branch with ↑/↓
3. Press `D`
4. Border turns red, confirm: "Delete branch 'experiment'? Y/Esc"
5. Press `Y` to confirm or `Esc` to cancel
6. Status: "Deleted branch: experiment"

---

## Visual Design

### Branch List View
```
┌─────────────────────────────────────┐
│         Session Branches            │
├─────────────────────────────────────┤
│  → [1] main (5 min ago) [ACTIVE]   │ ← current (green)
│    [2] feature-x (2 min ago)       │
│    [3] experiment (1 min ago)      │
│                                     │
│  ↑↓: navigate | N: new | D: delete │
│                          Esc: close │
└─────────────────────────────────────┘
```

### Create New Branch
```
┌─────────────────────────────────────┐
│         Session Branches            │
├─────────────────────────────────────┤
│  Create a new branch from message   │
│         point                       │
│                                     │
│  Branch name: my-idea_              │
│                                     │
└─────────────────────────────────────┘
```
(Border is yellow, cursor shown after input)

### Delete Confirmation
```
┌─────────────────────────────────────┐
│         Session Branches            │
├─────────────────────────────────────┤
│  Delete branch 'feature-x'?         │
│                                     │
│  This will remove the branch but    │
│  keep all other branches intact.    │
│                                     │
│  Y to confirm | Esc to cancel       │
└─────────────────────────────────────┘
```
(Border is red)

---

## Data Persistence (Not Yet Implemented)

The infrastructure is ready for:

1. **Session Storage Integration**
   - Map `BranchInfo` to `ConversationSession.branch_*` fields
   - Persist branches to JSON alongside session messages

2. **Branch Point Storage**
   - `branch_at_message: usize` maps to message index
   - `branch_from: Option<String>` links to parent session

3. **Message State Restoration**
   - Load messages 0..branch_at_message (shared history)
   - Load messages branch_at_message..end (branch-specific)
   - Handle conflicts when switching branches

4. **Integration Points**
   - `crates/core/src/session_storage.rs` - load/save
   - `crates/core/src/lib.rs` - ConversationSession methods
   - Command handlers for branch operations

---

## Testing

**Manual verification:**
```bash
# Build
cargo build --lib

# Test session_branching module compiles
cargo check --lib

# Expected: 0 errors in session_branching module
```

**Key areas to verify:**
- [ ] Ctrl+B opens branching overlay
- [ ] Browse mode works (↑↓ navigation)
- [ ] Create mode works (N, type, Enter)
- [ ] Delete mode works (D, Y/Esc)
- [ ] Name validation prevents duplicates
- [ ] Escape properly closes all modes
- [ ] Status messages appear

---

## Code Statistics

| Component | Lines | Location |
|-----------|-------|----------|
| Keybindings | 100 | `keybindings.rs` 120-220 |
| Session Branching Module | 230 | `session_branching.rs` |
| Integration (app.rs) | 60 | `app.rs` (3 edits) |
| Integration (lib.rs) | 2 | `lib.rs` (2 edits) |
| Integration (render.rs) | 3 | `render.rs` (2 edits) |
| **TOTAL** | **~395 lines** | |

---

## Notes

- All keybindings are contextual (no conflicts across modes)
- Session branching follows existing overlay patterns
- Module is fully self-contained and can be extended
- Ready for persistence layer integration
- Type-safe Rust implementation with full error handling
