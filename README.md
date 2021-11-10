[![Build Status](https://github.com/broadinstitute/millipede/workflows/CI/badge.svg)](https://github.com/broadinstitute/millipede/actions)
[![Documentation Status](https://readthedocs.org/projects/millipede/badge/?version=latest)](https://millipede.readthedocs.io/en/latest/?badge=latest)
      

# millipede: A library for Bayesian variable selection
```
                                        ..    ..
                           millipede      )  (
                      _ _ _ _ _ _ _ _ _ _(.--.)
                     {_{_{_{_{_{_{_{_{_{_( '_')
                     /\/\/\/\/\/\/\/\/\/\ `---
```

millipede is a PyTorch-based library for Bayesian variable selection that can be run on both CPU and GPU and
can handle datasets with numbers of data points and covariates in the tens of thousands or more.
 
## What is Bayesian variable selection?

Bayesian variable selection is a model-based approach for identifying parsimonious explanations of observed data.
In the context of generalized linear models with :math:`P` covariates :math:`{X_1, ..., X_P}` and responses :math:`Y`, 
Bayesian variable selection can be used to identify *sparse* subsets of covariates (i.e. far fewer than :math:`P`) 
that are sufficient for explaining the observed responses.

In more detail, Bayesian variable selection is formulated as a model selection problem in which we consider 
the space of :math:`2^P` models in which some covariates are included and the rest are excluded.
For example, one particular model might be :math:`Y = \beta_3 X_3 + \beta_9 X_9`.
A priori we assume that models with fewer included covariates are more likely than those with more included covariates.
The set of parsimonious models best supported by the data then emerges from the posterior distribution over the space of models.

What's especially appealing about Bayesian variable selection is that it provides an interpretable score
called the PIP (posterior inclusion probability) for each covariate :math:`X_p`. 
The PIP is a true probability and so it satisfies :math:`0 \le \rm{PIP} \le 1` by definition.
Covariates with large PIPs are good candidates for being explanatory of the response :math:`Y`.

Being able to compute PIPs is particularly useful for high-dimensional datasets with large :math:`P`.
For example, we might want to select a small number of covariates to include in a predictive model (i.e. feature selection). 
Alternatively, in settings where it is implausible to subject all :math:`P` covariates to 
some expensive downstream analysis (e.g. a laboratory experiment),
Bayesian variable selection can be used to select a small number of covariates for further analysis. 
  

## Requirements

millipede requires Python 3.8 or later and the following Python packages: [PyTorch](https://pytorch.org/), [pandas](https://pandas.pydata.org/), and [polyagamma](https://github.com/zoj613/polyagamma). 

Note that if you wish to run millipede on a GPU you need to install PyTorch with CUDA support. 
In particular if you run the following command from your terminal it should report True:
```
python -c 'import torch; print(torch.cuda.is_available())'
```


## Installation instructions

Install directly from GitHub:

```pip install git+https://github.com/broadinstitute/millipede.git```

Install from source:
```
git clone git@github.com:broadinstitute/millipede.git
cd millipede
pip install .
```

## Basic usage

Using millipede is easy:
```python
# create a VariableSelector object appropriate to your datatype
selector = NormalLikelihoodVariableSelector(dataframe,  # pass in the data
                                            'Response', # indicate the column of responses
                                            S=1,        # specify the expected number of covariates to include a priori
                                           )

# run the MCMC algorithm to compute posterior inclusion probabilities and other posterior quantities of interest
selector.run(T=1000, T_burnin=500)

# inspect the results
print(selector.summary)
```

See the Jupyter notebooks in the [notebooks](https://github.com/broadinstitute/millipede/tree/master/notebooks) directory for detailed example usage.


## Supported data types 

The covariates :math:`X` are essentially arbitrary and can be continuous-valued, binary-valued, a mixture of the two, etc.
Currently the response :math:`Y` can be any of the following:
- continuous-valued &nbsp;&nbsp; => &nbsp;&nbsp; use `NormalLikelihoodVariableSelector`
- binary-valued &nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; => &nbsp;&nbsp; use `BernoulliLikelihoodVariableSelector`
- bounded counts  &nbsp;&nbsp;&nbsp; &nbsp; &nbsp; => &nbsp;&nbsp; use `BinomialLikelihoodVariableSelector`
- unbounded counts  &nbsp;&nbsp; => &nbsp;&nbsp; use `NegativeBinomialLikelihoodVariableSelector`


## Scalability

Roughly speaking, the cost of the MCMC algorithms implemented in millipede is proportional
 to :math:`N \times P`, where :math:`N` is the total number of data points and :math:`P` is the total number of covariates. 
For an **approximate** guide to hardware requirements please consult the following table:

| Regime                                    | Expectations           |
| ------------------------------------------|------------------------|
| :math:`N x P \lesssim 10^7`               | Use a CPU              |
| :math:`10^7 \lesssim N x P \lesssim 10^9` | Use a GPU              |
| :math:`10^9 \lesssim N x P`               | You may be out of luck |


## Documentation

Read the docs [here](https://millipede.readthedocs.io/en/latest/)


## Contact information

Martin Jankowiak: mjankowi@broadinstitute.org


## References

Zanella, G. and Roberts, G., 2019. [Scalable importance tempering and Bayesian variable selection](https://rss.onlinelibrary.wiley.com/doi/abs/10.1111/rssb.12316). Journal of the Royal Statistical Society: Series B (Statistical Methodology), 81(3), pp.489-517.

Jankowiak, M., 2021. [Fast Bayesian Variable Selection in Binomial and Negative Binomial Regression](https://arxiv.org/abs/2106.14981). arXiv preprint arXiv:2106.14981.

## Citations

If you use millipede please consider citing:
```
@article{zanella2019scalable,
  title={Scalable importance tempering and Bayesian variable selection},
  author={Zanella, Giacomo and Roberts, Gareth},
  journal={Journal of the Royal Statistical Society: Series B (Statistical Methodology)},
  volume={81},
  number={3},
  pages={489--517},
  year={2019},
  publisher={Wiley Online Library}
}

@article{jankowiak2021fast,
  title={Fast Bayesian Variable Selection in Binomial and Negative Binomial Regression},
  author={Jankowiak, Martin},
  journal={arXiv preprint arXiv:2106.14981},
  year={2021}
}
```
