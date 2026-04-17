# TheTrashies MIDI Animatronic

This project is a simple framework used to create audio-animatronics with minimal configuration, enabling a DAW to control the movements via MIDI which is used to synchronise the movement between multiple animatronics and music.

Written in [CircuitPython](https://circuitpython.org/) and with the libraries [adafruit-circuitpython-midi](https://docs.circuitpython.org/projects/midi/en/latest/index.html) and [asyncio](https://docs.circuitpython.org/projects/asyncio/en/latest/index.html), we use the [Raspberry Pi Pico](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html) microcontroller, other boards including the Pico 2 should also work but are currently untested.

At this moment the Servos are connected directly to the GPIO pins, a later version will allow the user of servo drivers such as the PCA9685 allowing a larger amount of motors and potentially other boards to be utilised.

## Installation:

Download the [Latest release of CircuitPython](https://circuitpython.org/board/raspberry_pi_pico/) for you board, install it on your microcontroller.

Reconnect the board to your computer and a CIRCUITPYTHON drive should appear, copy all the files from this repository to it.

## Naming and Concepts

### GPIO Pins:

Assuming you're using a Pico or Pico 2 microcontroller, first locate the GPIO pin in [this diagram.](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html#layout_non-wireless)
For example, the pin "1" of the pico will become "board.GP0" in the configuration.

### MIDI Notes:

We use the MIDI note number to correlate to a note, for example, C4 becomes 60 in the configuration.
Use a [MIDI chart](https://audiodev.blog/midi-note-chart/) to figure out which note corresponds to which number.

### Servo positions:

The servo_calibration_tool.py file should help you figure you the servo positions, 5000 being roughly the middle of the servo, experiment with it to find the desired positions.

### Servo and Movement:

A Servo is representation of a motor, contains information such as the GPIO pin, rest position for when the animatronic is powered on and optional smoothing parameters.

A Movement moves the Servos together when a MIDI note is received, meaning that pressing and releasing a button will move those Servos to the given positions.

The Servo controls how that particular motor will move, the Movement will tell the servo when to move and where.

A Movement can contain one or more Servos, and they are not exclusive, we'll dive deeper in the examples.

## Configuration:

The file boot.py configures the MIDI device name, for those changes to be applied the board must be reconnected to the computer.

The file configuration.py declares the Servos and Movements for the animatronic, it is necessary to put the Servos and Movements in a array:

```python

neck_servo = Servo(...)
mouth_servo = Servo(...)

servos = [neck_servo, mouth_servo]
movements = [Movement(...), Movement(...)]

```

#### Example #1:

If your character has a mouth, declare a Servo named "mouth_servo" and a Movement named "mouth_open" 
The "mouth_open" movement will map the "mouth_servo" and given the positions, pressing the button will move the servo to the final_position, releasing will move it to the initial_position.

This is how the configuration file will look like:
```python

mouth_servo = Servo(
    name='mouth_servo',
    gpio_pin=board.GP0,
    rest_position=5400,
    wait_interval_ms=1, # Optional, for smoothing.
    movement_increment_amount=10, # Optional, for smoothing.
    )

servos = [
        mouth_servo
    ]

movements = [

    Movement(
        name="open_mouth",
        midi_note=77,
        servos=[
            {
                "servo_name": mouth_servo,
                "initial_position": 5400,
                "final_position": 6400
            }
        ]
    ), 
]
```

#### Example #2:

If you character has a neck, declare a Servo named "neck_servo", and two Movements, "neck_right" and "neck_left".
Despite having only one motor, two completely different movements are possible, and each one of them has a different button.

This is how the configuration file will look like:
```python

neck_servo = Servo(
    name='neck_servo',
    gpio_pin=board.GP0,
    rest_position=5400,
    wait_interval_ms=1, # Optional, for smoothing.
    movement_increment_amount=10, # Optional, for smoothing.
    )

servos = [
        neck_servo
    ]

movements = [
    Movement(
        name="neck_left",
        midi_note=77,
        servos=[
            {
                "servo_name": neck_servo,
                "initial_position": 5400,
                "final_position": 3200
            }
        ]
    ),
    Movement(
        name="neck_right",
        midi_note=78,
        servos=[
            {
                "servo_name": neck_servo,
                "initial_position": 5400,
                "final_position": 6400
            }
        ]
    ),
]

```

### Example #3:

If you character has a hip, you'll declare two Servos "hip_left" and "hip_right" and that allows you four Movements: "lean_forward", "lean_backwards", "lean_left" and "lean_right".
Each one of those movements will move both servos in different ways to achieve the desired movement, this is why they are separate entities.