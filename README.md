# shopifydsinternchallenge

Question 1:
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('2019 Winter Data Science Intern Challenge Data Set - Sheet1.csv')

#At the first glance at the data, I realized there are few large numbers in the order amount column. That's a flag.


#AOV = total avenue / # of orders taken

#the target feature is the order_amount

print(df['order_amount'].describe())

#std = 41282.54, this is a very large standard deviation, which means our data is very spread out
#-> outliers expected

#use boxplot to display data distribution

df.boxplot(column='order_amount')

plt.title('Order Amount Boxplot')

plt.show()
#Boxplot clearly shows extreme outliers. Since mean value (AOV) is sensitive to outliers, an alternative method
#is to use median value. However, if outliers can be removed, the AOV value will be more realistic.

#1) use median value

print(df['order_amount'].median())

#Another metric we can look into is the median, which will result in $284 for each order.

#2) remove outlier
#Assuming the high level goal is to understand TYPICAL consumers buying behavior, we can safely remove outliers

Q1 = df.quantile(0.25)

Q3 = df.quantile(0.75)

IQR = Q3 - Q1

df = df[~((df < (Q1 - 1.5 * IQR)) |(df > (Q3 + 1.5 * IQR))).any(axis=1)]

print(df['order_amount'].describe())

df.boxplot(column='order_amount')

plt.title('Order Amount Boxplot (Removed Extreme Outliers)')

plt.show()

#Removed 141 outliers, mean is $293.72


Question 2: 
a) 54

SELECT COUNT(OrderID) AS total_orders_by_speedy_express
FROM [Orders] JOIN [Shippers]
	ON Orders.ShipperID = Shippers.ShipperID
WHERE ShipperName = 'Speedy Express'


b) Peakcock

SELECT Employees.LastName, COUNT(*) AS total_orders
FROM [Employees] JOIN [Orders]
	ON Employees.EmployeeID = Orders.EmployeeID
GROUP BY Employees.EmployeeID
ORDER BY COUNT(*) DESC

First result shows peacock, 40 total orders 

Thought Process: Simple solution won’t consider edge cases. What about two employees with the same most orders amount?

Solution when two or more employees with the highest number of orders: 

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


c) Gorgonzola Telino

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

Thought process:
Customers in Germany (CustomerID, Country) 
Order (OrderID, CustomerID)
OrderDetails(OrderID, ProductID)
Products (productID, ProductName)

First, join these four tables, 
filter out the rows with country = Germany
group by product_id
order by how many counts for every product
=> output product id and product name
=> if more than two products ordered the most by customers in Germany, we can use similar approach like question 2b 



