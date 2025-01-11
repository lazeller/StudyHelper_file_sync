## Iago Attacks
see [[Iago attacks]]
## Timing Side Channel
- measure execution time
- Classic timing attack -> 
	- Simple Timing Analysis
	- Differential Timing Analysis
- For RSA (Montgomery), the execution time $T_{mont}$ depends on the plaintext m **and** on the secret d. 
- Divide the problem (one bit at the time!)
- We can measure if the victim was fast or slow (very small difference buried in noise)
- **Given a guess, we can predict if the victim was fast or slow using a model (e.g., simulation)**
- After many measurements, check for which guess the model best matches the measurement using a statistical test

## Power Side Channel
- measure some physical quantity influenced by execution
- Simple Power Analysis (SPA)
- Differential Power Analysis (DPA) (attack)
	- e.g. with the assumption that high and low Hamming weights behave observably different from each other --> put measurements with "extreme" Hamming weights into two buckets and compare the difference of their averages
- Correlation Power Analysis (CPA) (attack)
	- e.g. with the assumption that the hamming weight and power consumption are linearly correlated --> use Pearson Correlation

TODO: maybe add the pseudocode from the exercise session slides 02 to this tiny summary