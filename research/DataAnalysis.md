# Data Analysis

I didn't go crazy with my data analysis. Since my data wasn't being generated by a PRNG and came from environmental sources, I'm basically willing to call my data "random" in that it's not easily predictable. So all I really wanted to do was massage the data until it appeared to produce samples that were roughly uniform in distribution. I looked at two methods to evaluate the uniformity of the distribution: Chi-Square and Kolmogorov-Smirnov.

For information on how I generated/collected the data samples, see the [Data Collection](DataCollection.md) doc.

## Chi-Square Test

The chi-square test is often used when trying to determine correlation between a table of values - in our case, a histogram of the values we measured from the sensors and the number of times that result occurred. For example, if we observed rolling a 6-sided die, the values would range from 1-6 and then we would count the number of times each result occurred.

So I tried this, using the `chi2_contingency()` function in python. (You can find a very simple example [here](https://www.askpython.com/python/examples/chi-square-test).) But I quickly found that this test [doesn't work well with large data sets](https://stats.stackexchange.com/questions/440371/chi-square-test-with-high-sample-size-and-unbalanced-data).

## Kolmogorov-Smirnov Test

The next most popular suggestion was using the Kolmogorov-Smirnov Test. This is sort of a curve-fitting test, to see if the distribution of your data closely matches a well-known distribution. In my case, I wanted to know if the samples were [uniformly distributed](https://www.statology.org/uniform-distribution/). That would mean that any given value in the range appears to be equally likely to occur.

There's a nice NumPy function called `scipy.stats.kstest` (some info [here](https://www.statology.org/kolmogorov-smirnov-test-python/)) that returns a magic number called a [p-value](https://statisticsbyjim.com/glossary/p-value/), which you then compare to your chosen significance level, [alpha](https://statisticsbyjim.com/glossary/significance-level/). If your p-value is greater than alpha, then you "accept the [null hypothesis](https://statisticsbyjim.com/glossary/hypothesis-tests/)". 

It took me a while to get this working in python. I spent hours on this getting weird results until I figured out (with the help of an online forum) that I need to feed `kstest` a [CDF](https://en.wikipedia.org/wiki/Cumulative_distribution_function), not a [PDF](https://en.wikipedia.org/wiki/Probability_density_function).

## Analyzing the Data

Here's the basic process I followed to analyze the data from the Amulet of Entropy:

1. Gather 10,000 samples of data from a given sensor.
2. Generate a histogram of the data.
3. Convert the histogram to a probability density function (PDF).
4. Convert the PDF to a cumulative distribution function (CDF).
5. Feed the CDF to `kstest()`, comparing it to the "uniform" CDF, and calculate the p-value.

**Accepting the null hypothesis.** If the p-value is greater than the significance level (alpha), then the values in your samples appear to all be equally likely to occur. And that's good enough for me in this experiment.

I took the full-resolution samples, and then successively lopped off the highest order bits of each sample until the remaining truncated samples met the criteria.

## Results

The usual alpha value for these tests is usually 0.1 or even 0.05. Looking at the CDF and histogram, though, that seemed a little low. I went with an alpha of 0.5, which is probably ridiculous... but hey, you gonna fault me for being conservative?

Given that, here are the results for each sensor, in both "active" and "passive" modes. (See [Data Collection](DataCollection.md) for details.) The "usable bits" column shows how many least-significant bits I could use that appeared to be uniform, and the p-value that corresponded to that number of bits. Basically, if you added one more bit, the p-value would be less than my target of 0.5 (which, again, is pretty high for this sort of test).

|Sensor|Mode|Max Bits|Usable Bits|P-Value|
| :-: | :-:| :-: | :-: | --- |
|shot|N/A|12|3|0.57984586|
|light|passive|12|4|0.57234960|
|light|active|12|10|0.60882407|
|temp|passive|12|8|0.94602627|
|temp|active|12|9|0.53076195|
|motion|passive|16|5|0.99861798|
|motion|active|16|14|0.99518491|
|accel|passive|16|7|0.99458618|
|accel|active|16|13|0.99998669|

The idea then would be to alter the entropy harvesting code to take the given number of bits from each sensor's readings, probably using the worst case values ("passive").

I may or may not get around to altering the original code to do this. Right now, as of this writing, the code just takes the lowest 8 bits, which is not really good enough for the shot circuit or light sensor. It's probably okay for the motion/accel sensors - it would take very little motion to get good uniform samples.

## Going Further

If you really want to go down the rabbit hole on testing a data set for randomness, here are some places to start:

* [Random Number Testing](https://gerhardt.ch/random.php) (Ilja Gerhardt)
* [Diehard Test]()
* [Dieharder Test](https://webhome.phy.duke.edu/~rgb/General/dieharder.php) (Robert G. Brown)
* [ENT](https://www.fourmilab.ch/random/): A pseudorandom number sequence test
* [Randomness](http://www.ciphersbyritter.com/NETLINKS.HTM#RandomnessLinks) > Tests (Cyphers by Ritter)
