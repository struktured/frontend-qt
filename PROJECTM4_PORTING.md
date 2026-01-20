# projectM4 Porting Status

This document tracks the work needed to port the Qt frontend to projectM4.

## Completed
- ✅ Updated CMakeLists.txt to find `projectM4` instead of `libprojectM`
- ✅ Updated target linking from `libprojectM::shared/static` to `libprojectM::projectM`
- ✅ Fixed render call: `projectm_render_frame()` → `projectm_opengl_render_frame()`
- ✅ Created symlink for header compatibility: `/usr/local/include/libprojectM` → `/usr/local/include/projectM-4`

## API Changes Needed

The projectM4 C API has several breaking changes that need to be addressed:

### 1. Callback Signatures Changed
**Old (projectM3):**
```c
void callback(bool hardCut, unsigned int index, void* context)
```

**New (projectM4):**
```c
// Preset switch requested - no index parameter
void callback(bool is_hard_cut, void* user_data)

// Preset switch failed - different parameters
void callback(const char* preset_filename, const char* message, void* user_data)
```

**Files affected:**
- `src/common/qprojectm.hpp` - callback registrations and implementations

### 2. Removed APIs
- `projectm_key_handler()` - removed entirely
- `projectm_set_preset_switched_event_callback()` → `projectm_set_preset_switch_requested_event_callback()`
- `projectm_set_preset_rating_changed_event_callback()` - removed
- `projectm_is_preset_locked()` - removed
- `projectm_lock_preset()` → `projectm_set_preset_locked()`
- `projectm_set_shuffle_enabled()` - removed

### 3. Enum Changes
- `projectm_flags::PROJECTM_FLAG_DISABLE_PLAYLIST_LOAD` - need to verify new flag names
- `projectm_preset_rating_type` - removed or renamed

### 4. Files Needing Updates
1. `src/common/qprojectm.hpp` - Constructor and callback setup
2. `src/common/qprojectmwidget.hpp` - Keyboard shortcuts, preset locking, shuffle
3. `src/common/qprojectm_mainwindow.hpp` - If it uses ratings

## PipeWire Port (TODO)

To create the PipeWire frontend:

1. Create `cmake/FindPipeWire.cmake` module
2. Copy `src/ui-pulseaudio/` to `src/ui-pipewire/`
3. Update PipeWire-specific code:
   - Replace PulseAudio API calls with PipeWire equivalents
   - Update thread handling for PipeWire's callback model
4. Add `ENABLE_PIPEWIRE` option to main CMakeLists.txt
5. Add `src/ui-pipewire` subdirectory when enabled

## Build Instructions

```bash
# Install dependencies
apt-get install qtbase5-dev libqt5opengl5-dev libpipewire-0.3-dev

# Build and install projectM4
git clone --recursive https://github.com/projectM-visualizer/projectm.git
cd projectm && mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
make -j$(nproc) && sudo make install
sudo ldconfig

# Create header symlink (temporary fix)
sudo ln -s /usr/local/include/projectM-4 /usr/local/include/libprojectM

# Build frontend (will fail until API fixes are complete)
mkdir build && cd build
cmake .. -DCMAKE_PREFIX_PATH=/usr/local
make
```

## References
- projectM4 API docs: `/usr/local/include/projectM-4/`
- Callback changes: `/usr/local/include/projectM-4/callbacks.h`
- Parameter functions: `/usr/local/include/projectM-4/parameters.h`
