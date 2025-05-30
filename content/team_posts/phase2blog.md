---
title: "Project - Phase II"
date: 2025-05-27
draft: false
description: "Proof of Concept - Gathering Data and Designing"
slug: "phase2post"
tags: ["project", "Setup"]
authors:
  - "katherineahn"
  - "anoushkaabroal"
  - "arthurhuang"
  - "shivenajwaliya"
showAuthorsBadges: false
---
# Project Updates
We began this phase by revisiting and refining our user archetypes, making each persona more specific and realistic. We added names, contextual details, and motivations that better aligned with the updated features of our app. For example, our original traveler persona was reframed as Archibald Bridgemont, a parent relocating to Europe with his family. We expanded his feature set to include not only healthcare priority-based matching, but also an optional comparison to his current country's system—blending both familiarity and preference in the results. Similarly, Nancy the policymaker was rewritten to focus less on a single country and more on globally informed, evidence-based reform, reflecting our system’s international comparison and forecasting capabilities. Gale, the student, was also given more context and academic grounding, reinforcing his need for tools like time-series charts and visual comparisons.

These refined personas helped us anchor our technical decisions to real, use-driven goals. Updates to the full set of personas can be viewed in our [Phase I blog post](https://arthur-t-huang.github.io/Care-Compass-Blog/team_posts/phase1post/)

We also went back and articulated our data sources better, and found a new, more comprehensive data set as well, that changed some of the technicalities we wanted in our inital plan. Due to this, we also changed those concepts to match our new vision in our first blog post. 

This shift in data informed several important technical updates. Most notably, we moved away from our original idea of allowing users to assign custom weights to healthcare factors and instead implemented a k-nearest neighbors (k-NN) model to generate personalized country recommendations based on user priorities. We also introduced a time series forecasting model that enables users to view historical trends and predict future values for key healthcare indicators.

Together, these updates helped us move from a loosely scoped concept to a more concrete, implementable system with clearly defined features tied directly to our personas.

# Proof of Concept ML Model: 
We began exploring a linear regression model that we will continue to shape in Phase III. For this example, we standardized the Antimicrobial feature, as well as the Health Index scores, in order to create a line of best fit regression plot, and a residual graph as well.
We will most likely swap out the features based on what the user is asking (see wireframes), but we decided to explore these two features with our regression model for now. The results can be seen below: 

Line of Best Fit:
![image](LineOfBestFitProof.png)

Residual Plot:
![image](residualPlotProof.png)


# Data Sources:
At the end of Phase I, we had gathered data from two datasets, households impoverished by out of pocket payments, and general practitioners per 10,000 population. For Phase II, in addition to cleaning and visualizing these two datasets, we have also gathered, cleaned, and graphed out data from 9 new datasets, as updated in our [blog post for phase 1](https://arthur-t-huang.github.io/Care-Compass-Blog/team_posts/phase1post/#data-sources). 8 of these new datasets were pulled from the World Health Organization API, the other dataset web scraped from the Global Health Security Index, to give us that overall score we were looking for. For each of the World of Health datasets, the gatherer information was fitted into a dataframe which had distinct country and date columns, since every W.H.O dataset contained these two values. Additionally, the Global Health Index dataset contained 300+ columns of factors which were used to help calculate the overall score for a country (ex. Antimicrobial resistance, Access to potable water, etc). Some of these factors were then visualized in comparison to the the overall score of a country, and we plan on including factors with high correlation to overall score to the slider feature mentioned in the [blog post for phase 1](https://arthur-t-huang.github.io/Care-Compass-Blog/team_posts/phase1post/#data-sources).
We have visualized a couple of the datasets in different ways as they relate to the wireframes that we plan to implement: 
We have visualized a couple of the datasets in different ways as they relate to the wireframes that we plan to implement: 

## GHS Index Dataframe:
![image](Health_Score_Dataframe.png)

## Barchart:
![image](Healthscore_Barchart.png)

## Line Graphs:
![image](Expenditure_Line_Graph.png)
![image](impoverished_linegraph.png)

## Scatter Plots:
![image](Practitioners_Expenditure_Scatterplot.png)
![image](Antimicrobial_Resistance_Scatterplot.png)





# ER Diagrams, Relational Diagram, SQL DDL:

Below are localized ER diagrams for each personna: the green diagram corresponding to the mover Archibald, red diagram corresponding to the student Gale, and the blue diagram corresponding to the policymaker Nancy.

![image](moverER.png)
![image](studentER.png)
![image](policymakerER.png)

Further, the global ER model is seen here:

![image](globalER.png)

The relational diagram was further derived as:
![image](relationalDiagram.png)

## SQL DDL:
Based on our global data model, a first pass of our SQL DDL can be seen below.

<pre> 
DROP DATABASE IF EXISTS carecompass_database;
CREATE DATABASE carecompass_database;
USE carecompass_database;

-- create country table
DROP TABLE IF EXISTS country;
CREATE TABLE country
(
    id   INT PRIMARY KEY,
    name        VARCHAR(50),
    region      VARCHAR(50),
    strengths   VARCHAR(50),
    weaknesses  VARCHAR(50),
    score       FLOAT,
    info        VARCHAR(50),
    time        DATETIME
);

-- create users table
DROP TABLE IF EXISTS users;
CREATE TABLE users
(
    id               INT PRIMARY KEY,
    name             VARCHAR(50),
    userType         VARCHAR(50),
    qualityWeight    FLOAT,
    accessibilityWeight FLOAT,
    affordabilityWeight FLOAT,
    outcomeWeight       FLOAT,
    countryID       INT,
    FOREIGN KEY (countryID) REFERENCES country (id)
);

-- create score projection table
DROP TABLE IF EXISTS score_project;
CREATE TABLE score_project
(
    time    DATETIME,
    targetScore     FLOAT primary key
);


-- create overall score table
DROP TABLE IF EXISTS overall_score;
CREATE TABLE overall_score
(
    overallScore FLOAT PRIMARY KEY,
    qualityScore FLOAT,
    accessibilityScore FLOAT,
    affordabilityScore FLOAT,
    outcomeScore FLOAT
);

-- create factors table
DROP TABLE IF EXISTS factors;
CREATE TABLE factors
(
    factorID    INT PRIMARY KEY,
    name        VARCHAR(50),
    score       FLOAT,
    weight      FLOAT,
    countryID   INT,
    overallScore   FLOAT,
    targetScore     FLOAT,
    FOREIGN KEY (countryID) REFERENCES country(id),
    FOREIGN KEY (overallScore) REFERENCES overall_score(overallScore),
    FOREIGN KEY (targetScore) REFERENCES score_project(targetScore)
);


-- create comparator table
DROP TABLE IF EXISTS comparator;
CREATE TABLE comparator
(
    country1ID  INT,
    country2ID  INT,
    country3ID INT,
    FOREIGN KEY (country1ID) REFERENCES country(id),
    FOREIGN KEY (country2ID) REFERENCES country(id),
    FOREIGN KEY (country3ID) REFERENCES country(id)
);

-- create country_compare table
DROP TABLE IF EXISTS country_compare;
CREATE TABLE country_compare
(
    countryID   INT,
    Country1_ID   INT,
    Country2_ID    INT,
    Country3_ID   INT,
    FOREIGN KEY (countryID) REFERENCES country(id)

);

-- create user_score table
DROP TABLE IF EXISTS user_score;
CREATE TABLE user_score
(
    userID   INT,
    overallScore    FLOAT,
    FOREIGN KEY (userID) REFERENCES users(id),
    FOREIGN KEY (overallScore) REFERENCES overall_score(overallScore)
);
 </pre>

 # WireFrames:

Our current wireframes can be viewed here: 

![image](wfHomepage.png)
This wireframe shows the homepage of CareCompass. It introduces the different user archetypes, and the application as a whole.

![image](wfCountryComparator.png)
The _Country Comparator_ page is utilized by the student archetype where they are given the opportunity to choose 2-3 countries to compare 6 features' scores against one another in an organized table. Further, the student may choose to track a specific feature over time, given by a dropdown menu and visualized in a line plot (with different countries in different colors).

![image](wfStrengthsWeakness.png)
The _Strengths vs. Weaknesses_ page is also used by the student to look at one specific country and identify its strengths and weaknesses. The strengths and weaknesses are determined by features with the highest/lowest scores relative to all other countries in our datasets.

![image](wfSliders.png)
The _Sliders_ page illustrates a personalized and customizable aspect to our application, particularly geared toward a moving resident. There are 6 factors that may be considered and the sliders allow users to prioritize some over others. The result of using the sliders is a bar chart that visualizes countries' scores based on the weights provided by the user (sorted from highest score to lowest), as well as a gradient map of the scores. An additional feature of this page is for individuals to choose what country they are coming from, and whether or not they would like that country score to factor into their results.

![image](wfCountryProfile.png)
The _Country Profile_ is a dashboard that can be reached on its own or through links on other pages. This page is currently housed for residents to learn more about countries of interest to them. Additionally, users may see on the right side bar provided links to country profiles of similar score to that of the country of interest (country of which profile currently on).

![image](wfRegression.png)
The _Regression_ page may be utilized by the policymaker archetype as they consider how a given country's score may change over time depending on the region, certain factors, and time. 

![image](wfTargetScores.png)
The _Target Scores_ page illustrates a policymaker's ability to utilize CareCompass in a way to project their country's growth. Given the current 6 scores of a country, the user may input target scores and the projected time until those target scores are reached will be output in years (if applicable).



