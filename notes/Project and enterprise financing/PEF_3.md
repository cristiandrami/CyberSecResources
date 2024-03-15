
# Corporate bond financing


## Zero Bonds and coupon bonds 
A zero-coupon bond is _a debt security that doesn't pay interest but trades at a deep discount, rendering_ profit at maturity when it is redeemed.
With a zero, instead of getting interest payments, you buy the bond at a discount from the face value of the bond and are paid the face amount when the bond matures.

the basis for valuing fixed income securities is the discount function P(T) where 
- P(T) is the present value of a future cash flow of one currency unit at time T. It is also called discount factor
	- example in 6 mounts we have a cash flow of 1 euro and the current value is 0.9759 
	- ![[Pasted image 20240306160614.png]]
	- P(0.5) because 6 mounths are 0.5 year
- ![[Pasted image 20240306160931.png]]
	- In this case we know thw Zero Bond A, so we can calculate the value of the Zero Bond B at time 0 from it



On a perfect market the actual market price of such a zerobond has to be equal to the calculated present value.

Otherwise, arbitrage opportunities exist, that results in buy and sell orders that forces the market price towards the present value of the zerobond.



## EXAMPLE
![[Pasted image 20240306161444.png]]

- compro A a 480 ma fra 6 mesi vale 500 (fra 6 mesi ho +500)
- vendo B a 487.95 ma fra 6 mesi vale 500 (quindi conto di non avere 500, ovvero -500)

The arbitrage free price band is the price + or - the transaction costs.
![[Pasted image 20240306162205.png]]

![[Pasted image 20240306162224.png]]


## EXAMPLE 2
![[Pasted image 20240306162437.png]]


## Valuation rule 1 for zerobounds
![[Pasted image 20240306162539.png]]



## Example (present value of a zerobond)
![[Pasted image 20240306162701.png]]




## Statements about discount functions
1. for a particular time to maturity T one one discount factor P(T) exists
2. P(0) = 1, today the value of one currency unit is exactly 1 currency unit
3. P(infinite) -> 0, incresing time to maturity the present value converges to 0
4. P(T) <= 1, one currency unit is less in the future than today
5. P(T) > 0 the present vale of one currency unit is always positive
6. P(T) <= P(S) if T>S, the larger the time to maturity of a cash flow is the lower is its value today
![[Pasted image 20240306163457.png]]




### Spot rates (zero coupon yields)
Spot rates are alternatives to the discount function

The spot rate r(T) represents the annual interest rate underlying a zerobond with maturity of T years.

![[Pasted image 20240306163819.png]]
We can see that P(1) is equal in the discount function and inthe spot rates. 



There are 3 different types of rate term structures:
![[Pasted image 20240306163946.png]]
In 90% of all cases we have normal one, in 5% the other two.


![[Pasted image 20240306164456.png]]


![[Pasted image 20240306165726.png]]



## Coupon Bond 
When a corporation or government wants to borrow money from the public on a long-term basis it sells debt securities (bonds).

A bond is an interest-only loan (the borrower pays the interest every period, but the principal is repaid only at the end of the loan).

![[Pasted image 20240306180832.png]]



A bond can be split in a portfolio of zerobonds:
![[Pasted image 20240306181004.png]]



![[Pasted image 20240306181116.png]]
![[Pasted image 20240306181350.png]]

![[Pasted image 20240306181531.png]]

- The value of the cupon bond is the sum of the values of the zero bonds in the portfolio

![[Pasted image 20240306181803.png]]


## Valuation of coupon bonds between coupon payment dates 
