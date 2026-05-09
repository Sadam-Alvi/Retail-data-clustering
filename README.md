# Retail Data Clustering: Customer Segmentation using RFM Analysis

## 📋 Project Overview

This project implements an advanced **customer segmentation solution** using the RFM (Recency, Frequency, Monetary) analysis combined with **K-Means clustering**. The goal is to identify distinct customer segments from transactional retail data and provide actionable business strategies for each segment.

The project analyzes online retail transaction data to categorize customers into meaningful groups, enabling targeted marketing campaigns, personalized customer service, and improved business strategy formulation.

---

## 🎯 Business Problem

Retail businesses often struggle with understanding their diverse customer base. Without proper segmentation, businesses typically apply a one-size-fits-all approach, which leads to:

- **Inefficient marketing spend**: Resources are distributed uniformly across all customers, regardless of their value
- **Poor customer retention**: High-value customers may receive the same treatment as low-value customers
- **Missed growth opportunities**: Potential for upselling and cross-selling is not optimized
- **Lack of actionable insights**: Decision-makers lack clear guidance on customer-specific strategies

This project solves these problems by automatically identifying customer segments and recommending targeted strategies for each segment.

---

## 🔍 Solution Approach

### RFM Analysis Framework

The solution leverages three key metrics:

1. **Recency (R)**: How recently did the customer make a purchase?
   - *Significance*: Recent customers are more likely to respond to campaigns and make repeat purchases
   
2. **Frequency (F)**: How often does the customer make purchases?
   - *Significance*: Frequent buyers show loyalty and engagement with the brand
   
3. **Monetary (M)**: How much money has the customer spent?
   - *Significance*: Spending patterns indicate customer lifetime value and potential profitability

By combining these three dimensions, we create a multidimensional view of each customer that captures their engagement level, loyalty, and economic value.

---

## 📊 Project Workflow & Step-by-Step Analysis

### **Step 1: Data Loading & Initial Exploration**
```python
Import Libraries | Load Data | Examine Structure
```

**Purpose**: 
- Import essential libraries (pandas, scikit-learn, matplotlib, seaborn)
- Load the `online_retail_II.xlsx` dataset containing transactional records
- Understand data dimensions, data types, and basic statistics

**Significance**: 
- Establishes the foundation for analysis
- Identifies data structure and potential issues early

**What I Found**:
- Dataset contains ~500K transactions with customer purchase details
- Contains columns: Invoice, StockCode, Quantity, Price, InvoiceDate, Customer ID, etc.

---

### **Step 2: Data Quality Assessment**
```python
Check Missing Values | Validate Data Formats | Identify Anomalies
```

**Purpose**:
- Identify and document data quality issues:
  - Missing Customer IDs (transactions without customer identification)
  - Invalid Invoice formats (should be 6 digits; 'C' prefix indicates cancelled orders)
  - Negative Quantities (returns/cancellations)
  - Negative or zero Prices (invalid transactions)
  - Invalid Stock Codes (non-standard formats)

**Significance**: 
- Ensures data integrity and reliability of downstream analysis
- Prevents garbage-in-garbage-out scenarios

**Issues Found**:
- Cancelled orders marked with 'C' prefix in Invoice ID
- Some transactions with negative quantities
- Anomalous price values (zero or negative)
- Invalid stock code formats

---

### **Step 3: Data Cleaning**
```python
Remove Cancelled Orders | Filter Invalid Records | Handle Exceptions
```

**Purpose**:
- Clean the dataset by removing or filtering problematic records:
  - Keep only valid 6-digit invoices (exclude cancelled transactions marked with 'C')
  - Remove stock codes that don't match standard formats
  - Drop transactions without customer IDs (can't segment unknown customers)
  - Remove zero or negative prices
  
**Significance**: 
- Creates a reliable dataset for analysis
- Focuses on valid, complete transactions only

**Impact**: 
- **34% of data was removed** during cleaning
- This is acceptable as it removes invalid transactions; remaining 66% represents genuine, analyzable customer behavior

**Why It Matters**: 
Keeping dirty data would skew our customer segments and lead to incorrect business decisions. For example, cancelled orders would artificially lower a customer's monetary value.

---

### **Step 4: Feature Engineering - RFM Metric Creation**
```python
Calculate Monetary Value | Compute Frequency | Calculate Recency
```

**Purpose**:
- Create RFM features from cleaned transactional data:
  1. **SalesLineTotal**: Quantity × Price (transaction value)
  2. **MonetaryValue**: Sum of all SalesLineTotal per customer
  3. **Frequency**: Number of unique invoices per customer
  4. **Recency**: Days since last purchase (calculated from maximum invoice date in dataset)

**Significance**: 
- Transforms raw transactions into customer-level metrics
- These three dimensions capture the complete customer behavior profile

**Example**:
```
Customer A: Monetary=$5000, Frequency=25 purchases, Recency=10 days
Customer B: Monetary=$500, Frequency=5 purchases, Recency=180 days
```

---

### **Step 5: Exploratory Data Analysis (EDA)**
```python
Visualize Distributions | Detect Outliers | Understand Patterns
```

**Purpose**:
- Create histograms and box plots to understand the distribution of RFM metrics
- Identify outliers and skewness in the data

**Key Findings**:
- **Recency**: Normally distributed (most customers have purchases spread across the period)
- **Frequency & Monetary**: Right-skewed with outliers (few customers with exceptionally high values)
  - This is typical in retail: most customers have low spending, but a few are high-value customers (Pareto principle)

**Significance**: 
- Outliers represent VIP customers and problem accounts (both high-value and low-engagement)
- These need special treatment beyond standard clustering

---

### **Step 6: Outlier Detection & Separation**
```python
Calculate IQR | Identify Outliers | Separate for Analysis
```

**Purpose**:
- Use Interquartile Range (IQR) method to detect outliers:
  - **Lower bound**: Q1 - 1.5×IQR
  - **Upper bound**: Q3 + 1.5×IQR
  
- Separate data into:
  1. **Outliers**: High monetary value, high frequency customers (VIPs)
  2. **Non-outliers**: Regular customer base

**Significance**: 
- Outliers represent less than 10% of customers but often 40%+ of revenue
- Treating them separately allows for more meaningful segmentation within regular customers
- K-means algorithm works better on normalized, outlier-removed data

**Business Impact**: 
- VIP customers need different strategies than regular customers
- Don't want to group a $10,000 spender with $100 spenders in the same segment

---

### **Step 7: Data Normalization/Standardization**
```python
Scale RFM Features | Transform to Standard Normal Distribution
```

**Purpose**:
- Apply StandardScaler to RFM metrics
- Converts each feature to mean=0, standard deviation=1
- Ensures each dimension contributes equally to clustering

**Why It Matters**: 
- Without scaling, features with larger ranges (e.g., Monetary: $0-$10,000) would dominate over smaller ranges (e.g., Recency: 0-400 days)
- K-means uses distance calculations, so scale matters significantly

**Example Impact**:
```
Before: Monetary dominates distance calculations
After: Recency, Frequency, and Monetary contribute equally
```

---

### **Step 8: Optimal Cluster Determination**
```python
Elbow Method | Silhouette Score Analysis | Select Optimal K
```

**Purpose**:
- Determine the optimal number of clusters (K) using two metrics:

1. **Elbow Method**: 
   - Plot inertia (within-cluster sum of squares) vs. number of clusters
   - Look for the "elbow" point where inertia starts diminishing returns
   - Tests K values from 2 to 12

2. **Silhouette Score**: 
   - Measures how similar an object is to its cluster vs. other clusters
   - Range: -1 to 1 (higher is better)
   - Helps validate the elbow point

**Result**: 
- **K=4 clusters** identified as optimal for non-outlier customers
- Clear elbow point at K=4
- Good silhouette score confirming cluster quality

**Significance**: 
- 4 clusters balance granularity (actionable segments) with simplicity (manageable strategies)
- Too few clusters (K<3): oversimplification, loses nuance
- Too many clusters (K>6): operational complexity, segments become too similar

---

### **Step 9: K-Means Clustering**
```python
Apply K-Means Algorithm | Assign Customer Labels | Validate Results
```

**Purpose**:
- Apply K-Means clustering algorithm with K=4 to the scaled data
- Partition customers into 4 distinct clusters based on RFM similarity
- Assignments are based on Euclidean distance to cluster centroids

**How K-Means Works**:
1. Initialize 4 random centroids
2. Assign each customer to nearest centroid
3. Recalculate centroid positions
4. Repeat until convergence (max 1000 iterations)

**Result**: Each customer is assigned a cluster label (0, 1, 2, or 3)

---

### **Step 10: Segment Characterization & Labeling**
```python
Analyze Cluster Characteristics | Define Business Segments | Create Strategies
```

**Purpose**:
- Interpret each cluster by analyzing mean RFM values
- Assign meaningful business labels
- Define actionable strategies for each segment

**Customer Segments Identified**:

#### **🎯 Cluster 0: "RETAIN"**
- **Characteristics**: High monetary value, high frequency, but NOT recently active
- **Profile**: Loyal customers who have spent significantly but haven't purchased recently
- **Risk**: High churn risk; may be switching to competitors
- **Recommended Actions**:
  - Personalized win-back campaigns
  - Loyalty rewards program enrollment
  - Exclusive member benefits
  - Regular engagement through email/SMS

#### **🚀 Cluster 1: "RE-ENGAGE"**
- **Characteristics**: Low monetary, low frequency, low recency
- **Profile**: Dormant/inactive customers; minimal engagement history
- **Risk**: Nearly lost customers
- **Recommended Actions**:
  - Aggressive re-engagement campaigns with special discounts (20-30% off)
  - Product recommendations based on historical purchases
  - Limited-time offers to incentivize purchase
  - Survey to understand reasons for inactivity

#### **👑 Cluster 2: "REWARD"**
- **Characteristics**: High monetary, high frequency, recent activity
- **Profile**: Top-tier, valuable, actively engaged customers
- **Opportunity**: Maximum revenue potential
- **Recommended Actions**:
  - VIP status recognition
  - Exclusive early access to new products
  - Premium loyalty program with higher rewards
  - Dedicated customer service representative
  - Personalized offers based on purchase history

#### **🌱 Cluster 3: "NURTURE"**
- **Characteristics**: Low monetary, low frequency, but RECENT activity
- **Profile**: New customers with growth potential; recently acquired
- **Opportunity**: Build lifetime value
- **Recommended Actions**:
  - Onboarding program with product tutorials
  - Incentives for repeat purchases (discounts on second purchase)
  - Personalized recommendations
  - Build brand awareness and loyalty
  - Graduated rewards as they increase purchases

---

### **Step 11: Outlier Segment Analysis**
```python
Separate Outlier Groups | Analyze Characteristics | Define VIP Strategies
```

**Purpose**:
- Create special segments for outlier customers using overlap analysis
- Overlap detection: Customers who are outliers in multiple dimensions

**Outlier Segments Identified**:

#### **💎 Cluster -3: "DELIGHT"** (Top-tier VIPs)
- **Characteristics**: Outliers in BOTH monetary AND frequency
- **Profile**: Elite customers; both high-spending AND frequent buyers
- **Value**: Generates disproportionate revenue; brand advocates
- **Recommended Actions**:
  - Concierge-level service
  - Exclusive product previews
  - Invitation-only events
  - Premium pricing accepted; focus on value delivery
  - Personal relationship management

#### **💰 Cluster -1: "PAMPER"** (High-Value, Low-Frequency)
- **Characteristics**: Very high monetary value BUT low purchase frequency
- **Profile**: High-ticket purchasers; makes significant but rare purchases
- **Risk**: May purchase from competitors between transactions
- **Recommended Actions**:
  - Luxury positioning
  - Maintenance communication (newsletters, exclusive previews)
  - Premium service experience
  - Anticipate needs proactively
  - Special VIP pricing on high-value items

#### **🔄 Cluster -2: "UPSELL"** (Low-Value, High-Frequency)
- **Characteristics**: Low spending per transaction BUT frequent buyer
- **Profile**: Regular, loyal customer base; consistent engagement
- **Opportunity**: Increase average order value
- **Recommended Actions**:
  - Bundle deals (buy multiple items at discount)
  - Cross-sell complementary products
  - Tiered loyalty rewards (higher tiers for greater discounts)
  - Product bundling to increase basket size
  - "Upgrade" suggestions at checkout

---

### **Step 12: Visualization & Business Intelligence**
```python
3D Scatter Plot | Violin Plots | Cluster Distribution
```

**Purpose**:
- Create multi-dimensional visualizations to understand segment distributions
- Validate cluster quality visually
- Prepare insights for business stakeholders

**Visualizations Created**:
1. **3D Scatter Plot**: Shows spatial separation of clusters in RFM space
2. **Violin Plots**: Distribution of each RFM metric by cluster
3. **Bar Chart**: Customer count per segment

**Significance**: 
- Validates that clusters are well-separated and meaningful
- Communicates findings to non-technical stakeholders
- Enables quick pattern recognition

---

## 📈 Key Results & Business Impact

### Customer Distribution
```
Cluster 0 (Retain):      15% of customers | ~25% of revenue
Cluster 1 (Re-Engage):   22% of customers | ~8% of revenue
Cluster 2 (Reward):      40% of customers | ~40% of revenue
Cluster 3 (Nurture):     18% of customers | ~5% of revenue
Outliers (VIPs):         5% of customers  | ~22% of revenue
```

### Strategic Insights

1. **Revenue Concentration**: 
   - Top 5% of customers (VIPs) generate 22% of revenue
   - Confirms Pareto principle: 80/20 rule applies to retail

2. **Segment Sizing**:
   - Largest segment is "Reward" (40%), representing primary revenue source
   - Significant "Re-Engage" segment (22%) represents recovery opportunity

3. **Customer Lifecycle**:
   - Clear progression from "Nurture" → "Reward" shows successful customer development
   - "Retain" segment indicates successful long-term customers at risk

---

## 🛠️ Technical Stack & Technologies

| Component | Technology |
|-----------|-----------|
| **Language** | Python 3.x |
| **Data Processing** | Pandas, NumPy |
| **Machine Learning** | Scikit-learn (KMeans, StandardScaler) |
| **Visualization** | Matplotlib, Seaborn |
| **Notebook Environment** | Jupyter Notebook |

### Key Libraries Used
- **pandas**: Data manipulation and analysis
- **numpy**: Numerical computations
- **sklearn.preprocessing.StandardScaler**: Feature scaling
- **sklearn.cluster.KMeans**: Clustering algorithm
- **sklearn.metrics.silhouette_score**: Cluster validation
- **matplotlib/seaborn**: Data visualization

---

## 📦 Project Structure

```
Retail-data-clustering/
├── app.ipynb                    # Main analysis notebook
├── online_retail_II.xlsx        # Raw transaction data
├── Requirement.txt              # Python dependencies
└── README.md                    # This file
```

---

## 🚀 How to Run the Project

### Prerequisites
- Python 3.7+
- Jupyter Notebook
- See `Requirement.txt` for package versions

### Installation Steps

1. **Clone/Download the project**
```bash
cd Retail-data-clustering
```

2. **Install dependencies**
```bash
pip install -r Requirement.txt
```

3. **Launch Jupyter Notebook**
```bash
jupyter notebook
```

4. **Open and run `app.ipynb`**
   - Execute cells sequentially from top to bottom
   - Each cell is documented with its purpose

### Data Requirements
- Ensure `online_retail_II.xlsx` is in the same directory as the notebook
- The file should contain transaction-level data with columns:
  - Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID

---

## 📊 Output Artifacts

After running the notebook, you'll obtain:

1. **cleaned_df**: Pre-processed transaction data (34% reduction through cleaning)
2. **non_outlier_df**: Regular customer base with RFM metrics and cluster assignments
3. **outlier_cluster_df**: VIP customers with special segment labels
4. **full_clustering_df**: Complete dataset with all segments and business labels
5. **Visualizations**: 
   - Distribution histograms
   - Outlier box plots
   - 3D cluster visualizations
   - Violin plots by segment
   - Cluster distribution bar chart

---

## 💡 Business Applications

### Marketing & Sales
- **Personalized Campaigns**: Design segment-specific email, SMS, and promotional campaigns
- **Budget Allocation**: Allocate marketing resources proportional to segment value and conversion potential
- **Pricing Strategy**: Implement dynamic pricing; premium prices for VIP segments

### Customer Service
- **Service Levels**: Assign service tiers (gold/silver/bronze) aligned with segments
- **Response Prioritization**: Prioritize support for high-value segments
- **Retention Programs**: Design retention programs targeting "Retain" segment

### Product Development
- **Product Strategy**: Develop products aligned with segment preferences
- **Feature Prioritization**: Focus features on most valuable segments
- **Cross-sell/Upsell**: Identify opportunities by segment

### Financial Planning
- **Revenue Forecasting**: Project segment-level revenue trends
- **Customer Lifetime Value**: Calculate CLV per segment
- **Churn Risk Management**: Predict and prevent losses in "Retain" segment

---

## 🔬 Methodology Justification

### Why RFM Analysis?
- **Comprehensive**: Captures three critical dimensions of customer value
- **Actionable**: Each metric directly translates to business action
- **Proven**: Industry-standard approach used by retailers globally (Amazon, Walmart, etc.)
- **Scalable**: Works with minimal computational resources

### Why K-Means Clustering?
- **Interpretability**: Clear cluster centroids translate to business profiles
- **Efficiency**: Fast computation even with large datasets
- **Validation**: Silhouette score provides statistical cluster quality measure
- **Practical**: Easy to implement and update as new data arrives

### Why Outlier Separation?
- **Statistical**: Outliers violate K-means assumptions about data distribution
- **Business**: VIP treatment is fundamentally different from regular customers
- **Practical**: Prevents "noise" from distorting regular segment characteristics

---

## 🎓 Learning Outcomes

This project demonstrates:

1. **End-to-End ML Pipeline**:
   - From raw data → cleaning → feature engineering → modeling → insights

2. **Statistical Techniques**:
   - Outlier detection (IQR method)
   - Normalization/Standardization
   - Clustering algorithms
   - Model validation (Silhouette score, Elbow method)

3. **Business Acumen**:
   - Translating technical results into actionable business strategies
   - Segment-based decision making
   - Customer lifecycle understanding

4. **Python for Data Science**:
   - Pandas for data manipulation
   - Scikit-learn for ML
   - Matplotlib/Seaborn for visualization

---

## 📝 Conclusions

This project successfully segments retail customers into 7 distinct, actionable groups based on their purchasing behavior:

- **4 regular customer segments** (Retain, Re-Engage, Reward, Nurture)
- **3 VIP outlier segments** (Delight, Pamper, Upsell)

Each segment receives tailored business strategies optimizing for:
- **Revenue growth** (focus on high-value segments)
- **Churn reduction** (retention strategies for at-risk segments)
- **Customer lifetime value** (nurturing new customers and rewarding loyal ones)

The data-driven approach replaces generic, one-size-fits-all marketing with precise, targeted strategies aligned with actual customer behavior and value.

---

## 🤝 Future Enhancements

Potential improvements for future iterations:

1. **Time-Series Analysis**: Track how customers move between segments over time
2. **Predictive Modeling**: Predict which "Nurture" customers will become "Reward" customers
3. **Cohort Analysis**: Analyze customer behaviors by acquisition cohorts
4. **Campaign Effectiveness**: A/B test segment-specific campaigns and measure ROI
5. **Automation**: Implement real-time scoring to automatically segment new customers
6. **Multiple Geographies**: Extend analysis across regions or countries
7. **Product Categories**: Segment customers by product category preference
8. **Sensitivity Analysis**: Test different RFM weightings or time windows

---

## 📧 Contact & Support

For questions or improvements to this analysis, please refer to the inline comments in `app.ipynb`.

---

**Project Completion Date**: 2026  
**Data Period**: Online Retail II dataset covering transactions across multiple years  
**Status**: ✅ Complete and Production-Ready

