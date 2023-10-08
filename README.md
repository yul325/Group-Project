# Group-Project

Project 3 Outline: [[project-3.pdf]]
Project 3 Diagram: [[Project 3]]
### Development cycle of database
1. Conceptual Design
2. Logical Design
3. Schema evolution
4. Physical design
5. Implementation of an application

---
### **ArborDB**
1. network of IoT sensors used to monitor forest in US
2. The ArborDB database stores information about forests in the US and the network of devices that oversee these forests

### **ArborDB** Three phases
1. Database design
2. Query Series
3. Java App (JDBC)


# Phase 1 (Conceptual Design): 
---
- *ArborDB* monitors a number of forests within the US, each of which has a *unique forest number, name, area, acid level, and a sensor count*. In addition, the *location* of each forest is represented as a *Minimum Boundary Rectangle* (MBR), which is a rectangle that contains the forest using the minimum size. The MBR of a forest is stored as *MBR XMin, MBR XMax, MBR YMin, MBR YMax, which define the size of the forest’s MBR*.

``` PostgreSQL
CREATE TABLE FOREST(
	forestNumber integer,
	forestName varchar(),
	area decimal() as ((MBR_XMax-MBR_XMin)*(MBR_YMax-MBR_YMin)),
	acidLevel decimal(12,2),
	sensorCount integer,
	MBR_XMin decimal(), --- decimal()
	MBR_XMax decimal(), --- decimal()
	MBR_YMin decimal(), --- decimal()
	MBR_YMAX decimal(), --- decimal()

	stateName varchar(2)
	stateArea decimal(),
	percentage decimal() as (forestArea/ stateArea)*100,

	PRIMARY KEY (forestNumber),
	FOREIGN KEY(stateName) REFERENCES STATE(statenName),
	FOREIGN KEY(stateArea) REFERENCES STATE(area),
)
```

- ArborDB also stores the *states of the US* for determining which states are responsible for maintaining each forest. Each US state consists of a *unique name and is recognized by its abbreviation* (e.g., PA, NY, CA, etc.). *The area and population* of each state is stored in the ArborDB database. In addition, the location of each state is represented as a Minimum Boundary Rectangle (MBR) similar to forests. The MBR of a state is stored as *MBR XMin, MBR XMax, MBR YMin, MBR YMax*. As part of determining which states are responsible for maintaining each forest, *ArborDB stores the coverage of each forest with the percentage and area that a forest is covered by each state*.

```PostgreSQL
CREATE TABLE STATE(
	stateName char(2),
	population integer,
	area decimal() as ((MBR_XMax-MBR_XMin)*(MBR_YMax-MBR_YMin)),
	MBR_XMin decimal(), 
	MBR_XMax decimal(), 
	MBR_YMin decimal(), 
	MBR_YMax decimal(), 

	PRIMARY KEY (stateName),
);
```

```PostgreSQL
CREATE TABLE COVERAGE(
	stateName char(2);
	forestNumber integer;
	percentage decimal() as (forestArea/ stateArea)*100;
	stateArea decimal
	forestArea decimal

	PRIMARY KEY (stateName, forestNumber),
	FOREIGN KEY(forestArea) REFERENCES FOREST(area),
	FOREIGN KEY(stateArea) REFERENCES STATE(area)
)
```


- In addition to monitoring forests in the US and the area of each forest, ArborDB also keeps track of the different *species of trees* found within the US. Each tree species consists of a *unique species name (genus and epithet), an ideal temperature for survival, a largest recorded height for the tree species, common name(s), and a Raunkiaer life form*. There are seven different kinds of plant life forms based on the Raunkiaer system: Phanerophytes, Epiphytes, Chamae- phytes, Hemicryptophytes, Cryptophytes, Therophytes, and Aerophytes. The forests where each tree species is known to be found is stored in the ArborDB database in order to help trace the growth or decline of each tree species

```PostgreSQL
CHECK DOMAIN raunkiaer_life_form AS VARCHAR
	CHECK (VALUE IN ('Phanerophytes', 'Epiphytes', 'Chamae- phytes', 'Hemicryptophytes', 'Cryptophytes', 'Therophytes', 'Aerophytes'));

CREATE TABLE SPECIES(
	speciesName varchar(),
	temperature decimal(),
	heightMax decimal(),
	commonNames varchar(),
	lifeForm raunkiaer_life_form,

	PRIMARY KEY (speciesName)
);

CREATE TABLE TREES(
	speciesName varchar(),
	forestNumber integer(),

	PRIMARY KEY (speciesName, forestNumber),
	FOREIGN KEY (speciesName) REFERENCES SPECIES(speciesName),
	FOREIGN KEY (forestNumber) REFERENCES FOREST(forestNumber)
)

```

- As part of monitoring forests in the US, ArborDB stores basic information about the *US Forest Services’ workers* that are tasked with maintaining sensors. Each worker has a *unique SSN (Social Security Number)*, *name* (*first name, last name, middle initial*), *work phone number*(s), and *a rank*. As part of the efforts to maintain the forests within the US, ArborDB also stores the states where each worker is currently employed to ensure that each state with coverage of a forest is contributing to the forest’s maintenance

```PostgreSQL
CREATE TABLE WORKER(
	SSN integer,
	firstName varchar(),
	lastName varchar(),
	middleInitial char(1),
	phone varchar(10),
	workerRank varchar(),
	stateName char(2),

	PRIMARY KEY (SSN),
	FOREIGN KEY (stateName) REFERENCES STATE(stateName)
)
```

- As part of observing and understanding the forests of the US, ArborDB deploys a network of sensors that are maintained by a worker from the US Forest Service. Each sensor consists of a unique sensor id, the last time when the device was charged (last charged), the current energy of the device, and the last time that the device was read (last read). The sensor has a location where it was deployed, which is identified by its coordinate pair (X, Y).
```PostgreSQL
CREATE TABLE SENSOR(
	sensorID integer,
	lastCharged timestamp,
	lastRead timestamp,
	energy decimal(),
	x_coordinate decimal(),
	y_coordinate decimal(),

	forestNumber integer,

	PRIMARY KEY (sensorID),
	FOREIGN KEY forestNumber REFERENCES FOREST(forestNumber)
)
```


- In order to monitor the area surrounding a sensor, each device regularly generates a report stored in ArborDB database. Each report contains the time of reporting (report time) and the temperature recorded by the sensor.
```PostgreSQL
CREATE TABLE REPORT(
	reportTime timestamp,
	temperature decimal(),
	
	sensorID integer,

	PRIMARY KEY (sensorID, reportTime),
	FOREIGN KEY sensorID REFERENCES SENSOR(sensorID)
)
```
