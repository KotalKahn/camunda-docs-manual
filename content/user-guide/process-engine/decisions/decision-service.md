---

title: 'Decision Service in the Process Engine'
weight: 20

menu:
  main:
    name: "Decision Service"
    identifier: "user-guide-process-engine-decisions-service"
    parent: "user-guide-process-engine-decisions"
    pre: "Evaluate Decisions"
---

You can also execute decisions directly through the Decision Service. The
Decision Service allows you to executed a deployed decision by Decision
Definition Id, Decision Definition Key and Decision Definition Key + Version.
After the execution you can access the decision result. Please see the
following example:

```java
DecisionService decisionService = processEngine.getDecisionService();

VariableMap variables = Variables.createVariables()
  .putValue("status", "bronze")
  .putValue("sum", 1000);

DmnDecisionResult decisionResult = decisionService
  .evaluateDecisionByKey("decision", variables);

String result = decisionResult.getFirstOutput().get("result");
```

# Evaluating a Decision

## The Decision Key

## Passing Data

# Authorizations for Evaluating Decisions

The user need the permission `CREATE_INSTANCE` on the resource `DECISION_DEFINITION` to evaluate decisions. The resource id of the authorization is the key of the decision definition.

For more information about authorization please refer to the [Authorization Service]({{< relref "user-guide/process-engine/authorization-service.md" >}}).

# Working with the Decision Result

# History of Evaluated Decisions