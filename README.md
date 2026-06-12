# Swiggy Customer Churn and Segmentation Pipeline

This project is a small end‑to‑end example of how to create a synthetic Swiggy‑style customer dataset, predict churn in Python, and then use n8n to generate marketing actions for each customer segment.

It is designed as a learning project to show the full flow:
- Data creation and churn logic in Python
- Data quality checks and automated profiling
- Basic churn model and customer segmentation
- Exporting results and orchestrating actions in n8n

---

## 1. Project overview

The goal is to classify customers based on their purchase behaviour and suggest simple marketing actions for each group.

Main ideas:
- Identify customers who look **inactive** (churned).
- Identify **regular / loyal** customers.
- For inactive customers: suggest **coupon offers** to bring them back.
- For regular customers: suggest **thank‑you or loyalty messages**.

The project does not use real Swiggy data. It uses a randomly generated dataset that follows a similar pattern (orders, spend, days since last order, etc.).

---

## 2. Tech stack

- **Python (Google Colab / Jupyter)**  
  - `pandas`, `numpy`, `scikit-learn`
- **ydata_profiling** for automated EDA and data quality report
- **n8n** for workflow automation and marketing logic
- (Optional) Email / WhatsApp integration for sending messages

---

## 3. Python part (data + model)

Files:
- `swiggy_churn_project.ipynb` – main notebook
- `swiggy_customer_data.csv` – raw synthetic data
- `swiggy_customer_segments_with_actions.csv` – final data with segments and actions
- `swiggy_customer_data_profile.html` – profiling report

Steps in the notebook:

1. **Generate synthetic customer data**

   - Create 500 customers with:
     - `age`
     - `city`
     - `orders_last_30_days`
     - `avg_order_value`
     - `days_since_last_order`
     - `coupon_used_last_month`
     - `customer_support_calls`
   - Each row represents one customer.

2. **Define churn rule**

   - A simple rule marks a customer as churned (`churn = 1`) if:
     - `days_since_last_order > 45` **and**
     - `orders_last_30_days < 5`
   - Otherwise, `churn = 0`.

3. **Check data quality**

   - Check shape of the data.
   - Check for missing values.
   - Check for duplicate customer IDs.
   - Look at basic statistics to see if values are reasonable.

4. **Generate data profiling report**

   - Use `ydata_profiling` to create `swiggy_customer_data_profile.html`.
   - Open this file in a browser to view distributions, correlations, and possible data issues.

5. **Train a simple churn model**

   - Split the data into **train** and **test** sets.
   - Use a **Decision Tree Classifier** to predict `churn`.
   - Evaluate the model on the test set.

6. **Add segments and actions**

   - Use the model and rules to create:
     - `customer_segment`:
       - `Inactive (Churned)`
       - `Regular (Non-churned)`
     - `marketing_action`:
       - For inactive: “Send coupon on previous purchase category”
       - For regular: “Send thank‑you message and loyalty reward”
   - Save the final result as `swiggy_customer_segments_with_actions.csv`.

---

## 4. n8n part (workflow automation)

Files:
- `swiggy_n8n_workflow.json` – exported n8n workflow

High‑level workflow:

1. **Read customer data**
   - Upload `swiggy_customer_segments_with_actions.csv` to Google Drive.
   - In n8n, use a **Google Drive** node to download the CSV.
   - Use **Extract From CSV** to convert the file into items (one item per customer).

2. **Add advanced segmentation logic (Code node)**

   - Use a **Code** node (JavaScript) to apply a special rule:
     - If customer is `Inactive (Churned)`, has `avg_order_value > 1000` and `days_since_last_order > 60`:
       - `special_segment = "VIP_Dormant"`
       - `special_action = "Send 30% coupon + personal apology mail"`
     - Else if `Inactive (Churned)`:
       - `special_segment = "Normal_Inactive"`
       - `special_action = "Send 10% coupon"`
     - Else (regular):
       - `special_segment = "Regular"`
       - `special_action = "Send thank-you + loyalty points"`

3. **Build final message**

   - Use a **Set** node (or the Code node) to create:
     - `final_message` text, for example:  
       `Hi customer 10001, segment: Regular, action: Send thank-you + loyalty points`
   - This message is what would be sent via WhatsApp or email.

4. **(Optional) Send notifications**

   - The workflow is ready to be connected to:
     - A **Send Email** node (Gmail/SMTP), or
     - A **WhatsApp Cloud API** node.
   - In this project, the focus is on generating the message and logic, not on production email/WhatsApp setup.

---

## 5. How to run this project

### 5.1 Run Python notebook

1. Open `swiggy_churn_project.ipynb` in Google Colab or Jupyter.
2. Install dependencies if needed:
   ```python
   !pip install ydata_profiling
   ```
3. Run the cells in order.
4. Confirm that these files are created:
   - `swiggy_customer_data.csv`
   - `swiggy_customer_segments_with_actions.csv`
   - `swiggy_customer_data_profile.html`

### 5.2 View profiling report

1. Open `swiggy_customer_data_profile.html` in a web browser.
2. Explore variable stats, missing values, and correlations.

### 5.3 Import and run n8n workflow

1. Open your n8n instance.
2. Create a new workflow → Import → select `swiggy_n8n_workflow.json`.
3. Upload `swiggy_customer_segments_with_actions.csv` to your Google Drive.
4. Update the **Google Drive** node to point to your file.
5. Execute the workflow:
   - Check the Code node output for `special_segment` and `special_action`.
   - Check the final Set node for `final_message`.

---

## 6. What this project demonstrates

- Basic churn and customer segmentation using purchase history.
- Simple but clear data quality checks and automated profiling.
- Integration of model output into a workflow automation tool (n8n).
- Use of custom logic in n8n (JavaScript Code node) to define unique marketing rules.
- Generation of personalized messages that can be used for email or WhatsApp campaigns.

This project is suitable to show in a portfolio or as an academic assignment to demonstrate understanding of:
- Data preparation
- Churn logic and classification
- Automated EDA
- Workflow automation and basic marketing automation logic
