20250522
Testing RFID reading using RC522
https://lastminuteengineers.com/how-rfid-works-rc522-arduino-tutorial/
Data sheet: https://www.nxp.com/docs/en/data-sheet/MFRC522.pdf

Components:
- RF RC522 reader
- RFID tag / card
- Arduino UNO

Connections:
- There are 8 pins in RC522 yet IRQ pin may be ignored:
  - SDA -> PIN 10
  - SCL -> PIN 13
  - MOSI -> PIN 11
  - MISO -> 12
  - IRQ
  - GND -> GND
  - RST -> PIN 9
  - 3.3V -> 3.3V

Library:
- MFRC522 by GithubCommunity 1.4.12
  https://github.com/miguelbalboa/rfid

May the following the sample sketches to test, e.g.,
- Dumpinfo
- fireware_check
- ReadNUID
- rfid_default_keys
- rfid_read_personal_data
etc
