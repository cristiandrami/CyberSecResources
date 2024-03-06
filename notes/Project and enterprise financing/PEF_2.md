# Overdraft facility (most important short term credit financing instruments)

The features of this overdraft facility are:
- Current account 
- Current account line (short time period 1, 2 months to 1 year) 
- Short term 


A company overdraft the credit line when it takes more money that are allowed by the credit line from the bank.
![[Pasted image 20240306110631.png]]

In general using the overdraft adds additional costs to the company.



## Cost factors 
In general when a company uses credit financing instruments it has some costs that are:
- debit interest 
- credit and provision commission

Generally these credit and provision commission are added using a fixed addition to the debit interest, that can be calculated:
- from the credit line
- from the peak demand
- form the not used part (the less i use the more i pay the commission )

![[Pasted image 20240306111304.png]]




Another cost is:
- overdraft commission 



## Calculation of the pure interest costs

To do that we need:
1. divide the account period into intervals with a costant balance amount![[Pasted image 20240306111746.png]]
2. Then we can calculate the interest of each period
	1. interest of period z = ( balance value of period z * interest rate * number of days of period z) / ( 100 * 360 or 365 or  366 ovvero il numero di giorni dell'anno in considerazione )
	2. ![[Pasted image 20240306112134.png]]
3. Then we calculate the sum of the pure interest costs for the account period:
	1. ![[Pasted image 20240306112319.png]]
	2. where ZZ = ![[Pasted image 20240306112333.png]]
	3. and ZT = ![[Pasted image 20240306112346.png]]

	ZT è fisso perchè l'interesse è fisso, quindi a cambiare è solo ZZ poichè dipende dal periodo di tempo 





## Exercise 
![[Pasted image 20240306135940.png]]
![[Pasted image 20240306141324.png]]


Possiamo vedere qui che ci sono le date dei movimenti e quindi il calcolo diventa:
![[Pasted image 20240306140247.png]]

- dal 1/1 al 15/1 possiamo vedere che non abbiamo nulla quindi amount = 0 e numero giorni 15
- dal 16/1 al 1/2 abbiamo 7 milioni di outflow e quindi debiti, 17 giorni 
- dal 2/2 al 9/3 abbiamo  i 7 milioni di debito precedente e i 9 di inflow quindi in totale un credito di 2 milioni e sono 36 giorni
- e così via 


In modo grafico possiamo rappresentare la tabella di prima:
![[Pasted image 20240306140908.png]]


A questo punto ci calcoliamo ZT ovvero la componente fissa (giorni dell'anno / tasso interesse):
![[Pasted image 20240306141033.png]]


Ora abbiamo tutti i valori necessari e possiamo calcolare quale banca è meglio:
![[Pasted image 20240306141229.png]]

I credit commission vengono calcolati dalla prima banca con il monthly peak demand mentre dalla seconda con il credit line, quindi per gennaio:
- la prima banca calcola 7 milioni * 4% (ha segno negativo perchè è un costo)  questo lo faccio guardando il grafico 
- la seconda banca calcola sempre 10 milioni di line * 1% * totale di giorni  che è 90 in questo caso 




# Long term credit financing
The most important components of credit contracts are: 
- amount paid out (nominal value, face value) and the debt service payments 
- repayment structure 
- interest rate 
- duration (long term 5-10 years )
- collateralization, often mortgage-backed, notorial documentation of the mortgage and inclusion in the land registry 



## Amount paid out and debt service payments
==The **Nominal face amount** is the nominal value of the credit==, that is a benchmark for other components of the contract ( e. g. interest or some external service costs ESC)

==The **amount paid out** is the amputn actually paid out to the borrower== (ovvero quello che ho ritornato):
![[Pasted image 20240306142835.png]]



==The **debt service payments** is the amount the borrower has to pay in period t:==
![[Pasted image 20240306143202.png]]

Tutte le + sono cost components



## Repayment structures 
We have some different repayment structures, for example we have:

### Repayment at the end
In this case the total face value is repaid at the end of the period:
![[Pasted image 20240306143444.png]]
Gli interessi qui vengono pagati ogni anno sul face value, perchè non ritorniamo soldi ma solo gli interessi stessi.


### Repayment as Installment repayment

In this case the face value is repaid constantly.
![[Pasted image 20240306143652.png]]

### Repayment as Annuity payments 
In this case the face value is repaid with a costant cost.
![[Pasted image 20240306143850.png]]
All'inizio gli interessi saranno più alti del repayment del debito e così via...

![[Pasted image 20240306143949.png]]
**==L'annuity payment è il costo fisso annuale da pagare, dove:==**
- i è il tasso di interesse
- n sono gli anni totali
- k0 è il face value iniziale
- q sarebbe 1 + tasso interesse

![[Pasted image 20240306144236.png]]

- bilancio iniziale = 100
- interesse = bilancio iniziale * tasso interesse 
- repayment sarebbe annuity payment - interesse
- nuovo bilancio = bilancio iniziale - annuity payment


![[Pasted image 20240306144435.png]]





## Cost of debt calculation

The simplest used approach is the **yield to maturity** of the firm's debt.

- Step 1: ==we need to generate the **payment flow** of a firm's debt positions==

Financing costs include all expenses (cash outflows) a company has to accept to get external financing means (tutti i costi per ottenere aiuti esterni)

The main elements of these costs are:
- costs for using credit funds (interests)
- cost for acquiring capital 
	- one time external costs ESC example provision of collateral 
	- regular external service costs (ESC) example account management fee


If a market value is available then we have that (issued bond):
![[Pasted image 20240306145751.png]]
If a market value is not available then we have that (bank loan):
![[Pasted image 20240306150604.png]]
- 100 è il face value



## Commercial method: approximative yield to maturity calculation
Its calculation is based on this formula:
![[Pasted image 20240306151103.png]]

Facciamo la somma totale dei costi in un certo periodo ( la divisione per mean time to maturity serve a questo) e dividiamo tutto per l'available funds al tempo 0


The available funds at time 0 is equal to:
![[Pasted image 20240306151426.png]]
The mean time to maturity is calculated as:
![[Pasted image 20240306151448.png]]

## EXAMPLE (HOME EXAM)

![[Pasted image 20240306151527.png]]


1. ==First of all we need to represent the repayment structure ==
	1. ![[Pasted image 20240306151809.png]]
	2. In this case we have 2 annual installments
2. cash value calculation 
	1. ![[Pasted image 20240306151939.png]]
	2. al tempo 0 abbiamo 20 milioni di nominal value, il disago e la collaternal inclusion
	3. la collateral exclusion si mette al tempo 2 perchè è per la chiusura del debito
	4. l'agio la applico sul repaiment quindi qui su 10 milioni a tempo 1 e 10 milioni a tempo 2
	5. account management charge la applico a tempo 1 su 20 milioni, al tempo 2 su 20 milioni - 10 pagati quindi su 10 soltanto
	6. gli interessi liapplico a tempo 1 su 20 milioni, al tempo 2 su 20 milioni - 10 pagati quindi su 10 soltanto
3. Cost of capital approximative
	1. calcolo mean time to maturity ![[Pasted image 20240306152608.png]]
	2. quindi 0 periodi senza repyament + (2 periodi repyament +1)/2
	3. calcolo cost of capital ![[Pasted image 20240306152625.png]]
		1. ![[Pasted image 20240306152640.png]]
		2. i repayment non sono costi quindi non vanno messi 
4. calcolo cost of capital exact
	1. parto dai cashflows ![[Pasted image 20240306152947.png]]
	2. sommo t1 e t2 e ottenfo il PV(C1, C2) 
	3. ![[Pasted image 20240306153419.png]]
	4. con il solver di excel (sotto data) scelgo come target la cella che contiene PV(C1,C2)
		1. inserisco come valore il cashflow a tempo 0 con segno negativo 
		2. scelgo come cella che varia quella dello yield to maturity![[Pasted image 20240306153710.png]]
		3. faccio solve e ottengo il valore

