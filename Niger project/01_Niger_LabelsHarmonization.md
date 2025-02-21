---
title: "WFP datasets wrangling - Niger : Labels harmonization"
author: "ganlea"
date: "2024-05-21"
output: 
  html_document:
    toc: true
    toc_depth: 3
    toc_float: true
    number_sections: true
    code_folding: show
    keep_md: true
---





```r
library(haven)
library(labelled) # for general functions to work with labelled data
library(tidyverse) # general wrangling
library(dplyr)
library(Hmisc)
library(gtsummary) # to demonstrate automatic use of variable labels in summary tables
library(readxl)
library(foreign)
library(sjPlot)
library(sjmisc)
library(sjlabelled) # for example efc data set with variable labels
library(stringr)
```




```r
rm(list = ls())
```


```r
chemin = "C:/Users/LENOVO/Desktop/isep3/semestre2/Projet statistique sous R/Projet Niger - f"
dir_input_data = paste0(chemin,"/output_data/NER")
dir_output_data = paste0(chemin, "/output_data/Common labels data/NER")
```


```r
NER_Harmonization_variables <- read_excel(paste0(dir_input_data,"/NER_Harmonization.xlsx"), 
    sheet = "variables_harmonization")
#View(NER_Harmonization_variables)

NER_Harmonization_description <- read_excel(paste0(dir_input_data,"/NER_Harmonization.xlsx"), 
    sheet = "description")
#View(NER_Harmonization_description)
```


```r
lst_test = NER_Harmonization_description$Name

for(i in 1:length(lst_test)) {                              # Head of for-loop
  assign(lst_test[i],                                   # Read and store data frames
         read_dta(paste0(dir_input_data,"/",lst_test[i],".dta")))
}
# pdm 2020 ea_2020
```


# Data consistency  Check

## Drop duplicated observations


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_ea_2019 <- Niger_ea_2019 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_ea_2020 <- Niger_ea_2020 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_ea_2021 <- Niger_ea_2021 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_pdm_2019 <- Niger_pdm_2019 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_pdm_2020 <- Niger_pdm_2020 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_pdm_2021 <- Niger_pdm_2021 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
Niger_pdm_2022 <- Niger_pdm_2022 %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
```


```r
# poser questions boucles 
for (j in 1:length(lst_test)){
         df <-  get(lst_test[j], envir = .GlobalEnv)
          df <- df %>% dplyr::distinct() %>% dplyr::filter(!is.na(ADMIN1Name))
         }
```

## Consent check

```r
for (j in 1:length(lst_test)) {
  df=  get(lst_test[j], envir = .GlobalEnv)
  cat(lst_test[j], " : ", unique(df$RESPConsent),"\n")
}
# lorsque Respconsent prend uniquement NA ou 1 la base a été traitée au préalable et contient uniquement les individus consentants
```

```r
table(Niger_pdm_2021$RESPConsent) # la modalité la plus fréquente corresponds à Oui
```


```r
Niger_pdm_2021 <- Niger_pdm_2021  %>% filter(RESPConsent == 1)
```

## ID Check 

```r
for (j in 1:length(lst_test)){
         df=  get(lst_test[j], envir = .GlobalEnv)
         cat(lst_test[j], " : ", length(unique(df$ID)), "//", nrow(df), "\n")
         }
```



```r
# poser question sur l'élimination des individus sans ID
Niger_baseline_2018 <- Niger_baseline_2018 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_ea_2019 <- Niger_ea_2019 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_ea_2020 <- Niger_ea_2020 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_ea_2021 <- Niger_ea_2021 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_pdm_2021 <- Niger_pdm_2021 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_pdm_2022 <- Niger_pdm_2022 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_pdm_2020 <- Niger_pdm_2020 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))
Niger_pdm_2019 <- Niger_pdm_2019 %>% dplyr::distinct(ID,.keep_all = TRUE) %>% dplyr::filter(!is.na(ID))

# Toutes nos bases de données ont des ID donc pas bseoin au préalable d'exclure certaines pour après créer de nouveaux ID.
```

# Administrative check

```r
#***** Niger_baseline_2018
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(YEAR= "2018" %>% 
                  structure(label = "Annee"))

Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(SURVEY= "Baseline" %>% 
                  structure(label = "Type d'enquête"))

Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    ADMIN1Name == 1 ~ "Agadez",
    ADMIN1Name == 2 ~ "Diffa",
    ADMIN1Name == 3 ~ "Dosso",
    ADMIN1Name == 4 ~ "Maradi",
    ADMIN1Name == 5 ~ "Tahoua",
    ADMIN1Name == 6 ~ "Tillabery",
    ADMIN1Name == 7 ~ "Zinder",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_baseline_2018$ADMIN1Name)))

# admin1CODE
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "BAGAROUA" ~ "Bagaroua",
    ADMIN2Name == "TESSAOUA" ~ "Tessaoua",
    ADMIN2Name == "MADAROUNFA" ~ "Madarounfa",
    ADMIN2Name == "OUALLAM" ~ "Ouallam",
    ADMIN2Name == "MAGARIA" ~ "Magaria",
    ADMIN2Name == "MIRRIAH" ~ "Mirriah",
    ADMIN2Name == "ABALAK" ~ "Abalak",
    ADMIN2Name == "BOUZA" ~ "Bouza",
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "MAYAHI" ~ "Mayahi",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    ADMIN2Name == "INGALL" ~ "In-Gall",
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGAL" ~ "In-Gall",
    ADMIN2Name == "GOUDOUMARIA" ~ "Goudoumaria",
    ADMIN2Name == "GOUDOUMARI" ~ "Goudoumaria",
    ADMIN2Name == "MAINE SOROA" ~ "Maine Soroa",
    ADMIN2Name == "MAINA SOROA" ~ "Maine Soroa",
    ADMIN2Name == "LOGA" ~ "Loga",
    ADMIN2Name == "madarounfa" ~ "Madarounfa",
    ADMIN2Name == "guidan roumdji" ~ "Guidan Roumji",
    ADMIN2Name == "dakoro" ~ "Dakoro",
    ADMIN2Name == "tessaoua" ~ "Tessaoua",
    ADMIN2Name == "tassara" ~ "Tassara",
    ADMIN2Name == "gotheye" ~ "Ouallam",
    ADMIN2Name == "tarka" ~ "Tarka",
    
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_baseline_2018$ADMIN2Name)))

# admin2CODE
Niger_baseline_2018 = Niger_baseline_2018 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```


```r
#***** Niger_ea_2019

Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(YEAR= "2019" %>% 
                  structure(label = "Annee"))

Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(SURVEY= "Enquête annuelle" %>% 
                  structure(label = "Type d'enquête"))

Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_ea_2019$ADMIN1Name <- as.character(Niger_ea_2019$ADMIN1Name)

Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    ADMIN1Name == "agadez" ~ "Agadez",
    ADMIN1Name == "diffa" ~ "Diffa",
    ADMIN1Name == "dosso" ~ "Dosso",
    ADMIN1Name == "maradi" ~ "Maradi",
    ADMIN1Name == "tahoua" ~ "Tahoua",
    ADMIN1Name == "tillaberi" ~ "Tillabery",
    ADMIN1Name == "zinder" ~ "Zinder",
    ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_ea_2019$ADMIN1Name)))

# admin1CODE
Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name

Niger_ea_2019$ADMIN2Name <- as.character(Niger_ea_2019$ADMIN2Name)

Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "BAGAROUA" ~ "Bagaroua",
    ADMIN2Name == "TESSAOUA" ~ "Tessaoua",
    ADMIN2Name == "MADAROUNFA" ~ "Madarounfa",
    ADMIN2Name == "OUALLAM" ~ "Ouallam",
    ADMIN2Name == "MAGARIA" ~ "Magaria",
    ADMIN2Name == "MIRRIAH" ~ "Mirriah",
    ADMIN2Name == "ABALAK" ~ "Abalak",
    ADMIN2Name == "BOUZA" ~ "Bouza",
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "MAYAHI" ~ "Mayahi",
    ADMIN2Name == "AGUIE" ~ "Aguie",
    ADMIN2Name == "DAKORO" ~ "Dakoro",
    ADMIN2Name == "DUNGASS" ~ "Dungass",
    ADMIN2Name == "GOUDOUMARIA" ~ "Goudoumaria",
    ADMIN2Name == "GUIDANROUMDJI" ~ "Guidan Roumji",
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGALL" ~ "In-Gall",
    ADMIN2Name == "LOGA" ~ "Loga",
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_ea_2019$ADMIN2Name)))

# admin2CODE
Niger_ea_2019 = Niger_ea_2019 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```


```r
#***** Niger_ea_2020

Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(YEAR= "2020" %>% 
                  structure(label = "Annee"))

Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(SURVEY= "Enquête annuelle" %>% 
                  structure(label = "Type d'enquête"))

Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_ea_2020$ADMIN1Name <- as.character(Niger_ea_2020$ADMIN1Name)

Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(ADMIN1Name=case_when(
   # ADMIN1Name == "AGADEZ" ~ "Agadez",
   # ADMIN1Name == "DIFFA" ~ "Diffa",
   # ADMIN1Name == "DOSSO" ~ "Dosso",
    ADMIN1Name == "maradi" ~ "Maradi",
    ADMIN1Name == "tahoua" ~ "Tahoua",
    #ADMIN1Name == "TILLABERI" ~ "Tillabery",
    ADMIN1Name == "zinder" ~ "Zinder",
    #ADMIN1Name == "NIAMEY" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_ea_2020$ADMIN1Name)))

# admin1CODE
Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(adm1_ocha=case_when(
    #ADMIN1Name == "Agadez" ~ "NE001",
    #ADMIN1Name == "Diffa" ~ "NE002",
    #ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    #ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    #ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_ea_2020$ADMIN2Name <- as.character(Niger_ea_2020$ADMIN2Name)

Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    #ADMIN2Name == "BAGAROUA" ~ "Bagaroua",
    ADMIN2Name == "tessaoua" ~ "Tessaoua",#
    ADMIN2Name == "madarounfa" ~ "Madarounfa",#
    #ADMIN2Name == "OUALLAM" ~ "Ouallam",
    ADMIN2Name == "magaria" ~ "Magaria",#
    ADMIN2Name == "mirriah" ~ "Mirriah",#
    ADMIN2Name == "abalak" ~ "Abalak",#
    ADMIN2Name == "bouza" ~ "Bouza",#
    ADMIN2Name == "gazaoua" ~ "Gazaoua",#
    ADMIN2Name == "mayahi" ~ "Mayahi",#
    ADMIN2Name == "aguie" ~ "Aguie",#
    ADMIN2Name == "dakoro" ~ "Dakoro",#
    ADMIN2Name == "dungass" ~ "Dungass",#
    #ADMIN2Name == "GOUDOUMARIA" ~ "Goudoumaria",
    ADMIN2Name == "guidan_roumdji" ~ "Guidan Roumji",#
    #ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    #ADMIN2Name == "INGALL" ~ "In-Gall",
    #ADMIN2Name == "LOGA" ~ "Loga",
    #ADMIN2Name == "TARKA" ~ "Tarka",
    #ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_ea_2020$ADMIN2Name)))

# admin2CODE
Niger_ea_2020 = Niger_ea_2020 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```



```r
#***** Niger_ea_2021

Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(YEAR= "2020" %>% 
                  structure(label = "Annee"))

Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(SURVEY= "Enquête annuelle" %>% 
                  structure(label = "Type d'enquête"))

Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    #ADMIN1Name == "agadez" ~ "Agadez",
    ADMIN1Name == "diffa" ~ "Diffa",
    ADMIN1Name == "dosso" ~ "Dosso",
    ADMIN1Name == "maradi" ~ "Maradi",
    ADMIN1Name == "tahoua" ~ "Tahoua",
    ADMIN1Name == "tillaberi" ~ "Tillabery",
    ADMIN1Name == "zinder" ~ "Zinder",
    #ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_ea_2021$ADMIN1Name)))

# admin1CODE
Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "bagaroua" ~ "Bagaroua",
    ADMIN2Name == "tessaoua" ~ "Tessaoua",
    ADMIN2Name == "madarounfa" ~ "Madarounfa",
    ADMIN2Name == "ouallam" ~ "Ouallam",
    ADMIN2Name == "magaria" ~ "Magaria",
    ADMIN2Name == "mirriah" ~ "Mirriah",
    ADMIN2Name == "abalak" ~ "Abalak",
    ADMIN2Name == "bouza" ~ "Bouza",
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "mayahi" ~ "Mayahi",
    ADMIN2Name == "aguie" ~ "Aguie",
    ADMIN2Name == "dakoro" ~ "Dakoro",
    ADMIN2Name == "dungass" ~ "Dungass",
    ADMIN2Name == "goudoumaria" ~ "Goudoumaria",
    ADMIN2Name == "guidan_roumdji" ~ "Guidan Roumji",
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGALL" ~ "In-Gall",
    ADMIN2Name == "loga" ~ "Loga",
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "diffa" ~ "Diffa",
    ADMIN2Name == "n_guigmi" ~ "N'Guigmi",
    ADMIN2Name == "filingue" ~ "Filingue",
    ADMIN2Name == "maine_soroa" ~ "Maine Soroa",
    ADMIN2Name == "goure" ~ "Goure",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_ea_2021$ADMIN2Name)))

# admin2CODE
Niger_ea_2021 = Niger_ea_2021 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Diffa" ~ "NE002002",
    ADMIN2Name == "N'Guigmi" ~ "NE002006",
    ADMIN2Name == "Filingue" ~ "NE006005",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Goure" ~ "NE007003",
    ADMIN2Name == "Matameye" ~ "NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```


```r
#***** Niger_pdm_2021

Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(YEAR= "2021" %>% 
                  structure(label = "Annee"))

Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(SURVEY= "PDM" %>% 
                  structure(label = "Type d'enquête"))

Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    #ADMIN1Name == "agadez" ~ "Agadez",
    ADMIN1Name == "diffa" ~ "Diffa",
    ADMIN1Name == "dosso" ~ "Dosso",
    ADMIN1Name == "maradi" ~ "Maradi",#
    ADMIN1Name == "tahoua" ~ "Tahoua",#
    ADMIN1Name == "tillaberi" ~ "Tillabery",
    ADMIN1Name == "zinder" ~ "Zinder",#
    #ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_pdm_2021$ADMIN1Name)))

# admin1CODE
Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "bagaroua" ~ "Bagaroua",
    ADMIN2Name == "tessaoua" ~ "Tessaoua",#
    ADMIN2Name == "madarounfa" ~ "Madarounfa",#
    ADMIN2Name == "ouallam" ~ "Ouallam",
    ADMIN2Name == "magaria" ~ "Magaria",#
    ADMIN2Name == "mirriah" ~ "Mirriah",#
    ADMIN2Name == "abalak" ~ "Abalak",#
    ADMIN2Name == "bouza" ~ "Bouza",#
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "mayahi" ~ "Mayahi",#
    ADMIN2Name == "aguie" ~ "Aguie",#
    ADMIN2Name == "dakoro" ~ "Dakoro",#
    ADMIN2Name == "dungass" ~ "Dungass",#
    ADMIN2Name == "goudoumaria" ~ "Goudoumaria",
    ADMIN2Name == "guidan_roumdji" ~ "Guidan Roumji",#
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGALL" ~ "In-Gall",
    ADMIN2Name == "loga" ~ "Loga",
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "diffa" ~ "Diffa",
    ADMIN2Name == "n_guigmi" ~ "N'Guigmi",
    ADMIN2Name == "filingue" ~ "Filingue",
    ADMIN2Name == "maine_soroa" ~ "Maine Soroa",
    ADMIN2Name == "goure" ~ "Goure",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_pdm_2021$ADMIN2Name)))

# admin2CODE
Niger_pdm_2021 = Niger_pdm_2021 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Diffa" ~ "NE002002",
    ADMIN2Name == "N'Guigmi" ~ "NE002006",
    ADMIN2Name == "Filingue" ~ "NE006005",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Goure" ~ "NE007003",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```


```r
#***** Niger_pdm_2022

Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(YEAR= "2022" %>% 
                  structure(label = "Annee"))

Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(SURVEY= "PDM" %>% 
                  structure(label = "Type d'enquête"))

Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    ADMIN1Name == "AGADEZ" ~ "Agadez",
    ADMIN1Name == "DIFFA" ~ "Diffa",
    ADMIN1Name == "DOSSO" ~ "Dosso",
    ADMIN1Name == "MARADI" ~ "Maradi",#
    ADMIN1Name == "TAHOUA" ~ "Tahoua",#
    ADMIN1Name == "TILLABERI" ~ "Tillabery",
    ADMIN1Name == "ZINDER" ~ "Zinder",#
    #ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_pdm_2022$ADMIN1Name)))

# admin1CODE
Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "BAGAROUA" ~ "Bagaroua",#
    ADMIN2Name == "TESSAOUA" ~ "Tessaoua",#
    ADMIN2Name == "MADAROUNFA" ~ "Madarounfa",#
    ADMIN2Name == "OUALLAM" ~ "Ouallam",#
    ADMIN2Name == "MAGARIA" ~ "Magaria",#
    ADMIN2Name == "MIRRIAH" ~ "Mirriah",#
    ADMIN2Name == "ABALAK" ~ "Abalak",#
    ADMIN2Name == "BOUZA" ~ "Bouza",#
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "MAYAHI" ~ "Mayahi",#
    ADMIN2Name == "AGUIE" ~ "Aguie",#
    ADMIN2Name == "DAKORO" ~ "Dakoro",#
    ADMIN2Name == "DUNGASS" ~ "Dungass",#
    ADMIN2Name == "GOUDOUMARIA" ~ "Goudoumaria",#
    ADMIN2Name == "GUIDAN-ROUMDJI" ~ "Guidan Roumji",#
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGALL" ~ "In-Gall",#
    ADMIN2Name == "LOGA" ~ "Loga",#
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "DIFFA" ~ "Diffa",#
    ADMIN2Name == "n_guigmi" ~ "N'Guigmi",
    ADMIN2Name == "filingue" ~ "Filingue",
    ADMIN2Name == "MAINE-SOROA" ~ "Maine Soroa",#
    ADMIN2Name == "GOURE" ~ "Goure",#
    ADMIN2Name == "DAMAGARAM TAKAYA" ~ "Damagaram Takaya",#
    ADMIN2Name == "AYAROU" ~ "Ayerou",#
    ADMIN2Name == "GOTHEYE" ~ "Gotheye",#
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_pdm_2022$ADMIN2Name)))

# admin2CODE
Niger_pdm_2022 = Niger_pdm_2022 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Diffa" ~ "NE002002",
    ADMIN2Name == "N'Guigmi" ~ "NE002006",
    ADMIN2Name == "Filingue" ~ "NE006005",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Goure" ~ "NE007003",
    ADMIN2Name == "Damagaram Takaya" ~ "NE007001",
    ADMIN2Name == "Ayerou" ~ "NE006002",
    ADMIN2Name == "Gotheye" ~ "NE006006",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```



```r
#***** Niger_pdm_2020

Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(YEAR= "2020" %>% 
                  structure(label = "Annee"))

Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(SURVEY= "PDM" %>% 
                  structure(label = "Type d'enquête"))

Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_pdm_2020$ADMIN1Name <- as.character(Niger_pdm_2020$ADMIN1Name)
Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    ADMIN1Name == "AGADEZ" ~ "Agadez",
    ADMIN1Name == "DIFFA" ~ "Diffa",
    ADMIN1Name == "DOSSO" ~ "Dosso",
    ADMIN1Name == "Maradi" ~ "Maradi",#
    ADMIN1Name == "Tahoua" ~ "Tahoua",#
    ADMIN1Name == "TILLABERI" ~ "Tillabery",
    ADMIN1Name == "Zinder" ~ "Zinder",#
    #ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_pdm_2020$ADMIN1Name)))

# admin1CODE
Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_pdm_2020$ADMIN2Name <- as.character(Niger_pdm_2020$ADMIN2Name)
Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "BAGAROUA" ~ "Bagaroua",
    ADMIN2Name == "Tessaoua" ~ "Tessaoua",#
    ADMIN2Name == "Madarounfa" ~ "Madarounfa",#
    ADMIN2Name == "ouallam" ~ "Ouallam",
    ADMIN2Name == "Magaria" ~ "Magaria",#
    ADMIN2Name == "Mirriah" ~ "Mirriah",#
    ADMIN2Name == "Abalak" ~ "Abalak",#
    ADMIN2Name == "Bouza" ~ "Bouza",#
    ADMIN2Name == "Gazaoua" ~ "Gazaoua",#
    ADMIN2Name == "Mayahi" ~ "Mayahi",#
    ADMIN2Name == "AGUIE" ~ "Aguie",
    ADMIN2Name == "Dakoro" ~ "Dakoro",#
    ADMIN2Name == "dungass" ~ "Dungass",
    ADMIN2Name == "goudoumaria" ~ "Goudoumaria",
    ADMIN2Name == "Guidan Roumdji" ~ "Guidan Roumji",#
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",
    ADMIN2Name == "INGALL" ~ "In-Gall",
    ADMIN2Name == "loga" ~ "Loga",
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",
    ADMIN2Name == "DIFFA" ~ "Diffa",
    ADMIN2Name == "n_guigmi" ~ "N'Guigmi",
    ADMIN2Name == "filingue" ~ "Filingue",
    ADMIN2Name == "MAINE-SOROA" ~ "Maine Soroa",
    ADMIN2Name == "goure" ~ "Goure",
    ADMIN2Name == "DAMAGARAM TAKAYA" ~ "Damagaram Takaya",
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_pdm_2020$ADMIN2Name)))

# admin2CODE
Niger_pdm_2020 = Niger_pdm_2020 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Diffa" ~ "NE002002",
    ADMIN2Name == "N'Guigmi" ~ "NE002006",
    ADMIN2Name == "Filingue" ~ "NE006005",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Goure" ~ "NE007003",
    ADMIN2Name == "Damagaram Takaya" ~ "NE007001",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Tarka" ~ "NE007009",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```




```r
#***** Niger_pdm_2019

Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(YEAR= "2019" %>% 
                  structure(label = "Annee"))

Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(SURVEY= "PDM" %>% 
                  structure(label = "Type d'enquête"))

Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(ADMIN0Name= "Niger" %>% 
                  structure(label = "Nom du pays"))
Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(adm0_ocha= "NE" %>% 
                  structure(label = "Admin 0 ID"))

# admin1Name
Niger_pdm_2019$ADMIN1Name <- as.character(Niger_pdm_2019$ADMIN1Name)
Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(ADMIN1Name=case_when(
    ADMIN1Name == "agadez" ~ "Agadez",
    ADMIN1Name == "diffa" ~ "Diffa",
    ADMIN1Name == "dosso" ~ "Dosso",
    ADMIN1Name == "maradi" ~ "Maradi",
    ADMIN1Name == "tahoua" ~ "Tahoua",
    ADMIN1Name == "tillaberi" ~ "Tillabery",
    ADMIN1Name == "zinder" ~ "Zinder",
    #ADMIN1Name == "niamey" ~ "Niamey",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = label(Niger_pdm_2019$ADMIN1Name)))

# admin1CODE
Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(adm1_ocha=case_when(
    ADMIN1Name == "Agadez" ~ "NE001",
    ADMIN1Name == "Diffa" ~ "NE002",
    ADMIN1Name == "Dosso" ~ "NE003",
    ADMIN1Name == "Maradi" ~ "NE004",
    ADMIN1Name == "Tahoua" ~ "NE005",
    ADMIN1Name == "Tillabery" ~ "NE006",
    ADMIN1Name == "Zinder" ~ "NE007",
    ADMIN1Name == "Niamey" ~ "NE008",
    
    TRUE ~ as.character(ADMIN1Name)
  )%>% 
    structure(label = "Admin 1 ID"))

# admin2Name
Niger_pdm_2019$ADMIN2Name <- as.character(Niger_pdm_2019$ADMIN2Name)

Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(ADMIN2Name=case_when(
    ADMIN2Name == "BAGAROUA" ~ "Bagaroua",#
    ADMIN2Name == "TESSAOUA" ~ "Tessaoua",#
    ADMIN2Name == "MADAROUNFA" ~ "Madarounfa",#
    ADMIN2Name == "OUALLAM" ~ "Ouallam",#
    ADMIN2Name == "MAGARIA" ~ "Magaria",#
    ADMIN2Name == "MIRRIAH" ~ "Mirriah",#
    ADMIN2Name == "ABALAK" ~ "Abalak",#
    ADMIN2Name == "BOUZA" ~ "Bouza",#
    ADMIN2Name == "GAZAOUA" ~ "Gazaoua",
    ADMIN2Name == "MAYAHI" ~ "Mayahi",#
    ADMIN2Name == "AGUIE" ~ "Aguie",#
    ADMIN2Name == "DAKORO" ~ "Dakoro",#
    ADMIN2Name == "DUNGASS" ~ "Dungass",#
    ADMIN2Name == "GOUDOUMARIA" ~ "Goudoumaria",#
    ADMIN2Name == "GUIDANROUMDJI" ~ "Guidan Roumji",#
    ADMIN2Name == "IFEROUANE" ~ "Iferouane",#
    ADMIN2Name == "INGALL" ~ "In-Gall",#
    ADMIN2Name == "LOGA" ~ "Loga",#
    ADMIN2Name == "TARKA" ~ "Tarka",
    ADMIN2Name == "TASSARA" ~ "Tassara",#
    ADMIN2Name == "DIFFA" ~ "Diffa",
    ADMIN2Name == "n_guigmi" ~ "N'Guigmi",
    ADMIN2Name == "filingue" ~ "Filingue",
    ADMIN2Name == "MAINE-SOROA" ~ "Maine Soroa",
    ADMIN2Name == "goure" ~ "Goure",
    ADMIN2Name == "DAMAGARAM TAKAYA" ~ "Damagaram",
    ADMIN2Name == "TARKA" ~ "Tarka",#
    ADMIN2Name == "KANTCHE" ~ "Matameye",
    ADMIN2Name == "Kantche" ~ "Matameye",
    ADMIN2Name == "kantche" ~ "Matameye",
    ADMIN2Name == "Ville de Maradi" ~ "Madarounfa",
    ADMIN2Name == "belbedji" ~ "Tarka",
    ADMIN2Name == "BELBEDJI" ~ "Tarka",
    ADMIN2Name == "Belbedji" ~ "Tarka",
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = label(Niger_pdm_2019$ADMIN2Name)))

# admin2CODE
Niger_pdm_2019 = Niger_pdm_2019 %>%
  dplyr::mutate(adm2_ocha=case_when(
    ADMIN2Name == "Bagaroua" ~ "NE005002",
    ADMIN2Name == "Tessaoua" ~ "NE004008",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    ADMIN2Name == "Ouallam" ~ "NE006008",
    ADMIN2Name == "Magaria" ~ "NE007004",
    ADMIN2Name == "Mirriah" ~ "NE007006",
    ADMIN2Name == "Abalak" ~ "NE005001",
    ADMIN2Name == "Bouza" ~ "NE005004",
    ADMIN2Name == "Gazaoua" ~ "NE004004",
    ADMIN2Name == "Mayahi" ~ "NE004007",
    ADMIN2Name == "Aguie" ~ "NE004001",
    ADMIN2Name == "Dakoro" ~ "NE004003",
    ADMIN2Name == "Dungass" ~ "NE007002",
    ADMIN2Name == "Goudoumaria" ~ "NE002003",
    ADMIN2Name == "Guidan Roumji" ~ "NE004005",
    ADMIN2Name == "Iferouane" ~ "NE001004",
    ADMIN2Name == "In-Gall" ~ "NE001005",
    ADMIN2Name == "Loga" ~ "NE003007",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Tassara" ~ "NE005010",
    ADMIN2Name == "Diffa" ~ "NE002002",
    ADMIN2Name == "N'Guigmi" ~ "NE002006",
    ADMIN2Name == "Filingue" ~ "NE006005",
    ADMIN2Name == "Maine Soroa" ~ "NE002004",
    ADMIN2Name == "Goure" ~ "NE007003",
    ADMIN2Name == "Tarka" ~ "NE007009",
    ADMIN2Name == "Matameye" ~"NE007005",
    ADMIN2Name == "Madarounfa" ~ "NE004006",
    
    
    TRUE ~ as.character(ADMIN2Name)
  )%>% 
    structure(label = "Admin 2 ID"))
```

# househouls information 
## HHSize

```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T) 
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-1.png)<!-- -->

```r
  expss::val_lab(Niger_baseline_2018$HHSize)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-2.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2019$HHSize)
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-3.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2020$HHSize)
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-4.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2021$HHSize)
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-5.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2021$HHSize)
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-6.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2022$HHSize)
  
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-7.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2020$HHSize)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,HHSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/HHSize-8.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2019$HHSize)
  
unique(Niger_baseline_2018$HHSize)
unique(Niger_ea_2019$HHSize)
unique(Niger_ea_2020$HHSize)
unique(Niger_ea_2021$HHSize)
unique(Niger_pdm_2021$HHSize)
unique(Niger_pdm_2022$HHSize)
unique(Niger_pdm_2020$HHSize)
unique(Niger_pdm_2019$HHSize)

Niger_pdm_2021$HHSize <- as.numeric(Niger_pdm_2021$HHSize)
```

## HHEdu 





# Indicator checks

## Food consumption score

The Food consumption Score (FCS) is an index that aggregates household-level data on the diversity and frequency of food groups consumed over the last 7 days. It is then weighted according to the relative nutritional value of the consumed food groups. Food groups containing nutritionally dense foods (e.g. animal based products) are given greater weight than those containing less nutritional value (e.g. tubers) as follows: (main staples:2, pulses:3, vegetables:1, fruit:1, meat or fish:4, milk:4, sugar:0.5, oil:0.5).

### FCS : Céréales et tubercules


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T) 
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-1.png)<!-- -->

```r
  expss::val_lab(Niger_baseline_2018$FCSStap)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-2.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2019$FCSStap)
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-3.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2020$FCSStap)
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-4.png)<!-- -->

```r
  expss::val_lab(Niger_ea_2021$FCSStap)
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-5.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2021$FCSStap)
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-6.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2022$FCSStap)
  
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate(FCSStap = as.integer(FCSStap)) 
  expss::val_lab(Niger_pdm_2020$FCSStap)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSStap,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple-8.png)<!-- -->

```r
  expss::val_lab(Niger_pdm_2019$FCSStap)
```

### FCS : Céréales et tubercules - Sources


```r
# Codes d’acquisition des aliments 
# 1 = Production propre (récoltes, élevage) ; 2 = Pêche / Chasse ; 3 = Cueillette ; 4 = Prêts ; 5 = Marché (achat avec des espèces) ; 6 = Marché (achat à crédit) ;
# 7 = Mendicité ; 8 = Troc travail ou biens contre des aliments ; 9 = Dons (aliments) de membres de la famille ou d’amis ; 10 = Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc.
```



```r
typeof(Niger_baseline_2018$FCSStapSRf)
expss::val_lab(Niger_baseline_2018$FCSStapSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSStapSRf <- labelled::labelled(Niger_baseline_2018$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSStapSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSStapSRf)
expss::val_lab(Niger_ea_2019$FCSStapSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSStapSRf <- labelled::labelled(Niger_ea_2019$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSStapSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSStapSRf)
expss::val_lab(Niger_ea_2020$FCSStapSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSStapSRf <- labelled::labelled(Niger_ea_2020$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSStapSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSStapSRf)
expss::val_lab(Niger_ea_2021$FCSStapSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_, "16"=NA_real_)

Niger_ea_2021$FCSStapSRf <- labelled::labelled(Niger_ea_2021$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSStapSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSStapSRf)
expss::val_lab(Niger_pdm_2021$FCSStapSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSStapSRf <- labelled::labelled(Niger_pdm_2021$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSStapSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSStapSRf)
expss::val_lab(Niger_pdm_2022$FCSStapSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSStapSRf <- labelled::labelled(Niger_pdm_2022$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSStapSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2022-2.png)<!-- -->

```r
unique(Niger_pdm_2020$FCSStapSRf)
expss::val_lab(Niger_pdm_2020$FCSStapSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1, "7" =NA_real_)

Niger_pdm_2020$FCSStapSRf <- labelled::labelled(Niger_pdm_2020$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSStapSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSStapSRf)
expss::val_lab(Niger_pdm_2019$FCSStapSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSStapSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_pdm_2019$FCSStapSRf <- labelled::labelled(Niger_pdm_2019$FCSStapSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSStapSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSStapSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Staple Source Niger_pdm_2019-2.png)<!-- -->


### FCS : Légumineuses


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate(FCSPulse = as.integer(FCSPulse)) 
  expss::val_lab(Niger_pdm_2020$FCSPulse)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulse,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses-8.png)<!-- -->
### FCS : Légumineuses - Sources



```r
unique(Niger_baseline_2018$FCSPulseSRf)
expss::val_lab(Niger_baseline_2018$FCSPulseSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSPulseSRf <- labelled::labelled(Niger_baseline_2018$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSPulseSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSPulseSRf)
expss::val_lab(Niger_ea_2019$FCSPulseSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSPulseSRf <- labelled::labelled(Niger_ea_2019$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSPulseSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSPulseSRf)
expss::val_lab(Niger_ea_2020$FCSPulseSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSPulseSRf <- labelled::labelled(Niger_ea_2020$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSPulseSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSPulseSRf)
expss::val_lab(Niger_ea_2021$FCSPulseSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSPulseSRf <- labelled::labelled(Niger_ea_2021$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSPulseSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSPulseSRf)
expss::val_lab(Niger_pdm_2021$FCSPulseSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSPulseSRf <- labelled::labelled(Niger_pdm_2021$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSPulseSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSPulseSRf)
expss::val_lab(Niger_pdm_2022$FCSPulseSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSPulseSRf <- labelled::labelled(Niger_pdm_2022$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSPulseSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2022-2.png)<!-- -->

```r
unique(Niger_pdm_2020$FCSPulseSRf)
expss::val_lab(Niger_pdm_2020$FCSPulseSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSPulseSRf <- labelled::labelled(Niger_pdm_2020$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSPulseSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSPulseSRf)
expss::val_lab(Niger_pdm_2019$FCSPulseSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSPulseSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSPulseSRf <- labelled::labelled(Niger_pdm_2019$FCSPulseSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSPulseSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSPulseSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Pulses Source Niger_pdm_2019-2.png)<!-- -->
### FCS : Lait et produits laitiers


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSDairy = as.integer(FCSDairy))
  expss::var_lab(Niger_pdm_2020$FCSDairy)

Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairy,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys-8.png)<!-- -->
### FCS : Dairys - Sources



```r
unique(Niger_baseline_2018$FCSDairySRf)
expss::val_lab(Niger_baseline_2018$FCSDairySRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSDairySRf <- labelled::labelled(Niger_baseline_2018$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSDairySRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSDairySRf)
expss::val_lab(Niger_ea_2019$FCSDairySRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSDairySRf <- labelled::labelled(Niger_ea_2019$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSDairySRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSDairySRf)
expss::val_lab(Niger_ea_2020$FCSDairySRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSDairySRf <- labelled::labelled(Niger_ea_2020$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSDairySRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSDairySRf)
expss::val_lab(Niger_ea_2021$FCSDairySRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSDairySRf <- labelled::labelled(Niger_ea_2021$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSDairySRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSDairySRf)
expss::val_lab(Niger_pdm_2021$FCSDairySRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSDairySRf <- labelled::labelled(Niger_pdm_2021$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSDairySRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSDairySRf)
expss::val_lab(Niger_pdm_2022$FCSDairySRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSDairySRf <- labelled::labelled(Niger_pdm_2022$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSDairySRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2022-2.png)<!-- -->

```r
unique(Niger_pdm_2020$FCSDairySRf)
expss::val_lab(Niger_pdm_2020$FCSDairySRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSDairySRf <- labelled::labelled(Niger_pdm_2020$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSDairySRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2020-2.png)<!-- -->


```r
unique(Niger_pdm_2019$FCSDairySRf)
expss::val_lab(Niger_pdm_2019$FCSDairySRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSDairySRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSDairySRf <- labelled::labelled(Niger_pdm_2019$FCSDairySRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSDairySRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSDairySRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Dairys Source Niger_pdm_2019-2.png)<!-- -->
### FCS: Viande, poisson et oeufs


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSPr = as.integer(FCSPr))
  expss::var_lab(Niger_pdm_2020$FCSPr)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPr,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs-8.png)<!-- -->
### FCS: Viande, poisson et oeufs : Source


```r
unique(Niger_baseline_2018$FCSPrSRf)
expss::val_lab(Niger_baseline_2018$FCSPrSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSPrSRf <- labelled::labelled(Niger_baseline_2018$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSPrSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSPrSRf)
expss::val_lab(Niger_ea_2019$FCSPrSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSPrSRf <- labelled::labelled(Niger_ea_2019$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSPrSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSPrSRf)
expss::val_lab(Niger_ea_2020$FCSPrSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSPrSRf <- labelled::labelled(Niger_ea_2020$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSPrSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSPrSRf)
expss::val_lab(Niger_ea_2021$FCSPrSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSPrSRf <- labelled::labelled(Niger_ea_2021$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSPrSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSPrSRf)
expss::val_lab(Niger_pdm_2021$FCSPrSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSPrSRf <- labelled::labelled(Niger_pdm_2021$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSPrSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSPrSRf)
expss::val_lab(Niger_pdm_2022$FCSPrSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSPrSRf <- labelled::labelled(Niger_pdm_2022$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSPrSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSPrSRf)
expss::val_lab(Niger_pdm_2020$FCSPrSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSPrSRf <- labelled::labelled(Niger_pdm_2020$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSPrSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSPrSRf)
expss::val_lab(Niger_pdm_2019$FCSPrSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSPrSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSPrSRf <- labelled::labelled(Niger_pdm_2019$FCSPrSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSPrSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSPrSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Prs Source Niger_pdm_2019-2.png)<!-- -->

### FCS : Chair/viande rouge 


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSPrMeatF = as.integer(FCSPrMeatF))
  expss::var_lab(Niger_pdm_2020$FCSPrMeatF)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatF,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatF-8.png)<!-- -->


### FCS : Viande d'organe, telle que: (foie, reins, coeur et / ou autres abats) 


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSPrMeatO = as.integer(FCSPrMeatO))
  expss::var_lab(Niger_pdm_2020$FCSPrMeatO)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrMeatO,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrMeatO-8.png)<!-- -->
### FCS : Poissons et coquillage, tels que: (poissons, y compris le thon en conserve, les escargots et / ou d'autres fruits de mer remplacer par des exemples localement pertinents )


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSPrFish = as.integer(FCSPrFish))
  expss::var_lab(Niger_pdm_2020$FCSPrFish)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrFish,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrFish-8.png)<!-- -->

### FCS : Oeufs


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSPrEgg = as.integer(FCSPrEgg))
  expss::var_lab(Niger_pdm_2020$FCSPrEgg)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSPrEgg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS PrEgg-8.png)<!-- -->
### FCS : Légumes et feuilles , tels que : (épinards, oignons, tomates, carottes, poivrons, haricots verts, laitue, etc)


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSVeg = as.integer(FCSVeg))
  expss::var_lab(Niger_pdm_2020$FCSVeg)

Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVeg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Veg-8.png)<!-- -->

### FCS : Légumes et feuilles , tels que : (épinards, oignons, tomates, carottes, poivrons, haricots verts, laitue, etc) - Sources


```r
unique(Niger_baseline_2018$FCSVegSRf)
expss::val_lab(Niger_baseline_2018$FCSVegSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSVegSRf <- labelled::labelled(Niger_baseline_2018$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSVegSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSVegSRf)
expss::val_lab(Niger_ea_2019$FCSVegSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSVegSRf <- labelled::labelled(Niger_ea_2019$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSVegSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSVegSRf)
expss::val_lab(Niger_ea_2020$FCSVegSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSVegSRf <- labelled::labelled(Niger_ea_2020$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSVegSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSVegSRf)
expss::val_lab(Niger_ea_2021$FCSVegSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSVegSRf <- labelled::labelled(Niger_ea_2021$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSVegSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSVegSRf)
expss::val_lab(Niger_pdm_2021$FCSVegSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSVegSRf <- labelled::labelled(Niger_pdm_2021$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSVegSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSVegSRf)
expss::val_lab(Niger_pdm_2022$FCSVegSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSVegSRf <- labelled::labelled(Niger_pdm_2022$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSVegSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSVegSRf)
expss::val_lab(Niger_pdm_2020$FCSVegSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSVegSRf <- labelled::labelled(Niger_pdm_2020$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSVegSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSVegSRf)
expss::val_lab(Niger_pdm_2019$FCSVegSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSVegSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSVegSRf <- labelled::labelled(Niger_pdm_2019$FCSVegSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSVegSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSVegSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSVeg Source Niger_pdm_2019-2.png)<!-- -->
### FCS : Légumes oranges (légumes riches en Vitamine A)


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSVegOrg = as.integer(FCSVegOrg))
  expss::var_lab(Niger_pdm_2020$FCSVegOrg)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegOrg,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSOrg-8.png)<!-- -->

### FCS : Légumes à feuilles vertes,,  tels que : ( épinards, brocoli, amarante et/ou autres feuilles vert foncé , feuilles de manioc )


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSVegGre = as.integer(FCSVegGre))
  expss::var_lab(Niger_pdm_2020$FCSVegGre)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSVegGre,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSvegGre-8.png)<!-- -->

### FCS : Fruits,  tels que : (banane, pomme, citron, mangue, papaye, abricot, pêche, etc)


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSFruit = as.integer(FCSFruit))
  expss::var_lab(Niger_pdm_2020$FCSFruit)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruit,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit-8.png)<!-- -->

### FCS : Fruits,  tels que : (banane, pomme, citron, mangue, papaye, abricot, pêche, etc) - Sources


```r
unique(Niger_baseline_2018$FCSFruitSRf)
expss::val_lab(Niger_baseline_2018$FCSFruitSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSFruitSRf <- labelled::labelled(Niger_baseline_2018$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSFruitSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSFruitSRf)
expss::val_lab(Niger_ea_2019$FCSFruitSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSFruitSRf <- labelled::labelled(Niger_ea_2019$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSFruitSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSFruitSRf)
expss::val_lab(Niger_ea_2020$FCSFruitSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit  Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSFruitSRf <- labelled::labelled(Niger_ea_2020$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSFruitSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit  Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSFruitSRf)
expss::val_lab(Niger_ea_2021$FCSFruitSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSFruitSRf <- labelled::labelled(Niger_ea_2021$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSFruitSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSFruitSRf)
expss::val_lab(Niger_pdm_2021$FCSFruitSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSFruitSRf <- labelled::labelled(Niger_pdm_2021$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSFruitSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSFruitSRf)
expss::val_lab(Niger_pdm_2022$FCSFruitSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSFruitSRf <- labelled::labelled(Niger_pdm_2022$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSFruitSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSFruitSRf)
expss::val_lab(Niger_pdm_2020$FCSFruitSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSFruitSRf <- labelled::labelled(Niger_pdm_2020$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSFruitSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSFruitSRf)
expss::val_lab(Niger_pdm_2019$FCSFruitSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSFruitSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSFruitSRf <- labelled::labelled(Niger_pdm_2019$FCSFruitSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSFruitSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSFruitSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFruit Source Niger_pdm_2019-2.png)<!-- -->
### FCS : Huile/matières grasses/beurre: tels que (huile végétale, huile de palme, beurre de karité, margarine, autres huiles / matières grasses)


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSFat = as.integer(FCSFat))
  expss::var_lab(Niger_pdm_2020$FCSFat)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSFat,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat-8.png)<!-- -->
### FCS : Huile/matières grasses/beurre: tels que (huile végétale, huile de palme, beurre de karité, margarine, autres huiles / matières grasses) - Sources



```r
unique(Niger_baseline_2018$FCSFatSRf)
expss::val_lab(Niger_baseline_2018$FCSFatSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSFatSRf <- labelled::labelled(Niger_baseline_2018$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSFatSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSFatSRf)
expss::val_lab(Niger_ea_2019$FCSFatSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSFatSRf <- labelled::labelled(Niger_ea_2019$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSFatSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSFatSRf)
expss::val_lab(Niger_ea_2020$FCSFatSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSFatSRf <- labelled::labelled(Niger_ea_2020$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSFatSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSFatSRf)
expss::val_lab(Niger_ea_2021$FCSFatSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSFatSRf <- labelled::labelled(Niger_ea_2021$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSFatSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSFatSRf)
expss::val_lab(Niger_pdm_2021$FCSFatSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSFatSRf <- labelled::labelled(Niger_pdm_2021$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSFatSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSFatSRf)
expss::val_lab(Niger_pdm_2022$FCSFatSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSFatSRf <- labelled::labelled(Niger_pdm_2022$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSFatSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSFatSRf)
expss::val_lab(Niger_pdm_2020$FCSFatSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSFatSRf <- labelled::labelled(Niger_pdm_2020$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSFatSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSFatSRf)
expss::val_lab(Niger_pdm_2019$FCSFatSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSFatSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSFatSRf <- labelled::labelled(Niger_pdm_2019$FCSFatSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSFatSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSFatSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSFat Source Niger_pdm_2019-2.png)<!-- -->
### FSC : Sucre ou sucreries, tels que (sucre, miel, confiture, gâteau, bonbons, biscuits, viennoiserie et autres produits sucrés (boissons sucrées)  )



```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSSugar = as.integer(FCSSugar))
  expss::var_lab(Niger_pdm_2020$FCSSugar)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugar,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar-8.png)<!-- -->

### FSC : Sucre ou sucreries, tels que (sucre, miel, confiture, gâteau, bonbons, biscuits, viennoiserie et autres produits sucrés (boissons sucrées)  ) - Sources


```r
unique(Niger_baseline_2018$FCSSugarSRf)
expss::val_lab(Niger_baseline_2018$FCSSugarSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSSugarSRf <- labelled::labelled(Niger_baseline_2018$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSSugarSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSSugarSRf)
expss::val_lab(Niger_ea_2019$FCSSugarSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSSugarSRf <- labelled::labelled(Niger_ea_2019$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSSugarSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2019-2.png)<!-- -->

```r
unique(Niger_ea_2020$FCSSugarSRf)
expss::val_lab(Niger_ea_2020$FCSSugarSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSSugarSRf <- labelled::labelled(Niger_ea_2020$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSSugarSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSSugarSRf)
expss::val_lab(Niger_ea_2021$FCSSugarSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSSugarSRf <- labelled::labelled(Niger_ea_2021$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSSugarSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSSugarSRf)
expss::val_lab(Niger_pdm_2021$FCSSugarSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSSugarSRf <- labelled::labelled(Niger_pdm_2021$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSSugarSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSSugarSRf)
expss::val_lab(Niger_pdm_2022$FCSSugarSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSSugarSRf <- labelled::labelled(Niger_pdm_2022$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSSugarSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSSugarSRf)
expss::val_lab(Niger_pdm_2020$FCSSugarSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSSugarSRf <- labelled::labelled(Niger_pdm_2020$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSSugarSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSSugarSRf)
expss::val_lab(Niger_pdm_2019$FCSSugarSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSSugarSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSSugarSRf <- labelled::labelled(Niger_pdm_2019$FCSSugarSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSSugarSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSSugarSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSSugar Source Niger_pdm_2019-2.png)<!-- -->
### FCS : Condiments/épices: tels que (thé, café/cacao, sel, ail, épices, levure/levure chimique, tomate/sauce, viande ou poisson comme condiment, condiments incluant des petites quantités de lait/thé, café.) 



```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (FCSCond = as.integer(FCSCond))
  expss::var_lab(Niger_pdm_2020$FCSCond)
  
Niger_pdm_2019%>% 
  sjPlot::plot_frq(coord.flip =T,FCSCond,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond-8.png)<!-- -->

### FCS : Condiments/épices: tels que (thé, café/cacao, sel, ail, épices, levure/levure chimique, tomate/sauce, viande ou poisson comme condiment, condiments incluant des petites quantités de lait/thé, café.) ? - Sources


```r
unique(Niger_baseline_2018$FCSCondSRf)
expss::val_lab(Niger_baseline_2018$FCSCondSRf)
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8)

Niger_baseline_2018$FCSCondSRf <- labelled::labelled(Niger_baseline_2018$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_baseline_2018$FCSCondSRf)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_baseline_2018-2.png)<!-- -->

```r
typeof(Niger_ea_2019$FCSCondSRf)
expss::val_lab(Niger_ea_2019$FCSCondSRf)
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"=8)

Niger_ea_2019$FCSCondSRf <- labelled::labelled(Niger_ea_2019$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2019$FCSCondSRf)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2019-2.png)<!-- -->


```r
unique(Niger_ea_2020$FCSCondSRf)
expss::val_lab(Niger_ea_2020$FCSCondSRf)
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8,"11"= 8,"12" = NA_real_, "13"=8)

Niger_ea_2020$FCSCondSRf <- labelled::labelled(Niger_ea_2020$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2020$FCSCondSRf)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2020-2.png)<!-- -->

```r
unique(Niger_ea_2021$FCSCondSRf)
expss::val_lab(Niger_ea_2021$FCSCondSRf)
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_ea_2021$FCSCondSRf <- labelled::labelled(Niger_ea_2021$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_ea_2021$FCSCondSRf)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_ea_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2021$FCSCondSRf)
expss::val_lab(Niger_pdm_2021$FCSCondSRf)
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_,"16"=NA_real_)

Niger_pdm_2021$FCSCondSRf <- labelled::labelled(Niger_pdm_2021$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2021$FCSCondSRf)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2021-2.png)<!-- -->

```r
unique(Niger_pdm_2022$FCSCondSRf)
expss::val_lab(Niger_pdm_2022$FCSCondSRf)
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=6,"6"=10,"7"=8,"8"=9,"9"=7,"10"=6,"11"= 2, "12"=3, "13"=4,"14"=8,"15"=NA_real_)

Niger_pdm_2022$FCSCondSRf <- labelled::labelled(Niger_pdm_2022$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2022$FCSCondSRf)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2022-2.png)<!-- -->


```r
unique(Niger_pdm_2020$FCSCondSRf)
expss::val_lab(Niger_pdm_2020$FCSCondSRf)
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=5,"2"=10,"3"=10,"4"=4,"5"=9,"6"=1,"7" =NA_real_)

Niger_pdm_2020$FCSCondSRf <- labelled::labelled(Niger_pdm_2020$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2020$FCSCondSRf)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2020-2.png)<!-- -->

```r
unique(Niger_pdm_2019$FCSCondSRf)
expss::val_lab(Niger_pdm_2019$FCSCondSRf)
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate_at(c("FCSCondSRf"),recode,"1"=1,"2"=1,"3"=1,"4"=5,"5"=10,"6"=8,"7"=9,"8"=6,"9"=2,"10"=8, "11"=8)

Niger_pdm_2019$FCSCondSRf <- labelled::labelled(Niger_pdm_2019$FCSCondSRf, c("Production propre (récoltes, élevage)" = 1, "Pêche / Chasse" = 2, "Cueillette"= 3,"Prêts"=4,"Marché (achat avec des espèces)"=5,"Marché (achat à crédit)"=6,"Mendicité"=7,"Troc travail ou biens contre des aliments"=8,"Dons (aliments) de membres de la famille ou d’amis"=9,"Aide alimentaire de la société civile, ONG, gouvernement, PAM, etc"=10))

#check labels
expss::val_lab(Niger_pdm_2019$FCSCondSRf)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,FCSCondSRf,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCSCond Source Niger_pdm_2019-2.png)<!-- -->


### FCS computation


```r
#calculate FCS
Niger_baseline_2018 <- Niger_baseline_2018 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_baseline_2018$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_baseline_2018 <- Niger_baseline_2018 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_baseline_2018$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_baseline_2018$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_baseline_2018-2.png)<!-- -->


```r
#calculate FCS
Niger_ea_2019 <- Niger_ea_2019 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) + FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_ea_2019$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_ea_2019 <- Niger_ea_2019 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_ea_2019$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_ea_2019$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2019-2.png)<!-- -->


```r
#calculate FCS
Niger_ea_2020 <- Niger_ea_2020 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_ea_2020$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_ea_2020 <- Niger_ea_2020 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_ea_2020$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_ea_2020$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2020-2.png)<!-- -->


```r
#calculate FCS
Niger_ea_2021 <- Niger_ea_2021 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_ea_2021$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_ea_2021 <- Niger_ea_2021 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_ea_2021$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_ea_2021$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_ea_2021-2.png)<!-- -->


```r
#calculate FCS
Niger_pdm_2021 <- Niger_pdm_2021 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_pdm_2021$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_pdm_2021 <- Niger_pdm_2021 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_pdm_2021$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_pdm_2021$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2021-2.png)<!-- -->


```r
#calculate FCS
Niger_pdm_2022 <- Niger_pdm_2022 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_pdm_2022$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_pdm_2022 <- Niger_pdm_2022 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_pdm_2022$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_pdm_2022$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2022-2.png)<!-- -->


```r
#calculate FCS
Niger_pdm_2020 <- Niger_pdm_2020 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_pdm_2020$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_pdm_2020 <- Niger_pdm_2020 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_pdm_2020$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_pdm_2020$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2020-2.png)<!-- -->


```r
#calculate FCS
Niger_pdm_2019 <- Niger_pdm_2019 %>% mutate(FCS = (2 * FCSStap) + (3 * FCSPulse)+ (4*FCSPr) +FCSVeg  +FCSFruit +(4*FCSDairy) + (0.5*FCSFat) + (0.5*FCSSugar))
var_label(Niger_pdm_2019$FCS) <- "Score de consommation alimentaire"
#create FCG groups based on 21/25 or 28/42 thresholds - analyst should decide which one to use.
Niger_pdm_2019 <- Niger_pdm_2019 %>% mutate(
  FCSCat21 = case_when(
    FCS <= 21 ~ "Pauvre", between(FCS, 21.5, 35) ~ "Limite", FCS > 35 ~ "Acceptable"),
  FCSCat28 = case_when(
    FCS <= 28 ~ "Pauvre", between(FCS, 28.5, 42) ~ "Limite", FCS > 42 ~ "Acceptable"))
var_label(Niger_pdm_2019$FCSCat21) <- "Groupe de consommation alimentaire - Seuils du 21/35"
var_label(Niger_pdm_2019$FCSCat28) <-  "Groupe de consommation alimentaire - Seuils du 28/42"

Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat21,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,FCSCat28,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/FCS Niger_pdm_2019-2.png)<!-- -->
## reduced coping strategy index OU l’indice réduit des stratégies de survie (rCSI)

### rCSI : Consommer des aliments moins préférés et moins chers


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-1.png)<!-- -->

```r
Niger_ea_2019%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (rCSILessQlty = as.integer(rCSILessQlty))
  expss::var_lab(Niger_pdm_2020$rCSILessQlty)
  
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSILessQlty,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSILessQlty-8.png)<!-- -->

### rCSI : Emprunter de la nourriture ou compter sur l’aide des parents/amis


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-1.png)<!-- -->

```r
Niger_ea_2019%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (rCSIBorrow = as.integer(rCSIBorrow))
  expss::var_lab(Niger_pdm_2020$rCSIBorrow)
  
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIBorrow,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIBorrow-8.png)<!-- -->

### rCSI : Diminuer la quantité consommée pendant les repas


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-1.png)<!-- -->

```r
Niger_ea_2019%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (rCSIMealSize = as.integer(rCSIMealSize))
  expss::var_lab(Niger_pdm_2020$rCSIMealSize)
  
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealSize,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealSize-8.png)<!-- -->

### rCSI :  Restreindre la consommation des adultes  pour nourrir les enfants


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-1.png)<!-- -->

```r
Niger_ea_2019%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (rCSIMealAdult = as.integer(rCSIMealAdult))
  expss::var_lab(Niger_pdm_2020$rCSIMealAdult)
  
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealAdult,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealAdult-8.png)<!-- -->

### rCSI : Diminuer le nombre de repas par jour


```r
Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-1.png)<!-- -->

```r
Niger_ea_2019%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-2.png)<!-- -->

```r
Niger_ea_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-3.png)<!-- -->

```r
Niger_ea_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-4.png)<!-- -->

```r
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-5.png)<!-- -->

```r
Niger_pdm_2022%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-6.png)<!-- -->

```r
Niger_pdm_2020%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-7.png)<!-- -->

```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  mutate (rCSIMealNb = as.integer(rCSIMealNb))
  expss::var_lab(Niger_pdm_2020$rCSIMealNb)
  
Niger_pdm_2021%>% 
  sjPlot::plot_frq(coord.flip =T,rCSIMealNb,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rCSIMealNb-8.png)<!-- -->


```r
#calculate rCSI Score
Niger_baseline_2018 <- Niger_baseline_2018 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_baseline_2018$rCSI) <- "rCSI"

Niger_ea_2019 <- Niger_ea_2019 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_ea_2019$rCSI) <- "rCSI"

Niger_ea_2020 <- Niger_ea_2020 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_ea_2020$rCSI) <- "rCSI"

Niger_ea_2021 <- Niger_ea_2021 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_ea_2021$rCSI) <- "rCSI"

Niger_pdm_2021 <- Niger_pdm_2021 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_pdm_2021$rCSI) <- "rCSI"

Niger_pdm_2022 <- Niger_pdm_2022 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_pdm_2022$rCSI) <- "rCSI"

Niger_pdm_2020 <- Niger_pdm_2020 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_pdm_2020$rCSI) <- "rCSI"

Niger_pdm_2019 <- Niger_pdm_2019 %>% mutate(rCSI = rCSILessQlty  + (2 * rCSIBorrow) + rCSIMealSize + (3 * rCSIMealAdult) + rCSIMealNb)
var_label(Niger_pdm_2019$rCSI) <- "rCSI"
```


Households are divided in four classes according to the rCSI score: 0-3, 4-18, and 19 and above which correspond to IPC Phases 1, 2 and 3 and above respectively.


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_baseline_2018$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_baseline_2018 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_1-1.png)<!-- -->

```r
Niger_ea_2019 <- Niger_ea_2019 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_ea_2019$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_ea_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_2-1.png)<!-- -->


```r
Niger_ea_2020 <- Niger_ea_2020 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_ea_2020$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_ea_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_3-1.png)<!-- -->


```r
Niger_ea_2021 <- Niger_ea_2021 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_ea_2021$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_ea_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_4-1.png)<!-- -->


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_pdm_2021$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_pdm_2021 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_5-1.png)<!-- -->


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_pdm_2022$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_pdm_2022 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_6-1.png)<!-- -->


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_pdm_2020$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_pdm_2020 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_7-1.png)<!-- -->


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% mutate(
  rcsi_class = case_when(
    rCSI <= 3 ~ 1, between(rCSI, 3.5 , 18.5) ~ 2, rCSI > 18.5 ~ 3))

var_label(Niger_pdm_2019$rcsi_class) <-  "IPC Phases 3 4 18 19 "

Niger_pdm_2019 %>% 
  sjPlot::plot_frq(coord.flip =T,rcsi_class,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/rcsi_class_8-1.png)<!-- -->
## Stratégies d'adaptation aux moyens d'existence (LhCSI)


```r
# 1 = Non, je n'ai pas été confronté à une insuffisance de nourriture
# 2 = Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire
# 3 = Oui
# 4 = Non applicable
```


### LhCSI : Vendre des actifs/biens non productifs du ménage (radio, meuble, réfrigérateur, télévision, bijoux etc.)



```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIStress1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress1
Niger_baseline_2018$LhCSIStress1 <- ifelse(is.na(Niger_baseline_2018$LhCSIStress1), 5, Niger_baseline_2018$LhCSIStress1)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSIStress1 <- labelled::labelled(Niger_baseline_2018$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIStress1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIStress1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress1
Niger_ea_2019$LhCSIStress1 <- ifelse(is.na(Niger_ea_2019$LhCSIStress1), 5, Niger_ea_2019$LhCSIStress1)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIStress1 <- labelled::labelled(Niger_ea_2019$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIStress1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIStress1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIStress1
Niger_ea_2020$LhCSIStress1 <- ifelse(is.na(Niger_ea_2020$LhCSIStress1), 5, Niger_ea_2020$LhCSIStress1)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIStress1 <- labelled::labelled(Niger_ea_2020$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIStress1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIStress1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIStress1 <- labelled::labelled(Niger_ea_2021$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIStress1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_ea_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIStress1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIStress1 <- ifelse(is.na(Niger_pdm_2019$LhCSIStress1), 5, Niger_pdm_2019$LhCSIStress1)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIStress1 <- labelled::labelled(Niger_pdm_2019$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIStress1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIStress1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIStress1 <- ifelse(is.na(Niger_pdm_2020$LhCSIStress1), 5, Niger_pdm_2020$LhCSIStress1)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIStress1 <- labelled::labelled(Niger_pdm_2020$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIStress1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIStress1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIStress1 = dplyr::recode(LhCSIStress1,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIStress1 <- labelled::labelled(Niger_pdm_2021$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIStress1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIStress1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress1)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIStress1 <- labelled::labelled(Niger_pdm_2022$LhCSIStress1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIStress1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress1)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress1 Niger_pdm_2022-2.png)<!-- -->


### LhCSI : Vendre plus d’animaux (non-productifs) que d’habitude


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIStress2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress2
Niger_baseline_2018$LhCSIStress2 <- ifelse(is.na(Niger_baseline_2018$LhCSIStress2), 5, Niger_baseline_2018$LhCSIStress2)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSIStress2 <- labelled::labelled(Niger_baseline_2018$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIStress2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIStress2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress2
Niger_ea_2019$LhCSIStress2 <- ifelse(is.na(Niger_ea_2019$LhCSIStress2), 5, Niger_ea_2019$LhCSIStress2)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIStress2 <- labelled::labelled(Niger_ea_2019$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIStress2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIStress2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIStress2
Niger_ea_2020$LhCSIStress2 <- ifelse(is.na(Niger_ea_2020$LhCSIStress2), 5, Niger_ea_2020$LhCSIStress2)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIStress2 <- labelled::labelled(Niger_ea_2020$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIStress2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIStress2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIStress2 <- labelled::labelled(Niger_ea_2021$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIStress2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIStress2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIStress2 <- ifelse(is.na(Niger_pdm_2019$LhCSIStress2), 5, Niger_pdm_2019$LhCSIStress2)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIStress2 <- labelled::labelled(Niger_pdm_2019$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIStress2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIStress2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIStress2 <- ifelse(is.na(Niger_pdm_2020$LhCSIStress2), 5, Niger_pdm_2020$LhCSIStress2)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIStress2 <- labelled::labelled(Niger_pdm_2020$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIStress2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIStress2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIStress2 = dplyr::recode(LhCSIStress2,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIStress2 <- labelled::labelled(Niger_pdm_2021$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIStress2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIStress2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIStress2 <- labelled::labelled(Niger_pdm_2022$LhCSIStress2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIStress2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress2 Niger_pdm_2022-2.png)<!-- -->

### LhCSI : Dépenser l’épargne en raison d'un manque de nourriture ou d'argent pour acheter de la nourriture ?


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIStress3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018$LhCSIStress3 <- NA_real_ 

Niger_baseline_2018$LhCSIStress3 <- labelled::labelled(Niger_baseline_2018$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIStress3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIStress3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress3
Niger_ea_2019$LhCSIStress3 <- ifelse(is.na(Niger_ea_2019$LhCSIStress3), 5, Niger_ea_2019$LhCSIStress3)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIStress3 = dplyr::recode(LhCSIStress3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIStress3 <- labelled::labelled(Niger_ea_2019$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIStress3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIStress3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIStress3
Niger_ea_2020$LhCSIStress3 <- ifelse(is.na(Niger_ea_2020$LhCSIStress3), 5, Niger_ea_2020$LhCSIStress3)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIStress3 = dplyr::recode(LhCSIStress3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIStress3 <- labelled::labelled(Niger_ea_2020$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIStress3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIStress3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIStress3 <- labelled::labelled(Niger_ea_2021$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIStress3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIStress3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIStress3 <- ifelse(is.na(Niger_pdm_2019$LhCSIStress3), 5, Niger_pdm_2019$LhCSIStress3)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIStress3 = dplyr::recode(LhCSIStress3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIStress3 <- labelled::labelled(Niger_pdm_2019$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIStress3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIStress3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIStress3 <- ifelse(is.na(Niger_pdm_2020$LhCSIStress3), 5, Niger_pdm_2020$LhCSIStress3)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIStress3 = dplyr::recode(LhCSIStress3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIStress3 <- labelled::labelled(Niger_pdm_2020$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIStress3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIStress3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIStress3 = dplyr::recode(LhCSIStress3,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIStress3 <- labelled::labelled(Niger_pdm_2021$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIStress3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIStress3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIStress3 <- labelled::labelled(Niger_pdm_2022$LhCSIStress3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIStress3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress3 Niger_pdm_2022-2.png)<!-- -->

### LhCSI : Emprunter de l’argent / nourriture auprès d’un prêteur formel /banque


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIStress4)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress4
Niger_baseline_2018$LhCSIStress4 <- ifelse(is.na(Niger_baseline_2018$LhCSIStress4), 5, Niger_baseline_2018$LhCSIStress4)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSIStress4 <- labelled::labelled(Niger_baseline_2018$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIStress4)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIStress4)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIStress4
Niger_ea_2019$LhCSIStress4 <- ifelse(is.na(Niger_ea_2019$LhCSIStress4), 5, Niger_ea_2019$LhCSIStress4)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIStress4 <- labelled::labelled(Niger_ea_2019$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIStress4)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIStress4)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIStress4
Niger_ea_2020$LhCSIStress4 <- ifelse(is.na(Niger_ea_2020$LhCSIStress4), 5, Niger_ea_2020$LhCSIStress4)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIStress4 <- labelled::labelled(Niger_ea_2020$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIStress4)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIStress4)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIStress4 <- labelled::labelled(Niger_ea_2021$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIStress4)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIStress4)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIStress4 <- ifelse(is.na(Niger_pdm_2019$LhCSIStress4), 5, Niger_pdm_2019$LhCSIStress4)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIStress4 <- labelled::labelled(Niger_pdm_2019$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIStress4)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIStress4)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIStress4 <- ifelse(is.na(Niger_pdm_2020$LhCSIStress4), 5, Niger_pdm_2020$LhCSIStress4)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIStress4 <- labelled::labelled(Niger_pdm_2020$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIStress4)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIStress4)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIStress4 = dplyr::recode(LhCSIStress4,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIStress4 <- labelled::labelled(Niger_pdm_2021$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIStress4)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIStress4)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIStress4 <- labelled::labelled(Niger_pdm_2022$LhCSIStress4, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIStress4)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIStress4,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIStress4 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Réduire les dépenses non alimentaires essentielles telles que l’éducation, la santé (dont de médicaments)


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018$LhCSICrisis1 <- NA_real_ 

Niger_baseline_2018$LhCSICrisis1 <- labelled::labelled(Niger_baseline_2018$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSICrisis1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSICrisis1
Niger_ea_2019$LhCSICrisis1 <- ifelse(is.na(Niger_ea_2019$LhCSICrisis1), 5, Niger_ea_2019$LhCSICrisis1)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSICrisis1 = dplyr::recode(LhCSICrisis1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSICrisis1 <- labelled::labelled(Niger_ea_2019$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSICrisis1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSICrisis1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSICrisis1
Niger_ea_2020$LhCSICrisis1 <- ifelse(is.na(Niger_ea_2020$LhCSICrisis1), 5, Niger_ea_2020$LhCSICrisis1)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSICrisis1 = dplyr::recode(LhCSICrisis1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSICrisis1 <- labelled::labelled(Niger_ea_2020$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSICrisis1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSICrisis1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSICrisis1 <- labelled::labelled(Niger_ea_2021$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSICrisis1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSICrisis1 <- ifelse(is.na(Niger_pdm_2019$LhCSICrisis1), 5, Niger_pdm_2019$LhCSICrisis1)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSICrisis1 = dplyr::recode(LhCSICrisis1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSICrisis1 <- labelled::labelled(Niger_pdm_2019$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSICrisis1 <- ifelse(is.na(Niger_pdm_2020$LhCSICrisis1), 5, Niger_pdm_2020$LhCSICrisis1)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSICrisis1 = dplyr::recode(LhCSICrisis1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSICrisis1 <- labelled::labelled(Niger_pdm_2020$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSICrisis1 = dplyr::recode(LhCSICrisis1,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSICrisis1 <- labelled::labelled(Niger_pdm_2021$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSICrisis1 <- labelled::labelled(Niger_pdm_2022$LhCSICrisis1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis1 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Vendre des biens productifs ou des moyens de transport (machine à coudre, brouette, vélo, car, etc.)


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSICrisis2
Niger_baseline_2018$LhCSICrisis2 <- ifelse(is.na(Niger_baseline_2018$LhCSICrisis2), 5, Niger_baseline_2018$LhCSICrisis2)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSICrisis2 <- labelled::labelled(Niger_baseline_2018$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSICrisis2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSICrisis2
Niger_ea_2019$LhCSICrisis2 <- ifelse(is.na(Niger_ea_2019$LhCSICrisis2), 5, Niger_ea_2019$LhCSICrisis2)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSICrisis2 <- labelled::labelled(Niger_ea_2019$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSICrisis2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSICrisis2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSICrisis2
Niger_ea_2020$LhCSICrisis2 <- ifelse(is.na(Niger_ea_2020$LhCSICrisis2), 5, Niger_ea_2020$LhCSICrisis2)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSICrisis2 <- labelled::labelled(Niger_ea_2020$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSICrisis2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSICrisis2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSICrisis2 <- labelled::labelled(Niger_ea_2021$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSICrisis2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSICrisis2 <- ifelse(is.na(Niger_pdm_2019$LhCSICrisis2), 5, Niger_pdm_2019$LhCSICrisis2)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSICrisis2 <- labelled::labelled(Niger_pdm_2019$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSICrisis2 <- ifelse(is.na(Niger_pdm_2020$LhCSICrisis2), 5, Niger_pdm_2020$LhCSICrisis2)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSICrisis2 <- labelled::labelled(Niger_pdm_2020$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSICrisis2 = dplyr::recode(LhCSICrisis2,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSICrisis2 <- labelled::labelled(Niger_pdm_2021$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSICrisis2 <- labelled::labelled(Niger_pdm_2022$LhCSICrisis2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis2 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Retirer les enfants de l’école


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSICrisis3
Niger_baseline_2018$LhCSICrisis3 <- ifelse(is.na(Niger_baseline_2018$LhCSICrisis3), 5, Niger_baseline_2018$LhCSICrisis3)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSICrisis3 <- labelled::labelled(Niger_baseline_2018$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSICrisis3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSICrisis3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSICrisis3
Niger_ea_2019$LhCSICrisis3 <- ifelse(is.na(Niger_ea_2019$LhCSICrisis3), 5, Niger_ea_2019$LhCSICrisis3)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSICrisis3 <- labelled::labelled(Niger_ea_2019$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSICrisis3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSICrisis3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSICrisis3
Niger_ea_2020$LhCSICrisis3 <- ifelse(is.na(Niger_ea_2020$LhCSICrisis3), 5, Niger_ea_2020$LhCSICrisis3)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSICrisis3 <- labelled::labelled(Niger_ea_2020$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSICrisis3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSICrisis3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSICrisis3 <- labelled::labelled(Niger_ea_2021$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSICrisis3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSICrisis3 <- ifelse(is.na(Niger_pdm_2019$LhCSICrisis3), 5, Niger_pdm_2019$LhCSICrisis3)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSICrisis3 <- labelled::labelled(Niger_pdm_2019$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSICrisis3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSICrisis3 <- ifelse(is.na(Niger_pdm_2020$LhCSICrisis3), 5, Niger_pdm_2020$LhCSICrisis3)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSICrisis3 <- labelled::labelled(Niger_pdm_2020$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSICrisis3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSICrisis3 = dplyr::recode(LhCSICrisis3,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSICrisis3 <- labelled::labelled(Niger_pdm_2021$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSICrisis3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSICrisis3 <- labelled::labelled(Niger_pdm_2022$LhCSICrisis3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSICrisis3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSICrisis3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSICrisis3 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Vendre la maison ou des terrains


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIEmergency1
Niger_baseline_2018$LhCSIEmergency1 <- ifelse(is.na(Niger_baseline_2018$LhCSIEmergency1), 5, Niger_baseline_2018$LhCSIEmergency1)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSIEmergency1 <- labelled::labelled(Niger_baseline_2018$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency1)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIEmergency1
Niger_ea_2019$LhCSIEmergency1 <- ifelse(is.na(Niger_ea_2019$LhCSIEmergency1), 5, Niger_ea_2019$LhCSIEmergency1)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIEmergency1 <- labelled::labelled(Niger_ea_2019$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency1)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIEmergency1
Niger_ea_2020$LhCSIEmergency1 <- ifelse(is.na(Niger_ea_2020$LhCSIEmergency1), 5, Niger_ea_2020$LhCSIEmergency1)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIEmergency1 <- labelled::labelled(Niger_ea_2020$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency1)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIEmergency1 <- labelled::labelled(Niger_ea_2021$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency1)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIEmergency1 <- ifelse(is.na(Niger_pdm_2019$LhCSIEmergency1), 5, Niger_pdm_2019$LhCSIEmergency1)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIEmergency1 <- labelled::labelled(Niger_pdm_2019$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency1)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIEmergency1 <- ifelse(is.na(Niger_pdm_2020$LhCSIEmergency1), 5, Niger_pdm_2020$LhCSIEmergency1)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIEmergency1 <- labelled::labelled(Niger_pdm_2020$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency1)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIEmergency1 = dplyr::recode(LhCSIEmergency1,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIEmergency1 <- labelled::labelled(Niger_pdm_2021$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency1)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIEmergency1 <- labelled::labelled(Niger_pdm_2022$LhCSIEmergency1, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency1)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency1,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency1 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Mendier

```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018$LhCSIEmergency2 <- NA_real_ 

Niger_baseline_2018$LhCSIEmergency2 <- labelled::labelled(Niger_baseline_2018$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency2)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIEmergency2
Niger_ea_2019$LhCSIEmergency2 <- ifelse(is.na(Niger_ea_2019$LhCSIEmergency2), 5, Niger_ea_2019$LhCSIEmergency2)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIEmergency2 = dplyr::recode(LhCSIEmergency2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIEmergency2 <- labelled::labelled(Niger_ea_2019$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency2)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIEmergency2
Niger_ea_2020$LhCSIEmergency2 <- ifelse(is.na(Niger_ea_2020$LhCSIEmergency2), 5, Niger_ea_2020$LhCSIEmergency2)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIEmergency2 = dplyr::recode(LhCSIEmergency2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIEmergency2 <- labelled::labelled(Niger_ea_2020$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency2)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIEmergency2 <- labelled::labelled(Niger_ea_2021$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency2)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIEmergency2 <- ifelse(is.na(Niger_pdm_2019$LhCSIEmergency2), 5, Niger_pdm_2019$LhCSIEmergency2)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIEmergency2 = dplyr::recode(LhCSIEmergency2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIEmergency2 <- labelled::labelled(Niger_pdm_2019$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency2)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIEmergency2 <- ifelse(is.na(Niger_pdm_2020$LhCSIEmergency2), 5, Niger_pdm_2020$LhCSIEmergency2)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIEmergency2 = dplyr::recode(LhCSIEmergency2,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIEmergency2 <- labelled::labelled(Niger_pdm_2020$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency2)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIEmergency2 = dplyr::recode(LhCSIEmergency2,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIEmergency2 <- labelled::labelled(Niger_pdm_2021$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency2)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIEmergency2 <- labelled::labelled(Niger_pdm_2022$LhCSIEmergency2, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency2)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency2,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency2 Niger_pdm_2022-2.png)<!-- -->
### LhCSI : Vendre les derniers animaux femelles reproductrices


```r
#View labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_baseline_2018-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIEmergency3
Niger_baseline_2018$LhCSIEmergency3 <- ifelse(is.na(Niger_baseline_2018$LhCSIEmergency3), 5, Niger_baseline_2018$LhCSIEmergency3)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_baseline_2018$LhCSIEmergency3 <- labelled::labelled(Niger_baseline_2018$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_baseline_2018$LhCSIEmergency3)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2019-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 3 dans la variable LhCSIEmergency3
Niger_ea_2019$LhCSIEmergency3 <- ifelse(is.na(Niger_ea_2019$LhCSIEmergency3), 5, Niger_ea_2019$LhCSIEmergency3)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2019$LhCSIEmergency3 <- labelled::labelled(Niger_ea_2019$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2019$LhCSIEmergency3)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2020-1.png)<!-- -->

```r
# Remplace tous les NA par la modalité 5 dans la variable LhCSIEmergency3
Niger_ea_2020$LhCSIEmergency3 <- ifelse(is.na(Niger_ea_2020$LhCSIEmergency3), 5, Niger_ea_2020$LhCSIEmergency3)

#change labels
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_ea_2020$LhCSIEmergency3 <- labelled::labelled(Niger_ea_2020$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2020$LhCSIEmergency3)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2021-1.png)<!-- -->

```r
#update labels
Niger_ea_2021$LhCSIEmergency3 <- labelled::labelled(Niger_ea_2021$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_ea_2021$LhCSIEmergency3)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_ea_2021-2.png)<!-- -->

```r
#View labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$LhCSIEmergency3 <- ifelse(is.na(Niger_pdm_2019$LhCSIEmergency3), 5, Niger_pdm_2019$LhCSIEmergency3)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2019$LhCSIEmergency3 <- labelled::labelled(Niger_pdm_2019$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2019$LhCSIEmergency3)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$LhCSIEmergency3 <- ifelse(is.na(Niger_pdm_2020$LhCSIEmergency3), 5, Niger_pdm_2020$LhCSIEmergency3)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=4,"5"=3))
Niger_pdm_2020$LhCSIEmergency3 <- labelled::labelled(Niger_pdm_2020$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2020$LhCSIEmergency3)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(LhCSIEmergency3 = dplyr::recode(LhCSIEmergency3,"1"=1,"2"=2,"3"=3,"4"=4))
Niger_pdm_2021$LhCSIEmergency3 <- labelled::labelled(Niger_pdm_2021$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2021$LhCSIEmergency3)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$LhCSIEmergency3 <- labelled::labelled(Niger_pdm_2022$LhCSIEmergency3, c("Non, je n'ai pas été confronté à une insuffisance de nourriture" = 1, "Non, parce que j’ai déjà vendu ces actifs ou mené cette activité au cours des 12 derniers mois et je ne peux pas continuer à le faire" = 2, "Oui"= 3,"Non applicable"=4))
#check labels
expss::val_lab(Niger_pdm_2022$LhCSIEmergency3)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,LhCSIEmergency3,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/LhCSIEmergency3 Niger_pdm_2022-2.png)<!-- -->

# Self-evaluated resilience score (SERS) 

```r
sers_variables = Niger_baseline_2018 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_baseline_2018$SERSRebondir)
unique(Niger_baseline_2018$SERSRebondir)


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "tout à fait d'accord" = 1,
                 "d'accord" = 2,
                 "ni d'accord ni pas d'accord" = 3,
                 "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```


```r
sers_variables = Niger_ea_2019 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_ea_2019$SERSRebondir)
unique(Niger_ea_2019$SERSRebondir)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```

```r
sers_variables = Niger_ea_2020 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_ea_2020$SERSRebondir)
unique(Niger_ea_2020$SERSRebondir)

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```

```r
sers_variables = Niger_ea_2021 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_ea_2021$SERSRebondir)
unique(Niger_ea_2021$SERSRebondir)

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```


```r
sers_variables = Niger_pdm_2021 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_pdm_2021$SERSRebondir)
unique(Niger_pdm_2021$SERSRebondir)

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```


```r
sers_variables = Niger_pdm_2022 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_pdm_2022$SERSRebondir)
unique(Niger_pdm_2022$SERSRebondir)

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```


```r
sers_variables = Niger_pdm_2020 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_pdm_2020$SERSRebondir)
unique(Niger_pdm_2020$SERSRebondir)

Niger_pdm_2020 <- Niger_pdm_2020 %>%
  mutate(across(sers_variables,
                ~recode(., `0` = 5, `1` = 4, `2` = 3, `3` = 2, `4` = 1)))


Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```


```r
sers_variables = Niger_pdm_2019 %>% dplyr::select(gtsummary::starts_with("SERS")) %>% names()

expss::val_lab(Niger_pdm_2019$SERSRebondir)
unique(Niger_pdm_2019$SERSRebondir)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "tout à fait d'accord" = 1,
                  "d'accord" = 2,
                  "ni d'accord ni pas d'accord" = 3,
                  "pas d'accord" = 4,
                  "pas du tout d'accord" = 5))))
```
# ASSET BENEFIT INDICATOR (ABI) 

# Gender recodification

```r
# We need to recode gender label to:
# 0 = Femme
# 1 = Homme


#View labels
expss::val_lab(Niger_baseline_2018$ABISexparticipant)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_1 Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018$ABISexparticipant <- dplyr::recode(Niger_baseline_2018$ABISexparticipant, `2` = 0L,  .default = 1L)
Niger_baseline_2018$ABISexparticipant <- labelled::labelled(Niger_baseline_2018$ABISexparticipant,       c(`Femme` = 0, `Homme` = 1))
#Check new labels
expss::val_lab(Niger_baseline_2018$ABISexparticipant)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_1 Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$ABISexparticipant)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_2 Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019$ABISexparticipant<-dplyr::recode(Niger_ea_2019$ABISexparticipant, "1" = 1L, "2" = 0L)
Niger_ea_2019$ABISexparticipant <- labelled::labelled(Niger_ea_2019$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))

#Check labels
expss::val_lab(Niger_ea_2019$ABISexparticipant)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_2 Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$ABISexparticipant)
#Niger_ea_2020 %>% 
#  plot_frq(coord.flip =T, ABISexparticipant)
Niger_ea_2020$ABISexparticipant<-dplyr::recode(Niger_ea_2020$ABISexparticipant, `2` = 0L, .default = 1L)
Niger_ea_2020$ABISexparticipant <- labelled::labelled(Niger_ea_2020$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))
#Check new labels
expss::val_lab(Niger_ea_2020$ABISexparticipant)
'Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)'
```


```r
#View labels
expss::val_lab(Niger_ea_2021$ABISexparticipant)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_4 Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021$ABISexparticipant<-dplyr::recode(Niger_ea_2021$ABISexparticipant, `2` = 0L, .default = 1L)
Niger_ea_2021$ABISexparticipant <- labelled::labelled(Niger_ea_2021$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))

#Check labels
expss::val_lab(Niger_ea_2021$ABISexparticipant)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_4 Niger_ea_2021-2.png)<!-- -->


```r
expss::val_lab(Niger_pdm_2019$ABISexparticipant)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_5 Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$ABISexparticipant<-dplyr::recode(Niger_pdm_2019$ABISexparticipant, "1" = 1L, "2" = 0L)
Niger_pdm_2019$ABISexparticipant <- labelled::labelled(Niger_pdm_2019$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))
expss::val_lab(Niger_pdm_2019$ABISexparticipant)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_5 Niger_pdm_2019-2.png)<!-- -->

```r
expss::val_lab(Niger_pdm_2020$ABISexparticipant)
'Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)'
Niger_pdm_2020$ABISexparticipant<-dplyr::recode(Niger_pdm_2020$ABISexparticipant, "1" = 0L, "0" = 1L)
Niger_pdm_2020$ABISexparticipant <- labelled::labelled(Niger_pdm_2020$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))
expss::val_lab(Niger_pdm_2020$ABISexparticipant)
'Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)'
```





```r
#View labels
expss::val_lab(Niger_pdm_2021$ABISexparticipant)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_7 Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021$ABISexparticipant<-dplyr::recode(Niger_pdm_2021$ABISexparticipant, `2` = 0L, .default = 1L, .combine_value_labels = TRUE)
Niger_pdm_2021$ABISexparticipant <- labelled::labelled(Niger_pdm_2021$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))

#Check labels
expss::val_lab(Niger_pdm_2021$ABISexparticipant)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification_7 Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$ABISexparticipant)
'Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)'
Niger_pdm_2022$ABISexparticipant<-dplyr::recode(Niger_pdm_2022$ABISexparticipant, `2` = 0L, .default = 1L)
Niger_pdm_2022$ABISexparticipant <- labelled::labelled(Niger_pdm_2022$ABISexparticipant, c(`Femme` = 0, `Homme` = 1))


#Check labels
expss::val_lab(Niger_pdm_2022$ABISexparticipant)
'Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,ABISexparticipant)'
```



```r
abi_variables = Niger_baseline_2018 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_baseline_2018$ABIProteger)
unique(Niger_baseline_2018$ABIProteger)

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_ea_2019 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_ea_2019$ABIProteger)
unique(Niger_ea_2019$ABIProteger)

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_ea_2020 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_ea_2020$ABIProteger)
unique(Niger_ea_2020$ABIProteger)

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_ea_2021 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_ea_2021$ABIProteger)
unique(Niger_ea_2021$ABIProteger)

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_pdm_2021 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_pdm_2021$ABIProteger)
unique(Niger_pdm_2021$ABIProteger)

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_pdm_2022 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_pdm_2022$ABIProteger)
unique(Niger_pdm_2022$ABIProteger)

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_pdm_2020 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_pdm_2020$ABIProteger)
unique(Niger_pdm_2020$ABIProteger)

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```



```r
abi_variables = Niger_pdm_2019 %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

abi_variables <- abi_variables[! abi_variables %in% c('ABISexparticipant')]

expss::val_lab(Niger_pdm_2019$ABIProteger)
unique(Niger_pdm_2019$ABIProteger)

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(abi_variables,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0, .default = 888)))
```

# DEPART EN EXODE ET MIGRATION  



# Social capital index (Indice de capital social)




# DIVERSITE ALIMENTAIRE DES FEMMES 



# Régime alimentaire minimum acceptable (MAD)



# Date format check


```r
Sys.setlocale(locale = "en_US.UTF-8")
head(Niger_baseline_2018$SvyDatePDM, 5)

Niger_baseline_2018 <- Niger_baseline_2018 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM,format = ("%b %d, %Y, %I:%M:%S %p")))

Sys.setlocale(locale = "C")
```


```r
head(Niger_ea_2019$SvyDatePDM, 5)

Niger_ea_2019 <- Niger_ea_2019 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
head(Niger_ea_2020$SvyDatePDM, 5)
```


```r
head(Niger_ea_2021$SvyDatePDM, 5)

Niger_ea_2021 <- Niger_ea_2021 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM, format = ("%Y-%m-%dT%H:%M:%OS")))
```




```r
head(Niger_pdm_2021$SvyDatePDM, 5)

Niger_pdm_2021 <- Niger_pdm_2021 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
head(Niger_pdm_2022$SvyDatePDM, 5)

Niger_pdm_2022 <- Niger_pdm_2022 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
head(Niger_pdm_2020$SvyDatePDM, 5)
```


```r
head(Niger_pdm_2019$SvyDatePDM, 5)

Niger_pdm_2019 <- Niger_pdm_2019 %>% dplyr::mutate(SvyDatePDM =            as.Date(SvyDatePDM,format = ("%Y-%m-%dT%H:%M:%OS")))
```


# Gender recodification


```r
for (j in 1:length(lst_test)) {
  df=  get(lst_test[j], envir = .GlobalEnv)
  cat(lst_test[j], " : ", " Type : ",typeof(df$HHHSex), " Class : ",  class(df$HHHSex), "\n")
}
# lorsque Respconsent prend uniquement NA ou 1 la base a été traitée au préalable et contient uniquement les individus consentants
```



```r
# We need to recode gender label to:
# 0 = Femme
# 1 = Homme

#View labels
expss::val_lab(Niger_baseline_2018$HHHSex)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_baseline_2018-1.png)<!-- -->

```r
Niger_baseline_2018$HHHSex <- as.numeric(as.character(Niger_baseline_2018$HHHSex))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate_at(c("HHHSex"), recode,  `2` = 0L,  `1` = 1L)


#Check new labels
expss::val_lab(Niger_baseline_2018$HHHSex)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_baseline_2018-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2019$HHHSex)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019$HHHSex <- as.numeric(as.character(Niger_ea_2019$HHHSex))

Niger_ea_2019$HHHSex<-dplyr::recode(Niger_ea_2019$HHHSex, `2` = 0L,  `1` = 1L)

#Check labels
expss::val_lab(Niger_ea_2019$HHHSex)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2019-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2020$HHHSex)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020$HHHSex <- as.numeric(as.character(Niger_ea_2020$HHHSex))

Niger_ea_2020$HHHSex<-dplyr::recode(Niger_ea_2020$HHHSex, `2` = 0L,  `1` = 1L)

#Check new labels
expss::val_lab(Niger_ea_2020$HHHSex)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_ea_2021$HHHSex)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021$HHHSex <- as.numeric(as.character(Niger_ea_2021$HHHSex))


Niger_ea_2021$HHHSex<-dplyr::recode(Niger_ea_2021$HHHSex, `2` = 0L,  `1` = 1L)
#Check labels
expss::val_lab(Niger_ea_2021$HHHSex)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_ea_2021-2.png)<!-- -->


```r
expss::val_lab(Niger_pdm_2019$HHHSex)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019$HHHSex <- as.numeric(as.character(Niger_pdm_2019$HHHSex))


Niger_pdm_2019$HHHSex<-dplyr::recode(Niger_pdm_2019$HHHSex, `2` = 0L,  `1` = 1L)
expss::val_lab(Niger_pdm_2019$HHHSex)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2019-2.png)<!-- -->


```r
expss::val_lab(Niger_pdm_2020$HHHSex)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2020-1.png)<!-- -->

```r
Niger_pdm_2020$HHHSex <- as.numeric(as.character(Niger_pdm_2020$HHHSex))


Niger_pdm_2020$HHHSex<-dplyr::recode(Niger_pdm_2020$HHHSex, `2` = 0L,  `1` = 1L)

expss::val_lab(Niger_pdm_2020$HHHSex)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2020-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2021$HHHSex)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2021-1.png)<!-- -->

```r
Niger_pdm_2021$HHHSex <- as.numeric(as.character(Niger_pdm_2021$HHHSex))

Niger_pdm_2021$HHHSex<-dplyr::recode(Niger_pdm_2021$HHHSex, `2` = 0L,  `1` = 1L)

#Check labels
expss::val_lab(Niger_pdm_2021$HHHSex)
Niger_pdm_2021 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2021-2.png)<!-- -->


```r
#View labels
expss::val_lab(Niger_pdm_2022$HHHSex)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2022-1.png)<!-- -->

```r
Niger_pdm_2022$HHHSex <- as.numeric(as.character(Niger_pdm_2022$HHHSex))

Niger_pdm_2022$HHHSex<-dplyr::recode(Niger_pdm_2022$HHHSex, `2` = 0L,  `1` = 1L)

#Check labels
expss::val_lab(Niger_pdm_2022$HHHSex)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,HHHSex)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Gender recodification Niger_pdm_2022-2.png)<!-- -->

# Household head education level


```r
expss::val_lab(Niger_baseline_2018$HHHEdu)

Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=2,"4"=3,"5"=4,"6"=5))
Niger_baseline_2018$HHHEdu <- labelled::labelled(Niger_baseline_2018$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
#check labels
expss::val_lab(Niger_baseline_2018$HHHEdu)
#Niger_baseline_2018 %>% 
 # plot_frq(coord.flip =T,HHHEdu)
```


```r
expss::val_lab(Niger_ea_2019$HHHEdu)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,HHHEdu)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2019-1.png)<!-- -->

```r
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"6"=1,"1"=2,"2"=2,"3"=3,"4"=4,"5"=5))
Niger_ea_2019$HHHEdu <- labelled::labelled(Niger_ea_2019$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_ea_2019$HHHEdu)
Niger_ea_2019 %>% 
  plot_frq(coord.flip =T,HHHEdu)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2019-2.png)<!-- -->


```r
expss::val_lab(Niger_ea_2020$HHHEdu)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,HHHEdu)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2020-1.png)<!-- -->

```r
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=2,"4"=3,"5"=4,"6"=5))
Niger_ea_2020$HHHEdu <- labelled::labelled(Niger_ea_2020$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_ea_2020$HHHEdu)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,HHHEdu)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2020-2.png)<!-- -->


```r
expss::val_lab(Niger_ea_2021$HHHEdu)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,HHHEdu,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2021-1.png)<!-- -->

```r
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=3,"4"=3,"5"=4,"6"=5))
Niger_ea_2021$HHHEdu <- labelled::labelled(Niger_ea_2021$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_ea_2021$HHHEdu)
Niger_ea_2021 %>% 
  plot_frq(coord.flip =T,HHHEdu,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_ea_2021-2.png)<!-- -->


```r
expss::val_lab(Niger_pdm_2019$HHHEdu)
Niger_pdm_2019 %>% 
  plot_frq(coord.flip =T,HHHEdu,show.na = T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/Household head education level Niger_pdm_2019-1.png)<!-- -->

```r
Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=2,"4"=3,"5"=4,"6"=5))
Niger_pdm_2019$HHHEdu <- labelled::labelled(Niger_pdm_2019$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_pdm_2019$HHHEdu)
```


```r
expss::val_lab(Niger_pdm_2020$HHHEdu)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=2,"4"=3,"5"=3,"6"=4,"7"=5))
Niger_pdm_2020$HHHEdu <- labelled::labelled(Niger_pdm_2020$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_pdm_2020$HHHEdu)
```


```r
expss::val_lab(Niger_pdm_2021$HHHEdu)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=2,"4"=3,"5"=3,"6"=4,"7"=5))
Niger_pdm_2021$HHHEdu <- labelled::labelled(Niger_pdm_2021$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_pdm_2021$HHHEdu)
##
```


```r
expss::val_lab(Niger_pdm_2022$HHHEdu)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(HHHEdu = dplyr::recode(HHHEdu,"1"=1,"2"=2,"3"=3,"4"=3,"5"=4,"6"=5))
Niger_pdm_2022$HHHEdu <- labelled::labelled(Niger_pdm_2022$HHHEdu, c(`Aucune` = 1, `Alphabétisé ou Coranique` = 2, `Primaire`= 3,`Secondaire`=4, `Superieur`=5))
expss::val_lab(Niger_pdm_2022$HHHEdu)
```

# HHHMainActivity


```r
expss::val_lab(Niger_baseline_2018$HHHMainActivity)
Niger_baseline_2018 %>% 
  plot_frq(coord.flip =T,HHHMainActivity)
```

![](01_Niger_LabelsHarmonization_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
expss::val_lab(Niger_ea_2019$HHHMainActivity)
expss::val_lab(Niger_ea_2020$HHHMainActivity)
Niger_ea_2020 %>% 
  plot_frq(coord.flip =T,HHHMainActivity)
```

![](01_Niger_LabelsHarmonization_files/figure-html/unnamed-chunk-12-2.png)<!-- -->

```r
expss::val_lab(Niger_ea_2021$HHHMainActivity)
expss::val_lab(Niger_pdm_2019$HHHMainActivity)
expss::val_lab(Niger_pdm_2020$HHHMainActivity)
expss::val_lab(Niger_pdm_2020$HHHMainActivity)
expss::val_lab(Niger_pdm_2022$HHHMainActivity)
```

# HHHMatrimonial


```r
# Monogame    Polygame     Divorcé(e)   Veuf/Veuve    Célibataire 
#   1           2           3              4           5 
expss::val_lab(Niger_baseline_2018$HHHMatrimonial)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(HHHMatrimonial = dplyr::recode(HHHMatrimonial,"1"=1,"2"=2,"3"=3,"4"=4,"5"=5))
Niger_baseline_2018$HHHMatrimonial <- labelled::labelled(Niger_baseline_2018$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_baseline_2018$HHHMatrimonial)


Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(HHHMatrimonial = dplyr::recode(HHHMatrimonial,"5"=1,"1"=2,"2"=3,"3"=4,"4"=5))
Niger_ea_2019$HHHMatrimonial <- labelled::labelled(Niger_ea_2019$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_ea_2019$HHHMatrimonial)

expss::val_lab(Niger_ea_2020$HHHMatrimonial)
Niger_ea_2020$HHHMatrimonial <- labelled::labelled(Niger_ea_2020$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_ea_2020$HHHMatrimonial)


expss::val_lab(Niger_ea_2021$HHHMatrimonial)#Null
expss::val_lab(Niger_pdm_2022$HHHMatrimonial)#Null
Niger_ea_2021$HHHMatrimonial <- labelled::labelled(Niger_ea_2021$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_ea_2021$HHHMatrimonial)


Niger_pdm_2022$HHHMatrimonial <- labelled::labelled(Niger_pdm_2022$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_pdm_2022$HHHMatrimonial)

expss::val_lab(Niger_pdm_2020$HHHMatrimonial)
expss::val_lab(Niger_pdm_2019$HHHMatrimonial)

Niger_pdm_2019 <- 
  Niger_pdm_2019 %>% dplyr::mutate(HHHMatrimonial = dplyr::recode(HHHMatrimonial,"1"=2,"2"=3,"3"=4,"4"=5,"5"=1))
Niger_pdm_2019$HHHMatrimonial <- labelled::labelled(Niger_pdm_2019$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_pdm_2019$HHHMatrimonial)


Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(HHHMatrimonial = dplyr::recode(HHHMatrimonial,"1"=5,"2"=1,"3"=2,"4"=3,"5"=4))
Niger_pdm_2020$HHHMatrimonial <- labelled::labelled(Niger_pdm_2020$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_pdm_2020$HHHMatrimonial)


expss::val_lab(Niger_pdm_2021$HHHMatrimonial)
Niger_pdm_2021$HHHMatrimonial <- labelled::labelled(Niger_pdm_2021$HHHMatrimonial, c(`Monogame` = 1, `Polygame` = 2, `Divorcé(e)`= 3,`Veuf/Veuve`=4, `Célibataire`=5))
expss::val_lab(Niger_pdm_2021$HHHMatrimonial)
expss::val_lab(Niger_pdm_2022$HHHMatrimonial)
```

# HHSourceIncome


```r
expss::val_lab(Niger_baseline_2018$HHSourceIncome)
expss::val_lab(Niger_ea_2019$HHSourceIncome)
expss::val_lab(Niger_ea_2020$HHSourceIncome)
expss::val_lab(Niger_ea_2021$HHSourceIncome)
expss::val_lab(Niger_pdm_2019$HHSourceIncome)
expss::val_lab(Niger_pdm_2020$HHSourceIncome)
expss::val_lab(Niger_pdm_2021$HHSourceIncome)
expss::val_lab(Niger_pdm_2022$HHSourceIncome)
```
# Assistance

## Date assistance check

```r
tail(Niger_baseline_2018$DebutAssistance, 15)


Niger_baseline_2018 <- Niger_baseline_2018 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
tail(Niger_ea_2019$DebutAssistance, 5)

Niger_ea_2019 <- Niger_ea_2019 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance, origin = "1899-12-30"))
```


```r
Sys.setlocale(locale = "en_US.UTF-8")

head(Niger_ea_2020$DebutAssistance, 5)


Niger_ea_2020 <- Niger_ea_2020 %>% dplyr::mutate(DebutAssistance =            as.Date(paste(toupper(substring(DebutAssistance, 1, 1)), tolower(substring(DebutAssistance, 2))," 01", sep = ""),format = ("%b %Y %d")))

Sys.setlocale(locale = "C")
```


```r
head(Niger_ea_2021$DebutAssistance, 5)

Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(DebutAssistance = as.Date(as.character(dplyr::recode(DebutAssistance,                                                         "1"="2015-01-01",
        "2"="2015-01-01",
        "3"="2016-01-01",
        "4"="2017-01-01",
        "5"="2018-01-01",
        "6"="2019-01-01",
        "7"="2020-01-01",
        "8"="2021-01-01"))))
```


```r
head(Niger_pdm_2021$DebutAssistance, 5)

Niger_pdm_2021 <- Niger_pdm_2021 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance,format = ("%Y-%m-%dT%H:%M:%OS")))
```



```r
tail(Niger_pdm_2022$DebutAssistance, 5)

Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(DebutAssistance = as.Date(as.character(dplyr::recode(DebutAssistance,                                                         "1"="2015-01-01",
        "2"="2016-01-01",
        "3"="2017-01-01",
        "4"="2018-01-01",
        "5"="2019-01-01",
        "6"="2020-01-01",
        "7"="2021-01-01",
        "8"="2022-01-01"))))

#Niger_pdm_2022 <- Niger_pdm_2022 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
head(Niger_pdm_2020$DebutAssistance, 5)
Niger_pdm_2020 <- Niger_pdm_2020 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance,format = ("%Y-%m-%dT%H:%M:%OS")))
```


```r
head(Niger_pdm_2019$DebutAssistance, 5)

Niger_pdm_2019 <- Niger_pdm_2019 %>% dplyr::mutate(DebutAssistance =            as.Date(DebutAssistance, origin = "1899-12-30"))
```
## Date last assistance check


```r
expss::val_lab(Niger_baseline_2018$DateDerniereAssist)#Null
expss::val_lab(Niger_ea_2019$DateDerniereAssist)#Null
expss::val_lab(Niger_ea_2020$DateDerniereAssist)#Null
expss::val_lab(Niger_ea_2021$DateDerniereAssist)#Null
expss::val_lab(Niger_pdm_2021$DateDerniereAssist)#Null
expss::val_lab(Niger_pdm_2022$DateDerniereAssist)#Null
expss::val_lab(Niger_pdm_2020$DateDerniereAssist)#Null
expss::val_lab(Niger_pdm_2019$DateDerniereAssist)#Null

unique(Niger_baseline_2018$DateDerniereAssist)#Null
unique(Niger_ea_2019$DateDerniereAssist)#Null
unique(Niger_ea_2020$DateDerniereAssist)#Null
unique(Niger_ea_2021$DateDerniereAssist)#Null
unique(Niger_pdm_2021$DateDerniereAssist)#Null
unique(Niger_pdm_2022$DateDerniereAssist)#Null
unique(Niger_pdm_2020$DateDerniereAssist)#Null
unique(Niger_pdm_2019$DateDerniereAssist)#


expss::val_lab(Niger_pdm_2020$DateDerniereAssist)
Niger_pdm_2020 %>% 
  plot_frq(coord.flip =T,DateDerniereAssist,show.na =T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

```r
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate_at(c("DateDerniereAssist"),recode,"1"=1,"2"=2,"3"=3,"4"=3,"5"=3, "6"= NA_real_ )

Niger_pdm_2020$DateDerniereAssist <- labelled::labelled(Niger_pdm_2020$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

unique(Niger_pdm_2021$DateDerniereAssist)
Niger_pdm_2021<- Niger_pdm_2021 %>% dplyr::mutate(DateDerniereAssist = DateDerniereAssist/7)
Niger_pdm_2021 <- Niger_pdm_2021 %>%
  mutate(
    DateDerniereAssist = case_when(
      DateDerniereAssist < 1 ~ 1,
      DateDerniereAssist >= 1 & DateDerniereAssist < 3 ~ 2,
      DateDerniereAssist >= 3  ~ 3))

Niger_pdm_2021$DateDerniereAssist <- labelled::labelled(Niger_pdm_2021$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))



expss::val_lab(Niger_pdm_2022$DateDerniereAssist)
unique(Niger_pdm_2022$DateDerniereAssist)
Niger_pdm_2022 %>% 
  plot_frq(coord.flip =T,DateDerniereAssist,show.na =T)
```

![](01_Niger_LabelsHarmonization_files/figure-html/unnamed-chunk-15-2.png)<!-- -->

```r
Niger_pdm_2022 <- 
Niger_pdm_2022 %>% dplyr::mutate_at(c("DateDerniereAssist"),recode,"1"=1,"2"=2,"3"=3,"other"=3)

Niger_pdm_2022$DateDerniereAssist <- labelled::labelled(Niger_pdm_2022$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

Niger_baseline_2018$DateDerniereAssist <- labelled::labelled(Niger_baseline_2018$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

Niger_ea_2019$DateDerniereAssist <- labelled::labelled(Niger_ea_2019$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

Niger_ea_2020$DateDerniereAssist <- labelled::labelled(Niger_ea_2020$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

Niger_ea_2021$DateDerniereAssist <- labelled::labelled(Niger_ea_2021$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))

Niger_pdm_2019$DateDerniereAssist <- labelled::labelled(Niger_pdm_2019$DateDerniereAssist, c("moins d’une semaine" = 1, "entre 1 et 3 semaines" = 2,"plus de 3 semaines"=3))
```

## CashTransfert

```r
expss::val_lab(Niger_baseline_2018$CashTransfert)
unique(Niger_baseline_2018$CashTransfert)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_ea_2019$CashTransfert)
unique(Niger_ea_2019$CashTransfert)
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_ea_2020$CashTransfert)
unique(Niger_ea_2020$CashTransfert)
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_ea_2021$CashTransfert)
unique(Niger_ea_2021$CashTransfert)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_pdm_2021$CashTransfert)
unique(Niger_pdm_2021$CashTransfert)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,"0" = NA_real_))

expss::val_lab(Niger_pdm_2022$CashTransfert)
unique(Niger_pdm_2022$CashTransfert)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))


expss::val_lab(Niger_pdm_2020$CashTransfert)
unique(Niger_pdm_2020$CashTransfert)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))

Niger_pdm_2020$CashTransfert <- NA

expss::val_lab(Niger_pdm_2019$CashTransfert)
unique(Niger_pdm_2019$CashTransfert)
Niger_pdm_2019 <- 
 Niger_pdm_2019 %>% dplyr::mutate(CashTransfert = dplyr::recode(CashTransfert,"1"=1,"2"=2,"3"=3,.default = NA_real_))
```


## Type d'assistance


```r
var_type_assistance = c("BanqueCerealiere",
"TransfBenef",
"VivreContreTravail",
"ArgentContreTravail",
"ArgentetVivreContreTravail",
"DistribVivresSoudure",
"DistribArgentSoudure",
"BoursesAdo",
"BlanketFeedingChildren",
"BlanketFeedingWomen",
"MAMChildren",
"MASChildren",
"MAMPLWomen",
"FARNcommunaut",
"FormationRenfCapacite",
"CashTransfert",
"CantineScolaire"
)

expss::val_lab(Niger_baseline_2018$BanqueCerealiere)
unique(Niger_baseline_2018$BanqueCerealiere)


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```

```r
expss::val_lab(Niger_ea_2019$BanqueCerealiere)
unique(Niger_ea_2019$BanqueCerealiere)


Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```

```r
expss::val_lab(Niger_ea_2020$BanqueCerealiere)
unique(Niger_ea_2020$BanqueCerealiere)


Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```


```r
expss::val_lab(Niger_ea_2021$BanqueCerealiere)
unique(Niger_ea_2021$BanqueCerealiere)

Niger_ea_2021 <- Niger_ea_2021 %>%
  mutate(across(var_type_assistance,
                ~if_else(. == 0, NA_real_, .)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```


```r
expss::val_lab(Niger_pdm_2021$BanqueCerealiere)
unique(Niger_pdm_2021$BanqueCerealiere)

Niger_pdm_2021 <- Niger_pdm_2021 %>%
  mutate(across(var_type_assistance,
                ~if_else(. == 0, NA_real_, .)))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```


```r
expss::val_lab(Niger_pdm_2022$BanqueCerealiere)
unique(Niger_pdm_2022$BanqueCerealiere)

Niger_pdm_2022 <- Niger_pdm_2022 %>%
  mutate(across(var_type_assistance,
                ~if_else(. == 0, NA_real_, .)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```
## ArgentContreTravail

```r
expss::val_lab(Niger_baseline_2018$ArgentContreTravail)
unique(Niger_baseline_2018$ArgentContreTravail)


expss::val_lab(Niger_ea_2019$ArgentContreTravail)
unique(Niger_ea_2019$ArgentContreTravail)


expss::val_lab(Niger_ea_2020$ArgentContreTravail)
unique(Niger_ea_2020$ArgentContreTravail)


expss::val_lab(Niger_ea_2021$ArgentContreTravail)
unique(Niger_ea_2021$ArgentContreTravail)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(ArgentContreTravail = dplyr::recode(ArgentContreTravail,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_pdm_2021$ArgentContreTravail)
unique(Niger_pdm_2021$ArgentContreTravail)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(ArgentContreTravail = dplyr::recode(ArgentContreTravail,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_pdm_2022$ArgentContreTravail)
unique(Niger_pdm_2022$ArgentContreTravail)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(ArgentContreTravail = dplyr::recode(ArgentContreTravail,"1"=1,"2"=2,"3"=3,.default = NA_real_))


expss::val_lab(Niger_pdm_2020$ArgentContreTravail)
unique(Niger_pdm_2020$ArgentContreTravail)


expss::val_lab(Niger_pdm_2019$ArgentContreTravail)
unique(Niger_pdm_2019$ArgentContreTravail)
```

## BoursesAdo

```r
expss::val_lab(Niger_baseline_2018$BoursesAdo)
unique(Niger_baseline_2018$BoursesAdo)


expss::val_lab(Niger_ea_2019$BoursesAdo)
unique(Niger_ea_2019$BoursesAdo)


expss::val_lab(Niger_ea_2020$BoursesAdo)
unique(Niger_ea_2020$BoursesAdo)


expss::val_lab(Niger_ea_2021$BoursesAdo)
unique(Niger_ea_2021$BoursesAdo)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(BoursesAdo = dplyr::recode(BoursesAdo,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_pdm_2021$BoursesAdo)
unique(Niger_pdm_2021$BoursesAdo)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(BoursesAdo = dplyr::recode(BoursesAdo,"1"=1,"2"=2,"3"=3,.default = NA_real_))

expss::val_lab(Niger_pdm_2022$BoursesAdo)
unique(Niger_pdm_2022$BoursesAdo)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(BoursesAdo = dplyr::recode(BoursesAdo,"1"=1,"2"=2,"3"=3,.default = NA_real_))


expss::val_lab(Niger_pdm_2020$BoursesAdo)
unique(Niger_pdm_2020$BoursesAdo)

Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(BoursesAdo = dplyr::recode(BoursesAdo,"1"=1,"2"=2,"3"=3,.default = NA_real_))


expss::val_lab(Niger_pdm_2019$BoursesAdo)
unique(Niger_pdm_2019$BoursesAdo)
```




```r
var_type_assistance = c("BanqueCerealiere",
"TransfBenef",
"VivreContreTravail",
"ArgentContreTravail",
"ArgentetVivreContreTravail",
"DistribVivresSoudure",
"DistribArgentSoudure",
"BoursesAdo",
"BlanketFeedingChildren",
"BlanketFeedingWomen",
"MAMChildren",
"MASChildren",
"MAMPLWomen",
"FARNcommunaut",
"FormationRenfCapacite",
"CashTransfert",
"CantineScolaire"
)

expss::val_lab(Niger_pdm_2020$VivreContreTravail)
unique(Niger_pdm_2020$VivreContreTravail)



Niger_pdm_2020 <- Niger_pdm_2020 %>% 
 dplyr::mutate(across(all_of(var_type_assistance),
               ~labelled(as.numeric(as.character(.)), labels = c(
                 "Oui PAM" = 1,
                 "Oui, AUtre" = 2,
                 "NSP" = 3))))
```


```r
expss::val_lab(Niger_pdm_2019$BanqueCerealiere)
unique(Niger_pdm_2019$BanqueCerealiere)


Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(var_type_assistance,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui PAM" = 1,
                  "Oui, AUtre" = 2,
                  "NSP" = 3))))
```


## Participation à un groupe d'épargne 


```r
var_type_epargne = c("ExistGroupeEpargne",
"MembreGroupeEpargne",
"EpargneAvantPam",
"EpargneSansPam",
"PossibilitePret",
"AutreSourcePret",
"EpargnePieds"
)

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(var_type_epargne,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Oui" = 1,
                  "Non" = 0,
                  "Ne sait pas" = 888))))
```

## ExistGroupeEpargne


```r
expss::val_lab(Niger_baseline_2018$ExistGroupeEpargne)
unique(Niger_baseline_2018$ExistGroupeEpargne)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2019$ExistGroupeEpargne)
unique(Niger_ea_2019$ExistGroupeEpargne)
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2020$ExistGroupeEpargne)
unique(Niger_ea_2020$ExistGroupeEpargne)
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2021$ExistGroupeEpargne)
unique(Niger_ea_2021$ExistGroupeEpargne)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2021$ExistGroupeEpargne)
unique(Niger_pdm_2021$ExistGroupeEpargne)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2022$ExistGroupeEpargne)
unique(Niger_pdm_2022$ExistGroupeEpargne)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))


expss::val_lab(Niger_pdm_2020$ExistGroupeEpargne)
unique(Niger_pdm_2020$ExistGroupeEpargne)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888, .default = NA_real_))



expss::val_lab(Niger_pdm_2019$ExistGroupeEpargne)
unique(Niger_pdm_2019$ExistGroupeEpargne)
Niger_pdm_2019 <- 
 Niger_pdm_2019 %>% dplyr::mutate(ExistGroupeEpargne = dplyr::recode(ExistGroupeEpargne,"0"=0,"1"=1,"888"=888,.default = NA_real_))
```

```r
expss::val_lab(Niger_baseline_2018$PossibilitePret)
unique(Niger_baseline_2018$PossibilitePret)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2019$PossibilitePret)
unique(Niger_ea_2019$PossibilitePret)
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2020$PossibilitePret)
unique(Niger_ea_2020$PossibilitePret)
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2021$PossibilitePret)
unique(Niger_ea_2021$PossibilitePret)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2021$PossibilitePret)
unique(Niger_pdm_2021$PossibilitePret)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2022$PossibilitePret)
unique(Niger_pdm_2022$PossibilitePret)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))


expss::val_lab(Niger_pdm_2020$PossibilitePret)
unique(Niger_pdm_2020$PossibilitePret)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))



expss::val_lab(Niger_pdm_2019$PossibilitePret)
unique(Niger_pdm_2019$PossibilitePret)
Niger_pdm_2019 <- 
 Niger_pdm_2019 %>% dplyr::mutate(PossibilitePret = dplyr::recode(PossibilitePret,"0"=0,"1"=1,"888"=888,.default = NA_real_))
```


```r
expss::val_lab(Niger_baseline_2018$AutreSourcePret)
unique(Niger_baseline_2018$AutreSourcePret)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2019$AutreSourcePret)
unique(Niger_ea_2019$AutreSourcePret)
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2020$AutreSourcePret)
unique(Niger_ea_2020$AutreSourcePret)
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2021$AutreSourcePret)
unique(Niger_ea_2021$AutreSourcePret)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2021$AutreSourcePret)
unique(Niger_pdm_2021$AutreSourcePret)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2022$AutreSourcePret)
unique(Niger_pdm_2022$AutreSourcePret)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))


expss::val_lab(Niger_pdm_2020$AutreSourcePret)
unique(Niger_pdm_2020$AutreSourcePret)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888, .default = NA_real_))



expss::val_lab(Niger_pdm_2019$AutreSourcePret)
unique(Niger_pdm_2019$AutreSourcePret)
Niger_pdm_2019 <- 
 Niger_pdm_2019 %>% dplyr::mutate(AutreSourcePret = dplyr::recode(AutreSourcePret,"0"=0,"1"=1,"888"=888,.default = NA_real_))
```


```r
expss::val_lab(Niger_baseline_2018$EpargnePieds)
unique(Niger_baseline_2018$EpargnePieds)
Niger_baseline_2018 <- 
  Niger_baseline_2018 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2019$EpargnePieds)
unique(Niger_ea_2019$EpargnePieds)
Niger_ea_2019 <- 
  Niger_ea_2019 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2020$EpargnePieds)
unique(Niger_ea_2020$EpargnePieds)
Niger_ea_2020 <- 
  Niger_ea_2020 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_ea_2021$EpargnePieds)
unique(Niger_ea_2021$EpargnePieds)
Niger_ea_2021 <- 
  Niger_ea_2021 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2021$EpargnePieds)
unique(Niger_pdm_2021$EpargnePieds)
Niger_pdm_2021 <- 
  Niger_pdm_2021 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))

expss::val_lab(Niger_pdm_2022$EpargnePieds)
unique(Niger_pdm_2022$EpargnePieds)
Niger_pdm_2022 <- 
  Niger_pdm_2022 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))


expss::val_lab(Niger_pdm_2020$EpargnePieds)
unique(Niger_pdm_2020$EpargnePieds)
Niger_pdm_2020 <- 
  Niger_pdm_2020 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888, .default = NA_real_))



expss::val_lab(Niger_pdm_2019$EpargnePieds)
unique(Niger_pdm_2019$EpargnePieds)
Niger_pdm_2019 <- 
 Niger_pdm_2019 %>% dplyr::mutate(EpargnePieds = dplyr::recode(EpargnePieds,"0"=0,"1"=1,"888"=888,.default = NA_real_))
```
##################################
# DEPART EN EXODE ET MIGRATION  







##MigrationEmploi




```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```



```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```



```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```


```r
# Niger_ea_2021 <- Niger_ea_2021 %>% 
#   dplyr::mutate(across(MigrationEmploi,
#                 ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0)))
# 
 Niger_ea_2021 <- Niger_ea_2021 %>% 
   dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                   "Non" = 0,
                   "Oui" = 1 ))))
```


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```



```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2` = 0)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(MigrationEmploi,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ))))
```







## Raison migration


```r
#Niger_baseline_2018$RaisonMigration
# Convert labelled vector to numeric vector
#Niger_baseline_2018$RaisonMigration_numeric <- as.numeric(Niger_baseline_2018$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=1, "3"=6, "4"=9, "2"= 5))

# Convert labelled vector to numeric vector
Niger_baseline_2018$RaisonMigration <- as.numeric(Niger_baseline_2018$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=1, "3"=6, "9"=9, "5"=4))


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
#Niger_ea_2019$RaisonMigration
# Convert labelled vector to numeric vector
#Niger_ea_2019$RaisonMigration_numeric <- as.numeric(Niger_ea_2019$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=10, "2"= 1 ,"3"=11, "4"=6, "5"=9))





# Convert labelled vector to numeric vector
Niger_ea_2019$RaisonMigration <- as.numeric(Niger_ea_2019$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "10"=4, "2"= 1,"11"=5, "6"=6, "9"=9))


Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
#Niger_ea_2020$RaisonMigration
# Convert labelled vector to numeric vector
#Niger_ea_2020$RaisonMigration_numeric <- as.numeric(Niger_ea_2020$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=10, "2"= 1 ,"3"=11, "4"=6, "5"=9))



# Convert labelled vector to numeric vector
Niger_ea_2020$RaisonMigration <- as.numeric(Niger_ea_2020$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "10"=4, "2"= 1,"11"=5, "6"=6, "9"=9))


Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
#Niger_ea_2021$RaisonMigration
# Convert labelled vector to numeric vector
#Niger_ea_2021$RaisonMigration_numeric <- as.numeric(Niger_ea_2021$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6,"7"=7,"8"=8,"9"=9))



# # Convert labelled vector to numeric vector
# Niger_ea_2021$RaisonMigration <- as.numeric(Niger_ea_2021$RaisonMigration)
# 
# # Apply recoding using dplyr::recode
# Niger_ea_2021 <- Niger_ea_2021 %>% 
#   dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=1, "3"=6, "4"=9, "5"=4))


Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
#Niger_pdm_2021$RaisonMigration
# Convert labelled vector to numeric vector
#Niger_pdm_2021$RaisonMigration_numeric <- as.numeric(Niger_pdm_2021$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "1"=10 ,"3"=11, "4"=6, "5"=9,"other" = 9))




# Convert labelled vector to numeric vector
Niger_pdm_2021$RaisonMigration <- as.numeric(Niger_pdm_2021$RaisonMigration)

# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(RaisonMigration = dplyr::recode(RaisonMigration, "10"=4,"11"=5, "6"=6, "9"=9 , .default = 9))


Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(RaisonMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                 "Recherche d’opportunités économiques" = 1,
                 "Catastrophes naturelles (par ex., inondations, sécheresse, etc.)" = 2,
                 "Accès aux services de base (santé, éducation…)" = 3,
                 "Difficultés alimentaires conjoncturelles" = 4,
                  "Uniquement en année de crise alimentaire" = 5,
                 "La migration fait désormais partie des moyens d’existence classique" = 6,
                 "Guerre/conflit"= 7, 
                 "Violence ciblée ou persécution"= 8,
                 "Autres à préciser" = 9
                 ))))
```



## DureeMigration



```r
Niger_baseline_2018$DureeMigration


# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5))



# # Convert labelled vector to numeric vector
# Niger_baseline_2018$DureeMigration <- as.numeric(Niger_baseline_2018$DureeMigration)
# 
# # Apply recoding using dplyr::recode
# Niger_baseline_2018 <- Niger_baseline_2018 %>% 
#   dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "3"=6, "9"=9, "5"=5))


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```







```r
# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```



```r
# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5, "6"=6))



Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```


```r
# Apply recoding using dplyr::recode
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5, "6"=6))



Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```


```r
# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(DureeMigration = dplyr::recode(DureeMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5, "6"=6, "other"=6,.default = NA_real_))



Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(DureeMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Moins d’un mois" = 1,
                                                        "1 à 3 mois"=2,
                                                        "3 à 6 mois"=3,                             
                                                        "6 à 9 mois"=4,
                                                        "10 à 12 mois"=5,
                                                        "Plus de 12 mois"=6))))
```



##TendanceMigration


```r
Niger_baseline_2018$TendanceMigration


# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))



# # Convert labelled vector to numeric vector
# Niger_baseline_2018$TendanceMigration <- as.numeric(Niger_baseline_2018$TendanceMigration)
# 
# # Apply recoding using dplyr::recode
# Niger_baseline_2018 <- Niger_baseline_2018 %>% 
#   dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "3"=6, "9"=9, "5"=5))


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```





```r
Niger_ea_2019$TendanceMigration


# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))





Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```


```r
Niger_ea_2020$TendanceMigration


# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))



Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```


```r
# Apply recoding using dplyr::recode
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))





Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```



```r
Niger_pdm_2021$TendanceMigration


# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))



Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```


```r
# Apply recoding using dplyr::recode
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(TendanceMigration = dplyr::recode(TendanceMigration, "1"=1, "2"=2, "3"=3, "4"= 4, "5"=5,"6"=6))


Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(TendanceMigration,
                ~labelled(as.numeric(as.character(.)), labels = c(
                                                        "Beaucoup augmenté" = 1,
                                                        "Légèrement augmenté"=2, 
                                                        " Stable"=3,  
                                                        "Beaucoup baissé"=4, 
                                                        "Légèrement baissé"=5, 
                                                        "Ne sait pas"=6 ))))
```

##RaisonHausseMig


```r
Niger_baseline_2018$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig, "1"=10, "2"=1, "3"=11, "4"= 5))



# Convert labelled vector to numeric vector
Niger_baseline_2018$RaisonHausseMig <- as.numeric(Niger_baseline_2018$RaisonHausseMig)

# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>%
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"11"=4,"5"=5 ))


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```



```r
Niger_ea_2019$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig, "1"=10, "2"=1, "3"=11, "4"= 5))



# Convert labelled vector to numeric vector
Niger_ea_2019$RaisonHausseMig <- as.numeric(Niger_ea_2019$RaisonHausseMig)

# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>%
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"11"=4,"5"=5 ))


Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```






```r
Niger_ea_2020$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig, "1"=10, "2"=1, "3"=11, "4"= 5))



# Convert labelled vector to numeric vector
Niger_ea_2020$RaisonHausseMig <- as.numeric(Niger_ea_2020$RaisonHausseMig)

# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>%
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"11"=4,"5"=5 ))


Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```



```r
Niger_ea_2021$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig, "1"=1, "2"=2, "3"=3, "4"= 4))



# # Convert labelled vector to numeric vector
# Niger_ea_2021$RaisonHausseMig <- as.numeric(Niger_ea_2021$RaisonHausseMig)
# 
# # Apply recoding using dplyr::recode
# Niger_ea_2021 <- Niger_ea_2021 %>%
#   dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"4"=4 ))


Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```





```r
Niger_pdm_2021$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"1"=1, "2"=2, "3"=3, "4"= 4,.default = NA_real_))



# # Convert labelled vector to numeric vector
# Niger_pdm_2021$RaisonHausseMig <- as.numeric(Niger_pdm_2021$RaisonHausseMig)
# 
# # Apply recoding using dplyr::recode
# Niger_pdm_2021 <- Niger_pdm_2021 %>%
#   dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"11"=4,"5"=5 ))


Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```


```r
Niger_pdm_2019$RaisonHausseMig


# Apply recoding using dplyr::recode
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig, "1"=10, "2"=1, "3"=11, "4"= 5))



# Convert labelled vector to numeric vector
Niger_pdm_2019$RaisonHausseMig <- as.numeric(Niger_pdm_2019$RaisonHausseMig)

# Apply recoding using dplyr::recode
Niger_pdm_2019 <- Niger_pdm_2019 %>%
  dplyr::mutate(RaisonHausseMig = dplyr::recode(RaisonHausseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"11"=4,"5"=5 ))


Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(RaisonHausseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                                                                                   "Difficultés alimentaires conjoncturelles"= 1,
                                                                  "Manque d’opportunités économiques"=2,
                                                                  "Dégradation de l’environnement (pertes de bétail, baisse de la production et du rendement à cause de la sécheresse, et des faibles précipitations, etc.)"= 3,                                                                   "La migration fait désormais partie des moyens d’existence classique "=4,"Autres à préciser "=5))))
```



## RaisonBaisseMig


1.Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie
2.Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux
3.Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA
4.La situation alimentaire générale du village s’est améliorée
5. La migration fait désormais partie des moyens d’existence classique 6. Emergence d’opportunités économiques grâce aux actifs créés/réhabilités
7. Autres à préciser 







```r
Niger_baseline_2018$RaisonBaisseMig

# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig, "1"=3, "2"=10, "3"=3, "4"= 11,"5" =7))
# Convert labelled vector to numeric vector
Niger_baseline_2018$RaisonBaisseMig <- as.numeric(Niger_baseline_2018$RaisonBaisseMig)

# Apply recoding using dplyr::recode
Niger_baseline_2018 <- Niger_baseline_2018 %>%
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=4, "3"=3,"6"=6,"7"=7, "11"= 5))


Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                    ~labelled(as.numeric(as.character(.)), labels = c(                                                 "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7
    ))))
```


```r
Niger_ea_2019$RaisonBaisseMig
# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig, "1"=3, "2"=10, "3"=3, "4"= 11,"5" =7))



# Convert labelled vector to numeric vector
Niger_ea_2019$RaisonBaisseMig <- as.numeric(Niger_ea_2019$RaisonBaisseMig)

# Apply recoding using dplyr::recode
Niger_ea_2019 <- Niger_ea_2019 %>%
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=4, "3"=3,"6"=6,"7"=7, "11"= 5))


Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                  "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7 ))))
```


```r
Niger_ea_2020$RaisonBaisseMig
# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig, "1"=3, "2"=10, "3"=3, "4"= 11,"5" =7))
# Convert labelled vector to numeric vector
Niger_ea_2020$RaisonBaisseMig <- as.numeric(Niger_ea_2020$RaisonBaisseMig)

# Apply recoding using dplyr::recode
Niger_ea_2020 <- Niger_ea_2020 %>%
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=4, "3"=3,"6"=6,"7"=7, "11"= 5 ))


Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                  "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7))))
```







```r
Niger_ea_2021$RaisonBaisseMig
# Apply recoding using dplyr::recode
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig, "1"=1, "2"=2, "3"=3, "4"= 4,"5" =5,"6"=6,"7"=7,"8"=7))

# # Convert labelled vector to numeric vector
# Niger_ea_2021$RaisonBaisseMig <- as.numeric(Niger_ea_2021$RaisonBaisseMig)
# 
# # Apply recoding using dplyr::recode
# Niger_ea_2021 <- Niger_ea_2021 %>%
#   dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=5,"1"=1, "2"=2,"3"=3 ,"4"=4 ))


Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                  "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7))))
```





```r
Niger_pdm_2021$RaisonBaisseMig
# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"1"=3, "2"=10, "3"=3, "4"= 11,"5" =7,.default = NA_real_))



# Convert labelled vector to numeric vector
Niger_pdm_2021$RaisonBaisseMig <- as.numeric(Niger_pdm_2021$RaisonBaisseMig)

# Apply recoding using dplyr::recode
Niger_pdm_2021 <- Niger_pdm_2021 %>%
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=4, "3"=3,"6"=6,"7"=7, "11"= 5 ))


Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                 "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7 ))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                 "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                 "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7))))
```


```r
Niger_pdm_2019$RaisonBaisseMig
# Apply recoding using dplyr::recode
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig, "1"=3, "2"=10, "3"=3, "4"= 11,"5" =7,.default = NA_real_))



# Convert labelled vector to numeric vector
Niger_pdm_2019$RaisonBaisseMig <- as.numeric(Niger_pdm_2019$RaisonBaisseMig)

# Apply recoding using dplyr::recode
Niger_pdm_2019 <- Niger_pdm_2019 %>%
  dplyr::mutate(RaisonBaisseMig = dplyr::recode(RaisonBaisseMig,"10"=4, "3"=3,"6"=6,"7"=7, "11"= 5))


Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(RaisonBaisseMig,
                ~labelled(as.numeric(as.character(.)), labels = c(                                                 "Moins d’opportunités économiques ou insécurité au Nigéria ou en Lybie"= 1,
    "Voyage vers la Lybie/ Nigeria devenu trop dangereux/ couteux"=2,
    "Les ménages pauvres ont accès à une assistance régulière/ les bras valides sont occupés par les travaux FFA"=3,
    "La situation alimentaire générale du village s’est améliorée"=4,
    "La migration fait désormais partie des moyens d’existence classique"=5,
    "Emergence d’opportunités économiques grâce aux actifs créés/réhabilités"=6,
    "Autres à préciser "=7))))
```

# Social capital index (Indice de capital social)




## SCIConMembreGvrnmt


```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```

## SCIPersConMembreGvrnmt



```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)),`1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))


Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreGvrnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```

## SCIConMembreNGO

```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across( SCIConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```

## SCIPersConMembreNGO

```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)),`1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))


Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~recode(as.numeric(as.character(.)), `1` = 1, `2`=2, `3`=3,`4`=4,`888`=8,`8888`=9,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(SCIPersConMembreNGO,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Un membre de la famille ou un parent" = 1,
                  "Ami(e)" = 2 ,
                  "Voisin" =3,
                  "Connaissance (membre d'un groupe, ami d'un ami, etc.)"=4,
                  "Autre (précisez)"=5,
                  
                   "Ne sait pas" = 8,
                  "Refuse de répondre"= 9))))
```

## SCIAideAubesoin

```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across( SCIAideAubesoin,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```

##  SCICapAideGvnmt


```r
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```


```r
Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```



```r
Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~recode(as.numeric(as.character(.)), `1` = 1, `0` = 0,.default = NA_real_)))

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(  SCICapAideGvnmt,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1 ,
                   "Ne sait pas" = 888,
                  "Refuse de répondre"= 8888))))
```

# DIVERSITE ALIMENTAIRE DES FEMMES 






```r
#  PWMDDWOth1
# PWMDDWCond
#  PWMDDWSnf
#PWMDDWFruitOth 
#PWMDDWFruitOrg 
# PWMDDWVegOth
# PWMDDWVegOrg 
# PWMDDWVegGre 
# PWMDDWPrFish 
# PWMDDWPrMeatF
#PWMDDWPrMeatO 
# PWMDDWDairy
# PWMDDWNuts
# PWMDDWPulse
#PWMDDWStapRoo 
# PWMDDWStapCer
```



```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs
Niger_pdm_2020 <- Niger_pdm_2020 %>%
  mutate(across(all_of(sers_variables), ~ as.numeric(as.character(.)))) %>%
  mutate(across(all_of(sers_variables), ~ recode(., `0` = 0, `1` = 1)))

# Vérifiez qu'il n'y a pas de valeurs manquantes dans les colonnes sélectionnées
Niger_pdm_2020 <- Niger_pdm_2020 %>%
  mutate(across(all_of(sers_variables), ~ ifelse(is.na(.), 0, .)))



Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```



```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs

Niger_baseline_2018 <- Niger_baseline_2018 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs

Niger_ea_2019 <- Niger_ea_2019 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs

Niger_ea_2020 <- Niger_ea_2020 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs
Niger_ea_2021 <- Niger_ea_2021 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```



```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs

Niger_pdm_2021 <- Niger_pdm_2021 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs

Niger_pdm_2022 <- Niger_pdm_2022 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

# Conversion des colonnes sélectionnées en numérique et recodage des valeurs
Niger_pdm_2020 <- Niger_pdm_2020 %>%
  mutate(across(all_of(sers_variables), ~ as.numeric(as.character(.)))) %>%
  mutate(across(all_of(sers_variables), ~ recode(., `0` = 0, `1` = 1)))


Niger_pdm_2020 <- Niger_pdm_2020 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```


```r
# Sélection des colonnes pertinentes
sers_variables <- c("PWMDDWOth1", "PWMDDWCond", "PWMDDWSnf", "PWMDDWFruitOth", 
                    "PWMDDWFruitOrg", "PWMDDWVegOth", "PWMDDWVegOrg", "PWMDDWVegGre", 
                    "PWMDDWPrFish", "PWMDDWPrMeatF", "PWMDDWPrMeatO", "PWMDDWDairy", 
                    "PWMDDWNuts", "PWMDDWPulse", "PWMDDWStapRoo", "PWMDDWStapCer")

Niger_pdm_2019 <- Niger_pdm_2019 %>% 
  dplyr::mutate(across(sers_variables,
                ~labelled(as.numeric(as.character(.)), labels = c(
                  "Non" = 0,
                  "Oui" = 1))))
```

# Merging all data

## drop variables not needed
Pour merger les bases, Niger_ea_2019, Niger_pdm_2019, Niger_ea_2020 et Niger_pdm_2020 causait des problèmes à cause du type de labels de certaines variables ( qui n'étaient pas traitées dans le code). Donc ici nous identifion d'abord cees varibles puis nous les supprimons (variables suivies de "#" dans le chunck "variables not needed")





```r
data <- Niger_pdm_2019

# Fonction pour obtenir les types de labels
get_label_types <- function(data) {
  label_types <- lapply(data, function(variable) {
    if (is.labelled(variable)) {
      return(class(variable))
    } else {
      return("No labels")
    }
  })
  return(label_types)
}

# Afficher les types de labels pour chaque variable
label_types <- get_label_types(data)
for (i in seq_along(label_types)) {

      cat("Variable", names(label_types)[i], ": ", label_types[[i]], "\n")
}
```


```r
var_to_drop = c("RESPConsent",
                "ADMIN3Name",#
                "RESPAge",
                "RESPSex",#
                "HHHMainActivity",#
                "HHSourceIncome",#
                "AutreTransferts"

)

Niger_baseline_2018 <- Niger_baseline_2018  %>% dplyr::select(-var_to_drop)
Niger_ea_2019 <- Niger_ea_2019  %>% dplyr::select(-var_to_drop)
Niger_ea_2020 <- Niger_ea_2020  %>% dplyr::select(-var_to_drop)
Niger_ea_2021 <- Niger_ea_2021 %>% dplyr::select(-var_to_drop)
Niger_pdm_2021 <- Niger_pdm_2021  %>% dplyr::select(-var_to_drop)
Niger_pdm_2022 <- Niger_pdm_2022  %>% dplyr::select(-var_to_drop)
Niger_pdm_2020 <-Niger_pdm_2020 %>% dplyr::select(-var_to_drop)
Niger_pdm_2019 <- Niger_pdm_2019  %>% dplyr::select(-var_to_drop)
```


```r
Niger_baseline_2018 <- labelled::to_factor(Niger_baseline_2018)
Niger_ea_2019 <- labelled::to_factor(Niger_ea_2019)
Niger_ea_2020 <- labelled::to_factor(Niger_ea_2020)
Niger_ea_2021 <- labelled::to_factor(Niger_ea_2021)
Niger_pdm_2021 <- labelled::to_factor(Niger_pdm_2021)
Niger_pdm_2022 <- labelled::to_factor(Niger_pdm_2022)
Niger_pdm_2020 <- labelled::to_factor(Niger_pdm_2020)
#Niger_pdm_2019 <- labelled::to_factor(as.character(Niger_pdm_2019))
Niger_pdm_2019 <- labelled::to_factor(Niger_pdm_2019)


WFP_Niger <-plyr::rbind.fill(as.data.frame(Niger_baseline_2018),
as.data.frame(Niger_ea_2019),
as.data.frame(Niger_ea_2020),
as.data.frame(Niger_ea_2021),
as.data.frame(Niger_pdm_2021),
as.data.frame(Niger_pdm_2022),
as.data.frame(Niger_pdm_2020),
as.data.frame(Niger_pdm_2019))

'WFP_Niger <-plyr::rbind.fill(Niger_baseline_2018,
Niger_ea_2019,
Niger_ea_2020,
Niger_ea_2021,
Niger_pdm_2021,
Niger_pdm_2022,
Niger_pdm_2020,
Niger_pdm_2019)'
```



```r
abi_variables = WFP_Niger %>% dplyr::select(gtsummary::starts_with("ABI")) %>% names()
abi_variables <- c(abi_variables,"ActifCreationEmploi",
"BeneficieEmploi", "TRavailMaintienActif")

WFP_Niger <- WFP_Niger %>% 
 dplyr::mutate(across(all_of(abi_variables),
               ~labelled(as.numeric(as.character(.)), labels = c(
                 "Oui PAM" = 0,
                 "Oui, AUtre" = 1,
                 "NSP" = 888))))


WFP_Niger <- WFP_Niger %>% 
 dplyr::mutate(across("HHHSex",
               ~labelled(as.numeric(as.character(.)), labels = c(
                 "Femme" = 0,
                 "Homme" = 1))))
```




```r
WFP_Niger.sub <- WFP_Niger  %>% dplyr::select(ID, adm0_ocha, ADMIN0Name,adm1_ocha,ADMIN1Name,adm2_ocha,ADMIN2Name,SURVEY,YEAR,SvyDatePDM,HHHSex ,HHHAge, HHHEdu,everything())

WFP_Niger <- copy_labels(WFP_Niger.sub, WFP_Niger)
#
```


# Cleaning dirty variables

## Remove empty rows and/or columns



## Remove constant columns




# Data exportation 

## Variables labels


```r
'data <- WFP_Niger

# Fonction pour obtenir les types de labels
get_label_types <- function(data) {
  label_types <- lapply(data, function(variable) {
    if (is.labelled(variable)) {
      return(class(variable))
    } else {
      return("No labels")
    }
  })
  return(label_types)
}

# Afficher les types de labels pour chaque variable
label_types <- get_label_types(data)
for (i in seq_along(label_types)) {

      cat("Variable", names(label_types)[i], ": ", label_types[[i]], "\n")
}'
```

# labelliser les variables 

```r
# Lire les métadonnées d'harmonisation
Niger_Harmonization_variables_labels <- 
read_excel(paste0(dir_input_data,"/NER_Harmonization.xlsx"), 
    sheet = "variables_labels")

# Identifier les variables communes
variable_to_labels <- intersect(Niger_Harmonization_variables_labels$Variable_Name, names(WFP_Niger))

# Créer un tribble de métadonnées
WFP_Niger.sub_metadata <- tibble(
    variable = Niger_Harmonization_variables_labels$Variable_Name,
    variable_label = Niger_Harmonization_variables_labels$Variable_Label
)

# Convertir en vecteur nommé
WFP_Niger.sub_labels <- deframe(WFP_Niger.sub_metadata)

# Sélectionner le sous-ensemble de données
df <- WFP_Niger |> dplyr::select(variable_to_labels)

# Filtrer les étiquettes pour ne garder que celles dont les variables existent dans df
valid_labels <- WFP_Niger.sub_labels[names(WFP_Niger.sub_labels) %in% names(df)]

# Appliquer les étiquettes filtrées aux variables de df
WFP_Niger.sub_labelled <- df |> set_variable_labels(!!!valid_labels)
```


```r
WFP_Niger.sub <- WFP_Niger.sub_labelled  %>% dplyr::select(ID, adm0_ocha, ADMIN0Name,adm1_ocha,ADMIN1Name,adm2_ocha,ADMIN2Name,SURVEY,YEAR,SvyDatePDM,HHHSex , HHHEdu,everything())

WFP_Niger.sub_labelled <- copy_labels(WFP_Niger.sub, WFP_Niger.sub_labelled)
#
```


```r
haven::write_dta(WFP_Niger.sub_labelled,"WFP_Niger.dta")
```



```r
#devtools::install_github("pcctc/croquet")
library(croquet)
library(openxlsx)

wb <- createWorkbook()
add_labelled_sheet(WFP_Niger)
saveWorkbook(wb, "WFP_Niger.xlsx",overwrite = TRUE)
```



```r
rm(list = ls())
```





