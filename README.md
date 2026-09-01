# Reaction Data Analysis and Material-Efficient Reaction Screening

## Overview

This project explores what can be learned from experimental reaction data without treating reaction yield as the only measure of success.

The analysis uses reaction records from the **Open Reaction Database (ORD)** and focuses mainly on N-arylation and Buchwald-Hartwig type reactions. The goal was to clean and structure the reaction records, understand the distribution of experimental conditions, normalize material usage, and identify reactions that give a good balance between yield and resource consumption.

Rather than building a predictive model, this project focuses on **data analysis and reaction optimization**. The final part of the analysis uses Pareto efficiency and material intensity to identify a smaller set of reactions that are both synthetically effective and comparatively resource-efficient.


## What I wanted to find out

A high reaction yield does not necessarily mean that an experiment is efficient.

For example, two reactions may both give around 80% yield, while one uses considerably more catalyst and solvent than the other. Looking only at yield would make them appear equally successful.

This project therefore looks at several things together:

- Reaction yield
- Reaction temperature
- Reactant, reagent and catalyst quantities
- Solvent volume
- Reaction type
- Normalized material usage
- Estimated product mass
- Material intensity
- Pareto efficiency

The main question became:

> **Which reactions provide a good combination of high yield and relatively low material consumption?**


## Dataset

The analysis was performed on a subset of ORD reaction records containing **750 reactions**.

The reaction records were stored using the ORD schema and protobuf format. The data were converted into a tabular representation for analysis.

### Basic dataset statistics

| Property | Result |
|---|---:|
| Total reactions | 750 |
| Reactions with yield | 750 |
| Reactions with product SMILES | 750 |
| Reactions with temperature | 746 |
| Total input records | 4,489 |
| Inputs with reported moles | 3,748 |
| Input SMILES successfully parsed | 4,489 |
| Reaction classification coverage | 100% |
| Reactions with solvent information | 741 |

The input structures had a **100% SMILES parsing success rate**, which made it possible to perform molecular-level calculations on the recorded compounds.

## Data cleaning and validation

Before doing any analysis, I checked how complete the reaction records were.

The ORD schema contains considerably more information than is usually available in a simple reaction table. Each reaction can contain multiple inputs, reaction roles, amounts, identifiers, products and measurements.

I therefore extracted the relevant information into a pandas DataFrame and checked:

- Missing temperatures
- Missing yields
- Missing product structures
- Missing input information
- Reaction roles
- Reported amounts
- SMILES validity
- Product molecular weights
- Solvent volumes

The reaction inputs were also separated according to their reported roles.

### Reaction roles

The most common roles in the dataset were:

| Role | Records |
|---|---:|
| Reactant | 1,500 |
| Catalyst | 1,498 |
| Reagent | 750 |
| Solvent | 741 |

Other ORD roles can also occur, but these were the main categories encountered in this dataset.


## Reaction classification

The reactions were classified using the reaction type information contained in the ORD records.

There were **9 distinct reaction types** in the dataset.

The largest groups were:

- Chloro Buchwald-Hartwig amination: **290**
- Bromo Buchwald-Hartwig amination: **262**
- Unassigned / unrecognized: **92**
- Iodo Buchwald-Hartwig amination: **76**

The remaining reaction types were much smaller.

This imbalance was taken into account when interpreting the results. A reaction class with only a few examples can be interesting, but its statistics should not be treated as representative of that entire class.


## Yield and temperature

The reported yield had the following distribution:

| Statistic | Yield (%) |
|---|---:|
| Mean | 38.81 |
| Median | 39.93 |
| Standard deviation | 29.40 |
| Minimum | 0.00 |
| Maximum | 102.97 |

There were **6 reactions with reported yields above 100%**.

These values were retained during the exploratory analysis because they are part of the original experimental records, but they should be treated carefully. A reported yield above 100% is generally more likely to reflect experimental or data-recording effects than a physically meaningful yield.

Temperature was available for 746 reactions.

| Statistic | Temperature (°C) |
|---|---:|
| Mean | 107.92 |
| Median | 100 |
| Minimum | 20 |
| Maximum | 200 |

Interestingly, temperature alone was not strongly associated with yield.

The Spearman correlation between temperature and yield was:

**-0.099**

This is a weak relationship, suggesting that simply increasing reaction temperature does not explain the variation in yield across this dataset.


## Normalizing reaction quantities

Raw mass values are difficult to compare between reactions because the scale of an experiment can vary considerably.

For this reason, the quantities were normalized relative to the number of moles of the aryl halide.

The following quantities were calculated:

- Reactant mass per mol of aryl halide
- Reagent mass per mol of aryl halide
- Catalyst mass per mol of aryl halide
- Solvent volume per mol of aryl halide

This made it possible to compare reactions on a more consistent basis.

The median normalized values were approximately:

| Quantity | Median |
|---|---:|
| Reactant mass | 441.07 g/mol aryl |
| Reagent mass | 209.68 g/mol aryl |
| Catalyst mass | 73.58 g/mol aryl |
| Solvent volume | 4.69 L/mol aryl |

The correlations between these normalized quantities and yield were also weak.

For example:

- Reactant mass per mol aryl: **Spearman = 0.12**
- Reagent mass per mol aryl: **Spearman = -0.021**
- Catalyst mass per mol aryl: **Spearman = 0.014**
- Solvent volume per mol aryl: **Spearman = 0.001**

This reinforced the idea that there is no single experimental quantity that can explain reaction performance across the entire dataset.

## Pareto analysis

Instead of ranking reactions using yield alone, I used a Pareto-based approach.

A reaction was considered Pareto-efficient when there was no other reaction that performed better across the selected objectives without being worse in the remaining ones.

This gave:

**116 Pareto-efficient reactions**

which represents approximately:

**16.2% of the analytical dataset**

The Pareto frontier was particularly useful because it avoids forcing all reactions into a single ranking. A reaction can remain interesting because it provides a different trade-off between yield and resource usage.

The fraction of Pareto-efficient reactions also varied considerably between reaction types.

For example:

| Reaction type | Reactions | Pareto-efficient | Percentage |
|---|---:|---:|---:|
| Bromo N-arylation | 15 | 7 | 46.67% |
| Iodo Buchwald-Hartwig | 3 | 1 | 33.33% |
| Chloro N-arylation | 9 | 2 | 22.22% |
| Bromo Buchwald-Hartwig | 242 | 47 | 19.42% |
| Chloro Buchwald-Hartwig | 283 | 34 | 12.01% |
| Iodo Buchwald-Hartwig | 76 | 6 | 7.89% |

The small sample sizes for some reaction types mean these percentages should be interpreted as observations rather than general chemical rules.


## Identifying promising reactions

For a more practical shortlist, I applied a stricter set of conditions.

The reaction had to have:

- Yield above **70%**
- Relatively low normalized solvent usage
- Relatively low normalized catalyst usage

The resulting threshold values were based on the dataset:

```text
Yield threshold:              70%
Median normalized solvent:    4.68 L/mol aryl
Median normalized catalyst:   73.58 g/mol aryl
Median temperature:            100 °C
