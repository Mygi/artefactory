# Cellular Automata and Algorithms for thinking about complex state.

I have a bit of a passion for cellular automata. I discovered it when studying Philosophical Concepts of Time and Space at University in 2002. There where various ideas as to what space actually is. One that struck a cord with me was the cellular automata conception of space. 

The history of Cellular Automata(C.A) is pretty well known - but it is always worth taking off your hat to Conway's game of life in the 1940s and then the extensive work by John Von Neumann and Stanislaw Ulam in the 1950s. Ed Fredkin is probably the first thinker to speculate on the discretisation of space-time, whilst physicist John Wheeler takes it further, with his 'It from Bit' idea. In fact, I was surprised to find a 2017 article in Forbes by Paul Halpern on this very topic so I might just point to that as a better wrap up than I can give: https://www.forbes.com/sites/startswithabang/2017/09/26/it-from-bit-is-the-universe-a-cellular-automaton/#4ee81ddd46b7. 

A more recent protagonist? for C.As emerged in Stephen Wolfram  who discusses automata in detail and on a (somewhat) recent musing; even describes the space time C.A conception  in: What is Spacetime, here: https://writings.stephenwolfram.com/2015/12/what-is-spacetime-really/. 

I'm not really here to write about the history of C.A. I really just want to point out ways to use it to model real world problems and potentially use it to solve modern technical challenges. It is worth pointing out that there are many ways to write C.As as the principle of discrete contiguous network of automatons in space; each with a state that depends on that of itself and its neighbours - is a concept that can be extended for as many purposes as you can imagine. The simplest version being a connection of automata that only have on/off states and switch on and off relative to the on/off states of their neighbours. 

So lets get started with a problem I have been thing about recently.

##  Foreign Exchange Challenge

I'm going to set a problem. The first solution will be analytical. 

The date is January 1st 2018 and you have the following money in these 8 currencies (you may need to imagine yourself as slightly more well-off):

* 10,000 Swiss Francs
* 10,000 US Dollars
* 10,000 Great British pounds
* 10,000 Singaporean dollars
* 10,000 Euros
* 1,000,000 Yen
* 10,000 Australian Dollars
* 10,000 New Zealand Dollars

In the period of time from January 1st 2018 to January 1st 2020; using daily currency rates alone (rate at the time of the market opening in New Zealand - local Auckland time), what is the greatest return you can make by:

A: Returning it all to one currency

B: Retaining money in all currencies. We can work this out as the sum of of the percentage gain for each currency. Crude, but still meaningful.

This is not an easy problem. Before we consider optimizations, the immediate thought is to look at a graph and perform a brute force pathway search across all time points. How many pathways would that be?

- For 8 currencies and 365 days * 2 years = 5840 connected data points or nodes. 
- The standard answer would be the function f(5840) from:

$$
f(n)=\sum_{k=8}^n\sum_{ij=1}^n (A^k)_{ij}=n^8\sum_{k=0}^{n-3}n^k=\frac{n^8(1-n^{n-2})}{1-n}
$$

- You cannot re-enter the same node ( that would be taking you back in time)

- We are not using Hamiltonian paths, as we do not need to travel through all nodes.

- It is also worth noting that in problem 2, each pathway needs to return to its original node (a cycle) in order for currency to remain in place.

- You cannot move back in time - or trade backwards between currencies. So our graph is effectively directed.

- We are starting at 8 different points with 8 potential path searches.. There is a temptation to use combinations like:
  $$
  \frac{5840!}{8!(5840-8)!}
  $$
  
- Another way to look at it would be to consider that on each day, each currency could have exchanged with any of the seven others. This gives 7! exchanges. The following day another 7! exchanges are possible (assuming money is still in each account). This is starting to look like 
  $$
  7!^{730} = 5.92 exp2702
  $$
  
-  Also in this model,  we disregard how much would have been traded? Does it matter? Probably.

- In theory a transaction could be none / all. Meaning for each of those possible exchanges, there are two possible transactions. So we just doubled the problem as a minimum!

- In this case, you then take in the consideration of non-trading pathways over multiple days. Thus holding on to currency.

- Either way, my need for an Intel Xeon Phi went up as I try to work how long I would need to run this algorithm before finding out I may have inserted a bug.

### Optimizing

Ok an obvious optimization and a great simplification. Look at the max/min value for every exchange rate over that two year time and work off those. I don't mind this because I think we can work inductively by reducing those time frames to see how the return value moves. We would simply shrink the window recursively. 

The problem is that this potentially inhibits our ability to perform more regular exchanges. It may also mean that we lose the value of a longer term trade should we find reducing the window is generally successful. In other words, our answer will not be perfect. 

So we are effectively writing competing algorithms for the best analysed return value over time. I like this...

### Mapping it out. Two Currencies:

Ok let's start simple. Two currencies: from January to February of 2018.

1. Max vs Min:

   1. High: AUD$1.575
   2. Low: AUD$1.297
   3. Gain of 21.42%

2. Buy High / sell low Peaks/Troughs

   1. There we 9 Peak/trough intervals over the 2 years. If I returned all money to AUD and transferred everything at each maxima/minima, I would have made $36,593 AUD from an initial value of $10,000 AUD + CHF10,000 (roughly $23,764 AUD if I exchanged on the first day). That is a 54% return over two years (minus inflation)!
   2. If I needed to return my Swiss francs, I would have had my original 10,000 swiss francs and another AUD19,512. Almost double my initial!

3. Monthly Differentials: AUD vs CHF (Sample: January to February 2018)

   1. CHF buys AUD at

      1. High: $1.3324 
      2. Low: $1.30
      3. Maximum Gain of 2.49%

   2. Lets say that the average return for any month is 3% - one way or another. Then making a trade every month - would give me the joys of compound interest accrued monthly: 
      $$
      A(t)=P(1+r)^{t}
      $$

   3. In a non-decaying model, I end up with AUD$38,082.

   4. With a decaying harmonic model that drops by the 18% experienced in the Australian dollar - you can do even better with monthly returns increasing to 3.5% for about AUD $46,288 (where the original was doubled). 

 4. The great theoretical. In any given currency you could have a number of maximal shifts within a time-frame. You could:

     1. Find the number of times values vacillated by a small percentage and maximise each trade:

     2. In our January to February sample, there were 11 shifts (without decay) of 1% in daily rates. 375% return via compound interest!

     3. Whereas there was one 2% shift and another 3% in the same month (meaning we could double our time slots but reduce our return to 2.5%): A 321% return!

5. Before we completely conclude on the short-term approach, we need to bare in mind transaction costs and the challenge of getting those 1% gain on rates.  The transfer fee could cost anything from 0.5% to a $20 (Far better on high numbers) set fee - making the return reduce to 363% at best and possibly as low as 192%.  Whereas, the minima of the 2.5% option is a 258% return. 300% looks like a best-in-field outcome for two currency trading.

This leads to two base models to consider in our analysis. A fluctuating system that always normalises around some set of values (a sort of homeostasis if you will) - can be modelled with an analogy between sinusoidal motion and compound interest:
$$
{r}\times sin(\frac{365}{2\pi}t.n) => P(1+r)^{t}
$$
Of course periods of time are not harmonic. The law of averages can account for those variations in an analysis so long as the key features of number of exchanges and peak/trough returns are correct. A lot of information is still lost. We are effectively using the heart rate without its variability.

Decaying  models are a little trickier. For the decaying simple harmonic function:
$$
f(t) = r\times e^{-t}cos(2\pi \frac{t}{k})
$$
We will need to differentiate for the minima and maxima t values and then apply those t values to get the return rate. The period function will still dictate the number of exchanges we make. We get:
$$
\frac{\partial Rate}{\partial t} = 0 =>\: -\frac{2\pi}{k}sin(2\pi \frac{t}{k}) = 0
$$
This will become important as we look at more currencies. Yet it is worth mentioning that the modelling of an entire system has a certain amount of noise with at least a couple of overarching patterns. You could throw those into the model as well in order to minimise the error with the underlying data. The concept of error with curve of fit is especially well known and RMS would be fine for a discrete set of time periods on two variables (this error still matters a lot!).  Multiple currencies will need something more robust to measure error.

Conclusion: Peaks / Troughs and the least amount of intervals is the best solution  - so long as your gains sufficiently outpace the cost of the transaction.

###   Swapping - Three currency systems.

We are going to follow the same approach. We will start with single maxim/minima and reduce time periods. The table of Maximum buying rates works as follows - with each row representing the best buying rate. 

|      | CHF      | AUD         | USD        |
| ---- | -------- | ----------- | ---------- |
| CHF  | CHF 1    | CHF1.575    | CHF1.06383 |
| AUD  | AUD0.771 | AUD1        | AUD0.81    |
| USD  | USD1.03  | USD1.492537 | USD1       |

We can optimize this solution early by making sure we convert the most money at the best rates. This has limits of course - we need to follow chronological order but we can see that the greatest returns come from buying Australian dollars with Swiss francs (just ahead of buying AUD with USD). So we want to maximise our gains by shifting money to AUD at the end. Believe it or not this makes a significant difference with the worst case returning 39.5% whilst the best case returns 46.5%. Either way, this is significantly ahead of the the two currency system.

What is more value is that we now have the specification for a simple three currency system that we should be able to build on. We start with a directed graph:

![image-20200321104336890](C:\Users\ukon\Documents\image-20200321104336890.png)



Our plan is to find the highest cumulative value cycle. A cycle is simply any set of all transitions, where our transitions can be any of:
$$
\{\vec{AB},\vec{AC},\vec{BA},\vec{BC},\vec{CA},\vec{CB}\}
$$
Of course these are constraint free - so time should factor in. Nonetheless, in a brute force case we will be selecting the highest return percentage on 6! pathways. Something our computer can solve well.

So what happens when we add more peaks and troughs? 

Every peak/trough between two currencies creates two new vertices in our graph. So without time constraints, vertices will generate V! pathways to search. In a naïve approach, we could compare to the two currency model and assume we have 27 sets of peaks and troughs which adds up to 54 vertices. The algorithm is growing but we can still solve it in real time. 

Importantly, we are are not going to make more exchanges than there are vertices in our system. This is the transaction itself - we are simply optimising it. In fact, our example in the single peak/trough example was sufficient to describe the solution and point out some key concerns. We can always reduce our problem space to one single set of 6! cycles. We are just compounding the return from each cascading optimisation. 

The key feature of time in our occasion is the difference between frequency of transactions and return rates - and not all exchange systems are equal. In the case of Australian dollars vs Swiss francs - our biggest single peak/trough gain was 21.42% - this was using a 2 year breadth search and whilst he gain occurred in a briefer period of time - we could not ever predict the necessary size of the window. This rate decays until the 1 month search period yields a return that has dropped to around 2.5%. We are essentially trying to optimise  for the maximum amount of transactions against the best possible return rate within that decay. We have the following equations (using normalised fluctuations):
$$
T_{return} = R_oe^{-\lambda t}
$$

$$
Num_{Returns} = \sum_{i=0}^{i=n}\frac{\partial{T_{return}sin(\tau t)}}{\partial{t}}=0
$$

$$
Final_{return}=P_{init}(1+T_{return})^{Num_{returns}}
$$

As we increase the number of available currencies, we increase the number of opportunities to capitalize on higher return rates.

A quick example: 

* If 3 currencies gives us the opportunity to make 6 transactions per month of 2.5% gains, we would get  70 times our initial value. 

* We need to find the point where the doubling time of returns is less than the doubling time of transactions. So 5% gains would require 90 transactions. 10% gains would require 45 and so on and so forth. 

* As you add currencies you increase the number of nodes and vertices. In the case of 8 currencies we found there are 16 vertices per peak/trough interval.  If you could make 1% returns 11 times a month on all of those transactions you would get a return value in the quintrillions! Although, possibly a headache from making those 176 transactions per month.  

* Of course - this is purely theoretical in a perfect system. Lets see what our code yields:

* BUT: Important to note that adding a new currency to the system adds another order of magnitude to the potential of return, it is:

* $$
  (1+Rate)^{{2Node_{transactions}}^{currency}}
  $$

  



### Algorithm 1: Local minima and maxima detection in sequential data with optimisation

In the case of analysis, we are fortunate in that we have bounds on our problem. We have a strict window of time to search and we do not need to know if our minima is global or local. We also know our data ahead of time so we won't need a regression to calculate error as we are not working on a model of the data. So gradient descent might work - but it may not be essential. The golden-section search is another option - which may help if we can create windows of time that serve the purpose of a unimodel function. Lets start simpler (typescript is my hacking code but we will need to turn to c++ soon):

```typescript
export enum LimitTypes {
	globalMinima = 0,
	localMinima = 1,
	localMaxima = 2,
	globalMaxima = 3
}
export interface TwoCurrencyModel {
timeStep: int;
value: double;
}
export interface MaximaMinima {
timePoint: int;
type: LimitTypes;
value: double;
}

export function findMaximaMinamaForWindow(input: TwoCurrencyModel[], windowSize: int): MaximaMinima[] {
	const results: MaximaMinima[] = [];
	// NB: lookahead - need to handle last window
	for( let i = 0; i+windowSize < input.length; i+= windowSize ){
		const localMax = Math.max(input.slice(i, windowSize));
		const localMaxima = {
		 	timePoint: localMax.timeStep,
		 	type: LimitTypes.localMaxima,
		 	value:localMax.value
		}
		result.push( localMaxima );
		const localMin = Math.min(input.slice(i, windowSize));
		const localMinima = {
		 	timePoint: localMax.timeStep,
		 	type: Min,
		 	value:localMax.value
		}
		result.push( localMinima );
	}
    return results;
}
```

This looks nearly adequate, but we will get a bug where a minima/maxima occurs at the end of the window and then continues to rise and fall afterwards. If we reduce down to two day windows - there will be local maxima and minima every two days with tiny returns. Do we need overlapping windows? What about:

```typescript
export function findMaximaMinama(input: TwoCurrencyModel[]): MaximaMinima[] {
	const results: MaximaMinima[] = [];
	let localMaxima = {
		 	timePoint: input[0].timeStep,
		 	type: LimitTypes.localMaxima,
		 	value:input[0].value
		}
	let localMinima = {
		 	timePoint: input[0].timeStep,
		 	type: Min,
		 	value:input[0].value
		}
	let isAcending = false;
	// I actually want to push this to a single index:
    // INDEX: Max / MIN
    // Like setNextMinima / setNextMaxima
	for( let i = 0; i < input.length; i++ ){
		if( input[i].value > localMaxima.value) {
			localMaxima.mapFrom(input);
			if( !isAscending ) {
				results.push(localMinima);
			}
			isAscending = true;
		} else {
			localMinima.mapFrom(input);
			if( isAscending ) {
				results.push(localMazima);
			}
			isAscending = false;
		}
		
	}
    return results;
}
export function reduceWindows(input: MaximaMinima[], windowSize: number): MaximAMinima[] {
    const transformed: MaximaMinima[] =[]
    for( let i = 0; i < windowSize; i++ ) {
        transformed.push(
            Math.max( input.filter( x => x.timePoint > 		 i*windowSize && i < i*windowSize + windowSize)))
            .push(Math.min( input.filter( x => x.timePoint > i*windowSize && i < i*windowSize + windowSize))
    }
    return transformed;
}
    export function optimizeWindowSize(input: MaximaMinima[]): number {
        const returns: numer[];
        let( i = input.length; i > 0; i = input.length / i) 		{
        	returns.push(calculateReturns(reduceWindows( input, i ) );
        }
        return( Math.Max(returns));
    }
  export function calculateReturns(input: MaximaMinima[]) {
      const avgReturn = Math.avg(input => inpput.max - input.min);
      const transactions = input.length; // x2 ?
      return Math.Power(1 + AvgReturn, transactions);
  }
```

This would capture every switch. You could then apply the windowing algorithm to the results.

We can choose any window size (down to 1 set), so we can generate all possibilities until optimised, where our optimisation is to look for: 

* Average Return  vs number of transactions that satisfies:

* $$
  MAX(1+Avg_{return})^{Transactions}
  $$

 Now a spec file.

Finally, do this for every possible two currency system of all you have.

#### Algorithm 2: Build a directed graph of peaks and troughs



#### Algorithm 3: Optimized search through the directed graph 



#### Algorithm 4: Optimising periodicals based on curve fitting of returns model and window size per two node system.



#### Algorithm 5: Putting it together.



Algorithm 6: Error

## Prediction



#### SQL Queries

Select Max(BuyingRate), ToCurrencyISO, Month(Date) as Month, YEAR(Date) as Year from ExchangeRates
where 
FromCurrencyISO = 'SGD'
group by ToCurrencyISO, Month(Date), YEAR(Date) 
order by ToCurrencyISO, year, month



#### References

##### 1. Faulhaber's formula for sum of  falling powers

##### [https://en.wikipedia.org/wiki/Faulhaber%27s_formula](https://en.wikipedia.org/wiki/Faulhaber's_formula)

https://math.stackexchange.com/questions/1899499/how-to-convert-sum-into-formula

##### 2. Bank of Canada Exchange Rates

https://www.bankofcanada.ca/rates/exchange/daily-exchange-rates/

##### 3. Bank of Japan Exchange Rates

https://www.stat-search.boj.or.jp/ssi/cgi-bin/famecgi2

##### 4. Swiss national Bank currency Exchange Rates

https://www.snb.ch/en/iabout/stat/statrep/id/current_interest_exchange_rates#t3

##### 5. Excel Rates for one-to-one

https://excelrates.com/

##### 6. European Central Bank Exchange Rates

https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html

##### 7. Bank of England pound sterling exchange rates

https://www.bankofengland.co.uk/boeapps/database/index.asp?Travel=NIxIRx&levels=2&XNotes=Y&B41122XNode3951.x=9&B41122XNode3951.y=18&Nodes=X3951X3952X3955X3958X3961X3965X3969X3972X3975X3978X3981X3985X3989X3992X3995X3998X4001X4004X4007X4010X4013X4016X4019X41122&SectionRequired=I&HideNums=-1&ExtraInfo=true#BM

##### 8. Reserve Bank of Australia Exchange Rates

https://rba.gov.au/statistics/historical-data.html



