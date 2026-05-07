# Customer 360 Analytics Pipeline

### Identity Resolution · RFM Segmentation · CLTV Modeling · Python → MySQL → Tableau

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-orange?logo=mysql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-Public-E87722?logo=tableau&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.3-F7931E?logo=scikitlearn&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-1D9E75)

---


## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution Overview](#2-solution-overview)
3. [Pipeline Architecture](#3-pipeline-architecture)
4. [Tech Stack](#4-tech-stack)
5. [Project Structure](#5-project-structure)
6. [Step-by-Step Walkthrough](#6-step-by-step-walkthrough)
   - [Layer 1 — Python](#layer-1--python)
   - [Layer 2 — MySQL](#layer-2--mysql)
   - [Layer 3 — Tableau](#layer-3--tableau)
7. [Key Results](#7-key-results)
8. [Business Insights](#8-business-insights)
9. [Skills Gained](#9-skills-gained)
10. [How to Run](#10-how-to-run)

---

## 1. Problem Statement

Large enterprises operating across web, mobile, in-store, and CRM ecosystems accumulate fragmented customer data across disconnected systems. As a result, the same customer often exists as multiple records with inconsistent identifiers such as names, email formats, or phone numbers.
This fragmentation creates three major business challenges:

**1. Inflated Customer Counts
Duplicate customer records lead to repeated marketing communication, increased acquisition costs, and poor customer experience.

**3. Incomplete Customer Profiles
Without a unified Customer360 view, organizations cannot accurately measure customer lifetime value, purchase behaviour, engagement patterns, or churn risk.

**3. Limited Personalization & Segmentation
Disconnected data prevents CRM and product teams from identifying high-value customers, optimizing cross-sell opportunities, and delivering personalized engagement at scale.

>**Enterprise CRM systems commonly experience duplicate rates between 10–30%, significantly impacting downstream analytics, campaign targeting, and business decision-making.
