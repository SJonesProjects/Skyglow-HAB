# Skyglow-HAB
//This code was implemented into an REU project measuring light pollution using lux sensors on an Arduino.
/* TSL2591 Digital Light Sensor */ /* Dynamic Range: 600M:1 */
/* Maximum Lux: 88K */

#include <Adafruit_Sensor.h> #include “Adafruit_TSL2591.h” #include “Wire.h”
extern “C” {

#include “utility/twi.h” // from Wire library, so we can do bus scanning }

#define TCAADDR 0x70

void tcaselect(uint8_t i) { if (i > 7) return;

Wire.beginTransmission(TCAADDR); Wire.write(1 << i); Wire.endTransmission();

}

// Example for demonstrating the TSL2591 library – public domain!

// connect SCL to analog 5
// connect SDA to analog 4
// connect Vin to 3.3-5V DC
// connect GROUND to common ground

Adafruit_TSL2591 tsl4 = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)
Adafruit_TSL2591 tsl5 = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)

Adafruit_TSL2591 tsl6 = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)
Adafruit_TSL2591 tsl7 = Adafruit_TSL2591(2591); // pass in a number for the sensor identifier (for your use later)

/**************************************************************************/

/*
Displays some basic information on this sensor from the unified

sensor API sensor_t type (see Adafruit_Sensor for more information) */

/**************************************************************************/ void displaySensorDetails(Adafruit_TSL2591 tsl)

{
sensor_t sensor;
tsl.getSensor(&sensor); Serial.println(“————————————“);

Serial.print (“Sensor: Serial.print (“Driver Ver: Serial.print (“Unique ID: Serial.print (“Max Value:

Serial.print (“Min Value:

“); Serial.println(sensor.name); “); Serial.println(sensor.version);

“); Serial.println(sensor.sensor_id);
“); Serial.print(sensor.max_value); Serial.println(” lux”);

“); Serial.print(sensor.min_value); Serial.println(” lux”);

Serial.print (“Min Value: “); Serial.print(sensor.min_value); Serial.println(” lux”); Serial.print (“Resolution: “); Serial.print(sensor.resolution); Serial.println(” lux”); Serial.println(“————————————“);
Serial.println(“”);

delay(500); }

/**************************************************************************/

/*
Configures the gain and integration time for the TSL2591

*/ /**************************************************************************/ void configureSensor(Adafruit_TSL2591 tsl)
{

// You can change the gain on the fly, to adapt to brighter/dimmer light situations

//tsl.setGain(TSL2591_GAIN_LOW); tsl.setGain(TSL2591_GAIN_MED);
// tsl.setGain(TSL2591_GAIN_HIGH);

// 1x gain (bright light) // 25x gain

// 428x gain

// Changing the integration time gives you a longer time over which to sense light
// longer timelines are slower, but are good in very low light situtations! tsl.setTiming(TSL2591_INTEGRATIONTIME_100MS); // shortest integration time (bright light) // tsl.setTiming(TSL2591_INTEGRATIONTIME_200MS);

// tsl.setTiming(TSL2591_INTEGRATIONTIME_300MS);
// tsl.setTiming(TSL2591_INTEGRATIONTIME_400MS);
// tsl.setTiming(TSL2591_INTEGRATIONTIME_500MS);
// tsl.setTiming(TSL2591_INTEGRATIONTIME_600MS); // longest integration time (dim light)

/* Display the gain and integration time for reference sake */ Serial.println(“————————————“);
Serial.print (“Gain: “);
tsl2591Gain_t gain = tsl.getGain();

switch(gain)

{
case TSL2591_GAIN_LOW:

Serial.println(“1x (Low)”);

break;
case TSL2591_GAIN_MED:

Serial.println(“25x (Medium)”);

break;
case TSL2591_GAIN_HIGH:

Serial.println(“428x (High)”);

break;
case TSL2591_GAIN_MAX:

Serial.println(“9876x (Max)”);

break; }

Serial.print (“Timing: “); Serial.print((tsl.getTiming() + 1) * 100, DEC); Serial.println(” ms”); Serial.println(“————————————“); Serial.println(“”);

}

/**************************************************************************/

/**************************************************************************/

/*
Used for beginning multiple sensors through multiplexer

*/ /**************************************************************************/ void tslStart(int t, Adafruit_TSL2591 tsl)
{

tcaselect(t);

if (tsl.begin())

{
Serial.println(“Found TSL5 sensor.”);

} else

{
Serial.println(“TSL5 not found … check your wiring?”);

while (1); }

/* Display some basic information on this sensor */ displaySensorDetails(tsl);

/* Configure the sensor */ configureSensor(tsl);

}

/**************************************************************************/

/*
Program entry point for the Arduino sketch

*/ /**************************************************************************/ void setup(void)
{

Serial.begin(115200); while (!Serial);

Wire.begin();

Serial.println(“Starting Adafruit TSL2591 Test!”);

tslStart(4,tsl4); tslStart(5,tsl5); tslStart(6,tsl6); tslStart(7,tsl7);

// Now we’re ready to get readings … move on to loop()! }

/**************************************************************************/

/*
Shows how to perform a basic read on visible, full spectrum or

infrared light (returns raw 16-bit ADC values) */

/**************************************************************************/ void simpleRead(Adafruit_TSL2591 tsl)

{

{
// Simple data read example. Just read the infrared, fullspecrtrum diode
// or ‘visible’ (difference between the two) channels.
// This can take 100-600 milliseconds! Uncomment whichever of the following you want to read uint16_t x = tsl.getLuminosity(TSL2591_VISIBLE);
//uint16_t x = tsl.getLuminosity(TSL2591_FULLSPECTRUM);
//uint16_t x = tsl.getLuminosity(TSL2591_INFRARED);

Serial.print(“[ “); Serial.print(millis()); Serial.print(” ms ] “); Serial.print(“Luminosity: “);
Serial.println(x, DEC);

}

/**************************************************************************/

/*
Show how to read IR and Full Spectrum at once and convert to lux

*/ /**************************************************************************/ void advancedRead(Adafruit_TSL2591 tsl)
{

// More advanced data read example. Read 32 bits with top 16 bits IR, bottom 16 bits full spectrum // That way you can do whatever math and comparisons you want!
uint32_t lum = tsl.getFullLuminosity();
uint16_t ir, full;

ir = lum >> 16;
full = lum & 0xFFFF;
Serial.print(“[ “); Serial.print(millis()); Serial.print(” ms ] “); Serial.print(“IR: “); Serial.print(ir); Serial.print(” “); Serial.print(“Full: “); Serial.print(full); Serial.print(” “); Serial.print(“Visible: “); Serial.print(full – ir); Serial.print(” “); Serial.print(“Lux: “); Serial.println(tsl.calculateLux(full, ir));

}

/**************************************************************************/

/*
Performs a read using the Adafruit Unified Sensor API.

*/ /**************************************************************************/ void unifiedSensorAPIRead(Adafruit_TSL2591 tsl)
{

/* Get a new sensor event */ sensors_event_t event; tsl.getEvent(&event);

/* Display the results (light is measured in lux) */
Serial.print(“[ “); Serial.print(event.timestamp); Serial.print(” ms ] “); if ((event.light == 0) |

(event.light > 4294966000.0) |

(event.light <-4294966000.0)) {

/* If event.light = 0 lux the sensor is probably saturated */
/* and no reliable data could be generated! */
/* if event.light is +/- 4294967040 there was a float over/underflow */ Serial.println(“Invalid data (adjust gain or timing)”);

} else

{

{
Serial.print(event.light); Serial.println(” lux”);

} }

/**************************************************************************/

/*
Arduino loop function, called once ‘setup’ is complete (your own code

should go here) */

/**************************************************************************/ void loop(void)

{
//simpleRead(); tcaselect(4); advancedRead(tsl4);
// unifiedSensorAPIRead();

tcaselect(5); advancedRead(tsl5); delay(500);

tcaselect(6); advancedRead(tsl6); delay(500);

tcaselect(7); advancedRead(tsl7); delay(500); Serial.println();

}
