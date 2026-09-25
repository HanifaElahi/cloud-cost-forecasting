# How the model works, and how I got there

*The detailed walkthrough behind the case study — how I built the forecast step by step, including the approaches I tested and dropped.*

The main question was simple: **how does cloud cost change as usage grows?**

I started by checking whether the cost was tied to demand. When that didn't explain the bill, I looked at what the cost was actually made up of. I then tested a simple per-user approach, but that also didn't fit the pattern.

That led to the final approach: **model the infrastructure in steps, and let the cost change when more capacity is needed.**

**The whole method in one line:**

![How the cost is worked out](../assets/0_method_flow.png)

---

## Step 1 — I first checked whether cost followed demand

**Question:** Can we link cloud cost to leads, enrolments, and revenue?

I put the monthly cloud cost next to leads, enrolments, and revenue and looked at how they changed month to month.

**What I found:** They didn't really move together.

Leads varied quite a bit between the quieter and busier months, while the cloud cost stayed fairly flat.

![More leads didn't move the bill](../assets/1_demand_vs_cost.png)

This meant a model based on demand wouldn't work. Something like "cost per lead" or "cost as a percentage of revenue" would assume the bill goes up when demand goes up — but the bill wasn't doing that.

**So I dropped the demand-based approach.**

---

## Step 2 — I looked at what was actually driving the bill

**Question:** If demand isn't driving the bill, what is?

I broke the bill into two parts:

* **Fixed cost** — servers, database, storage, and other costs that stay in place regardless of how many people use the platform.
* **Variable cost** — costs that change as usage increases, such as data transfer.

**What I found:** Most of the bill was fixed, while a smaller part changed with usage.

That explained why the bill stayed fairly flat even when the number of leads changed.

Most of the cost was coming from the infrastructure itself, rather than the number of leads.

**So I started the forecast with the fixed cost, and then looked at how the variable part changed with usage.**

---

## Step 3 — I tested a simple per-user model

**Question:** Could we explain the variable part with a cost per user?

Since the bill had a fixed and a variable part, my first try was to model them separately — a fixed base, plus a cost per user.

For example, a base of $8,000 a month and $1 per user:

* 5,000 users → $13,000
* 10,000 users → $18,000
* 50,000 users → $58,000

Simple, but it didn't match what the data was showing.

Back in Step 1, the bill hadn't climbed smoothly — it stayed fairly flat, then jumped when more capacity was needed. A per-user model assumes the opposite: a little more cost with every user, in a straight line.

So the cost looked more like:

**flat → jump → flat → jump**

rather than a straight line.

**I dropped the per-user model because it wouldn't capture these jumps.**

---

## Step 4 — I changed the model to work in steps

**Question:** If the cost moves in steps, can we model those steps directly?

The jumps happen when the current setup reaches its limit and we need a bigger one. So instead of directly estimating the cost, I modelled which setup is needed and let the cost follow from that.

I built the forecast around different infrastructure sizes — small, medium, large, and so on.

For each month, the model checks:

* how many users are active
* how many users we expect at the busiest point
* which setup is needed
* what that setup costs
* the small usage-based cost on top

The setup only changes when we reach its limit. So going from 10,000 to 20,000 users doesn't mean the cost doubles. We stay on the same setup until it reaches its limit, then move to the next one.

![Cost rises in steps rather than a straight line](../assets/2_step_function.png)

This gives the forecast a clearer view of when the bigger cost increases happen.

**So this became the final model.**

---

## Step 5 — I then looked at the cost per user

**Question:** Once the model is built, what does the cost look like per user?

I took the total cost at each infrastructure size and divided it by the number of users.

There isn't one fixed cost per user. It changes depending on how many users are sharing the same infrastructure.

When there are fewer users, the fixed cost is spread across fewer users, so the cost per user is higher. As the user base grows, the same cost is spread across more users, so the cost per user goes down.

![Cost per user falls as the user base grows](../assets/3_cost_per_user.png)

For example, with a fixed cost of $20,000 a month:

|     Users | Cost per user |
| --------: | ------------: |
|    20,000 |         $1.00 |
|   200,000 |         $0.10 |
| 2,000,000 |         $0.01 |

The total bill hasn't changed. The same fixed cost is simply being shared across more users.

That's why using one fixed cost-per-user number would be misleading — it depends on the size of the user base.

---

## What is based on data vs. what is an assumption

**Based on historical data:**

* the starting fixed cost
* the split between fixed and variable costs

**Assumptions in the forecast:**

* how large each future infrastructure step will be
* what percentage of users will be active at the busiest point

Both are kept as inputs, so they can be changed without rebuilding the model.

---

## In short

I started by checking whether demand explained the cloud bill. It didn't.

I then broke the bill into fixed and variable costs and tested whether a simple cost-per-user approach could explain the variable part. It couldn't, because the cost didn't increase smoothly with every additional user.

The final model therefore treats infrastructure as a series of capacity steps. Costs stay relatively flat until more capacity is needed, then move up to the next level.

The same model can then be used to see **when those cost increases happen and how the cost per user changes as the user base grows.**

---

*Note: All figures in this walkthrough are illustrative. The dataset is synthetic and was created solely for this case study. It does not represent any real company's data, infrastructure, users, or cloud costs.*
