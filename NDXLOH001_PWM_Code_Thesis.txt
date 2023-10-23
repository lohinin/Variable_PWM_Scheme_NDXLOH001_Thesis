#include <LiquidCrystal.h>  //library for use with LCD
#include <PinChangeInterrupt.h> //Arduino Uno has pins 2 and 3 default enabled for interrupts,
//this library enables other pins for interrupt, used for pushbuttons    

//initialise LCD library, defining which LCD pin is associated with each Arduino pin
const int RS = 12, E = 11, DB4 = 5, DB5 = 4, DB6 = 3, DB7 = 2;
LiquidCrystal lcd(RS, E, DB4, DB5, DB6, DB7);

const int LED = 10; //initialise Arduino pin D10 as interface with LED array
const int uv_ain = A0;  //initialise Arduino pin A0 as interface with UV sensor
double max_intensity = 60.0; //initialise variable to store maximum UV LED array intensity level: 0<=max_intensity<=255, stepped from 0-100%
double pulse_rate = 5.0; //initialise variable to store LED array pulse rate: 0.25<pulse rate<15 Hz. Default 5 Hz.
const double step_intensity = 100.0/255; //step size between array intensity values
int delta_t = 31.192 * pow(pulse_rate, -1.001) * max_intensity / 60; //set delay according to pulse rate
long int t0 = millis(); //variable to set starting time, as a reference against which to make plots

#define PULSE_RATE_UP 9
#define PULSE_RATE_DOWN 8
#define INTENSITY_UP 7
#define INTENSITY_DOWN 6

void setup()
{
    Serial.begin(9600); //set baud rate
    pinMode(INTENSITY_DOWN, INPUT_PULLUP);  //initialise pins as input pins, with default HIGH value
    pinMode(INTENSITY_UP, INPUT_PULLUP);
    pinMode(PULSE_RATE_DOWN, INPUT_PULLUP);
    pinMode(PULSE_RATE_UP, INPUT_PULLUP);

    pinMode(LED, OUTPUT); //setup Arduino pin to LED array as output mode, to transmit pulse rate/intensity
    pinMode(uv_ain, INPUT); //setup Arduino pin to UV sensor as input mode, to receive UV emission feedback

    lcd.begin(16, 2); //initialise LCD screen interface, specifying width=16, height=2

    attachPCINT(digitalPinToPCINT(INTENSITY_DOWN), intensityDown, FALLING);
    attachPCINT(digitalPinToPCINT(INTENSITY_UP), intensityUp, FALLING);
    attachPCINT(digitalPinToPCINT(PULSE_RATE_DOWN), pulseRateDown, FALLING);
    attachPCINT(digitalPinToPCINT(PULSE_RATE_UP), pulseRateUp, FALLING);

}
void loop()
{

    //LED array brightening, dimming and pulsing
    //brighten, max value 255
    for (int dutyCycle = 0; dutyCycle <= max_intensity * 0.255; dutyCycle += 1)
    {
        analogWrite(LED, dutyCycle);
        delay(delta_t);  //determines pulse rate
        Serial.print(analogRead(uv_ain));//print sensor reading to output
        Serial.print("|");//print character to split values
        Serial.println(t0 - millis());//print current time since start
    }

    //dim
    for (int dutyCycle = max_intensity * 0.255; dutyCycle >= 0; dutyCycle -= 1)
    {
        analogWrite(LED, dutyCycle);
        delay(delta_t);  //determines pulse rate

        Serial.print(analogRead(uv_ain));//print sensor reading to output
        Serial.print("|");//print character to split values
        Serial.println(t0 - millis());//print current time since start
    }
    //Serial.println(t0 - millis());//print current time since start
}


void intensityDown()
{
    if (max_intensity > 0)
    {
        max_intensity = max_intensity - step_intensity; //increment down by 1 point on the scale from 0-100%
        //delta_t = 31.192 * pow(pulse_rate, -1.001) * max_intensity / 60; //set new delay according to new pulse rate
    }
    lcd.setCursor(0, 0);  // set LCD cursor to column 0, line 0
    lcd.print("                ");
    lcd.setCursor(0, 0);  // set LCD cursor to column 0, line 0
    char intense_down_output[17];
    char st_intensity[17];
    dtostrf(abs(max_intensity), 5, 2, st_intensity);
    sprintf(intense_down_output, "INTSTY:%s%%", st_intensity);  //INTSTY is abbreviation for Intensity
    lcd.print(intense_down_output); //print new pulse rate on LCD
}

void intensityUp()
{
    if (max_intensity < 100)
    {
        max_intensity = max_intensity + step_intensity; //increment up by 1 point on the scale from 0-100%
        //delta_t = 31.192 * pow(pulse_rate, -1.001) * max_intensity / 60; //set new delay according to new pulse rate
    }
    lcd.setCursor(0, 0);  // set LCD cursor to column 0, line 0
    lcd.print("                ");
    lcd.setCursor(0, 0);  // set LCD cursor to column 0, line 0
    char intense_up_output[17];
    char st_intensity[17];
    dtostrf(max_intensity, 5, 2, st_intensity);
    sprintf(intense_up_output, "INTSTY:%s%%", st_intensity);  //INTSTY is abbreviation for Intensity
    lcd.print(intense_up_output); //print new pulse rate on LCD
}

void pulseRateDown()
{
    if (pulse_rate > 0.25)
    {
        pulse_rate = pulse_rate - 0.25;
        //delta_t = 31.192 * pow(pulse_rate, -1.001) * max_intensity / 60; //set new delay according to new pulse rate
    }

    lcd.setCursor(0, 1);  // set LCD cursor to column 0, line 1
    lcd.print("                ");
    lcd.setCursor(0, 1);  // set LCD cursor to column 0, line 1
    char pulse_down_output[17];
    char st_pulse_rate[17];
    dtostrf(pulse_rate, 5, 2, st_pulse_rate);
    sprintf(pulse_down_output, "PLSE_RT:%sHz", st_pulse_rate);
    lcd.print(pulse_down_output); //print new pulse rate on LCD
}

void pulseRateUp()
{
    if (pulse_rate < 15)    //upper limit of pulse rate is 15 Hz
    {
        pulse_rate = pulse_rate + 0.25;
        //delta_t = 31.192*pow(pulse_rate,-1.001)*max_intensity/60; //set new delay according to new pulse rate
    }
    lcd.setCursor(0, 1);  // set LCD cursor to column 0, line 1
    lcd.print("                ");
    lcd.setCursor(0, 1);  // set LCD cursor to column 0, line 1
    char pulse_up_output[17];
    char st_pulse_rate[17];
    dtostrf(pulse_rate, 5, 2, st_pulse_rate);
    sprintf(pulse_up_output, "PLSE_RT:%sHz", st_pulse_rate);
    lcd.print(pulse_up_output); //print new pulse rate on LCD
}








