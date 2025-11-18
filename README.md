# Udacity A/B Testing — Statistical Significance & CUPED Variance Reduction

This project analyzes an A/B experiment on a Udacity-style course signup page.

The goal is to determine whether a **new experimental page** leads to better user conversion behavior than the **original control page**, using:

- Classical A/B testing with **two-proportion Z-tests**
- **Conversion funnel** analysis (CTR → Enrollments → Payments)
- **CUPED** (Controlled Using Pre-Experiment Data) for variance reduction

---

## 1. Problem Statement

Udacity is testing a new course signup page:

- **Control group**: Users who see the original page.
- **Experiment group**: Users who see the new design.

We want to know:

> Does the experiment page **significantly improve** user behavior at any step of the funnel (clicks, enrollments, or payments), or are observed differences just random noise?

---

## 2. Dataset

The dataset consists of **daily aggregated metrics** for each variant:

| Column        | Description                              |
|---------------|------------------------------------------|
| `Date`        | Day of observation                       |
| `Pageviews`   | Number of visitors who saw the page      |
| `Clicks`      | Number of users who clicked “Enroll”     |
| `Enrollments` | Number of users who completed enrollment |
| `Payments`    | Number of paying users                   |
| `group`       | `"control"` or `"experiment"`            |

Each row corresponds to one day and one group (control/experiment).

---

## 3. Conversion Funnel

We define a simple funnel:

> **Pageviews → Clicks → Enrollments → Payments**

From the raw counts, we compute **conversion rates**:

- **Click-Through Rate (CTR)**  
  \[
  \text{CTR} = \frac{\text{Clicks}}{\text{Pageviews}}
  \]

- **Enrollment Rate**  
  \[
  \text{Enrollment Rate} = \frac{\text{Enrollments}}{\text{Clicks}}
  \]

- **Payment Rate**  
  \[
  \text{Payment Rate} = \frac{\text{Payments}}{\text{Enrollments}}
  \]

These rates allow us to compare behavior between control and experiment **fairly**, even if traffic volumes differ.

---

## 4. A/B Test – Statistical Method

To decide whether observed differences between control and experiment are real or due to chance, we use a **two-proportion Z-test**.

For each metric (e.g., CTR), we have:

- \( p_1 \): proportion in the **experiment** group  
- \( p_2 \): proportion in the **control** group  
- \( n_1, n_2 \): sample sizes (e.g., total pageviews, clicks, or enrollments)

We first compute the **pooled proportion** under the null hypothesis \( H_0 \) (no difference):

\[
p = \frac{x_1 + x_2}{n_1 + n_2}
\]

where \( x_1, x_2 \) are the counts of “successes” in experiment and control.

Then the **Z-score** is:

\[
z = \frac{p_1 - p_2}{\sqrt{p(1 - p)\left(\frac{1}{n_1} + \frac{1}{n_2}\right)}}
\]

The **p-value** is computed from the Z-score and tells us:

> “If there were actually **no difference** between control and experiment, how likely is it to observe a difference at least this large, just by chance?”

- If **p-value < 0.05** → we call it **statistically significant**.
- If **p-value ≥ 0.05** → we treat the difference as **not statistically significant**.

---

## 5. A/B Test Results (Unadjusted)

After computing conversion rates and running Z-tests:

| Metric           | Interpretation (Experiment vs Control)                      |
|------------------|------------------------------------------------------------|
| **CTR**          | p ≈ 0.93 → No significant difference in click-through rate |
| **Enrollment Rate** | p ≈ 0.000008 → Significant improvement in enrollments      |
| **Payment Rate** | p ≈ 0.008 → Significant improvement in payments            |

This suggests that while the **top of the funnel (CTR)** did not change meaningfully, the **mid and bottom funnel** (enrollments and payments) improved for users who reached those stages.

However, we still need to be careful: variability and user-level differences can inflate noise.  
That’s where **CUPED** comes in.

---

## 6. CUPED – Variance Reduction

### 6.1 Intuition

Even with random assignment, users naturally differ in behavior. Some are:

- More active,
- More likely to click,
- More likely to pay.

These pre-existing differences increase **variance**, making it harder to detect real treatment effects.

**CUPED (Controlled Using Pre-Experiment Data)** uses a **pre-experiment covariate** (a variable measured before the test and correlated with the outcome) to remove some of that noise:

- It **does not change the mean treatment effect**,
- It **reduces variance**, improving test sensitivity.

Conceptually:

> Adjust each user’s outcome based on how they behaved *before* the experiment, so we’re comparing like-with-like.

### 6.2 Mathematical Formulation

Let:

- \( Y \): outcome during the experiment (e.g., pageviews, revenue),
- \( X \): pre-experiment covariate (e.g., past pageviews),
- \( \bar{X} \): mean of \( X \),
- \( \theta \): adjustment coefficient.

CUPED defines an **adjusted outcome**:

\[
Y_{\text{adj}} = Y - \theta (X - \bar{X})
\]

where

\[
\theta = \frac{\text{Cov}(Y, X)}{\text{Var}(X)}
\]

Interpretation:

- \( X - \bar{X} \) captures how much a user’s pre-experiment behavior deviates from the average.
- \( \theta \) tells us how strongly pre-experiment behavior predicts the experiment-period outcome.
- Subtracting this term removes the “predictable” part of \( Y \) due to \( X \), reducing noise.

The **average treatment effect** (difference in means between experiment and control) remains the same, but the **variance of the estimator is reduced**.

### 6.3 Implementation in This Project

The original dataset does not contain true pre-experiment metrics, so this project:

- **Simulates** a pre-experiment covariate: `pageviews_prev`, constructed to be correlated with current `Pageviews`.
- Computes \( \theta \) from sample covariance and variance.
- Creates a **CUPED-adjusted metric** `pageviews_cuped` and re-runs significance testing using this adjusted outcome.

This demonstrates the CUPED technique conceptually, which is commonly used in large-scale industrial A/B testing (e.g., at Microsoft, Netflix, etc.).

### 6.4 CUPED Result

For the CUPED-adjusted pageviews metric:

- **Z-score ≈ −0.515**  
- **p-value ≈ 0.6068**

This p-value is **well above 0.05**, indicating **no statistically significant difference** between control and experiment, even after variance reduction.

This strengthens the conclusion that:
> The experiment did not produce a robust, measurable improvement in the tested metric.

---

## 7. Final Conclusion

Putting it all together:

- The experiment **did not significantly improve CTR**.
- Some intermediate funnel metrics (**Enrollment Rate, Payment Rate**) showed statistically significant improvements in the unadjusted analysis.
- After applying **CUPED** to reduce variance and validate robustness, the adjusted analysis shows **no statistically significant overall difference** between control and experiment.
- Therefore, from a conservative, robust statistical standpoint, the new page **does not provide clear, reliable evidence of improvement** over the existing page.

From a product perspective, this would suggest:

> “Do not roll out the new design based solely on this experiment; either iterate further or re-test with more refined changes and/or user-level data.”

---

## 8. Tech Stack

- **Language:** Python 3  
- **Core libraries:**  
  - `pandas`, `numpy` – data manipulation  
  - `matplotlib`, `seaborn` – visualization  
  - `statsmodels` – statistical tests (two-proportion z-tests)  
- **Environment:** Jupyter Notebook

---

## 9. Project Structure

```text
ml-abtest-udacity/
│
├── data/
│   ├── Final Project Baseline Values - Sheet1.csv
│   ├── Final Project Results - Control.csv
│   └── Final Project Results - Experiment.csv
│
├── notebooks/
│   └── 01_ab_test_and_cuped.ipynb   # Full analysis (EDA, A/B test, CUPED)
│
├── requirements.txt
└── README.md
