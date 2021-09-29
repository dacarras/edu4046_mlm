Métodos Cuantitativos de investigación en Educación
================

# Preparacion de datos

``` r
#------------------------------------------------------------------------------
# prepare data
#------------------------------------------------------------------------------

#--------------------------------------
# load data
#--------------------------------------

library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
data_lux <- readRDS('data_lux.rds')

#--------------------------------------
# centering variables
#--------------------------------------

data_lux <- data_lux %>%
            # socioeconomic status
            mutate(ses_raw = ses_irt) %>%                   # copy of the original variable
            mutate(ses   = r4sda::z_score(ses_irt)) %>%     # variable is standardized
            mutate(ses_c = r4sda::c_mean(ses, id_j)) %>%    # means by cluster
            mutate(ses_g = r4sda::c_mean(ses, ctry)) %>%    # grand mean  
            mutate(ses_m = ses-ses_g)%>%                    # centered to the grand mean
            mutate(ses_b = ses_c-ses_g) %>%                 # school means (cgm)
            mutate(ses_w = ses_m-ses_b)%>%                  # centered within cluster
            mutate(ses_q = ses_w*ses_w) %>%                 # square
            # books at home
            mutate(boo_raw = books) %>%                     # copy of the original variable
            mutate(boo   = r4sda::z_score(books)) %>%       # variable is standardized
            mutate(boo_c = r4sda::c_mean(boo, id_j)) %>%    # means by cluster
            mutate(boo_g = r4sda::c_mean(boo, ctry)) %>%    # grand mean  
            mutate(boo_m = boo-boo_g)%>%                    # centered to the grand mean
            mutate(boo_b = boo_c-boo_g) %>%                 # school means (cgm)
            mutate(boo_w = boo_m-boo_b)%>%                  # centered within cluster
            # take children to library
            mutate(lib_raw = library) %>%                   # copy of the original variable
            mutate(lib   = r4sda::z_score(library)) %>%     # variable is standardized
            mutate(lib_c = r4sda::c_mean(lib, id_j)) %>%    # means by cluster
            mutate(lib_g = r4sda::c_mean(lib, ctry)) %>%    # grand mean  
            mutate(lib_m = lib-lib_g) %>%                   # centered to the grand mean
            mutate(lib_b = lib_c-lib_g) %>%                 # school means (cgm)
            mutate(lib_w = lib_m-lib_b) %>%                 # centered within cluster   
            # read to children
            mutate(rea_raw = read) %>%                      # copy of the original variable
            mutate(rea   = r4sda::z_score(read)) %>%        # variable is standardized
            mutate(rea_c = r4sda::c_mean(rea, id_j)) %>%    # means by cluster
            mutate(rea_g = r4sda::c_mean(rea, ctry)) %>%    # grand mean  
            mutate(rea_m = rea-rea_g) %>%                   # centered to the grand mean
            mutate(rea_b = rea_c-rea_g) %>%                 # school means (cgm)
            mutate(rea_w = rea_m-rea_b)                     # centered within cluster
```

# Modelos de Tabla A1

``` r
#------------------------------------------------------------------------------
# prepare data
#------------------------------------------------------------------------------

# ----------------------------------------------- 
# model equations
# -----------------------------------------------

f00 <- as.formula('score ~ 1 + (1 | id_j)')
f01 <- as.formula('score ~ 1 + ses_w + (ses_w | id_j)')
f02 <- as.formula('score ~ 1 + ses_w + ses_q + (1 | id_j)')
f03 <- as.formula('score ~ 1 + ses_w + boo_w + lib_w + (1 | id_j)')
f04 <- as.formula('score ~ 1 + ses_w + ses_b + (1 | id_j)')

# ----------------------------------------------- 
# fit models
# -----------------------------------------------

m00 <- lme4::lmer(f00, data = data_lux, REML = FALSE)
m01 <- lme4::lmer(f01, data = data_lux, REML = FALSE)
m02 <- lme4::lmer(f02, data = data_lux, REML = FALSE)
m03 <- lme4::lmer(f03, data = data_lux, REML = FALSE)
m04 <- lme4::lmer(f04, data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# display estimates
# -----------------------------------------------

texreg::screenreg(list(
  m00, m01, m02, m03, m04
  ), 
    star.symbol = "*", 
    center = TRUE, 
    doctype = FALSE,
    dcolumn = TRUE, 
    booktabs = TRUE,
    single.row = FALSE
    )
```

# Modelos de SES

``` r
#------------------------------------------------------------------------------
# model building
#------------------------------------------------------------------------------

# ----------------------------------------------- 
# model equations
# -----------------------------------------------

f01 <- as.formula('score ~ 1 + (1 | id_j)')
f02 <- as.formula('score ~ 1 + ses_w + (1| id_j)')
f03 <- as.formula('score ~ 1 + ses_w + ses_b + (1 | id_j)')

# ----------------------------------------------- 
# fit models
# -----------------------------------------------

m01 <- lme4::lmer(f01, data = data_lux, REML = FALSE)
m02 <- lme4::lmer(f02, data = data_lux, REML = FALSE)
m03 <- lme4::lmer(f03, data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# display estimates
# -----------------------------------------------

texreg::screenreg(list(
  m01, m02, m03
  ), 
    star.symbol = "*", 
    center = TRUE, 
    doctype = FALSE,
    dcolumn = TRUE, 
    booktabs = TRUE,
    single.row = FALSE
    )


# ----------------------------------------------- 
# contextual effect
# -----------------------------------------------

library(multcomp)
summary(glht(m03, linfct = c("ses_b - ses_w = 0")))
```

# Modelos posibles entre *y*<sub>*i**j*</sub> y *x*<sub>*i**j*</sub>

``` r
#------------------------------------------------------------------------------
# prepare data
#------------------------------------------------------------------------------

# ----------------------------------------------- 
# Regresion OLS
# -----------------------------------------------

m01 <- lm(score ~ 1 + ses_m, data = data_lux)

# ----------------------------------------------- 
# Regression with clustered errors
# -----------------------------------------------

library(srvyr)
data_lux_svy <- data_lux %>%
                as_survey_design(
                ids = id_j)


m02 <- survey::svyglm(score ~ 1 + ses_m, design = data_lux_svy)

# ----------------------------------------------- 
# ANCOVA model
# -----------------------------------------------

m03 <- lme4::lmer(score ~ 1 + ses_m + (1 | id_j), 
       data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# Mundlak Device
# -----------------------------------------------

m04 <- lme4::lmer(score ~ 1 + ses_m + ses_b + (1 | id_j), 
       data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# Within only Model
# -----------------------------------------------

m05 <- lme4::lmer(score ~ 1 + ses_w + (1 | id_j), 
       data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# Disagregated Model
# -----------------------------------------------

m06 <- lme4::lmer(score ~ 1 + ses_w + ses_b + (1 | id_j), 
       data = data_lux, REML = FALSE)

# ----------------------------------------------- 
# display estimates
# -----------------------------------------------

texreg::screenreg(list(
  m01, m02, m03, m04, m05, m06
  ), 
    star.symbol = "*", 
    center = TRUE, 
    doctype = FALSE,
    dcolumn = TRUE, 
    booktabs = TRUE,
    single.row = FALSE
    )
```

# Italy Example

``` r
#------------------------------------------------------------------------------
# fit models
#------------------------------------------------------------------------------

# ----------------------------------------------- 
# read in data
# -----------------------------------------------

ita_16 <- readRDS('ita_16.rds')

# ----------------------------------------------- 
# model equations
# -----------------------------------------------

f00 <- as.formula('civ ~ 1 + (1 | id_j)')
f01 <- as.formula('civ ~ 1 + opd_m + (1 | id_j)')
f02 <- as.formula('civ ~ 1 + opd_m + opd_b + (1 | id_j)')
f03 <- as.formula('civ ~ 1 + opd_w +  (1 | id_j)')
f04 <- as.formula('civ ~ 1 + opd_w + opd_b + (1 | id_j)')

# ----------------------------------------------- 
# fit models
# -----------------------------------------------

m00 <- lme4::lmer(f00, data = ita_16, REML = FALSE)
m01 <- lme4::lmer(f01, data = ita_16, REML = FALSE)
m02 <- lme4::lmer(f02, data = ita_16, REML = FALSE)
m03 <- lme4::lmer(f03, data = ita_16, REML = FALSE)
m04 <- lme4::lmer(f04, data = ita_16, REML = FALSE)

# ----------------------------------------------- 
# display estimates
# -----------------------------------------------

texreg::screenreg(list(
  m00, m01, m02, m03, m04
  ), 
    star.symbol = "*", 
    center = TRUE, 
    doctype = FALSE,
    dcolumn = TRUE, 
    booktabs = TRUE,
    single.row = FALSE
    )
```

    ========================================================================================
                           Model 1    Model 2       Model 3       Model 4       Model 5     
    ----------------------------------------------------------------------------------------
    (Intercept)               -0.02      -0.02         -0.02         -0.02         -0.02    
                              (0.03)     (0.03)        (0.03)        (0.03)        (0.03)   
    opd_m                                 0.24 ***      0.24 ***                            
                                         (0.02)        (0.02)                               
    opd_b                                               0.01                        0.25 ** 
                                                       (0.08)                      (0.08)   
    opd_w                                                             0.24 ***      0.24 ***
                                                                     (0.02)        (0.02)   
    ----------------------------------------------------------------------------------------
    AIC                     9538.29    9261.07       9263.07       9270.53       9263.07    
    BIC                     9556.73    9285.64       9293.77       9295.09       9293.77    
    Log Likelihood         -4766.15   -4626.54      -4626.53      -4631.26      -4626.53    
    Num. obs.               3450       3434          3434          3434          3434       
    Num. groups: id_j        170        170           170           170           170       
    Var: id_j (Intercept)      0.14       0.13          0.13          0.14          0.13    
    Var: Residual              0.86       0.81          0.81          0.81          0.81    
    ========================================================================================
    Note: *** p < 0.001, ** p < 0.01, * p < 0.05

``` r
#------------------------------------------------------------------------------
# fit models
#------------------------------------------------------------------------------

# ----------------------------------------------- 
# read in data
# -----------------------------------------------

ita_16 <- readRDS('ita_16.rds')

# ----------------------------------------------- 
# model equations
# -----------------------------------------------

f01 <- as.formula('civ ~ 1 + opd_m + opd_b + (1 | id_j)')
f02 <- as.formula('civ ~ 1 + opd_w + opd_b + (1 | id_j)')

# ----------------------------------------------- 
# fit models
# -----------------------------------------------

m01 <- lme4::lmer(f01, data = ita_16, REML = FALSE)
m02 <- lme4::lmer(f02, data = ita_16, REML = FALSE)

# ----------------------------------------------- 
# display estimates
# -----------------------------------------------

texreg::screenreg(list(
  m01, m02
  ), 
    star.symbol = "*", 
    center = TRUE, 
    doctype = FALSE,
    dcolumn = TRUE, 
    booktabs = TRUE,
    single.row = FALSE
    )
```

    ==================================================
                            Model 1       Model 2     
    --------------------------------------------------
    b_0                        -0.02         -0.02    
                               (0.03)        (0.03)   
    b_1  (opd_ij - opd_..)      0.24 ***              
                               (0.02)                 
    b_1  (opd_ij - opd_.j)                    0.24 ***
                                             (0.02)   
    b_2  (opd_.j - opd_..)      0.01          0.25 ** 
                               (0.08)        (0.08)   
    --------------------------------------------------
    AIC                      9263.07       9263.07    
    BIC                      9293.77       9293.77    
    Log Likelihood          -4626.53      -4626.53    
    Num. obs.                3434          3434       
    Num. groups: id_j         170           170       
    Var: id_j (Intercept)       0.13          0.13    
    Var: Residual               0.81          0.81    
    ==================================================
    Note: *** p < 0.001, ** p < 0.01, * p < 0.05
