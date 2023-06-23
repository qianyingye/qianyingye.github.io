---
title: 'R｜convert rtf to csv'
date: 2023-06-23
permalink: /posts/2012/08/blog-post-4/
tags:
  - R
  - rtf
  - csv
---


```R
library(readtext)
library(tidyverse)
write_rtf_data_to_csv <- function(input_folder, output_folder, output_file) {
  # Get the list of RTF files in the input folder
  rtf_files <- list.files(path = input_folder, pattern = "\\.rtf$", full.names = TRUE)
  
  # Initialize an empty data frame to store the contents of all files
  all_content <- data.frame(content = character(), stringsAsFactors = FALSE)
  
  # Loop over the RTF files and read in the contents
  for (i in seq_along(rtf_files)) {
    file_content <- readtext(rtf_files[i])
    content <- strsplit(file_content[[2]], split = "\n\n\n\n\n\n \n")
    df_content <- data.frame(content, stringsAsFactors = FALSE)
    colnames(df_content) <- c("content")
    all_content <- rbind(all_content, df_content)
  }
  
  # Write the combined data frame to a CSV file
  write.csv(all_content, file.path(output_folder, output_file), row.names = FALSE)
}

# Example usage:
write_rtf_data_to_csv(input_folder, output_folder, output_file)
```
