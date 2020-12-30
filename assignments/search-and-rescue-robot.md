---
layout: default
permalink: /assignment/search-and-rescue-robot/
---

# Search And Rescue Robot
## Assignment criteria

*   Identify and critically assess the elements needed within a physical computing system 
*   Interface a programmable controller with peripheral devices such as sensors, switches, key pads, motors, lights, sound, displays and other input devices and actuators.
*   Determine what types of devices are appropriate for various products and processes.
*   Design and implement 'control' algorithms for the relevant hardware platforms

{% highlight clojure %}
#include <ZumoMotors.h>
#include <QTRSensors.h>
#include <ZumoReflectanceSensorArray.h>
#include "NewPing.h"
#include "Adafruit_Sensor.h"
#include "Adafruit_TSL2561_U.h"
#include "pgmspace.h"
#include "utility/twi.h"
#include "Wire.h"

#define QTR_THRESHOLD 500 // microseconds

#define CRAWL_COUNT 2

#define CALIBRATION 50
#define NUM_SENSORS 6
ZumoReflectanceSensorArray reflectanceSensors(QTR_NO_EMITTER_PIN);
unsigned int sensor_values[NUM_SENSORS];

ZumoMotors motors;
int left = 95;
int right = 107;

//Pin 12 is the Trigger and pin 13 is the Echo
NewPing sonar(12, 13);


Adafruit_TSL2561_Unified tsl = Adafruit_TSL2561_Unified(TSL2561_ADDR_FLOAT, 12345);

float lastLightValue = 0.0f;
float currentLightValue = 0.0f;

boolean leftFirst = true;


boolean leftErrorCheck = false;
boolean rightErrorCheck = false;

int ELCount = 0;
int ERCount = 0;

enum state{Corridor, LeftError, RightError, LeftRoom,RightRoom, EmptyRoom, OccupiedRoom, Fire, LeftWall, RightWall, Other};
state currentState = Corridor;

void setup()
{
	Serial.begin(9600);
	Serial.println("Starting");
	if(!tsl.begin())
	{
		Serial.print("Ooops, no TSL2561 detected ... Check your wiring or I2C ADDR!");
		while(1);
	}
	displaySensorDetails();
	configureSensor();
}
void loop()
{
	while(true){
		//the zumo then outputs the light sensor readings	
		Serial.print("previous light value: ");
		Serial.println(lastLightValue);
		currentLightValue = getLightValue();
		Serial.print("current light value : ");
		Serial.println(currentLightValue);
		lastLightValue = currentLightValue;
		//if the currentLightValue is above 3000 then the currentState is set to Fire.
		if(currentLightValue>3000.0f){
			currentState = Fire;
		}
		delay(1000);
		switch(currentState){
			case Corridor:
			{
				if((ERCount >= 2) || (ELCount >= 2))
				{
					if(ERCount >= 2)
					{
						Serial.println("Right Shuffle");
						shuffleRight();
					}
					if(ELCount >= 2)
					{
						Serial.println("Left Shuffle");
						shuffleLeft();
					}
				}
				else
				{
					if(leftFirst){
						//The sensors are read an
						reflectanceSensors.read(sensor_values);
						Serial.print("Left : ");
						Serial.println(sensor_values[0]);
						Serial.print("Right: ");
						Serial.println(sensor_values[5]);
						if((sensor_values[0] > QTR_THRESHOLD)||(sensor_values[5] > QTR_THRESHOLD))
						{
							Serial.println("On The Edge");
							if(sensor_values[0] > QTR_THRESHOLD)
							{
								shuffleRight();
							}
							else if(sensor_values[0] > QTR_THRESHOLD)
							{
								shuffleLeft();
							}
						}
						//This checks if a wall is present on the left.
						if(checkLeft())
						{
							//if a wall is present on the left then the user is informed.
							Serial.println("Left"); 
							delay(1000);
							//The zumo then checks for a wall on the right
							if(checkRight())
							{
								//If a wall is detected on the right then the user is informed and the zumo moves forward
								Serial.println("Right");
								delay(1000);
								for(int i = 0 ;i < CRAWL_COUNT;i++){
									crawl();
									delay(1000);
								}
							}
							else
							{
								//if no wall is found on the first check to the right then the zumo performs an extended check
								if(extendedCheckRight())
								{
									//if a wall is found on the extended check the the user is informed
									Serial.println("Extended Right");
									//the extended right check counter is incremented
									ERCount++;
									delay(1000);
									//the zumo then move forward
									for(int i = 0 ;i < CRAWL_COUNT;i++){
										crawl();
										delay(1000);
									}
								}
								else
								{
									//if no wall is found on the extended right check then the current state is set to RightError
									currentState = RightError;
									//leftFirst is set to false because an error has been found on the right
									leftFirst = false;
								}
							}
						}
						else
						{
							//if no wall is detected on the ledt then an extended left check is done.
							if(extendedCheckLeft())
							{
								//if a wall is founf then the user is informed.
								Serial.println("Extended Left");
								//the extended left counter is incremented
								ELCount++;
								delay(1000);
								//the zumo then checks to the right
								if(checkRight())
								{
									//if a wall is found on the right then the user is informed
									Serial.println("Right");
									delay(1000);
									//the zumo then moves forward
									for(int i = 0 ;i < CRAWL_COUNT;i++){
										crawl();
										delay(1000);
									}
								}
								else
								{
									//if no wall is detected thrn the zumo does an extended right check
									if(extendedCheckRight())
									{
										//the user is informed that a wall is founf
										Serial.println("Extended Right");
										//the extended right check count is incremented
										ERCount++;
										delay(1000);
										//the zumo moves forward
										for(int i = 0 ;i < CRAWL_COUNT;i++){
											crawl();
											delay(1000);
										}
									}
									else
									{
										//if no wall is detected on the right then the currentState is set to RightError
										currentState = RightError;	
										//leftFirst is set to false so that the zumo checks the right first nect time.
										leftFirst = false;
									}
								}
							}
							else
							{
								//if no wall is detected on the left after both checks then the currentState is set to LeftError
								currentState = LeftError;	
							}

						}

					}
					else
					{
						//the sensor values are read and outputted to help debug the zumo
						reflectanceSensors.read(sensor_values);
						Serial.print("Left : ");
						Serial.println(sensor_values[0]);
						Serial.print("Right: ");
						Serial.println(sensor_values[5]);
						//if either of the end sensors on the reflectance array are above the threshold then the zumo is on the edge
						if((sensor_values[0] > QTR_THRESHOLD)||(sensor_values[5] > QTR_THRESHOLD))
						{
							//the user is informed of the situation
							Serial.println("On The Edge");
							//the zumo then moves over in order to compensate
							if(sensor_values[0] > QTR_THRESHOLD)
							{
								shuffleRight();
							}
							else if(sensor_values[0] > QTR_THRESHOLD)
							{
								shuffleLeft();
							}
						}
						//the zumo checks for a wall on the rightError
						if(checkRight())
						{
							//the user is informed that a wall has been founf
							Serial.println("Right"); 
							delay(1000);
							//the zumo checks for a wall on the left
							if(checkLeft())
							{
								//if a wall has been found the user is informed
								Serial.println("Left");
								delay(1000);
								//the zumo moves forward
								for(int i = 0 ;i < CRAWL_COUNT;i++){
									crawl();
									delay(1000);
								}
							}
							else
							{
								//the no wall is found then the zumo performs and extended chack
								if(extendedCheckLeft())
								{
									//if a wall is found then the user is found
									Serial.println("Extended Left");
									//the extended counter is incremented
									ELCount++;
									delay(1000);
									//the zumo moves forward
									for(int i = 0 ;i < CRAWL_COUNT;i++){
										crawl();
										delay(1000);
									}
								}
								else
								{
									//if no wall is found then the currentState is set to LeftError
									currentState = LeftError;
									//leftFirst is set to true so that the zumo checks left first
									leftFirst = true;
								}
							}
						}
						else
						{
							//if no wall is found then the zumo does an extended check
							if(extendedCheckRight())
							{
								//if a wall is found then the user is informed
								Serial.println("Extended Right");
								//the extended check counter is incremented
								ERCount++;
								delay(1000);
								//the zumo then checks for a wall on the left
								if(checkLeft())
								{
									//if a wall is found then the user is informed
									Serial.println("Left");
									delay(1000);
									//the zumo moved forward
									for(int i = 0 ;i < CRAWL_COUNT;i++){
										crawl();
										delay(1000);
									}
								}
								else
								{
									//if no wall is found then the zumo does an extended check
									if(extendedCheckLeft())
									{
										//the user is informed if a wall is found
										Serial.println("Extended Left");
										//the extended counter is incremented
										ELCount++;
										delay(1000);
										//the zumo moves forward
										for(int i = 0 ;i < CRAWL_COUNT;i++){
											crawl();
											delay(1000);
										}
									}
									else
									{
										//if no wall is found on the left then the currentState is set to LeftError
										currentState = LeftError;
										//leftFirst is set to true
										leftFirst = true;	
									}
								}
							}
							else
							{
								//if no wall is no wall is found on the right then the currentState is set to rightError
								currentState = RightError;	
							}

						}
					}
				}
				break;
			}
			case LeftError:
			{
				//leftErrorCheck starts as false and only reached true once the else statement has been ran
				if(leftErrorCheck)
				{
					//the user is informed that there is an error on the left
					Serial.println("LeftError");
					//the zumo moves forward.
					crawl();
					//the zumo turns to the left.
					turnLeft();
					//the zumo moves forward some more
					crawl();
					crawl();
					crawl();
					//the reflectance sensor values are then updated
					reflectanceSensors.read(sensor_values);
					//if either if the end sensors are above the threshold than the zumo is on the edge and the user and informed and the zumo shuffles to the other side to compensate
					if(sensor_values[0] > QTR_THRESHOLD)
					{
						Serial.println("on the edge");
						shuffleRight();
					} 
					else if (sensor_values[5] > QTR_THRESHOLD)
					{
						Serial.println("on the edge");
						shuffleLeft();
					}
					//if a wall is detected on the right and on the left then the zumo is in a corridor
					if((checkRight() || extendedCheckRight())&&(checkLeft() || extendedCheckLeft()))
					{
						//the user is informed of the current location
						Serial.println("in a corridor");
						//the currentState is set to Corridor
						currentState = Corridor;
					}
					else
					{
						//the user is told that the zumo is in a room
						Serial.println("In a room");
						//the currentState is set to LeftRoom
						currentState = LeftRoom;
					}
					//leftErrorCheck is set to false because the error has been resolved
					leftErrorCheck = false;
				}
				else
				{
					//the zumo shuffles to the left because this is the first time that this state has been used
					Serial.println("Left Shuffle");
					shuffleLeft();
					//currentState is set back to Corridoe
					currentState = Corridor;
					//leftErrorCheck is set to true so that next time different actions are performed
					leftErrorCheck = true;
				}
				break;
			}
			case RightError:
			{
				//rightErrorCheck starts as false and only reached true once the else statement has been ran
				if(rightErrorCheck)
				{
					//the user is informed that there is an error on the right
					Serial.println("rightError");
					//the zumo moves forward.
					crawl();
					//the zumo then turns to the right.
					turnRight();
					//the zumo then moves forward some more.
					crawl();
					crawl();
					crawl();
					//the zumo then checks if it is currently on a wall.
					//if it is on a wall then the zumo shuffles over and informs the user that it is on the edge
					reflectanceSensors.read(sensor_values);
					if(sensor_values[0] > QTR_THRESHOLD)
					{
						Serial.println("on the edge");
						shuffleRight();
					} 
					else if (sensor_values[5] > QTR_THRESHOLD)
					{
						Serial.println("on the edge");
						shuffleLeft();
					}
					//if a wall is detected on both the left and right then the zumo is in a corridor
					if((checkLeft() || extendedCheckLeft() ) &&(checkRight() || extendedCheckRight()))
					{
						//the user is informed that the zumo is in a corridor
						Serial.println("in a corridor");
						//the currentState is set to Corridor
						currentState = Corridor;
					}
					else
					{
						//the currentState is set to RightRoom
						currentState = RightRoom;
					}
					//right error check is reset to false because the error has been resolved
					rightErrorCheck = false;
				}
				else
				{
					//because this is the first time the state RightError has been used the zumo shuffles to the right and informs the user.
					Serial.println("Right Shuffle");
					shuffleRight();
					//the currentState is set back to Corridor.
					currentState = Corridor;
					//rightErrorCheck is set to true to the next time this state is reached the result will be different.
					rightErrorCheck = true;
				}
				break;
			}
			case LeftRoom:
			{
				//When the zumo enters a room on the left the serial prints a message to tell the user that its in a room on the right.
				Serial.println("in a room on the left");
				//The zumo then uses the ultrasonic sensor to check if anything is in the room
				if(sonar.ping_cm() <= 20)
				{
					//If something is detected within the room the user is informed and the state is changed to Occupied rooom
					Serial.println("Summat in here Captain");
					currentState = OccupiedRoom;
				}
				else
				{
					//If something is detected within the room the user is informed and the state is changed to Empty rooom
					Serial.println("Empty");
					currentState = EmptyRoom;
				}
				//squareUp() is called which aligns the zumo with the wall of the room opposite the door
				squareUp();
				delay(1000);
				//The zumo the calls spin which turns the zumo 180 degrees
				spin();
				delay(1000);
				//forwardToFullWall() is called whick makes the zumo travel to the corridor wall opposite to the room
				forwardToFullWall();
				delay(1000);
				//The zumo then moves backwards by a small amount to line it up
				motors.setSpeeds(-100,-100);
				delay(200);
				motors.setSpeeds(0,0);
				delay(1000);
				//The zumo then turns to the left
				motors.setSpeeds(-200,200);
				delay(275);
				motors.setSpeeds(0,0);
				delay(1000);
				//The zumo then moves forwards
				for(int i = 0;i<6;i++){
					crawl();
				}
				//The zumo should now be in the corridor after the room it has just checked. the currentState is set to Corridor.
				currentState = Corridor;
				break;
			}
			case RightRoom:
			{
				//When the zumo enters a room on the right the serial prints a message to tell the user that its in a room on the right.
				Serial.println("in a room on the right");
				//The zumo then uses the ultrasonic sensor to check if anything is in the room
				if(sonar.ping_cm() <= 20)
				{
					//If something is detected within the room the user is informed and the state is changed to Occupied rooom
					Serial.println("Summat in here Captain");
					currentState = OccupiedRoom;
				}
				else
				{
					//If something is detected within the room the user is informed and the state is changed to Empty rooom
					Serial.println("Empty");
					currentState = EmptyRoom;
				}
				//squareUp() is called which aligns the zumo with the wall of the room opposite the door
				squareUp();
				delay(1000);
				//The zumo the calls spin which turns the zumo 180 degrees
				spin();
				delay(1000);
				//forwardToFullWall() is called whick makes the zumo travel to the corridor wall opposite to the room
				forwardToFullWall();
				delay(1000);
				//The zumo then moves backwards by a small amount to line it up
				motors.setSpeeds(-100,-100);
				delay(200);
				motors.setSpeeds(0,0);
				delay(1000);
				//The zumo then turns to the right
				motors.setSpeeds(200,-200);
				delay(275);
				motors.setSpeeds(0,0);
				delay(1000);
				//The zumo then moves forwards
				for(int i = 0;i<6;i++){
					crawl();
				}
				//The zumo should now be in the corridor after the room it has just checked. the currentState is set to Corridor.
				currentState = Corridor;
				break;
			}
			case Fire:
			{
				//If the current state is fire the the zumo should stop and not procede any further.
				motors.setSpeeds(0,0);
				//a serial message is also printed out to tell the user that the zumo has detected the fire
				Serial.println("The Fire has been reached");
				delay(1000);
				break;
			}
			case EmptyRoom:
			{
				break;
			}
			case OccupiedRoom:
			{
				break;
			}
			default:
			delay(1000);
		}
	}
}
void crawl(){
	//this moves the zumo forward.
	motors.setSpeeds(left,right);
	delay(100);
	motors.setSpeeds(0,0);
}
boolean checkLeft(){
	//This takes a reading on the left refelctance array sensor and stores this as start.
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float start = sensor_values[0];
	//the zumo then turns to the left
	motors.setSpeeds(-100,195);
	delay(260);
	motors.setSpeeds(0,0);
	//The zumo then takes a second reading and stores this as finish
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float finish = sensor_values[0];
	//The zumo then turns back round to the right
	motors.setSpeeds(100,-200);
	delay(230);
	motors.setSpeeds(0,0);
	Serial.print(start);
	Serial.print(" - ");
	Serial.println(finish);
	//the zumo returns true if a wall has been detected.
	return finish>start+CALIBRATION;
}
boolean extendedCheckLeft(){
	//This takes a reading on the left refelctance array sensor and stores this as start.
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float start = sensor_values[0];
	//the zumo then turns to the left
	motors.setSpeeds(-100,195);
	delay(260);
	motors.setSpeeds(0,0);
	delay(1000);
	//the zumo then moves forwards
	motors.setSpeeds(left,right);
	delay(100);
	//if the zumo is currently not in a room or a corridor then it moves twice as far.
	if ((currentState == LeftError)||(currentState == RightError)){
		delay(100);
	}
	motors.setSpeeds(0,0);
	delay(1000);
	//The zumo then takes a second reading and stores this as finish
	reflectanceSensors.read(sensor_values);
	float finish = sensor_values[0];
	//the zumo then turns back to the right.
	motors.setSpeeds(100,-200);
	delay(235);
	motors.setSpeeds(0,0);

	delay(1000);
	Serial.print(start);
	Serial.print(" - ");
	Serial.println(finish);
	//the zumo returns true if a wall has been detected.
	return finish>start+CALIBRATION;
}
boolean checkRight(){
	//This takes a reading on the left refelctance array sensor and stores this as start.
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float start = sensor_values[5];
	//the zumo then turns to the left
	motors.setSpeeds(200,-100);
	delay(240);
	motors.setSpeeds(0,0);
	//The zumo then takes a second reading and stores this as finish
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float finish = sensor_values[5];
	//The zumo then turns back round to the right
	motors.setSpeeds(-200,100);
	delay(230);
	motors.setSpeeds(0,0);
	Serial.print(start);
	Serial.print(" - ");
	Serial.println(finish);
	//the zumo returns true if a wall has been detected.
	return finish>start+CALIBRATION;
}
boolean extendedCheckRight(){
	//This takes a reading on the left refelctance array sensor and stores this as start.
	delay(1000);
	reflectanceSensors.read(sensor_values);
	float start = sensor_values[5];
	//the zumo then turns to the right
	motors.setSpeeds(200,-100);
	delay(240);
	motors.setSpeeds(0,0);
	//the zumo then moves forwards
	delay(1000);
	motors.setSpeeds(left,right);
	delay(100);
	//if the zumo is currently not in a room or a corridor then it moves twice as far.
	if ((currentState == LeftError)||(currentState == RightError)){
		delay(100);
	}
	motors.setSpeeds(0,0);
	delay(1000);
	//The zumo then takes a second reading and stores this as finish
	reflectanceSensors.read(sensor_values);
	float finish = sensor_values[5];
	//the zumo then turns back to the right.
	motors.setSpeeds(-200,100);
	delay(230);
	motors.setSpeeds(0,0);
	delay(1000);
	Serial.print(start);
	Serial.print(" - ");
	Serial.println(finish);
	//the zumo returns true if a wall has been detected.
	return finish>start+CALIBRATION;
}
void shuffleLeft(){
	//the zumo spins left
	ELCount = 0;
	delay(1000);
	motors.setSpeeds(-100,195);
	delay(260);
	motors.setSpeeds(0,0);
	delay(1000);
	//Then moves forwards
	motors.setSpeeds(left,right);
	delay(100);
	motors.setSpeeds(0,0);
	delay(1000);
	//then spins back to the right
	motors.setSpeeds(100,-200);
	delay(235);
	motors.setSpeeds(0,0);
	delay(1000);
}
void shuffleRight(){
	//The zumo spins to the right
	ERCount = 0;
	delay(1000);
	motors.setSpeeds(200,-100);
	delay(240);
	motors.setSpeeds(0,0);
	delay(1000);
	//Then moves forward
	motors.setSpeeds(left,right);
	delay(100);
	motors.setSpeeds(0,0);
	delay(1000);
	//Then spins back to the left
	motors.setSpeeds(-200,100);
	delay(230);
	motors.setSpeeds(0,0);
	delay(1000);
}
void turnLeft(){
	motors.setSpeeds(-200,200);
	delay(250);
	motors.setSpeeds(0,0);
}
void turnRight(){
	motors.setSpeeds(200,-200);
	delay(250);
	motors.setSpeeds(0,0);
}
void squareUp(){
	reflectanceSensors.read(sensor_values);
	while((sensor_values[0]< QTR_THRESHOLD)&&(sensor_values[5]< QTR_THRESHOLD))
	{
		motors.setSpeeds(left,right);
		reflectanceSensors.read(sensor_values);
	}
	motors.setSpeeds(0,0);
	reflectanceSensors.read(sensor_values);
	while((sensor_values[0]< QTR_THRESHOLD)||(sensor_values[5]< QTR_THRESHOLD))
	{
		if(sensor_values[0] > sensor_values[5])
		{
			motors.setSpeeds(-100,100);
			delay(100);
		}
		else
		{
			motors.setSpeeds(100,-100);
			delay(100);
		}
		motors.setSpeeds(0,0);
		reflectanceSensors.read(sensor_values);
	}
	Serial.println("Squared up");
}
void spin(){
	Serial.println("Spinning");
	motors.setSpeeds(200,-200);
	delay(500);
	motors.setSpeeds(0,0);
}
void forwardToFullWall(){ 
	reflectanceSensors.read(sensor_values);
	while(!((sensor_values[0]>QTR_THRESHOLD) && (sensor_values[5]>QTR_THRESHOLD)))
	{
		motors.setSpeeds(left,right);
		reflectanceSensors.read(sensor_values);
	}
	motors.setSpeeds(0,0);
}
void displaySensorDetails(){
	sensor_t sensor;
	tsl.getSensor(&sensor);
	Serial.println("------------------------------------");
	Serial.print  ("Sensor:       "); Serial.println(sensor.name);
	Serial.print  ("Driver Ver:   "); Serial.println(sensor.version);
	Serial.print  ("Unique ID:    "); Serial.println(sensor.sensor_id);
	Serial.print  ("Max Value:    "); Serial.print(sensor.max_value); Serial.println(" lux");
	Serial.print  ("Min Value:    "); Serial.print(sensor.min_value); Serial.println(" lux");
	Serial.print  ("Resolution:   "); Serial.print(sensor.resolution); Serial.println(" lux");  
	Serial.println("------------------------------------");
	Serial.println("");
	delay(500);
}
void configureSensor(){
	tsl.setIntegrationTime(TSL2561_INTEGRATIONTIME_13MS);
	Serial.println("------------------------------------");
	Serial.print  ("Gain:         "); Serial.println("Auto");
	Serial.print  ("Timing:       "); Serial.println("13 ms");
	Serial.println("------------------------------------");
}
float getLightValue(){
	sensors_event_t event;
	tsl.getEvent(&event);
	if (event.light)
	{
		return event.light;
	}
	else
	{
		Serial.println("Sensor overload");
		return NULL;
	}
}
{% endhighlight %}
