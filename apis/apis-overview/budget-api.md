# Budget API (v2)

The Budget API allows you to create, update, and delete recurring budget rules to control your Kubernetes spending. Weekly and monthly budgets can be established to set limits on cost spend, with the option to configure alerts for reaching specified budget thresholds via email, Slack, or Microsoft Teams.

{% swagger method="post" path="/model/budget" baseUrl="http://<your-kubecost-address>" summary="Set recurring budget rule or update existing rule" %}
{% swagger-description %}
Creates a recurring budget rule or updates a recurring budget rule when provided the ID of the existing rule.
{% endswagger-description %}

{% swagger-parameter in="body" name="name" type="string" required="true" %}
Name of the budget rule
{% endswagger-parameter %}

{% swagger-parameter in="body" name="values" type="map[string][]string" required="true" %}
Used for specifying the group and name where the budget rule is applied to in the form of two linked values. Accepts `namespace` and `cluster` for the first value, followed by the individual item. For example, when applying a budget rule to a namespace named `kubecost`, this parameter is configured as `values=namespace:kubecost`.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="interval" type="string" required="true" %}
The interval that the budget will reset with (either `weekly` or `monthly`).
{% endswagger-parameter %}

{% swagger-parameter in="body" name="intervalDay" type="int" %}
The day the budget will reset. When `interval=weekly`, `intervalDay` is the day of the week, with `intervalDay=0` for Sunday, `intervalDay=1` for Monday, etc. When `interval=monthly`, `intervalDay` corresponds with the day of the month.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="spendLimit" type="int" required="true" %}
The budget limit value. Currency can be configured in _Settings >_ Cloud Cost Settings > Currency dropdown.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="id" type="string" %}
Only should be used when updating a budget rule, ID of the budget rule being modified. For more info, see the [Using the `id` parameter](budget-api.md#using-the-ip-parameter) section below.
{% endswagger-parameter %}

{% swagger-parameter in="body" name="action" type="[]*BudgetAction" required="false" %} Optional configurations for providing visibility when your budget exceeds a specified percentage threshold. This parameter can generate emails, and Slack or Microsoft Teams messages to suit your work environment. For more information, see the [Using Budget Actions](budget-api.md#Using Budget Actions) section below.
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```
{
    "code": 200,
    "data": [
        {
            "name": "<budget name>",
            "id": "<budget id>",
            "values": {
                "<namespace or cluster>": [
                    "<name of namespace of cluster>"
                ]
            },
            "kind": "",
            "interval": "",
            "intervalDay": ,
            "spendLimit": ,
            "actions": [
                {
                    "amount": 0,
                    "percentage": 1,
                    "slackWebhooks": [],
                    "msTeamsWebhooks": [],
                    "emails": [],
                    "lastFired": ""
                }
            ],
            "resources": {
                "namespaces": [],
                "clusters": []
            },
            "window": {
                "start": "",
                "end": ""
            },
            "currentSpend":
        }
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="get" path="/model/budgets" baseUrl="http://<your-kubecost-address>" summary="Get recurring budget rule(s)" %}
{% swagger-description %}
Lists all existing recurring budget rules
{% endswagger-description %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "code": 200,
    "data": [
        {
            "name": "<budget name>",
            "id": "<budget id>",
            "values": {
                "<namespace or cluster>": [
                    "<name of namespace of cluster>"
                ]
            },
            "kind": "",
            "interval": "",
            "intervalDay": ,
            "spendLimit": ,
            "actions": [
                {
                    "percentage": 1,
                    "slackWebhooks": [],
                    "msTeamsWebhooks": [],
                    "emails": [],
                    "lastFired": ""
                }
            ],
            "resources": {
                "namespaces": [],
                "clusters": []
            },
            "window": {
                "start": "",
                "end": ""
            },
            "currentSpend":
        }
}
```
{% endswagger-response %}
{% endswagger %}

{% swagger method="delete" path="/model/deleteBudget" baseUrl="https://<your-kubecost-address>" summary="Delete recurring budget rule" %}
{% swagger-description %}
Deletes a budget rule defined by `id`
{% endswagger-description %}

{% swagger-parameter in="path" name="id" type="string" required="true" %}
ID of the recurring budget rule to be deleted
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```javascript
{
    "code": 200,
    "data": []
}
```
{% endswagger-response %}
{% endswagger %}

## Formatting parameters when creating/updating budget rules

Creating and updating recurring budget rules uses POST requests, which will require submitting a JSON object in the body of your request instead of adding parameters directly into the path (such as when deleting a recurring budget rule). See the [Examples](budget-api.md#examples) section below for more information on formatting your requests.

## Using the `id` parameter

The `id` parameter when using the endpoint `/budget` is considered optional, but its use will vary depending on whether you want to create or update a budget rule.

When creating a new budget rule, `id` should not be used. An ID for the budget rule will then be randomly generated in the response. When updating an existing budget rule, `id` needs to be used to identify which budget rule you want to modify, even if you only have one existing rule.

The `id` value of your recurring budget is needed to update or delete it. If you don't have the `id` value saved, you can retrieve it using `/budgets`, which will generate all existing budgets and their respective `id` values.

## Using Budget Actions

You can configure greater visibility towards tracking your budgets using the `actions` parameter, which will allow you to create alerts for when your budget spend has passed a specified percentage threshold, and send those alerts to you or your team via email, Slack, or Microsoft Teams.

When providing values for `actions`, `percentage` refers to the percentage of `spendLimit` which will result in an alert. For example, if `"spendLimit": 2000` is configured for a weekly budget rule and `"percentage": 50` is configured, an alert will be sent to all listed emails/webhooks if spending surpasses $1000 USD for the week.

```
"actions" : [
            {
                "percentage": 100,
                "slackWebhooks": [
                    "<example Slack webhook>"
                ],
                "emails": [
                    "foo@kubecost.com",
                    "bar@kubecost.com"
                ],
                "msTeamsWebhooks": [
                    "<example Teams webhook>"
                ]
            }
        ]
```

## Configuring currency

The `spendLimit` parameter will always be determined using your configured currency type. You can manually change your currency type in Kubecost by selecting _Settings_, then scrolling to Currency and selecting your desired currency from the dropdown (remember to confirm your choice by selecting _Save_ at the bottom of the page).

Kubecost does **not** convert spending costs to other currency types; it will only change the symbol displayed in the UI next to costs. For best results, configure your currency to what matches your spend.

## Examples

**Create a recurring budget rule for my test cluster which resets every Wednesday with a budget of $100.00 USD, and will send an alert via email when spending has exceeded 75% of the spend limit.**

```
curl --location '<your-kubecost-address>/model/budget' \
--header 'Content-Type: application/json' \
--data-raw '{
        "name": "budget-rule",
        "values": {
            "cluster":["test"]
        },
        "kind": "soft",
        "interval": "weekly",
        "intervalDay": 3,
        "spendLimit": 100,
        "actions" : [
            {
                "percentage": 75,
                "emails": [
                    "foo@kubecost.com",
                ]
            }
        ]
}'
```

**Create a recurring budget rule for my `kubecost` namespace which resets on the 1st of every month with a budget of $400.00 USD, and will send an alert via Slack and Microsoft Teams when spending has exceeded $100.00 of the spend limit.**

```
curl --location '<your-kubecost-address>/model/budget' \
--header 'Content-Type: application/json' \
--data-raw '{
        "name": "budget-rule-2",
        "values": {
            "namespace":["kubecost"]
        },
        "kind": "soft",
        "interval": "monthly",
        "intervalDay": 1,
        "spendLimit": 400,
        "actions" : [
             {
                "percentage": 25,
                "slackWebhooks": [
                    "<example Slack webhook>"
                ],
                "msTeamsWebhooks": [
                    "<example Teams webhook>"
                ]
            }
        ]
}'
```
