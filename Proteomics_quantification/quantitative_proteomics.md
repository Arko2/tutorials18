Analysis of Quantitative Proteomics Data
================

The result of a proteomic quantitative analysis is a list of peptide and protein abundances for every protein in different samples, or abundance ratios between the samples. The downstream interpretation of these data varies according to the experimental design. In this chapter we will describe different generic methods for the interpretation of quantitative datasets.

The dataset used here for illustrative purposes is freely available through the ProteomeXchange [(1)](#references) consortium via the PRIDE [(2)](#references) partner repository under the accession number PXD000441, and available in the *resources* folder under *Results\_MQ\_5cell-line-mix*. It consists of five cell lines derived from acute myeloid leukemia (AML) patients measured with a spiked-in internal stanard (IS) obtained from the combination of the same five AML cell lines that have been metabolically labeled with heavy isotopes [(3)](#references) and analyzed using MaxQuant [(4)](#references) version 1.4.1.2. See [(5)](#references) for details.

This chapter introduces the basic methods, and is by no means aiming at covering all possibilities or present a reference workflow. Please continue exploring the data on your own and critically adapt the interpretation workflow to your experiment - there is no one workflow fits all!

Installation
------------

This tutorial uses [R](https://www.r-project.org), a language that allows the simple manipulation of large datasets. We will use R from the open source [RSudio](www.rstudio.com) environment. Please make sure to have RStudio installed on your computer.

Create a new project via the *File* -&gt; *New Projet* menu. Select *New Directory* -&gt; *Empty Project*, and select a folder for your project. Create a new script via the *File* -&gt; *New File* -&gt; *R Script* menu.

The commands used in this tutorial are given in line and the results produced are preceded by '\#\#'. You can copy the commands of this tutorial in your script, where one line corresponds to a command. The commands can be run line by line using the *Run* button. Alternatively, you can run the entire script using the *Source* button. Note that it is also possible to run the commands in the *Console*.

We are going to use the [ggplot2](http://ggplot2.org/) library. If you did not install this package already, install it usng the *Packages* tab or run `install.packages("ggplot2")`.

``` r
library(ggplot2)
```

File import
-----------

The MaxQuant report files are tab separated text files that can readily be imported in R as data frame. A data frame allows the convenient manipulation of large tables. Please download the file [*proteinGroups\_5cell-line-mix.txt*](https://github.com/mvaudel/tutorials/blob/master/Expression_data/proteinGroups_5cell-line-mix.txt) to your project folder. It should appear in the *Files* panel.

``` r
proteinGroupsInput <- read.table(file = "proteinGroups_5cell-line-mix.txt", header = T, stringsAsFactors = F, sep = "\t")
```

You should see the proteinGroupsInput in your *Environment* panel. You can get the size of the table using the *length* function.

``` r
nColumns <- length(proteinGroupsInput)
nLines <- length(proteinGroupsInput$Protein.IDs)
paste("Number of columns: ", nColumns, ", number of protein groups: ", nLines, sep="")
```

    ## [1] "Number of columns: 191, number of protein groups: 3982"

Filtering of the protein groups
-------------------------------

MaxQuant indicates contaminants, decoy sequences, and proteins only identified by site by '+' in their columns. You can access the number of every category by using the *table* function.

``` r
nContaminants <- table(proteinGroupsInput$Contaminant)
paste("Number of Contaminants: ", nContaminants[2], ", Others: ", nContaminants[1], sep="")
```

    ## [1] "Number of Contaminants: 104, Others: 3878"

``` r
nDecoys <- table(proteinGroupsInput$Reverse)
paste("Number of decoys: ", nDecoys[2], ", Others: ", nDecoys[1], sep="")
```

    ## [1] "Number of decoys: 76, Others: 3906"

``` r
nOnlyIdentifiedBySite <- table(proteinGroupsInput$Only.identified.by.site)
paste("Number of only identified by site: ", nOnlyIdentifiedBySite[2], ", Others: ", nOnlyIdentifiedBySite[1], sep="")
```

    ## [1] "Number of only identified by site: 109, Others: 3873"

Filter these lines and put the result in a new table named \_proteinGroups.

``` r
proteinGroups <- proteinGroupsInput[proteinGroupsInput$Contaminant != '+' & proteinGroupsInput$Reverse != '+' & proteinGroupsInput$Only.identified.by.site != '+',]

nLines <- length(proteinGroups$Protein.IDs)
nFiltered <- length(proteinGroupsInput$Protein.IDs) - length(proteinGroups$Protein.IDs)
paste("New number of protein groups: ", nLines, ", Number of protein groups removed: ", nFiltered, sep="")
```

    ## [1] "New number of protein groups: 3732, Number of protein groups removed: 250"

Extraction of the quantitative columns
--------------------------------------

The expression values of the five cell lines used are stored in different columns as detailed below. They correspond to two states of the disease, *Diagnosis* and *Relapse*.

| Column                  | Cell Line | Condition |
|-------------------------|-----------|-----------|
| Ratio H/L normalized E1 | Molm-13   | Relapse   |
| Ratio H/L normalized E2 | MV4-11    | Diagnosis |
| Ratio H/L normalized E3 | NB4       | Relapse   |
| Ratio H/L normalized E4 | OCI-AML3  | Diagnosis |
| Ratio H/L normalized E5 | THP-1     | Relapse   |

Note that when importing the data in R, spaces and special characters in the column titles were replaces by dots.

We will now store these values in a new table and save the columns of the cell lines at different conditions in two vectors.

``` r
proteinGroupsRatios <- data.frame(proteinIDs = proteinGroups$Protein.IDs, 
                     molm13 = proteinGroups$Ratio.H.L.normalized.E1, 
                     mv411 = proteinGroups$Ratio.H.L.normalized.E2,
                     nb4 = proteinGroups$Ratio.H.L.normalized.E3,
                     ociAml3 = proteinGroups$Ratio.H.L.normalized.E4,
                     thp1 = proteinGroups$Ratio.H.L.normalized.E5)
diagnosisCellLines <- c("mv411", "ociAml3")
relapseCellLines <- c("molm13", "nb4", "thp1")
```

The ratios can be plotted against each others for different cell lines in a scatter plot using the code below.

``` r
cellLineX <- "mv411"
cellLineY <- "ociAml3"
scatterPlot <- ggplot()
scatterPlot <- scatterPlot + geom_point(aes(x=proteinGroupsRatios[,cellLineX], y=proteinGroupsRatios[,cellLineY]), alpha=0.2, col="blue", size=1, na.rm = T)
scatterPlot <- scatterPlot + xlab(cellLineX) + ylab(cellLineY)
plot(scatterPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/scatter_plot-1.png)

It is possible to extract the ratios of a given protein.

``` r
proteinGroupsRatios[proteinGroupsRatios$proteinIDs == "A0FGR8",]
```

    ##   proteinIDs molm13   mv411     nb4 ociAml3    thp1
    ## 2     A0FGR8 1.0297 0.97241 0.82373 0.87853 0.81302

It is possible to extract the ratios of a given protein for a given condition.

``` r
proteinGroupsRatios[proteinGroupsRatios$proteinIDs == "A0FGR8", names(proteinGroupsRatios) %in% diagnosisCellLines]
```

    ##     mv411 ociAml3
    ## 2 0.97241 0.87853

Note that some proteins have missing values.

``` r
proteinGroupsRatios[proteinGroupsRatios$proteinIDs == "A0JLT2",]
```

    ##   proteinIDs molm13 mv411 nb4 ociAml3 thp1
    ## 3     A0JLT2    NaN   NaN NaN     NaN  NaN

Filtering based on the number of missing values
-----------------------------------------------

It is possible to see whether a value is missing using the is.na function.

``` r
is.na(proteinGroupsRatios[proteinGroupsRatios$proteinIDs == "A0JLT2",])
```

    ##   proteinIDs molm13 mv411  nb4 ociAml3 thp1
    ## 3      FALSE   TRUE  TRUE TRUE    TRUE TRUE

The number of missing values for a condition can be obtained with the sum function.

``` r
sum(is.na(proteinGroupsRatios[proteinGroupsRatios$proteinIDs == "A0JLT2", names(proteinGroupsRatios) %in% diagnosisCellLines]))
```

    ## [1] 2

Filter the ratio table to have at least two valid values in every conition, i.e. no missing value at diagnosis and maximum one at relapse. Doing the sum on every row is done using rowSums.

``` r
nMissingDiagnosis <- rowSums(is.na(proteinGroupsRatios[, names(proteinGroupsRatios) %in% diagnosisCellLines]))
nMissingRelapse <- rowSums(is.na(proteinGroupsRatios[, names(proteinGroupsRatios) %in% relapseCellLines]))
validProteinGroupsRatios <- proteinGroupsRatios[nMissingDiagnosis == 0 & nMissingRelapse <= 1,]

nLines <- length(validProteinGroupsRatios$proteinIDs)
nFiltered <- length(proteinGroupsRatios$proteinIDs) - length(validProteinGroupsRatios$proteinIDs)
paste("New number of protein groups: ", nLines, ", Number of protein groups removed: ", nFiltered, sep="")
```

    ## [1] "New number of protein groups: 1936, Number of protein groups removed: 1796"

The number of protein presenting missing values can be plotted using the code below.

``` r
categoryDiagnosis <- character(length(nMissingDiagnosis))
categoryDiagnosis[] <- "Diagnosis"
categoryRelapse <- character(length(nMissingRelapse))
categoryRelapse[] <- "Relapse"
categories <- c(categoryDiagnosis, categoryRelapse)
values <- c(nMissingDiagnosis, nMissingRelapse)

missingValuesHistogramPlot <- ggplot()
missingValuesHistogramPlot <- missingValuesHistogramPlot + geom_bar(aes(x=values, fill=categories), position = "dodge")
missingValuesHistogramPlot <- missingValuesHistogramPlot + xlab("Number of missing values") + ylab("Number of proteins")
plot(missingValuesHistogramPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/plot_missing_values_histogram-1.png)

Transformation of the ratios
----------------------------

The MaxQuant ratios are estimated as *Heavy/Light*, but our reference was on the *Heavy* channel, we therefore need to invert the ratios in order to obtain a value representing the cell line to internal standard ratio.

``` r
validProteinGroupsRatios[,"molm13"] <- 1/validProteinGroupsRatios[,"molm13"]
validProteinGroupsRatios[,"mv411"] <- 1/validProteinGroupsRatios[,"mv411"]
validProteinGroupsRatios[,"nb4"] <- 1/validProteinGroupsRatios[,"nb4"]
validProteinGroupsRatios[,"ociAml3"] <- 1/validProteinGroupsRatios[,"ociAml3"]
validProteinGroupsRatios[,"thp1"] <- 1/validProteinGroupsRatios[,"thp1"]
```

As one can see, the ratios are distributed over positive values around 1.

``` r
categories <- c()
categoryCellLine <- character(length(validProteinGroupsRatios[,"molm13"]))
categoryCellLine[] <- "molm13"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"mv411"]))
categoryCellLine[] <- "mv411"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"nb4"]))
categoryCellLine[] <- "nb4"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"ociAml3"]))
categoryCellLine[] <- "ociAml3"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"thp1"]))
categoryCellLine[] <- "thp1"
categories <- c(categories, categoryCellLine)
values <- c(validProteinGroupsRatios[,"molm13"],
            validProteinGroupsRatios[,"mv411"],
            validProteinGroupsRatios[,"nb4"],
            validProteinGroupsRatios[,"ociAml3"],
            validProteinGroupsRatios[,"thp1"])
ratiosDensityPlot <- ggplot()
ratiosDensityPlot <- ratiosDensityPlot + geom_density(aes(x=values, col=categories, fill = categories), alpha = 0.1, na.rm=TRUE)
ratiosDensityPlot <- ratiosDensityPlot + xlim(0, 5)
ratiosDensityPlot <- ratiosDensityPlot + xlab("Ratio") + ylab("Density of proteins")
plot(ratiosDensityPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/ratio_densities-1.png)

Before further processing, we log transform these ratios to restore the symmetry around 1.

``` r
validProteinGroupsRatios[,"molm13"] <- log2(validProteinGroupsRatios[,"molm13"])
validProteinGroupsRatios[,"mv411"] <- log2(validProteinGroupsRatios[,"mv411"])
validProteinGroupsRatios[,"nb4"] <- log2(validProteinGroupsRatios[,"nb4"])
validProteinGroupsRatios[,"ociAml3"] <- log2(validProteinGroupsRatios[,"ociAml3"])
validProteinGroupsRatios[,"thp1"] <- log2(validProteinGroupsRatios[,"thp1"])
```

Now the distributions are centered around 0 and the ratios symmetrically distributed.

``` r
categories <- c()
categoryCellLine <- character(length(validProteinGroupsRatios[,"molm13"]))
categoryCellLine[] <- "molm13"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"mv411"]))
categoryCellLine[] <- "mv411"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"nb4"]))
categoryCellLine[] <- "nb4"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"ociAml3"]))
categoryCellLine[] <- "ociAml3"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"thp1"]))
categoryCellLine[] <- "thp1"
categories <- c(categories, categoryCellLine)
values <- c(validProteinGroupsRatios[,"molm13"],
            validProteinGroupsRatios[,"mv411"],
            validProteinGroupsRatios[,"nb4"],
            validProteinGroupsRatios[,"ociAml3"],
            validProteinGroupsRatios[,"thp1"])
ratiosDensityPlot <- ggplot()
ratiosDensityPlot <- ratiosDensityPlot + geom_density(aes(x=values, col=categories, fill = categories), alpha = 0.1, na.rm=TRUE)
ratiosDensityPlot <- ratiosDensityPlot + xlab("Ratio") + ylab("Density of proteins")
plot(ratiosDensityPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/ratio_log_densities-1.png)

Normalization of the ratios
---------------------------

The ratios provided by MaxQuant are already normalized, but it can be useful to conduct an additional normalization to correct for eventual biases. Note: Don't forget to exclude missing values.

``` r
cellLineMedian <- median(validProteinGroupsRatios[,"molm13"], na.rm = T)
validProteinGroupsRatios[,"molm13"] <- validProteinGroupsRatios[,"molm13"] - cellLineMedian
cellLineMedian <- median(validProteinGroupsRatios[,"mv411"], na.rm = T)
validProteinGroupsRatios[,"mv411"] <- validProteinGroupsRatios[,"mv411"] - cellLineMedian
cellLineMedian <- median(validProteinGroupsRatios[,"nb4"], na.rm = T)
validProteinGroupsRatios[,"nb4"] <- validProteinGroupsRatios[,"nb4"] - cellLineMedian
cellLineMedian <- median(validProteinGroupsRatios[,"ociAml3"], na.rm = T)
validProteinGroupsRatios[,"ociAml3"] <- validProteinGroupsRatios[,"ociAml3"] - cellLineMedian
cellLineMedian <- median(validProteinGroupsRatios[,"thp1"], na.rm = T)
validProteinGroupsRatios[,"thp1"] <- validProteinGroupsRatios[,"thp1"] - cellLineMedian
```

Removing the median put the center of the distributions at 0.

``` r
categories <- c()
categoryCellLine <- character(length(validProteinGroupsRatios[,"molm13"]))
categoryCellLine[] <- "molm13"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"mv411"]))
categoryCellLine[] <- "mv411"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"nb4"]))
categoryCellLine[] <- "nb4"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"ociAml3"]))
categoryCellLine[] <- "ociAml3"
categories <- c(categories, categoryCellLine)
categoryCellLine <- character(length(validProteinGroupsRatios[,"thp1"]))
categoryCellLine[] <- "thp1"
categories <- c(categories, categoryCellLine)
values <- c(validProteinGroupsRatios[,"molm13"],
            validProteinGroupsRatios[,"mv411"],
            validProteinGroupsRatios[,"nb4"],
            validProteinGroupsRatios[,"ociAml3"],
            validProteinGroupsRatios[,"thp1"])
ratiosDensityPlot <- ggplot()
ratiosDensityPlot <- ratiosDensityPlot + geom_density(aes(x=values, col=categories, fill = categories), alpha = 0.1, na.rm=TRUE)
ratiosDensityPlot <- ratiosDensityPlot + xlab("Ratio") + ylab("Density of proteins")
plot(ratiosDensityPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/ratio_log_densities_normalized-1.png)

t-test
------

We are going to evaluate the significance of the ratio between the two groups using a t-test. This example does not have sufficient number of replicates to draw any biological conclusion but this is just an example. Below is an example using A0FGR8.

``` r
valuesDiagnostic <- validProteinGroupsRatios[validProteinGroupsRatios$proteinIDs == "A0FGR8", names(validProteinGroupsRatios) %in% diagnosisCellLines]
valuesRelapse <- validProteinGroupsRatios[validProteinGroupsRatios$proteinIDs == "A0FGR8", names(validProteinGroupsRatios) %in% relapseCellLines]
t.test(valuesDiagnostic, valuesRelapse, alternative = "two.sided", paired = F)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  valuesDiagnostic and valuesRelapse
    ## t = -0.44869, df = 2.1995, p-value = 0.694
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5336322  0.4247502
    ## sample estimates:
    ## mean of x mean of y 
    ## 0.1340211 0.1884621

Note that the test run is actually a Welch Two Sample t-test, an extension of the Student t-test more reliable for samples of unequal variances. We create new columns containing the test statistics, as well as the fold change and -10log(p-value).

``` r
pValues <- c()
ts <- c()
fcs <- c()
pLog <- c()
for (i in 1:length(validProteinGroupsRatios$proteinIDs)) {
  valuesDiagnostic <- validProteinGroupsRatios[i, names(validProteinGroupsRatios) %in% diagnosisCellLines]
  valuesRelapse <- validProteinGroupsRatios[i, names(validProteinGroupsRatios) %in% relapseCellLines]
  test <- t.test(valuesDiagnostic, valuesRelapse, alternative = "two.sided", paired = F)
  pValues <- c(pValues, test$p.value)
  pLog <- c(pLog, -log10(test$p.value))
  ts <- c(ts, test$statistic)
  medianDiagnostic <- median(as.numeric(valuesDiagnostic), na.rm = T)
  medianRelapse <- median(as.numeric(valuesRelapse), na.rm = T)
  fc <- medianRelapse-medianDiagnostic
  fcs <- c(fcs, fc)
}
validProteinGroupsRatios$fc <- fcs
validProteinGroupsRatios$tTestPValue <- pValues
validProteinGroupsRatios$tTestPValueLog <- pLog
validProteinGroupsRatios$tTestStatistic <- ts
```

QQ Plot
-------

We will now see whether the t-test statistics actually follow a t-distribution. This is achieved by drawing a quantile-quantile plot (qq-plot) where the quantile of the observed t-statistics is plotted against the theoretical quantiles of the Student t distribution.

``` r
degreesOfFreedom <- 5-2 # 5 cell lines

expectedQuantiles <- rt(length(validProteinGroupsRatios$tTestStatistic), df=degreesOfFreedom)
expectedQuantiles <- sort(expectedQuantiles)
measuredQuantiles <- validProteinGroupsRatios$tTestStatistic
measuredQuantiles <- sort(measuredQuantiles)

qqPlot <- ggplot()
qqPlot <- qqPlot + geom_point(aes(x=expectedQuantiles, y=measuredQuantiles), size = 1, col = "blue")
qqPlot <- qqPlot + geom_line(aes(x=expectedQuantiles, y=expectedQuantiles), size = 1, alpha = 0.5, linetype = "dotted")
qqPlot <- qqPlot + xlab("Expected Quantile") + ylab("Observed Quantile")
plot(qqPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/qq_plot-1.png)

It is possible to make the same plot for the observed and expected p-values, based on these quantiles. The red dots show a tolerated deviation ratio of from the expected p-values as set in the *ppTolerance* variable. Note that the measured p-values deviate from the diagonal at low p-values, indicating that our dataset does not fully satisfy the hypotheses of the t-test.

``` r
ppTolerance <-1.1

expectedPValues <- pt(expectedQuantiles, lower.tail = expectedQuantiles < 0, df=degreesOfFreedom)
expectedPValues <- sort(expectedPValues)
expectedPValuesLog <- -log10(expectedPValues)
measuredPValues <- validProteinGroupsRatios$tTestPValue
measuredPValues <- sort(measuredPValues)
measuredPValuesLog <- -log10(measuredPValues)

limitLow <- expectedPValues/ppTolerance
limitLow <- -log10(limitLow)
limitHigh <- ppTolerance * expectedPValues
limitHigh <- -log10(limitHigh)

ppPlot <- ggplot()
ppPlot <- ppPlot + geom_point(aes(x=expectedPValuesLog, y=measuredPValuesLog), size = 1, col = "blue")
ppPlot <- ppPlot + geom_line(aes(x=expectedPValuesLog, y=expectedPValuesLog), size = 1, alpha = 0.5, linetype = "dashed", col="black")
ppPlot <- ppPlot + geom_line(aes(x=expectedPValuesLog, y=limitLow), size = 1, alpha = 0.5, linetype = "dotted", col="red")
ppPlot <- ppPlot + geom_line(aes(x=expectedPValuesLog, y=limitHigh), size = 1, alpha = 0.5, linetype = "dotted", col="red")
ppPlot <- ppPlot + xlab("Expected p-value [-log10(p)]") + ylab("Observed p-value [-log10(p)]")
plot(ppPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/pp_plot-1.png)

We calculate lambda as the ratio of the observed p-value and the expected p-value. A density plot shows that the majority of the p-values are within a 1.1 deviation ratio.

``` r
lambda <- measuredPValues/expectedPValues
lambdaLog <- log(lambda, base = ppTolerance)

lambdaHistogramPlot <- ggplot()
lambdaHistogramPlot <- lambdaHistogramPlot + geom_density(aes(x=lambdaLog), col="blue", fill = "blue", alpha = 0.1)
lambdaHistogramPlot <- lambdaHistogramPlot + geom_vline(aes(xintercept=-1), col="red", alpha = 0.8, linetype="dotted")
lambdaHistogramPlot <- lambdaHistogramPlot + geom_vline(aes(xintercept=1), col="red", alpha = 0.8, linetype="dotted")
lambdaHistogramPlot <- lambdaHistogramPlot + xlab(paste("Lambda (Observed p-value / Expected p-value) [log", ppTolerance, "]", sep="")) + ylab("Density of proteins")
plot(lambdaHistogramPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/hist_lambda-1.png)

``` r
corePopulation <- round(100*length(lambdaLog[abs(lambdaLog) < 1])/length(lambdaLog))/100
```

70% of the p-values is contained within the 1.1 deviation ratio. This percentile will be used in the following as core population.

Multiple hypothesis testing
---------------------------

The more we run t-tests, the more chances we have to get a low p-value by chance. For example, when we run the test just one time, we are unlikely to find a p &lt; 1% by chance, whereas if we run the test 100 times, we are likely to find at least one p &lt; 1% by chance. This problem is called the multiple hypothesis testing.

It is possible to adjust the p-values to correct for multiple hypothesis testing. This can be done using the `p.adjust` function of the stats package. Note that numerous methods were established to correct for multiple hypothesis testing. The simplest approach is to multiply the p-values by the number of tests, called a *Bonferroni* correction. It is also possible to control the expected share of incorrect findings among the significant results, a False Discovery Rate (FDR). This is called a Benjamini and Hochberg correction, "BH", [(6)](#references).

``` r
validProteinGroupsRatios$bonferroniPValue <- p.adjust(validProteinGroupsRatios$tTestPValue, method = "bonferroni")
validProteinGroupsRatios$bhFDR <- p.adjust(validProteinGroupsRatios$tTestPValue, method = "BH")

bhPlot <- ggplot()
bhPlot <- bhPlot + geom_line(aes(x=validProteinGroupsRatios$tTestPValueLog, 100 * validProteinGroupsRatios$bonferroniPValue), col="red", alpha = 0.8)
bhPlot <- bhPlot + geom_line(aes(x=validProteinGroupsRatios$tTestPValueLog, 100 * validProteinGroupsRatios$bhFDR), col="blue", alpha = 0.8)
bhPlot <- bhPlot + xlab("Original p-value") + ylab("Bonferroni p-value [%] (red) - BH FDR [%] (blue)")
plot(bhPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/bh_plot-1.png)

According to these two methods, no result can be retained accounting for multiple hypothesis testing.

Independent weighting hypothesis
--------------------------------

The following code shows the histogram of fold changes between diagnostic and relapse for all proteins, as estimated in the t-test section by the difference between the median value of relapse cell lines compaired to the median value of diagnostic cell lines. The red line shows a normal distribution with mean and standard deviation calibrated on the median and the 34th percentiles around the median, respectively.

``` r
nBinsInOne <- 10
binSize <- 1/nBinsInOne
minFC <- floor(min(validProteinGroupsRatios$fc))
maxFC <- ceiling(max(validProteinGroupsRatios$fc))

quantilesFC <- quantile(validProteinGroupsRatios$fc, c(0.16, 0.5, 0.84), na.rm = T, names = F)
medianFC <- quantilesFC[2]
interQuantilesFC <- quantilesFC[3] - quantilesFC[1]
xDistribution <- minFC + (binSize * (1:(nBinsInOne * (maxFC - minFC))))
yDistribution <- dnorm(x = xDistribution, mean = medianFC, sd = interQuantilesFC/2)
scale <- length(validProteinGroupsRatios$fc)/nBinsInOne
yDistributionNorm <- scale * yDistribution

fcHistogramPlot <- ggplot()
fcHistogramPlot <- fcHistogramPlot + geom_histogram(aes(x=validProteinGroupsRatios$fc), col="blue", fill = "blue", alpha = 0.1, binwidth = binSize)
fcHistogramPlot <- fcHistogramPlot + geom_line(aes(x=xDistribution, yDistributionNorm), col="red", alpha = 0.8)
fcHistogramPlot <- fcHistogramPlot + xlab("Fold Change") + ylab("Number of proteins")
plot(fcHistogramPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/fc_histogram-1.png)

With the hypothesis that false positives introduced by multiple hypothesis testing are likely to distribute according to the rest of the fold changes, independently of the p-value, we can use the fold change as independent weighting hypothesis (IWH) to spread the hits on another dimension. The resulting plot is called a *volcano plot*.

The proteins on the upper side of the plot are the ones with the lowest p-value (note the inverted log10 scale). In order to avoid false positives, we are going to require these significant hits to have a high fold change. One typically uses a fold change of 2 (note the log2 scale).

``` r
pLimit <- 0.05
pLimitLog <- -log10(pLimit)
regulationConfidence <- character(length(validProteinGroupsRatios$tTestPValueLog))
regulationConfidence <- ifelse(validProteinGroupsRatios$tTestPValue < pLimit, "Significant but not Differentially Expressed", "Not Significant")
regulationConfidence <- ifelse(validProteinGroupsRatios$tTestPValue < pLimit & abs(validProteinGroupsRatios$fc) > 1, "Differentially Expressed", regulationConfidence)

volcanoPlot <- ggplot()
volcanoPlot <- volcanoPlot + geom_point(aes(x=validProteinGroupsRatios$fc, y=validProteinGroupsRatios$tTestPValueLog, col=regulationConfidence), size = 1, alpha = 0.5)
volcanoPlot <- volcanoPlot + geom_hline(aes(yintercept = pLimitLog), col="blue", alpha = 0.8, linetype="dotted", size = 1)
volcanoPlot <- volcanoPlot + geom_vline(aes(xintercept = -1), col="blue", alpha = 0.8, linetype="dotted", size = 1)
volcanoPlot <- volcanoPlot + geom_vline(aes(xintercept = 1), col="blue", alpha = 0.8, linetype="dotted", size = 1)
volcanoPlot <- volcanoPlot + scale_color_manual(values=c("darkgreen", "darkRed", "darkorange"), name="Protein Category")
volcanoPlot <- volcanoPlot + xlab("Fold Change [log2]") + ylab("p-value [-log(p)]")
plot(volcanoPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/volcano_plot-1.png)

Note that in contrary to the previous method, we are using arbitrary thresholds and we have no control on the error rate. A control of the error rate using IHW can be done using the [IHW package](http://bioconductor.org/packages/devel/bioc/vignettes/IHW/inst/doc/introduction_to_ihw.html) of bioconductor.

Missing values imputation
-------------------------

For the following, we are going to impute the missing values and assign them the value 0.

``` r
for (i in 1:length(diagnosisCellLines)) {
  validProteinGroupsRatios[,diagnosisCellLines[i]] <- ifelse(is.na(validProteinGroupsRatios[,diagnosisCellLines[i]]), 0, validProteinGroupsRatios[,diagnosisCellLines[i]])
}
for (i in 1:length(relapseCellLines)) {
  validProteinGroupsRatios[,relapseCellLines[i]] <- ifelse(is.na(validProteinGroupsRatios[,relapseCellLines[i]]), 0, validProteinGroupsRatios[,relapseCellLines[i]])
}
```

Principal component analysis
----------------------------

We will now draw a principal component analysis (PCA) plot.

``` r
pcaInput <- data.frame(validProteinGroupsRatios$molm13, 
                       validProteinGroupsRatios$mv411, 
                       validProteinGroupsRatios$nb4, 
                       validProteinGroupsRatios$ociAml3, 
                       validProteinGroupsRatios$thp1)
pca <- prcomp(pcaInput)
pc1 <- pca$rotation[,1]
pc2 <- pca$rotation[,2]
totalStd <- sum(pca$sdev)
contribution1 <- round(100*pca$sdev[1]/totalStd)
contribution2 <- round(100*pca$sdev[2]/totalStd)
names <- names(validProteinGroupsRatios)[2:6]

pcaPlot <- ggplot()
pcaPlot <- pcaPlot + geom_point(aes(x=pc1, y=pc2, col=names))
pcaPlot <- pcaPlot + xlab(paste("PC1 [", contribution1, "%]", sep=""))
pcaPlot <- pcaPlot + ylab(paste("PC2 [", contribution2, "%]", sep=""))
plot(pcaPlot)
```

![](quantitative_proteomics_files/figure-markdown_github/pca-1.png)

References
----------

1.  [Vizcaino, J.A. *et al.*, *ProteomeXchange provides globally coordinated proteomics data submission and dissemination*, Nature Biotechnology, 2014](https://www.ncbi.nlm.nih.gov/pubmed/24727771)
2.  [Martens, L. *et al.*, *PRIDE: the proteomics identifications database*, Proteomics, 2005](https://www.ncbi.nlm.nih.gov/pubmed/16041671)
3.  [Geiger, T. *et al.*, *Super-SILAC mix for quantitative proteomics of human tumor tissue*, Nature Methods, 2010](https://www.ncbi.nlm.nih.gov/pubmed/20364148)
4.  [Cox, J. and Mann, M, *MaxQuant enables high peptide identification rates, individualized p.p.b. range mass accuracies and proteome-wide protein quantification*, Nature Biotechnology, 2008](https://www.ncbi.nlm.nih.gov/pubmed/19029910)
5.  [Aasebo, E. *et al.*, *Performance of super-SILAC based quantitative proteomics for comparison of different acute myeloid leukemia (AML) cell lines*, Proteomics, 2014](https://www.ncbi.nlm.nih.gov/pubmed/25044641)
6.  [Benjamini, Y. and Hochberg, Y., *Controlling the false discovery rate: a practical and powerful approach to multiple testing*, Journal of the Royal Statistical Society, 1995](http://www.math.tau.ac.il/~ybenja/MyPapers/benjamini_hochberg1995.pdf)
