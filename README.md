
<!-- README.md is generated from README.Rmd. Please edit that file -->

# SOAE-HPAI

## Overview

``` r
if (!nzchar(Sys.getenv("HPAI_DATA_URL"))) {
    stop("the data URL needs to be provided as an environment variable")
}

library(ggplot2)
library(dplyr)
requireNamespace("readr")

## from https://github.com/thesnakeguy/SOAE_GlobalFishingWatch/blob/master/southern_ocean_fishing.qmd
theme_antarctic <- function() {
  theme_minimal(base_size = 12) +
    theme(
      plot.title       = element_text(face = "bold", size = 14),
      plot.subtitle    = element_text(colour = "grey40", size = 11),
      plot.caption     = element_text(colour = "grey55", size = 8,
                                      hjust = 0, margin = margin(t = 8)),
      legend.position  = "right",
      panel.grid.minor = element_blank(),
      panel.grid.major = element_line(colour = "grey90", linewidth = 0.3),
      strip.text       = element_text(face = "bold", size = 11)
    )
}

## place the ggplot legend inside the plot bounds
internal_legend <- function(x, y) {
    theme(strip.text.x = element_blank(),
          strip.background = element_rect(colour = "white", fill = "white"),
          legend.position = c(x, y))
}
```

``` r
## this can take more than one try sometimes
for (tries in 1:3) {
    x <- tryCatch({
        readr::read_csv(Sys.getenv("HPAI_DATA_URL"), show_col_types = FALSE)
    }, error = function(e) NULL)
    if (!is.null(x)) break
    Sys.sleep(2)
}

if (is.null(x)) stop("could not read data")

names(x) <- tolower(gsub("[[:space:][:punct:]]+", "_", names(x)))
x$status[is.na(x$status)] <- "Unknown"

x <- x %>% mutate(cumulative_cases = cumsum(.data$status == "Confirmed"),
                  cumulative_suspected = cumsum(.data$status %in% c("Confirmed", "Suspected")))
```

``` r
px <- bind_rows(x %>% dplyr::select("date_reported", N = "cumulative_cases") %>%
                  mutate(type = "Confirmed"),
                x %>% dplyr::select("date_reported", N = "cumulative_suspected") %>%
                  mutate(type = "Confirmed and\nsuspected"))

ggplot(px, aes(.data$date_reported, .data$N, colour = .data$type, group = .data$type)) +
    geom_path() +
    theme_antarctic() + labs(x = "Date", y = "Cumulative cases", colour = NULL) +
    internal_legend(0.2, 0.9)
```

<img src="README_files/figure-gfm/plot-1.png" alt="" style="display: block; margin: auto;" />
