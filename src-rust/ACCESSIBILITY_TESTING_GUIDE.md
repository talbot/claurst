# Color Blind Accessibility Testing Guide

## Overview
This guide provides step-by-step instructions for testing the Deuteranopia theme and UI accessibility enhancements.

---

## Part 1: Visual Testing

### Test 1: Theme Selection
**Goal**: Verify Deuteranopia theme is selectable

**Steps**:
1. Launch Claude Code
2. Type `/theme` and press Enter
3. Look for "Deuteranopia" in the theme list
4. Verify it shows description: "Red-green color blind friendly — blue/yellow/gray palette"
5. Navigate to it and press Enter
6. Verify it's selected and persists on restart

**Expected Result**: ✓ Deuteranopia appears in list and applies successfully

---

### Test 2: Error Message Colors
**Goal**: Verify errors display in orange, not red

**Steps**:
1. With Deuteranopia theme active
2. Trigger an error (e.g., invalid command or failed tool)
3. Observe error message formatting

**Expected Visual**:
```
✗ Error
  Error message content here...
  More error details...
```

**Expected Colors**:
- Symbol (✗): RGB(255, 140, 0) - Orange
- Text: RGB(255, 140, 0) - Orange
- Background: Dark gray or black

**Color Values to Verify**:
- NOT red (255, 0, 0) ✓
- NOT green (0, 255, 0) ✓
- IS orange (255, 140, 0) ✓

---

### Test 3: Code Block Language Labels
**Goal**: Verify code blocks show language labels

**Steps**:
1. Ask Claude to provide code examples
2. Look at rendered code blocks
3. Observe the label above each code block

**Expected Visual**:
```
  [python]
  ┌──────────────────────────────────────────────────
  │ def hello():
  │     print("Hello, world!")
  │
  │ hello()
  └──────────────────────────────────────────────────
```

**Expected Details**:
- Language label appears above code block
- Format: `[language_name]`
- Styled in gray color with DIM modifier
- Clearly visible but not overwhelming

**Test Variations**:
- Code block with language: `[rust]`, `[javascript]`, `[java]`
- Code block without language: `[code]` (default)
- Multiple code blocks: Each shows correct language

---

### Test 4: Message Role Indicators
**Goal**: Verify user and assistant messages show role labels

**Steps**:
1. Send a message in the conversation
2. Look at Claude's response
3. Observe role labels on both messages

**Expected Visual - User Message**:
```
[You] › Your message content here
```

**Expected Visual - Assistant Message**:
```
[Claude] ◆ Assistant response here
           with multiple lines
           of content...
```

**Expected Details**:
- User messages: `[You]` in blue + `›` symbol
- Assistant messages: `[Claude]` in blue + `◆` symbol
- Labels appear at start of first line
- Help distinguish conversation flow

---

### Test 5: Color Palette Verification
**Goal**: Verify all theme colors are accessible

**With Deuteranopia Theme Active, Verify These Colors**:

| Element | RGB Value | Appears As | Notes |
|---------|-----------|-----------|--------|
| Error | (255, 140, 0) | Orange | ✓ Not red, not green |
| Success | (0, 150, 200) | Blue | ✓ Distinct from orange |
| Warning | (255, 180, 0) | Gold | ✓ Yellow-orange range |
| Info | (0, 255, 255) | Cyan | ✓ High contrast |
| Border | (100, 100, 100) | Gray | ✓ Neutral dividers |
| Disabled | (120, 120, 120) | Light Gray | ✓ Muted appearance |
| Text | (220, 220, 220) | White-gray | ✓ Readable on dark |

**Color Absence Check**:
- NO pure red (255, 0, 0) ✓
- NO pure green (0, 255, 0) ✓
- NO red/green distinction ✓

---

## Part 2: Accessibility Testing

### Test 6: Contrast Ratio Verification
**Goal**: Verify WCAG AA compliance (4.5:1 minimum)

**Using Online Tools**:
1. Visit: https://webaim.org/resources/contrastchecker/
2. Test error text:
   - Foreground: RGB(255, 140, 0) orange
   - Background: RGB(20, 20, 20) dark
   - Expected ratio: > 4.5:1 ✓

3. Test info text:
   - Foreground: RGB(220, 220, 220) light gray
   - Background: RGB(20, 20, 20) dark
   - Expected ratio: > 4.5:1 ✓

4. Test border/divider:
   - Foreground: RGB(100, 100, 100) gray
   - Background: RGB(20, 20, 20) dark
   - Expected ratio: > 4.5:1 ✓

**Expected Result**: All contrast ratios > 4.5:1

---

### Test 7: Color Blind Simulator Testing
**Goal**: Verify theme works with simulated color blindness

**Using Coblis (http://www.color-blindness.com/coblis-color-blindness-simulator/)**:

1. Take a screenshot of error message
2. Upload to Coblis
3. Simulate "Deuteranopia"
4. Verify error message still visible
5. Verify text is readable
6. Verify color distinction clear

**Expected Result**: All UI elements remain visible and distinguishable

**Using Color Oracle (https://www.colororacle.org/)**:
1. Install Color Oracle
2. Enable Deuteranopia filter
3. Use Claude Code with Deuteranopia theme
4. Verify error messages visible
5. Verify code blocks identifiable
6. Verify role labels readable

---

### Test 8: Screen Reader Testing
**Goal**: Verify text alternatives accessible

**Manual Testing**:
1. Read error message text (not relying on color)
2. Read role labels: `[You]` and `[Claude]`
3. Read code block language: `[rust]`, `[python]`, etc.
4. Verify all information understandable

**Expected Result**: All UI information understandable from text alone

**Using NVDA (Free Screen Reader)**:
1. Install NVDA on Windows
2. Enable screen reader
3. Read through error message
4. Read through conversation
5. Verify labels and structure clear

---

### Test 9: No Color-Only Information
**Goal**: Verify no critical info conveyed by color alone

**Checklist**:
- [ ] Error indicated by: ✗ symbol + orange color + text
- [ ] Success indicated by: Blue color + text + context
- [ ] Warning indicated by: Gold color + text + context
- [ ] Disabled state indicated by: Gray color + text + symbol
- [ ] Code blocks identified by: Language label + [code] + structure
- [ ] Messages identified by: [You]/[Claude] label + symbol + color

**Expected Result**: All checkmarks pass

---

### Test 10: Multi-Modal Distinction
**Goal**: Verify elements distinguishable by multiple means

**Test Elements**:

**Error Message**:
- [ ] Visible by color (orange) alone? YES
- [ ] Visible by symbol (✗) alone? YES
- [ ] Visible by text ("Error") alone? YES
- [ ] Multiple means work together? YES

**Role Labels**:
- [ ] Distinguishable by color (blue)? YES
- [ ] Distinguishable by text ([You]/[Claude])? YES
- [ ] Distinguishable by symbol (›/◆)? YES
- [ ] Together provide clear distinction? YES

**Code Blocks**:
- [ ] Visible by language label? YES
- [ ] Visible by border/structure? YES
- [ ] Visible by context? YES

---

## Part 3: Regression Testing

### Test 11: Other Themes Still Work
**Goal**: Verify no regression in other themes

**For Each Theme** (Default, Dark, Light, Solarized, Nord, Dracula, Monokai):

1. Select theme from `/theme` menu
2. Observe error messages
3. Observe code blocks
4. Observe role labels
5. Verify all elements render correctly
6. Verify colors are appropriate for theme
7. Verify no visual artifacts

**Expected Result**: All themes work correctly, unchanged

---

### Test 12: All Message Types
**Goal**: Verify all message types render correctly

**Test Each Type**:
- [ ] User text messages - Show [You] label
- [ ] Assistant text messages - Show [Claude] label
- [ ] Tool use blocks - Display tool name + parameters
- [ ] Tool results - Show output or error appropriately
- [ ] Thinking blocks - Display with [thinking] label
- [ ] Code blocks in markdown - Show language label
- [ ] Error tool results - Show orange ✗ Error

**Expected Result**: All message types render with enhancements

---

### Test 13: Terminal Compatibility
**Goal**: Verify works on different terminals

**Test On**:
- [ ] Windows Terminal
- [ ] PowerShell
- [ ] WSL Ubuntu
- [ ] MacOS Terminal
- [ ] Linux xterm

**For Each Terminal**:
1. Launch Claude Code
2. Select Deuteranopia theme
3. Verify colors display correctly
4. Verify symbols render (✗, ◆, ›, ┌, └, │)
5. Verify text is readable

**Expected Result**: Works consistently across all terminals

---

## Part 4: User Experience Testing

### Test 14: Theme Persistence
**Goal**: Verify theme selection persists

**Steps**:
1. Select Deuteranopia theme
2. Close Claude Code
3. Reopen Claude Code
4. Check active theme from settings

**Expected Result**: Theme is Deuteranopia (persisted)

---

### Test 15: Performance
**Goal**: Verify no performance degradation

**Measures**:
1. Startup time: Should be same as before
2. Rendering speed: No lag when displaying messages
3. Memory usage: No significant increase
4. Theme switching: Instant (< 100ms)

**Expected Result**: No measurable performance impact

---

### Test 16: Configuration
**Goal**: Verify theme in config file

**Steps**:
1. Set Deuteranopia theme
2. Locate config file: `~/.clauderc` or `config.json`
3. Open with text editor
4. Find theme setting: `"theme": "deuteranopia"`
5. Manually change to different theme
6. Restart Claude Code
7. Verify new theme applied

**Expected Result**: Config file correctly reflects theme choice

---

## Test Execution Checklist

### Pre-Testing
- [ ] Build successful (0 errors)
- [ ] Claude Code launches without errors
- [ ] All themes selectable from menu

### Color Blind Specific
- [ ] Test 1: Theme selection - PASS
- [ ] Test 2: Error colors - PASS
- [ ] Test 3: Code block labels - PASS
- [ ] Test 4: Role indicators - PASS
- [ ] Test 5: Color palette - PASS

### Accessibility Standards
- [ ] Test 6: Contrast ratios - PASS
- [ ] Test 7: Color blind simulator - PASS
- [ ] Test 8: Screen reader - PASS
- [ ] Test 9: No color-only info - PASS
- [ ] Test 10: Multi-modal - PASS

### Regression Testing
- [ ] Test 11: Other themes - PASS
- [ ] Test 12: Message types - PASS
- [ ] Test 13: Terminal compatibility - PASS

### User Experience
- [ ] Test 14: Theme persistence - PASS
- [ ] Test 15: Performance - PASS
- [ ] Test 16: Configuration - PASS

---

## Color Reference Card

### Deuteranopia Palette

```
┌─────────────────────────────────────────────────┐
│ DEUTERANOPIA COLOR PALETTE                      │
├─────────────────────────────────────────────────┤
│ Error        │ RGB(255, 140,   0) │ Orange      │
│ Success      │ RGB(  0, 150, 200) │ Blue        │
│ Warning      │ RGB(255, 180,   0) │ Gold        │
│ Info         │ RGB(  0, 255, 255) │ Cyan        │
│ Action       │ RGB(  0, 150, 200) │ Blue        │
│ Disabled     │ RGB(120, 120, 120) │ Gray        │
│ Accent Prime │ RGB(  0, 150, 200) │ Blue        │
│ Accent Sec   │ RGB(180, 140, 255) │ Purple      │
│ Text Light   │ RGB(220, 220, 220) │ White-Gray  │
│ Text Dark    │ RGB( 40,  40,  40) │ Dark Gray   │
│ Border       │ RGB(100, 100, 100) │ Medium Gray │
└─────────────────────────────────────────────────┘
```

### What to Avoid

```
❌ NOT Red               RGB(255,   0,   0)
❌ NOT Green            RGB(  0, 255,   0)
❌ NOT Pure Yellow      RGB(255, 255,   0)
❌ NOT Light Red        RGB(255, 100, 100)
❌ NOT Light Green      RGB(100, 255, 100)
```

---

## Known Limitations & Future Work

### Current Limitations
1. Only Deuteranopia supported (not Protanopia or Tritanopia)
2. Colors not user-customizable
3. No high-contrast variant
4. Terminal must support 24-bit true color

### Future Enhancements
1. Add Protanopia theme (red-blind)
2. Add Tritanopia theme (blue-yellow blind)
3. Add high-contrast theme
4. User-customizable color schemes
5. Theme auto-detection

---

## Troubleshooting

### Issue: Theme doesn't apply
**Solution**:
1. Restart Claude Code
2. Check that theme is in settings file
3. Verify no errors in console
4. Try other theme, then Deuteranopia again

### Issue: Colors look wrong
**Solution**:
1. Verify terminal supports 24-bit color
2. Check terminal color settings
3. Try different terminal application
4. Verify Deuteranopia theme is active

### Issue: Code block labels cut off
**Solution**:
1. Expand terminal window wider
2. Check terminal font size
3. Verify language name is reasonable length
4. Labels should never cause line wrap

### Issue: Error message hard to read
**Solution**:
1. Verify Deuteranopia theme is active
2. Check terminal background color
3. Adjust terminal contrast/brightness
4. Try on different terminal

---

## Documentation References

- **Implementation Details**: See `DEUTERANOPIA_IMPLEMENTATION.md`
- **Color Definitions**: See `crates/tui/src/theme_colors.rs`
- **Rendering Logic**: See `crates/tui/src/messages/mod.rs`
- **Theme System**: See `crates/tui/src/theme_screen.rs`

---

## Feedback & Issues

To report issues with the Deuteranopia theme:
1. Describe the problem clearly
2. Include screenshot if possible
3. Mention terminal and OS
4. Include color blind type if applicable
5. Provide steps to reproduce

---

**Last Updated**: April 2026
**Status**: Ready for Testing
**Compatibility**: All terminals with 24-bit color support
