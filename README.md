# 🛰️ **Case Study: Post-Flood Recovery Analysis (Feni 2024)**
### **Integrating SAR Hydrology with NTL Socio-Economic Indicators**

---

## **1. Executive Summary**
This research evaluates the 2024 flash flood event in Feni, Bangladesh. By bridging **physical remote sensing** (Sentinel-1 SAR) with **human activity proxies** (VIIRS Night-time Lights), we examine how economic wealth and population density influence the speed of infrastructural recovery.

> **Important links:** 

*   [GEE script](https://code.earthengine.google.com/3b1c33d68d9bd615b2a9c5f4802c5372)
*   [Case study report pdf](https://github.com/Ashik-Abdullah-Chowdhury/Cohort-2_Assignment-9/blob/main/Case_Study_Report.pdf)
*   [Flood extent map](https://github.com/Ashik-Abdullah-Chowdhury/Cohort-2_Assignment-9/blob/main/feni%20fw%20copy.png)
*   [Study area map](https://github.com/Ashik-Abdullah-Chowdhury/Cohort-2_Assignment-9/blob/main/Study_Area(Feni).png)
*   [Integrated Analysis of Flood Extent, NTL Recovery, and Socio-Economic Indicators](https://github.com/Ashik-Abdullah-Chowdhury/Cohort-2_Assignment-9/blob/main/ntlrecovery.png)


---

## **2. Methodology & Modeling**

### **2.1 Flood Mapping Workflow**
* **Active Sensor:** Sentinel-1 SAR (VH) for all-weather detection.
* **Refinement:** Topographic filtering (<50m elevation) and NDWI-based permanent water masking to isolate "event-specific" inundation.

### **2.2 NTL Recovery Index ($Y_{rec}$)**
We formulated an index to normalize recovery against the pre-flood baseline:
$$Y_{rec} = \frac{NTL_{Recovery} - NTL_{Post}}{NTL_{Pre} + 0.1}$$

### **2.3 Multivariate Regression Model**
The drivers of regional resilience were analyzed using the following linear model:
$$Y_{rec} = \beta_0 + \beta_1(PopDensity) + \beta_2(WealthIndex) + \epsilon$$

---

## **3. Empirical Results**

The multivariate regression yielded the following relationship:
> **$Y_{rec} = 0.64 - 0.017(PopDensity) + 0.35(WealthIndex)$**

| Variable | Coefficient ($\beta$) | Impact on Recovery |
| :--- | :---: | :--- |
| **Intercept ($\beta_0$)** | $0.64$ | Baseline recovery trend |
| **Population Density ($\beta_1$)** | $-0.017$ | Weak negative  |
| **Wealth Index ($\beta_2$)** | $0.35$ | Strong positive  |

---

## **4. Core Findings**
* **Wealth as a Catalyst:** The strong positive correlation with wealth indicates that financial liquidity and decentralized power resources are the primary predictors of rapid recovery.
* **Density Vulnerability:** High-density zones exhibit a "recovery lag" due to complex grid damage and prolonged waterlogging in impervious urban environments.

---

## **5. Technical Limitations**
* **Sub-canopy Flooding:** Possible underestimation in forested patches (SAR limitation).
* **Spatial Decoupling:** NTL radiance captures commercial hubs better than residential-only population clusters.
* **Displacement:** Static population data cannot capture real-time human migration during the event.

---
**Author:** Ashik Abdullah Chowdhury  
*B.Sc (Hon's) in Forestry, University of Chittagong*
