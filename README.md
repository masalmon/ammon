[![Build Status](https://travis-ci.org/masalmon/ammon.svg?branch=master)](https://travis-ci.org/masalmon/ammon) [![Build status](https://ci.appveyor.com/api/projects/status/6a9mh4llv8uew4xx?svg=true)](https://ci.appveyor.com/project/masalmon/ammon) [![codecov.io](https://codecov.io/github/masalmon/ammon/coverage.svg?branch=master)](https://codecov.io/github/masalmon/ammon?branch=master)

Please note that this package is under development.

Installation
============

``` r
library("devtools")
install_github("masalmon/ammon", build_vignettes=TRUE)
```

Introduction
============

This package aims at supporting the analysis of PM2.5 measures made with RTI MicroPEM. It is called ammon like Zeus Ammon (<https://en.wikipedia.org/wiki/Amun#Greece> ) because it helps us to Analyse Micropem MONitoring data in a very good, nearly godly, way.

The goal of the package functions is to help in two main tasks:

-   Checking individual MicroPEM output files after, say, one day of data collection.

-   Building a data base based on output files, and clean and transform the data for further analysis.

For the examination of individual files, the package provides a function for transforming the output of a RTI MicroPEM into an object of a R6 class called `MicroPEM`, functions for examining this information in order to look for possible problems in the data. The package moreover provides a Shiny app used for the field work of the CHAI project, but that could easily be adapted to other contexts.

This document aims at providing an overview of the functionalities of the package.

Checking individual files: from input data to `MicroPEM` objects
================================================================

The MicroPEM device outputs a csv file with all the information about the measures:

-   the measures themselves (relative humidity corrected nephelometer),

-   other measures that can help interpret them or check that no problem occured (temperature, relative humidity, battery, orifice pressure, inlet pressure, flow, accelerometer variables, reasons for shutdown, and variables related to user compliance),

-   a reminder of parameters set by the user (calibration parameters, frequency of measures)

-   and information about the device (filter ID, version of the software, etc). This is a lot of information, compiled in a handy csv format that is optimal for not loosing any data along the way, but not practical for analysis.

Therefore, the `ammon` package offers a R6 class called `MicroPEM` for storing the information, that will be easier to use by other functions. The class has fields with measures over time and a field that is a list containing all the information located at the top of the MicroPEM output file, called `control`. Here is a picture of a RTI MicroPEM output file showing how the information is stored in the R6 class.

![alt text](vignettes/outputRTI.png)

We will start by presenting the `control` field.

`control` field
---------------

This field is a data.frame (dplyr tbl\_df) that includes 41 variables:

-   `downloadDate` which is the date at which the files was downloaded from the device to a PC. It is a `POSIXt`.

-   `totalDownloadTime` gives the total download time, in seconds.

-   `deviceSerial` is the serial number of the device, which could be useful for e.g. finding a faulty device based on many output files.

-   `dateTimeHardware` indicates the date of release of the device. It is a `POSIXt`.

-   `dateTimeSoftware` indicates the date of release of the software used on the device. It is a `POSIXt`.

-   `version` indicates the version of the software.

-   `participantID` indicated the participantID, deduced either from the corresponding cell in the output file, or from the filename.

-   `filterID` is the ID number of the filter used for these measures.

-   `participantWeight` is a numeric variable.

-   `inletAerosolSize` is a factor variable, either "PM2.5" or "PM10". Please note that this variable does not provide confirmation that this aerosol size was measured, since it was chosen by hand by the person preparing the device for these measures. One could choose "PM2.5" and put the inlet for PM10, in which case PM10, not PM2.5, would be measured.

-   `laserCyclingVariablesDelay`

-   `laserCyclingVariablesSamplingTime`

-   `laserCyclingVariablesOffTime`

-   `SystemTimes` indicates whether the device was always on, or whether it was on/off, in which case the length of the on and off periods are given in seconds.

-   `nephelometerSlope`

-   `nephelometerOffset`

-   `nephelometerLogInterval` indicates how many seconds there are between measures of PM.

-   `temperatureSlope`

-   `temperatureOffset`

-   `temperatureLog` indicates how many seconds there are between measures of temperature.

-   `humiditySlope`

-   `humidityOffset`

-   `humidityLog` indicates how many seconds there are between measures of humidity.

-   `inletPressureSlope`

-   `inletPressureOffset`

-   `inletPressureLog` indicates how many seconds there are between measures of inlet pressure.

-   `inletPressureHighTarget`

-   `inletPressureLowTarget`

-   `orificePressureSlope`

-   `orificePressureOffset`

-   `orificePressureLog` indicates how many seconds there are between measures of orifice pressure.

-   `orificePressureHighTarget`

-   `orificePressureLowTarget`

-   `flowLog` indicates how many seconds there are between measures of flow.

-   `flowHighTarget`

-   `flowLowTarget`

-   `flowWhatIsThis`

-   `accelerometerLog`

-   `batteryLog` indicates how many seconds there are between measures of battery.

-   `ventilationSlope`

-   `ventilationOffset`

`measures` field
----------------

This field is a data.frame (dplyr tbl\_df) with these 15 columns:

-   `timeDate` is a `POSIXt` giving the date and time of each measure.

-   `nephelometer` is a numeric variable indicating the RH-corrected PM2.5 concentration in \(\micro g/m^3\)

-   `temperature` is a numeric variable, in centigrade.

-   `relativeHumidity` is a proportion.

-   `battery`is a numeric variable.

-   `orifice pressure` and `inlet pressure` are numeric variables, in inches of water.

-   `flow` is a numeric variable, in liters per minute.

-   `xAxis`, `yAxis` and `zAxis` are accelerometer measures in three dimensions, with `vectorSum` being their sum.

-   `shutDownReason` is a factor variable indicating the reason for shutdown, in case a shutdown happened at this timepint.

-   `wearingCompliance` and `validityWearingCompliance` are respectively a logical and a character variables.

The `convertOutput` function.
-----------------------------

The `convertOutput` only takes two arguments as input: the path to the output file, and the version of the output file, either "CHAI" or "Columbia" (version with one blank line after each line with content). The result of a call to this function is an object of the class `MicroPEM`. Below is a example of a call to `convertOutput`.

``` r
library("ammon")
MicroPEMExample <- convertOutput(system.file("extdata", "dummyCHAI.csv", package = "ammon"),
version="CHAI")
class(MicroPEMExample)
```

    ## [1] "MicroPEM" "R6"

Visualizing information contained in a `MicroPEM` object
--------------------------------------------------------

### Plot method

The R6 `microPEM` class has its own plot method. It allows to draw a plot of all time-varying measures against the `timeDate` field. It takes two arguments: the `MicroPEM` object to be plotted, and the type of plots to be produced, either a "plain" `ggplot2` plot with 6 facets, or its interactive version produced with the `ggiraph` package -- the corresponding values of type are respectively "plain" and "interactive".

Below we show to examples of uses of the plot method on a `MicroPEM` object.

This is a "plain" plot.

``` r
data("dummyMicroPEMChai")
par(mar=c(1,4,2,1))
dummyMicroPEMChai$plot()
```

![](README_files/figure-markdown_github/unnamed-chunk-4-1.png)

This is a nicer and interactive representation: you can look at what happens if you put your mouse over the time series. It is to be used as visualization tool as well, not as a plot method for putting a nice figure in a paper.

``` r
library("ggiraph")
p <- dummyMicroPEMChai$plot(type = "interactive")
ggiraph(code = {print(p)}, width = 10, height = 10)
```

### `summary` method

Plotting the `MicroPEM` object is already a good way to notice any problem. Another methods aims at providing more compact information about the time-varying measures. It is called `summary` and outputs a table with summary statistics for each time-varying measures, except timeDate.

Below is an example of use of this method.

``` r
library("xtable")
data("dummyMicroPEMChai")
results <- dummyMicroPEMChai$summary()
results %>% knitr::kable()
```

| measure          |  No. of not missing values|  Median|        Mean|  Minimum|  Maximum|   Variance|
|:-----------------|--------------------------:|-------:|-----------:|--------:|--------:|----------:|
| nephelometer     |                       8634|   49.00|  49.3745657|    45.00|    93.00|  1.6780557|
| temperature      |                       2878|   84.50|  84.6830438|    82.30|    87.60|  1.7180023|
| relativeHumidity |                       8634|   54.60|  55.0061733|    46.20|    64.90|  7.6665285|
| battery          |                       1464|    4.10|   4.0872268|     3.90|     4.30|  0.0078272|
| orificePressure  |                       2878|    0.15|   0.1505455|     0.14|     0.16|  0.0000072|
| inletPressure    |                       2878|    0.11|   0.1111015|     0.10|     0.13|  0.0000538|
| flow             |                       2878|    0.77|   0.7703023|     0.77|     0.78|  0.0000029|

### Shiny app developped for the CHAI project

In the context of the [CHAI project](http://www.chaiproject.org/), we developped a Shiny app based on the previous functions, that allows to explore a MicroPEM output file. The app is called by the function `runShinyApp` with no argument. There is one side panel where one can choose the file to analyse. There are four tabs:

-   One with the output of a call to `summaryTimeVarying`,

-   One with the output of a call to the `alarmCHAI` function that performs a few checks specific to the CHAI project,

-   One with the output of a call to the plot method,

-   One with the output of a call to `summarySettings`.

This app allows the exploration of a MicroPEM output file with no R experience.

Below we show screenshots of the app.

![alt text](vignettes/shinyTabSummary.png)

![alt text](vignettes/shinyTabAlarm.png)

![alt text](vignettes/shinyTabPlot.png)

![alt text](vignettes/shinyTabSettings.png)

### The `filterTimeDate` function

One could be interested in only a part of the time-varying measures, e.g. the measures from the afternoon. Using the `filterTimeDate`function on a `MicroPEM` object, one can get a `MicroPEM` object with shorter fields for the time-varying variables, based on the values of `fromTime`and `untilTime` that should be `POSIXct`.

In the code below, we don't want measures from the first 12 hours of measures.

``` r
# load the lubridate package
library('lubridate')
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

``` r
# load the dummy MicroPEM object
data('dummyMicroPEMChai')
# look at the dimensions of the data.frame
dummyMicroPEMChai$measures %>% head() %>% knitr::kable()
```

| timeDate            |  nephelometer|  temperature|  relativeHumidity|  battery|  orificePressure|  inletPressure|  flow|  xAxis|  yAxis|  zAxis|  vectorSum| shutDownReason                | wearingCompliance |  validityWearingComplianceValidation| originalDateTime   |
|:--------------------|-------------:|------------:|-----------------:|--------:|----------------:|--------------:|-----:|------:|------:|------:|----------:|:------------------------------|:------------------|------------------------------------:|:-------------------|
| 2015-07-03 08:02:18 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|     NA|     NA|     NA|         NA| Manual Stop in Pump Test Menu | NA                |                                    0| 07/03/2015 8:02:18 |
| 2015-07-03 08:02:32 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|     NA|     NA|     NA|         NA| USB was disconnected          | NA                |                                    0| 07/03/2015 8:02:32 |
| 2015-07-03 08:05:51 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|     NA|     NA|     NA|         NA| Button 1 pressed              | NA                |                                    0| 07/03/2015 8:05:51 |
| 2015-07-03 08:05:52 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|     NA|     NA|     NA|         NA| Start button                  | NA                |                                    0| 07/03/2015 8:05:52 |
| 2015-07-03 08:05:55 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|   0.26|  -0.37|  -1.01|       1.10|                               | NA                |                                    0| 07/03/2015 8:05:55 |
| 2015-07-03 08:06:00 |            NA|           NA|                NA|      4.3|               NA|             NA|    NA|   1.05|   0.03|  -0.07|       1.06|                               | NA                |                                    0| 07/03/2015 8:06:00 |

``` r
# command for only erasing measures from the first twelve hours
shorterMicroPEM <- filterTimeDate(MicroPEMObject=dummyMicroPEMChai,
untilTime=NULL,
fromTime=min(dummyMicroPEMChai$measures$timeDate, na.rm=TRUE) + hours(12))
# look at the dimensions of the data.frame
shorterMicroPEM$measures %>% head() %>% knitr::kable()
```

| timeDate            |  nephelometer|  temperature|  relativeHumidity|  battery|  orificePressure|  inletPressure|  flow|  xAxis|  yAxis|  zAxis|  vectorSum| shutDownReason | wearingCompliance |  validityWearingComplianceValidation| originalDateTime    |
|:--------------------|-------------:|------------:|-----------------:|--------:|----------------:|--------------:|-----:|------:|------:|------:|----------:|:---------------|:------------------|------------------------------------:|:--------------------|
| 2015-07-03 20:02:20 |            49|           NA|              54.3|       NA|               NA|             NA|    NA|   0.98|   0.10|  -0.10|       0.99|                | NA                |                                    0| 07/03/2015 20:02:20 |
| 2015-07-03 20:02:25 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|   0.97|   0.09|  -0.13|       0.99|                | NA                |                                    0| 07/03/2015 20:02:25 |
| 2015-07-03 20:02:30 |            49|           NA|              54.5|       NA|               NA|             NA|    NA|   0.98|   0.10|  -0.12|       0.99|                | NA                |                                    0| 07/03/2015 20:02:30 |
| 2015-07-03 20:02:35 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|   0.97|   0.17|   0.01|       0.99|                | NA                |                                    0| 07/03/2015 20:02:35 |
| 2015-07-03 20:02:40 |            49|         85.9|              54.7|       NA|             0.15|           0.12|  0.78|  -0.33|   0.27|  -0.98|       1.07|                | NA                |                                    0| 07/03/2015 20:02:40 |
| 2015-07-03 20:02:45 |            NA|           NA|                NA|       NA|               NA|             NA|    NA|  -0.26|   0.29|  -1.00|       1.07|                | NA                |                                    0| 07/03/2015 20:02:45 |

From a bunch of output files to data ready for further analysis
===============================================================
