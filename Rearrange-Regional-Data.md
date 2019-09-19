-   [Motivation](#motivation)
-   [The Correspondence Table](#the-correspondence-table)
    -   [NUTS1 Correspondence](#nuts1-correspondence)
    -   [NUTS2 Correspondence](#nuts2-correspondence)
    -   [NUTS3 Correspondence](#nuts3-correspondence)
-   [Problems from inconsistent NUTS level
    data](#problems-from-inconsistent-nuts-level-data)
    -   [Inconsistent levels](#inconsistent-levels)
    -   [Small country problems](#small-country-problems)
-   [Problems of Mislabelling](#problems-of-mislabelling)
    -   [Recoded regions](#recoded-regions)
    -   [Discountinued regions](#discountinued-regions)
    -   [Changed regions](#changed-regions)
-   [Special territorial units](#special-territorial-units)
    -   [Extraterritorial units](#extraterritorial-units)
    -   [Non-EU member states](#non-eu-member-states)
-   [Conclusions](#conclusions)
    -   [Simple filtering and error
        handling](#simple-filtering-and-error-handling)
    -   [Warnings](#warnings)
    -   [Consultation with Eurostat](#consultation-with-eurostat)

Motivation
----------

My supposedly reproducible research codes do not work from last year,
when the same products with the same IDs were reported under the
`NUTS2013` region boundary definitions. After spending countless days
with fixing my datasets, I learned a lot about the problematic
transition from `NUTS2013` to `NUTS2016` data products. I’d like to
create a package vignette, and possibly some helper functions to save
many hours of work for users of regional datasets.

The Correspondence Table
------------------------

The Correspondence table is supposed to explain how NUTS regions have
changed and give guidance on what to do with the data. This is a
multi-sheet Excel file that is not tidy, and it is not complete. At the
time of writing this article, the changes in Slovenia and Greece are not
in the table.

    #Download the latest Eurostat correspondence file.
    #Change the subdirectory as you wish, I download it to 'data-raw'.
    if(! dir.exists('data-raw')) dir.create('data-raw')
    if(! file.exists(file.path('data-raw' ,'NUTS2013-NUTS2016.xlsx'))) {
      download.file ( url = 'https://ec.europa.eu/eurostat/documents/345175/629341/NUTS2013-NUTS2016.xlsx', destfile = file.path('data-raw', 'NUTS2013-NUTS2016.xlsx' ))
    }

    #The Excel file downloaded from Eurostat may or may not work on your system. If you are unlucky, you can try my copy. 
    if(! file.exists(file.path('data-raw' ,'NUTS2013-NUTS2016.xlsx'))) {
      download.file ( url = 'https://github.com/antaldaniel/eurostat_regional/raw/master/data-raw/NUTS2013-NUTS2016.xlsx', destfile = file.path('data-raw', 'NUTS2013-NUTS2016.xlsx' ))
    }

    regions <- readxl::read_excel( file.path('data-raw', 'NUTS2013-NUTS2016.xlsx'),
                       sheet = 'NUTS2013-NUTS2016', 
                       skip = 1 , col_names = T) %>%
      select (1:12) %>%  #Extra unusued columns that will give a warning
      purrr::set_names(., c("rowid", "code13", "code16", 
                            "country_name", 'nuts1_name', 'nuts2_name',
                            'nuts3_name', 'change', 'nuts_level', 
                            'sort_countries', 'sort_13', 'sort_16')) %>%
      mutate ( name = case_when ( 
        !is.na(nuts1_name) ~ nuts1_name, 
        !is.na(nuts2_name) ~ nuts2_name,
        !is.na(nuts3_name) ~ nuts3_name, 
        !is.na(country_name)~ country_name,
        TRUE ~ NA_character_))

    ## New names:
    ## * `` -> ...1
    ## * `` -> ...13
    ## * `` -> ...14
    ## * `` -> ...15

    #Our reference vector are the NUTS2016 codes
    nuts_2016_codes <- unique (regions$code16)

It is a good idea to have a comprehensive look of what happened, because
each data set may have a different problem. I organized the regions into
`unchanged_regions`, `changed_regions` and `discontinued_regions`. These
tables will help in correcting the data. Unfortunately, I found some
regions in Slovenia and Greece that are not in any of these categories.

    ##In these cases, the code13 == code16
    unchanged_regions <- regions %>% filter ( is.na(change)) %>%
      fill ( nuts1_name ) %>%
      fill ( nuts2_name ) %>%
      select ( code13, code16, name )

    ## In these cases code13 != code16
    changed_regions <- regions %>% filter ( !is.na(change)) %>%
      fill ( nuts1_name ) %>%
      fill ( nuts2_name ) %>%
      select ( code13, code16, name, nuts_level, change )

    discontinued_regions  <- changed_regions %>%
      filter ( change == "discontinued")

### NUTS1 Correspondence

    nuts1_correspondence <- readxl::read_excel( file.path('data-raw', 'NUTS2013-NUTS2016.xlsx'),
     sheet = 'Correspondence NUTS-1', 
     skip = 0 , col_names = T) %>%
      set_names ( ., c("code13", "code16", "nuts1_name", "change", "resolution")) %>%
      mutate_if ( is.factor, as.character )

    knitr::kable (head(nuts1_correspondence,6))

<table>
<thead>
<tr class="header">
<th style="text-align: left;">code13</th>
<th style="text-align: left;">code16</th>
<th style="text-align: left;">nuts1_name</th>
<th style="text-align: left;">change</th>
<th style="text-align: left;">resolution</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRB</td>
<td style="text-align: left;">CENTRE — VAL DE LOIRE</td>
<td style="text-align: left;">new NUTS 1 region, identical to ex-NUTS 2 region FR24</td>
<td style="text-align: left;">FRB=FR24</td>
</tr>
<tr class="even">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRC</td>
<td style="text-align: left;">BOURGOGNE-FRANCHE-COMTÉ</td>
<td style="text-align: left;">new NUTS 1 region, merge of ex-NUTS 2 regions FR26 and FR43</td>
<td style="text-align: left;">FRC=FR26+FR43</td>
</tr>
<tr class="odd">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRD</td>
<td style="text-align: left;">NORMANDIE</td>
<td style="text-align: left;">new NUTS 1 region, merge of ex-NUTS 2 regions FR23 and FR25</td>
<td style="text-align: left;">FRD=FR23+FR25</td>
</tr>
<tr class="even">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRE</td>
<td style="text-align: left;">NORD-PAS DE CALAIS-PICARDIE</td>
<td style="text-align: left;">new NUTS 1 region, merge of ex-NUTS 2 regions FR22 and FR30</td>
<td style="text-align: left;">FRE=FR22+FR30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRF</td>
<td style="text-align: left;">ALSACE-CHAMPAGNE-ARDENNE-LORRAINE</td>
<td style="text-align: left;">new NUTS 1 region, merge of ex-NUTS 2 regions FR 21, FR 41 and FR42</td>
<td style="text-align: left;">FRF=FR21+FR41+FR42</td>
</tr>
<tr class="even">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">FRG</td>
<td style="text-align: left;">PAYS DE LA LOIRE</td>
<td style="text-align: left;">new NUTS 1 region, identical to ex-NUTS 2 region FR51</td>
<td style="text-align: left;">FRG=FR51</td>
</tr>
</tbody>
</table>

### NUTS2 Correspondence

Similarly, there is a table for NUTS2 level. You have to beware that all
NUTS2 regions are contained in wider NUTS1 regions. So there are two
logically possible cases: a NUTS2 change affected only one NUTS1 region
(changes were made within a larger region), or it affected several NUTS1
regions. If you work with data that has both NUTS1 and NUTS2 level
information, such as any of the internet useage products, for example,
`isoc_r_iuse_i`, you must correct NUTS1 and NUTS2 level problems
consistently.

    nuts2_correspondence <- readxl::read_excel( 
      file.path('data-raw', 'NUTS2013-NUTS2016.xlsx'),                                            sheet = 'Correspondence NUTS-2',                                            skip = 0 , col_names = TRUE) %>%
      select ( 1:5 ) %>%
      set_names ( ., c("code13", "code16", "nuts1_name",
                       "change", "resolution")) %>%
      filter ( is.na(code13) + is.na(code16) < 2) #The table has empty rows

    ## New names:
    ## * `` -> ...6
    ## * `` -> ...7
    ## * `` -> ...8

    knitr::kable (head(nuts2_correspondence,6))

<table>
<thead>
<tr class="header">
<th style="text-align: left;">code13</th>
<th style="text-align: left;">code16</th>
<th style="text-align: left;">nuts1_name</th>
<th style="text-align: left;">change</th>
<th style="text-align: left;">resolution</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">IE01</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">Border, Midland and Western</td>
<td style="text-align: left;">discontinued</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">IE04</td>
<td style="text-align: left;">Northern and Western</td>
<td style="text-align: left;">new region</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="odd">
<td style="text-align: left;">IE02</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">Southern and Eastern</td>
<td style="text-align: left;">discontinued</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">IE05</td>
<td style="text-align: left;">Southern</td>
<td style="text-align: left;">new region, made from ex-IE023, IE024 and IE025</td>
<td style="text-align: left;">IE05=IE023+IE024+IE025</td>
</tr>
<tr class="odd">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">IE06</td>
<td style="text-align: left;">Eastern and Midland</td>
<td style="text-align: left;">new region</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">FR24</td>
<td style="text-align: left;">FRB0</td>
<td style="text-align: left;">Centre — Val de Loire</td>
<td style="text-align: left;">recoded and relabelled</td>
<td style="text-align: left;">FRB0=FR24</td>
</tr>
</tbody>
</table>

### NUTS3 Correspondence

The NUTS3 regions are small regions within the NUTS2 regions. There are
only a few NUTS3 level statistical products available, and in the
absence of an even lower resolution, most changes at this level cannot
be reconciled.

However, many corrections on NUTS1 or NUTS2 level require reconciliation
with the use of NUTS3 data, which, sadly, contains a lot of relabelling
and other changes. So if you really want to fix everything, you have to
dwell into the NUTS3 level, too.

    nuts3_correspondence <- readxl::read_excel( 
      file.path('data-raw', 'NUTS2013-NUTS2016.xlsx'),                                            sheet = 'Correspondence NUTS-3',                                            skip = 0 , col_names = TRUE) %>%
      select ( 1:5 ) %>%
      set_names ( ., c("code13", "code16", "nuts1_name",
                       "change", "resolution")) %>%
      filter ( is.na(code13) + is.na(code16) < 2) #The table has empty rows

    knitr::kable (head(nuts3_correspondence,6))

<table>
<thead>
<tr class="header">
<th style="text-align: left;">code13</th>
<th style="text-align: left;">code16</th>
<th style="text-align: left;">nuts1_name</th>
<th style="text-align: left;">change</th>
<th style="text-align: left;">resolution</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">DE915</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">Göttingen</td>
<td style="text-align: left;">discontinued; merged with ex-DE919</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">DE919</td>
<td style="text-align: left;">NA</td>
<td style="text-align: left;">Osterode am Harz</td>
<td style="text-align: left;">discontinued; merged with ex-DE915</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="odd">
<td style="text-align: left;">NA</td>
<td style="text-align: left;">DE91C</td>
<td style="text-align: left;">Göttingen</td>
<td style="text-align: left;">new region, merge of ex-DE915 and DE919</td>
<td style="text-align: left;">DE91C=DE915+DE919</td>
</tr>
<tr class="even">
<td style="text-align: left;">DEB16</td>
<td style="text-align: left;">DEB1C</td>
<td style="text-align: left;">Cochem-Zell</td>
<td style="text-align: left;">boundary shift</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DEB19</td>
<td style="text-align: left;">DEB1D</td>
<td style="text-align: left;">Rhein-Hunsrück-Kreis</td>
<td style="text-align: left;">boundary shift</td>
<td style="text-align: left;">NA</td>
</tr>
<tr class="even">
<td style="text-align: left;">IE011</td>
<td style="text-align: left;">IE041</td>
<td style="text-align: left;">Border</td>
<td style="text-align: left;">boundary shift</td>
<td style="text-align: left;">NA</td>
</tr>
</tbody>
</table>

Problems from inconsistent NUTS level data
------------------------------------------

### Inconsistent levels

In many cases, the name and/or the metadata description of the
statistical product refers to NUTS2 level data, but in fact, you receive
a mixed data table that has data from all NUTS levels.

This may create some extra filtering work, especially if you want to be
reproducible, tidy, or just want to check consistencies, but this can be
a fortunate arrangement that Eurostat should keep and expand in the
future with better metadata. In this case, usually you have a lot of
information present to correct the some problems.

For example, in
[isoc\_r\_iuse\_i](https://appsso.eurostat.ec.europa.eu/nui/show.do?dataset=isoc_r_iuse_i&lang=en),
you find the following geo units that do not conform the current,
NUTS2016 definion, and you cannot correct them on the basis of the
correspondence table, because it does not contain in the current version
information about Slovenia and Greece: SI01,SI02,EL1,EL2.

However, the following regions can be corrected: FRB, FRC, FRD, FRE,
FRF, FRG, FRH, FRI, FRJ, FRK, FRL, FRM, FRY, PL2, PL4, PL5, PL6, PL7,
PL8, PL9, IE04, IE05, IE06, FRB0, FRC1, FRC2, FRD1, FRD2, FRE1, FRE2,
FRF1, FRF2, FRF3, FRG0, FRH0, FRI1, FRI2, FRI3, FRJ1, FRJ2, FRK1, FRK2,
FRL0, FRM0, FRY1, FRY2, FRY3, FRY4, LT01, LT02, HU11, HU12.

In this case, with care you can add 4 NUTS1 regions to your dataset.

The usual problem with information society indicators is that they do
not contain NUTS2 level information for larger countries. In order to
avoid unnecessary data loss, Eurostat (correctly) decided to report
NUTS1 level data for large countries and NUTS2 level data for smaller
countries. As far as I know the reason is that the microdata are surveys
with fixed 1000-1500 national sample size, which would allow a regional
breakup in the case of Estonia on NUTS3 level but only at NUTS1 level in
Great Britain and Germany. The way the samples are designed that they
must be representative for Northern Ireland, which is a NUTS2 region,
and therefore in the case of the UK you have inconsistent NUTS levels
within a single country.

While the best representation of the data is what Eurostat publishes, a
lot of warnings would be required here in the title of the product and
the metadata. If you start to join this nominally NUTS2 datasets with
other NUTS2 data, you will immediately loose the large member states!
This is actually a very strong case to make all regional data products
cross-level, i.e. whenever a data is avaiable on NUTS2 level, it should
be reported in the same product on NUTS1 and NUTS0 levels, too. \[^I
will come back to this problem later.\]

### Small country problems

Malta and Luxembourg are much smaller than the provinces of Germany, or
the British home countries Northern Ireland or Wales. Statistical
regions were designed to create more homogeneous territorial units in
size and other characteristics. This means that countries smaller than a
usual NUTS1 region in larger countries do not have a NUTS1 level
division. Very small countries do not have a NUTS2 division. Cyprus,
Estonia, Luxembourg, Malta have only NUTS3 divisions.

So, the rule of NUTS regions is NUTS0 &gt;= NUTS1 &gt;= NUTS2 &gt;
NUTS3, where the NUTS0 level stands for the largest territorial
division, which is national boundary.

In this case, `LU` = `LU0` = `LU00`. If you want to have a complete map
of NUTS2 regions of Europe, obviously you have to treat Luxembourg and
the other small member states as NUTS2 regions. Provided that you have
the data!

There is an inconsistency how the data is present in the datasets, but
you can almost always find the `LU` code. If `LU0` is missing, you can
just create a copy of the `LU` data and label it `LU0` on NUTS1 level,
and repeat the same as `LU00` on NUTS2 level. It also follows that NUTS0
codes are always made of 2 characters, NUTS1 is always two alphabetic
characters followed by a number, and NUTS2 is always two alphabetic
followed by 2 numbers.

So far I have shown problems that can be solved with simple filtering or
relabelling if you understand the hiearchy of NUTS levels. A far more
serious problem when NUTS2013 and NUTS2016 data got mixed without proper
metadata.

Problems of Mislabelling
------------------------

A typical mislabelling problem is that in the same data/metadata column
`geo` you find mixed region codes, following seemingly randomly
`NUTS2013` and `NUTS2016` codes. This is a real error, and it requires
more than data tiding to fix these problems, up to the level they can be
fixed.

### Recoded regions

The simplest error is a recoding error. You will most likely run into
this problem with France, a country that comprehensively changes its
NUTS units. To avoid confusion, the French altered the alphanumeric
codes completely, and replaced the letter:letter:number structure of the
NUTS codes to letter:letter:letter, like instead of using an `FR7` like
nomenclature, they use a hexadecimal-like `FRK`. This is a wise move but
it is not followed up by the Eurostat data.

In many cases, the recoding means that the region did not change, but
the abbreviated code did. For example, the change is merely `FRK1=FR72`.
You have to look up all instances of `FR72` labels in your data, and if
they are present, change them to `FRK1`.

In some cases, the name of the regionn changed (should you use them in
your table, beware), but the region itself is the same. For example, the
name of `MAKROREGION PÓŁNOCNO-ZACHODNI` was changed, but its code, `PL4`
and its boundaries remained the same. Because of many possible character
encoding problems, I do not recommend the use of names in statistical
analysis. If you need them in visualizations, add them back in the last,
visualiztion stage.

For example, the
[tgs00096](https://ec.europa.eu/eurostat/web/products-datasets/product?code=tgs00096)
has consistently wrong labels for all French NUTS2 regions. The data in
this product (Population on 1 January by NUTS 2 region) goes back to
pre-NUTS2013 definitions, so this is not purely the same problem. The
earliest data in this product is from 2007, so four NUTS definition
changes may affect them. (See the [history of
NUTS](https://ec.europa.eu/eurostat/web/nuts/history) on the Eurostat
website.)

### Discountinued regions

Many regions are discontinued, so you may have data from the `NUTS2013`
definition that you will not use in a `NUTS2016` map or analysis. Do not
discard these data points yet, because you may be able to use them to
re-create some missing `NUTS2016` observations.

### Changed regions

In the correspondence files, you find hints how to solve missing case
problems. For example, if `IE05` is missing from your NUTS2 table, you
can try to impute it with the equation `IE05=IE023+IE024+IE025`. In this
particular case, because `IE023` is a NUTS3 small regions’s code, you
have to go down to that particular level to impute the missing NUTS2
case. If you are lucky, the NUTS3 data is in the dataset (incorrectly
labelled as NUTS2), or it is available from a NUTS3-level product.

There are a few extra caveats here:

-   You can technically do the imputation if the change does not mean a
    change of NUTS3 boundaries, only a rearrangement of constituent
    lower level boundaries. In this case, *the changes are additive*,
    and you can impute the data *provided the data itself is additive*.
    You can add together population numbers or euro figures.

-   You cannot get the missing data with addition and subtraction in the
    case the data is not atomic, and therefore it is not additive. You
    cannot add or subtracted population density or euro per capita
    figures. In this case, you can only do the imputation if you
    additively come up with euro and population figures for
    `IE023+IE024+IE025`, and divide the result.

-   If the change is not a re-arrangement of lower level, lower
    resolution data, than you cannot come to any precise formula to fill
    in the missing observation.

-   You will not be able to fill the missing observation when any of the
    constituent data is unavailable. You will not be able to get the
    `IE05` data if any of `IE023` or `IE024` or `IE025` is missing.

Special territorial units
-------------------------

### Extraterritorial units

Some NUTS codes end with `Z`, such as `ITZ`, `ITZZ` and `ITZZZ`, in this
case, extra-regional data for Italy. You can treat this as a kind of a
correction raw, and it usually, or probably always empty in the case of
regional statistics. It is used for data that cannot be connected to a
region.

However, the fact that they are always `NA` values can cause problems
with imputation algorithms, so the best practice is to filter these
empty rows out.

### Non-EU member states

It is a welcome development that more and more regional statistics
include EEA members, candidate or potential candidate countries.
Sometimes you can find regional data about Albania, Andora,
Bosnia-Herzegovina, Iceland, Kosovo, Lichtenstein, Montenegro, North
Macedonia, Serbia, Switzerland, Turkey and Norway, and probably further
states, too.

If the coding is correct, you can easily put the data on a map, too,
because the maps that are used by the `eurostat` package contain the
polygons of these regions. However, the NUTS correspondence table do not
include them, so there is no recourse to check their consistency. They
sometimes pop up, sometimes not, so make sure if you have too many
observations, check for them.

Conclusions
-----------

While I did write a comprehensive code to these problems, it has so many
excemptions that I do not suggest do include it in the `eurostat`
package, because it would require a lot of work to test all possible
errors and to maintain the quality of the code as further changes may
come in the future.

### Simple filtering and error handling

I think that the simple cases can and should be encoded in the eurostat
package. For example, a very simple function could create two additional
columns from the `geo` variable, like I did, a `code13` and a `code16`
one, which contains the correct code for `NUTS2013` for backward
compatiblity with pre-exsiting maps and reseaerch, and the current
`NUTS2016` definition. The `eurostat` package should also allow the use
of `NUTS2013` and `NUTS2016` maps.

I also believe that a very simple fix could be done to create, whenever
they are missing, the obviously equivalent rows. For example, if there
is a `LU` or `LU0` data present, it should be mutated into a `LU00` row,
too. This is not prone to error, and allows the use of `NUTS0`, `NUTS1`
and `NUTS2` maps joining datasets.

### Warnings

In the case of `discontinued` and `changed` `geo` definitions, I think
that the `get_eurostat` function should give a warning, someling like
“Beware, the downloaded data contain geo codes that follow the earlier
national and regional coding nomenclature. Read our vignette.”

### Consultation with Eurostat

There is a statistical regulation on how the NUTS change have to be
carried out in statistics, and I am sure that Eurostat more or less
complies with them, but it does not mean that it complies with the best
possible way. The current solution is extremely error-prone and it
should be recommended to Eurostat to change it.

-   The inconsistent label levels and label codes make the joining of
    different statistical tables almost impossible. Especially French,
    German, British and Polish data, furthermore the small country data
    is almost always lost in joining operations. It would be far better
    to provide table with two coding columns, like the solution above
    with `geo_code13` and `geo_code16`, or with the addition of an
    explicit auxillary column saying that the actual row is defined as
    `NUTS2013` or `NUTS2016`. This is not unambigous, because in some
    cases the codes changed, and in other they did not. A `PL2` code can
    be both `NUTS2013` or `NUTS2016`.

-   The mixed presence of `NUTS2013` and `NUTS2016` data makes panel
    data arrangement next to impossible. These should be separate
    statistical products with separate name and metadata explanations.

-   Some products go back as far as the `NUTS2003` definitions, and it
    should be clear, and if possible, consistent, how transition from
    `NUTS2013` to `NUTS2006`, then to `NUTS2010`, then to `NUTS2013` and
    eventually `NUTS2016` happened.

-   The introduction of mixed level statistical products, like
    [isoc\_r\_iuse\_i](https://appsso.eurostat.ec.europa.eu/nui/show.do?dataset=isoc_r_iuse_i&lang=en),
    is welcome, because it allows users to follow the correspondence
    tables, but it should be done with correct descriptions and in a
    consistent way.

-   It is a bad practice that whenever NUTS or NACE or COFOG or other
    statistical definitions change, Eurostat withdraws statistical
    products. These products should be made available forever in an
    archive section, with warnings that they are not subject to further
    revisions and improvements, and they use out-of-date statistical
    definitions. Such datasets may be used in many academic and
    non-academic populations, which cannot be reviewed and fact checked
    if the data disappears. With the increasing use of reproducible
    research techniques in academia and among professional users,
    disappearing datasets, or datasets that are not renamed but changed
    in essence cause extraordinary amont of debugging work.
