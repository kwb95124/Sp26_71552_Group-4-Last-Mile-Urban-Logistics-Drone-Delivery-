# NAME:
# Sp26_71552_Group 4: "Last-Mile" Urban Logistics (Drone Delivery)
# Urban Logistics: Drone Delivery System

# Team Members:
Guy Myer  
David Moreno  
Mariana Munoz                      
Nadia Nazeem  
Kevin Behlke  


# Project Overview

This project models a drone-based delivery system designed to improve efficiency in last-mile urban logistics. Last-mile delivery refers to the final stage of the supply chain, where goods are transported from a local hub to the customer’s door. Although it is the shortest part of the delivery process, it is often the most complex and expensive due to factors such as traffic congestion, frequent stops, and the need for individualized deliveries in urban environments.

To address these challenges, companies are increasingly exploring solutions such as drone delivery to improve speed and reduce operational costs. Our system focuses on organizing and optimizing drone operations while ensuring safety and reliability through structured data management.


# System Focus and Objectives 

The primary focus of this system is:
- Fleet maintenance  
- Route optimization  

Our goal is to design a system that keeps drone delivery operations efficient, safe, and scalable by enforcing key constraints such as drone status validation, maintenance tracking, and charging station limitations.


# Data Model
<img width="1270" height="801" alt="Screenshot 2026-04-02 215114" src="https://github.com/user-attachments/assets/cae8f091-7f6f-4a00-97c3-cb9eb8acadf8" />

The data model illustrates how drones, trips, packages, and supporting entities interact to manage delivery operations and enforce system constraints.

At the center of the system is the Drone entity, which connects operational activity with maintenance and resource management. Each drone can complete many trips, accumulate maintenance logs over time, and participate in charging sessions. This allows the system to track both performance and upkeep, ensuring that drones remain reliable for delivery operations.

A Storage_Facility serves as the operational hub, housing both drones and technicians. One facility can support many drones and many technicians, creating a structured environment for both deployment and maintenance. Technicians generate maintenance logs for drones, allowing the system to monitor servicing and enforce maintenance requirements based on flight hours.

On the delivery side, the system connects warehouses, routes, trips, and packages to represent how goods move through the network. A warehouse can store many packages and support multiple routes. Each route can be used for many trips, and each trip is assigned to one drone. Packages are linked to trips through the Trip_Packages table, which represents a many-to-many relationship. This allows a single trip to carry multiple packages while maintaining flexibility in delivery assignments.

Shipping destinations are connected to packages to represent delivery endpoints, while routes ensure that trips follow structured and optimized paths. Together, these relationships allow the system to efficiently coordinate deliveries across an urban environment.

The model also supports key operational constraints. Charging stations and charging sessions track when and where drones are charging, ensuring that only one drone occupies a station at a given time. In addition, maintenance logs track drone usage and enforce servicing once flight-hour thresholds are exceeded. These constraints help maintain safety and prevent operational conflicts.


# Business Rules

The system enforces the following rules:

1. Drones must have a valid status (In-Flight, Charging, Maintenance) before being assigned to a trip.  
2. Maintenance logs must be created when a drone exceeds 50 flight hours.  
3. Charging stations can only serve one drone at a time, with start and end times recorded.

# Data Dictionary

# Simple Queries:

## Query for drone models in the Athens area that is AeroX Pro

### Managerial Explanation: This query helps management identify which deliveries in the Athens area are being handled by a specific drone model. This is useful for tracking how certain models are being deployed in different locations, evaluating model performance in a target service area, and supporting decisions about where particular drone types should be used most often.

#### SELECT shipping_destination.destinationAddress, Drone.model FROM Trip JOIN Drone ON Trip.Drone_droneID = Drone.droneID JOIN shipping_destination  ON shipping_destination.Trip_TripID = Trip.TripID WHERE destinationAddress LIKE '%Athens%' and model = "AeroX Pro";

<img width="630" height="126" alt="image" src="https://github.com/user-attachments/assets/04d96ebe-dd93-42c3-b390-49c87ac37051" />

## Drone and status 

### Managerial Explanation: This query gives managers a quick snapshot of the current condition of the fleet by showing each drone’s status, battery level, and flight hours. It helps support day-to-day operational decisions by showing which drones may be available, which may need attention soon, and which are approaching maintenance thresholds.

#### SELECT Drone.droneID, Drone.status, Drone.battery, Drone.flightHours FROM Drone;
<img width="394" height="259" alt="image" src="https://github.com/user-attachments/assets/01714170-e013-42b6-8f09-1390ac0ec179" />

## Drones needing maintenance (> 50 hours)
### Managerial Explanation: This query helps managers identify drones that have exceeded the maintenance threshold and may require servicing. It is useful for preventative maintenance planning, reducing the risk of equipment failure, and making sure the fleet remains safe and reliable during delivery operations.

#### SELECT Drone.droneID, Drone.flightHours, Drone.status FROM Drone WHERE Drone.flightHours > 50;

<img width="570" height="269" alt="image" src="https://github.com/user-attachments/assets/841a03f3-8bc1-4e49-8d5a-32c36262bd78" />

## Packages assigned to trips
### Managerial Explanation: This query helps managers understand how packages are being distributed across trips. It is useful for monitoring package loads, verifying that delivery assignments are being made correctly, and evaluating whether trips are being used efficiently to carry multiple deliveries at once.

#### SELECT Trip_Packages.Trip_TripID, Packages.packageID,  Packages.packageType, Packages.weight, Packages.packageContent FROM Trip_Packages JOIN Packages ON Trip_Packages.Packages_packageID = Packages.packageID;

<img width="656" height="475" alt="image" src="https://github.com/user-attachments/assets/5be83f28-4f71-4607-9b51-bf8fd4d75a40" />

## Total distance per drone
### Managerial Explanation: This query allows managers to measure how heavily each drone is being used by showing the total distance it has traveled. It helps with workload balancing, identifying high-use drones, planning maintenance more effectively, and evaluating whether fleet usage is being distributed efficiently.

#### SELECT Trip.Drone_droneID, SUM(Trip.tripDistance) AS total_distance_flown FROM Trip GROUP BY Trip.Drone_droneID;

<img width="413" height="230" alt="image" src="https://github.com/user-attachments/assets/e0bba2ff-25ff-47ae-9134-513ae0c88e36" />

## Packages with destination
### Managerial Explanation: This query helps managers track what packages are going to which delivery destinations. It is useful for monitoring delivery patterns, confirming routing accuracy, and understanding customer demand by location. Over time, this can support better route planning and service allocation.

#### SELECT packageID, packageType, weight, destinationAddress FROM Packages JOIN shipping_destination ON Packages.shipping_destination_destinationID = shipping_destination.destinationID;

<img width="580" height="335" alt="image" src="https://github.com/user-attachments/assets/57b459ef-9da2-475f-a3ba-7f28711d1aa6" />

# Complex Queries 

## Use this to see which drones that are in flight are low on battery (and need to find a charging station soon)
### Managerial Explanation: This query helps managers quickly evaluate which active drones are in strong condition and which may need charging soon. Instead of only showing a battery number, it translates battery levels into more practical categories, making it easier to prioritize operational decisions and respond quickly when a drone may need to leave service for charging.

#### SELECT *, CASE 	WHEN battery > 67 THEN "Good to go"       WHEN battery > 33 THEN "Monitor Battery" WHEN battery <= 32 THEN "Charge Soon" END AS BatteryLevel FROM Drone WHERE status = "In-Flight";

<img width="872" height="102" alt="image" src="https://github.com/user-attachments/assets/64bd169e-1daa-423f-b661-4983b993598a" />

## This query lists all of the drones, their ids, their current status, their total trips made, and their total distance traveled.
### Managerial Explanation: This query gives managers a stronger view of drone productivity by combining operational status with trip count and total distance traveled. It helps identify the most heavily used drones, compare fleet performance, and support decisions about workload balancing, resource planning, and maintenance scheduling.

#### SELECT Drone.droneID, Drone.status AS "CurrentStatus", COUNT(Trip.tripID) AS "TotalTrips", SUM(Trip.tripDistance) AS "TotalDistance" FROM Drone LEFT JOIN Trip ON Drone.droneID = Trip.Drone_droneID GROUP BY Drone.droneID, Drone.status HAVING totalTrips > 0 ORDER BY totalDistance DESC;

<img width="624" height="264" alt="image" src="https://github.com/user-attachments/assets/b01a1b30-0362-4093-9978-3d42e92d0b70" />

## This query calculates how many packages each warehouse has handled and the total weight of those packages, then ranks the warehouses from highest to lowest activity, for warehouses that have handled at least one package.
### Managerial Explanation: This query helps management evaluate warehouse activity by showing how many packages each warehouse has handled and the total weight processed. It is useful for comparing warehouse workload, identifying high-volume locations, and supporting decisions related to staffing, resource allocation, and operational efficiency.

#### SELECT Warehouse.warehouseID, COUNT(Packages.packageID) AS TotalPackagesHandled, SUM(Packages.weight) AS TotalWeightHandled FROM Warehouse JOIN Packages ON Warehouse.warehouseID = Packages.Warehouse_warehouseID GROUP BY Warehouse.warehouseID HAVING COUNT(Packages.packageID) > 0 ORDER BY TotalPackagesHandled DESC;

<img width="553" height="175" alt="image" src="https://github.com/user-attachments/assets/855c610f-49e0-45a0-a2a6-58c596e3f4c4" />

## SUB QUERY: Drones with above average flight hours 
### Managerial Explanation: This query helps managers identify drones that are being used more heavily than the rest of the fleet. It is useful for spotting overutilized assets, determining which drones may need closer monitoring, and making better decisions about balancing usage across the fleet.

#### SELECT Drone.droneID, Drone.flightHours FROM Drone WHERE Drone.flightHours > ( SELECT AVG(Drone.flightHours) FROM Drone);

<img width="250" height="148" alt="image" src="https://github.com/user-attachments/assets/f4282411-8081-460a-9203-cf4e81eecc9f" />

## EXISTS: Find technicians who have performed at least one maintenance job
### Managerial Explanation:This query helps managers confirm which technicians are actively contributing to maintenance operations. It can be useful for workforce evaluation, staffing reviews, and understanding which personnel are involved in maintaining fleet readiness.

#### SELECT Technicians.technicianID, Technicians.techniciansFName, Technicians.techniciansLName FROM Technicians WHERE EXISTS (    SELECT * FROM maintenance_logs  WHERE maintenance_logs.Technicians_technicianID = Technicians.technicianID );

<img width="430" height="225" alt="image" src="https://github.com/user-attachments/assets/b667a269-68d2-406b-aff1-9cbe3b62db0c" />

## Find drones that have at least one trip over 10 miles
### Managerial Explanation: This query helps managers identify drones that are being used for longer-distance deliveries. This is useful for evaluating route demands, determining which drones are handling more intensive assignments, and assessing whether long-distance trips are being assigned appropriately based on drone capability and battery constraints.

#### SELECT Drone.droneID FROM Drone WHERE EXISTS (    SELECT *     FROM Trip    WHERE Trip.Drone_droneID = Drone.droneID  AND Trip.tripDistance > 10 );

<img width="174" height="286" alt="image" src="https://github.com/user-attachments/assets/736f6748-ae2f-4028-bd26-715ad4d4d793" />

