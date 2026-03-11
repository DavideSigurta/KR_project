# Smart Home OWL Ontology
### Knowledge Representation — University of Verona
**Author:** Davide Giuseppe Sigurtà

---

## Project Overview

This project implements a **OWL ontology for a Smart Home domain** . The ontology models the spatial structure of a home, its connected devices, automation controllers, and scenes without modeling social actors (persons). 
---

## Domain and Motivation

A smart home is modeled from the perspective of **comfort automation and device management**. The domain covers:

- **Where** things are: rooms, zones, outdoor spaces
- **What** is installed: sensors (measure) and actuators (act)
- **Who** controls them: home hubs and mobile apps
- **How** they are automated: scenes activated by controllers
- **How** they communicate: network connectivity

This framing follows the course principle that the purpose and relevance scheme determine what gets represented.

---

## Ontology Structure

### Classes (26 total)

#### Primitive Classes (19)

| Branch | Classes |
|---|---|
| **Location** | `Room`, `Zone`, `OutdoorSpace` |
| **Device** | `Sensor`, `Actuator`, `TemperatureSensor`, `MotionSensor`, `Camera`, `Light`, `Thermostat`, `SmartLock` |
| **Controller** | `MobileApp`, `HomeHub` |
| **Other** | `Scene`, `Network` |

#### Defined Classes (7)

| Class | Definition | DL Construct |
|---|---|---|
| `ActiveDevice` | `Device ⊓ (isActive value true)` | HasValue |
| `MonitoredLocation` | `Location ⊓ (∃ isMonitoredBy . Camera)` | Existential `∃` |
| `EquippedRoom` | `Room ⊓ (∃ contains . Device)` | Existential `∃` |
| `ClimateControlledRoom` | `Room ⊓ (∃ contains . Thermostat)` | Existential `∃` |
| `SensorOnlyRoom` | `Room ⊓ (∀ contains . Sensor)` | Universal `∀` |
| `WellEquippedRoom` | `Room ⊓ (≥2 contains . Device)` | Cardinality |
| `HubControlledDevice` | `Device ⊓ (∀ isControlledBy . HomeHub)` | Universal `∀` |

---

### Object Properties (11)

| Property | Domain | Range | Characteristics |
|---|---|---|---|
| `locatedIn` | `Device` | `Room` | **Functional**, Inverse: `contains` |
| `contains` | `Room` | `Device` | Inverse: `locatedIn` |
| `hasRoom` | `Zone` | `Room` | Inverse: `isPartOfZone` |
| `isPartOfZone` | `Room` | `Zone` | Inverse: `hasRoom` |
| `monitors` | `Camera` | `Location` | Inverse: `isMonitoredBy` |
| `isMonitoredBy` | `Location` | `Camera` | Inverse: `monitors` |
| `controls` | `Controller` | `Device` | Inverse: `isControlledBy` |
| `isControlledBy` | `Device` | `Controller` | Inverse: `controls` |
| `activates` | `Controller` | `Scene` | — |
| `connectedTo` | `Device` | `Network` | **Functional** |
| `isLocatedInZone` | `Device` | `Zone` | Inferred via property chain |

**Property Chain:**
`locatedIn ∘ isPartOfZone ⊑ isLocatedInZone`
If a device is in a Room, and that Room is part of a Zone, the reasoner automatically infers the device is located in that Zone without any explicit assertion.

---

### Data Properties (4)

| Property | Domain | Range | Characteristics |
|---|---|---|---|
| `deviceName` | `Device` | `xsd:string` | **Functional** |
| `temperatureValue` | `TemperatureSensor` | `xsd:float` | — |
| `brightnessLevel` | `Light` | `xsd:integer` | — |
| `isActive` | `Device` | `xsd:boolean` | — |

---

### Key Disjointness Axioms

- **DisjointUnion** on `Location`: every Location is exactly one of `Room`, `Zone`, or `OutdoorSpace` — the partition is complete and closed.
- **DisjointClasses** between all top-level branches: `Location`, `Device`, `Controller`, `Scene`, `Network`.
- **DisjointClasses** between `Sensor` and `Actuator`.
- **DisjointClasses** among all leaf subclasses (`Camera`, `MotionSensor`, `TemperatureSensor`; `Light`, `Thermostat`, `SmartLock`; `MobileApp`, `HomeHub`).

---

## ABox — Individuals (20)

| Individual | Class | Notable assertions |
|---|---|---|
| `LivingRoom` | `Room` | Contains HueBulb_01, EntranceCam, FrontDoorLock |
| `Kitchen` | `Room` | Contains Nest_Thermo, KitchenTempSensor |
| `Bedroom` | `Room` | Contains BedroomLight, BedroomMotion |
| `SecurityRoom` | `Room` | Contains only SecurityMotion (Sensor) |
| `Garden` | `OutdoorSpace` | Monitored by GardenCam |
| `LivingArea` | `Zone` | hasRoom: LivingRoom, Kitchen |
| `NightZone` | `Zone` | hasRoom: Bedroom |
| `GardenCam` | `Camera` | monitors: Garden |
| `EntranceCam` | `Camera` | monitors: LivingRoom, locatedIn: LivingRoom |
| `BedroomMotion` | `MotionSensor` | locatedIn: Bedroom |
| `SecurityMotion` | `MotionSensor` | locatedIn: SecurityRoom |
| `KitchenTempSensor` | `TemperatureSensor` | temperatureValue: 21.5 |
| `Nest_Thermo` | `Thermostat` | locatedIn: Kitchen |
| `HueBulb_01` | `Light` | isActive: true, brightnessLevel: 80 |
| `BedroomLight` | `Light` | isActive: **false**, brightnessLevel: 0 |
| `FrontDoorLock` | `SmartLock` | controlled by MainHub AND OwnerPhone |
| `MainHub` | `HomeHub` | controls: HueBulb_01, Nest_Thermo, BedroomLight, FrontDoorLock |
| `OwnerPhone` | `MobileApp` | controls: FrontDoorLock, GardenCam |
| `NightMode` | `Scene` | activated by MainHub and OwnerPhone |
| `HomeWifi` | `Network` | All devices connected here |

---

## How to Open

1. Download `smarthome.owl`
2. Open **Protégé 5.5**
3. File → Open → select `smarthome.owl`
4. Reasoner → **HermiT** → Start Reasoner
