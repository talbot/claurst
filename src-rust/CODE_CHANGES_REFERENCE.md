# Code Changes Reference

## Key Implementation Details

### 1. Case Conversion Operators

#### VimOperator Enum (prompt_input.rs, lines 106-115)
```rust
pub enum VimOperator {
    Delete,
    Change,
    Yank,
    /// Uppercase region (gU).
    Uppercase,
    /// Lowercase region (gu).
    Lowercase,
}
```

#### Helper Functions (prompt_input.rs, after line 344)
```rust
/// Convert text region to uppercase.
fn uppercase_region(text: &str) -> String {
    text.chars().map(|c| {
        c.to_uppercase().next().unwrap_or(c)
    }).collect()
}

/// Convert text region to lowercase.
fn lowercase_region(text: &str) -> String {
    text.chars().map(|c| {
        c.to_lowercase().next().unwrap_or(c)
    }).collect()
}
```

#### apply_operator_range() Function Enhancement
Added handling for case conversion in match statement:
```rust
VimOperator::Uppercase => {
    let upper = uppercase_region(selected);
    let new_text = format!("{}{}{}", &text[..from], upper, &text[to..]);
    (new_text, from)
}
VimOperator::Lowercase => {
    let lower = lowercase_region(selected);
    let new_text = format!("{}{}{}", &text[..from], lower, &text[to..]);
    (new_text, from)
}
```

#### vim_g() Function Enhancement
Added case conversion operator handling:
```rust
"U" => {
    // `gU` — start case conversion uppercase operator
    *pending = VimPendingState::Operator { op: VimOperator::Uppercase, count };
    false
}
"u" => {
    // `gu` — start case conversion lowercase operator
    *pending = VimPendingState::Operator { op: VimOperator::Lowercase, count };
    false
}
```

#### vim_operator() Function Enhancement
Added case conversion operator matching:
```rust
let op_char = match op {
    VimOperator::Delete => "d",
    VimOperator::Change => "c",
    VimOperator::Yank => "y",
    VimOperator::Uppercase => "U",
    VimOperator::Lowercase => "u",
};

// In doubled operator section:
if op != VimOperator::Yank {
    let new_content = match op {
        VimOperator::Delete => String::new(),
        VimOperator::Change => {
            *mode = VimMode::Insert;
            String::new()
        }
        VimOperator::Uppercase => uppercase_region(selected),
        VimOperator::Lowercase => lowercase_region(selected),
        VimOperator::Yank => unreachable!(),
    };
    text.drain(ls..le);
    text.insert_str(ls, &new_content);
    *cursor = ls;
    return true;
}
```

### 2. Go to Line Dialog

#### GoToLineDialog Struct (app.rs, after line 144)
```rust
/// State for the Go to Line dialog (Ctrl+G in message pane).
#[derive(Debug, Clone)]
pub struct GoToLineDialog {
    /// Input field for line number.
    pub input: String,
    /// Whether the dialog is currently active.
    pub active: bool,
    /// Total number of lines (for validation feedback).
    pub total_lines: usize,
}

impl GoToLineDialog {
    pub fn new() -> Self {
        Self {
            input: String::new(),
            active: false,
            total_lines: 0,
        }
    }

    pub fn open(&mut self, total_lines: usize) {
        self.input.clear();
        self.active = true;
        self.total_lines = total_lines;
    }

    pub fn close(&mut self) {
        self.active = false;
        self.input.clear();
    }

    /// Parse the input as a line number (1-indexed).
    /// Returns None if invalid or out of range.
    pub fn parse_line_number(&self) -> Option<usize> {
        let line_num: usize = self.input.trim().parse().ok()?;
        if line_num >= 1 && line_num <= self.total_lines {
            Some(line_num)
        } else {
            None
        }
    }
}
```

#### App Struct Integration (app.rs, line ~507)
```rust
/// Go to Line dialog (Ctrl+G in message pane).
pub go_to_line_dialog: GoToLineDialog,
```

#### App::new() Initialization (app.rs, line ~749)
```rust
go_to_line_dialog: GoToLineDialog::new(),
```

## Usage Examples

### Case Conversion Usage

#### Uppercase Examples
```vim
gUw      " Uppercase word
gUU      " Uppercase entire line
gU$      " Uppercase from cursor to end of line
3gUU     " Uppercase 3 lines
gUe      " Uppercase to end of word
gU0      " Uppercase from cursor to start of line
```

#### Lowercase Examples
```vim
guw      " Lowercase word
guu      " Lowercase entire line
gu$      " Lowercase from cursor to end of line
3guu     " Lowercase 3 lines
gue      " Lowercase to end of word
gu0      " Lowercase from cursor to start of line
```

#### Toggle Case
```vim
~        " Toggle case of character at cursor
3~       " Toggle case of 3 characters
```

### Go to Line Dialog Usage

```rust
// Activation (to be implemented in event handler)
if key == Ctrl+G {
    app.go_to_line_dialog.open(total_lines);
}

// User input handling
if dialog.active {
    if key.is_digit() {
        dialog.input.push(key);
    }
    if key == Enter {
        if let Some(line) = dialog.parse_line_number() {
            app.jump_to_line(line);
        }
        dialog.close();
    }
    if key == Escape {
        dialog.close();
    }
}
```

## Integration Points

### For Event Handlers
- Ctrl+G in message pane → activate go_to_line_dialog
- Character input → append to dialog.input
- Enter → parse_line_number() and jump
- Escape → dialog.close()

### For Rendering
- Check `go_to_line_dialog.active`
- Display input overlay with prompt
- Show validation feedback (valid/invalid)
- Show total lines count

### For Vim Operators
- Case conversion works automatically with all vim motions
- No additional event handler required
- Integrates through existing apply_vim_key() system

## Testing Patterns

### Case Conversion Tests
```rust
#[test]
fn test_uppercase_word() {
    let mut text = "hello world".to_string();
    let mut cursor = 0;
    // Simulate gUw
    uppercase_word(&mut text, &mut cursor);
    assert_eq!(text, "HELLO world");
}

#[test]
fn test_lowercase_line() {
    let mut text = "HELLO WORLD\nline2".to_string();
    let mut cursor = 0;
    // Simulate guu
    lowercase_line(&mut text, &mut cursor);
    assert_eq!(text, "hello world\nline2");
}
```

### Go to Line Tests
```rust
#[test]
fn test_parse_valid_line() {
    let dialog = GoToLineDialog {
        input: "42".to_string(),
        active: true,
        total_lines: 100,
    };
    assert_eq!(dialog.parse_line_number(), Some(42));
}

#[test]
fn test_parse_invalid_line() {
    let dialog = GoToLineDialog {
        input: "150".to_string(),
        active: true,
        total_lines: 100,
    };
    assert_eq!(dialog.parse_line_number(), None);
}

#[test]
fn test_parse_non_numeric() {
    let dialog = GoToLineDialog {
        input: "abc".to_string(),
        active: true,
        total_lines: 100,
    };
    assert_eq!(dialog.parse_line_number(), None);
}
```

## Performance Notes

### Time Complexity
- Case conversion: O(n) where n = selected text length
- Go to Line parsing: O(m) where m = input string length

### Space Complexity
- Case conversion: O(n) for result string
- Go to Line dialog: O(1) - fixed size struct (3 fields)

### Memory Allocations
- One String allocation per case conversion operation
- No allocations for Go to Line dialog operations

## Dependencies

No new external dependencies added. Implementation uses only:
- `std::str::chars()` - Unicode iteration
- `char::to_uppercase()`, `char::to_lowercase()` - Built-in unicode conversion
- `String::parse()` - Standard parsing

All functionality built on existing Rust standard library.
