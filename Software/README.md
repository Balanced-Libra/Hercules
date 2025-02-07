# Python GUI

## Purpose

The Python application provides a user‑friendly, desktop‑based interface that lets the user:

- **Manually control each servo** with sliders and increment/decrement buttons.
- **Record and manage motion sequences**: Save current positions, reorder them, and play them back.
- **Communicate with the ESP32**: Over Serial for direct control or via Wi‑Fi for remote sequence execution.
- **Export sequences**: Save a sequence as a binary file matching the ESP32’s expected format.

## Key Functions and Design Choices

### GUI Layout and Controls

- **Sliders and Buttons**: Each of the six servos is represented by a slider (ranging from 0 to 180°), along with plus/minus buttons for fine adjustments.
- **Sequence Management**: A listbox shows saved positions, and buttons (**Save, Remove, Move Up/Down**) let you organize your sequence.
- **Playback Controls**: Separate buttons and sliders allow for playing back an entire sequence or individual positions at a chosen speed.

  *Reasoning*: A clear, modular layout helps users quickly understand and interact with the robotic arm, whether making live adjustments or programming a routine.

### Serial Communication

- `initialize_serial()`, `close_serial()`, and `is_serial_connected()` manage the serial port.
- A dedicated command sender thread (using `command_sender()`) reads commands from a thread‑safe queue (`command_queue`) and sends them to the ESP32.

  *Reasoning*: Separating serial communication into its own thread avoids blocking the GUI and ensures reliable command delivery.

### Smooth Movements and Easing

- `move_servo_smoothly()`, `playback_worker()`, and `play_position_worker()` implement smooth transitions by interpolating between current and target positions using the same easing function concept as in the firmware.

  *Reasoning*: Consistency between firmware and GUI interpolation creates smooth, predictable movements, enhancing user experience.

### Thread‑Safe GUI Updates

- All GUI updates from background threads are funneled through a `gui_queue` and processed by `process_gui_queue()`.

  *Reasoning*: Tkinter isn’t thread‑safe; this design ensures all updates occur in the main thread, preventing crashes or erratic behavior.

### Wi‑Fi Control

- A separate control window (opened via `open_control_window()`) handles Wi‑Fi connection settings and displays available sequences retrieved via HTTP.
- `execute_selected_sequence_wifi()` sends an HTTP POST request to the ESP32’s `/execute_sequence` endpoint.

  *Reasoning*: Offering both Serial and Wi‑Fi control provides flexibility for remote operation and troubleshooting.

### Sequence Export and File Upload

- `export_sequence()` packs the saved sequence into a binary file using Python’s `struct` module, following the same format the ESP32 expects.

  *Reasoning*: This allows you to prepare sequences offline and then upload them to the ESP32, bridging the gap between development and field deployment.
