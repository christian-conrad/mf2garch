# MF2-GARCH Toolbox for Matlab (developed by Christian Conrad and Julius Schoelkopf, 2025)

A Matlab package for estimating and forecasting using the multiplicative factor multi-frequency GARCH (MF2-GARCH)  as proposed in Conrad & Engle (2025) accompanying the paper „Long-term volatility shapes the stock markets sensitivity to news“ by Conrad, Schoelkopf, and Tushteva (2024): 

* A comprehensive toolbox for estimating and forecasting using the MF2-GARCH-rw-m.
* Five applications: estimation, news-impact-curve, illustration of long-term component, out-of-sample forecasting, illustration of forecasting behavior 

## Suggested Citation
Please cite as: 
> Conrad, Christian and Schoelkopf, Julius Theodor and Tushteva, Nikoleta, Long-Term Volatility Shapes the Stock Market's Sensitivity to News (2024). Available at SSRN:  http://dx.doi.org/10.2139/ssrn.4632733

and 

> Conrad, Christian and Julius Schoelkopf. 2025. MF2-GARCH Toolbox for Matlab. Matlab package version 0.1.0. 

## Contact 
Please address any questions about the Matlab code to:
* Christian Conrad,  Heidelberg University, Department of Economics. Email: christian.conrad [at] awi.uni-heidelberg.de 
* Julius Schoelkopf, Heidelberg University, Department of Economics. Email: julius.schoelkopf [at] awi.uni-heidelberg.de 

We do not assume any responsibilities for results produced with the available code. Please let us know, if you have suggestions for further versions or find any bugs. 

# Applications 

## Estimation of the MF2-GARCH-rw-m model in Matlab for S&P 500 stock returns. 
Define daily log-returns as $y_t=\sigma_t Z_t=$ $\sqrt{h_t \tau_t} Z_t$ where $Z_t$ is i.i.d. and has a symmetric density with mean zero and variance one. $\sigma_t^2$ denotes the conditional variance and the short- and long-term volatility components are given by $h_t$ and $\tau_t$. Let `y` be this (Tx1) vector of daily log-returns. The short-term volatility component is defined as a unit variance GJR-GARCH(1,1)
```math
h_t=(1-\phi)+\left(\alpha+\gamma \mathbf{1}_{\left\{r_{t-1}<0\right\}}\right) \frac{r_{t-1}^2}{\tau_{t-1}}+\beta h_{t-1}
```
and the long-term component is specified as a MEM equation for the conditional expectation of $V_t$ (squared deGARCHed returns):
```math
\tau_t=\lambda_0+\lambda_1 V_{t-1}^{(m)}+\lambda_2 \tau_{t-1}
```
where
```math
V_{t-1}^{(m)}=\frac{1}{m} \sum_{j=1}^m V_{t-j}=\frac{1}{m} \sum_{j=1}^m \frac{y_{t-j}^2}{h_{t-j}}.
```
The estimation of this model can be done using the following function from our toolbox in Matlab
```matlab
[coeff, qmle_se, p_value_qmle,  Z, h, tau, sigma_annual, tau_annual, annual_unconditional_vola, foptions]  = mf2_garch_estimation(y,foptions); 
```

The function `mf2_garch_estimation(y,foptions)` provides you with an estimation output for the seven parameters $\left(\alpha, \gamma, \beta, \lambda_1, \lambda_2, \lambda_3\right)$ of the short- and long-term component in the command window obtained by maximizing the log likelihood. The output of the function are the vectors for the coefficient estimates (`coeff')`, the Bollerslev-Wooldridge robust standard errors  (`qmle_se`), and the corresponding p-values  (`p_value_qmle`). Moreover, the function `mf2_garch_estimation(y,foptions)` provides you with fitted values for $Z$, the conditional volatility as well as the long- and short-term component. the shocks (`Z`), the fitted values for the short (`h`) and long-term component (`tau` or annualized `tau_annual`) as well as the time series for the annualized conditional volatility (`sigma_annual`) and the estimate for the annualized unconditional volatility (`annual_unconditional_vola`). 

For the long-term component, you need to specify $m$, i.e. the number days over which $V_t^m$ is computed. Choose whether you want to use a fixed value of $m$ or let the optimal $m$ be selected as the one that minimizes the BIC. The `foptions` structure contains the researcher's choice for $m$. You either specifiy `foptions.choice = 'BIC'` if the optimal $m$ needs to be selected or `foptions.choice = 'fix'` together with the choice of your $m$ as `foptions.m=63`. For the fitted values, the code discards the first two years of y (i.e., 2 times 252 trading days) to account for lags of the squared deGARCHed returns when comparing models using the BIC. You could decrease this, but you need to discard at least $2m$ values. The Matlab function uses constraints on the parameters following assumption 2 (for the short-term component) and assumption 3 (for the long-term component) of Conrad & Engle (2025). For details on the estimation, see section A.1.1 in Conrad & Engle (2025). 

The following application of the MF-2GARCH replicates the second panel in Table 2 in Conrad & Engle (2025) for the MF2-GARCH-rw-m. In Conrad & Engle (2025), all models were estimates using OxMetrics. We use daily S&P 500 log-return data from January 1971 to June 2023. For the sub-period 1971-1983, the return data was initially obtained from the Federal Reserve Bank of St. Louis database.  Data from 1983 onwards are from TickData. 

```matlab
%% Import the return data to Matlab (S&P500 returns from 1971-2023) 

% Read the data into a table
Returns = readtable('data/SP500_1971_2023_06_30_ret.xlsx');

% Extract the column 'RET_SPX' from the table and store it 
y = Returns.RET_SPX;

%% Select the m for the estimation
foptions.choice = 'fix'; % choices: 'BIC' or 'fix' (specify m) 

% If f.options.choice = 'fix', please specify the m you choose here: 
foptions.m=63;

%  Example A (Estimation) for regresion output: 
mf2_garch_estimation(y,foptions); 
```
This yields the following output in the command window: 
```matlab
===================== Estimation results MF2-GARCH-rw-m =====================

The optimal m was specified by the user: m = 63
Log-Likelihood Function = -16678.611, BIC = 2.524
Estimated fourth moment of the innovations: kappa = 5.441
     Parameter      Coefficient    Standard Error     p-value      Significance
    ____________    ___________    ______________    __________    ____________

    {'mu'      }      0.030395       0.0069646       1.2758e-05       "***"    
    {'alpha'   }     0.0032236       0.0026327          0.22078       ""       
    {'gamma'   }       0.16169        0.020378       2.2204e-15       "***"    
    {'beta'    }       0.83956        0.017385                0       "***"    
    {'lambda_0'}      0.017512       0.0072155         0.015226       "**"     
    {'lambda_1'}       0.11183        0.046429         0.016013       "**"     
    {'lambda_2'}       0.87014        0.051675                0       "***"    

Output reports Bollerslev-Wooldridge robust standard errors (see Conrad and Engle (2025), 
equation (27)).
Covariance stationarity condition satisfied (see Conrad and Engle (2025),
equation (7)): Gamma_m = 0.778
Annualized unconditional volatility = 16.043
==============================================================================
```

If you additionally want to have the fitted values, specify the output as follows 
```matlab
[coeff, qmle_se, p_value_qmle,  Z, h, tau, sigma_annual, tau_annual, annual_unconditional_vola, foptions]  = mf2_garch_estimation(y,foptions);
```
You can use this output for instance for a figure of the estimated conditional volatility and long-term volatility over the full-sample. For this figure, we use annualized quantitities. Grey shaded areas represent NBER recession periods for the US.
```matlab
% Extract the date column (not required for estimation, only for figure) 
dates = datetime(Returns.OBS, 'InputFormat', 'MM/dd/yyyy'); 

% Figure of time series
mf2_garch_time_series(dates, sigma_annual, tau_annual);
```
The function exports the following figure in the figures folder: 
<img src="figures/Time Series.png" width="50%" />
