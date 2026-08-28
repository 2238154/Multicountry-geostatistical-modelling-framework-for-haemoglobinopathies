# Global-Haemoglobinopathy-Modelling-Framework

This repository contains the modelling pipeline for the global geostatistical analysis of haemoglobinopathy data. The pipeline prepares curated epidemiological datasets, supports covariate selection, evaluates and cross-validates candidate modelling approaches, generates gridded predictions with associated prediction uncertainty, produces administrative-unit-level burden estimates, and identifies priority locations for future epidemiological studies.

## Workflow

This repository is organised into scripts that follow the modelling workflow outlined below:

### **Covariates:** Covariate raster preparation

This script prepares the raster covariate layers used in the modelling framework and aligns them to a common global raster grid for downstream model fitting and prediction.

### **A:** Dataset preparation and covariate selection

This script prepares alternative dataset versions for model testing. These versions are used to assess the effect of different data preparation decisions, including duplicate-coordinate handling, exclusion of multi-location records, exclusion of Hardy-Weinberg-derived estimates, quality-appraisal filtering, geographic disaggregation, and retention or removal of records identified as spatial noise.

* **Datasets:**

  * **Dataset 1:** Baseline dataset retaining all eligible records. Duplicate coordinates are resolved by selecting one record per location, and records identified as spatial noise through density-based clustering are removed.

  * **Dataset 2:** Dataset excluding records that represent multiple locations. Duplicate coordinates are resolved by selecting one record per location, and records identified as spatial noise through density-based clustering are removed.

  * **Dataset 3:** Dataset excluding records with estimates computed under Hardy-Weinberg equilibrium assumptions. Duplicate coordinates are resolved by selecting one record per location, and records identified as spatial noise through density-based clustering are removed.

  * **Dataset 4:** Quality-filtered dataset retaining only quality-appraised records classified as low risk of bias. Duplicate coordinates are resolved by selecting one record per location, and records identified as spatial noise through density-based clustering are removed.

  * **Dataset 5:** Quality-filtered dataset retaining only quality-appraised records classified as low risk of bias, while preserving selected records from trusted sources that were not formally quality appraised. Duplicate coordinates are resolved by selecting one record per location, and records identified as spatial noise through density-based clustering are removed.

  * **Dataset 6:** Quality-filtered and geographically disaggregated dataset. Records are first restricted to quality-appraised observations classified as low risk of bias. ADM0 and ADM1 records are then disaggregated to higher-resolution, population-weighted locations before duplicate coordinates are resolved using predefined priority rules. Records identified as spatial noise through density-based clustering are removed.

  * **Dataset 7:** Quality-filtered dataset retaining only quality-appraised records classified as low risk of bias. Duplicate coordinates are resolved by selecting one record per location, while records identified as spatial noise through density-based clustering are retained.

  * **Dataset 8:** Final dataset used for modelling. This quality-filtered and geographically disaggregated dataset retains records identified as spatial noise through density-based clustering. Records are first restricted to quality-appraised observations classified as low risk of bias. ADM0 and ADM1 records are then disaggregated to higher-resolution, population-weighted locations before duplicate coordinates are resolved using predefined priority rules.

* **Duplicate-coordinate selection logic:**

  Records with identical longitude and latitude values are grouped together, and one record is retained from each duplicate-coordinate group using a sequential priority framework. The aim is to ensure that each location contributes only one observation to the modelling dataset while preserving the most informative and reliable record available for that coordinate.

  The selection rules are applied in the following order:

  1. **Original records are prioritised over disaggregated records, where applicable.**
     When `prefer_original = TRUE`, and an `origin_disaggregation` field is available, original records are retained over geographically disaggregated records at the same coordinate. If all records in the duplicate group are disaggregated, no record is removed at this step.

  2. **Single-location records are prioritised over multi-location records.**
     Records not flagged as multi-location entries are retained over records representing multiple locations. If all records in the group have the same multi-location status, no record is removed at this step.

  3. **Directly reported  estimates are prioritised over Hardy-Weinberg-derived estimates.**
     Records not marked as Hardy-Weinberg-computed estimates are retained over records recalculated under Hardy-Weinberg equilibrium assumptions. If all records in the group have the same recalculation status, no record is removed at this step.

  4. **Low-risk-of-bias records are prioritised.**
     Records classified as low risk of bias are retained over records with higher or unclassified risk of bias. If all records in the group have the same bias classification, no record is removed at this step.

  5. **Broad-population records are prioritised over ethnicity-specific records, unless ethnicity-specific records provide substantially greater sample size.**
     Records with `ethnicity_name` coded as `NULL` or `Mixed/Multiple` are treated as broad-population records. These are prioritised over ethnicity-specific records unless the combined sample size of the ethnicity-specific records is at least twice the combined sample size of the broad-population records.

  6. **Sample size is prioritised when one record is substantially larger.**
     The largest and second-largest unique sample sizes within the duplicate-coordinate group are compared. If the largest sample size is at least twice the second-largest sample size, the record with the largest sample size is retained.

  7. **Recency is prioritised when sample-size differences are not substantial.**
     If the largest sample size is not at least twice the second-largest sample size, the most recent record is retained.

  8. **Remaining ties are resolved by sample size.**
     If more than one record remains after the previous steps, the record with the largest sample size is retained.

  9. **Final unresolved ties are resolved by row order.**
     If records remain tied after all priority rules are applied, the first record in the grouped dataset is retained.

  This rule-based approach balances data quality, geographic specificity, representativeness, sample size, and recency when resolving duplicate-coordinate records.

### **B:** Model testing and cross-validation across dataset versions

This script tests and cross-validates models across the alternative dataset versions prepared in Script A. The aim is to determine which data preparation steps improve model performance and should be retained in the final modelling dataset.

### **C-F:** Final model testing and cross-validation, spatial prediction, burden estimation, and priority-site detection

These scripts apply the final modelling framework to generate spatial predictions, quantify prediction uncertainty, estimate haemoglobinopathy burden at administrative-unit level, validate model performance, and identify priority locations for future epidemiological surveillance. The differences across scripts C-F are annotated below:

1. **C:** Uses dataset restrcited by covariate availability and output is based on formula with IID effect + spatial effect + covariates.
2. **D:** Uses dataset restrcited by covariate availability and output is based on formula with IID effect + spatial effect.
3. **E:** Uses dataset not restrcited by covariate availability and retains data from geographically isolated countries; while output is based on formula with IID effect + spatial effect.
4. **F:** Uses dataset not restrcited by covariate availability and excludes data from geographically isolated countries; while output is based on formula with IID effect + spatial effect. [Selected version]
   
## Repository Structure

* **Input:** Contains the input files required for the modelling workflow. The input data for covariate raster construction are used in the workflow but excluded from this repository and must be obtained from their original sources:
   * `Administrative units`: https://www.geoboundaries.org/
   * `Healthcare site data`: https://www.healthsites.io/
   * `Consanguinity data`: https://consang.net/
   * `Religious diversity data`: https://dhsprogram.com/ & https://www.pewresearch.org/religion/2014/04/04/global-religious-diversity/#religious-diversity-by-region
   * `Ethnic fractionalization data`: https://dhsprogram.com/ & https://cadmus.eui.eu/entities/publication/eb27e2eb-d110-557a-80d5-be537cb8318c
   * `Population data`: https://hub.worldpop.org/geodata/summary?id=80032
   * `Urbanization data`: https://hub.worldpop.org/geodata/listing?id=146
   * `Malaria incidence data`: https://data.malariaatlas.org/maps?layers=Malaria:202508_Global_Pf_Incidence_Rate
   * `Service availability for managment and prevention of haemoglobinopathies`: https://www.ithanet.eu/db/ithamaps
   * `Counselling availability data`: https://www.ithanet.eu/db/ithamaps

* **Output:** Contains generated outputs from the modelling workflow, including processed datasets, model outputs, prediction surfaces, uncertainty estimates, burden estimates, validation results, and priority-site outputs. Outputs from covariate-raster construction are excluded from this repository.

* **Scripts:** Contains the `.Rmd` scripts used in the modelling workflow:
  * `Covariates.Rmd`: Covariate raster preparation.
  * `A.Rmd`: Dataset preparation and covariate selection.
  * `B.Rmd`: Model testing and cross-validation across dataset versions.
  * `C.Rmd`: Modelling framework output when covariates are utilized.
  * `D.Rmd`: Modelling framework output when covariates are not utilized [for direct comparison with output of C.Rmd].
  * `E.Rmd`: Modelling framework output when covariates are not utilized and their availability does not restrict the dataset [for direct comparison with output of F.Rmd].
  * `F.Rmd`: Final modelling framework output when covariates are not utilized, their availability does not restrict the dataset and geogrpahically isolated countries are excluded.

**Note:** The code below can be used to run multiple scripts automatically, but I suggest doing so only if you have inspected and have become familiar with the whole workflow and its requirements.
```
library(rmarkdown)
files <- c("path to file 1.Rmd", "path to file 2.Rmd") # Modify the list to include all file paths for the R Markdown files of interest to render consecutively.
for (f in files) {message("Rendering: ", f)
                  rmarkdown::render(input = f, clean = TRUE, envir = new.env())}
```
