# RFM_Analysis_with_-DAX-

When reporting customer analysis to management, total sales alone may not give the full picture.

Buying a lot does not always mean the customer is the most valuable.

That is why RFM Analysis is useful:

Recency – how recently the customer purchased
Frequency – how often the customer purchased
Monetary – how much the customer spent

By combining these three rules, we can categorise customers more meaningfully and build dashboards that support better sales follow-up, retention, and customer strategy.

This helps management see not only who spent the most, but also who is active, loyal, inactive, or worth re-engaging.


### Recency	how recently the customer purchased

Recency Score >	Days Since Last Purchase

    Last Invoice Date = max (fct_sales[OrderDate])

    Last Invoice ALL Date = MAXX(ALL(fct_sales), fct_sales[OrderDate])	

    Days Since Last Purchase = DATEDIFF([Last Invoice Date],[Last Invoice ALL Date],DAY)
	
Recency Score >	Months Since Last Purchase 
	
    Months Since Last Purchase = 
		DATEDIFF([Last Invoice Date],
					[Last Invoice ALL Date],MONTH
				)

### Giving Score Logic 	

(If last purchase is >= 365) = 0 

if not we will give (1- last transaction days) and divided by 365. 

But if there has customer who never buy before will get same score with today purchase customer. 

To handle this we will give blank for those who never buy before. Recent buy is 1 and the rest (1-last trans)/365

    Recency Score = 
	If(
	    ISBLANK([Days Since Last Purchase]),"",
	If (
	    [Days Since Last Purchase] > 365, 0,
	    (1-Divide([Days Since Last Purchase],365))
	 )
    )


Categorised in Table 	

    Months Since Last Purchase CC = 

	SWITCH(TRUE(), 

			[Months Since Last Purchase] >= 0 && [Months Since Last Purchase] <=1, "1",
	        			
			[Months Since Last Purchase] = 2 , "2",
	        			
			[Months Since Last Purchase] = 3 , "3",
	        			
			[Months Since Last Purchase] >= 4 && [Months Since Last Purchase] < 7, "4 to 6",
	        			
			[Months Since Last Purchase] >= 7 && [Months Since Last Purchase] < 13, "7 to 12",
	        			
			[Months Since Last Purchase] >= 13, "13 or more" 
	        			
			)


Sorting column 	Months Since Last Purchase Sorting =  

    SWITCH(TRUE(), 
	        [Months Since Last Purchase] >= 0 && [Months Since Last Purchase] <=1, 6,
	        [Months Since Last Purchase] = 2 , 5,
	        [Months Since Last Purchase] = 3 , 4,
	        [Months Since Last Purchase] >= 4 && [Months Since Last Purchase] < 7, 3,
	        [Months Since Last Purchase] >= 7 && [Months Since Last Purchase] < 13, 2,
	        [Months Since Last Purchase] >= 13, 1
	        )

## Frequency how often the customer purchased

Total Transactions 	

    TY Transaction = DISTINCTCOUNT(fct_sales[OrderNumber])

Average Days Formula	

    Avg Days Bet Purchases = 
	    VAR TotalPurchases = [TY Transaction]
	    VAR AvgDays = 

    IF( TotalPurchases > 1, 
	    -- 4. for those customer who buy more than one time, we will calculate as following steps, if not Blank()
	    
      AVERAGEX(-- 3. Find AVERAGEX for days between 
	    ADDCOLUMNS( -- 2. ADDCOLUMNS to know "Days Between" 
	            SUMMARIZE(--1. Create new column in SUMMARISE as "PrevDate"
	                    fct_sales, 
	                    fct_sales[CustomerKey],fct_sales[OrderDate], 
	                    "PrevDate", 
	                    CALCULATE(Max(fct_sales[OrderDate]),
	                                FILTER(
	                                    ALL(fct_sales),
	                                    fct_sales[CustomerKey] = EARLIER(fct_sales[CustomerKey]) && 
	                                    fct_sales[OrderDate] < EARLIER(fct_sales[OrderDate])
	                                    )
	                                )
	                    ) --1. 
	                    ,"DaysBetween", DATEDIFF([PrevDate],fct_sales[OrderDate],day) -- 2. 
	                ),[DaysBetween] -- 3. 
	        ), BLANK() -- 4. 
	)
    RETURN AvgDays
Frequency Score	

    Frequency Score = 
	--Step 1 to calculate average days between purchases (never buy 0, if buy calculate avg
	VAR Step1 = 
	    IF([Avg Days Bet Purchases] > 365, 0,
	        1- (DIVIDE([Avg Days Bet Purchases],365,BLANK()))
	    )
	-- If customer buy only one time, step 1 will return 1 
	-- to handle this we will give 0 for those customer too
	VAR Step2 = IF(ISBLANK([Avg Days Bet Purchases]), 0, step1)
	Return Step2

Categorised in Table ((Frequency Score)	

    Frequency Of Purchases CC = 
	        SWITCH(TRUE(), 
	                [TY Transaction] = 1 , "1",
	                [TY Transaction] >= 2 && [TY Transaction] < 10, "2 to 9", 
	                [TY Transaction] >= 10 && [TY Transaction] < 50, "10 to 49", 
	                [TY Transaction] >= 50 && [TY Transaction] < 200, "50 to 199", 
	                [TY Transaction] >= 200 ,  "200 or more"
	        )
Sorting column (Frequency Score)

    Frequency Of Purchases Sorting = 
	        SWITCH(TRUE(), 
	                [TY Transaction] = 1 , 5,
	                [TY Transaction] >= 2 && [TY Transaction] < 10, 4, 
	                [TY Transaction] >= 10 && [TY Transaction] < 50, 3, 
	                [TY Transaction] >= 50 && [TY Transaction] < 200, 2, 
	                [TY Transaction] >= 200 , 1
	        )

## Monetary how much the customer spent

Calculating based on how much they buy 

    Monetary Score = 
	    SWITCH(TRUE(), 
	    [TY Sales] > 10000 , 10,
	    [TY Sales] > 9000 , 9,
	    [TY Sales] > 8000 , 8,
	    [TY Sales] > 7000 , 7,
	    [TY Sales] > 6000 , 6,
	    [TY Sales] > 5000 , 5,
	    [TY Sales] > 4000 , 4,
	    [TY Sales] > 3000 , 3,
	    [TY Sales] > 2000 , 2,
	    [TY Sales] > 1000 , 1,
	    [TY Sales] > 1 , 0.5,
	    0 -- 0 is for those who never buy before 
	    )

## RFM Score Calculation 

    RFM Score = [Recency Score] * [Frequency Score] *[Monetary Score]


# RFM vs CLV Analysis
### RFM = current customer engagement 
### CLV = long-term customer value 

So:

•	highest spender historically may not be active now 

•	most active customer now may not yet generate highest total value 

To correct business approach, we should NOT use: only RFM or only CLV to define customer value. 

We should combine both for better Business Approach.

## CLV Calculation 
CLV =(Average Purchase Value) × (Purchase Frequency) × (Customer Lifespan)

## DAX Formula 
    --Average Sales Amount
	Avg SalesAmt = 
    AVERAGEX(fct_sales,[TY OrderQty] * RELATED(dim_product[ProductPrice]))
	
	-- Purchase Frequency 
    TY Transaction = DISTINCTCOUNT(fct_sales[OrderNumber])

	-- Cusotmer Lifespan (days)
	Customer Lifespan Days = 
    DATEDIFF(
    MIN(fct_sales[OrderDate]),
    MAX(fct_sales[OrderDate]),
    DAY
    )

	--Customer Lifespan Year
	Customer Lifespan Years = DIVIDE([Customer Lifespan Days], 365)


## Practical Business Logic

Who is buying actively now?				= RFM

Who generated biggest business value?	= CLV

Who is at risk of leaving?				= High CLV + Low RFM

Who has future growth potential?		= High RFM + Medium CLV

Who deserves VIP treatment? 			=	High both

## DAX logic for RFM & CLV Level to define customer segment: 
    RFM Level = 
    SWITCH(
    TRUE(),
    [RFM Score] <= 2, "Low",
    [RFM Score] <= 5, "Medium",
    [RFM Score] <= 8, "High"
    )

	CLV Level = 
    SWITCH(
    TRUE(),
    [CLV] < 12000, "Low",
    [CLV] < 15000, "Medium",
    "High"
    )

## Business Segmentation Strategy

Champions				= High RFM + High CLV		> VIP, loyalty rewards

Loyal Customers			= High RFM + Medium CLV		> Upsell/cross-sell

Big Spenders at Risk	= High CLV + Low RFM		> Retention campaign

New Potential Customers	= Medium RFM + Low CLV		> Nurture growth

Lost Customers			= Low both					> Re-engagement or ignore

## DAX Formula for Customer Segmentation

    Customer Segment = 
    SWITCH(
    TRUE(),
    [RFM Level] = "High" && [CLV Level] = "High",
    "Champion",
    [RFM Level] = "High" && [CLV Level] = "Medium",
    "Loyal Growth",
    [RFM Level] = "High" && [CLV Level] = "Low",
    "Potential Customer",
    [RFM Level] = "Low" && [CLV Level] = "High",
    "At Risk VIP",
    [RFM Level] = "Medium" && [CLV Level] = "High",
    "Needs Attention",
    [RFM Level] = "Low" && [CLV Level] = "Low",
    "Low Value",
    "Regular"
    )

## Example : 

High CLV + Low RFM = These customers are dangerous to lose.

Because:

•	they already proved spending power 

•	but activity is dropping 

Management usually prioritizes these customers.

High RFM + Lower CLV = These customers are:

•	active 

•	loyal 

•	engaged 

But spending still small.

Business goal:

•	grow wallet share 

•	increase basket size 

•	convert to premium customers

