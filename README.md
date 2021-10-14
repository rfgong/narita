# IVaps

This repository supports the Approximate Propensity Score (APS) instrumental variables approach introduced in "Algorithm is Experiment" (Narita and Yata, forthcoming). In this empirical context, treatment recommendations are made by some known algorithm. 

As stated in the paper's introduction: for each covariate value x, the Approximate Propensity Score is the average probability of a treatment recommendation in a shrinking neighborhood around x. Treatment effects can be estimated by two-stage least squares (2SLS) where
we regress the outcome on the treatment with the algorithm’s recommendation as an IV and APS as a control.

D<sub>i</sub> = &gamma;<sub>0</sub> + &gamma;<sub>1</sub> Z<sub>i</sub> + &gamma;<sub>2</sub> APS<sub>i</sub> + &nu;<sub>i</sub> (First Stage)

Y<sub>i</sub> = &beta;<sub>0</sub> + &beta;<sub>1</sub> D<sub>i</sub> + &beta;<sub>2</sub> APS<sub>i</sub> + &epsilon;<sub>i</sub> (Second Stage)

Y<sub>i</sub> is the outcome of interest, D<sub>i</sub> is the binary treatment assignment (possibly made by humans), and Z<sub>i</sub> is the binary treatment recommendation made by a known algorithm.

Covariate balance of predetermined characteristics across treatment groups, conditional on APS, can further establish the validity of estimated treatment effects.

W<sub>i</sub> = &gamma;<sub>0</sub> + &gamma;<sub>1</sub> Z<sub>i</sub> + &gamma;<sub>2</sub> APS<sub>i</sub> + &eta;<sub>i</sub>

W<sub>i</sub> is a predetermined characteristic.

**Python, R and Stata implementations are available.**

# Example

This section demonstrates code functionality in a simple policy context. Subjects that satisfy a cutoff rule using three variables are treated.

See the paper's "Empirical Policy Application" for a discussion of the example results.

### Python Example

```python
from ivaps import *

df = pd.read_stata("data/safety_net_elig.dta")

# Your Treatment Assignment Rule
def predict(X):
    return ((X[:,0] >= 0.202) & (X[:,1] >= 25000) & (X[:,2] <= 0.03)).astype(int)

# Estimate APS
df["aps"] = estimate_aps(predict = predict, X = df[["sum_pctg_ssi_mdcd_days", "ucc_per_bed", "profit_margin"]], C = [0,1,2], S = 10000, delta = 0.05, nprocesses = 2)

# Instrumental Variables
result = estimate_treatment_effect(aps = df.aps, Y = df.tot_con_sus2020_07_31, Z = df.safety_net, D = df.safety_dollars_adj)

# Covariate Balance
result = covariate_balance_test(aps = df.aps, X = df[["occupancy","beds"]], Z = df.safety_net)
```

Download **ivaps.html** for detailed method documentation.

### R Example

```R
library(haven)
source("ivaps.R")

df <- read_dta("data/safety_net_elig.dta")

# Your Treatment Assignment Rule
predict <- function(X) {
  return(as.integer((X[, 1] >= 0.202) & (X[, 2] >= 25000) & (X[,3] <= 0.03)))
}

# Estimate APS
df$aps <- estimate_aps(predict = predict, X = df[c("sum_pctg_ssi_mdcd_days", "ucc_per_bed", "profit_margin")], C = c(1,2,3), S = 10000, delta = 0.05, nprocesses = 2)

# Instrumental Variables
result <- estimate_treatment_effect(data = df, aps = "aps", Y = "tot_con_sus2020_07_31", Z = "safety_net", D = "safety_dollars_adj")

# Covariate Balance
result <- covariate_balance_test(data = df, aps = "aps", X = c("occupancy", "beds"), Z = "safety_net")
```

### Stata Example

See **stata** folder.

#### References
Yusuke Narita, Kohei Yata.
<b>Algorithm is Experiment: Machine Learning, Market Design, and Policy Eligibility Rules</b>, 2021.
[<a href="https://arxiv.org/abs/2104.12909">arxiv</a>]
