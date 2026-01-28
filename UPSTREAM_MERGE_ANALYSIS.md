# Upstream Merge Analysis

This document summarizes changes from upstream labwc that can be merged into this fork.

## Summary Statistics

- **Upstream commits ahead**: 101 commits (as of latest fetch)
- **Fork commits ahead**: 110 commits (fork-specific features)
- **Upstream tag**: 0.9.3-25-g7d7ece21
- **Fork tag**: 0.9.3-34-gbccee6d7

## Key Upstream Improvements to Consider

### 1. **Window Switcher (Cycle) Enhancements** ⭐ HIGH PRIORITY
- **Scrollable OSD**: New scrollable window switcher implementation (`742c2b53`)
- **Better filtering**: Support for filtering by workspace, output, and app_id
- **Improved structure**: Refactored cycle types into `cycle.h` (`dfe428ae`)
- **New features**:
  - `<action name="NextWindow" workspace="current|all">` support
  - `<action name="NextWindow" output="" and identifier="">` support
- **Bug fixes**:
  - Ensure `server->cycle` is cleared in `destroy_cycle()` (`97b31429`)
  - Halt window switcher on Reconfigure (`9f50971e`)
  - Added `server->cycle_preview_tree` for better preview handling

**Files affected**: `include/cycle.h`, `src/cycle/cycle.c`, `src/cycle/osd-scroll.c` (new), `src/cycle/osd-classic.c`, `src/cycle/osd-thumbnail.c`

### 2. **View Geometry Improvements** ⭐ HIGH PRIORITY
- **Better position handling**: Renamed `last_layout_geometry` to `last_placement` structure (`ee52853e`)
- **Output-relative positioning**: Keep and restore output-relative position on layout changes (`37c7de32`)
- **Improved placement**: Save last placement info before layout change (`f58b5322`)
- **Code cleanup**: Eliminated `store_natural_geometry` arguments (`f5909ac5`)
- **Documentation**: Added documentation for `adjust_floating_geometry()` (`a964d41d`)

**Files affected**: `include/view.h`, `src/view.c`

### 3. **Output Management** ⭐ MEDIUM PRIORITY
- **Error handling**: Suppress error when output position is unavailable (`7d7ece21`)
- **Virtual output improvements**: Some changes to `include/output-virtual.h` and `src/output-virtual.c`

**Files affected**: `src/output.c`, `include/output-virtual.h`, `src/output-virtual.c`

### 4. **XWayland Fixes** ⭐ MEDIUM PRIORITY
- **Focus bug fix**: Do not try to focus a window that was already in focus (`c1c156ef`)

**Files affected**: `src/xwayland.c`

### 5. **Configuration Improvements** ⭐ LOW PRIORITY
- **Workspace config**: Add config option for initial workspace selection (`64aec6ff`)
- **Parse error fix**: Prevent wrong `parse_bool()` error message (`276d4e61`)
- **RCXML sync**: Sync `rcxml.window_switcher` with XML format (`e2d83ff7`)

**Files affected**: `include/config/rcxml.h`, `src/config/rcxml.c`, `src/workspaces.c`

### 6. **Protocol Cleanup** ⭐ LOW PRIORITY
- **Removed protocol**: Remove `wlr-input-inhibitor-unstable-v1` (`9600c73e`)

**Files affected**: `protocols/wlr-input-inhibitor-unstable-v1.xml` (deleted), `protocols/meson.build`

### 7. **Documentation** ⭐ LOW PRIORITY
- **GTK4 example**: Add example for GTK4 composing (`caa9b90e`)

**Files affected**: `docs/`

### 8. **CI/Build Improvements** ⭐ LOW PRIORITY
- **Check script**: Run checkpatch.pl processes with max 16 args each (`f09a0c2b`)
- **CI updates**: Disable IRC notifications, disable no-font check

**Files affected**: `.github/workflows/`, `scripts/check`

## Potential Conflicts to Watch For

### 1. **View Geometry Changes** ⚠️ CONFLICT DETECTED
**CRITICAL**: The fork uses `last_layout_geometry` extensively (31 occurrences found):
- `src/view.c`: Multiple functions using `last_layout_geometry`:
  - `last_layout_geometry_is_valid()`
  - `update_last_layout_geometry()`
  - `apply_last_layout_geometry()`
  - `view_invalidate_last_layout_geometry()`
  - Used in tiling and layout adjustment code
- `src/interactive.c`: Also uses `last_layout_geometry`
- `include/view.h`: Field definition

**Upstream change**: Replaced `struct wlr_box last_layout_geometry` with:
```c
struct {
    char *output_name;
    struct wlr_box relative_geo;  // output-relative coordinates
    struct wlr_box layout_geo;     // layout coordinates
} last_placement;
```

**Impact**: This is a **breaking change** that will require:
1. Refactoring all `last_layout_geometry` usage to `last_placement.layout_geo`
2. Potentially adapting fork's tiling logic to use the new structure
3. Testing that tiling mode and smart resize still work correctly

**Recommendation**: This should be done in Phase 3 with extensive testing.

### 2. **Cycle/Window Switcher** ⚠️ CONFLICT DETECTED
**CRITICAL**: The fork uses `cycle_osd_output_criteria` which upstream renamed to `cycle_output_filter`:
- `include/config/types.h`: `enum cycle_osd_output_criteria` exists
- `include/config/rcxml.h`: `output_criteria` field uses old enum

**Upstream changes**:
- Renamed `cycle_osd_output_criteria` → `cycle_output_filter` (`64af2061`)
- Added `cycle_filter` structure with workspace, output, and app_id filters
- Changed `cycle_begin()` signature to accept `struct cycle_filter`
- Refactored OSD creation to use `cycle_osd_output` structure
- Added `get_outputs_by_filter()` function
- Changed `rc.window_switcher.style` → `rc.window_switcher.osd.style`

**Files affected in fork**:
- `include/config/types.h` - enum rename needed
- `include/config/rcxml.h` - field rename needed
- `src/cycle/cycle.c` - function signature changes
- `src/action.c` - calls to `cycle_begin()` need updating

**Impact**: Medium - requires updating enum names and function calls, but logic should be compatible.

### 3. **Output Virtual**
The fork has virtual output commands (`--virtual-output-add`, `--virtual-output-remove`). Upstream has some changes to virtual output handling that should be compatible but need verification.

### 4. **Workspace Features**
The fork has workspace switching commands. Upstream added initial workspace selection config option - should be compatible.

## Recommended Merge Strategy

### Phase 1: Low-Risk Changes (Easy Wins)
1. XWayland focus fix (`c1c156ef`)
2. Output error suppression (`7d7ece21`)
3. Configuration parse error fix (`276d4e61`)
4. Protocol cleanup (`9600c73e`)
5. CI/build improvements

### Phase 2: Medium-Risk Changes (Requires Testing)
1. View geometry improvements (check for conflicts with tiling mode)
2. Workspace config option (should be compatible with fork's workspace commands)
3. Documentation updates

### Phase 3: High-Risk Changes (Requires Careful Integration)
1. Window switcher/cycle enhancements (major refactoring)
   - Test with fork's conditional keybinds
   - Verify compatibility with fork's custom features
2. Complete view geometry refactoring
   - Ensure tiling mode still works correctly
   - Test smart resize functionality

## Files to Review Before Merging

1. `src/view.c` - Check for conflicts with `last_placement` changes
2. `src/cycle/cycle.c` - Major refactoring, needs careful review
3. `include/cycle.h` - New structure definitions
4. `src/config/rcxml.c` - Configuration parsing changes
5. `src/workspaces.c` - Workspace selection changes

## Testing Checklist After Merge

- [ ] Window switcher works with fork's conditional keybinds
- [ ] Tiling mode still functions correctly
- [ ] Smart resize tiling works with new view geometry
- [ ] Virtual output commands still work
- [ ] Workspace switching commands are compatible
- [ ] View positioning/restoration works correctly
- [ ] XWayland windows focus correctly
- [ ] No regressions in existing fork features

## Commands to Help with Merging

```bash
# View specific upstream commit
git show upstream/master:<commit-hash>

# Create a test branch for merging
git checkout -b merge-upstream-changes

# Try merging specific commits (cherry-pick)
git cherry-pick <commit-hash>

# View conflicts in detail
git diff master upstream/master -- <file>

# Test merge without committing
git merge --no-commit --no-ff upstream/master
```

## Notes

- The fork has significant custom features (tiling mode, virtual outputs, conditional keybinds) that should be preserved
- Upstream changes are mostly improvements and bug fixes that should integrate well
- The cycle/window switcher refactoring is the biggest change and needs the most attention
- Most other changes are isolated improvements that should merge cleanly
