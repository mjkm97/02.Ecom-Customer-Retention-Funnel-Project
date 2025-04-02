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
            - **코드예시: VIP 그룹의 리텐션 분석(롤링리텐션)**
        
                
                ```sql
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

<img width="459" alt="image (37)" src="https://github.com/user-attachments/assets/3773a49b-4cf4-4a38-9aba-c3ceb26127d6" />


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
<img width="759" alt="스크린샷 2025-02-17 오후 2 16 21" src="https://github.com/user-attachments/assets/36058081-5bc1-4d9e-af0e-1b8c7a472975" />

![image (38)](https://github.com/user-attachments/assets/5243fa46-976f-4dba-bbad-bbf0816d9a09)




📌 분석할 데이터:

- 신규 vs 기존 고객 여부
- VIP 고객 정의 기준(예: 전체 구매 금액 평균의 1.5배 이상 지불한 구매자)
    - 이유: 기존에 VIP를 1년 단위를 기준으로 매출 50% 이상 차지하는 고객들을 VIP로 선정하고자 했으나, 고객 간 제품 구매 수, 구매 가격에 대한 차이가 유의미하지 않아 VIP 수가 1년 단위 제품 구매 고객의 약 4~50%를 차지하게 됨 → 운영한 지 3년 밖에 되지 않았다는 점과 구매 빈도가 적은 인형이라는 점을 고려하여 3년 전체 고객 평균 구매 금액의 1.5배 이상을 구매한 고객들을 VIP로 정의함.

---

## **2. 본론**

### **2-1. 분기별 구매고객 추세변화**

- **목적**: 본격적으로 인사이트 도출을 위한 소비자 데이터 분석 전, 고객 행동의 경향성 파악을 통해 구매고객의 데이터 추세 파악
![image (39)](https://github.com/user-attachments/assets/6999242d-73f6-478e-8415-b558d1e25649)

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

- 첫 주문 달을 기준으로, 첫 달(month0)에 구매를 한 고객 수 집계

- (month1) 첫 번째 달에도 구매하고 두 번째 달에도 구매한 고객 수 집계

- (month2) 첫 번째 달에 구매하고 세 번째 달에도 구매한 고객 수 집계. 두 번째 구매 여부는 상관 없음

- (month3) 첫 번째 달에 구매하고 네 번째 달에도 구매한 고객 수 집계. 두 번째, 세 번째 달  구매 여부는 상관 없음


 
  **📌 유지율(Retention) 분석 요약**

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
![image (42)](https://github.com/user-attachments/assets/1fb8b25c-9352-44fb-9826-bb0e82c30a1c)
<img width="810" alt="image (41)" src="https://github.com/user-attachments/assets/6c0dfe36-d2e0-4992-bbc8-a963ec6da70b" />


VIP 
<img width="809" alt="image (44)" src="https://github.com/user-attachments/assets/d0e9a531-1bd6-48cb-b5c3-d41819552aa7" />
<img width="820" alt="image (43)" src="https://github.com/user-attachments/assets/56ad4507-712b-4ae4-9ec5-daf341637531" />


📌  **결론: 추가로 분석해볼 수 있는 데이터 포인트**

- **고객 세그먼트별 비교 (RFM 분석) -** 재구매 가능성이 높은 고객군 파악
    - 고객당 평균 구매액 (AOV, Average Order Value) = 총 매출 ÷ 주문 수
    - 고객당 구매 제품 개수 (UPM, Unit per Member) = 판매 제품 개수 ÷ 고객 수
    - 평균 재구매 횟수 = 고객당 평균 주문 건수
- **신규 방문자 구매 행동 분석**
- 제품별 유지율 분석: 특정 제품이 유지율에 미치는 영향 분석
- **퍼널분석**을 통해 일반 고객의 이탈 원인과 VIP 고객의 이탈 원인이 같은 지 검증 필요.

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
 
