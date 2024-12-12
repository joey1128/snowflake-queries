# HubSpot Snowflake Data Share Repository

Find our [Snowflake Data Share Slide Deck]([https://docs.google.com/presentation/d/1WNayQcGZF3RkcfEpXBxP51trmUojqYieAM2uo-VrSH4/edit?usp=sharing](https://eng11e.seismic.com/i/evTkL8Grl8W6grjJksNkCciuEMsxfIBXgxqfeWJ8d1GMAwTh5sG5pLkK49d97nmrPYE___W___3eYyoCq___VIKuow0PLUSSIGNRTNXR9cGvg47xW1MrMiwyGnWpbxZ92i5VePLUSSIGNT8B0nCA)), which includes diagrams, queries, demonstration videos, and more.

## Table of Contents
1. [Overview](#overview)
2. [Why Activate Snowflake Data Share?](#why-activate-snowflake-data-share)
   - [Functional Benefits](#functional-benefits)
   - [Cost Benefits](#cost-benefits)
   - [Time-to-Value](#time-to-value)
3. [Key Queries and Use Cases](#key-queries-and-use-cases)
   - [For Marketing Teams](#1-for-marketing-teams)
   - [For Sales Teams](#2-for-sales-teams)
   - [For Customer Success Teams](#3-for-customer-success)
   - [For Data Teams](#4-for-data-teams)
   - [For Content Marketing Teams](#5-for-content-marketing)
4. [Getting Started](#getting-started)
5. [Advanced Features](#advanced-features)
6. [Conclusion](#conclusion)
7. [FAQ](#faq)

---

## Overview
This repository contains a collection of queries and resources designed to unlock the full potential of HubSpot's Snowflake Data Share. By connecting HubSpot data to your Snowflake data warehouse, you can enable advanced reporting, unify data across systems, and power scalable, data-driven decision-making for your organization.

HubSpot’s Operations Hub Enterprise (OHE) with Snowflake Data Share simplifies how businesses access and utilize CRM data, delivering real-time insights without additional storage or data transfer costs. Whether you're a marketer, data analyst, sales leader, or customer success professional, this repository provides a roadmap to extract value and solve common business challenges with unified reporting.

---

## Why Activate Snowflake Data Share?

### Functional Benefits
- **Proprietary HubSpot Tables**: Gain access to unique data tables, unavailable via API, for deeper reporting.
- **Unmatched Sync Frequency**: Choose between live updates every 15 minutes or daily schema updates.

### Cost Benefits
- **No Storage Costs**: Snowflake’s Data Sharing technology avoids storage fees by eliminating the need for data copying.
- **Fixed Integration Costs**: Costs are predictable regardless of data volume.

### Time-to-Value
- **Turnkey Installation**: Quick and straightforward setup.
- **Rapid Data Ingestion**: Updates occur in minutes for real-time analysis.

---

## Key Queries and Use Cases
This repository is categorized by user persona and business needs. Here’s a summary of what you can achieve:

### 1. **For Marketing Teams**

**Get all Contacts with Email Activity and Lifecycle Stage:**
```sql
SELECT
    C.objectid as ContactID, 
    C.property_firstname as FirstName,
    C.property_lastname as LastName,
    C.property_hs_email_last_open_date As LastEmailOpenDate,
    C.property_hs_email_last_click_date As LastEmailClickDate,
    C.property_lifecyclestage
FROM
    V2_LIVE.OBJECTS_CONTACTS c
ORDER BY C.property_hs_email_last_open_date ASC
SELECT
    C.objectid as ContactID, 
    C.property_firstname as FirstName,
    C.property_lastname as LastName,
    C.property_hs_email_last_open_date As LastEmailOpenDate,
    C.property_hs_email_last_click_date As LastEmailClickDate,
    C.property_lifecyclestage
FROM
    V2_LIVE.OBJECTS_CONTACTS c
ORDER BY C.property_hs_email_last_open_date ASC

```

**List Email Click Events for Contacts:**
```sql
SELECT 
    contacts.objectid As Contact_ID,
    contacts.property_email As Contact_Email,
    email_clicks.OCCURREDAT,
    email_clicks.PROPERTY_HS_CLICK_ORIGINAL_URL As URL, 
    email_clicks.PROPERTY_HS_EMAIL_CAMPAIGN_ID As Campaign_ID, emails.property_hs_campaign_name As Campaign_Name,
    emails.property_hs_name As Email, 
    campaigns.property_hs_name As Campaign, campaigns.property_hs_start_date As Campaign_Start, campaigns.property_hs_end_date As Campaign_End
FROM 
    OBJECTS_CONTACTS as contacts JOIN 
    EVENTS_CLICKED_LINK_IN_EMAIL_V2 as email_clicks ON contacts.OBJECTID=email_clicks.objectid JOIN
    OBJECTS_MARKETING_EMAILS as emails On emails.property_hs_origin_asset_id = email_clicks.property_hs_email_meta_content_id JOIN
    OBJECTS_CAMPAIGNS as campaigns On campaigns.property_hs_origin_asset_id = emails.property_hs_campaign_name
ORDER BY email_clicks.OCCURREDAT DESC


```

**Report on Deal revenue that is attributed by Contacts through Ad Interactions and Ad Metrics:**
```sql
SELECT
ad_interactions.PROPERTY_HS_AD_NETWORK As Ad_Network,
ad_interactions.PROPERTY_HS_AD_CAMPAIGN_NAME As Campaign,
ad_interactions.PROPERTY_HS_INTERACTION_TYPE As Interaction_Type,
ad_interactions.PROPERTY_HS_UTM_CAMPAIGN As UTM_Campaign, 
ad_interactions.PROPERTY_HS_UTM_MEDIUM As UTM_Medium, 
ad_interactions.PROPERTY_HS_UTM_SOURCE As UTM_Source, 
ad_metrics.OCCURREDAT As Report_Date, 
ad_metrics.PROPERTY_HS_AD_AMOUNT_SPENT As Ad_AmountSpent, 
ad_metrics.PROPERTY_HS_AD_CLICKS As Ad_Clicks, 
ad_metrics.PROPERTY_HS_AD_ENGAGEMENTS As Ad_Engagements, 
ad_metrics.PROPERTY_HS_AD_IMPRESSIONS As Ad_Impressions, 
ad_metrics.PROPERTY_HS_AD_LIKES As Ad_Likes, 
ad_metrics.PROPERTY_HS_AD_NETWORK_CONVERSIONS As Ad_Network_Conversions, 
contacts.PROPERTY_FIRSTNAME As First_Name, contacts.PROPERTY_LASTNAME As Last_Name,
deals.PROPERTY_DEALNAME As Deal_Name, deals.property_amount As Deal_Amount
FROM
EVENTS_AD_INTERACTION As ad_interactions JOIN 
EVENTS_AD_METRICS_IMPORTED_V0 As ad_metrics ON ad_interactions.PROPERTY_HS_AD_CAMPAIGN_ID=ad_metrics.PROPERTY_HS_AD_CAMPAIGN_ID JOIN 
OBJECTS_CONTACTS As contacts ON ad_interactions.objectid=contacts.objectid JOIN 
ASSOCIATIONS_CONTACTS_TO_DEALS ON contacts.OBJECTID=associations_contacts_to_deals.CONTACT_OBJECTID JOIN 
OBJECTS_DEALS As deals ON associations_contacts_to_deals.DEAL_OBJECTID=deals.OBJECTID;
WHERE
contacts.property_createdate < DATE_TRUNC ('MONTH', CURRENT_DATE)
AND contacts.property_createdate >= DATE_TRUNC ('MONTH', CURRENT_DATE) - INTERVAL '1 MONTH'



```

**List of top contacts completing a particular activity on website (Custom Event):**
```sql
SELECT
    DATE_TRUNC('month', eki.occurredat) AS month,
    oc.property_email,
    COUNT(*) AS interaction_count
FROM
    events_key_customer_interactions eki
JOIN
    objects_contacts oc ON eki.objectid = oc.objectid
GROUP BY
    month,
    oc.property_email
ORDER BY
    month,
    interaction_count



```

**Lead Source Effectiveness (Custom Events):**
```sql
SELECT
    ecls.Property_lead_source_category,
    COUNT(CASE WHEN oc.property_lifecyclestage = 'lead' THEN 1 END) AS number_of_leads,
    COUNT(CASE WHEN oc.property_lifecyclestage = 'customer' THEN 1 END) AS number_of_customers
FROM
    events_custom_lead_sources_v2 ecls
JOIN
    objects_contacts oc ON ecls.objectid = oc.objectid  
GROUP BY
    ecls.Property_lead_source_category
ORDER BY
    ecls.Property_lead_source_category;



```
---
### 2. **For Sales Teams**

**Revenue Amount for each Contact who have associated Deals:**
```sql
SELECT
    C.objectid as ContactID, 
    C.property_firstname as FirstName,
    C.property_lastname as LastName,
    D.property_dealname as Deal,
    D.property_amount as Amount,
    D.property_closedate as DateClosed
FROM
    V2_LIVE.OBJECTS_DEALS D INNER JOIN
    V2_LIVE.ASSOCIATIONS_DEALS_TO_CONTACTS DC ON D.OBJECTID = DC.DEAL_OBJECTID INNER JOIN
    V2_LIVE.OBJECTS_CONTACTS as C On C.OBJECTID = DC.CONTACT_OBJECTID
WHERE
DATE_PART (YEAR,C.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
AND D.property_closedate IS NOT NULL AND D.property_amount IS NOT NULL
ORDER BY D.property_closedate DESC

```

**List Custom Event SaaS platform interactions:**
```sql
SELECT
    ce.occurredat,
    c.property_firstname,
    c.property_lastname, 
    ce.property_hs_browser,
    ce.property_hs_device_type,
    ce.property_software_version
FROM 
    V2_LIVE.EVENTS_SOFTWARE_USAGE_EVENT ce JOIN
    V2_LIVE.OBJECTS_CONTACTS c On c.objectid = ce.objectid
WHERE
    DATE_PART (YEAR,c.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
ORDER BY ce.occurredat DESC


```

**Calculate hierarchical Revenue for Companies:**
```sql
WITH CTE (CompanyID, ParentCompanyID, ChildCompanyID, TotalRevenue) 
as (
  SELECT 
    ac.property_unparsed_hs_object_id as CompanyID, 
    ac.property_unparsed_hs_parent_company_id as ParentCompanyID, 
    null as ChildCompanyID, coalesce(ac.property_total_revenue,0) as TotalRevenue
  FROM V2_LIVE.OBJECTS_COMPANIES ac
  WHERE ac.property_name LIKE '%HubSpot%'

  UNION ALL

  SELECT 
    ac.property_unparsed_hs_object_id as CompanyID, 
    ac.property_unparsed_hs_parent_company_id as ParentCompanyID, 
    v.ChildCompanyID as ChildCompanyID, 
    coalesce(v.TotalRevenue,0) as TotalRevenue
  FROM V2_LIVE.OBJECTS_COMPANIES ac INNER JOIN CTE v on ac.property_unparsed_hs_object_id = v.ParentCompanyID 
  WHERE ac.property_name LIKE '%HubSpot%'
)
SELECT 
    v.CompanyID as id, 
    ac.property_name as name, 
    v.TotalRevenue as revenue
FROM (SELECT CompanyID, sum(TotalRevenue) as TotalRevenue FROM CTE GROUP BY CompanyID) v
INNER JOIN V2_LIVE.OBJECTS_COMPANIES ac on v.CompanyID=ac.property_unparsed_hs_object_id 
WHERE ac.property_name LIKE '%HubSpot%'



```


**Calculate Customer Retention Rate:**
```sql
SELECT 
    'NEW MEMBER COHORT' as cohort, 
    COUNT(DISTINCT customer_id) AS total_customers, 
    COUNT(DISTINCT CASE WHEN month(purchase_date) > month(first_purchase_date) THEN customer_id ELSE NULL END) AS retained_customers, 
    ROUND((COUNT(DISTINCT CASE WHEN month(purchase_date) > month(first_purchase_date) THEN customer_id ELSE NULL END) * 100.0) / COUNT(DISTINCT customer_id), 2) AS retention_rate

FROM (

    WITH first_deal_created_date AS (
      SELECT
        C.OBJECTID as customer_id, MIN(TO_DATE(D.PROPERTY_CREATEDATE)) AS first_deal_created_date
      FROM
        V2_LIVE.OBJECTS_DEALS D INNER JOIN
        V2_LIVE.ASSOCIATIONS_DEALS_TO_CONTACTS DC ON D.OBJECTID = DC.DEAL_OBJECTID INNER JOIN
        V2_LIVE.OBJECTS_CONTACTS as C On C.OBJECTID = DC.CONTACT_OBJECTID
      WHERE
        DATE_PART (YEAR,C.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
      GROUP BY customer_id
    )

    SELECT 
        C.OBJECTID as customer_id, 
        C.PROPERTY_LASTNAME as customer_lastname,  
        x.first_deal_created_date as first_purchase_date, 
        TO_DATE(D.PROPERTY_CREATEDATE) as purchase_date

    FROM 
    V2_LIVE.OBJECTS_DEALS as D INNER JOIN 
    V2_LIVE.ASSOCIATIONS_DEALS_TO_CONTACTS DC ON D.OBJECTID = DC.DEAL_OBJECTID INNER JOIN
    V2_LIVE.OBJECTS_CONTACTS as C On C.OBJECTID = DC.CONTACT_OBJECTID INNER JOIN 
    first_deal_created_date as x On x.customer_id = C.OBJECTID

) AS customer_purchases

GROUP BY cohort;


```

**Rep Ranking by Product & Margin:**
```sql
WITH RevenuePerRep AS (
    SELECT
        LI.property_name AS Product_Name,
        D.PROPERTY_HUBSPOT_OWNER_ID AS Rep_Id,
        SUM(LI.PROPERTY_AMOUNT) AS total_revenue
    FROM
        OBJECTS_DEALS D
    JOIN
        ASSOCIATIONS_DEALS_TO_LINE_ITEMS Assoc ON D.OBJECTID = Assoc.DEAL_OBJECTID
    JOIN
        OBJECTS_LINE_ITEMS LI ON Assoc.LINE_ITEM_OBJECTID = LI.OBJECTID
    WHERE
        D.property_hs_is_closed_won = 'true'
        AND EXTRACT(YEAR FROM D.property_closedate) = EXTRACT(YEAR FROM CURRENT_DATE)
    GROUP BY
        LI.property_name, D.PROPERTY_HUBSPOT_OWNER_ID
)
SELECT
    Product_Name,
    Rep_Id,
    total_revenue,
    RANK() OVER (PARTITION BY Product_Name ORDER BY total_revenue DESC) AS rep_rank
FROM
    RevenuePerRep


```
---
### 3. **For Customer Success**

**Cumulative Time in Stage/Status for Service/Sales Teams:**
```sql
with t1 as (
    select
        *,
        NVL(
            LEAD(updatedat) OVER(
                PARTITION BY objectid
                ORDER BY
                    updatedat
            ),
            current_date()
        ) AS next_change_date,
        DATEDIFF(minutes, updatedat, next_change_date) as minutes_spent_in_stage,
        DATEDIFF(hours, updatedat, next_change_date) as hours_spent_in_stage,
        DATEDIFF(days, updatedat, next_change_date) as days_spent_in_stage
    FROM
        V2_LIVE.OBJECT_PROPERTIES_HISTORY
    WHERE
        name = 'hs_pipeline_stage'
        --group by objectid
    order by
        2,
        5
)
select
    objectid,
    value,
    sum(minutes_spent_in_stage) minutes_in_stage,
    sum(hours_spent_in_stage) hours_in_stage,
    sum(days_spent_in_stage) days_in_stage
from
    t1
where
    value not in ('closedwon', 'closedlost')
group by
    1,
    2
order by
    1,
    2

```

**Report on how many inquiries are coming into a connected Sales/ Support Inbox:**
```sql
select 
    DATE_TRUNC('MONTH', ingestedat) AS Month,
    COUNT(*) AS EnquiryCount
FROM
    objects_engagements
WHERE 
    property_hs_engagement_type = 'INCOMING_EMAIL' and property_hs_email_to_email = 'SalesEnquiries@21250579.hs-inbox.com'
GROUP BY 
    Month
ORDER BY
    MONTH


```

**Detecting Inactive Platform Users:**
```sql
SELECT 
    c.objectid,
    c.property_email,
    COUNT(e.id) AS recent_activities
FROM 
    OBJECTS_CONTACTS c
LEFT JOIN 
    EVENTS_SOFTWARE_USAGE_EVENT e ON c.objectid = e.objectid
WHERE 
    e.occurredat > CURRENT_TIMESTAMP - INTERVAL '90 DAY'
GROUP BY 
    c.objectid, c.property_email
HAVING 
    recent_activities < 3
ORDER BY 
    recent_activities;



```

**Rep Availability:**
```sql
with t1 as (
    select
        *,
        NVL(
            LEAD(updatedat) OVER(
                PARTITION BY objectid
                ORDER BY
                    updatedat
            ),
            current_date()
        ) AS next_change_date,
        DATEDIFF(minutes, updatedat, next_change_date) as minutes_spent_in_stage,
        DATEDIFF(hours, updatedat, next_change_date) as hours_spent_in_stage,
        DATEDIFF(days, updatedat, next_change_date) as days_spent_in_stage
    FROM
        V2_LIVE.OBJECT_PROPERTIES_HISTORY
    WHERE
        name = 'hs_pipeline_stage'
        --group by objectid
    order by
        2,
        5
)
select
    objectid,
    value,
    sum(minutes_spent_in_stage) minutes_in_stage,
    sum(hours_spent_in_stage) hours_in_stage,
    sum(days_spent_in_stage) days_in_stage
from
    t1
where
    value not in ('closedwon', 'closedlost')
group by
    1,
    2
order by
    1,
    2



```

**Spot Companies with High Support Ticket Volume:**
```sql
SELECT 
    c.PROPERTY_NAME AS company_name,
    COUNT(t.PROPERTY_HS_TICKET_ID) AS tickets,
    AVG(t.PROPERTY_HS_TIME_TO_CLOSE_IN_OPERATING_HOURS) AS avg_resolution_time,
    MAX(t.PROPERTY_HS_TICKET_PRIORITY) AS max_priority
FROM 
    OBJECTS_TICKETS t
JOIN 
    ASSOCIATIONS_TICKETS_TO_COMPANIES atc
ON 
    t.OBJECTID = atc.TICKET_OBJECTID 
JOIN 
    OBJECTS_COMPANIES c
ON 
    atc.COMPANY_OBJECTID = c.OBJECTID 
GROUP BY 
    c.PROPERTY_NAME
HAVING 
    COUNT(t.PROPERTY_HS_TICKET_ID) > 1
    OR AVG(t.PROPERTY_HS_TIME_TO_CLOSE_IN_OPERATING_HOURS) > 12;




```

---
### 4. **For Data Teams**

**Get Top Properties with Changes in Last 7 Days:**
```sql
select
name
, count(iff(datediff(day, updatedat, current_date) <= 7, value, null)) changes_l7d,
count(iff(datediff(day, updatedat, current_date) <= 30, value, null)) changes_l30d,
count(iff(datediff(day, updatedat, current_date) <= 90, value, null)) changes_l90d,
from object_properties_history
group by 1
order by 2 desc
limit 10
```

**List Applications Associated with a Contact:**
```sql
SELECT 
    a.property_application_date,
    a.property_type,
    a.property_status,
    c.property_firstname, c.property_lastname
FROM 
    OBJECTS_APPLICATIONS a Join
    ASSOCIATIONS_APPLICATIONS_TO_CONTACTS ac On a.objectid = ac.application_objectid Join
    OBJECTS_CONTACTS c On c.objectid = ac.contact_objectid
WHERE
DATE_PART (YEAR,C.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
AND a.property_status IS NOT NULL
ORDER BY a.property_application_date DESC

```

**List Creation by Users/Owners Properties:**
```sql
SELECT 'Lists' As Asset, BU, COUNT(*) FROM (
SELECT  
    u.property_primary_business_unit As BU, 
    u.property_hs_email As Email, 
    l.property_hs_list_name As AssetName 
FROM 
    V2_LIVE.OBJECTS_USERS u JOIN
    V2_LIVE.OBJECTS_LISTS l ON l.property_hs_created_by_user_id = u.property_unparsed_hs_internal_user_id
WHERE l.property_hs_created_by_user_id > 0 and l.property_hs_is_read_only = false and l.property_hs_is_public = true
) seq 
GROUP BY BU

```

**Copy Contacts as CSV to AWS S3:**
```sql
COPY INTO 's3://hs-rep-insights-data-v1/hs-contacts.csv'
FROM (
    SELECT 
        c.OBJECTID AS ID,
        c.PROPERTY_EXTERNAL_ID AS USER_ID,
        c.PROPERTY_FIRSTNAME AS FIRSTNAME,
        c.PROPERTY_LASTNAME AS LASTNAME,
        c.PROPERTY_EMAIL AS EMAIL
    FROM
    V2_DAILY.OBJECTS_CONTACTS c
    WHERE 
    LENGTH(c.PROPERTY_EXTERNAL_ID) > 0 AND
    DATE_PART (YEAR,c.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
  )
  CREDENTIALS = (AWS_KEY_ID='AWSKEY' AWS_SECRET_KEY='AWSSECRETKEY')
  FILE_FORMAT = (TYPE = CSV COMPRESSION = NONE NULL_IF=() PARSE_HEADER = TRUE)
  SINGLE = TRUE
  OVERWRITE = TRUE
  HEADER = TRUE

```

**Cross-Portal Reporting Example:**
```sql
(
WITH closed_tickets AS (
  SELECT
    COUNT(DISTINCT objectid) AS closed_ticket_count
  FROM
    OBJECTS_TICKETS
  WHERE
    PROPERTY_CLOSED_DATE IS NOT NULL
),
total_tickets AS (
  SELECT
    COUNT(DISTINCT objectid) AS total_ticket_count
  FROM
    objects_tickets
)

SELECT
  'Company A' As Portal,
  closed_tickets.closed_ticket_count,
  total_tickets.total_ticket_count,
  (
    CAST(closed_tickets.closed_ticket_count AS FLOAT) / CAST(total_tickets.total_ticket_count AS FLOAT)
  ) * 100 AS ticket_closure_rate
FROM
  closed_tickets,
  total_tickets
)
UNION ALL
(
WITH closed_tickets_2 AS (
  SELECT
    COUNT(DISTINCT objectid) AS closed_ticket_count
  FROM
    HUB_23826378_DB.V2_DAILY.OBJECTS_TICKETS t
  WHERE
    t.PROPERTY_CLOSED_DATE IS NOT NULL
),
total_tickets_2 AS (
  SELECT
    COUNT(DISTINCT objectid) AS total_ticket_count
  FROM
    HUB_23826378_DB.V2_DAILY.OBJECTS_TICKETS t
)
SELECT
  'Company B' As Portal,
  closed_tickets_2.closed_ticket_count,
  total_tickets_2.total_ticket_count,
  (
    CAST(closed_tickets_2.closed_ticket_count AS FLOAT) / CAST(total_tickets_2.total_ticket_count AS FLOAT)
  ) * 100 AS ticket_closure_rate
FROM
  closed_tickets_2,
  total_tickets_2
)
```
---
### 5. **For Content Marketing**

**List Custom Event E-Commerce interactions for Contacts:**
```sql
SELECT
    ce.occurredat,
    c.property_firstname As FirstName,
    c.property_lastname As LastName, 
    ce.property_event_type As EventType,
    ce.property_item_id As ItemID,
    ce.property_item_name As ItemName,
    ce.property_item_category_l1 As ItemCategory,
    ce.property_item_price As ItemPrice
FROM 
    EVENTS_AWS_ECOMMERCE ce JOIN
    OBJECTS_CONTACTS c On c.objectid = ce.objectid
WHERE
    DATE_PART (YEAR,c.property_createdate) = DATE_PART (YEAR, CURRENT_DATE)
ORDER BY ce.occurredat DESC

```

**Page View Trend By URL with Drilldowns:**
```sql
SELECT
  property_hs_base_url,
  SUM(CASE WHEN occurredat >= CURRENT_DATE - INTERVAL '7 DAYS' THEN 1 ELSE 0 END) AS view_count_last_7_days,
  SUM(CASE WHEN occurredat >= CURRENT_DATE - INTERVAL '30 DAYS' THEN 1 ELSE 0 END) AS view_count_last_30_days,
  SUM(CASE WHEN occurredat >= CURRENT_DATE - INTERVAL '90 DAYS' THEN 1 ELSE 0 END) AS view_count_last_90_days
FROM
  events_visited_page
GROUP BY
  property_hs_base_url
ORDER BY
  view_count_last_7_days DESC
```

**Most Visited Pages with UTM Breakdown:**
```sql
SELECT 
    PROPERTY_HS_URL AS page_url,
    PROPERTY_HS_UTM_SOURCE AS utm_source,
    PROPERTY_HS_UTM_MEDIUM AS utm_medium,
    COUNT(*) AS visit_count
FROM events_visited_page
GROUP BY PROPERTY_HS_URL, PROPERTY_HS_UTM_SOURCE, PROPERTY_HS_UTM_MEDIUM
ORDER BY visit_count DESC
LIMIT 10;
```

**Top Ranked Web Pages by Hour:**
```sql
SELECT 
    DATE_TRUNC('HOUR', OCCURREDAT) AS event_hour,
    PROPERTY_HS_URL AS page_url,
    COUNT(*) AS total_pageviews,
    RANK() OVER (PARTITION BY DATE_TRUNC('HOUR', OCCURREDAT) ORDER BY COUNT(*) DESC) AS page_rank
FROM events_visited_page
GROUP BY event_hour, page_url
ORDER BY event_hour, page_rank;

```

**Journey Reports for Time Between Custom Events:**
```sql
SELECT DISTINCT
    c.OBJECTID AS contact_id,
    o.OCCURREDAT AS oauth_login_time,
    s.OCCURREDAT AS software_usage_time,
    TIMESTAMPDIFF(MINUTE, o.OCCURREDAT, s.OCCURREDAT) AS time_between_events_seconds
FROM 
    objects_contacts c
JOIN 
    events_oauth_login o ON c.objectid = o.OBJECTID
JOIN 
    events_software_usage_event s ON c.objectid = s.OBJECTID
WHERE 
    o.EVENTTYPEID = '6-12031101' 
    AND s.EVENTTYPEID = '6-12029431'
    AND o.OCCURREDAT < s.OCCURREDAT
ORDER BY 
    contact_id, oauth_login_time;

```

---

## Getting Started

### 1. **Install Snowflake Data Share**
Follow our step-by-step guide to integrate HubSpot data into Snowflake with just a few clicks. Resources include setup videos and documentation for seamless activation.

### 2. **Explore Queries by Use Case**
This repository includes beginner, intermediate, and advanced SQL queries to meet diverse business needs. Review the annotated queries for detailed explanations and actionable insights.

### 3. **Iterate and Scale**
Leverage these queries as templates to create custom reports tailored to your organization’s KPIs. Use them as building blocks to address industry-specific challenges and scale your data-driven strategies.

---

## Advanced Features

### Reverse ETL Integration
Bring data back into HubSpot from Snowflake for smarter segmentation, automation, and personalization. Example use cases include:
- Dynamic company hierarchy revenue tracking.
- Enhanced customer targeting and campaign personalization.

---

## Conclusion
HubSpot’s Snowflake Data Share empowers scaling businesses to unlock the value of their data by enabling unified, real-time reporting across teams. Whether you are driving marketing ROI, improving sales accuracy, or enhancing customer retention, this repository provides everything you need to activate and maximize value.

### Next Steps
1. **Activate the Snowflake Data Share.**
2. **Run sample queries to jumpstart your reporting.**
3. **Iterate and scale your analytics for greater impact.**

---

## FAQ

**Why Snowflake?**
Snowflake’s powerful data sharing capabilities allow you to centralize your data without incurring additional storage costs.

**What makes this better than API integrations?**
With proprietary HubSpot tables and near real-time updates, the Snowflake Data Share offers unique capabilities and faster insights.

**What resources are available for activation?**
This repository includes guides, demos, and best practices to help your team get started quickly.

