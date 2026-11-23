# Maintenance process

The [maintenance workflows](model-hub-maintenance.md) find problems automatically. This page describes how the Ersilia team acts on them.

## Roles

The Lead Software Engineer oversees maintenance: they coordinate the work, assign tasks in the weekly technology meeting, and follow up on them.

Whoever is assigned a model or an issue is responsible for it until it's resolved: they track its progress, coordinate the fix, and ask other team members for help when they need it. If they can't finish the task, or find something blocking it, they tell the Lead Software Engineer as early as possible, so the task can be reassigned or the priorities changed.

## When a model fails

Every Monday, the weekly tests check 10 models. For each model that fails, a "Failed weekly maintenance tests" issue is opened in its repository and its status changes to `In maintenance`. The team then:

{% stepper %}
{% step %}
#### Assign the model

In the weekly technology meeting, the team reviews the [weekly testing](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/weekly_model_testing.md) and [failing models](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/failing_models.md) reports, and assigns each failing model to a team member.
{% endstep %}

{% step %}
#### Diagnose and fix it

The assigned person finds the cause of the failure and fixes it. The [maintenance skills](maintenance-skills.md) can do much of this work.
{% endstep %}

{% step %}
#### Test it again

Once the fix is merged, they run **Test Single Model** in ersilia-maintenance. If the model passes, its status goes back to `Ready` and its maintenance issue is closed automatically.
{% endstep %}

{% step %}
#### Report back

They confirm in the next technology meeting that the model works again.
{% endstep %}
{% endstepper %}

The goal is to fix each model before the next weekly meeting.

## Reviewing open issues

The weekly tests skip models with open issues, so these need their own review. Each week, a team member reviews a few models with open issues, usually five, to update their status, add any new information, and decide whether the issue can be closed or needs more work.

## Resolving issues

The team aims to resolve maintenance issues within a week. When an issue can't be closed in that time, update it with:

* Its current state.
* What's blocking it.
* When it's expected to be resolved.

Regular updates matter more than closing issues quickly, so that problems stay visible and decisions are recorded. An issue can be closed when the problem is resolved, worked around, or deliberately postponed with a reason.

## Monthly review

Once a month, the technology meeting looks at the state of the whole Hub:

* The [monthly health report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/monthly_health_report.md) and its trends.
* Recurring problems and long-term priorities.
* Models whose original source code has changed, from the [updated source code report](https://github.com/ersilia-os/ersilia-maintenance/blob/main/reports/updated_models.md).
* The list of open issues.
