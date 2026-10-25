# Assess Impact, Fairness, Privacy, and Oversight

## Learning objectives

Identify who can be affected by an AI system, find risks in its data and use, and write practical safeguards that can be monitored.

## 1. Start with the problem, not the model

Ask whether the task should be automated, what alternatives exist, who benefits, who bears the cost of errors, and who can challenge an outcome. A prediction score does not decide what action is fair or appropriate. Define intended users, affected non-users, and contexts where the system should not be used.

## 2. Fairness and representation

Data can underrepresent groups, encode past decisions, or use proxies for sensitive characteristics. Overall accuracy may hide large differences in error rates. Where lawful and appropriate, measure data coverage and key metrics for relevant groups and intersections. Small subgroup samples create uncertainty; report sample sizes and intervals rather than overconfident rankings.

Fairness measures can conflict. Equal false-positive rates, equal false-negative rates, calibration, and equal selection rates are different goals and cannot always all be satisfied simultaneously. Choose criteria based on the context, consult affected stakeholders, and explain trade-offs. Removing a sensitive column does not remove its proxies.

## 3. Privacy and data governance

Collect only data needed for the defined purpose. Check consent, permissions, licenses, retention, access, and deletion requirements before training. Avoid including secrets or personal information in public examples and logs. Aggregation or de-identification reduces some risk but does not automatically eliminate re-identification.

## 4. Explainability and communication

Explain the system's role, input requirements, uncertainty, known limitations, and what a user can do when it is wrong. Feature importance is not a causal explanation; a plausible explanation is not proof that a model used the stated reason. Avoid presenting confidence scores as guarantees unless they are calibrated and understood.

## 5. Human oversight and recourse

Set explicit review boundaries for high-impact or uncertain cases. Reviewers need time, information, authority, and training to override a recommendation; a human click-through that cannot meaningfully change the result is not effective oversight. Provide a correction or appeal path where appropriate and track whether it works.

## 6. Monitor after release

Monitor quality, subgroup behavior, incidents, user feedback, and changes in input data. Define alert owners and actions. Reassess risks when the model, data, interface, or deployment context changes. Maintain a record of versions, evaluations, known issues, and decisions.

## Practice

Imagine an image classifier used to sort product photos. List the people affected, groups/cameras/lighting conditions missing from the dataset, consequences of false predictions, data permissions, and a human correction route. Write three release conditions that must be met before broader use.

## Common mistakes

- Assuming a technical metric alone defines fairness.
- Treating removal of protected attributes as proof that a model is unbiased.
- Collecting sensitive data “just in case” without need or permissions.
- Calling a human review step effective without checking whether reviewers can intervene.
- Publishing an average score without the limits, context, and affected groups.
