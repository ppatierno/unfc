# unfc

![](images/uNFC.jpg)

uNFC - NFC library for .Net platforms

uNFC library allow to use NFC integrated circuit connected to your PC (via serial) or to embedded system based on Windows Embedded Compact or .Net Micro Framework.

It supports all three types of .Net Framework :

* .Net Framework 4.0 for PC based on Windows 7 / 8 or embedded systems based on Windows Embedded Standard 7 and Windows Embedded 8 Standard;
* .Net Compact Framework 3.5 / 3.9 for embedded systems based on Windows Embedded Compact 7 and Windows Embedded Compact 2013;
* .Net Micro Framework 4.2 and 4.3 for embedded systems like Netduino boards and .Net Gadgeteed boards;

The library supports NXP PN532 chip but it also defines a little framework so that you can develop a managed driver for a new chip and change it without modifying the upper layers and interface to the user application. The support for PN532 chip provides all three possible communication channels for it : I2C, SPI and HSU. Tipically the HSU (High Speed UART) channel is used for connecting to a serial port on PC or Windows Embedded Compact based system. I2C and SPI connections are better for .Net Micro Framework based boards.

The development and testing was made using RFID/NFC Breakout board from Elecfreaks.
More information [here](http://www.elecfreaks.com/store/rfid-nfc-c-82.html)

The reference for PN532 managed driver is the official NXP user manual available [here](http://www.nxp.com/documents/user_manual/141520.pdf)

Following a simple application example for Netduino Plus board.

You have to choose the communication layer to use and create one of the classes that implements IPN532CommunicationLayer interface.

After that you have to pass this instance to NfcPN532Reader class constructor and you can register to TagDetected and TagLost events raised when the chip detects and losts a tag.
Last operation is to call Open() method on NfcPN532Reader class instance with tag type to detect.

In the TagDetected event handler you can access the tag, using one of the classes that are based on NfcTagConnection abstract class (eg. if you detected Mifare Classic 1K tag, you have to use NfcMifareTagConnection class). You can get the connection to the tag from NfcTagEventArgs inside your event handler. On the connection class you have Write and Read method and some other methods specific based for the tag detected.

```
public class Program
{
        static private INfcReader nfc;

        public static void Main()
        {
            // write your code here

            
            // HSU Communication layer
            // MOSI/SDA (TX) --> DIGITAL 0 (RX COM1)
            // SCL/RX (RX) --> DIGITAL 1 (TX COM1)
            IPN532CommunicationLayer commLayer = new PN532CommunicationHSU(SerialPorts.COM1);

            // SPI Communication layer
            // SCK --> DIGITAL 13
            // MISO --> DIGITAL 12
            // MOSI/SDA --> DIGITAL 11
            // SCL/RX -> DIGITAL 10
            // IRQ --> DIGITAL 8
            //IPN532CommunicationLayer commLayer = new PN532CommunicationSPI(SPI.SPI_module.SPI1, Pins.GPIO_PIN_D10, Pins.GPIO_PIN_D8);

            // I2C Communication layer
            // MOSI/SDA --> ANALOG 4 (SDA)
            // SCL/RS --> ANALOG 5 (SCL)
            // IRQ --> DIGITAL 8
            //IPN532CommunicationLayer commLayer = new PN532CommunicationI2C(Pins.GPIO_PIN_D8);
                        
            nfc = new NfcPN532Reader(commLayer);
            nfc.TagDetected += nfc_TagDetected;
            nfc.TagLost += nfc_TagLost;
            nfc.Open(NfcTagType.MifareUltralight);

            InterruptPort button = new InterruptPort(Pins.ONBOARD_SW1, true, Port.ResistorMode.Disabled, Port.InterruptMode.InterruptEdgeHigh);
            button.OnInterrupt += button_OnInterrupt;
            
            Thread.Sleep(Timeout.Infinite);
        }

        static void button_OnInterrupt(uint data1, uint data2, DateTime time)
        {
            nfc.Close();
        }

        static void nfc_TagLost(object sender, NfcTagEventArgs e)
        {
            Debug.Print("LOST " + HexToString(e.Connection.ID));
        }

        static void nfc_TagDetected(object sender, NfcTagEventArgs e)
        {
            Debug.Print("DETECTED " + HexToString(e.Connection.ID));

            byte[] data;

            switch (e.NfcTagType)
            {
                case NfcTagType.MifareClassic1k:
                    
                    NfcMifareTagConnection mifareConn = (NfcMifareTagConnection)e.Connection;
                    mifareConn.Authenticate(MifareKeyAuth.KeyA, 0x08, new byte[] { 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF });
                    mifareConn.Read(0x08);

                    data = new byte[16];
                    for (byte i = 0; i < data.Length; i++)
                        data[i] = i;

                    mifareConn.Write(0x08, data);

                    mifareConn.Read(0x08);
                    break;

                case NfcTagType.MifareUltralight:

                    NfcMifareUlTagConnection mifareUlConn = (NfcMifareUlTagConnection)e.Connection;

                    for (byte i = 0; i < 16; i++)
                    {
                        byte[] read = mifareUlConn.Read(i);
                    }

                    mifareUlConn.Read(0x08);

                    data = new byte[4];
                    for (byte i = 0; i < data.Length; i++)
                        data[i] = i;

                    mifareUlConn.Write(0x08, data);

                    mifareUlConn.Read(0x08);
                    break;

                default:
                    break;
            }
        }

        static private string hexChars = "0123456789ABCDEF";

        /// <summary>
        /// Convert hex byte array in a hex string
        /// </summary>
        /// <param name="value">Byte array with hex values</param>
        /// <returns>Hex string</returns>
        static internal string HexToString(byte[] value)
        {
            StringBuilder hexString = new StringBuilder();
            for (int i = 0; i < value.Length; i++)
            {
                hexString.Append(hexChars[(value[i] >> 4) & 0x0F]);
                hexString.Append(hexChars[value[i] & 0x0F]);
            }
            return hexString.ToString();
        }
}
```
