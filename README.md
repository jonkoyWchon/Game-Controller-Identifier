# Game Controller Identifier

## Project Overview

Game Controller Identifier is an AI-to-hardware project that uses a webcam to classify an image as either **Game Controller** or **Background**. A Python program loads a model exported from Google Teachable Machine, processes webcam frames with OpenCV, and sends a serial message to an Arduino Mega 2560. The Arduino updates a character LCD according to the message it receives.

This repository is for an individual AI-Hardware Pipelines project. The demonstration photos and development notes below should be completed with evidence from your own testing.

## Pipeline

1. The webcam captures a frame.
2. OpenCV center-crops the frame to a square, resizes it to 224 × 224 pixels, converts BGR to RGB, and normalizes the pixel values.
3. The trained Keras model predicts a class.
4. The serial-enabled Python script checks whether confidence is at least 80%.
5. The script sends `on` or `off` to the Arduino when the mapped prediction changes.
6. The Arduino reads the newline-terminated serial message and updates the LCD.

## Class Mapping

The supplied `Models/labels.txt` file defines these classes:

| Model index | Class | Serial message | Expected LCD text |
|---:|---|---|---|
| `0` | Game Controller | `on` | `Controller` |
| `1` | Background | `off` | `Background` |

The serial script uses a confidence threshold of `0.80` (80%). A prediction below that threshold does not send a message.

## Repository Contents

```text
Game_Controller_Identifier/
├── README.md
├── requirements.txt
├── .gitignore
├── LCD_Controller/
│   └── LCD_Controller.ino
└── Models/
    ├── GoogleTeachableCode-KerasOpenCV.py
    ├── GoogleTeachableCode-KerasOpenCV-Serial.py
    ├── keras_model.h5
    └── labels.txt
```

- `LCD_Controller/LCD_Controller.ino` — Arduino sketch that listens for `on` and `off` over serial and updates the LCD.
- `Models/GoogleTeachableCode-KerasOpenCV.py` — webcam classifier that prints the predicted class and confidence.
- `Models/GoogleTeachableCode-KerasOpenCV-Serial.py` — webcam classifier that also sends messages to the Arduino.
- `Models/keras_model.h5` — trained Teachable Machine model.
- `Models/labels.txt` — model class names and indexes.
- `requirements.txt` — Python dependencies.

## Hardware

- Arduino Mega 2560
- Computer with a working webcam
- 16×2 character LCD compatible with the Arduino `LiquidCrystal` library
- USB cable for the Arduino
- Jumper wires and any required LCD contrast adjustment components

### LCD wiring

The sketch uses the Arduino `LiquidCrystal` library and these pin assignments:

| LCD signal | Arduino pin |
|---|---:|
| RS | 12 |
| Enable (E) | 11 |
| D4 | 5 |
| D5 | 4 |
| D6 | 3 |
| D7 | 2 |

Connect LCD power, ground, and contrast according to the documentation for your particular LCD. Check the pinout before powering the circuit.

The Arduino sketch uses `Serial.begin(9600)`, so the Python `BAUD_RATE` must remain `9600`.

## Setup

### 1. Upload the Arduino sketch

1. Open `LCD_Controller/LCD_Controller.ino` in the Arduino IDE.
2. Connect the Arduino Mega 2560.
3. Select **Arduino Mega or Mega 2560** and the correct port from the **Tools** menu.
4. Upload the sketch.
5. The LCD should initially display `Waiting...`.

The sketch uses the built-in `LiquidCrystal` library; it does not use `LiquidCrystal_I2C`.

### 2. Install Python packages

Use a compatible Python version for the TensorFlow/Keras packages available for your operating system. It is recommended to use a virtual environment.

From the repository root:

**Windows PowerShell**
```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

TensorFlow, `tf-keras`, Python, and operating-system compatibility can vary. If package installation or model loading fails, select compatible versions and document the versions that worked on your computer.

### 3. Configure the serial port

Open `Models/GoogleTeachableCode-KerasOpenCV-Serial.py` and find:

```python
SERIAL_PORT = "COM7"
```

Change this value if the Arduino is assigned a different port. In the Arduino IDE, check **Tools → Port**. On Windows, ports commonly look like `COM3`; macOS/Linux use device paths.

Close the Arduino IDE Serial Monitor before starting the Python serial-enabled script, because it can occupy the same serial port.

### 4. Run webcam classification

From the repository root:

```bash
python Models/GoogleTeachableCode-KerasOpenCV.py
```

This runs the webcam classifier without sending commands to the Arduino.

### 5. Run the complete AI-hardware pipeline

From the repository root:

```bash
python Models/GoogleTeachableCode-KerasOpenCV-Serial.py
```

Click the webcam window and press `Esc` or `q` to quit. Keep `keras_model.h5` and `labels.txt` in the same `Models` directory as the Python scripts; the scripts load these files relative to their own location.

## Demonstration Evidence

**Replace the placeholders below with your own photos or video.** Do not use stock images as proof of your build.

### Hardware Setup

_Add a photo of your actual Arduino Mega 2560, LCD, and wiring._

### Game Controller Classification

_Add a screenshot of your webcam classifier recognizing the game controller, including the prediction and confidence output._

### LCD: Controller

_Add a photo of your actual LCD displaying `Controller` after the model recognizes the game controller._

### LCD: Background

_Add a photo of your actual LCD displaying `Background` when the background class is recognized._

### Video Demonstration

_Add a link to a short video showing the webcam prediction and the corresponding LCD update, if available._

## Development Notes and Troubleshooting

Use this section to record what happened during your own setup and testing. Include the error or unexpected behavior, the steps you tried, and the change that resolved it.

Issues worth documenting if they occurred during your development:

- **LCD output:** Record any time the LCD showed only a test message, blank characters, or blocks, and what wiring/code/contrast checks you performed.
- **Serial communication:** Record how you verified the Arduino port, matched the 9600 baud rate, and ensured the Serial Monitor was closed.
- **Python packages and model loading:** Record the Python version and package versions used, and any TensorFlow/Keras compatibility errors.
- **Camera access:** Record any webcam index or operating-system camera permission changes.
- **Classification behavior:** Record how the 80% confidence threshold affected false predictions or rapid display changes.

### My troubleshooting log

- **Problem:** _Describe the actual issue._
- **What I tried:** _List the tests or changes you made._
- **Solution:** _Describe what fixed it, or state if it remains unresolved._
- **Lesson learned:** _Explain what you learned._

## Limitations and Notes

- The model's performance depends on its training examples, lighting, camera position, and background.
- The serial script currently uses `COM7`; this must be changed on systems that assign another port.
- The class indexes in `labels.txt` must match the model's output order and `MESSAGE_FOR_LABEL` mapping.
- This repository contains the trained model file, so the model is included with the code rather than downloaded separately.

## Conclusion

This project demonstrates a pipeline from webcam image classification to physical hardware output. A Teachable Machine model performs the classification, Python/OpenCV handles image processing, serial communication passes the result to an Arduino, and the Arduino displays the corresponding text on an LCD.

## License

No license has been selected for this repository yet. Add a license if you want to specify how others may use, modify, and redistribute the project.
