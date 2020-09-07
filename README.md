# shopifydsinternchallenge

Question 1:

```
import pandas as pd
import matplotlib.pyplot as plt
df = pd.read_csv('2019 Winter Data Science Intern Challenge Data Set - Sheet1.csv')
```

#At the first glance at the data, I realized there are few large numbers in the order amount column. That's a flag.

#AOV = total avenue / # of orders taken

#the target feature is the order_amount
```
print(df['order_amount'].describe())
```

#std = 41282.54, this is a very large standard deviation, which means our data is very spread out
#-> outliers expected

#use boxplot to display data distribution
```
df.boxplot(column='order_amount')

plt.title('Order Amount Boxplot')

plt.show()
```

![Image of Boxplot2 (with outliers)](https://github.com/candichen66/shopifydsinternchallenge/blob/master/boxplot1.png)


#Boxplot clearly shows extreme outliers. Since mean value (AOV) is sensitive to outliers, an alternative method
is to use median value. However, if outliers can be removed, the AOV value will be more realistic.

S1) use median value

```
print(df['order_amount'].median())
```

#Another metric we can look into is the median, which will result in $284 for each order.

S2) remove outlier
#Assuming the high level goal is to understand TYPICAL consumers buying behavior, we can safely remove outliers

```
unique_order_amount = df['order_amount'].sort_values(ascending=False).unique()

print(unique_order_amount)
```

#extreme values: 704000 154350 102900  77175  51450  25725
#However, to avoid flasely deleting all extreme values, I look through their total_items, only first one (704000) indicates atypical purchase behavior.

#delete order_amount == 704000

```
df = df[df['order_amount'] != 704000]

print(df['order_amount'].describe())
```

#display boxplot 

```
df.boxplot(column='order_amount')
plt.title('Order Amount Boxplot (Removed Outliers (704000))')
plt.show()
```

#Removed 17 outlier, mean is $754.09

#Although the graph still shows many outliers, the mean value decreases significantly.

![Image of Boxplot2 (with outliers)](https://github.com/candichen66/shopifydsinternchallenge/blob/master/boxplot2.png)


Question 2: 

a) 54
```
SELECT COUNT(OrderID) AS total_orders_by_speedy_express

FROM [Orders] JOIN [Shippers]

	ON Orders.ShipperID = Shippers.ShipperID
	
WHERE ShipperName = 'Speedy Express'
```

b) Peakcock
```
SELECT Employees.LastName, COUNT(*) AS total_orders

FROM [Employees] JOIN [Orders]

	ON Employees.EmployeeID = Orders.EmployeeID
	
GROUP BY Employees.EmployeeID

ORDER BY COUNT(*) DESC
```

First result shows peacock, 40 total orders 

Thought Process: Simple solution won’t consider edge cases. What about two employees with the same most orders amount?
Solution when two or more employees with the highest number of orders: 

```
SELECT Employees.LastName

FROM [Employees] JOIN [Orders]

	ON Employees.EmployeeID = Orders.EmployeeID
	
GROUP BY Employees.EmployeeID

HAVING COUNT(*) IN 

(SELECT COUNT(*) AS most_orders

FROM [Employees] JOIN [Orders]

	ON Employees.EmployeeID = Orders.EmployeeID
	
GROUP BY Employees.EmployeeID

ORDER BY COUNT(*) DESC

LIMIT 1)
```

c) Gorgonzola Telino

```
SELECT Products.ProductName, COUNT(Products.ProductID) AS total_ordered_amount

FROM [Orders] JOIN [Customers] 

	ON Orders.CustomerID = Customers.CustomerID
	
    JOIN [OrderDetails] 
    
    ON OrderDetails.OrderID = Orders.OrderID
    
    JOIN [Products] 
    
    ON Products.ProductID = OrderDetails.ProductID
    
WHERE Customers.Country = 'Germany'

GROUP BY Products.ProductID

ORDER BY COUNT(Products.ProductID) DESC
```

Thought process:
Customers in Germany (CustomerID, Country) 
Order (OrderID, CustomerID)
OrderDetails(OrderID, ProductID)
Products (productID, ProductName)

First, join these four tables, 
filter out the rows with country = Germany
group by product_id
order by how many counts for every product

=> output product name and total ordered amount

=> if more than two products ordered the most by customers in Germany, we can use similar approach like question 2b 



