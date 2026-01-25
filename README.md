# Cab-Booking-Analysis
```sql
Create Database Project;
use Project;
```

```sql
-- 1. Customers
Create table Customers (
CustomerID INT PRIMARY KEY,
Name VARCHAR(100),
Phone VARCHAR(15),
Email VARCHAR(100),
JoinDate DATE);
```

```sql
INSERT INTO Customers (CustomerID, Name, Phone, Email, JoinDate) VALUES
(1, 'Soham Pandey', '9876543210', 'soham@example.com', '2024-01-10'),
(2, 'Jay Tiwari', '8765432109', 'jay@example.com', '2024-02-15'),
(3, 'Arnav Yadav', '7654321098', 'arnav1@example.com', '2024-03-01'),
(4, 'AKASH Singh', '6543210987', 'aksh@example.com', '2024-04-05');
```

```sql
Select * from Customers;
```

-- 2. Drivers

Create table Drivers (
DriverID INT PRIMARY KEY,
Name VARCHAR(100),
Phone VARCHAR(15),
LicenseNumber VARCHAR(50),
JoinDate DATE,
Rating FLOAT);

INSERT INTO Drivers (DriverID, Name, Phone, LicenseNumber, JoinDate, Rating) VALUES
(1, 'Raj Pandey', '9123456789', 'DL12345678', '2023-09-01', 4.5),
(2, 'Sunny Chaudhary', '9234567890', 'DL87654321', '2023-10-12', 3.2),
(3, 'Riya P', '9345678901', 'DL23456789', '2024-01-20', 2.8),
(4, 'Seema Kapoor', '9456789012', 'DL34567890', '2024-03-15', 4.0);


````sql

Select * from Drivers;
```

-- 3. Cabs

Create table Cabs(
CabID INT PRIMARY KEY,
DriverID INT,
CabType VARCHAR(20), -- 'Sedan', 'SUV', etc.
PlateNumber VARCHAR(20),
FOREIGN KEY (DriverID) REFERENCES Drivers(DriverID));

INSERT INTO Cabs (CabID, DriverID, CabType, PlateNumber) VALUES
(1, 1, 'Sedan', 'KA01AB1234'),
(2, 2, 'SUV', 'KA01CD5678'),
(3, 3, 'Sedan', 'KA01EF9012'),
(4, 4, 'SUV', 'KA01GH3456');

````sql
Select * from Cabs;

-- 4. Bookings
 
Create table Bookings(
BookingID INT PRIMARY KEY,
CustomerID INT,
CabID INT,
BookingTime DATETIME,
TripStartTime DATETIME,
TripEndTime DATETIME,
PickupLocation VARCHAR(100),
DropoffLocation VARCHAR(100),
Status VARCHAR(20), -- 'Completed', 'Cancelled', 'Ongoing'
FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
FOREIGN KEY (CabID) REFERENCES Cabs(CabID));

INSERT INTO Bookings (BookingID, CustomerID, CabID, BookingTime, TripStartTime, TripEndTime, PickupLocation, DropoffLocation, Status) VALUES
(101, 1, 1, '2025-05-01 08:00:00', '2025-05-01 08:10:00', '2025-05-01 08:40:00', 'Downtown', 'Airport', 'Completed'),
(102, 2, 2, '2025-05-01 09:00:00', NULL, NULL, 'Station', 'Mall', 'Cancelled'),
(103, 1, 3, '2025-05-02 10:00:00', '2025-05-02 10:15:00', '2025-05-02 10:50:00', 'Downtown', 'Hospital', 'Completed'),
(104, 3, 4, '2025-05-03 11:30:00', '2025-05-03 11:45:00', '2025-05-03 12:30:00', 'Mall', 'University', 'Completed'),
(105, 4, 1, '2025-05-04 14:00:00', NULL, NULL, 'Airport', 'Downtown', 'Cancelled');

````sql
Select * from Bookings;

```
-- 5. TripDetails

Create table TripDetails(
TripID INT PRIMARY KEY,
BookingID INT,
Distance FLOAT, -- in kilometers
Fare DECIMAL(10,2),
DriverRating FLOAT,
FOREIGN KEY (BookingID) REFERENCES Bookings(BookingID));

INSERT INTO TripDetails (TripID, BookingID, Distance, Fare, DriverRating) VALUES
(1001, 101, 12.5, 250.00, 5.0),
(1002, 103, 10.0, 200.00, 4.0),
(1003, 104, 15.0, 300.00, 3.5);
````sql

Select * from TripDetails;

-- Note: Bookings 102 and 105 are cancelled, so they don’t appear here.

```

-- 6. Feedback

Create table Feedback(
FeedbackID INT PRIMARY KEY,
BookingID INT,
CustomerFeedback TEXT,
ReasonForCancellation VARCHAR(100),
FOREIGN KEY (BookingID) REFERENCES Bookings(BookingID));

INSERT INTO Feedback (FeedbackID, BookingID, CustomerFeedback, ReasonForCancellation) VALUES
(501, 102, 'Cab was late, had to cancel.', 'Driver Delay'),
(502, 105, 'Change of plans.', 'Customer Personal Reason');

````sql
Select * from Feedback;
 
 
 ##. Identify customers who have completed the most bookings. What insights can you
##draw about their behavior?
SELECT c.CustomerID, c.Name,
       COUNT(b.BookingID) AS completed_count
FROM Customers c
JOIN Bookings b ON c.CustomerID = b.CustomerID
WHERE b.Status = 'Completed'
GROUP BY c.CustomerID, c.Name
ORDER BY completed_count DESC
limit 1;

-- 2 Find customers who have canceled more than 30% of their total bookings
SELECT c.Name, 
       COUNT(CASE WHEN b.Status = 'Cancelled' THEN 1 END) * 1.0 / COUNT(b.BookingID) AS CancellationRate
FROM Customers c
JOIN Bookings b ON c.CustomerID = b.CustomerID
GROUP BY c.CustomerID
HAVING CancellationRate > 0.3;

-- 3 . Determine the busiest day of the week for bookings
SELECT DAYNAME(b.BookingTime) AS day_of_week,
       COUNT(*) AS bookings_count
FROM Bookings b
GROUP BY DAYOFWEEK
(b.BookingTime)
ORDER BY bookings_count DESC;

-- # Driver Performance & Efficiency

-- 1 Identify drivers who have received an average rating below 3.0 in the past three
-- months.
SELECT d.DriverID, d.Name,
       AVG(td.DriverRating) AS avg_driver_rating,
       COUNT(td.TripID) AS trips_count
FROM Drivers d
JOIN Cabs c ON d.DriverID = c.DriverID
JOIN Bookings b ON c.CabID = b.CabID
JOIN TripDetails td ON b.BookingID = td.BookingID
WHERE b.TripEndTime >= DATE_SUB(current_date(), INTERVAL 3 MONTH)
GROUP BY d.DriverID
HAVING AVG(td.DriverRating) < 3.0;

-- 2 Find the top 5 drivers who have completed the longest trips in terms of distance.

SELECT  d.DriverID, d.Name,
       SUM(td.Distance) AS total_distance_km,
       COUNT(td.TripID) AS trips_completed
FROM Drivers d
JOIN Cabs c ON d.DriverID = c.DriverID
JOIN Bookings b ON c.CabID = b.CabID
JOIN TripDetails td ON b.BookingID = td.BookingID
WHERE b.Status = 'Completed'
GROUP BY d.DriverID, d.Name
ORDER BY total_distance_km DESC
LIMIT 5;

# Identify drivers with a high percentage of canceled trips.
SELECT d.DriverID, d.Name,
COUNT(b.BookingID) AS total_bookings,
SUM(CASE WHEN b.Status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_count,
ROUND(100.0 * SUM(CASE WHEN b.Status = 'Cancelled' THEN 1 ELSE 0 END) / NULLIF(COUNT(b.BookingID),0),2) AS cancel_pct
FROM Drivers d
JOIN Cabs c ON d.DriverID = c.DriverID
JOIN Bookings b ON c.CabID = b.CabID
GROUP BY d.DriverID, d.Name
HAVING total_bookings > 0
ORDER BY cancel_pct DESC;

# Revenue & Business Metrics
 # 3 Calculate the total revenue generated by completed bookings in the last 6 months.
 SELECT SUM(td.Fare) AS total_revenue_last_6_months
FROM TripDetails td
JOIN Bookings b ON td.BookingID = b.BookingID
WHERE b.Status = 'Completed'
  AND b.TripStartTime >= DATE_SUB(CURRENT_DATE(), INTERVAL 6 MONTH);
  
  #Identify the top 3 most frequently traveled routes based on PickupLocation and
# DropoffLocation.
SELECT b.PickupLocation,
       b.DropoffLocation,
       COUNT(*) AS route_count
FROM Bookings b
WHERE b.Status = 'Completed' OR b.Status = 'Cancelled' 
GROUP BY b.PickupLocation, b.DropoffLocation
ORDER BY route_count DESC
LIMIT 3;

## Determine if higher-rated drivers tend to complete more trips and earn higher fares.
SELECT d.DriverID, d.Name,
       AVG(td.DriverRating) AS avg_rating,
       COUNT(td.TripID) AS trips_completed,
       SUM(td.Fare) AS total_earnings,
       ROUND(SUM(td.Fare) / NULLIF(COUNT(td.TripID),0),2) AS avg_fare_per_trip
FROM Drivers d
JOIN Cabs c ON d.DriverID = c.DriverID
JOIN Bookings b ON c.CabID = b.CabID
JOIN TripDetails td ON b.BookingID = td.BookingID
WHERE b.Status = 'Completed'
GROUP BY d.DriverID, d.Name
ORDER BY avg_rating DESC;

-- Analyze the average waiting time (difference between booking time and trip start
-- time) for different pickup locations.
SELECT b.PickupLocation,
       COUNT(*) AS trips_count,
       ROUND(AVG(TIMESTAMPDIFF(MINUTE, b.BookingTime, b.TripStartTime)),2) AS avg_wait_minutes
FROM Bookings b
WHERE b.TripStartTime IS NOT NULL
GROUP BY b.PickupLocation
ORDER BY avg_wait_minutes DESC;




 









`
