# Charging-of-Interest
An algorithm to recalculate the charging of interest on the consumer credits  
A link to the code at Jupyter nbviewer: https://nbviewer.org/github/Carolus-XII-Rex/Charging-Intersest/blob/master/InterestCharging.ipynb
  
  
## Tasking

I got this task during my work at Sberbank. The procedure of charging interest on the ~40 million consumer credits had to be checked.  
As information relating to credits' operations must not leave bank databases, I have made up a few examples to show typical cases, at the same time taking into account the real volume of data.  

The data consists of two tables:  
1. Rates 
   - contains the interest rates of each credit (rates may change over time, the dates of establishing new rates are provided)
2. ActualOperations
   - contains credits' operations

The operations are the following:
- Credit Issuing: sets the amount of *BalanceOwed*
- Charging interest: charges interest according to the formula:  
  *interest = BalanceOwed * InterestRate / 100 / DaysInYear * DaysInPeriod*  
   where *DaysInPeriod* is the number of days from last charging interest (or credit issuing for the first charging)
- Credit Repayment: decreases the amount of *BalanceOwed*
- Charging interest on the date of restructurization: treated the same as Charging interest
- Overdue payment: transfers a certain amount from *BalanceOwed* to *DebtOwed*
- Charging interest (on debt): charges interest according to the formula:  
  *interest = DebtOwed * InterestRate / 100 / DaysInYear * DaysInPeriod*  
   where *DaysInPeriod* is the number of days from last charging interest on debt (or overdue payment for the first charging)
   

## Algorithm description

First comes the 'detailed' approach. As the algorithm was also to be applied to mortgage and car loans I had written the class Credit. 
Thus it could be further extended to match slightly different conditions of charging interest on those credit types.  

After running the algorithm we can see a couple of credits with interest charged correctly (1 and 2).  

As for the 3 credit we see a typical mistake: 
there is a charging operation (on 2021-04-05), where an extra day was taken into the payment period. Later on (on 2021-05-05) it was 'returned' but there was a Credit repayment between those dates.
So the *BalanceOwed* had decreased by 2021-05-05 thus decreasing the 'returned' amount.  

The 4 credit has another mistake: from a certain date (2021-09-30) there were penalties as there had been no Credit repayments.
Those operations had to be made as separate ones but by mistake were added to ordinary charging operations.  

Thus we have an algorithm which allows to recalculate the interest charging and analyse the results. The main problem here is that it takes ~2.2 seconds per credit on my local machine.
As it was mentioned earlier there are ~40 million of those which sums up to insane time for processing. So there is an optimized version for that (see Expedited option).  

There are several ways for optimization here:
1. The rework of the algorithm:  
    We are now considering establishing the interst rate as a special 'actual operation' so those can be concatenated with the actual operations DataFrame. 
    We will iterate through this one DataFrame (sorted by credit_id and date), keeping in memory the following variables:
    - current_id
    - current_balance_owed
    - current_debt_owed
    - current_balance_previous_date
    - current_debt_previous_date
    - current_rate
    - current_diff = 0
    - current_abs_diff = 0  
    We will not create a daily information DataFrame (which is costly both in terms of memory and speed). Instead we will just change those variables according to the operation as we iterate through them.
   
 2. The memory problem:  
    Encoding the operations and mapping credit ids (in real life there are integers up to billion) to \[0:NumberOfUniqueIds\) would allow us to downcast dtypes of the DataFrame.
 
 3. Using *Dask*:  
    With proper setting up *Dask* client at a big computational cluster we may speed up the iterating through the DataFrame.
    
After applying those things we are now at ~0.1 seconds per credit on my local machine + the potential of Dask allows us to accelerate more through using a cluster.



