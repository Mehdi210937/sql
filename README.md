USE classicmodels;

-- Afficher toutes les tables de base
SELECT * FROM customers;
SELECT * FROM employees;
SELECT * FROM offices;
SELECT * FROM orderdetails;
SELECT * FROM orders;
SELECT * FROM payments;
SELECT DISTINCT COUNT(customerNumber) FROM payments;
SELECT * FROM productlines;
SELECT * FROM products;

---------------------------------------------------------
-- Chiffre d'affaires total et par catégorie (via payments)
---------------------------------------------------------
SELECT 
    C.customerNumber, 
    SUM(P.amount) AS total_amount
FROM payments P  
JOIN customers C ON P.customerNumber = C.customerNumber  
GROUP BY C.customerNumber  
LIMIT 1000;

---------------------------------------------------------
-- Entraînement pour le devoir : chiffre d'affaires par commande
---------------------------------------------------------
SELECT 
    o.orderNumber AS numero_commande,
    SUM(od.quantityOrdered * od.priceEach) AS chiffre_affaires
FROM orders o
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY o.orderNumber
ORDER BY chiffre_affaires DESC;

---------------------------------------------------------
-- Ventes totales par client
---------------------------------------------------------
SELECT 
    c.customerName AS client,
    SUM(od.quantityOrdered * od.priceEach) AS ventes_totales
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY c.customerName
ORDER BY ventes_totales DESC;

---------------------------------------------------------
-- Les produits les plus rentables (Top 10 par chiffre d'affaires)
---------------------------------------------------------
SELECT 
    p.productName AS produit,
    SUM(od.quantityOrdered * od.priceEach) AS chiffre_affaires
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
GROUP BY p.productName
ORDER BY chiffre_affaires DESC
LIMIT 10;

---------------------------------------------------------
-- Les produits en stock par catégorie
---------------------------------------------------------
SELECT 
    pl.productLine AS categorie,
    SUM(p.quantityInStock) AS total_stock
FROM productlines pl
JOIN products p ON pl.productLine = p.productLine
GROUP BY pl.productLine
ORDER BY total_stock DESC;

---------------------------------------------------------
-- Évolution chronologique des ventes
---------------------------------------------------------
SELECT 
    o.orderDate AS date_commande,
    SUM(od.quantityOrdered * od.priceEach) AS total_ventes
FROM orders o
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY o.orderDate
ORDER BY o.orderDate;

---------------------------------------------------------
-- Top clients en termes de chiffre d'affaires (Top 10)
---------------------------------------------------------
SELECT 
    c.customerName AS client,
    SUM(od.quantityOrdered * od.priceEach) AS total_chiffre_affaires
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY c.customerName
ORDER BY total_chiffre_affaires DESC
LIMIT 10;

---------------------------------------------------------
-- Performance des bureaux
---------------------------------------------------------
SELECT 
    o.city AS bureau,
    SUM(od.quantityOrdered * od.priceEach) AS total_chiffre_affaires
FROM offices o
JOIN employees e ON o.officeCode = e.officeCode
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
JOIN orders o2 ON c.customerNumber = o2.customerNumber
JOIN orderdetails od ON o2.orderNumber = od.orderNumber
GROUP BY o.city
ORDER BY total_chiffre_affaires DESC;

---------------------------------------------------------
-- Clients ayant atteint leur limite de crédit
---------------------------------------------------------
SELECT 
    c.customerName AS client,
    c.creditLimit AS limite_credit,
    SUM(od.quantityOrdered * od.priceEach) AS total_achats
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY c.customerName, c.creditLimit
HAVING total_achats >= c.creditLimit * 0.9
ORDER BY total_achats DESC;

---------------------------------------------------------
-- Ventes par régions géographiques
---------------------------------------------------------
SELECT 
    c.country AS pays,
    SUM(od.quantityOrdered * od.priceEach) AS total_ventes
FROM orders o
JOIN customers c ON o.customerNumber = c.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY c.country
ORDER BY total_ventes DESC;

---------------------------------------------------------
-- Ventes par continents
---------------------------------------------------------
SELECT 
    CASE 
        WHEN country IN ('USA', 'Canada') THEN 'Amérique du Nord'
        WHEN country IN ('France', 'Spain', 'Italy', 'Germany', 'UK', 'Finland', 'Norway', 'Austria', 'Belgium', 'Denmark', 'Switzerland', 'Sweden', 'Ireland') THEN 'Europe'
        WHEN country IN ('Australia', 'New Zealand') THEN 'Océanie'
        WHEN country IN ('Singapore', 'Philippines', 'Japan', 'Hong Kong') THEN 'Asie'
        ELSE 'Autres'
    END AS continent,
    SUM(total_ventes) AS ventes_totales
FROM (
    SELECT 
        c.country AS country,
        SUM(od.quantityOrdered * od.priceEach) AS total_ventes
    FROM orders o
    JOIN customers c ON o.customerNumber = c.customerNumber
    JOIN orderdetails od ON o.orderNumber = od.orderNumber
    GROUP BY c.country
) AS ventes_par_pays
GROUP BY continent
ORDER BY ventes_totales DESC;

---------------------------------------------------------
-- Rentabilité des fournisseurs
---------------------------------------------------------
SELECT 
    p.productVendor AS fournisseur,
    SUM(od.quantityOrdered * od.priceEach) AS chiffre_affaires,
    SUM(od.quantityOrdered * p.buyPrice) AS cout_total,
    SUM(od.quantityOrdered * od.priceEach) - SUM(od.quantityOrdered * p.buyPrice) AS profit
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
GROUP BY p.productVendor
ORDER BY profit DESC;

---------------------------------------------------------
-- Les produits les plus rentables avec marge bénéficiaire
---------------------------------------------------------
SELECT 
    p.productName AS produit,
    SUM(od.quantityOrdered * od.priceEach) AS chiffre_affaires,
    SUM(od.quantityOrdered * p.buyPrice) AS cout_total,
    SUM(od.quantityOrdered * od.priceEach) - SUM(od.quantityOrdered * p.buyPrice) AS marge_beneficiaire
FROM products p
JOIN orderdetails od ON p.productCode = od.productCode
GROUP BY p.productName
ORDER BY marge_beneficiaire DESC;

---------------------------------------------------------
-- Analyse des marges bénéficiaires globales
---------------------------------------------------------
SELECT 
    SUM(od.quantityOrdered * od.priceEach) AS chiffre_affaires,
    SUM(od.quantityOrdered * p.buyPrice) AS couts_totaux,
    SUM(od.quantityOrdered * od.priceEach) - SUM(od.quantityOrdered * p.buyPrice) AS marge_totale
FROM orderdetails od
JOIN products p ON od.productCode = p.productCode;

---------------------------------------------------------
-- Analyse des produits invendus
---------------------------------------------------------
SELECT 
    p.productName AS produit,
    p.quantityInStock AS stock_disponible
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode
WHERE od.orderNumber IS NULL;

---------------------------------------------------------
-- 1. Total des ventes par pays
---------------------------------------------------------
SELECT 
    customers.country, 
    SUM(payments.amount) AS total_sales
FROM customers
JOIN payments ON customers.customerNumber = payments.customerNumber
GROUP BY customers.country
ORDER BY total_sales DESC;

---------------------------------------------------------
-- 2. Total des ventes par ville
---------------------------------------------------------
SELECT 
    customers.city, 
    customers.country, 
    SUM(payments.amount) AS total_sales
FROM customers
JOIN payments ON customers.customerNumber = payments.customerNumber
GROUP BY customers.city, customers.country
ORDER BY total_sales DESC;

---------------------------------------------------------
-- 3. Valeur moyenne des commandes par client
---------------------------------------------------------
SELECT 
    customers.customerName, 
    AVG(payments.amount) AS average_order_value
FROM customers
JOIN payments ON customers.customerNumber = payments.customerNumber
GROUP BY customers.customerName
ORDER BY average_order_value DESC;

---------------------------------------------------------
-- 4. Nombre de clients distincts
---------------------------------------------------------
SELECT COUNT(DISTINCT customerNumber) AS total_customers
FROM customers;

---------------------------------------------------------
-- 5. Taux de rétention des clients (basé sur les commandes)
---------------------------------------------------------
SELECT 
    YEAR(orderDate) AS year, 
    COUNT(DISTINCT customerNumber) AS active_customers
FROM orders
GROUP BY year
ORDER BY year;

---------------------------------------------------------
-- 6. Modes de paiement les plus utilisés
---------------------------------------------------------
SELECT 
    paymentMethod, 
    COUNT(*) AS payment_count
FROM payments
GROUP BY paymentMethod
ORDER BY payment_count DESC;

---------------------------------------------------------
-- 7. Top 10 des produits les plus vendus (quantité commandée)
---------------------------------------------------------
SELECT 
    products.productName, 
    SUM(orderdetails.quantityOrdered) AS total_quantity_sold
FROM orderdetails
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY products.productName
ORDER BY total_quantity_sold DESC
LIMIT 10;

---------------------------------------------------------
-- 8. Top 10 des produits générant le plus de revenus
---------------------------------------------------------
SELECT 
    products.productName, 
    SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total_revenue
FROM orderdetails
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY products.productName
ORDER BY total_revenue DESC
LIMIT 10;

---------------------------------------------------------
-- 9. Analyse des produits les moins performants (Top 10)
---------------------------------------------------------
SELECT 
    products.productName, 
    SUM(orderdetails.quantityOrdered) AS total_quantity_sold
FROM orderdetails
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY products.productName
ORDER BY total_quantity_sold ASC
LIMIT 10;

---------------------------------------------------------
-- 10. Répartition des produits vendus par type de commande
---------------------------------------------------------
SELECT 
    orderNumber, 
    SUM(quantityOrdered) AS total_quantity,
    CASE 
        WHEN SUM(quantityOrdered) < 25 THEN 'Petite Commande'
        WHEN SUM(quantityOrdered) BETWEEN 26 AND 60 THEN 'Commande Moyenne'
        ELSE 'Grande Commande'
    END AS order_type
FROM orderdetails
GROUP BY orderNumber;

---------------------------------------------------------
-- 11. Comparaison des marges bénéficiaires des catégories de produits
---------------------------------------------------------
SELECT 
    productLine, 
    SUM(orderdetails.priceEach * orderdetails.quantityOrdered) AS total_revenue,
    SUM(products.buyPrice * orderdetails.quantityOrdered) AS total_cost,
    (SUM(orderdetails.priceEach * orderdetails.quantityOrdered) - SUM(products.buyPrice * orderdetails.quantityOrdered)) AS profit_margin
FROM orderdetails
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY productLine
ORDER BY profit_margin DESC;

---------------------------------------------------------
-- 12. Analyse de la saisonnalité des commandes
---------------------------------------------------------
SELECT 
    YEAR(orders.orderDate) AS year, 
    MONTH(orders.orderDate) AS month, 
    SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total_sales
FROM orderdetails
JOIN orders ON orderdetails.orderNumber = orders.orderNumber
GROUP BY year, month
ORDER BY year, month;

---------------------------------------------------------
-- 13. Comparaison des ventes annuelles par catégorie de produits
---------------------------------------------------------
SELECT 
    YEAR(orders.orderDate) AS year,
    products.productLine, 
    SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total_sales
FROM orderdetails
JOIN orders ON orderdetails.orderNumber = orders.orderNumber
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY year, products.productLine
ORDER BY year, total_sales DESC;

---------------------------------------------------------
-- 14. Nombre de commandes par client (Top 20)
---------------------------------------------------------
SELECT 
    customers.customerName, 
    COUNT(orders.orderNumber) AS total_orders
FROM customers
JOIN orders ON customers.customerNumber = orders.customerNumber
GROUP BY customers.customerName
ORDER BY total_orders DESC
LIMIT 20;

---------------------------------------------------------
-- 15. Répartition des ventes par catégorie de produits et régions
---------------------------------------------------------
SELECT 
    customers.country, 
    products.productLine, 
    SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total_sales
FROM customers
JOIN orders ON customers.customerNumber = orders.customerNumber
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY customers.country, products.productLine
ORDER BY total_sales DESC;

---------------------------------------------------------
-- 16. Agences qui performent par rapport aux autres (basé sur les ventes)
---------------------------------------------------------
SELECT 
    offices.city AS office_city, 
    SUM(payments.amount) AS total_sales
FROM employees
JOIN customers ON employees.employeeNumber = customers.salesRepEmployeeNumber
JOIN payments ON customers.customerNumber = payments.customerNumber
JOIN offices ON employees.officeCode = offices.officeCode
GROUP BY offices.city
ORDER BY total_sales DESC;

---------------------------------------------------------
-- 17. Ventes totales par mois pour identifier les pics de ventes
---------------------------------------------------------
SELECT 
    DATE_FORMAT(orderDate, '%Y-%m') AS month, 
    SUM(orderdetails.quantityOrdered * orderdetails.priceEach) AS total_sales
FROM orders
JOIN orderdetails ON orders.orderNumber = orderdetails.orderNumber
GROUP BY month
ORDER BY month;

---------------------------------------------------------
-- 18. Clients récurrents basés sur le nom et prénom
---------------------------------------------------------
SELECT 
    CONCAT(customers.contactFirstName, ' ', customers.contactLastName) AS customer_name, 
    COUNT(orders.orderNumber) AS total_orders
FROM customers
LEFT JOIN orders ON customers.customerNumber = orders.customerNumber
GROUP BY customer_name
HAVING total_orders > 1
ORDER BY total_orders DESC;

---------------------------------------------------------
-- 19. Produits générant le plus de revenu (profit brut)
---------------------------------------------------------
SELECT 
    products.productName, 
    SUM(orderdetails.quantityOrdered * (orderdetails.priceEach - products.buyPrice)) AS total_profit
FROM orderdetails
JOIN products ON orderdetails.productCode = products.productCode
GROUP BY products.productName
ORDER BY total_profit DESC
LIMIT 10;

---------------------------------------------------------
-- 20. Analyse des villes sous-exploitées (moins de commandes)
---------------------------------------------------------
SELECT 
    customers.city, 
    COUNT(orders.orderNumber) AS total_orders
FROM customers
LEFT JOIN orders ON customers.customerNumber = orders.customerNumber
GROUP BY customers.city
ORDER BY total_orders ASC
LIMIT 10;

SELECT 
    ROUND(
        (SUM(CASE WHEN shippedDate <= requiredDate THEN 1 ELSE 0 END) / COUNT(*)) * 100, 
        2
    ) AS on_time_delivery_rate
FROM orders
WHERE shippedDate IS NOT NULL;

SELECT 
    e.employeeNumber, 
    CONCAT(e.firstName, ' ', e.lastName) AS sales_rep,
    SUM(od.quantityOrdered * od.priceEach) AS total_sales
FROM employees e
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
JOIN orders o ON c.customerNumber = o.customerNumber
JOIN orderdetails od ON o.orderNumber = od.orderNumber
GROUP BY e.employeeNumber
ORDER BY total_sales DESC;
