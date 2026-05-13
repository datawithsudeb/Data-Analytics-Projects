# Uber Trip Data Analysis

### *A Comprehensive Operational & Revenue Intelligence Case Study for Ride-Hailing Analytics*

**Created By:** Sudeb Paul

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Business Objective](#2-business-objective)
3. [Tools & Technologies](#3-tools--technologies)
4. [Dataset Overview](#4-dataset-overview)
5. [Data Cleaning & Preparation](#5-data-cleaning--preparation)
6. [Feature Engineering](#6-feature-engineering)
7. [Driver-Level Aggregation Logic](#7-driver-level-aggregation-logic)
8. [Rider-Level Aggregation Logic](#8-rider-level-aggregation-logic)
9. [Dashboard Architecture](#9-dashboard-architecture)
10. [Trip Overview Dashboard](#10-trip-overview-dashboard)
11. [Driver Performance Dashboard](#11-driver-performance-dashboard)
12. [Rider Behavior Dashboard](#12-rider-behavior-dashboard)
13. [Revenue & Payment Dashboard](#13-revenue--payment-dashboard)
14. [Root Cause Dashboard](#14-root-cause-dashboard)
15. [Executive-Level Business Insights](#15-executive-level-business-insights)
16. [Strategic Recommendations](#16-strategic-recommendations)
17. [Business Impact Summary](#17-business-impact-summary)
18. [Conclusion](#18-conclusion)

---

# 1. Introduction

Ride-hailing platforms generate millions of trip-level records every day. Hidden inside these transactions are operational inefficiencies, rider behavior patterns, revenue opportunities, and driver performance trends that directly influence business growth.

This project focuses on building a complete end-to-end analytics solution for Uber trip operations using spreadsheet-based analytics and interactive dashboards. The goal was not only to visualize the data but also to uncover the business story behind the numbers.

Using 50,000 Uber trip records, this analysis explores:

* How efficiently trips are completed
* Which drivers contribute the most value
* Why cancellations happen
* How rider retention impacts revenue
* Which operational areas are leaking revenue
* What strategies can improve platform performance

The entire solution was developed using Google Sheets for data transformation and Google Looker Studio for dashboarding and storytelling.

---

# 2. Business Objective

The primary objective of this project was to transform raw trip-level transactional data into actionable business intelligence that can support operational and strategic decision-making.

The analysis aims to answer the following business questions:

### Operational Questions

* What is the overall trip completion rate?
* Which cities generate the highest trip demand?
* During which hours does demand peak?
* What factors contribute to cancellations and no-shows?

### Driver Performance Questions

* Which drivers generate the highest revenue?
* Are drivers being efficiently utilized?
* Which drivers exhibit abnormal cancellation behavior?

### Rider Behavior Questions

* Are riders retained after their first trip?
* Who are the highest-value riders?
* How concentrated is revenue among repeat riders?

### Revenue Questions

* Which payment methods generate the most revenue?
* How much revenue is being lost due to failed trips?
* Does trip distance strongly influence fare generation?

---

# 3. Tools & Technologies

| Tool                 | Purpose                                                         |
| -------------------- | --------------------------------------------------------------- |
| Google Sheets        | Data cleaning, transformation, aggregation, feature engineering |
| Google Looker Studio | Dashboard development and data visualization                    |

---

# 4. Dataset Overview

The project begins with a raw transactional dataset containing 50,000 Uber trips. Each row represents a single ride transaction captured across multiple cities.

The dataset contains operational, financial, geographic, and behavioral attributes.

## Raw Dataset Structure

The primary `uber_trips` sheet contains the following raw attributes:
 
| Column Name      | Data Type  | Description                                              |
| :--------------- | :--------- | :------------------------------------------------------- |
| `trip_id`        | `INT`      | Unique identifier for each trip (Primary Key)            |
| `driver_id`      | `INT`      | Unique identifier for the assigned driver                |
| `rider_id`       | `INT`      | Unique identifier for the rider                          |
| `city`           | `STRING`   | City where the trip took place                           |
| `pickup_lat`     | `FLOAT`    | Latitude coordinate of the pickup location               |
| `pickup_lng`     | `FLOAT`    | Longitude coordinate of the pickup location              |
| `drop_lat`       | `FLOAT`    | Latitude coordinate of the drop-off location             |
| `drop_lng`       | `FLOAT`    | Longitude coordinate of the drop-off location            |
| `distance_km`    | `FLOAT`    | Distance of the trip in kilometres                       |
| `fare_amount`    | `FLOAT`    | Fare charged for the trip                                |
| `status`         | `STRING`   | Raw trip status (Completed / Cancelled / No-Show)        |
| `payment_method` | `STRING`   | Mode of payment used (Card / Cash / UPI / Wallet)        |
| `pickup_time`    | `DATETIME` | Timestamp when the rider was picked up                   |
| `drop_time`      | `DATETIME` | Timestamp when the rider was dropped off                 |

---

# 5. Data Cleaning & Preparation

Before analysis, the dataset underwent a structured cleaning and transformation process in Google Sheets.

## Data Cleaning Activities

### Timestamp Standardization

Pickup and drop timestamps were standardized to ensure consistency across all calculations.

### Missing Value Validation

Trips containing invalid timestamps or incomplete trip information were identified and checked before dashboard integration.

### Duplicate Verification

The `trip_id` column was validated to ensure every trip remained unique.

### Numeric Formatting

Distance, fare, duration, and revenue fields were converted into proper numeric formats for accurate aggregation.

### Status Standardization

Trip statuses were standardized into three consistent operational categories:

* Completed
* Cancelled
* No-Show

### Date Hierarchy Extraction

Date-related fields such as year, month, day, and hour were extracted to support time-series analysis and dashboard filtering.

---

# 6. Feature Engineering

To enrich the raw transactional dataset, several derived columns were created using spreadsheet formulas.

These engineered features became the analytical foundation of the dashboards.

## Feature Engineering Logic

All feature engineering was performed in **Google Sheets** using formulas to enrich the raw dataset with calculated columns. The final enriched sheet structure is as follows:
 
| Column Name          | Formula / Method                                                                                                                    | Description                                              |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------- |
| `pickup_date`        | `=INT(pickup_time)`                                                                                                                 | Date portion extracted from pickup timestamp             |
| `year`               | `=YEAR(pickup_date)`                                                                                                                | Year of the trip                                         |
| `month`              | `=MONTH(pickup_date)`                                                                                                               | Month of the trip (numeric)                              |
| `day`                | `=DAY(pickup_date)`                                                                                                                 | Day of the month                                         |
| `hour`               | `=HOUR(pickup_time)`                                                                                                                | Hour of pickup (0–23)                                    |
| `time_bucket`        | `=IFS(hour<6,"Night", hour<12,"Morning", hour<18,"Afternoon", TRUE,"Evening")`                                                      | Categorized time-of-day segment                          |
| `trip_duration_mint` | `=(drop_time - pickup_time) * 1440`                                                                                                 | Trip duration in minutes                                 |
| `speed_kmph`         | `=distance_km / (trip_duration_mint / 60)`                                                                                          | Average speed in km/h                                    |
| `completed_flag`     | `=IF(status="Completed", 1, 0)`                                                                                                     | Binary flag for completed trips                          |
| `cancelled_flag`     | `=IF(status="Cancelled", 1, 0)`                                                                                                     | Binary flag for cancelled trips                          |
| `No-Show_flag`       | `=IF(status="No-Show", 1, 0)`                                                                                                       | Binary flag for no-show trips                            |
| `Revenue`            | `=IF(status="Completed", fare_amount, 0)`                                                                                           | Actual revenue earned (only for completed trips)         |
| `revenue_per_km`     | `=IF(distance_km>0, Revenue/distance_km, 0)`                                                                                        | Revenue generated per kilometre                          |
| `distance_bucket`    | `=IFS(distance_km<3,"Short (0-3 km)", distance_km<7,"Medium (3-7 km)", distance_km<15,"Long (7-15 km)", TRUE,"Very Long (15+ km)")` | Categorized distance segment                             |
| `fare_per_km`        | `=IF(distance_km>0, fare_amount/distance_km, 0)`                                                                                    | Fare charged per kilometre                               |
| `trip_count`         | `=COUNTIF(driver_id_range, driver_id)`                                                                                              | Total trips attributed to the driver                     |
| `lost_revenue`       | `=IF(status<>"Completed", fare_amount, 0)`                                                                                          | Potential revenue lost due to cancellations and no-shows |

---

# 7. Driver-Level Aggregation Logic

To analyze driver efficiency and operational reliability, a dedicated driver summary table was created.

Each driver was evaluated using:

* Trip volume
* Completion rate
* Cancellation rate
* Revenue contribution
* Average fare
* Average distance

This layer enabled the identification of:

* High-performing drivers
* Underutilized drivers
* High-cancellation drivers
* Revenue concentration among drivers

---

# 8. Rider-Level Aggregation Logic

A rider-focused analytical layer was created to understand customer engagement and monetization patterns.

This sheet tracks:

* Rider frequency
* Revenue contribution
* Average trip behavior
* Cancellation tendencies
* Repeat usage

The objective was to identify:

* High-value riders
* One-time users
* Revenue concentration
* Retention opportunities

---

# 9. Dashboard Architecture

The dashboard solution was divided into five business-focused pages.

Each page answers a specific operational or strategic business question.

| Dashboard Page      | Business Focus               |
| ------------------- | ---------------------------- |
| Trip Overview       | Platform performance         |
| Driver Performance  | Driver productivity          |
| Rider Behavior      | Rider engagement & retention |
| Revenue & Payment   | Revenue intelligence         |
| Root Cause Analysis | Cancellation investigation   |

The dashboards were connected live to Google Sheets, enabling dynamic filtering and real-time interaction.

---

# 10. Trip Overview Dashboard

## Business Story

The Trip Overview Dashboard acts as the operational heartbeat of the platform.
![Trip Overview Dashboard](https://github.com/datawithsudeb/Data-Analytics-Projects/blob/main/Uber%20Trip%20Data%20Analysis/Trip%20Overview%20Dashboard.png)

At first glance, the business appears healthy:

* 50,000 total trips
* 85.08% completion rate
* ₹679.7k revenue generated

However, deeper analysis reveals a more nuanced operational story.

Demand is highly concentrated during evening hours between 3 PM and 9 PM, suggesting strong commuter and post-work travel demand. Cities such as Boston, San Francisco, and Chicago slightly outperform others in total trip volume, though overall city contribution remains balanced.

The trip trend line also shows a visible drop toward the end of the analysis period, potentially indicating:

* Supply shortages
* Driver inactivity
* Seasonal fluctuations
* External disruptions

## Key Operational Insights

* Platform completion rate remains strong at 85.08%.
* Approximately 15% of potential trips fail due to cancellations and no-shows.
* Evening hours represent the platform's most critical operational window.
* Revenue trends closely mirror trip volume trends.
* Average trip duration of 21 minutes indicates moderate urban travel density.

---

# 11. Driver Performance Dashboard

## Business Story

The Driver Performance Dashboard reveals one of the most important operational findings in the project:
![Driver Performance Dashboard](https://github.com/datawithsudeb/Data-Analytics-Projects/blob/main/Uber%20Trip%20Data%20Analysis/Driver%20Perormance%20Dashboard.png)

The platform has a large driver base, but most drivers are significantly underutilized.

Although there are 8,963 active drivers, the average driver completes only 6 trips. This suggests:

* Excess supply
* Low driver engagement
* Inefficient trip allocation
* Poor retention of active drivers

Another critical insight emerges from cancellation behavior.

A very small group of drivers contributes disproportionately high cancellation rates. While most drivers remain within healthy operational limits, a few fall into the 70–90% and >90% cancellation buckets.

These drivers directly damage:

* Rider trust
* Platform reliability
* Revenue realization

## Key Driver Insights

* Revenue contribution is concentrated among a small set of high-performing drivers.
* Most drivers operate at very low utilization levels.
* High-cancellation drivers represent operational risk.
* Mid-tier drivers represent the largest growth opportunity for supply optimization.

---

# 12. Rider Behavior Dashboard

## Business Story

The Rider Dashboard tells a retention problem story.
![Rider Behavior Dashboard](https://github.com/datawithsudeb/Data-Analytics-Projects/blob/main/Uber%20Trip%20Data%20Analysis/Rider%20Behavior%20Dashboard.png)

The platform is highly effective at acquiring users but struggles to retain them.

Out of 38,352 riders:

* 28,684 riders completed only one trip
* Average trips per rider is only 1.11
* Repeat rider rate is just 25.2%

This means the majority of riders leave after their first experience.

However, the small group of repeat riders contributes disproportionately higher revenue. Riders completing 5–6 trips generate far greater lifetime value compared to one-time users.

This reveals a major business opportunity:
The platform’s fastest growth lever may not be customer acquisition — it may be customer retention.

## Key Rider Insights

* One-time riders dominate the platform.
* Repeat riders are extremely valuable.
* Revenue is concentrated in low-spend buckets.
* Retention performance is currently weak.
* No-show behavior remains manageable but still impacts operations.

---

# 13. Revenue & Payment Dashboard

## Business Story

The Revenue Dashboard highlights both platform strength and revenue leakage.
![Revenue & Payment Dashboard](https://github.com/datawithsudeb/Data-Analytics-Projects/blob/main/Uber%20Trip%20Data%20Analysis/Revenue%20%26%20Payment%20Dashboard.png)

On one side:

* Revenue generation remains stable
* All cities contribute consistently
* Payment distribution is balanced

On the other side:

* ₹119k revenue is lost due to cancellations and no-shows
* Nearly 15% of potential revenue never materializes

This represents one of the largest optimization opportunities in the business.

Interestingly, payment methods show almost identical revenue contributions, indicating:

* Strong digital payment adoption
* Healthy diversification
* No dependency on a single payment channel

The scatter analysis also confirms that distance is a major revenue driver. Longer trips consistently produce higher fares, reinforcing the importance of improving long-distance trip completion.

## Key Revenue Insights

* Revenue generation is operationally stable.
* Lost revenue remains significantly high.
* Distance strongly influences monetization.
* Payment behavior is evenly distributed across methods.
* City-level revenue distribution is balanced.

---

# 14. Root Cause Dashboard

## Business Story

The Root Cause Dashboard uncovers the operational drivers behind failed trips.
![Root Cause Dashboard](https://github.com/datawithsudeb/Data-Analytics-Projects/blob/main/Uber%20Trip%20Data%20Analysis/Root%20Cause%20Dashboard.png)

The most important discovery:
Long-distance trips (7–15 km) contribute nearly 50% of all cancellations.

This strongly suggests:

* Driver reluctance toward long-distance routes
* Fare dissatisfaction
* Increased rider cancellations on expensive trips
* Matching inefficiencies for medium-long journeys

Interestingly, cancellation rates remain consistent across:

* Cities
* Time periods
* Payment methods

This indicates the issue is systemic rather than localized.

The platform-wide nature of cancellations suggests that operational policies — not geography — are driving the problem.

## Key Root Cause Insights

* Cancellation rate (~10%) is the primary source of operational leakage.
* Long-distance trips are disproportionately vulnerable.
* Cancellation behavior is consistent across cities.
* Payment method has minimal impact on trip success.
* Platform-wide operational improvements are required.

---

# 15. Executive-Level Business Insights

## 1. The Platform Has Strong Demand but Weak Retention

Demand acquisition is healthy, but rider retention remains low. Most users leave after a single trip.

## 2. Revenue Leakage Is a Major Business Problem

Nearly ₹119k in potential revenue is lost due to failed trips.

## 3. Driver Supply Exists but Is Poorly Utilized

The platform has enough drivers, but activity levels remain extremely low.

## 4. Long-Distance Trips Are Operationally Fragile

The majority of cancellations occur in the 7–15 km segment.

## 5. Evening Hours Represent the Most Critical Revenue Window

Peak demand occurs between 3 PM and 9 PM.

## 6. High-Value Riders Are Extremely Important

A very small repeat-rider segment contributes disproportionately high value.

---

# 16. Strategic Recommendations

## 1. Reduce Long-Distance Trip Cancellations

Introduce:

* Distance-based incentives
* Better driver-trip matching
* Upfront pricing transparency
* Long-route bonus programs

Expected Impact:

* Reduced cancellation rate
* Higher realized revenue
* Improved rider satisfaction

---

## 2. Improve Driver Utilization

Launch:

* Trip milestone incentives
* Peak-hour bonuses
* Driver reactivation campaigns

Expected Impact:

* Increased trip supply
* Higher platform efficiency
* Better service availability

---

## 3. Build a Rider Retention Strategy

Implement:

* Loyalty rewards
* Personalized offers
* First-repeat-trip discounts
* Rider subscription models

Expected Impact:

* Increased repeat rate
* Higher lifetime value
* Reduced acquisition dependency

---

## 4. Optimize Peak-Hour Operations

Focus operational efforts during:

* 3 PM – 9 PM demand window

Actions:

* Surge incentives
* Smart driver positioning
* Dynamic pricing optimization

Expected Impact:

* Higher completion rates
* Faster pickups
* Increased revenue capture

---

## 5. Proactively Manage High-Cancellation Drivers

Introduce:

* Performance monitoring
* Driver coaching
* Temporary restrictions for repeat offenders

Expected Impact:

* Improved platform reliability
* Better rider trust
* Lower operational leakage

---

## 6. Recover Lost Revenue

Target:

* ₹119k unrealized revenue opportunity

Methods:

* Reduce cancellations
* Improve matching accuracy
* Encourage completion of medium-long routes

Expected Impact:

* Immediate revenue recovery
* Improved operational margins

---

# 17. Business Impact Summary

| Business Area | Key Finding            | Potential Impact              |
| ------------- | ---------------------- | ----------------------------- |
| Operations    | 15% trip failure rate  | Revenue leakage               |
| Drivers       | Underutilized supply   | Efficiency opportunity        |
| Riders        | Weak retention         | Growth limitation             |
| Revenue       | ₹119k lost revenue     | Monetization opportunity      |
| Demand        | Strong evening peaks   | Peak optimization opportunity |
| Long Trips    | High cancellation risk | Operational redesign required |

---

# 18. Conclusion

This project demonstrates how transactional ride-hailing data can be transformed into meaningful operational intelligence using spreadsheet analytics and dashboard storytelling.

Beyond visualizations, the analysis uncovers critical business realities:

* Retention is weak
* Driver utilization is inefficient
* Long-distance cancellations are hurting revenue
* High-value riders remain underleveraged

By addressing these areas strategically, the platform can:

* Improve operational efficiency
* Increase rider loyalty
* Recover lost revenue
* Strengthen platform reliability
* Drive long-term sustainable growth

This project showcases not just dashboard development skills, but also business thinking, analytical storytelling, KPI design, and operational problem-solving using real-world ride-hailing analytics.
