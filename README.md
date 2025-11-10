# sql_syntax_kf_analisa
CREATE TABLE kf_tabel_analisa AS
WITH CalculatedData AS (
    SELECT
        t.transaction_id,
        t.date,
        t.branch_id,
        t.customer_name,
        t.product_id,
        t.discount_percentage,
        t.rating AS rating_transaksi,
        kc.branch_name,
        kc.kota,
        kc.provinsi,
        kc.rating AS rating_cabang,
        kp.product_name,
        kp.price AS actual_price,

# Hitung persentase_gross_laba
        CASE
            WHEN kp.price <= 50000 THEN 0.10
            WHEN kp.price > 50000 AND kp.price <= 100000 THEN 0.15
            WHEN kp.price > 100000 AND kp.price <= 300000 THEN 0.20
            WHEN kp.price > 300000 AND kp.price <= 500000 THEN 0.25
            WHEN kp.price > 500000 THEN 0.30
            ELSE 0.00
        END AS persentase_gross_laba,

        -- 2. Hitung nett_sales
        (kp.price - (kp.price * t.discount_percentage)) AS nett_sales

    FROM 
        kf_final_transaction t
    LEFT JOIN 
        kf_kantor_cabang kc ON t.branch_id = kc.branch_id
    LEFT JOIN 
        kf_product kp ON t.product_id = kp.product_id
)

SELECT
    transaction_id,
    date,
    branch_id,
    branch_name,
    kota,
    provinsi,
    rating_cabang,
    customer_name,
    product_id,
    product_name,
    actual_price,
    discount_percentage,
    persentase_gross_laba,
    nett_sales,
    
# Hitung nett_profit menggunakan kolom dari CTE
    (nett_sales * persentase_gross_laba) AS nett_profit,
    
    rating_transaksi

FROM 
    CalculatedData;
