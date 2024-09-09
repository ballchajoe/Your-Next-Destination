## Capstone Project: Take Flight
=========================

## Project Overview
One area of interest for me is transportation, especially for either airline or automotive industry. A challenge an airline could potentially face is flight and fleet logistics. If airlines can know in advance which destinations will be popular in the upcoming month, they can adjust their flight schedules and fleet maintenance to increase or decrease flights to certain destinations. With this logistical knowledge, profit can be made not only from airline ticket sales but also saving from overhead and unneeded flight maintenance. To start off, my model will forecast which destinations will be in the top 10 on a monthly basis for the next 24 months but could be adjusted to greater than 10.

## Original Data Dictionary
* DEPARTURES_PERFORMED: Completed scheduled flights
* SEATS: Total number of seats of the flights
* PASSENGERS: Total number of passengers on the flights
* CARRIER: Airline name abbreviated
* CARRIER_NAME: Designated airline name
* ORIGIN_AIRPORT_ID: Flight's origin airport ID
* ORIGIN_CITY_NAME: Flight's origin city
* ORIGIN_COUNTRY_NAME: Flight's origin country
* DEST_AIRPORT_ID: Flight's destination airpot ID
* DEST_CITY_NAME: Flight's destination city
* DEST_COUNTRY_NAME: Flight's destination country
* YEAR: Year flight performed
* MONTH: Month flight performed
* DATE: Year-Month-Day

## Data Dictionary for Model
* Datetime index ranging from 2013-01-01 to 2024-05-01
* Destinations:
    * Cities flown to with more than 5000 passengers per month
    * Also missing less than 24 months of datess 

## Project Organization

### Repository 

* `data` 
    - link TBD

* `model`
    - TBD

* `notebooks`
    - 01-data-loading-cleaning
    - 02-eda
    - 03-pre-processing
    - 04-modelling
    - 05-findings

* `docs`
    - Your Next Destination_Sprint1.pdf
    - Your Next Destination_Sprint2.pdf

* `references`
    - L_AIRPORT_ID.csv
    - L_CARRIER_HISTORY.csv

* `src`
    - TBD

* `.gitignore`
    - Part of Git, includes files and folders to be ignored by Git version control

* `capstone_joe_cha.yml`
    - Conda environment specification

* `README.md`
    - Project landing page (this page)

* `LICENSE`
    - Project license