3A-
WITH MonthlyRevenue AS (
    SELECT
        RT.unified_customer_id,
        RT.customer_name,
        COALESCE(SUM(RT.tariff_price), 0) AS Monthly_Mobile_Revenue,
        COALESCE(SUM(RN.offer_price), 0) AS Monthly_Fixed_Revenue,
        COALESCE(SUM(RM.Product_Price), 0) AS Monthly_Market_Revenue
    FROM
        VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT
    LEFT JOIN
        VF_USER_READONLY.RN_OFFER_HISTORY RN ON RT.unified_customer_id = RN.unified_customer_id
    LEFT JOIN
        VF_USER_READONLY.RM_ORDER_HISTORY RM ON RT.GSM_No = RM.GSM_No
    WHERE
        EXTRACT(MONTH FROM RT.subscriptionn_start_date) = 9
        AND EXTRACT(YEAR FROM RT.subscriptionn_start_date) = 2023
        AND RT.unified_customer_id IN (
            SELECT DISTINCT
                RT.unified_customer_id
            FROM
                VF_USER_READONLY.RM_ORDER_HISTORY RM
            JOIN
                VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT ON RM.GSM_No = RT.GSM_No
            WHERE
                EXTRACT(MONTH FROM RM.Order_Date) = 9
                AND EXTRACT(YEAR FROM RM.Order_Date) = 2023
        )
    GROUP BY
        RT.unified_customer_id, RT.customer_name
)

SELECT
    unified_customer_id,
    customer_name,
    Monthly_Mobile_Revenue,
    Monthly_Fixed_Revenue,
    Monthly_Market_Revenue,
    (Monthly_Mobile_Revenue + Monthly_Fixed_Revenue + Monthly_Market_Revenue) AS Monthly_Total_Coverage_Revenue
FROM
    MonthlyRevenue;

3B-
WITH MonthlyRevenue AS (
    SELECT
        RT.unified_customer_id,
        RT.customer_name,
        COALESCE(SUM(RT.tariff_price), 0) AS Monthly_Mobile_Revenue,
        COALESCE(SUM(RN.offer_price), 0) AS Monthly_Fixed_Revenue,
        COALESCE(SUM(RM.Product_Price), 0) AS Monthly_Market_Revenue
    FROM
        VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT
    LEFT JOIN
        VF_USER_READONLY.RN_OFFER_HISTORY RN ON RT.unified_customer_id = RN.unified_customer_id
    LEFT JOIN
        VF_USER_READONLY.RM_ORDER_HISTORY RM ON RT.GSM_No = RM.GSM_No
    WHERE
        EXTRACT(MONTH FROM RT.subscriptionn_start_date) = 9
        AND EXTRACT(YEAR FROM RT.subscriptionn_start_date) = 2023
        AND RT.unified_customer_id IN (
            SELECT DISTINCT
                RT.unified_customer_id
            FROM
                VF_USER_READONLY.RM_ORDER_HISTORY RM
            JOIN
                VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT ON RM.GSM_No = RT.GSM_No
            WHERE
                EXTRACT(MONTH FROM RM.Order_Date) = 9
                AND EXTRACT(YEAR FROM RM.Order_Date) = 2023
        )
    GROUP BY
        RT.unified_customer_id, RT.customer_name
)

SELECT
    unified_customer_id,
    customer_name,
    Monthly_Mobile_Revenue,
    Monthly_Fixed_Revenue,
    Monthly_Market_Revenue,
    (Monthly_Mobile_Revenue + Monthly_Fixed_Revenue + Monthly_Market_Revenue) AS Monthly_Total_Coverage_Revenue,
    CASE
        WHEN (Monthly_Mobile_Revenue + Monthly_Fixed_Revenue + Monthly_Market_Revenue) <= 5000 THEN 'Bronze RedRoyal'
        WHEN (Monthly_Mobile_Revenue + Monthly_Fixed_Revenue + Monthly_Market_Revenue) > 5000 AND (Monthly_Mobile_Revenue + Monthly_Fixed_Revenue + Monthly_Market_Revenue) <= 10000 THEN 'Silver RedLoyal'
        ELSE 'Gold RedRoyal'
    END AS RedRoyal_Segment
FROM
    MonthlyRevenue;

4A-
WITH SegmentRevenue AS (
    SELECT
        RT.unified_customer_id,
        RT.customer_name,
        COALESCE(SUM(RT.tariff_price), 0) AS Monthly_Mobile_Revenue,
        COALESCE(SUM(RN.offer_price), 0) AS Monthly_Fixed_Revenue,
        COALESCE(SUM(RM.Product_Price), 0) AS Monthly_Market_Revenue,
        (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) AS Monthly_Total_Coverage_Revenue,
        CASE
            WHEN (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) <= 5000 THEN 'Bronze RedRoyal'
            WHEN (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) > 5000 AND (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) <= 10000 THEN 'Silver RedLoyal'
            ELSE 'Gold RedRoyal'
        END AS RedRoyal_Segment
    FROM
        VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT
    LEFT JOIN
        VF_USER_READONLY.RN_OFFER_HISTORY RN ON RT.unified_customer_id = RN.unified_customer_id
    LEFT JOIN
        VF_USER_READONLY.RM_ORDER_HISTORY RM ON RT.GSM_No = RM.GSM_No
    WHERE
        EXTRACT(MONTH FROM RT.subscriptionn_start_date) = 9
        AND EXTRACT(YEAR FROM RT.subscriptionn_start_date) = 2023
        AND RT.unified_customer_id IN (
            SELECT DISTINCT
                RT.unified_customer_id
            FROM
                VF_USER_READONLY.RM_ORDER_HISTORY RM
            JOIN
                VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT ON RM.GSM_No = RT.GSM_No
            WHERE
                EXTRACT(MONTH FROM RM.Order_Date) = 9
                AND EXTRACT(YEAR FROM RM.Order_Date) = 2023
        )
    GROUP BY
        RT.unified_customer_id, RT.customer_name
)
, RankedCustomers AS (
    SELECT
        unified_customer_id,
        customer_name,
        Monthly_Total_Coverage_Revenue,
        ROW_NUMBER() OVER (PARTITION BY RedRoyal_Segment ORDER BY Monthly_Total_Coverage_Revenue DESC) AS rank
    FROM
        SegmentRevenue
)

SELECT
    unified_customer_id,
    customer_name,
    Monthly_Total_Coverage_Revenue,
    RedRoyal_Segment
FROM
    RankedCustomers
WHERE
    rank <= 3;

4B-
WITH SegmentRevenue AS (
    SELECT
        RT.unified_customer_id,
        RT.customer_name,
        COALESCE(SUM(RT.tariff_price), 0) AS Monthly_Mobile_Revenue,
        COALESCE(SUM(RN.offer_price), 0) AS Monthly_Fixed_Revenue,
        COALESCE(SUM(RM.Product_Price), 0) AS Monthly_Market_Revenue,
        (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) AS Monthly_Total_Coverage_Revenue,
        CASE
            WHEN (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) <= 5000 THEN 'Bronze RedRoyal'
            WHEN (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) > 5000 AND (COALESCE(SUM(RT.tariff_price), 0) + COALESCE(SUM(RN.offer_price), 0) + COALESCE(SUM(RM.Product_Price), 0)) <= 10000 THEN 'Silver RedLoyal'
            ELSE 'Gold RedRoyal'
        END AS RedRoyal_Segment
    FROM
        VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT
    LEFT JOIN
        VF_USER_READONLY.RN_OFFER_HISTORY RN ON RT.unified_customer_id = RN.unified_customer_id
    LEFT JOIN
        VF_USER_READONLY.RM_ORDER_HISTORY RM ON RT.GSM_No = RM.GSM_No
    WHERE
        EXTRACT(MONTH FROM RT.subscriptionn_start_date) = 9
        AND EXTRACT(YEAR FROM RT.subscriptionn_start_date) = 2023
        AND RT.unified_customer_id IN (
            SELECT DISTINCT
                RT.unified_customer_id
            FROM
                VF_USER_READONLY.RM_ORDER_HISTORY RM
            JOIN
                VF_USER_READONLY.RT_SUBSCRIPTION_HISTORY RT ON RM.GSM_No = RT.GSM_No
            WHERE
                EXTRACT(MONTH FROM RM.Order_Date) = 9
                AND EXTRACT(YEAR FROM RM.Order_Date) = 2023
        )
    GROUP BY
        RT.unified_customer_id, RT.customer_name
),
RankedCustomers AS (
    SELECT
        unified_customer_id,
        customer_name,
        Monthly_Total_Coverage_Revenue,
        ROW_NUMBER() OVER (PARTITION BY RedRoyal_Segment ORDER BY Monthly_Total_Coverage_Revenue DESC) AS rank
    FROM
        SegmentRevenue
)
, Top3TotalRevenue AS (
    SELECT
        RedRoyal_Segment,
        SUM(Monthly_Total_Coverage_Revenue) AS TotalRevenue
    FROM
        RankedCustomers
    WHERE
        rank <= 3
    GROUP BY
        RedRoyal_Segment
)

SELECT
    RedRoyal_Segment,
    TotalRevenue,
    (TotalRevenue / (SELECT SUM(TotalRevenue) FROM Top3TotalRevenue)) * 100 AS PercentageOfTotalRevenue
FROM
    Top3TotalRevenue;
