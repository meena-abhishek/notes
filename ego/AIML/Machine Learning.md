###### Linear Regression
supervised learning - 
machine learns input to output label mappings
- classification - output is categorical variable
- regression - output is a continous variable

hyperplane,
in a n-dimensional space, a (n-1)-dimensional subspace

slope $m = \frac{rise}{run}$ 

![[linear_regression.png]]
![[cost_function.png]]

sqauring in mean square error prevents +ve and -ve errors from cancelling out and penalizes large errors more heavily

![[slope_intercept.png]]

loss function - error of a single sample
cost function - avg of losses over the entire dataset

*gradient descent*
used to minimize a function.

![[gradient_descent.png]]

linear model,
prediction is linear combination of inputs.

we use activation functions in neural networks to make the model learn non-linear relationships

correlation coefficient,
measures how strongly two variables are related and in which direction.

fitting coefficients,
numerical parameters of a model that are determined by fitting model to observed data.

10-fold cross validation,
split the data into 10 equal parts, then train on 9 parts and validate on the remaining 1 part each round.
