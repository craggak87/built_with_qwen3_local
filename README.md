### ✅ Prerequisites

- **Python 3.x** installed on the Raspberry Pi 4.
- **PyQt5** and **pymodbus** Python libraries:
  ```bash
  pip3 install PyQt5 pymodbus
  ```

---

### 📁 Project Structure

- `hmi_app.py` (main application file)

---

### 🧠 Key Concepts

- **ModbusTCP** is used for communication.
- **Coils (Discrete Outputs)** are used for toggling outputs.
- **Input Registers** are used for reading input status.
- **PyQt5** is used for the GUI.
- **pymodbus** is used for Modbus communication.

---

### 🧱 Code: `hmi_app.py`

```python
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QPushButton, QLabel, QVBoxLayout, QTimer
from pymodbus.client import ModbusTcpClient

class HMIApp(QWidget):
    def __init__(self):
        super().__init__()
        self.initUI()
        self.client = ModbusTcpClient('192.168.1.10', port=502)  # Replace with your PLC's IP
        self.timer = QTimer()
        self.timer.timeout.connect(self.update_status)
        self.timer.start(1000)  # Poll every second

    def initUI(self):
        self.setWindowTitle('Mitsubishi FX5 HMI')
        self.layout = QVBoxLayout()

        self.connect_button = QPushButton('Connect to PLC')
        self.connect_button.clicked.connect(self.connect_to_plc)
        self.layout.addWidget(self.connect_button)

        self.toggle_button = QPushButton('Toggle Output (Y0)')
        self.toggle_button.clicked.connect(self.toggle_output)
        self.layout.addWidget(self.toggle_button)

        self.status_label = QLabel('Status: Not Connected')
        self.layout.addWidget(self.status_label)

        self.setLayout(self.layout)

    def connect_to_plc(self):
        if self.client.connect():
            self.status_label.setText('Connected to PLC')
        else:
            self.status_label.setText('Failed to connect to PLC')

    def toggle_output(self):
        if self.client.is_connected():
            # Example: Write to coil address 0 (Mitsubishi Y0)
            self.client.write_coil(0, True)
            self.status_label.setText('Output Toggled')
        else:
            self.status_label.setText('Not connected to PLC')

    def update_status(self):
        if self.client.is_connected():
            response = self.client.read_input_registers(0, 1)
            if response.isError():
                self.status_label.setText('Error reading input register')
            else:
                self.status_label.setText(f'Input Status: {response.registers[0]}')
        else:
            self.status_label.setText('Not connected to PLC')

    def closeEvent(self, event):
        self.client.close()
        event.accept()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = HMIApp()
    ex.show()
    sys.exit(app.exec_())
```

---

### 📌 Notes for Configuration

- **PLC IP Address**: Replace `'192.168.1.10'` with the IP of your **Mitsubishi FX5 PLC**.
- **Modbus Port**: The default is `502`, but verify your PLC's configuration.
- **Register Addresses**:
  - Coils (outputs): Use `write_coil()` for discrete outputs (e.g., `0` for Y0).
  - Input Registers: Use `read_input_registers()` for read-only inputs (e.g., `0` for X0).
- **Addressing**: Modbus uses 0-based addressing, but this may vary by PLC. Confirm your PLC's Modbus mapping.

---

### 🧪 Example Use Case

1. **Run the application**:
   ```bash
   python3 hmi_app.py
   ```

2. **Click "Connect to PLC"** to establish communication.
3. **Click "Toggle Output (Y0)"** to send a signal to the PLC.
4. **Monitor Input Status** in the label.

---

### 🧩 Extension Ideas

- Add more **buttons** for other outputs.
- Add **labels** for multiple input registers.
- Include a **status indicator** (e.g., LED) for visual feedback.
- Add **logging** or **error handling** for robustness.

---

### 📚 Documentation

- [PyQt5 Documentation](https://doc.qt.io/qtforpython/)
- [pymodbus Documentation](https://pymodbus.readthedocs.io/en/latest/)

---

This basic HMI application provides a foundation for interacting with a Mitsubishi FX5 PLC using ModbusTCP and PyQt5. It can be
expanded to include more complex functionality as needed.
