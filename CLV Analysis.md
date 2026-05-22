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
