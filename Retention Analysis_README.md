# 프로젝트 01. 리텐션 분석: 이커머스의 마케팅 전략 수립을 위한 EDA

### 주제:  이커머스**의 마케팅 전략 수립을 위한 EDA**

- **본 프로젝트 중 본인은 리텐션분석을 맡았습니다.**
- **고객 세분화 및 리텐션 분석을 업무에서 사용해본 경험은 있으나, SQL활용하여 분석해본 경험이 없어, SQL코드 연습을 목적으로 진행하였습니다.**
- **프로젝트를 읽으실 때, 논리의 흐름보다는 코드와 시각화를 중심으로 평가 부탁드립니다.**

<aside>

목차

1. 서론
    1. 프로젝트 소개
    2. 데이터셋 소개 & EDA 접근 방법
2.  본론
    
    2-1. 리텐션 분석 
    
    2-2. RFM 분석
    
    2-3. 퍼널분석
    
3. 요약 
    1. **Appendix : 코드 및 기타 분석**
    - **프로젝트 코드**
        1. **고객 세분화 및 고객 등급 정의**
            - **최근 1년간, 3년간 주문액**
                
                ```sql
                #최근 1년간 주문액
                SELECT 
                    price_usd 
                FROM orders 
                WHERE created_at BETWEEN '2014-03-19' AND '2015-03-19';
                
                #최근 3년간 주문액
                SELECT 
                    price_usd 
                FROM orders 
                WHERE created_at BETWEEN '2012-03-19' AND '2015-03-19';
                ```
                
        2. **전체 고객 및 VIP 그룹의 리텐션 분석 - 유지율**
            - **전체 고객 및 VIP 그룹의 리텐션 분석(롤링리텐션)**
                
                ```sql
                #전체고객 rolling retention
                WITH firstlast_order AS (
                	SELECT DISTINCT user_id, min(created_at) AS first_ord_date, max(created_at) AS last_ord_date
                	FROM orders
                	GROUP BY user_id
                	),
                user_od AS(
                	SELECT o.user_id
                	, DATE_FORMAT(fo.first_ord_date,'%Y-%m-01')AS first_order_month
                	, DATE_FORMAT(fo.last_ord_date,'%Y-%m-01')AS last_order_month
                  FROM firstlast_order fo
                  LEFT JOIN orders o ON o.user_id = fo.user_id
                )
                SELECT first_order_month
                		, COUNT(DISTINCT user_id) AS month0
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 1 MONTH) <= last_order_month THEN user_id END) AS month1
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 2 MONTH) <= last_order_month THEN user_id END) AS month2
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 3 MONTH) <= last_order_month THEN user_id END) AS month3
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 4 MONTH) <= last_order_month THEN user_id END) AS month4
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 5 MONTH) <= last_order_month  THEN user_id END) AS month5
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 6 MONTH) <= last_order_month  THEN user_id END) AS month6
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 7 MONTH) <= last_order_month  THEN user_id END) AS month7
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 8 MONTH) <= last_order_month  THEN user_id END) AS month8
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 9 MONTH) <= last_order_month  THEN user_id  END) AS month9
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 10 MONTH) <= last_order_month  THEN user_id END) AS month10
                	     , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 11 MONTH) <= last_order_month THEN user_id END) AS month11
                FROM user_od
                GROUP BY first_order_month ORDER BY first_order_month ;
                
                #VIP rolling retention
                
                WITH Avg_price AS (
                    SELECT AVG(total_purchase) AS avg_price
                    FROM (
                        SELECT user_id, SUM(price_usd) AS total_purchase
                        FROM orders
                        GROUP BY user_id
                    ) AS user_purchase
                ),
                VIP AS (
                    SELECT o.user_id, SUM(o.price_usd) AS total_purchase, a.avg_price
                    FROM orders o
                    CROSS JOIN Avg_price a
                    GROUP BY o.user_id, a.avg_price
                    HAVING SUM(o.price_usd) >= 1.5 * a.avg_price
                ),
                firstlast_order AS (
                	SELECT DISTINCT user_id, min(created_at) AS first_ord_date, max(created_at) AS last_ord_date
                	FROM orders
                	WHERE user_id IN(SELECT user_id FROM VIP)
                	GROUP BY user_id
                	),
                user_od AS(
                	SELECT o.user_id
                	, DATE_FORMAT(fo.first_ord_date,'%Y-%m-01')AS first_order_month
                	, DATE_FORMAT(fo.last_ord_date,'%Y-%m-01')AS last_order_month
                  FROM firstlast_order fo
                  LEFT JOIN orders o ON o.user_id = fo.user_id
                )
                SELECT first_order_month
                			, COUNT(DISTINCT user_id) AS month0
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 1 MONTH) <= last_order_month THEN user_id END) AS month1
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 2 MONTH) <= last_order_month THEN user_id END) AS month2
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 3 MONTH) <= last_order_month THEN user_id END) AS month3
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 4 MONTH) <= last_order_month THEN user_id END) AS month4
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 5 MONTH) <= last_order_month  THEN user_id END) AS month5
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 6 MONTH) <= last_order_month  THEN user_id END) AS month6
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 7 MONTH) <= last_order_month  THEN user_id END) AS month7
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 8 MONTH) <= last_order_month  THEN user_id END) AS month8
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 9 MONTH) <= last_order_month  THEN user_id  END) AS month9
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 10 MONTH) <= last_order_month  THEN user_id END) AS month10
                	    , COUNT(DISTINCT CASE WHEN DATE_ADD(first_order_month, INTERVAL 11 MONTH) <= last_order_month THEN user_id END) AS month11
                FROM user_od
                GROUP BY first_order_month ORDER BY first_order_month ;
                
                ```
                
            - **전체 고객 및 VIP 그룹의 리텐션 분석(클래식리텐션)**
                
                ```sql
                # 유저별 최초 주문일 검색 테이블
                
                WITH Avg_price AS (
                    SELECT AVG(total_purchase) AS avg_price
                    FROM (
                        SELECT user_id, SUM(price_usd) AS total_purchase
                        FROM orders
                        GROUP BY user_id
                    ) AS user_purchase
                ),
                VIP AS (
                    SELECT o.user_id, SUM(o.price_usd) AS total_purchase, a.avg_price
                    FROM orders o
                    CROSS JOIN Avg_price a
                    GROUP BY o.user_id, a.avg_price
                    HAVING SUM(o.price_usd) >= 1.5 * a.avg_price
                ),
                
                first_order AS (
                	SELECT DISTINCT user_id,
                			CASE WHEN user_id IN (SELECT user_id
                								FROM vip) THEN 'VIP'
                			ELSE 'Regular' END AS 'User_Segment', min(created_at) AS first_ord
                	FROM orders
                	GROUP BY 1
                ),
                
                # 첫 주문 이후 n개월 주문 여부
                order_month AS (SELECT DISTINCT fo.user_id, User_segment,
                		CASE WHEN created_at = date_format( date_format( first_ord, '%Y-%m-01'), '%Y-%m-01') THEN 0
                		WHEN created_at > date_format( date_format( first_ord, '%Y-%m-01'), '%Y-%m-01') AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 1 month) THEN 1
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 1 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 2 month) THEN 2
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 2 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 3 month) THEN 3
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 3 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 4 month) THEN 4
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 4 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 5 month) THEN 5
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 5 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 6 month) THEN 6
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 6 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 7 month) THEN 7
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 7 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 8 month) THEN 8
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 8 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 9 month) THEN 9
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 9 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 10 month) THEN 10
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 10 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 11 month) THEN 11
                		WHEN created_at > date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 11 month) AND created_at <= date_add(date_format( first_ord, '%Y-%m-01'), INTERVAL 12 month) THEN 12
                		END AS month_num
                
                FROM first_order fo JOIN orders o ON fo.user_id = o.user_id
                ORDER BY month_num desc
                ),
                
                # 추가 주문 순서
                seq_table AS (
                	SELECT user_id, User_segment, month_num
                			FROM order_month
                )
                
                SELECT User_segment,
                		CASE WHEN month_num = 0 THEN 'm-0'
                		WHEN month_num = 1  THEN 'm-1'
                		WHEN month_num = 2  THEN 'm-2'
                		WHEN month_num = 3  THEN 'm-3'
                		WHEN month_num = 4  THEN 'm-4'
                		WHEN month_num = 5  THEN 'm-5'
                		WHEN month_num = 6  THEN 'm-6'
                		WHEN month_num = 7  THEN 'm-7'
                		WHEN month_num = 8  THEN 'm-8'
                		WHEN month_num = 9  THEN 'm-9'
                		WHEN month_num = 10 THEN 'm-10'
                		WHEN month_num = 11 THEN 'm-11'
                		WHEN month_num = 12 THEN 'm-12'
                		ELSE 'unknown'
                		END AS month_range,
                		count(user_id) user_cnt
                FROM seq_table
                GROUP BY 1,2
                ORDER BY 1,2;
                ```
                
        
         
        
        1.  **세분화된 고객 등급별 구매 행동 분석**
            - **분기별 구매고객 추세변화**
                
                ```sql
                #RFM_monthly 
                WITH customer_orders AS (
                    SELECT
                        o.user_id,
                        DATE_FORMAT(o.created_at, '%Y-%m') AS order_month, -- 월별 분석
                        CASE
                            WHEN MONTH(o.created_at) BETWEEN 1 AND 3 THEN CONCAT(YEAR(o.created_at), '-Q1_Custom')
                            WHEN MONTH(o.created_at) BETWEEN 4 AND 6 THEN CONCAT(YEAR(o.created_at), '-Q2_Custom')
                            WHEN MONTH(o.created_at) BETWEEN 7 AND 9 THEN CONCAT(YEAR(o.created_at), '-Q3_Custom')
                            WHEN MONTH(o.created_at) BETWEEN 10 AND 12 THEN CONCAT(YEAR(o.created_at), '-Q4_Custom')
                        END AS custom_quarter, -- 사용자 지정 분기
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count,
                        SUM(o.items_purchased) AS total_unit,
                        DATE_FORMAT(MAX(o.created_at), '%Y-%m') AS last_order_month‘
                    FROM orders o
                    GROUP BY o.user_id, order_month, custom_quarter
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders
                ),
                customer_segments AS (
                    SELECT
                        co.user_id,
                        co.order_month,
                        co.custom_quarter,
                        co.total_spent,
                        co.order_count,
                        co.total_unit,
                        co.last_order_month,
                        CASE
                            WHEN co.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM customer_orders co
                )
                SELECT 
                    cs.order_month,
                    cs.custom_quarter,
                
                    -- VIP 관련 데이터
                    ROUND(SUM(CASE WHEN cs.segment = 'VIP' THEN cs.total_spent ELSE 0 END), 0) AS vip_revenue,
                    COUNT(DISTINCT CASE WHEN cs.segment = 'VIP' THEN cs.user_id END) AS vip_user_count,
                    ROUND(SUM(CASE WHEN cs.segment = 'VIP' THEN cs.order_count ELSE 0 END), 0) AS vip_order_count,
                    ROUND(SUM(CASE WHEN cs.segment = 'VIP' THEN cs.order_count ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'VIP' THEN cs.user_id END), 0), 2) AS vip_freq,
                    ROUND(SUM(CASE WHEN cs.segment = 'VIP' THEN cs.total_spent ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'VIP' THEN cs.user_id END), 0), 0) AS vip_aov,
                    ROUND(SUM(CASE WHEN cs.segment = 'VIP' THEN cs.total_unit ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'VIP' THEN cs.user_id END), 0), 0) AS vip_upm,
                
                    -- Non-VIP 관련 데이터
                    ROUND(SUM(CASE WHEN cs.segment = 'Non_VIP' THEN cs.total_spent ELSE 0 END), 0) AS non_vip_revenue,
                    COUNT(DISTINCT CASE WHEN cs.segment = 'Non_VIP' THEN cs.user_id END) AS non_vip_count,
                    ROUND(SUM(CASE WHEN cs.segment = 'Non_VIP' THEN cs.order_count ELSE 0 END), 0) AS non_vip_order_count,
                    ROUND(SUM(CASE WHEN cs.segment = 'Non_VIP' THEN cs.order_count ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'Non_VIP' THEN cs.user_id END), 0), 2) AS non_vip_freq,
                    ROUND(SUM(CASE WHEN cs.segment = 'Non_VIP' THEN cs.total_spent ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'Non_VIP' THEN cs.user_id END), 0), 0) AS non_vip_aov,
                    ROUND(SUM(CASE WHEN cs.segment = 'Non_VIP' THEN cs.total_unit ELSE 0 END) / 
                          NULLIF(COUNT(DISTINCT CASE WHEN cs.segment = 'Non_VIP' THEN cs.user_id END), 0), 0) AS non_vip_upm
                
                FROM customer_segments cs
                GROUP BY cs.order_month, cs.custom_quarter
                ORDER BY cs.order_month, cs.custom_quarter;
                
                ```
                
            - **최근 3개월 구매 행동 분석**
                
                ```sql
                WITH recent_orders AS (
                    SELECT 
                        o.user_id, 
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count,
                        SUM(o.items_purchased) AS total_items -- 'items_purchased'로 총 구매한 제품 수 합산
                    FROM orders o
                    WHERE DATE(o.created_at) >= DATE_SUB(DATE('2015-3-19'), INTERVAL 3 MONTH) -- 최근 3개월
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders -- 평균 주문 금액의 1.5배를 VIP 기준으로 설정
                ),
                customer_segments AS (
                    SELECT 
                        ro.user_id,
                        ro.total_spent,
                        ro.order_count,
                        ro.total_items, -- 총 제품 수
                        CASE
                            WHEN ro.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM recent_orders ro
                )
                SELECT 
                    cs.segment,
                    COUNT(cs.user_id) AS user_count,  -- 세그먼트별 사용자 수
                    COALESCE(ROUND(AVG(cs.total_spent / cs.order_count), 2), 0) AS avg_order_value,  -- 세그먼트별 평균 주문 금액 (최근 3개월) 
                    COALESCE(SUM(cs.total_spent), 0) AS total_spent_3_months,  -- 세그먼트별 총 구매 금액 (최근 3개월)
                    COALESCE(SUM(cs.order_count), 0) AS total_order_count_3_months,  -- 세그먼트별 총 구매 횟수 (최근 3개월)
                    COALESCE(SUM(cs.total_items) / COUNT(cs.user_id), 0) AS unit_per_member  -- 세그먼트별 한 사람이 구매한 제품 개수 (UPM)
                FROM customer_segments cs
                GROUP BY cs.segment
                ORDER BY 1 DESC;
                ```
                
            - **그룹별 최근 3개월 간 평균 구매 금액**
                
                ```sql
                WITH recent_orders AS (
                    SELECT 
                        o.user_id, 
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count
                    FROM orders o
                    WHERE DATE(o.created_at) >= DATE_SUB(DATE('2015-03-19'), INTERVAL 3 MONTH)
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders
                ),
                customer_segments AS (
                    SELECT 
                        ro.user_id,
                        ro.total_spent,
                        ro.order_count,
                        CASE
                            WHEN ro.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM recent_orders ro
                )
                SELECT 
                	'2014-12-19 ~ 2015-03-19' AS period,
                    cs.segment,
                    COALESCE(ROUND(AVG(cs.total_spent / cs.order_count), 0), 0) AS avg_order_value
                FROM customer_segments cs
                GROUP BY cs.segment
                ORDER BY cs.segment DESC;
                ```
                
            - **그룹별 최근 3개월 간 총 매출**
                
                ```sql
                WITH recent_orders AS (
                    SELECT 
                        o.user_id, 
                        SUM(o.price_usd) AS total_spent
                    FROM orders o
                    WHERE DATE(o.created_at) BETWEEN '2014-12-19' AND '2015-03-19'
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders
                ),
                customer_segments AS (
                    SELECT 
                        ro.user_id,
                        ro.total_spent,
                        CASE
                            WHEN ro.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM recent_orders ro
                )
                SELECT 
                    '2014-12-19 ~ 2015-03-19' AS period, -- 날짜 범위 추가
                    cs.segment,
                    COALESCE(ROUND(SUM(cs.total_spent), 0), 0) AS total_spent_3_months
                FROM customer_segments cs
                GROUP BY cs.segment
                ORDER BY cs.segment DESC;
                
                ```
                
            - **그룹별 최근 3개월 간 총 구매 건수**
                
                ```sql
                WITH recent_orders AS (
                    SELECT 
                        o.user_id, 
                        SUM(o.price_usd) AS total_spent,  -- 총 구매 금액
                        COUNT(DISTINCT o.order_id) AS order_count  -- 총 주문 횟수
                    FROM orders o
                    WHERE DATE(o.created_at) >= DATE_SUB(DATE('2015-03-19'), INTERVAL 3 MONTH) -- 최근 3개월
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders -- VIP 기준: 평균 주문 금액 * 1.5
                ),
                customer_segments AS (
                    SELECT 
                        ro.user_id,
                        ro.total_spent,
                        ro.order_count,
                        CASE
                            WHEN ro.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM recent_orders ro
                )
                SELECT 
                    '2014-12-19 ~ 2015-03-19' AS period, -- 날짜 범위 추가
                    cs.segment,
                    COALESCE(SUM(cs.order_count), 0) AS total_order_count_3_months
                FROM customer_segments cs
                GROUP BY cs.segment
                ORDER BY cs.segment DESC;
                ```
                
            - **그룹별 최근 3개월 간 한 사람이 구매한 제품 개수**
                
                ```sql
                WITH recent_orders AS (
                    SELECT 
                        o.user_id, 
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count,
                        SUM(o.items_purchased) AS total_items
                    FROM orders o
                    WHERE DATE(o.created_at) BETWEEN '2014-12-19' AND '2015-03-19' -- 최근 3개월
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders
                ),
                customer_segments AS (
                    SELECT 
                        ro.user_id,
                        ro.total_items,
                        CASE
                            WHEN ro.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Non_VIP'
                        END AS segment
                    FROM recent_orders ro
                )
                SELECT 
                    '2014-12-19 ~ 2015-03-19' AS period, -- 날짜 범위 추가
                    cs.segment,
                    COALESCE(ROUND(SUM(cs.total_items) / COUNT(cs.user_id), 1), 0) AS unit_per_member -- 세그먼트별 평균 구매 제품 수
                FROM customer_segments cs
                GROUP BY cs.segment
                ORDER BY cs.segment DESC;
                ```
                
        
        1.  **신규 방문자 구매 행동 분석**
            - **기간별 신규 고객 수**
                
                ```sql
                WITH First_Purchases AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2012-03-19' AND '2013-03-18'
                    GROUP BY user_id
                ),
                New_Customers_2012 AS (
                    SELECT '2012-03-19~2013-03-18' AS period, COUNT(*) AS new_customers
                    FROM First_Purchases
                ),
                First_Purchases_2013 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2013-03-19' AND '2014-03-18'
                    GROUP BY user_id
                    HAVING MIN(created_at) NOT IN (
                        SELECT first_purchase_date FROM First_Purchases
                    )
                ),
                New_Customers_2013 AS (
                    SELECT '2013-03-19~2014-03-18' AS period, COUNT(*) AS new_customers
                    FROM First_Purchases_2013
                ),
                First_Purchases_2014 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2014-03-19' AND '2015-03-19'
                    GROUP BY user_id
                    HAVING MIN(created_at) NOT IN (
                        SELECT first_purchase_date FROM First_Purchases_2013
                    )
                ),
                New_Customers_2014 AS (
                    SELECT '2014-03-19~2015-03-19' AS period, COUNT(*) AS new_customers
                    FROM First_Purchases_2014
                )
                SELECT * FROM New_Customers_2012
                UNION ALL
                SELECT * FROM New_Customers_2013
                UNION ALL
                SELECT * FROM New_Customers_2014
                ORDER BY period;
                ```
                
            - **기간별 신규 고객 평균 구매액, 평균 제품 구매 개수**
                
                ```sql
                WITH First_Purchases_2012 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2012-03-19' AND '2013-03-18'
                    GROUP BY user_id
                ),
                New_Customers_2012 AS (
                    SELECT '2012-03-19~2013-03-18' AS period, 
                           ROUND(AVG(o.price_usd), 2) AS new_customer_spent_avg,
                           ROUND(AVG(o.items_purchased), 2) AS new_customer_items_purchased_avg
                    FROM First_Purchases_2012 fp
                    JOIN orders o ON fp.user_id = o.user_id
                ),
                First_Purchases_2013 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2013-03-19' AND '2014-03-18'
                    GROUP BY user_id
                    HAVING user_id NOT IN (SELECT user_id FROM First_Purchases_2012)
                ),
                New_Customers_2013 AS (
                    SELECT '2013-03-19~2014-03-18' AS period, 
                           ROUND(AVG(o.price_usd), 2) AS new_customer_spent_avg,
                           ROUND(AVG(o.items_purchased), 2) AS new_customer_items_purchased_avg
                    FROM First_Purchases_2013 fp
                    JOIN orders o ON fp.user_id = o.user_id
                ),
                First_Purchases_2014 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2014-03-19' AND '2015-03-19'
                    GROUP BY user_id
                    HAVING user_id NOT IN (SELECT user_id FROM First_Purchases_2012)
                           AND user_id NOT IN (SELECT user_id FROM First_Purchases_2013)
                ),
                New_Customers_2014 AS (
                    SELECT '2014-03-19~2015-03-19' AS period, 
                           ROUND(AVG(o.price_usd), 2) AS new_customer_spent_avg,
                           ROUND(AVG(o.items_purchased), 2) AS new_customer_items_purchased_avg
                    FROM First_Purchases_2014 fp
                    JOIN orders o ON fp.user_id = o.user_id
                )
                SELECT * FROM New_Customers_2012
                UNION ALL
                SELECT * FROM New_Customers_2013
                UNION ALL
                SELECT * FROM New_Customers_2014
                ORDER BY period;
                ```
                
            - **기간별 신규 고객이 구매한 제품의 비율**
                
                ```sql
                WITH First_Purchases_2012 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2012-03-19' AND '2013-03-18'
                    GROUP BY user_id
                ),
                First_Purchase_Details_2012 AS (
                    SELECT o.user_id, 
                           '2012-03-19~2013-03-18' AS period,
                           o.primary_product_id,
                           ROW_NUMBER() OVER (PARTITION BY o.user_id ORDER BY o.created_at) AS rn
                    FROM orders o
                    JOIN First_Purchases_2012 fp 
                        ON o.user_id = fp.user_id AND o.created_at = fp.first_purchase_date
                ),
                New_Customers_2012 AS (
                    SELECT period, primary_product_id
                    FROM First_Purchase_Details_2012
                    WHERE rn = 1
                ),
                First_Purchases_2013 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2013-03-19' AND '2014-03-18'
                    GROUP BY user_id
                    HAVING user_id NOT IN (SELECT user_id FROM First_Purchases_2012)
                ),
                First_Purchase_Details_2013 AS (
                    SELECT o.user_id, 
                           '2013-03-19~2014-03-18' AS period,
                           o.primary_product_id,
                           ROW_NUMBER() OVER (PARTITION BY o.user_id ORDER BY o.created_at) AS rn
                    FROM orders o
                    JOIN First_Purchases_2013 fp 
                        ON o.user_id = fp.user_id AND o.created_at = fp.first_purchase_date
                ),
                New_Customers_2013 AS (
                    SELECT period, primary_product_id
                    FROM First_Purchase_Details_2013
                    WHERE rn = 1
                ),
                First_Purchases_2014 AS (
                    SELECT user_id, 
                           MIN(created_at) AS first_purchase_date
                    FROM orders
                    WHERE created_at BETWEEN '2014-03-19' AND '2015-03-19'
                    GROUP BY user_id
                    HAVING user_id NOT IN (SELECT user_id FROM First_Purchases_2013)
                ),
                First_Purchase_Details_2014 AS (
                    SELECT o.user_id, 
                           '2014-03-19~2015-03-19' AS period,
                           o.primary_product_id,
                           ROW_NUMBER() OVER (PARTITION BY o.user_id ORDER BY o.created_at) AS rn
                    FROM orders o
                    JOIN First_Purchases_2014 fp 
                        ON o.user_id = fp.user_id AND o.created_at = fp.first_purchase_date
                ),
                New_Customers_2014 AS (
                    SELECT period, primary_product_id
                    FROM First_Purchase_Details_2014
                    WHERE rn = 1
                ),
                Product_Counts AS (
                    SELECT period, primary_product_id, COUNT(*) AS count
                    FROM (
                        SELECT * FROM New_Customers_2012
                        UNION ALL
                        SELECT * FROM New_Customers_2013
                        UNION ALL
                        SELECT * FROM New_Customers_2014
                    ) AS combined
                    GROUP BY period, primary_product_id
                ),
                Total_Orders_Per_Period AS (
                    SELECT period, SUM(count) AS total_orders
                    FROM Product_Counts
                    GROUP BY period
                )
                SELECT p.period, 
                       p.primary_product_id,  
                       ROUND((p.count / t.total_orders) * 100, 2) AS percentage
                FROM Product_Counts p
                JOIN Total_Orders_Per_Period t 
                    ON p.period = t.period
                ORDER BY p.period, p.primary_product_id;
                ```
                
        
        1.  **퍼널 분석(Funnel Analysis) - 이탈률**
            - **고객 그룹별 각 단계 이탈률**
                
                ```sql
                with customer_orders AS (
                    SELECT
                        o.user_id,
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count
                    FROM orders o
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders -- 평균 주문 금액의 1.5배를 VIP 기준으로 설정(전체 주문 고객의 10%를 VIP라고 정의)
                ),
                customer_segments AS (
                    SELECT
                        co.user_id,
                        CASE
                            WHEN co.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            WHEN co.total_spent < (SELECT vip_threshold FROM avg_order_value) THEN 'Non_VIP'
                			END AS segment
                    FROM customer_orders co
                ),
                ranked_pageviews as (
                select
                	wp.website_session_id,
                	ws.user_id,
                	ws.created_at,
                	wp.pageview_url,
                	row_number() over (partition by ws.user_id,
                	wp.website_session_id
                order by
                	wp.created_at) as step_num
                from
                	website_sessions ws
                join website_pageviews wp on
                	wp.website_session_id = ws.website_session_id
                ),
                funnel_flow as (
                select
                	rp.website_session_id,
                	rp.user_id,
                	rp.created_at,
                	GROUP_CONCAT(rp.pageview_url order by rp.step_num separator ' → ') as user_funnel_flow
                from
                	ranked_pageviews rp
                group by
                	rp.user_id,
                	rp.website_session_id,
                	rp.created_at
                order by
                	rp.user_id,
                	rp.website_session_id
                )
                select
                	coalesce (cs.segment, 'Non Buyer') as segment,
                	(count(case when ff.user_funnel_flow like '%/home%' or ff.user_funnel_flow like '%lander%' then 1 end)-count(case when ff.user_funnel_flow like '%/products%' then 1 end))/count(case when ff.user_funnel_flow like '%/home%' or ff.user_funnel_flow like '%lander%' then 1 end) * 100 as `/home or /lander → /products`,
                	(count(case when ff.user_funnel_flow like '%/products%' then 1 end)-count(case when ff.user_funnel_flow like '%/the-original-mr-fuzzy%' or ff.user_funnel_flow like '%/the-forever-love-bear%' or ff.user_funnel_flow like '%/the-birthday-sugar-panda%' or ff.user_funnel_flow like '%/the-hudson-river-mini-bear%' then 1 end))/count(case when ff.user_funnel_flow like '%/products%' then 1 end) * 100 as `/products → /product_detail`,
                	(count(case when ff.user_funnel_flow like '%/the-original-mr-fuzzy%' or ff.user_funnel_flow like '%/the-forever-love-bear%' or ff.user_funnel_flow like '%/the-birthday-sugar-panda%' or ff.user_funnel_flow like '%/the-hudson-river-mini-bear%' then 1 end)-count(case when ff.user_funnel_flow like '%/cart%' then 1 end))/count(case when ff.user_funnel_flow like '%/the-original-mr-fuzzy%' or ff.user_funnel_flow like '%/the-forever-love-bear%' or ff.user_funnel_flow like '%/the-birthday-sugar-panda%' or ff.user_funnel_flow like '%/the-hudson-river-mini-bear%' then 1 end) * 100 as `/product_detail → /cart`,
                	(count(case when ff.user_funnel_flow like '%/cart%' then 1 end)-count(case when ff.user_funnel_flow like '%/shipping%' then 1 end))/count(case when ff.user_funnel_flow like '%/cart%' then 1 end) * 100 as `/cart →/shipping`,
                	(count(case when ff.user_funnel_flow like '%/shipping%' then 1 end)-count(case when ff.user_funnel_flow like '%/billing%' then 1 end))/count(case when ff.user_funnel_flow like '%/shipping%' then 1 end) * 100 as `/shipping →/billing`,
                	(count(case when ff.user_funnel_flow like '%/billing%' then 1 end)-count(case when ff.user_funnel_flow like '%/thank-you-for-your-order' then 1 end))/count(case when ff.user_funnel_flow like '%/billing%' then 1 end) * 100 as `/billing →/thank0you-for-your-order`
                from
                	funnel_flow ff
                left join customer_segments cs
                on ff.user_id = cs.user_id 
                group by
                	cs.segment;
                ```
                
            - **고객 그룹별 페이지 체류 시간**
                
                ```sql
                WITH customer_orders AS (
                    SELECT
                        o.user_id,
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count
                    FROM orders o
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders -- VIP 기준: 평균 주문 금액의 1.5배
                ),
                customer_segments AS (
                    SELECT
                        co.user_id,
                        CASE
                            WHEN co.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            ELSE 'Regular Customer'
                        END AS segment
                    FROM customer_orders co
                ),
                ranked_pageviews AS (
                    SELECT
                        wp.website_session_id,
                        ws.user_id,
                        ws.created_at,
                        wp.pageview_url,
                        wp.created_at AS page_creation,
                        ROW_NUMBER() OVER (PARTITION BY ws.user_id, wp.website_session_id ORDER BY wp.created_at) AS step_num
                    FROM website_sessions ws
                    JOIN website_pageviews wp ON wp.website_session_id = ws.website_session_id
                ),
                time_diff_calculated AS (
                    SELECT 
                        rp.user_id,
                        COALESCE(cs.segment, 'Non Buyer') AS segment,
                        rp.step_num,
                        TIMESTAMPDIFF(SECOND, LAG(rp.page_creation) OVER (PARTITION BY rp.user_id, rp.website_session_id ORDER BY rp.step_num), rp.page_creation) AS time_diff
                    FROM ranked_pageviews rp
                    LEFT JOIN customer_segments cs ON rp.user_id = cs.user_id
                )
                SELECT 
                    segment,
                    step_num AS page,
                    AVG(time_diff) AS avg_time_on_page
                FROM time_diff_calculated
                WHERE time_diff IS NOT NULL
                GROUP BY segment, step_num
                HAVING COUNT(time_diff) > 0
                ORDER BY segment, step_num;
                ```
                
            - **고객 그룹별 유입 페이지 이탈률**
                
                ```sql
                with customer_orders AS (
                    SELECT
                        o.user_id,
                        SUM(o.price_usd) AS total_spent,
                        COUNT(DISTINCT o.order_id) AS order_count
                    FROM orders o
                    GROUP BY o.user_id
                ),
                avg_order_value AS (
                    SELECT AVG(price_usd) * 1.5 AS vip_threshold FROM orders -- 평균 주문 금액의 1.5배를 VIP 기준으로 설정(전체 주문 고객의 10%를 VIP라고 정의)
                ),
                customer_segments AS (
                    SELECT
                        co.user_id,
                        CASE
                            WHEN co.total_spent >= (SELECT vip_threshold FROM avg_order_value) THEN 'VIP'
                            WHEN co.total_spent < (SELECT vip_threshold FROM avg_order_value) THEN 'Non_VIP'
                			END AS segment
                    FROM customer_orders co
                ),
                ranked_pageviews as (
                select
                	wp.website_session_id,
                	ws.user_id,
                	ws.created_at,
                	wp.pageview_url,
                	row_number() over (partition by ws.user_id,
                	wp.website_session_id
                order by
                	wp.created_at) as step_num
                from
                	website_sessions ws
                join website_pageviews wp on
                	wp.website_session_id = ws.website_session_id
                ),
                funnel_flow as (
                select
                	rp.website_session_id,
                	rp.user_id,
                	rp.created_at,
                	GROUP_CONCAT(rp.pageview_url order by rp.step_num separator ' → ') as user_funnel_flow
                from
                	ranked_pageviews rp
                group by
                	rp.user_id,
                	rp.website_session_id,
                	rp.created_at
                order by
                	rp.user_id,
                	rp.website_session_id
                )
                select
                	coalesce(cs.segment, 'Non-Buyer'),
                	(count(case when ff.user_funnel_flow like '/home%' then 1 end) - count(case when ff.user_funnel_flow like '/home %' then 1 end)) / count(case when ff.user_funnel_flow like '/home%' then 1 end) * 100 as `/home`,
                	(count(case when ff.user_funnel_flow like '/lander-1%' then 1 end) - count(case when ff.user_funnel_flow like '/lander-1 %' then 1 end)) / count(case when ff.user_funnel_flow like '/lander-1%' then 1 end) * 100 as `/lander-1`,
                	(count(case when ff.user_funnel_flow like '/lander-2%' then 1 end) - count(case when ff.user_funnel_flow like '/lander-2 %' then 1 end)) / count(case when ff.user_funnel_flow like '/lander-2%' then 1 end) * 100 as `/lander-2`,
                	(count(case when ff.user_funnel_flow like '/lander-3%' then 1 end) - count(case when ff.user_funnel_flow like '/lander-3 %' then 1 end)) / count(case when ff.user_funnel_flow like '/lander-3%' then 1 end) * 100 as `/lander-3`,
                	(count(case when ff.user_funnel_flow like '/lander-4%' then 1 end) - count(case when ff.user_funnel_flow like '/lander-4 %' then 1 end)) / count(case when ff.user_funnel_flow like '/lander-4%' then 1 end) * 100 as `/lander-4`,
                	(count(case when ff.user_funnel_flow like '/lander-5%' then 1 end) - count(case when ff.user_funnel_flow like '/lander-5 %' then 1 end)) / count(case when ff.user_funnel_flow like '/lander-5%' then 1 end) * 100 as `/lander-5`
                from
                	funnel_flow ff
                left join customer_segments cs
                on ff.user_id = cs.user_id 
                group by
                	cs.segment;
                ```
                
</aside>

## 1. 서론

### a. 프로젝트 소개

Maven Fuzzy Factory는 3년간 운영된 가상의 이커머스 플랫폼으로, 마케팅 전략 수립을 위해 EDA를 진행하려고 합니다.  리텐션 분석, RFM 분석, 퍼널 분석을 활용해 고객 세분화 및 구매 행동을 분석했습니다.

- **문제 상황**: VIP 고객은 전체 고객의 10%이지만, 매출의 17%를 차지합니다. 2015년 Q4~Q1 동안 VIP 고객 수, 구매 횟수, 빈도를 증가시켜 매출 성장을 목표로 했습니다.
- **분석 목표**: 반복 구매 고객이 기업 매출의 40% 이상을 차지하며, 고객 유지율 5% 증가 시 수익이 25~95% 증가할 수 있다는 연구를 바탕으로, EDA를 통해 고객 데이터의 추세와 현황을 파악하고자 했습니다.

**문제 정의**

- VIP 고객군을 정의하고 유지율(Retention Rate)을 높일 방법을 찾아야 함. → 리텐션분석
- 고객의 구매 행동을 분석하여 효과적인 유입 전략을 수립해야 함. → RFM 분석, 퍼널분석

**목적**

- 구매 금액에 따라 고객 등급을 나누어, 고객특성 파악 후 맞춤형 마케팅 플랜 설계 지원
    - VIP 고객군 vs Non_VIP 정의하고, 유지율 및 구매 행동을 분석하여 충성도를 높이는 전략을 마련
    - 신규 방문자 중 구매 고객의 특성을 파악하여 전환율을 높이는 방법을 모색
- 전체 퍼널 분석을 통해 고객 유입부터 구매까지의 흐름을 최적화

### b. 데이터셋 소개 & EDA 접근 방법

https://www.kaggle.com/code/rubenman/maven-fuzzy-factory

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image.png)

**[고객 세분화 및 고객 등급 정의]**

- **고객 세분화 로직**
    
    **조건 1: 주문 금액 상위 25%, 최근 1년간 주문한 고객**
    
    - **단점:** 해당 비즈니스가 스타트업 단계(업력: 3년)인 것을 고려했을 때, 최근 1년의 데이터만 반영을 하는 것은 너무 데이터가 제한적이라서 분석이 어려움. 잠재적 VIP 고객(과거 높은 소비 but 최근 구매 X) 제외될 가능성이 있다.
    
    **조건 2: 주문 금액 상위 25%, 최근 3년간 주문한 고객**
    
    - **장점:** 장기적인 소비 패턴을 반영하여 안정적인 VIP 그룹을 설정 가능, 1년 기준 주문액과 3년 기준 주문액의 분포가 유사함. 기준 기간이 길어짐에 따라 고객 별 구매 횟수(Buying Frequency) 를 살펴볼 수 있다.
    - **단점:** 최근 구매 빈도가 낮은 고객이 포함될 가능성이 있음, 상위 25% 기준이 절대적이라 업계 상황에 따라 적절하지 않을 수 있다.
    
    **✅ 채택한 조건 - 조건 3: 전체 고객의 평균 주문 금액 × 1.5배 이상 소비한 고객, 최근 3년간 주문** 
    
    - **장점:** 고객별 장기적인 소비 성향을 반영. 1년 기준 주문액과 3년 기준 주문액의 분포가 유사하여 트랜드 파악에 문제가 없음. 비즈니스 성장이나 성장에 따라 평균 주문 금액이 변동되면서 VIP 기준이 유동적으로 조절 됨. 구매 기준 기간이 길어짐에 따라 고객 별 구매 횟수(Buying Frequency) 를 살펴볼 수 있다.
    - **단점:** 최근 구매 빈도가 낮은 고객이 포함될 가능성 있다.

| 고객 등급  | 코드 명칭 | 정의  |
| --- | --- | --- |
| VIP | VIP | 전체 고객의 평균 구매 금액 * 1.5 배 이상 구매한 고객  |
| 일반고객 | Non_VIP  | 구매 이력은 있으나 VIP 가 아닌 고객 |
| 신규구매고객 | New_Customers | 1년 단위로 구분해서 이전 연도에 구매한 적이 없다가 새로 구매한 고객 |
| 잠재적 고객 | Non_Buyer | 구매 이력이 없는 고객  |

**VIP 고객 세분화 조건 로직 : 최근 3년간 주문 기준, 전체 고객의 평균 주문 금액 × 1.5배 이상 소비한 고객**

- **장점:** 고객별 장기적인 소비 성향을 반영. 1년 기준 주문액과 3년 기준 주문액의 분포가 유사하여 트렌드 파악에 문제가 없음. 비즈니스 성장이나 성장에 따라 평균 주문 금액이 변동되면서 VIP 기준이 유동적으로 조절 됨. 구매 기준 기간이 길어짐에 따라 고객 별 구매 횟수(Frequency)를 살펴볼 수 있다.
- **단점:** 최근 구매 빈도가 낮은 고객이 포함될 가능성 있다.

**[최근 3년간, 1년간 주문액에 대한 분포도]**

![스크린샷 2025-02-17 오후 2.16.21.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2025-02-17_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.16.21.png)

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%201.png)

📌 분석할 데이터:

- 신규 vs 기존 고객 여부
- VIP 고객 정의 기준(예: 전체 구매 금액 평균의 1.5배 이상 지불한 구매자)
    - 이유: 기존에 VIP를 1년 단위를 기준으로 매출 50% 이상 차지하는 고객들을 VIP로 선정하고자 했으나, 고객 간 제품 구매 수, 구매 가격에 대한 차이가 유의미하지 않아 VIP 수가 1년 단위 제품 구매 고객의 약 4~50%를 차지하게 됨 → 운영한 지 3년 밖에 되지 않았다는 점과 구매 빈도가 적은 인형이라는 점을 고려하여 3년 전체 고객 평균 구매 금액의 1.5배 이상을 구매한 고객들을 VIP로 정의함.

---

## **2. 본론**

### **2-1. 분기별 구매고객 추세변화**

- **목적**: 본격적으로 인사이트 도출을 위한 소비자 데이터 분석 전, 고객 행동의 경향성 파악을 통해 구매고객의 데이터 추세 파악

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%202.png)

- **전체 기간:** 2012년 3월 - 2015년 3월
- 구매 고객을 VIP, 일반 고객 두 그룹으로 나누어 총 구매 금액, 구매 건수, 고객 수, 일반 고객과 VIP 비율, 고객 1인당 구매 빈도, 고객 1인당 구매액으로 분기별 구매 고객의 추세를 나타낸다.
    - **해석:** MavenFuzzyFactory 이커머스가 성장하면서 VIP 회원이 늘어남에 따라, 구매액, 주문 건수, 고객 수가 늘어나고 있으나, VIP 고객 1인당 지표 - 고객 1인당 구매 빈도(주문 건수/고객 수), 고객 1인당 구매액(구매액/고객 수) 는 성장하지 못하고 있다. → 고객 1인당 구매 빈도와 구매액을 늘릴 수 있는 방법은 무엇일까?
    

### **2-2. 전체 고객 및 VIP 그룹의 리텐션 분석 - 유지율**

**📌 분석할 데이터**

- 리텐션의 정의: 이커머스는 판매 제품에 따라 구매 주기가 한 달 보다 길 수 있으므로, 고객마다 첫 번째 달과 마지막 구매 달을 산정하여 리텐션 계산
    - **고객의 주문 건(구매)** 을 기준으로 **롤링 리텐션**을 계산
        
        → 롤링 리텐션(Rolling Retention): 특정 기준일에 가입한 사용자 중, 특정 날짜까지 한 번이라도 재구매 또는 방문한 비율
        
- 일반고객, VIP 고객군의 **롤링 리텐션**을 분석 비교
    - 참고 - 클래식 리텐션과 롤링 리텐션 설명
        1. **클래식 리텐션(Classic Retention)**: 특정 기준일(예: 첫 구매일)에 가입한 사용자 중, 이후 특정 기간(예: N일, N주, N개월) 후에도 활동을 유지한 비율.
        2. **롤링 리텐션(Rolling Retention)**: 특정 기준일에 가입한 사용자 중, 특정 날짜까지 **한 번이라도** 재방문한 비율.

**📌 지표 정의 설명**

첫 주문 달을 기준으로, 첫 달(month0)에 구매를 한 고객 수 집계

(month1) 첫 번째 달에도 구매하고 두 번째 달에도 구매한 고객 수 집계

(month2) 첫 번째 달에 구매하고 세 번째 달에도 구매한 고객 수 집계. 두 번째 구매 여부는 상관 없음

(month3) 첫 번째 달에 구매하고 네 번째 달에도 구매한 고객 수 집계. 두 번째, 세 번째 달  구매 여부는 상관 없음

📌 **유지율(Retention) 분석 요약**

**[첫구매 후 월별 평균 유지율(재구매 발생)]**

- 기간: 2012년 3월 - 2015년 3월 (3년)

| 구매고객 구분 | 첫구매 달 | 1달 후  | 2달 후 | 3달 후 | 4달 후 |
| --- | --- | --- | --- | --- | --- |
| VIP | - | 41% | 18% |  3% | 1% |
| Non_VIP | - | 1% | 1% | 0% | 0% |

**데이터 요약**

- **해석**: 일반 고객의 첫 구매 달부터 마지막 구매 달까지의 구매 리텐션은  month1, month2 기준 1%에 불과하였으나, VIP 고객의 구매 리텐션은 첫 구매 달로부터 month1, month2, month3, month4 각각 41%, 18%, 3%, 1%였다.
- **결론**(Action Item): 일반 고객과 VIP 고객의 구매 행동 패턴이나 인구 통계적 특성에서 차이가 있는 지 확인이 필요하다. 정기적으로 VIP고객의 리탠션을 확인 하여, VIP 고객의 리탠션 높은 원인이 무엇인지 파악이 필요하다.

**[Retention 비율 계산을 위한 구매 건 수 집계 &  비율]**

전체고객 

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%203.png)

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/2e5430be-46f8-42ef-a827-6e15a56dc76b.png)

VIP 

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%204.png)

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%205.png)

📌  **결론: 추가로 분석해볼 수 있는 데이터 포인트**

- **고객 세그먼트별 비교 (RFM 분석) -** 재구매 가능성이 높은 고객군 파악
    - 고객당 평균 구매액 (AOV, Average Order Value) = 총 매출 ÷ 주문 수
    - 고객당 구매 제품 개수 (UPM, Unit per Member) = 판매 제품 개수 ÷ 고객 수
    - 평균 재구매 횟수 = 고객당 평균 주문 건수
- **신규 방문자 구매 행동 분석**
- 제품별 유지율 분석: 특정 제품이 유지율에 미치는 영향 분석
- **퍼널분석**을 통해 일반 고객의 이탈 원인과 VIP 고객의 이탈 원인이 같은 지 검증 필요.

---

### 2-2. RFM 고객 세분화 분석 EDA

RFM: Recency Frequency Monetary 분석

- 목적
    - 소비자 데이터 경향성 EDA 통한 비즈니스 성장 전략에 지원
    - 본격적으로 인사이트 도출을 위한 소비자 데이터 분석 전, 고객 행동의 경향성 파악을 통해 세그먼트 별로 고객 행동을 파악.
    - 고객 세분화를 위한 구매금액조건 Thredshold 설정 시 참고

- 지표: 구매 금액, 구매 건수,  평균 구매 금액, 구매자 수,  평균 구매 제품 수
    - 가장 최근 3개월
    - 지난 3년간 분기별 추세변화

**[최근 3개월 구매 고객 행동 분석 (VIP 고객과 Non-VIP 고객)]**

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%206.png)

**데이터 요약**

- 기간: 2014-12-19 ~ 2015-03-19(최근 3개월)
- VIP 고객 비중: 15%
- Non-VIP 고객 비중**:** 85%

**전략 제안**

- Non-VIP 고객 중에서 VIP로 승급할 가능성이 있는 고객을 선별하고, 이들에게 맞춤형 혜택을 제공
- 인센티브(할인, 무료배송)를 제공하는 것이 필요하고 일정 기간 내 **특정 금액 이상**을 소비한 고객을 VIP로 자동 승급하는 **VIP 승급 인센티브 프로그램**을 운영함으로써 자연스럽게 Non_VIP 고객이 자연스럽게 VIP로 전환되도록 유도.

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%207.png)

**데이터 요약**

- VIP 고객 총 매출**:** 97,506 USD
- Non-VIP 고객 총 매출**:** 298,863 USD
- Non-VIP 고객이 ****전체 매출의 75% 차지

**전략 제안**

- VIP 고객군: 마케팅 TouchPoint를 늘리고, 마케팅 예산과 리소스를 더 늘려서, 이탈 방지
- Non-VIP 고객: 평균구매비용을 높일 수 있도록, 장바구니 금액 threshold 를 여러구간으로 나누어 효과적인 쿠폰 운영을 위한 분석 진행하여 인사이트를 기반으로 쿠폰 발급
    - threshold 정할 때 마진/이익 분석 필요
    
    예. 알리익스프레스의 쿠폰
    
    ![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%208.png)
    

 

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%209.png)

**데이터 요약**

- VIP 고객 평균 주문 금액(AOV): 97 USD
- Non-VIP 고객 평균 주문 금액(AOV): 57 USD
- VIP 고객 AOV가 Non-VIP 고객 대비 ****약 70% 더 높음

**전략제안**

- 구매금액을 기준으로 VIP 고객을 정의하였으므로 VIP 고객의 지속적인 구매를 유도하려면 맞춤형 할인과 365 멤버십 혜택(생일쿠폰 등) 을 강화
- 할인 쿠폰, 추천 리워드, 멤버십 업그레이드와 같은 다양한 프로모션을 활용

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%2010.png)

**데이터 요약**

- VIP 고객 총 구매 건수: 1,046건
- Non-VIP 고객 총 구매 건수: 5,278건
- VIP 고객이 Non-VIP 고객 대비 ****구매 건수가 1/5 수준

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%2011.png)

**데이터 요약**

- VIP 고객 평균 구매 제품 개수: 2.1개
- Non-VIP 고객 평균 구매 제품 개수: 1.2개

**전략 제안**

- VIP 고객: 주문건 당 물품 구매개수를 상승시킬 수 있도록  관련 제품 추가 구매  시 할인을 제공하는 전략을 도입 및 업셀링 & 크로스 셀링을 위한 추천 서비스 개발 필요

- [참고: 신규 구매고객 구매 행동 EDA]
    - 신규 구매 고객은 MavenFuzzyFactory 회사가 시작된 2012-03-19를 기준으로 1년 단위 별(2012-03-19 ~ 2013-03-18, 2013-03-19 ~ 2014-03-18, 2014-03-19, 2015-03-19)로 이전에 구매한 기록이 없고 해당 기간에 처음 구매한 고객을 신규 구매 고객(order_count = 1)으로 정의하고 분석을 진행
    
    **신규 구매 고객 수**
    
    ![스크린샷 2025-02-14 오후 2.50.36.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2025-02-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.50.36.png)
    
    **데이터 요약**
    
    - 최초 구매 건을 기준으로 신규 구매 고객 수를 파악
    - 기간 별 신규 고객 수가 각 기간 약 142.54%, 약 118.03% 증가
        - 3,634명 → 8,814명 → 19,217명
    
    **신규 고객이 구매한 제품의 비율** 
    
    ![스크린샷 2025-02-14 오후 2.53.49.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2025-02-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.53.49.png)
    
    **데이터 요약** 
    
    - 신규 고객들은 ‘The Original Mr. Fuzzy’ 제품을 선호
    
    **신규 구매 고객의 평균주문금액 및 구매제품개수**
    
    ![스크린샷 2025-02-14 오후 3.05.52.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2025-02-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_3.05.52.png)
    
    **데이터 요약** 
    
    - 평균 구매액
        - 각 기간 약 9.57%, 약 14.92% 증가
        - $50.66 → $55.51 → $63.79
    - 평균 제품 구매 개수
        - 각 기간 9%, 약 23.85% 증가
        - 1개 → 1.09개 →1.35개
        
    

---

### **2-3. 퍼널 분석(Funnel Analysis) - 이탈률**

📌 분석할 데이터:

방문 → 장바구니 추가 → 결제 완료까지 단계별 이탈률

- 고객 분류에 따라 나누어서 단계별 이탈률을 분석 실시

특정 단계에서의 이탈률

VIP 고객과 다른 고객의 퍼널 차이 분석 (세그먼트)

**[설명]**

MavenFuzzyFactory 회사의 웹 사이트에서 물건을 구매하는 절차를 6단계로 나누어서 해당 단계에서 전 단계 대비 남은 고객 수를 계산한 뒤, % 단위로 계산하여 이탈률을 도출.

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%2012.png)

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%2013.png)

![image.png](%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%2001%20%E1%84%85%E1%85%B5%E1%84%90%E1%85%A6%E1%86%AB%E1%84%89%E1%85%A7%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%8B%E1%85%B5%E1%84%8F%E1%85%A5%E1%84%86%E1%85%A5%E1%84%89%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%86%E1%85%A1%E1%84%8F%E1%85%A6%E1%84%90%E1%85%B5%E1%86%BC%20%E1%84%8C%E1%85%A5%201c9339280c3780f28fc1d128e9df0466/image%2014.png)

1단계의 /home과 /lander의 경우 둘 모두 시작점이자 하나의 세션에서 둘 모두를 방문하는 경우는 없었기에 하나로 묶었다.

2단계의 경우 /product를 제외한 둘은 제품 상세 페이지로 유의미한 결과를 얻을 수 없을 것이라고 판단하여 하나로 묶었다.

5단계의 경우 결제한다는 하나의 절차를 두 개로 나뉘어진 것이기 때문에 (둘을 모두 방문한 케이스가 없음) 하나로 묶어서 계산했다.

6단계까지 도달해야 물건을 구매한 것이라고 판단된다.

- /home과 /lander-1 페이지의 경우 웹 사이트의 시작이므로 무조건 100%이므로 비율 대신 해당 경로로 방문한 사용자 수를 집계했다.
- 해당 데이터를 유입 경로(UTM_Source)와 고객 유형 세그먼트로 나누어서 시각화 후 비교했다.

---

## 3. 결론 요약

- **VIP 고객 vs Non-VIP 고객**
    - **VIP 고객 수 증가에도 불구하고 구매 빈도 및 평균 구매액 정체** VIP 고객 유지 및 재구매 유도 전략 필요
    - **VIP 고객 비율(15%)이 낮아 매출 기여도 부족** → 충성도 강화 및 반복 구매 유도 필요
    - **VIP 고객 매출이 75% 차지하지만 평균 주문 금액이 낮음** → 장기적인 성장 전략 필요
- **신규 구매 고객**
    - **신규 구매 고객 수 증가 추세** → 충성 고객 전환 전략 필요
    - **신규 고객 대부분 특정 제품 구매** → 플래그십 제품 홍보 및 신제품 개발에 활용 가능
    - **첫 구매 후 재구매율 급감 (VIP 1개월 유지율 41% → 3개월 후 3%)** → 첫 구매 후 리마케팅 전략 필수
- **퍼널**
    - **홈페이지 및 광고에서 이탈률 높음** → 광고 최적화 및 UI/UX 개선 필요
    - **결제 과정 복잡, 사용자 세션 종료 많음** → 결제 프로세스 간소화 및 로딩 속도 개선 필요
    - **VIP와 일반 고객의 페이지 이탈률 차이 없음** → VIP 혜택 강화 및 맞춤 마케팅 필요