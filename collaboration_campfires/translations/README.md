
# Evaluating the status of translations in R

The script `translations_status.R` produces datasets regarding message
translations from a local clone of <https://github.com/r-devel/r-svn>, a
mirror of the SVN repository containing the source code for R. The
script is designed to be run interactively and requires the file path of
the `r-svn` repo and an output directory to be specified. The datasets
are saved in the output directory with the names `metadata.csv` and
`message_status.csv`.

## Metadata CSV

A CSV with one record per PO file in the R sources. Includes the
variables:

-   `sha`: the shortened SHA for the git commit of the r-svn repo that
    the data were obtained from
-   `date`: the date of the git commit of the r-svn repo that the data
    were obtained from
-   `package`: the name of a package containing messages to be
    translated
-   `po_file`: the name of `.po` files in the package sources
-   `component`: the “component” of the package the PO file relates to,
    either “C”, “R”, “RGui” (the latter is only in the base package)
-   `language`: the name of the language in English with the region as a
    suffix if applicable (e.g. “English_GB” vs “English”)
-   `r_version`: the name of the R version to PO file relates do (does
    not always match `pot_creation_date`)
-   `bug_reports`: where to report bugs related to this PO file
-   `pot_creation_date`: the date the PO template file was created (when
    messages last updated), YYYY-MM-DD format
-   `po_revision_date`: the date the PO file was revised, YYYY-MM-DD
    format
-   `last_translator`: the name and email of the last translator
-   `team`: the name and/or email of the translation team

## Message status CSV

A CSV with one record per message in each PO file in the R source.
Includes the variables `sha`, `date`, `package`, `po_file`, `component`,
and `language` as above, plus

-   `message`: a message in the PO file
-   `translated`: a logical value indicating if the message has been
    translated
-   `fuzzy`: a logical value indicating if the translation is flagged as
    “fuzzy”, i.e. a fuzzy match of an old translation to a message that
    has had a minor update

## Example plot

``` r
library(dplyr)
library(forcats)
library(ggplot2)
library(readr)
```

``` r
message_status <- read_csv("message_status.csv")
```

Plot the counts of correctly translated messages from the C or R code

``` r
ggplot(filter(message_status, translated, !fuzzy, component != "RGui"),
       aes(x = fct_infreq(language))) +
    geom_bar(stat = "count", fill = "steelblue") +
    theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1),
          legend.position = "none") +
    labs(x = NULL, y = "# translated messages",
         subtitle = "Correctly translated messages in base and default packages")
```

![](README_files/figure-gfm/message_status_plot-1.png)<!-- --> There is
one message translation in standard English - this is to add information
about the locale to the startup message in R.

## Alternative script and data

The `get_r_translation_status.R` script is an alternative approach to
obtaining data on translation status in R. Unlike the above, it
considers all commits since translations were introduced. For each
commit, `get_r_translation_status_for_revision.R` is called to compute
the number of of translated messages.

Only non-fuzzy translations are counted and translations for the RGui
are ignored. The messages are extracted with
`potools:::get_po_messages`.

The file `4168b6fff27eafad68a4b134dba5c7d09e090fcb.csv` contains the
results for the r-svn commit with hash
4168b6fff27eafad68a4b134dba5c7d09e090fcb. This CSV has columns

-   `git_commit`: the commit hash
-   `package`: the name of a package in the R sources
-   `language`: the ISO 639 code of the language, including variant
-   `type`: either “C” or “R”
-   `n_translated`: number of correctly translated messages
-   `n_untranslated`: number of oncorrectly translated messages

Currently, the code over-counts the number translated by 1, since it
counts the empty message at the start of the PO files.

The code to convert the ISO 639 codes to names also mistakenly converts
`nn` to Nederlands (Dutch) vs Nynorsk (Norwegian).
