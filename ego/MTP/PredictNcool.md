reactive strategies reduces memory power consumption when temperature reaches the maximum operating limit. eg when it reaches 80$\degree$c.
aggresive throttling.

proactive strategies predicts the temperature before it reaches the limit, lets say if at 78$\degree$c it predicts that it will exceed max limit then it applies some policy.
spreads cooling over time.

steady state, temperature gradient stops changing, incoming heat == outgoing heat

thermal equilibrium, no heatflow between system a and system b.

TSV layer 1 -- microbump -- TSV layer 2

reactive will causes either freq. invocations or aggresive throttling.
proactive approach, applies the appropriate DTM level and minimize number of cooling periods and aggresive throttling.

contributions:
- runtime steady-state temperature predictor that uses layout/floorplan designer insights to reduce model parameters.
- proactive DTM policy to reduce memory energy consumption and improve application performance.

more parameters -> more training samples needed to estimate them reliably.

fewer parameters -> fewer computations -> lower inference latency.

reactive DTM, based on current temperature
proactive DTM, utilizes predicted temperature

migration vs throttling

from one temp to steady-state on constant heat, time follows thermal time constant curve.

$$f(p,q,r)=\sum_{i=1}^{4}\sum_{j=1}^{4}\sum_{k=1}^{8} W(i,j,k)(p,q,r)\,A(i,j,k)+\sum_{i=1}^{4}\sum_{j=1}^{4} L(i,j)(p,q,r)\,O(i,j)+C(p,q,r)$$

C - baseline heating, intercept term

we use prediction because, hotspot resistance matrices doesn't consider temperature dependent leakage power and symmetries in floorplan.

insights:
- positive cross-coupling between access rate and temperature
- larger the distance, smaller the weight

high accuracy doesn't mean the system properties are learned by the predictor. extensive training data is required for the learned weights to capture system properties.

*symmetries*
- symmetrically located ranks with respect to a given rank have same weight.
- weights can be reused for symmetrically located ranks in a layer. need to train for only 3 locations, ${(1,1), (2,1), (2,2)}$ 
- symmetry in cross-coupling

*using symmetry in training*
- ranks that have same weight, their access rates can be added and a single weight could be learned for both.
- other 2 symmetries can also be used.

insight,
- lower layer has the maximum temperature, we can go from rank to channel level by assuming most DTM are applied when temp reaches a max limit and memory controller distributes accesses uniformly to ranks in channel.

if predictor takes a long time 442$\mu s$ and you want that to be less than 10% of the total control-loop time, then the control epoch must be at least 4.42 milliseconds.A longer epoch means the controller updates less often, so it reacts more slowly to temperature changes.

more layers/channels, more leakage, less accuracy of linear predictor

2d memory become a bottleneck in case of lbm, which is a highly memory intensive workload and thus, has significant migrated accesses.

less number of channels closed in pnc in comparison to fastcool, causing less stalls.

