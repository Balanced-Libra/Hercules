# ESP32 Firmware

## Purpose

The firmware runs on the ESP32 and is responsible for:

- **Servo Control**: Driving six servos (with one inverted for mechanical reasons) to move the arm.
- **Sequence Execution**: Reading and smoothly interpolating between positions from files stored in SPIFFS (the onboard file system).
- **File Management**: Allowing sequences (motion routines) to be uploaded, listed, and deleted.
- **Communication**: Accepting commands via both Wi‑Fi (using an asynchronous web server) and a serial interface.

## Key Functions and Design Choices

### Initialization and Setup

- `setup()`: Initializes the Serial interface, mounts SPIFFS, attaches servos (setting them to a neutral 90° position), connects to Wi‑Fi, and sets up HTTP endpoints.

  *Reasoning*: Ensures the system starts in a known state and is ready for both local (serial) and remote (Wi‑Fi) control.

### Web Server Endpoints

- `/execute_sequence` (HTTP POST): Starts sequence execution by spawning a FreeRTOS task, allowing non‑blocking operation.
- `/list_sequences` (HTTP GET): Lists all stored sequence files in a JSON array.

  *Reasoning*: By using asynchronous routes and tasks, the firmware can handle long‑running movements without blocking other operations.

### Serial Command Handling

In the `loop()`, the firmware reads serial commands such as:

- `GET_STORAGE_INFO`
- `BEGIN_UPLOAD`
- `LIST_SEQUENCES`
- `EXECUTE_SEQUENCE`
- `HOME`
- Direct servo commands (e.g., `2:150`)

  *Reasoning*: This dual‑mode command interface makes the system flexible for debugging, testing, and different methods of control.

### Sequence Execution and Interpolation

- `executeSequence()`: Reads a binary file of positions, then uses an easing function (`easeInOutSine()`) to smoothly interpolate servo positions.
- `returnToDefault()`: Moves the arm to its “home” (90°) position using similar interpolation.

  *Reasoning*: Smooth motion is crucial for mechanical stability and realistic arm movement. The easing function provides gentle acceleration and deceleration.

### File and Storage Management

Functions such as:

- `getSequenceList()`
- `getCurrentStorageUsage()`
- `deleteSequence()`

  *Reasoning*: Allowing users to upload and manage multiple sequences helps in creating complex routines without overloading the device’s storage.
