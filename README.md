```
FinalPLPA/
├── Data/
│   ├── No Iron (Correct Format).xlsx
│   └── Microplate_Key.xlsx
├── Script/
│   └── Final_Project.Rmd
├── Output/
│   └── 000014.png
└── README.md
```


# 96-Well Bacterial Growth Curve Analysis
For the use of generating publishable figures for 96-Well Plate Growth Curves

# Prerequisites:
1. Have RStudio Version 4.4.2
2. 96-Well Plate must be in similar format to Example_Plate.xlsx
3. A separate excel file as a "Key" - A file labeling each well. Unlabeled wells will automatically be marked as N/A. See Example_Key.xlsx for the specific format.

# Steps
1. Firstly, download Final_Project.Rmd
2. Run Final_Project.rmd in R studio
3. Import your dataset and key using the import dataset (Top Right Tab under "Environment")

# Code
Firstly, the code will read in your files. Note: you will need to manually change the name of your files. Plate_data must be your dataset, and Key_data must be your key file. 
This code follows the standard format for a 96-well plate reader, so the second column is dropped - assuming temperature is constant throughout the experiment. 

```{r}
# Load necessary libraries
library(tidyr)      # for pivot_longer (data reshaping)
library(dplyr)      # for joining and filtering
library(ggplot2)    # for plotting
library(readr)      # for cvs reading
library(readxl)
# 1. Read the plate reader data
plate_data <- read_excel("No Iron (Correct Format).xlsx") 

# Drop the second column (temperature or unrelated readings, e.g. "T 600") if present
plate_data <- plate_data[ , -2]

# 2. Read the microplate key (well layout) data, skipping the header row of column numbers
key_data <- read_excel("Microplate_Key.xlsx", skip = 1, na = c("", "NA"))
```

Afterwards, the code will standardize your key to a long format and assign names.

Next, the data will be formatted to a standard numeric format.
```{r}
# --- Load required packages ---
library(readxl)
library(dplyr)
library(tidyr)
library(ggplot2)

# --- Load plate data ---
plate_data <- read_excel("No Iron (Correct Format).xlsx")  # Update filename if different

# Convert time strings "HH:MM:SS" to numeric minutes
time_parts <- strsplit(as.character(plate_data$Time), ":", fixed = TRUE)

plate_data$Time_min <- as.numeric(difftime(plate_data$Time, plate_data$Time[1], units = "mins"))


# Convert OD values from character to numeric (if needed)
plate_data[ , !(names(plate_data) %in% c("Time", "Time_min"))] <- 
  lapply(plate_data[ , !(names(plate_data) %in% c("Time", "Time_min"))], as.numeric)
```
Then, the data and the key will be merged. Sanity checks are included to ensure the merged data is correct. If the check passes, then the code will plot the dataset.

```{r}
# --- Merge and validate ---
merged_data <- inner_join(plate_long, key_long, by = "Well")

# Clean up OD column if not already numeric
merged_data <- merged_data %>%
  mutate(OD = as.numeric(OD))

# Sanity checks
stopifnot(nrow(merged_data) > 0)
stopifnot(all(!is.na(merged_data$Time_min)))
stopifnot(all(!is.na(merged_data$OD)))


# --- Plot ---
ggplot(merged_data, aes(x = Time_min, y = OD, color = Condition,
                        group = interaction(Condition, Replicate))) +
  geom_line(size = 1) +
  labs(title = "Bacterial Growth Curves by Condition",
       x = "Time (min)", y = "OD600",
       color = "Condition") +
  theme_minimal()
```

Note: This is not the full code - just important snippits. If you are interested in reading the full code, it can be found in Final_Project.rmd

